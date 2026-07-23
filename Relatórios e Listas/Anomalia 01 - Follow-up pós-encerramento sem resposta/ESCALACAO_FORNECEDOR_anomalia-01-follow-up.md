# Pacote de escalação — Anomalia 01: bot não reengaja follow-up após encerramento da conversa

**Preparado:** 22/07/2026 · Projeto Métricas BOT Betnacional (Victor Costa, victor.costa@nsx.bet)
**Destino:** suporte Zendesk — produto **AI agents - Advanced** (ex-Ultimate AI)
**Severidade proposta:** Alta — ~40–60 clientes/dia ficam **sem qualquer atendimento** ao escrever de novo após um atendimento encerrado, de forma contínua desde 29/06/2026.
**Substitui:** o pacote "órfãos de silêncio" de 22/07 (diagnóstico corrigido — os exemplos eram follow-ups, não conversas novas; pacote antigo arquivado).

---

## 1. Resumo executivo

Desde **29/06/2026**, quando o AI agent **encerra uma conversa** (cliente respondeu o BSAT, ou 15 min de inatividade) e o ticket é fechado, **qualquer nova mensagem do cliente na mesma conversa do widget cai no vazio**: o Zendesk cria um novo ticket (o anterior está fechado) atribuído ao usuário integrado "AI Agent", mas **o bot nunca reengaja** — nenhuma resposta, nenhum menu, nenhuma tag de diálogo, e **nenhuma conversa nova é criada no motor do bot** (Data Export v3). O ticket morre fechado por automação sem que ninguém o veja.

- **Julho (01–21): 817 clientes** nessa condição (~39/dia; pico 64/dia). Junho 29–30: 88. **Total desde a regressão: 905.**
- **92,7% dos casos**: o cliente tinha acabado de **responder o BSAT** (muitas vezes com nota máxima) e emendou uma mensagem — "obrigado", "de nada", ou um problema novo. Gap mediano entre o fechamento e a nova mensagem: **15 segundos** (78,6% ≤ 30s; 99,8% ≤ 2min).
- Antes de 29/06 esse volume era **~1 caso/dia** (junho 01–28 = 32 no total). O salto foi de um dia para o outro: 28/06 = 0 → **29/06 = 36 → 30/06 = 52 → 01/07 = 62** — sem nenhuma mudança de configuração do nosso lado (audit log 24/06–02/07 revisado item a item).
- **Comportamento esperado** (e observado até 28/06): nova mensagem após o encerramento → novo ticket → **o bot reengaja com o menu inicial**, como uma nova iteração.

**English summary (for vendor):** Since 2026-06-29, when the AI agent ends a conversation (BSAT answered or 15-min inactivity) and the ticket is closed, **any follow-up message the customer sends in the same widget conversation is lost**: Zendesk creates a new ticket (assigned to the integrated "AI Agent" user), but **the bot never re-engages** — zero replies, zero dialog tags, and **no new conversation is created in the bot engine** (verified via Data Export v3). July 01–21 = **817 affected customers** (~39/day, peak 64); 92.7% had just answered the BSAT survey seconds earlier (median gap between ticket closure and the follow-up: **15 seconds**). Before 06-29 this was ~1/day; the jump was overnight (0 → 36 → 52 → 62/day) with **no configuration change on our side** (Zendesk audit log 06-24→07-02 checked). Expected behavior (and observed until 06-28): a follow-up after closure should open a new ticket **and the bot should re-engage with the welcome menu**. We need the AI agent platform changelog around 06-29 and the conversation-lifecycle/delivery logs for the sample pairs below.

## 2. Identificação do ambiente

| Item | Valor |
|---|---|
| Subdomínio Zendesk | `nsxhelp` |
| Bot (AI agents - Advanced) | `695e6d4ce3944b22197c9a7e` ("nsxhelp-chat") |
| Organization ID (Ultimate) | `695e6d4c027f1cd2e5f96400` · região US |
| Integração Web Widget | snippet key `3dca589b-c1f3-4df2-92fc-1c59d596c379` · integrationId `66c38d68f29ff3076dd7bbb3` |
| Usuário integrado "AI Agent" | `50032929128731` |
| Marca afetada | Bet Nacional (`28331414811547`) |
| Config relevante do widget | "Permitir que os clientes iniciem várias conversas" = ATIVO |

## 3. Assinatura do par de tickets (fingerprint)

**Ticket-pai (o atendimento que encerrou):** canal `native_messaging`, atendido ponta a ponta pelo AI agent (`advanced_ia` + tags de fluxo), encerrado por BSAT (`bsat_saida` + `resolvido_ia`, 92,7%) ou inatividade (`conversa_inativa` + `resolvido_ia`, 7,3%); fechado por trigger na criação.

**Ticket órfão (o follow-up):** criado **segundos depois** (mediana 15s) pelo **mesmo requester**, na mesma conversa do widget:
1. Canal `native_messaging`, assunto "Conversation with Web User …";
2. Assignee = usuário integrado "AI Agent" (50032929128731), grupo vazio;
3. **Só comentários do cliente — nenhuma resposta do bot**;
4. **Nenhuma tag de diálogo** (sem `advanced_ia`, sem `welcome_entrada`, sem `*_entrada`/`*_saida`). A partir de ~09/07, parte recebe as auto-tags do triage nativo (`intent__*`, `language__*`, `sentiment__*`) — sem mudar o comportamento;
5. Nasce `open`; é fechado por automação sem atendimento (julho: 584 closed / 115 solved / 120 open);
6. **Não existe como conversa nova no Data Export v3 do bot.**

## 4. Evidências

### 4.1 Censo do ticket-pai (22/07) — a anomalia é follow-up, não conversa nova

Para **cada** órfão de jun+jul (idade ≥ 24h; 1.143 tickets), buscamos via API o ticket anterior do mesmo requester:

| Categoria | Junho | Julho (01–21) |
|---|---|---|
| **Follow-up** (pai com `advanced_ia` fechado ≤ 15 min antes) | 120 (88 em 29–30/06) | **817 (93,1%)** |
| Conversa nova (sem ticket anterior) / pai de meses atrás | 142 | 61 (baseline antiga, ~3/dia, estável) |

A distribuição é **bimodal perfeita**: ou o pai fechou ≤ 2 min antes (99,8% dos follow-ups), ou o ticket anterior é de meses atrás. Zero casos intermediários.

### 4.2 A nova mensagem nunca chega ao motor do bot

Data Export v3 de 15–21/07: **23.020 conversas, zero com `bot_messages_count = 0`** — e os follow-ups órfãos **não geram conversa nova no export**. A conversa-pai existe, foi respondida e encerrada normalmente; a mensagem seguinte do cliente não produz **nenhum** evento no lado do bot. → A falha está no **ciclo de vida da conversa pós-encerramento / entrega do follow-up** (plataforma AI agent ↔ Sunshine), não no conteúdo dos fluxos.

### 4.3 Linha do tempo (por dia, UTC, idade ≥ 24h)

- Baseline até 28/06: **~1,1/dia** (junho 01–28 = 32).
- **Salto abrupto:** 28/06 = 0 → **29/06 = 36 → 30/06 = 52 → 01/07 = 62** → sustentado em 20–64/dia até hoje (proporcional ao volume do canal).

### 4.4 Nenhuma mudança de config no Zendesk

Audit log de 24/06 a 02/07 revisado item a item: nenhuma alteração de trigger de roteamento, webhook, target ou app perto de 29/06.

### 4.5 Pistas adicionais

- **~09/07:** auto-tags `intent__*`/`sentiment__*` passaram a aparecer nos follow-ups órfãos (triage nativo) — cosmético, mas indica release de plataforma no período; confirmar se tocou o ciclo de vida de conversas.
- O mesmo cliente que é ignorado no follow-up **é atendido normalmente se voltar horas depois** (nova conversa) — a falha é específica da janela pós-encerramento na mesma conversa.

## 5. Exemplos para inspeção (pares pai → órfão, sem resposta até 23/07 00:00 UTC)

| Ticket-pai (atendido, fechado) | Órfão (sem resposta) | Criado (UTC) | Gap | Encerramento do pai |
|---|---|---|---|---|
| #2858604 | **#2858616** | 21/07 21:47 | 7s | BSAT respondido |
| #2858685 | **#2858697** | 21/07 22:13 | 12s | BSAT (nota máxima 4 min antes) |
| #2858730 | **#2858736** | 21/07 22:29 | 16s | BSAT |
| #2858779 | **#2858787** | 21/07 22:50 | 19s | BSAT |
| #2854008 | **#2854009** | 20/07 13:32 | 4s | BSAT (caso índice) |
| #2828728 | **#2828731** | 12/07 17:40 | 3s | BSAT (órfão recebeu `sentiment__very_negative`, sem resposta) |
| #2835350 | **#2835356** | 15/07 00:25 | 5s | BSAT |
| #2858990 | **#2859044** | 22/07 00:15 | 73s | inatividade |
| #2858401 | **#2858460** | 21/07 20:45 | 35s | inatividade |
| #2857710 | **#2857718** | 21/07 16:42 | 6s | BSAT |

Listas completas (metadados, sem PII): `anomalia-01_followup_2026-06.csv` (120) e `anomalia-01_followup_2026-07.csv` (819), nesta pasta.

## 6. O que pedimos ao fornecedor

1. **Changelog/releases da plataforma AI agents - Advanced e da integração Sunshine/Messaging em 28–30/06/2026** (e ~09/07) para a região US — o que mudou no **ciclo de vida da conversa após o encerramento pelo fluxo** (BSAT/inatividade)?
2. **Logs de entrega/roteamento dos pares da seção 5**: quando o cliente enviou a mensagem de follow-up, o que a plataforma fez? A conversa foi considerada encerrada e a mensagem descartada? Houve tentativa de reengajamento rejeitada?
3. Confirmação do **comportamento esperado do produto**: após o bot encerrar a conversa e o ticket fechar, uma nova mensagem do cliente na mesma conversa deveria reengajar o bot (novo ticket + welcome)? Era o que ocorria até 28/06.
4. Se houver **configuração do nosso lado** que controle o reengajamento pós-encerramento (janela de sessão, multi-conversas, comportamento do widget), indicar exatamente qual e o valor recomendado.
5. **Correção e prevenção**: restabelecer o reengajamento + alarme para tickets atribuídos ao bot sem resposta.

## 7. Como reproduzimos a contagem (para auditoria)

- Universo: Zendesk Search `assignee:50032929128731 created>=2026-06-01`, filtrando localmente: sem `advanced_ia` e sem nenhuma tag além de `intent__*`/`language__*`/`sentiment__*`/`logado`, **idade ≥ 24h** (tags do bot são gravadas na materialização; tickets em andamento parecem órfãos e se normalizam — sem o corte, a contagem infla).
- Classificação: para cada órfão, ticket anterior do mesmo requester via `/users/{id}/tickets/requested`; follow-up = pai com `advanced_ia` fechado ≤ 15 min antes (na prática, segundos).
- Scripts read-only: `integracao/censo_pais_orfaos.py` (censo completo) · `integracao/orfaos_contagem_limpa.py` (contagem diária) · `integracao/orfaos_vs_dataexport.py` (cruzamento com Data Export v3).

---
*Gerado na sessão de 22/07/2026 (censo do ticket-pai). Substitui o pacote "órfãos de silêncio" — diagnóstico corrigido pelo Dono + censo.*
