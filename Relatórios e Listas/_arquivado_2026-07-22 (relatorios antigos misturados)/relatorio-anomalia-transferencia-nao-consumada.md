# Relatório Executivo — Anomalia "Transferência Não Consumada" (BOT Betnacional)

**Data:** 21/07/2026 · **Autor da análise:** Claude (apoio) · **Dono:** Victor (Customer Service, NSX/Betnacional)
**Escopo:** tickets de junho/2026 com as tags `resolvido_ia` **e** `transferencia_saida` na marca BOT Betnacional.
**Fonte:** API Zendesk (somente leitura) + Genie/Databricks. Base de data: `created_at` (UTC). Denominador oficial de junho = **146.778**.
**Status:** causa raiz **identificada** (incidente técnico da plataforma) — confirmação final pendente de correlação com log de deploy/incidente.

---

## 1. Sumário executivo

Em junho/2026, **6.069 atendimentos** foram marcados simultaneamente como "retido pelo bot" (`resolvido_ia`) **e** "transferido para humano" (`transferencia_saida`) — uma combinação que não deveria existir. A investigação (leitura ticket-a-ticket via API + agregados no Genie) concluiu que esses são **clientes que pediram atendimento humano, tiveram seus dados coletados pelo bot, mas nunca chegaram a um agente** — e acabaram fechados automaticamente como se o bot os tivesse retido.

A causa **não é do fluxo conversacional do bot nem falta de agente**: é um **problema técnico da plataforma no mecanismo de transferência**, **concentrado em uma janela de ~5 dias (18 a 22 de junho de 2026, pico no dia 19)**. Fora dessa janela, o fenômeno é residual (5 casos no dia 11; 27 em todo o mês de julho).

**Impacto:** 6.069 clientes que precisavam de humano não foram atendidos e hoje contam indevidamente como "retenção" do bot. São recuperáveis (a conversa pode ser reaberta) e há lista nominal (por `ticket_id`) disponível.

---

## 2. O que é a anomalia (definição e volume)

| Métrica | Valor |
|---|---|
| Tickets com `resolvido_ia` + `transferencia_saida` (marca BOT, junho) | **6.069** (Genie) · 6.068 (API) |
| % da marca BOT Betnacional (87.460 retidos) | **6,94%** |
| % do total de junho (146.778) | **4,13%** |
| Retenção "pura" (`resolvido_ia` sem transferência) | 81.391 |
| **Retenção reportada hoje** (81.391 + 6.069) | **87.460 (59,59%)** |

O padrão típico do ticket: `welcome_entrada` → fluxo de dúvida → `transferencia_entrada` (coleta de nome/CPF/motivo) → `transferencia_saida` + rótulo "transferido" → **sem agente** → `conversa_inativa` (inatividade 15 min) → `resolvido_ia` → gatilho "Resolvidos IA" move para a marca BOT e fecha.

---

## 3. Causa raiz — incidente técnico da plataforma (18–22/jun)

**A anomalia é um incidente, não um defeito crônico.** Distribuição dos 6.068 por dia (API, horário de Brasília):

| Dia (jun/2026) | Dia da semana | Tickets |
|---|---|---|
| 11 | Quarta | 5 (fundo) |
| **18** | Quinta | 888 |
| **19** | **Sexta** | **1.773 (pico)** |
| **20** | Sábado | 1.689 |
| **21** | Domingo | 1.250 |
| **22** | Segunda | 463 |
| **Total 18–22** | | **6.063 (99,9%)** |

- **99,9% dos casos ocorreram entre 18 e 22 de junho**, em **todas as horas do dia** (não é problema de horário/turno).
- **Julho ≈ 0 (27 casos no mês inteiro)** → o incidente terminou/foi corrigido na virada do mês.
- O bot emitiu, no ponto da transferência, sua **mensagem de erro técnico** (template padrão, em inglês): *"I'm really sorry, there seems to be a technical problem. I hope I'll be back to working order soon."*, seguida do encerramento por inatividade — **verificado visualmente nos tickets pelo Dono**.

**Conclusão:** o mecanismo de transferência/handoff (integração AI agent → fila humana) sofreu uma **indisponibilidade técnica entre 18 e 22/jun**; toda solicitação de humano nesse período caiu no erro e foi fechada como retida.

---

## 4. Evidências que sustentam a conclusão

1. **Nenhum agente humano envolvido** (amostra de 100 tickets, junho, marca BOT, as duas tags):
   - 100% entraram no fluxo de transferência (`transferencia_entrada`);
   - **0% tiveram agente designado** (`assignee` vazio);
   - **0 de 15** tiveram qualquer comentário de agente humano (só cliente + bot);
   - **82%** foram encerrados por inatividade (`conversa_inativa`).
2. **Disponibilidade descartada:** o atendimento humano é **24/7** (confirmado pela operação) e a distribuição horária das transferências **não consumadas** (marca BOT) é praticamente **idêntica** à das **consumadas** (marca Bet Nacional) — 75,1% vs 81,4% em horário comercial, mesma curva (Genie, junho). Havia agente disponível; a transferência simplesmente não conectou.
3. **Assinatura de plataforma uniforme:** `resolution_tier = assisted_escalation` em 5.944/6.068 e `channel_group = digital` — comportamento consistente de um mesmo mecanismo falhando.
4. **Concentração temporal** (seção 3): incompatível com desistência do cliente ou design; compatível com outage.

---

## 5. Fluxos de origem — onde a transferência falhou

Dos 6.068, **2.254 passaram por um único fluxo** antes da transferência (atribuição inequívoca). Os fluxos que mais alimentaram transferências falhas:

| Fluxo de origem | Tickets (fluxo único) |
|---|---|
| Promoções | 459 |
| Ainda não recebi a aposta | 454 |
| Informação de saque | 341 |
| Dúvida mercado/cassino | 283 |
| Validação de identidade | 244 |
| Encerrar aposta | 89 |
| Bônus não recebido | 78 |
| Informação de depósito | 77 |
| Cadastro | 57 |

Os demais (≈63%) passaram por 2+ fluxos antes de transferir (a ordem exata não é rastreável no Zendesk — ver §7). **Leitura:** o incidente atingiu justamente os intents de **maior valor e urgência** — pagamento, aposta pendente, saque e KYC/identidade.

---

## 6. Impacto

- **6.069 clientes** pediram atendimento humano, entregaram seus dados e **não foram atendidos** — hoje classificados como "retenção do bot".
- **Distorção de leitura:** a retenção de junho (59,59%) deve ser reportada **decomposta**: 81.391 retenção genuína + **6.069 transferência não consumada** (falha de experiência, não retenção real).
- **Recuperáveis:** a conversa pode ser reaberta; existe lista nominal por `ticket_id` para reabordagem proativa (§8).
- **Reputacional/operacional:** concentrado em intents sensíveis (dinheiro, aposta, identidade), num fim de semana de alto volume (19–21/jun).

---

## 7. Recomendações

1. **Confirmar o incidente** correlacionando a janela **18–22/jun** com: log de alterações do AI agent, status/incidentes do Zendesk/Sunshine e deploys internos.
2. **Validação nos dados (rápida):** verificar se as transferências **consumadas** (marca Bet Nacional) **caíram** em 18–22/jun — a queda espelhada confirma o desvio de transferências bem-sucedidas para falhas.
3. **Recuperação:** reabordar os 6.069 clientes (lista disponível), priorizando saque/aposta/identidade.
4. **Prevenção:** alerta/monitor sobre erros do nó de transferência; KPI mensal **"transferência não consumada → 0"**; corrigir a inconsistência de tag `recoperacao_conta` vs `recuperacao_conta`.
5. **Reporte:** sempre decompor a retenção (pura vs transferência não consumada) no relatório e no dashboard.

---

## 8. Metodologia, rastreabilidade e limitações

- **Somente leitura (GET).** Nenhuma escrita na conta Zendesk. Nenhum dado de cliente salvo em disco; listas exportadas contêm apenas metadados (`ticket_id`, `created_at`, `status`).
- **Contagem fecha exata:** enumeração via `/search/export` retornou 6.095 all-time (6.068 junho + 27 julho), batendo com a contagem — 100% rastreável.
- **Limitações conhecidas:**
  - A **ordem intra-conversa** das tags **não existe no Zendesk** (todas gravadas na materialização do ticket); a sequência real vive no Ultimate/Sunshine.
  - A **mensagem de erro** aparece no transcript do Sunshine (tela do Zendesk), **não** nos comentários do Data Lake — por isso a confirmação do texto é visual, não por API em massa.
  - A **Search API** deste tenant não faz AND de múltiplas tags nem filtro de data confiável; contagens oficiais vêm do Genie/Databricks; enumerações via `/search/export` + filtro local.

## Anexos

- **Listas (metadados):** `integracao/anomalia_transf_nao_consumada_junho.csv` (6.068) · `integracao/anomalia_transf_nao_consumada_julho.csv` (27).
- **Tickets de exemplo (para inspeção no Zendesk):** janela do incidente — `2698488` (18/jun), `2727315`, `2727317`, `2727305`, `2727369` (19–20/jun); fundo — `2660011`, `2659940`, `2659978` (11/jun).
- **Scripts read-only:** `integracao/anomalia_6069.py`, `anomalia_6069_tally.py`, `anomalia_deep.py`, `anomalia_fluxo_origem.py`, `anomalia_padroes_junho.py`.
