# Catálogo canônico de anomalias — BOT Betnacional

**Criado em 22/07/2026** (reorganização do zero por ordem do Dono — relatórios antigos misturados foram para `_arquivado_2026-07-22`).
**Regra: uma anomalia = uma pasta = um relatório HTML + listas CSV separadas por mês (jun/jul).** Não misturar anomalias: padrões diferentes, causas provavelmente diferentes.
Fonte dos números: censo do ticket-pai sobre 1.143 órfãos jun+jul (`integracao/censo_pais_orfaos.{py,csv}`, 22/07, idade ≥ 24h) + API Zendesk read-only + Data Export Ultimate. Contagem oficial = Genie/Databricks.

---

## Anomalia 01 — Follow-up pós-encerramento sem resposta 🔴 ATIVA (regressão de 29/06)

**Nome do Dono:** "Cliente finalizou atendimento, enviou uma nova mensagem e não recebeu resposta."
**Mecânica:** o Advanced AI atende ponta a ponta; a conversa encerra (BSAT respondido ou inatividade) e o gatilho fecha o ticket. O cliente escreve de novo **na mesma conversa do widget segundos depois** (mediana 15s; 78,6% ≤ 30s) → como o ticket fechou, nasce um **novo ticket** atribuído ao "AI Agent" — e o bot **nunca reengaja** (sem welcome, sem menu, sem `advanced_ia`, sem resposta).
**Comportamento esperado (Dono):** nova mensagem após encerramento → novo ticket → bot reengaja com o menu inicial.

- **Números:** julho 01–21 = **817** (93,1% dos 878 órfãos limpos) · junho = 120, sendo **88 só em 29–30/06** e ~32 espalhados (baseline ~1,1/dia pré-29/06). Desde a regressão (29/06–21/07): **905 clientes**, ~39/dia, pico 64 (04–05/07).
- **Perfil do ticket-pai (julho):** `bsat_saida` **759 (92,7%)** · `conversa_inativa` 60 (7,3%). Junho: 113 bsat · 2 inativa · 5 transferência.
- **Salto diário:** 28/06 = 0 → **29/06 = 36 → 30/06 = 52 → 01/07 = 62** → sustentado 20–64/dia.
- **Destino:** a automação fecha sem atendimento (julho: 584 closed / 115 solved / 120 open).
- **Variante com auto-tags** (`intent__`/`sentiment__` — ex. #2828731, #2835356): **mesma anomalia**; o auto-triage nativo só "ilumina" os mesmos órfãos. 445/817 (54,5%) têm auto-tags em julho. **Origem datada (audit 22/07): EAP "Copiloto de Conhecimento" ativado em 07/07 20:54** (account_setting) → ramp das tags a partir de 09/07. Ativação interna, não release do fornecedor.
- **Evidência Data Export:** a conversa-pai existe no export (bot respondeu e encerrou); **nenhuma conversa nova é criada no motor** para o follow-up (23.020 conversas em 15–21/07, zero com `bot_messages_count=0`). O motor nunca fica sabendo da nova mensagem → falha no reengajamento/entrega, não nos fluxos.
- **Causa raiz:** não fechada do nosso lado — audit do Zendesk limpo (24/06–02/07); alavancas suspeitas: ciclo de vida da conversa pós-encerramento no AI agent (Ultimate) + config do widget ("Permitir múltiplas conversas" ativo). **→ ESCALAÇÃO AO FORNECEDOR** (pacote na pasta).
- **Pasta:** `Anomalia 01 - Follow-up pós-encerramento sem resposta/`
- ⏳ Teste pendente: existe follow-up pós-encerramento que REENGAJA (novo ticket com `advanced_ia` <15min após pai fechado)? Se zero desde 29/06 → falha total, não intermitente.

## Anomalia 02 — Conversa nova sem resposta (silêncio verdadeiro) 🟡 baseline antiga

**Mecânica:** cliente **sem nenhum ticket anterior** (ou com ticket-pai de meses atrás) abre conversa e não recebe **nenhuma** resposta — nem bot, nem formulário, nem humano. É o "órfão clássico": a conversa não é entregue ao motor do AI agent.
- **Números:** junho = **142** (inclui pico de 22 em **11/06** — mini-incidente já conhecido) · julho 01–21 = **61** (~2,9/dia). **Estável, sem salto em 29/06** → NÃO é a regressão; existe desde o rollout (~0,1–0,2% do canal).
- ⚠ **Contaminação em junho:** casos da Anomalia 03 (formulário, fechados e sem tags) entram nesta assinatura por tags — os 4 exemplos do Dono de junho estão aqui. Separação exige conteúdo dos comentários → **Genie O1/O2** reconcilia.
- **Causa raiz:** aberta (falha de entrega Widget/Sunshine → AI agent, intermitente). Escalação secundária na pasta (enviar junto ou após a da 01).
- **Pasta:** `Anomalia 02 - Conversa nova sem resposta/`

## Anomalia 03 — Formulário nativo indevido ("A form was sent") 🔴 ATIVA (~30/dia em julho)

**Mecânica:** em vez do AI agent, responde a **resposta PADRÃO nativa do canal Zendesk Messaging** (identificada 22/07: comentário de autor −1/sistema via `chat_transcript`; não há usuário-bot; config em Admin → Canais → Mensagens): "Olá! Fale mais sobre você…" + formulário. Se o cliente preenche → humano (com fricção); se não → morre sem atendimento.
- **Números (Genie O1+O1b, 22–23/07):** "Fale mais sobre você" **nasce com o rollout** (pré-rollout = 0 → **426** em 22–31/05, ~43/dia) · jun **1.574** (~52/dia; diário 30–108, **29/06=38 → sem quebra**, independente da An. 01) · jul 01–22 **655** (~30/dia, pico 51 em 01/07) → **ativa até hoje.** 99,98% assignee humano · ~100% closed. **Validação por frase:** jun/jul = 100% "Fale mais sobre você" (sem inflação por macro; "A form was sent" = 0 no lake — evento não ingerido). **Variante "24 horas" = fluxo padrão PRÉ-rollout, EXTINTA** (15.102 em 01–21/05 · 113 em 22–31/05 · 0 desde junho).
- **Dois subgrupos:** (a) *preencheu → **atendido por humano via chat*** — ✅ **confirmado pela API (23/07)** via locutores do transcript (15/15 tickets com falas de agente; a O2 do Genie que dizia "99,7% sem atendimento" foi **refutada** — método cego, ver regra nº 5); (b) *morreu sem preencher* (assignee "AI Agent", ≤1 tag) — **o lake NÃO vê este subgrupo** (transcript não ingerido; Genie deu 0 AI Agent, mas #2777310/#2775076 têm a frase via API) → só medível pela API: **182 parcial** (22/05→jun).
- **Config confirmada (Dono, print 23/07):** a resposta padrão do canal "Betnacional" (Admin → Canais → Mensagens) é exatamente o form desta anomalia; "Testar agora" abre o Menu do AI agent normalmente → quando o AI agent engaja há Menu, quando não engaja o canal serve o form. Config coerente; o defeito é a entrega da conversa ao AI agent.
- **Separação da An. 01 confirmada:** 8/8 exemplos verificados (jun+jul, incl. #2817850) são **conversa nova** (sem ticket-pai). Os três modos coexistem: conversa nova→form (03) · conversa nova→nada (02) · follow-up→nada (01).
- **Variantes:** "24 horas / clique no botão" (#2595080, #2594575, #2574512 — mai) = fluxo padrão pré-rollout, **extinta desde junho**; form de CPF "retomado" (#2817850, 09/07 — sem pai; origem do pré-preenchimento em aberto).
- **Pendências:** medir subgrupo (b) de julho via API. (Bloco O do Genie CONCLUÍDO; Admin conferido.)
- **Pasta:** `Anomalia 03 - Formulário nativo (A form was sent)/`

## Anomalia 04 — Transferência não consumada 🟢 SÓ JULHO (esperado = zero)

**Mecânica:** cliente completa a coleta da transferência (nome/CPF/e-mail), bot confirma a transferência e **não entrega à fila humana**; 15min → `conversa_inativa` + `resolvido_ia` (indevido, vira "retenção").
- **Escopo (decisão do Dono, 22/07): SOMENTE JULHO em diante.** O incidente de junho (18–22/jun) foi interno e está resolvido — **não consta nos relatórios** (CSV de junho preservado em `_arquivado_2026-07-22`; a decomposição da retenção de junho no relatório consolidado segue intacta — 81.391 + 6.069).
- **Números:** julho = **27** (mini-rajadas; maior: 10 em ~2h em 07→08/07; último caso 19/07). O esperado é zero — caso novo = nó de transferência ainda lança exceção. KPI de rotina a partir de julho.
- **Pasta:** `Anomalia 04 - Transferência não consumada/` (relatório HTML + CSV jul).

## ~~Anomalia 05~~ — REMOVIDA em 23/07 (e-mail está FORA DE ESCOPO)

**Correção do Dono (23/07):** tickets de atendimento por **e-mail** (ex.: #2852929, #2836357, #2837012 — brand Suporte Email 41733182485147) **não entram em nenhuma contagem nem em nenhum relatório** — o escopo do projeto é SOMENTE chat (regra perene, já documentada). O fato de o auto-triage marcar `intent__` em e-mail não é uma anomalia a reportar; é apenas a razão da **regra de medição nº 0** (abaixo). Relatório e lista removidos (a lista antiga 758, que misturava e-mail, segue no `_arquivado` como histórico do erro de medição). A observação do Dono sobre "chats com intent que morreram sem resposta" é a **variante com auto-tags das Anomalias 01/02** — já coberta lá. A origem do auto-triage (EAP "Copiloto de Conhecimento", 07/07 20:54, interna) fica registrada na Anomalia 01.

---

## Reclassificações (não são mais anomalias próprias)

- **"Anomalia D — `intent__` sem `advanced_ia`" (569, jul) → DISSOLVIDA.** Reconciliação com o censo: 329 estão nele (305 = Anomalia 01, 23 = Anomalia 02, 1 pai antigo); o resto é cauda em voo (idade<24h) ou fora do critério. As auto-tags são **atributo** (auto-triage ligado ~09/07), não fenômeno. Resíduo de 240 a conferir (baixa prioridade).
- **"Bot antigo `1.0_olá`" → FALSO LEAD** (macro de saudação; não existe bot legado).
- **"Órfãos = conversa nunca entregue" (escalação de 22/07) → DIAGNÓSTICO CORRIGIDO**: 93% eram Anomalia 01 (follow-up), não conversa nova. Pacote antigo arquivado; novo pacote na pasta da Anomalia 01.

## Regras de medição (valem para 01 e 02)

0. **ESCOPO = SOMENTE CHAT (`native_messaging`).** Ticket de e-mail (brands Suporte Email etc.) **NUNCA** aparece em contagem, lista ou relatório — nem como "anomalia". Toda consulta por tag (`intent__` etc.) filtra canal obrigatoriamente (Dono, 23/07 — reforço de regra perene).
1. **Idade ≥ 24h** para contar órfão (tags gravam na materialização; conversa em andamento parece órfã e se normaliza).
2. Universo: assignee "AI Agent" (50032929128731) + sem `advanced_ia` + só tags de ruído (`intent__`/`language__`/`sentiment__`/`logado`).
3. Separação 01×02 = **ticket-pai do mesmo requester com `advanced_ia` fechado ≤15min antes** (censo via API; o Genie não faz esse self-join com facilidade).
4. Monitoramento diário: `integracao/orfaos_contagem_limpa.py` (contagem) + `integracao/censo_pais_orfaos.py` (censo completo, ~15min).
5. **"Houve resposta humana?" em messaging NUNCA se responde por `author_id` de comentário** (lake ou API): a fala do agente fica DENTRO do `chat_transcript` (autor −1); `author_id>0` só captura notas internas. Único caminho: **locutores do transcript via API** (`integracao/analisar_locutores_transcript.py`). Caso-prova: O2 do Genie refutada em 23/07 (dizia 99,7% sem atendimento; 15/15 verificados tinham agente falando).
6. **Toda resposta do Genie se confirma por segundo caminho (API/amostra) antes de virar conclusão** — exigência do Dono (23/07), após a O2 vir estruturalmente errada.
