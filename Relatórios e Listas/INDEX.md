# Índice — Relatórios e Listas (BOT Betnacional)

Atualizado: **22/07/2026 (reorganização do zero)**. Regra: **uma anomalia = uma pasta** (relatório HTML + CSVs por mês). Definições canônicas em **`CATALOGO_ANOMALIAS.md`** (ler primeiro). Números provisórios da API (oficial = Databricks/Genie). Listas só com metadados (sem PII).

---

## 📁 Anomalia 01 - Follow-up pós-encerramento sem resposta 🔴 ATIVA (regressão 29/06)
Cliente atendido ponta a ponta; conversa encerra (BSAT 92,7% / inatividade); ele escreve de novo segundos depois (mediana 15s) → novo ticket no vazio, bot nunca reengaja. **Jul 01–21 = 817 (93,1% dos órfãos) · desde 29/06 = 905.**
- `relatorio-anomalia-01-follow-up-sem-resposta.html`
- `ESCALACAO_FORNECEDOR_anomalia-01-follow-up.md` — **pacote PT+EN pronto p/ o Dono abrir ticket no suporte Zendesk** (substitui o pacote antigo, arquivado)
- `anomalia-01_followup_2026-06.csv` (120) · `anomalia-01_followup_2026-07.csv` (819)

## 📁 Anomalia 02 - Conversa nova sem resposta 🟡 baseline antiga (~3/dia)
Cliente sem histórico recente abre conversa e não recebe nada — a conversa nunca chega ao motor. Estável desde o rollout; **não** é a regressão. **Jun = 142 (pico 22 em 11/06) · jul 01–21 = 61.** ⚠ Junho contaminado por casos da An. 03 (reconciliar via Genie O1/O2).
- `relatorio-anomalia-02-conversa-nova-sem-resposta.html`
- `ESCALACAO_FORNECEDOR_anomalia-02-conversa-nova.md` — item secundário p/ o MESMO ticket da An. 01
- `anomalia-02_conversa-nova_2026-06.csv` (142) · `anomalia-02_conversa-nova_2026-07.csv` (62)

## 📁 Anomalia 03 - Formulário nativo (A form was sent) 🔴 ATIVA (~30/dia em julho)
A **resposta padrão nativa do canal Messaging** (autor −1/sistema via `chat_transcript`) atende em vez do AI agent. Genie O1 (22/07): mai 15.642 (maioria pré-rollout) · jun 1.574 · **jul 655 (~30/dia) — não cessou**; 99,98% roteados a humano. Subgrupo "morreu sem preencher" (assignee AI Agent) é invisível ao lake → **182 parcial** via API. Próximas: Genie **O1b** e **O2**.
- `relatorio-anomalia-03-formulario-nativo.html`
- `anomalia-03_formulario_parcial_{2026-05,2026-06,TODOS}.csv`

## 📁 Anomalia 04 - Transferência não consumada 🟢 SÓ JULHO (esperado = zero)
Cliente completa a coleta da transferência e não é entregue à fila humana; fecha como `resolvido_ia` (indevido). **Jul = 27** (mini-rajadas; último 19/07). **Junho excluído dos relatórios por decisão do Dono (22/07)** — incidente interno resolvido; CSV preservado no `_arquivado`.
- `relatorio-anomalia-04-transferencia-nao-consumada.html` (julho-only)
- `anomalia-04_transf-nao-consumada_2026-07.csv`
- ⚠ A pasta antiga `Anomalia de clientes não transferidos/` está com handle aberto (Explorer/navegador) — conteúdo já copiado para cá; **apagar a antiga quando liberar**.

## ~~📁 Anomalia 05~~ — REMOVIDA em 23/07 (e-mail fora de escopo)
Tickets de e-mail **não entram em nenhum relatório** (correção do Dono). O relatório foi apagado; ⚠ resta o CSV travado por outro programa (`anomalia-05_intent-fora-do-chat_2026-07.csv`) — **apagar a pasta quando o arquivo for fechado**. Regra de medição nº 0 no `CATALOGO_ANOMALIAS.md`.

---

## Na raiz desta pasta (transversais)
- **`CATALOGO_ANOMALIAS.md`** — definições canônicas, reclassificações (anomalia D dissolvida; "bot antigo" falso lead; diagnóstico da escalação antiga corrigido) e regras de medição.
- **`WHATSAPP_RESUMO_ANOMALIAS.txt`** — resumo executivo das anomalias em formatação WhatsApp (grupo Zendesk + executivos; reunião 24/07). Colar direto.
- `INDEX.md` (este arquivo) · `mapa_fluxos_inventario.csv` (outra sessão) · `relatorio-consolidado-retencao-bot-betnacional.md` (outra sessão)

## 📁 `_arquivado_2026-07-22 (relatorios antigos misturados)/`
Relatórios/listas antigos em que as anomalias se misturavam (inclui o pacote de escalação SUPERADO dos "órfãos de silêncio" — banner de aviso no topo). **Não usar para números** — só histórico.

## Na RAIZ do projeto (`C:\ClaudeCode\Zendesk\`)
- **`mapa-fluxos-advanced-ai.html`** — mapa MEDIDO dos fluxos (Data Export, 15–21/07): fluxograma geral + drill-down por fluxo (36 fluxos, clique a clique). Linkado pela aba Mapa de Fluxos do dashboard-geral.
- `dashboard-geral.html` e `dashboard-bot-betnacional.html` (outra sessão — só edições aditivas combinadas; 23/07: banner+link para o mapa medido) · `relatorio-sem-advanced-ia.html` (raiz, referenciado pelo dashboard — ⚠ v2 antiga, anterior ao censo; atualizar em sessão combinada)
