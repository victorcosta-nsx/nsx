# Relatório — Retenção, Fluxos e Satisfação do BOT Betnacional
## Junho/2026 · versão validada de 16/07/2026

**Universo oficial:** 146.778 tickets de suporte com a tag `advanced_ia`, `created_at` (UTC) entre 01/06 e 30/06/2026.
**Fontes:** Genie Zendesk (Data Lake, catálogo `silver_providers.nsx_zendesk_support`) + gatilho verificado no Zendesk Admin.
**Documentos-irmãos:** `LOGICA_ATENDIMENTO_BOT_BETNACIONAL.md` (lógica oficial) · `MASTER_PLAN_METRICAS_BOT.md` (plano e pendências).
**Status:** números estruturais VALIDADOS (fecham exato); decomposição fina dos retidos e anomalias em investigação (marcadas ⏳).

---

## 1. Governança do dado (por que estes números são confiáveis)

1. **Denominador oficial = 146.778** — critério `created_at` em UTC. Extrações por `_partition_date` dão 146.479 (1.486 tickets de junho só foram ingeridos no Data Lake em julho) — **partição nunca é critério de negócio**. As telas do Zendesk (Explorer usa data de finalização; painel do Advanced AI usa "started at") também divergem — **a fonte oficial é o Genie/API**.
2. **Métrica nativa de "transferidos" do Zendesk é inutilizável por desenho:** o bot nunca encerra sozinho — todo atendimento passa por um node de transferência (nos fluxos BSAT, Abandono e Transferência humana), então ~100% constam como "transferidos". Toda métrica é feita por **tags, rótulos e marca**.
3. **Premissa:** todo ticket é rastreável; toda contagem fecha no total; resíduos são listados, nunca ignorados.

## 2. As marcas — resolvido empiricamente

O gatilho **"Resolvidos IA"** (ativo desde 21/05/2026, verificado no Admin) define a marca na criação do ticket:

> QUANDO o ticket é criado E as tags contêm `resolvido_ia` → Marca = **BOT Betnacional** + Status = **Fechado**.

- **Marca BOT Betnacional** (50678377473307, criada 21/05/2026 com o Ultimate AI) = tickets que o bot encerrou (retidos).
- **Marca Bet Nacional** (28331414811547, marca-mãe) = tickets que chegaram à operação humana (transferidos).
- A marca não é escolhida por ninguém: é consequência automática da tag terminal. Essa é a explicação institucional para "o que são as marcas".

## 3. O número central — retenção de junho/2026

### Matriz marca × tags (fecha exato no denominador)

| Marca | `resolvido_ia` puro | `transferencia_saida` puro | AMBAS ⏳ | NENHUMA | Total |
|---|---|---|---|---|---|
| BOT Betnacional | 81.391 | 0 | 6.069 | 0 | **87.460** |
| Bet Nacional | 0 | 59.308 | 0 | 4 ⏳ | **59.312** |
| Callcenter + Teste (residual conhecido) | | 6 | | | **6** |
| **TOTAL** | | | | | **146.778 ✓** |

### Leitura

| Métrica | Valor | % |
|---|---|---|
| **RETIDOS pelo BOT** (marca BOT Betnacional) | **87.460** | **59,59%** |
| — resolvido_ia puro | 81.391 | 55,45% do total |
| — ⏳ anomalia "transferência não consumada" | 6.069 | 4,13% do total |
| **TRANSFERIDOS para humano** (marca Bet Nacional) | **59.312** | **40,41%** |
| Residual conhecido (registros callcenter + teste) | 6 | ~0% |

### ⏳ A anomalia dos 6.069 (`resolvido_ia` + `transferencia_saida` juntas)

6,9% da marca BOT tem as duas tags-mestre simultaneamente — combinação que não deveria existir. Padrão típico: `transferencia_entrada` + `transferencia_saida` + `conversa_inativa` + `resolvido_ia`. **Hipótese em investigação:** o cliente completou a coleta de dados da transferência, mas abandonou antes de interagir com o agente; o fluxo de abandono rodou por cima e o gatilho fechou o ticket na marca BOT. Se confirmado, são **clientes que pediram humano e não chegaram lá** — contam como retenção no custo, mas são falha de experiência. Até fechar (rodada 3, bloco I), reportamos os retidos decompostos em 81.391 + 6.069.

### Decomposição preliminar dos retidos (⏳ a refinar com a rodada de decomposição)

Dos 87.460 com `resolvido_ia` (extrações sobre a mesma base created_at):
- **11.242 (12,9%) encerraram SEM abandono** — jornada completa com feedback (11.239 têm `bsat_entrada`).
- **76.218 (87,1%) abandonaram em algum ponto** (`conversa_inativa`) — destes, 7.498 já tinham chegado ao BSAT (abandonaram só a pesquisa, atendimento completo). O restante se divide entre: abandonou com resposta entregue, abandonou no meio de fluxo, abandonou no menu — separação exata pendente (blocos C da rodada 1).

## 4. Mapa do canal chat — isolamento do filtro

- **Contaminação: ZERO.** Todos os 260.162 tickets `advanced_ia` desde 21/05 são 100% canal Native Messaging. O filtro `advanced_ia` não traz nada de e-mail/telefone/API.
- **Chats fora do filtro:** 3.253 tickets Native Messaging de junho NÃO têm `advanced_ia` (~2,2% do canal). Não são perdidos do Ultimate — são três mundos paralelos:
  1. 🔴 **Bot básico ANTIGO** (tags `1.0_olá`, `1.15_informar_cpf`...): **1.531 tickets** — ver investigação abaixo;
  2. Registros Teleperformance/callcenter (823);
  3. E-mail/omnichannel (~525-560).
- ⚠ O mapa por brand será re-publicado com o critério oficial created_at (a extração atual usou partição — números não fecham no denominador). As 3 brands antes não identificadas foram mapeadas em 21/07/2026 via API Zendesk: **41733182485147 = Suporte Email** (e-mail/omnichannel), **28331419129627 = Ouvidor Digital**, **26958014471195 = NSX Brasil** (Help Center principal).
- **Exceção mapeada:** o fluxo `informe_rendimentos_2026` entra direto, sem `welcome_entrada` (209 tickets) — exceção legítima à regra do Welcome.

### 🔴 Investigação prioritária — clientes caindo no BOT ANTIGO

A operação estima **500 a 1.500 tickets/mês "perdidos"** em clientes que caem no bot básico antigo (sem a apresentação/menu do Advanced AI). Os dados confirmam a escala: **1.531 tickets com `1.0_olá` em junho**. O mistério: todos os web widgets do site e do app usam o ID do Advanced AI — o bot antigo não deveria receber ninguém, e não se sabe a raiz. Hipóteses em teste (rodada 3, bloco M): app em versão antiga (SDK com ID antigo), conversas persistentes reabertas na thread antiga, fallback quando o Advanced AI está indisponível, entrypoint esquecido.

## 5. Fluxos — conclusão por tags de entrada/saída (junho, sem Transferencia)

### ⚠ Correções de instrumentação feitas em julho/2026 (registrar no histórico)

Cinco fluxos tinham tags de saída ausentes/erradas, corrigidas em jul/26 — **os números de junho SUBESTIMAM a conclusão real deles** (correção não retroativa; re-medir com julho):
1. Passo_a_Passo_Cadastro (saídas sem tag) · 2. Menu_apos_fluxo (saídas gravavam Welcome_saida) · 3. Bolsa_familia (tags erradas) · 4. Informacao_deposito (saídas sem tag) · 5. Promocoes (saídas sem tag). — Recuperacao_conta foi auditado e estava correto.

### Vencedores (conclusão ≥ 80%)
| Fluxo | Entradas | Conclusão |
|---|---|---|
| Bonus_nao_recebido | 1.586 | 100% |
| Duvida_mercado_cassino | 8.640 | 93,84% |
| Ainda_n_recebi_aposta | 12.284 | 88,55% |
| Solicitar_bonus_retencao | 505 | 86,93% |
| Informacao_saque | 11.699 | 84,60% |
| Validacao_identidade | 6.999 | 82,97% |
| Duvida_apostas | 7.348 | 82,50% |
| Encerrar_aposta | 3.284 | 81,79% |

### Críticos (⚠ = corrigido em jul; re-medir antes de decidir)
| Fluxo | Entradas | Conclusão |
|---|---|---|
| Menu_apos_fluxo ⚠ | 1.530 | 13,27% |
| Passo_a_Passo_Cadastro ⚠ | 8.131 | 17,07% |
| Bolsa_familia ⚠ | 4.905 | 24,77% |
| Informacao_deposito ⚠ | 12.034 | 36,63% |
| Informe_rendimentos_2026 | 398 | 45,73% |
| Recuperacao_conta | 6.948 | 59,04% |
| Promocoes ⚠ | 21.472 | 59,92% |
| Cadastro | 28.814 | 63,49% |

Investigações qualitativas: Passo_a_Passo_Cadastro — 89,6% dos sem-saída são inatividade real (UX de cliques "2º PASSO"); Recuperacao_conta — 97,5% inatividade (menu com 2 opções que não cobrem casos reais).

**Nota de leitura:** entrada-sem-saída ≠ abandono necessariamente. Fluxos com o node de texto livre ABERTO permitem saída lateral (cliente pede outro assunto e migra de fluxo sem ganhar a `_saida`). A separação saída-lateral × morte-no-fluxo está na fila de dados (bloco E da rodada 1).

## 6. Satisfação (BSAT)

**Regra oficial de atribuição:** a nota BSAT pertence ao fluxo cuja tag de SAÍDA foi a última aplicada antes de `bsat_entrada`. Nota gravada em bsatScore (campo 50734727828507; 1–5, −1 = "Agora não"). `bsat_entrada` sem nota = abandono da pesquisa (conta na taxa de resposta, não na média). Ler médias só com n ≥ 30.

**Funil (30.028 com `bsat_entrada`):** 2.571 (8,56%) sem a pergunta (falha de disparo, investigar) · 11.442 (38,1%) deram nota · 4.861 (16,2%) "Agora não" · 11.154 (37,1%) abandonaram a pesquisa.

**Distribuição das 11.442 notas:** 😡 6.419 (56,1%) · 🙁 623 · 😐 787 · 🙂 1.301 · 😍 2.312 (20,2%). **Média 2,34 · 31,6% positivas · 61,5% negativas.**

**Por fluxo de origem:** melhores — Cadastro 3,32 (1.319 aval.) e Bolsa_familia 3,27 (290); piores — Duvida_mercado_cassino 1,45 (438) e Encerrar_aposta 1,72 (360); maior volume — Ainda_n_recebi_aposta 2,09 (3.640).

**O insight que define a meta:** retenção ≠ satisfação. Duvida_mercado_cassino conclui 93,84% com nota 1,45; Ainda_n_recebi_aposta conclui 88,55% com nota 2,09; Cadastro conclui 63,49% com nota 3,32. Os fluxos que mais retêm são os que mais frustram — **KPI de gestão em par: Retenção × BSAT (baseline junho: 59,59% × 2,34).**

## 7. Pendências e próximos passos

| # | Frente | Status |
|---|---|---|
| 1 | Anomalia 6.069 (transferência não consumada?) | Rodada 3 — bloco I |
| 2 | 🔴 Bot antigo / tickets perdidos (500–1500/mês) | Rodada 3 — bloco M |
| 3 | ~~Brands desconhecidas~~ (✅ identificadas 21/07 via API) + mapa do canal com created_at | J2 — mapa por marca com created_at |
| 4 | Fluxos novos de julho + BSAT deles | Rodada 3 — K |
| 5 | RA (token de cobrança, 72h): onde vive nos dados; consumo vs cota anual | Rodada 3 — L + painel admin |
| 6 | Decomposição fina dos retidos, welcome/inferência, saída lateral, motivo do contato dos transferidos | Blocos B–G (rodada 1) |
| 7 | Dashboard HTML definitivo | Refazer quando #1 e #6 fecharem |

---
*Números estruturais validados por dois caminhos independentes (marca via gatilho × tags) fechando exatos em 146.778. Substitui todas as versões anteriores.*
