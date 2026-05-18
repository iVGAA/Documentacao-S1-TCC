---
file: glossario-tecnico.md
section: Glossário Técnico
purpose: Termos de domínio (água, regulação, telemetria) + termos técnicos do projeto EcoData
depends_on: [src/data/glossary.ts, AGENT-CONTEXT.md]
last_updated: 2026-05-18
status: draft
---

# Glossário Técnico

> **Para agentes de IA**: este arquivo consolida o vocabulário do TCC EcoData em duas partes. A **Parte A** reúne termos do domínio (qualidade da água, hidrologia, instituições, legislação, conceitos) e replica o conteúdo do glossário público (`src/data/glossary.ts`) em ordem alfabética. A **Parte B** define os termos técnicos do projeto (Next.js, Drizzle, Vercel, Neon, etc.) usados nas seções de arquitetura e banco de dados. Cada entrada segue o formato `**Termo** — definição` e pode incluir fonte oficial, fórmula ou contexto adicional.

---

## Parte A — Termos do domínio (água, regulação, telemetria)

Entradas alfabéticas, com fonte indicada quando aplicável. Reflete o glossário público apresentado em `/glossario`.

**Autodepuração** — capacidade natural do rio de se recuperar de uma perturbação (como um lançamento de esgoto) ao longo de seu curso. Ocorre por processos físicos (diluição, sedimentação), químicos (oxidação) e biológicos (ação de microrganismos). Depende diretamente da vazão e da temperatura da água.

Fonte: CETESB.

**Bacias PCJ** — conjunto de bacias hidrográficas dos rios Piracicaba, Capivari e Jundiaí, no leste de São Paulo. Abastece mais de 5 milhões de pessoas em 76 municípios e contribui para o Sistema Cantareira.

Fonte: Agência PCJ — https://agencia.baciaspcj.org.br/

**Balneabilidade** — avaliação se a água está segura para contato humano (natação, canoagem e similares), baseada principalmente na contagem de bactérias coliformes. A CETESB classifica praias e pontos de recreação como Próprios ou Impróprios. O EcoData não possui dados de coliformes em tempo real e, portanto, não avalia balneabilidade.

Fonte: CETESB · CONAMA 274/2000.

**Boletim** — publicação mensal integrada do EcoData com dados de qualidade (SIMQUA) + quantidade (SSPCJ) por trecho de rio. Ciclo de vida: `draft → approved → published → archived`. Gerado em PDF via `@react-pdf/renderer` e armazenado em Vercel Blob.

**CETESB** — Companhia Ambiental do Estado de São Paulo. Responsável pelo monitoramento da qualidade da água nos rios paulistas, incluindo as estações de qualidade das Bacias PCJ.

Fonte: CETESB — https://cetesb.sp.gov.br/

**Chuva no Ponto (Precipitação)** — quantidade de chuva acumulada no período, medida em milímetros (mm) por uma estação pluviométrica no ponto de monitoramento. 1 mm de chuva equivale a 1 litro de água por metro quadrado. Influencia diretamente a vazão e o nível dos rios.

Fonte: Agência PCJ · SSPCJ.

**Classe de Enquadramento** — classificação oficial (Resolução CONAMA 357/2005) que define a qualidade mínima da água, baseada no uso pretendido. Classes vão de Especial (melhor) a 4 (mais degradada). Classe 2 permite abastecimento com tratamento convencional e recreação. Classe 3 exige tratamento mais avançado.

Fonte: CONAMA 357/2005.

**Condutividade Elétrica** — mede a capacidade da água de conduzir eletricidade (µS/cm), indicando a quantidade de sais e minerais dissolvidos. Valores muito altos sugerem poluição por esgoto ou resíduos industriais.

Fonte: CETESB · SIMQUA.

**CONAMA 357/2005** — Resolução federal que classifica corpos d'água em classes (Especial a 4) e define os padrões mínimos de qualidade para cada uso. É a base legal para todo o monitoramento de qualidade de água no Brasil.

Fonte: CONAMA — Resolução nº 357, de 17 de março de 2005.

**Estação telemétrica** — ponto físico de coleta automática (sensor + datalogger + comunicação) que transmite leituras horárias para o SIMQUA (qualidade) ou para o SSPCJ (quantidade). O EcoData consome essas leituras via API e persiste em Neon Postgres a cada hora.

**IET (Índice de Estado Trófico)** — mede o nível de enriquecimento de nutrientes (fósforo e nitrogênio) na água, fenômeno chamado de eutrofização. Quanto maior o IET, maior a proliferação de algas e menor a qualidade da água para abastecimento e recreação.

Fonte: CETESB.

**IQA (Índice de Qualidade da Água)** — índice criado pela CETESB (adaptado da NSF) que resume a qualidade da água em uma nota de 0 a 100, calculado a partir de 9 parâmetros físico-químicos e bacteriológicos. Faixas: Ótima (79–100), Boa (51–79), Regular (36–51), Ruim (19–36) e Péssima (0–19). O EcoData exibe o IQA oficial publicado pelo SSPCJ.

Fórmula: IQA = ∏ qᵢ^wᵢ, onde qᵢ é a qualidade do parâmetro i e wᵢ é seu peso.

Fonte: CETESB · Agência PCJ.

**Nível da Água** — altura da coluna de água no rio em relação a uma referência fixa (régua ou sensor), medida em metros (m). Variações no nível indicam cheias, secas ou mudanças bruscas causadas por chuvas intensas. É monitorado em tempo real pelas estações telemétricas do SSPCJ.

Fonte: Agência PCJ · SSPCJ.

**OD (Oxigênio Dissolvido)** — quantidade de oxigênio presente na água (mg/L), essencial para peixes e organismos aquáticos respirarem. Quanto maior o OD, mais saudável o rio. Proxy direto para vida aquática.

Fonte: CETESB · SIMQUA.

**pH** — escala de 0 a 14 que mede a acidez ou alcalinidade da água. Abaixo de 7 é ácido, 7 é neutro, acima de 7 é alcalino. Rios saudáveis devem ter pH entre 6,0 e 9,0 (Resolução CONAMA 357/2005, classes 1 a 3).

Fonte: CONAMA 357/2005 · CETESB.

**Poluição Pontual vs. Difusa** — poluição pontual vem de uma fonte identificável (por exemplo, um cano de esgoto despejando no rio). Poluição difusa se espalha por uma grande área sem ponto único de origem (por exemplo, agrotóxicos e sedimentos carregados pela chuva de campos agrícolas). Ambas degradam a qualidade da água, mas exigem abordagens de controle diferentes.

Fonte: CETESB · CONAMA 357/2005.

**Precipitação (mm)** — ver "Chuva no Ponto".

**Q7,10** — vazão mínima de 7 dias consecutivos com período de retorno de 10 anos. Pior cenário provável de seca para o rio, usado como referência para outorga de uso da água. Se a vazão se aproxima desse valor, é sinal de emergência hídrica.

Fonte: Agência PCJ · ANA.

**Resolução (legislação)** — ato normativo expedido por órgão colegiado (ex.: CONAMA, ANA) que define padrões técnicos e legais para um tema. No contexto do EcoData, a principal é a Resolução CONAMA 357/2005, que estabelece classes de qualidade.

**SIMQUA** — Sistema de Monitoramento de Qualidade das Águas da CETESB. Fornece dados horários de qualidade (OD, pH, turbidez, condutividade e temperatura) para as estações de monitoramento nas Bacias PCJ. Principal fonte de dados de qualidade do EcoData.

Fonte: CETESB · SIMQUA.

**SPAguas (ex-DAEE)** — Departamento de Águas e Energia Elétrica do estado de São Paulo. Órgão estadual responsável pela gestão dos recursos hídricos, outorgas e fiscalização. Referenciado no TCC como autoridade institucional, ainda que a fonte primária de quantidade no EcoData seja a Agência PCJ/SSPCJ.

Fonte: SPAguas (governo do estado de SP) — antigo DAEE.

**SSPCJ (Sistema de Suporte à Decisão das Bacias PCJ)** — sistema da Agência PCJ que fornece dados telemétricos horários de quantidade (precipitação, nível e vazão) para mais de 50 estações nas Bacias PCJ. Fonte de dados hidrológicos do EcoData. Também referenciado como "SSD PCJ".

Fonte: Agência PCJ · SSPCJ.

**Temperatura da Água** — afeta diretamente quanto oxigênio a água consegue dissolver: água mais quente retém menos oxigênio. No verão, a capacidade de oxigenação diminui naturalmente.

Fonte: CETESB · SIMQUA.

**Turbidez (NTU)** — medida de quão turva ou barrenta está a água, em Unidades Nefelométricas de Turbidez. Causada por partículas de terra, areia e poluentes suspensos. Alta turbidez dificulta o tratamento da água e prejudica a vida aquática.

Fonte: CETESB · SIMQUA.

**Vazão (m³/s)** — volume de água que passa por um ponto do rio a cada segundo, medido em metros cúbicos por segundo. 1 m³/s equivale a 1.000 litros por segundo. Vazão alta aumenta a capacidade do rio de diluir poluentes.

Fonte: Agência PCJ · SSPCJ.

---

## Parte B — Termos técnicos do projeto

Subconjunto de termos da stack do EcoData usados nas seções de arquitetura (§6), banco de dados (§7) e diagramas de sequência (§5).

**App Router (Next.js)** — sistema de roteamento baseado em diretórios (`app/`) introduzido no Next.js 13 e estabilizado nas versões 14/15. Suporta React Server Components por padrão, layouts aninhados, route handlers, server actions, streaming via Suspense e segmentos paralelos/interceptados. O EcoData usa exclusivamente App Router; nenhuma rota legada em `pages/`.

Fonte: https://nextjs.org/docs/app

**Cron job** — tarefa agendada que executa em horários fixos. No EcoData, configurada em `vercel.json` (campo `crons`) e roteada para endpoints `POST /api/cron/*` protegidos por `Authorization: Bearer ${CRON_SECRET}`. Aciona ingest horário de SIMQUA e SSPCJ.

Fonte: https://vercel.com/docs/cron-jobs

**Drizzle ORM** — ORM TypeScript-first que gera SQL parametrizado em tempo de build, sem runtime de reflexão. Schema declarado em `src/db/schema.ts` (tabelas `stations`, `readings`, `bulletins`, `pipeline_runs`, `audit_logs`, …) e migrations geradas via `drizzle-kit`. O EcoData usa o driver `drizzle-orm/neon-http` para edge-compatibility.

Fonte: https://orm.drizzle.team/

**Edge Network (Vercel)** — rede global de PoPs (Points of Presence) da Vercel que serve assets estáticos, executa Edge Functions e Edge Middleware próximo ao usuário. O EcoData tira proveito disso para o roteamento e cache do mapa público (`/`) e da página de estações (`/admin/estacoes`).

Fonte: https://vercel.com/docs/edge-network

**ISR (Incremental Static Regeneration)** — estratégia de renderização do Next.js em que páginas estáticas são regeneradas sob demanda após expiração (`revalidate`) ou via `revalidatePath`/`revalidateTag`. No EcoData, rotas como `/rio/[id]` e `/relatorios` usam ISR com revalidação atrelada ao cron de ingest, evitando rebuilds completos.

Fonte: https://nextjs.org/docs/app/building-your-application/data-fetching/incremental-static-regeneration

**Mermaid** — linguagem declarativa em Markdown para descrever diagramas (fluxograma, sequência, ER, classes, gantt). Renderizada pelo GitHub, MkDocs e tooling de TCC. Usada nas seções §3 (casos de uso), §5 (sequência) e §7 (ER) sem dependência de ferramenta visual externa.

Fonte: https://mermaid.js.org/

**Neon Postgres** — banco Postgres serverless com separação compute/storage, branching instantâneo e driver HTTP para ambientes edge. Hosta o schema do EcoData (`drizzle-orm/neon-http`). Suporta connection pooling nativo e cold start na ordem de centenas de ms.

Fonte: https://neon.tech/

**RSC (React Server Component)** — modelo de componente do React que executa exclusivamente no servidor, sem JS enviado ao cliente, com acesso direto a banco de dados, sistema de arquivos e segredos. No EcoData, todas as páginas do App Router são RSC por padrão; ilhas de interatividade (mapa Leaflet, filtros) usam `"use client"`.

Fonte: https://react.dev/reference/rsc/server-components

**Tailwind v4** — versão major do Tailwind CSS com motor Oxide (Rust) e configuração via `@theme` em CSS, eliminando `tailwind.config.js`. Suporta tokens com `oklch()`, `@utility` custom e `@variant` declarativo. O EcoData adota Tailwind v4 com design tokens definidos em `app/globals.css`.

Fonte: https://tailwindcss.com/blog/tailwindcss-v4

**Vercel Blob** — storage de objetos da Vercel para arquivos binários (PDFs, imagens, anexos), acessível via SDK `@vercel/blob`. O EcoData persiste os PDFs de boletim gerados (`bulletins.pdf_url`) em Blob, com URLs imutáveis assinadas e CDN integrada.

Fonte: https://vercel.com/docs/storage/vercel-blob

---

## Referências cruzadas

- `src/data/glossary.ts` — fonte da verdade do glossário público exibido em `/glossario`
- `04-casos-de-uso-descricoes.md` — UC-05 (buscar termo no glossário)
- `06-arquitetura.md` — uso de Next.js App Router, Vercel Edge, Neon, Vercel Blob
- `07-banco-de-dados.md` — schema Drizzle (tabelas `stations`, `readings`, `bulletins`)
- `AGENT-CONTEXT.md` — tabela-resumo dos 10 termos essenciais do domínio

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: glossary
public_glossary_source: src/data/glossary.ts
total_terms_domain: 24
total_terms_technical: 10
```
<!-- /agent-extractable -->
