# ⛔ SUPERADO (22/07/2026) — NÃO ENVIAR

> **Diagnóstico corrigido pelo censo do ticket-pai:** 93% destes "órfãos" (incluindo os 5 exemplos abaixo) são **follow-ups pós-encerramento** — a conversa CHEGOU ao bot, foi atendida e encerrada; a falha é o não-reengajamento quando o cliente escreve de novo. Usar o pacote novo: `Relatórios e Listas/Anomalia 01 - Follow-up pós-encerramento sem resposta/ESCALACAO_FORNECEDOR_anomalia-01-follow-up.md` (+ o da Anomalia 02 para o resíduo de conversas novas). Mantido só como histórico.

# Pacote de escalação — Conversas do Web Widget NÃO entregues ao AI agent ("órfãos de silêncio")

**Preparado:** 22/07/2026 · Projeto Métricas BOT Betnacional (Victor Costa, victor.costa@nsx.bet)
**Destino:** suporte Zendesk — produto **AI agents - Advanced** (ex-Ultimate AI)
**Severidade proposta:** Alta — clientes ficam **sem qualquer atendimento** (nem bot, nem humano), de forma contínua desde 29/06/2026.

---

## 1. Resumo executivo

Desde **29/06/2026**, cerca de **1,0–1,3% das conversas** iniciadas no Web Widget (canal Messaging/Sunshine) **criam o ticket no Zendesk mas nunca chegam ao AI agent**: o bot não envia nenhuma mensagem, nenhuma tag de diálogo é aplicada e **a conversa não existe no Data Export do bot** (v3). O cliente escreve e não recebe resposta alguma; o ticket fica atribuído ao usuário integrado "AI Agent" até ser fechado por automação, sem que ninguém o veja.

- **Julho (01–21): 878 conversas nessa condição** (média ~42/dia). Regressão ativa até hoje.
- Antes de 29/06 a taxa era ~0,18% (5–13/dia) — o salto foi de **~7×** e ocorreu **de um dia para o outro** (28/06 = 5 → 29/06 = 41 → 30/06 = 58 → 01/07 = 63).
- O **audit log do Zendesk (24/06–02/07) não mostra nenhuma mudança de configuração** do nosso lado (triggers, webhooks, apps, roteamento) que explique o salto → a causa está na plataforma do AI agent / ponte Sunshine.

**English summary (for vendor):** Since 2026-06-29, ~1.0–1.3% of Web Widget (Sunshine Messaging) conversations create a Zendesk ticket but are **never delivered to the AI agent**: zero bot replies, zero dialog tags, and the conversation is **absent from the bot's Data Export v3**. Customers get no response at all. July 01–21 = 878 affected conversations (~42/day), regression started abruptly on 2026-06-29 (5 → 41 → 58 → 63 per day) with **no configuration change on our side** (Zendesk audit log 06-24→07-02 checked). We need the AI agent platform changelog around 06-29 and the Sunshine→bot handoff logs for the sample tickets below.

## 2. Identificação do ambiente

| Item | Valor |
|---|---|
| Subdomínio Zendesk | `nsxhelp` |
| Bot (AI agents - Advanced) | `695e6d4ce3944b22197c9a7e` ("nsxhelp-chat") |
| Organization ID (Ultimate) | `695e6d4c027f1cd2e5f96400` · região US |
| Integração Web Widget | snippet key `3dca589b-c1f3-4df2-92fc-1c59d596c379` · integrationId `66c38d68f29ff3076dd7bbb3` |
| Usuário integrado "AI Agent" | `50032929128731` |
| Marca afetada | Bet Nacional (`28331414811547`) |

## 3. Assinatura do ticket órfão (fingerprint)

Todos os afetados têm exatamente este padrão:

1. Canal `native_messaging` (Web Widget/Sunshine), assunto "Conversation with Web User …";
2. **Assignee = usuário integrado "AI Agent"** (50032929128731), grupo vazio;
3. **1 comentário, só do cliente — nenhuma resposta do bot**;
4. **Nenhuma tag de diálogo do bot** (sem `advanced_ia`, sem `welcome_entrada`, sem `*_entrada`/`*_saida`, sem `resolvido_ia`). A partir de ~20/07 recebem apenas as auto-tags do triage nativo do Zendesk (`intent__*`, `language__*`, `sentiment__*`);
5. Nasce `open`; fecha depois por automação, sem atendimento;
6. **Não existe no Data Export v3 do bot** (verificado — seção 4).

## 4. Evidências

### 4.1 A conversa não chega ao motor do bot (teste discriminante, 22/07)

Baixamos o **Data Export v3** (get-signed-urls) de **15–21/07**: **23.020 conversas, e ZERO com `bot_messages_count = 0`** — toda conversa registrada no bot tem resposta do bot. No mesmo período, o Zendesk tem **224 tickets órfãos** (idade > 24h, sem resposta até hoje). Conclusão: **as conversas órfãs não têm nenhum rastro no lado do bot** — a falha está na entrega da conversa (ponte Web Widget/Sunshine → AI agent), não no conteúdo dos fluxos.

### 4.2 Linha do tempo (contagem por dia, UTC, critério idade > 24h)

- Rollout do Advanced: 21/05/2026. Baseline 22/05–28/06: **5–13/dia (~0,18%)**.
- **Salto abrupto em 29/06**: 28/06 = 5 → **29/06 = 41 → 30/06 = 58 → 01/07 = 63**.
- Julho 01–21 = **878** (pico 04/07 = 70; dia 19/07 = 56 acompanhando pico de volume do canal).

### 4.3 Taxa normalizada pelo volume do canal (conversas no Data Export + órfãos)

| Dia | Conversas no bot (export) | Órfãos Zendesk | Taxa de falha |
|---|---|---|---|
| 01/07 | 5.005 | 63 | **1,24%** |
| 04/07 | 5.246 | 70 | **1,32%** |
| 08/07 | 2.822 | 35 | **1,23%** |
| 12/07 | 3.683 | 47 | **1,26%** |
| 15–21/07 (semana) | 23.020 | 224 | **~0,96%** |

A taxa é estável (~1%) e **proporcional ao volume** — falha intermitente/probabilística, presente em todas as horas do dia, não um outage pontual nem um problema de horário.

### 4.4 Nenhuma mudança de config no Zendesk

Audit log de 24/06 a 02/07 revisado item a item: nenhuma alteração de trigger de roteamento, webhook, target ou app perto de 29/06 (apenas montagem de fluxo de feature nova em 24/06, edições de macro/campo e exclusão de views em 02/07).

### 4.5 Comportamentos correlatos (pistas)

- **Antes de 29/06** o mesmo tipo de conversa não-entregue caía no **responder nativo do Zendesk** (formulário "Fale mais sobre você" → fila humana). Em 29/06 o modo de falha virou **silêncio total**. A raiz (conversa não roteada ao AI agent) parece a mesma; o fallback é que mudou.
- **Follow-up pós-resolução não reengaja o bot:** caso #2854008 → #2854009 (20/07): o bot resolve, o ticket fecha, o cliente escreve de novo **na mesma conversa** segundos depois → novo ticket órfão. Config do widget tem "Permitir que os clientes iniciem várias conversas" ativo.
- **Intermitência por cliente:** o mesmo requester ora é atendido pelo bot, ora cai no vazio (ex.: requester 52708959478427 — ticket #2819827 com bot × #2809525 sem).
- **~20/07** houve outra mudança de plataforma percebida: auto-tags `intent__*` passaram a ser aplicadas em conversas de messaging (inclusive nos órfãos). Confirmar se essa release também tocou o roteamento.

## 5. Exemplos para inspeção (órfãos verdadeiros, sem resposta até 23/07 00:00 UTC)

| Ticket | Criado (UTC) | Estado |
|---|---|---|
| **#2858616** | 21/07 21:47 | open, 0 tags, sem resposta |
| **#2858697** | 21/07 22:13 | open, 0 tags, sem resposta |
| **#2858736** | 21/07 22:29 | open, 0 tags, sem resposta |
| **#2858787** | 21/07 22:50 | open, 0 tags, sem resposta |
| **#2854009** | 20/07 | caso follow-up (par #2854008 resolvido pelo bot na mesma conversa) |

Listas completas (metadados, sem PII): `sem-advanced_A-silencio_2026-07.csv` (esta pasta) e `integracao/lista_orfaos_p1_desde_rollout.csv`.

## 6. O que pedimos ao fornecedor

1. **Changelog/releases da plataforma AI agents - Advanced e da ponte Sunshine em 28–30/06/2026** (e também ~20/07) para a região US;
2. **Logs de entrega das conversas dos tickets-exemplo** (seção 5): a conversa chegou à fila do bot? Foi rejeitada? Timeout?
3. Explicação de por que o ticket é criado e atribuído ao "AI Agent" **sem** o bot receber a conversa;
4. Correção e prevenção (alarme para conversas atribuídas ao bot sem resposta);
5. Orientação sobre o caso follow-up (#2854008→#2854009): conversa reaberta após resolução deveria reengajar o bot?

## 7. Como reproduzimos a contagem (para auditoria)

- Zendesk Search: `assignee:50032929128731 created>=2026-06-01`, filtrando localmente: sem `advanced_ia` e sem nenhuma tag além de `intent__*`/`language__*`/`sentiment__*`/`logado`.
- **Regra de idade ≥ 24h** (tags do bot são gravadas na materialização da conversa; tickets recém-criados em andamento parecem órfãos e se normalizam sozinhos — sem o corte de idade a contagem infla).
- Data Export v3: `POST /data-export/v3/get-signed-urls` por dia; conversas com `bot_messages_count = 0` = zero em 23.020.

---
*Gerado na sessão de 22/07/2026. Scripts read-only: `integracao/orfaos_vs_dataexport.py`, `orfaos_contagem_limpa.py`, `export_volume_dias.py`, `checar_rajada_18h.py`.*
