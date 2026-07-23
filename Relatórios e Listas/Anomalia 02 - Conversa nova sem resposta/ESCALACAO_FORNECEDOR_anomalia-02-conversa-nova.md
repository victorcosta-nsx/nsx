# Pacote de escalação — Anomalia 02: conversas novas não entregues ao AI agent

**Preparado:** 22/07/2026 · Projeto Métricas BOT Betnacional (Victor Costa, victor.costa@nsx.bet)
**Destino:** suporte Zendesk — **AI agents - Advanced**. **Recomendação: enviar como item secundário no MESMO ticket da Anomalia 01** (mesma ponte, modos de falha distintos).
**Severidade proposta:** Média — volume pequeno (~3/dia) mas contínuo desde o rollout; cliente fica sem qualquer atendimento.

---

## 1. Resumo

Além do problema principal (Anomalia 01 — follow-up pós-encerramento não reengajado, desde 29/06), existe um modo de falha **mais antigo e menor**: conversas de clientes **sem nenhum ticket anterior** (ou com histórico de meses atrás) criam o ticket no Zendesk, atribuído ao usuário integrado "AI Agent", mas **nunca chegam ao motor do bot** — zero respostas, zero tags de diálogo, e a conversa **não existe no Data Export v3**.

- **Julho (01–21): 61 casos (~2,9/dia)** · junho: 142 (com pico isolado de 22 em **11/06** — mini-incidente). Taxa ~0,1–0,2% do canal, **estável desde o rollout (21/05)** — não mudou em 29/06.
- Fingerprint idêntico ao da Anomalia 01 exceto pela ausência de ticket-pai recente: `native_messaging`, assignee "AI Agent" (50032929128731), só comentários do cliente, sem `advanced_ia`, fechado por automação sem atendimento.

**English summary:** Separate from the post-closure follow-up regression (Anomaly 01), ~3 conversations/day from customers with **no prior ticket** are created in Zendesk (assigned to the integrated "AI Agent" user) but **never reach the bot engine** — zero replies, zero dialog tags, absent from Data Export v3. Rate ~0.1–0.2% of the channel, stable since rollout (2026-05-21), unchanged on 06-29. Intermittent delivery failure in the Web Widget/Sunshine → AI agent bridge. We ask for delivery logs for the samples below and whether the platform has retry/alerting for undelivered conversations.

## 2. Ambiente

Mesmo da Anomalia 01: subdomínio `nsxhelp` · bot `695e6d4ce3944b22197c9a7e` · org `695e6d4c027f1cd2e5f96400` (US) · integrationId `66c38d68f29ff3076dd7bbb3` · usuário "AI Agent" `50032929128731`.

## 3. Exemplos para inspeção (sem resposta até 23/07 00:00 UTC)

| Ticket | Criado (UTC) | Status |
|---|---|---|
| **#2859006** | 22/07 00:02 | open |
| **#2858201** | 21/07 19:20 | open |
| **#2857149** | 21/07 13:40 | open |
| **#2856012** | 21/07 01:22 | open |
| **#2855776** | 21/07 00:03 | open |
| **#2853334** | 20/07 02:23 | open |

Listas completas (metadados): `anomalia-02_conversa-nova_2026-06.csv` (142) · `anomalia-02_conversa-nova_2026-07.csv` (62).

## 4. Pedidos

1. **Logs de entrega** das conversas dos exemplos: a conversa chegou à fila do bot? Foi rejeitada? Timeout?
2. Por que o ticket é criado e atribuído ao "AI Agent" **sem** o bot receber a conversa?
3. Existe retry/alarme na plataforma para conversa atribuída ao bot sem resposta?
4. O pico de 22 casos em **11/06/2026** corresponde a algum incidente registrado do lado da plataforma?

---
*Contagem reproduzível: censo em `integracao/censo_pais_orfaos.py`, categoria CONVERSA_NOVA/PAI_SEM_ADVANCED (idade ≥ 24h).*
