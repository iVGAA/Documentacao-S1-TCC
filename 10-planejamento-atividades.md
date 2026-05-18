---
file: 10-planejamento-atividades.md
section: 10. Planejamento das Atividades
purpose: Cronograma retrospectivo, derivado do histórico de commits
depends_on: []
last_updated: 2026-05-18
status: draft
---

# 10. Planejamento das Atividades

> **Para agentes de IA**: este planejamento é **retrospectivo, baseado em git log real** — não é projeção. Use para entender quando cada parte do sistema foi construída.

## Visão geral

O projeto EcoData foi executado de forma individual por Pedro Henrique Rocha (com assistência de IA em pair-programming para geração de planos, código e revisão), em três milestones principais ao longo de aproximadamente três meses calendário (28/fev/2026 a 18/mai/2026). A execução não foi contínua: houve concentração de trabalho em poucos dias intensivos, intercalados com janelas de espera para validação acadêmica e calibração de fontes externas (CETESB SIMQUA e SSD PCJ).

O número total de commits no período é **279** (incluindo o commit inicial do scaffold gerado pelo `create-next-app`). Para fins de cronograma acadêmico, agrupamos os commits em sprints lógicos alinhados às fases do `ROADMAP.md`.

| Milestone | Período                  | Fases    | Entrega principal                                              |
|-----------|--------------------------|----------|----------------------------------------------------------------|
| v1.0      | 2026-02-28 a 2026-03-11  | 1-8      | Plataforma de monitoramento em tempo real (mapa, /rio, /glossario, ingest SIMQUA + SSD PCJ) |
| v2.0      | 2026-04-13 a 2026-04-29  | 9-16     | Gerador de boletim em PDF (@react-pdf/renderer, charts server-side, /relatorios) |
| v3.0      | 2026-05-06 a 2026-05-13  | 17-23    | Pipeline de dados definitivo + admin observability + backfill 2016-2025 |
| Polish    | 2026-05-15 a 2026-05-18  | —        | UI polish, A11y, sparklines, mini-viz, hardening de auth, dados experimentais |

## Sprints detalhados

Cada sprint abaixo corresponde a uma fase do `ROADMAP.md` (quando o mapeamento é direto) ou a uma onda de trabalho coerente (quando múltiplas fases foram executadas em paralelo em uma única sessão). A coluna **commit referência** indica o primeiro commit relevante do sprint.

### Milestone v1.0 — Plataforma de monitoramento em tempo real

| Sprint | Data início | Data fim   | Fase | Atividades principais                                                                       | Commit referência |
|--------|-------------|------------|------|---------------------------------------------------------------------------------------------|-------------------|
| 1      | 2026-02-28  | 2026-02-28 | 1    | Scaffold Next.js 15 + camada de dados (tipos, rios, glossário) + shell de layout            | 5adfc90           |
| 2      | 2026-02-28  | 2026-02-28 | 2    | Leaf components (StatusIndicator, ComparisonChart) + HeroSection + BasinsSection com mapa SVG + RiverCard + BulletinSection com Tabs | 4ad8f82           |
| 3      | 2026-02-28  | 2026-02-28 | 3    | GlossarySection (4 cards) + CompareSection (lado a lado) + integração no `page.tsx` + a11y WCAG AA + lazy-load Recharts | 5c98371           |
| 4      | 2026-03-01  | 2026-03-02 | 4    | Restyle visual completo (Plus Jakarta Sans, scroll animations, semáforo tab colors, restyle de HeroSection, BasinsSection, Footer com SVG wave, StatusIndicator, RiverCard, GlossarySection, CompareSection) + expansão de dados multi-mês + `useMonth` context | 9c013e9           |
| 5      | 2026-03-03  | 2026-03-03 | 8    | Bootstrap de infra: Drizzle + Neon + Vercel Blob + Route Handler `/api/health` + deep health check + roundtrip Vitest | b482179           |
| 6      | 2026-03-03  | 2026-03-03 | 9*   | Redesign water-themed: GSAP ScrollTrigger, SemaphoreSection, ComparisonSection, RiverCard em 3 camadas, mapa SVG interativo com 10 pontos | 355e481           |
| 7      | 2026-03-03  | 2026-03-03 | 10*  | Chat grounding com Gemini + rota `/api/chat` com streaming e rate limit + componentes (bubble, panel, message, suggestions) + ChatWidget integrado via `next/dynamic` | 937f52a           |
| 8      | 2026-03-04  | 2026-03-04 | 11*  | Pipeline de scraping do boletim oficial (PDF) + extração Gemini + `runPipeline` + rotas `/api/pipeline/run` e `/api/pipeline/review` + seed das 10 estações PCJ | fb54cce           |
| 9      | 2026-03-04  | 2026-03-04 | 12*  | `MonthId` dinâmico + db-loader com fallback + Server Component `page.tsx` + MonthProvider dinâmico + cron `check-new-bulletin` + Upstash rate limiter docs | 3468079           |
| 10     | 2026-03-04  | 2026-03-04 | 13*  | Autenticação Bearer nas rotas `/api/pipeline/*` + headers CSP + correções de glossário, chart `-1` warning, toggle UX | 51316a7           |
| 11     | 2026-03-09  | 2026-03-09 | 1.2  | Motor de classificação CONAMA 357 (TDD: testes falhando primeiro, depois implementação) + schema v3 com validação | b61ddbf           |
| 12     | 2026-03-09  | 2026-03-09 | 2.1  | Pipeline foundation: schema migration, cron-auth, parameter seeding + data freshness + StaleBanner | d4348fb           |
| 13     | 2026-03-09  | 2026-03-09 | 2.2  | Pipeline de ingestão SIMQUA (TDD) + rota cron SIMQUA com testes                             | 327f69a           |
| 14     | 2026-03-09  | 2026-03-09 | 2.3  | Pipeline de ingestão SSD PCJ com rota cron + retenção de dados (agregação diária) + limpeza de alertas | 79104dd           |
| 15     | 2026-03-09  | 2026-03-09 | 3.1  | Cores CONAMA + MapStation + query + GeoJSON + página `/mapa` com server fetch, dynamic import e skeleton | 57e8ccd           |
| 16     | 2026-03-09  | 2026-03-09 | 3.2  | Marcadores de estação com cores CONAMA + popup card estilizado + `fitBounds` + `MapTopBar` | 7c9c449           |
| 17     | 2026-03-09  | 2026-03-09 | 3.3  | Controle de camadas: `RiverLayer`, `SubBasinLayer`, `LayerControlPanel` (TDD) + legenda CONAMA | 145ffd3           |
| 18     | 2026-03-10  | 2026-03-10 | 4.1  | Camada de dados para dashboard + classificação IQA + Header com rotas reais + Skeleton      | 3b5d405           |
| 19     | 2026-03-10  | 2026-03-10 | 4.2  | Home page v3 com hero, semaphore grid, boletim cards e ISR + mapa preview com Leaflet       | a4338a5           |
| 20     | 2026-03-10  | 2026-03-10 | 4.3  | Página station detail (`/rio/[id]`) com `StatusBanner`, `ParameterCards`, `ComplianceBar`, `IqaBadge`, `SourceLinks` + gráficos temporais + `PeriodSelector` + `TechnicalExpand` + export CSV | 1d0b992           |
| 21     | 2026-03-10  | 2026-03-10 | 5    | Rewrite chat grounding v3 + simple-markdown + API route com `getDashboardData` + componentes chat sem `useMonth` + `ChatWidgetLoader` no layout | 48e50c5           |
| 22     | 2026-03-10  | 2026-03-10 | 5+   | Correções diversas: acentuação pt-BR, z-index do Leaflet, seed histórico, expansão SIMQUA para 5 estações, substituição de GeoJSON fabricado por rios reais e polígonos ANA BHO 2017, tile Voyager/Satélite | b7e0999           |
| 23     | 2026-03-10  | 2026-03-10 | 6    | Página `/glossario` com 19 termos categorizados + página `/sobre` + Glossário no Header + responsividade mobile + otimização Lighthouse (code splitting, bundle) | 5bd8ef7           |
| 24     | 2026-03-10  | 2026-03-10 | 8    | Design system formalizado: design tokens + framer-motion + Header transparent-to-solid + Footer dark com grid 5 colunas + Hero rewrite (SplitText, particles, waves, counters) + `MapPreview` cinematográfico + `BoletimCards` accordion + redesign `/glossario`, `/sobre` e `/rio/[id]` com tema water | c67d021           |
| 25     | 2026-03-11  | 2026-03-11 | 8+   | Polish v1.0 final: acentuação, badges, mobile, header transparente no `/rio`, gradiente até o topo, glossário cards expandem independentemente, README completo do projeto | 56b2e6d           |

\* Em v1.0 a numeração interna do ROADMAP foi consolidada posteriormente em fases 1-8; os sprints 6-10 correspondem a iterações de design e features que hoje estão agregadas nas fases 4 e 8 do roadmap.

**Total v1.0**: 25 sprints, 111 commits, 12 dias úteis (28/02 a 11/03).

### Milestone v2.0 — Gerador de boletim em PDF

| Sprint | Data início | Data fim   | Fase | Atividades principais                                                                                       | Commit referência |
|--------|-------------|------------|------|-------------------------------------------------------------------------------------------------------------|-------------------|
| 26     | 2026-04-13  | 2026-04-13 | 9    | Station Mapping: schema com `bulletinCode`, `waterClass` corrigida, `station-pairs.ts` com configuração dos 5 pares | fceb118           |
| 27     | 2026-04-13  | 2026-04-13 | 10   | Statistics Engine: tipos, helpers de lookup, fetcher de média histórica SSD PCJ, cálculos mensais (precipitação, vazão, conformidade OD), séries temporais diárias, fachada orquestradora | eb8fd46           |
| 28     | 2026-04-13  | 2026-04-13 | 11   | Chart Generation: módulo de renderização (chartjs-node-canvas) com gráfico precipitação+vazão + script de validação dos 6 gráficos | 29c2111           |
| 29     | 2026-04-13  | 2026-04-13 | 12   | Assets: logos institucionais SVG placeholder, mapas de sub-bacia SVG, fotos aéreas SVG, módulo de configuração de assets tipado | 08adec6           |
| 30     | 2026-04-13  | 2026-04-13 | 13   | PDF Template: instalação `@react-pdf/renderer` + componentes para boletim PCJ (capa, introdução, páginas de estação) | 819cc68           |
| 31     | 2026-04-13  | 2026-04-13 | 14   | Generation Pipeline: orquestração de stats, charts, PDF, blob upload e registro em `reports`                | 2a8d117           |
| 32     | 2026-04-13  | 2026-04-13 | 15   | Reports Page: `/relatorios` com listagem de boletins e download, ISR com on-demand revalidation             | 5815c37           |
| 33     | 2026-04-13  | 2026-04-13 | 16   | Automation: API default para mês anterior + documentação cron mensal + testes unitários                     | 53b9f3e           |
| 34     | 2026-04-14  | 2026-04-14 | 14.1 | Hardening do pipeline em produção: lazy-init `chartjs-node-canvas`, externalização de pacotes nativos, troca de `chartjs-node-canvas` por `canvas` + `chart.js` direto, conversão de assets SVG → PNG, fallback local para PDF em dev, layout vertical dos gráficos + labels de referência | 28ed3c0           |
| 35     | 2026-04-14  | 2026-04-14 | 12.1 | Assets reais: mapas e fotos das sub-bacias gerados com rios e estações reais + extração de logos institucionais reais do boletim PDF de referência | 92533b8           |
| 36     | 2026-04-14  | 2026-04-14 | 10.1 | Correção dos cálculos de médias históricas com fallback hardcoded + preenchimento de `simquaIds` faltantes + aviso quando dados de qualidade indisponíveis + dimensões e fontes dos gráficos | c3af4c1           |
| 37     | 2026-04-22  | 2026-04-22 | 14.2 | Geração de boletim em produção: permitir UI generation no Vercel para SSDPCJ + externalização `@react-pdf/renderer` (fix `.notdef`) + permitir sobrescrever + registro Roboto real + revalidate cache | f06b52c           |
| 38     | 2026-04-29  | 2026-04-29 | 8.2  | Auth mock + middleware + dashboard admin com métricas reais do Postgres + preview/edição do PDF antes de publicar + rename DAEE → SPAguas + selo Projeto Experimental | 4d8d4e8           |

**Total v2.0**: 13 sprints, 41 commits, distribuídos em 4 dias intensivos (13/04, 14/04, 22/04, 29/04).

### Milestone v3.0 — Pipeline de dados definitivo

| Sprint | Data início | Data fim   | Fase | Atividades principais                                                                                        | Commit referência |
|--------|-------------|------------|------|--------------------------------------------------------------------------------------------------------------|-------------------|
| 39     | 2026-05-06  | 2026-05-07 | 8.3  | Design system formalizado + review autônomo (pt-BR, tokens semânticos, anti-patterns P0) + pipeline completa Boletim Builder (INTAKE + VALUE + COUNCIL + DECISIONS) | 9b08705           |
| 40     | 2026-05-07  | 2026-05-08 | 16.1 | Boletim-builder S1-S7: design spec, protótipos, dual renderer skeleton (PDF canonical + HTML preview), template seed SSDPCJ Oficial v1, admin dashboard redesign, editor canvas + drag + resize + page management + inspector + block palette + undo/redo + auto-save | b992fbb           |
| 41     | 2026-05-08  | 2026-05-08 | 16.2 | Impeccable audit + correções P1-P3: atividade (agrupar alertas + filtro + timeline), editor (zoom + delete + hover outline), estações (3-col xl + freshness footer), configurações (chips DECISIONS), templates (thumbnail abstrato), fork-on-write em versões locked, tests S3, smoke print A4/A3 | caf09cd           |
| 42     | 2026-05-12  | 2026-05-13 | 16.3 | Template editor v2: design spec from brainstorm + plano de 12 fases + scaffold `/edit-v2` sob feature flag + zod schema com sections + slot keys + migration v1→v2 + section type registry (Capa, Texto Livre, Custom) | 2320bc0           |
| 43     | 2026-05-13  | 2026-05-13 | 16.4 | Template editor v2 features: primitives lib (`HistoryStack`, `computeDragResult`), shell com 3 mode tabs + resizable panels, outline com dnd-kit sortable, section canvas, data palette + slash via cmdk, binding highlight + tokens toggle, sample-context resolver, 4 section types completos (introdução, resumo-de-estação, resumo-de-parâmetro, comparativo-conama), section-add dialog, unlock section, drag/resize blocks, assets API + management, drag-drop image upload, POST `/api/templates`, auto-save, publish version, gate <1024px, Playwright smoke | b48821f           |
| 44     | 2026-05-13  | 2026-05-13 | 16.5 | Admin onboarding: welcome modal + checklist + tracker + canonical filename `YYYY_boletim_integrado_MES.pdf` + remoção métrica CONAMA alerts + ghost code de template feature | 6c2f51a           |
| 45     | 2026-05-13  | 2026-05-13 | 17   | Phase 17: Schema Foundation — `monthly_stats` + `ingest_logs`                                                | 7211ac5           |
| 46     | 2026-05-13  | 2026-05-13 | 18   | Phase 18: Cron Resilience — retry + per-station logs + persistência em `ingest_logs`                         | 858466c           |
| 47     | 2026-05-13  | 2026-05-13 | 21   | Phase 21: Station Expansion — seed de 35 estações de quantidade extras (5 → 40 total)                        | f00fa0a           |
| 48     | 2026-05-13  | 2026-05-13 | 19   | Phase 19: Monthly Stats Compute — `compute-monthly-stats.ts` + OD compliance via média horária + endpoint admin | 69e63b7           |
| 49     | 2026-05-13  | 2026-05-13 | 22   | Phase 22: Admin Pipeline UI — `/admin/pipeline` observability dashboard                                      | 088f704           |
| 50     | 2026-05-13  | 2026-05-13 | 20   | Phase 20: Historical Seed — backfill 2016-2025 (~600 rows) + correção SSD PCJ usando query params lowercase (UNBLOCKS bulletin numbers) | fe3b44f           |
| 51     | 2026-05-13  | 2026-05-13 | 23   | Phase 23: E2E + Validation — `VALIDATION.md` para o seminário + review-driven fixes (race condition, timeouts, status) + Impeccable critique em `/admin/pipeline` | 30cc92c           |

**Total v3.0**: 13 sprints, 36 commits, distribuídos em 8 dias (06/05 a 13/05). As fases 17-23 foram executadas em um único dia (13/05), em ondas paralelas conforme planejado no `ROADMAP.md` (Wave 1: Phase 17 → Waves 2/3: 18, 19, 21, 20, 22 → Wave 4: 23).

### Polish — UI, A11y, hardening final

| Sprint | Data início | Data fim   | Fase | Atividades principais                                                                                       | Commit referência |
|--------|-------------|------------|------|-------------------------------------------------------------------------------------------------------------|-------------------|
| 52     | 2026-05-15  | 2026-05-15 | —    | Operator-grade observability rebuild do pipeline + topbar do mapa redesenhada + popup overhaul + honest stale signaling + chart captions em pt-BR plain language + loading skeletons + Suspense streaming + back-to-site + unified relative-time + chat FAB desligado | e0ee9c2           |
| 53     | 2026-05-15  | 2026-05-15 | —    | Foundations shadcn: sonner + AlertDialog + LifecycleBadge + tokens-only canvas + shadcn modals no editor legacy/v2 | 92e6e4e           |
| 54     | 2026-05-15  | 2026-05-15 | —    | Impeccable P0 sweep no admin (bullet, sparkles, 9px, top-border, OKLCH) + glossário com busca, anchors, copy-link, fonte oficial + content polish | 977a974           |
| 55     | 2026-05-16  | 2026-05-16 | —    | A11y: touch targets ≥44px + mobile-first tables (stack-to-card) no admin + 3 direções visuais para `/admin/estacoes` via Huashu (Tufte / Field.io / Hara) | 9390add           |
| 56     | 2026-05-17  | 2026-05-17 | —    | Brilho admin + glossário: sparklines, heatmap, mini-viz, period serif                                       | ef9bc5c           |
| 57     | 2026-05-18  | 2026-05-18 | —    | Hardening final: untrack de planning/tests/design/docs do repo público + warning experimental expandido no boletim + remoção de logos institucionais + footer disclaimer por página + clarify no-generate policy + auth gates em admin/cron + fix de auth bypass em generate-bulletin e preview-pdf | 5618149           |

**Total Polish**: 6 sprints, 12 commits, 4 dias (15/05 a 18/05).

## Métricas

- **Total de commits**: 279 (incluindo o `Initial commit from Create Next App`)
- **Total de fases entregues**: 23 fases formais do `ROADMAP.md` (1-8, 9-16, 17-23) + 6 sprints de polish
- **Total de sprints documentados**: 57
- **Duração calendário**: 80 dias (28/fev/2026 a 18/mai/2026)
- **Dias com commits**: 22 dias úteis (concentração alta — execução em ondas, não contínua)
- **Milestones**: 3 entregues (v1.0, v2.0, v3.0) + polish
- **Modelo de execução**: solo dev com pair-programming de IA via `gsd-*` (Get Stuff Done) commands (`gsd-spec-phase`, `gsd-plan-phase`, `gsd-execute-phase`, `gsd-code-review`)
- **TDD aplicado**: fases 1.2 (CONAMA), 2.2 (SIMQUA), 2.3 (SSD PCJ), 3.3 (layer controls), 16.1 (default month) — ~22% das fases iniciais com testes escritos antes do código
- **Linguagens**: TypeScript (100% do código de aplicação), SQL (migrations Drizzle), Markdown (documentação técnica e planning)

## Pontos de tensão e learnings

Esta seção documenta riscos materializados durante a execução, extraídos de commits com prefixos `fix:`, `chore:` e `refactor:` que sinalizam re-trabalho:

1. **Renderização de PDF em produção** (14/04 e 22/04): a transição de `chartjs-node-canvas` para `canvas` + `chart.js` direto, a externalização de `@react-pdf/renderer` (`.notdef` glyphs) e o registro de fontes Roboto reais consumiram um dia inteiro de hardening (~10 commits). Causa raiz: divergência entre runtime local (Linux nativo) e Vercel (Lambda) em pacotes nativos.
2. **Acentuação pt-BR e z-index Leaflet** (10/03): commit consolidado `b7e0999` agregou múltiplas correções de UX que escaparam dos testes unitários — sinal de falta de cobertura visual automatizada em v1.0.
3. **Query params SSD PCJ** (13/05): commit `fe3b44f` marcou como **UNBLOCKS bulletin numbers** — bug crítico em que parâmetros precisavam ser `lowercase`. Bloqueio descoberto somente na validação E2E da Phase 23. Mitigação: adicionar smoke test de contrato de API externa no roadmap pós-v3.0.
4. **Hobby plan Vercel** (10/03): commit `d090849` removeu crons do `vercel.json` ao descobrir que o plano Hobby não suporta schedule horário. Replanejamento: cron passou a ser disparado por GitHub Actions ou Upstash QStash (decisão registrada em `DECISIONS.md`).
5. **Auth bypass** (18/05): commits finais `6b25592` e `b98c5df` fecharam vetores de bypass em rotas admin e cron — encontrados durante revisão final antes de tornar repo público. Mitigação adotada: middleware unificado + audit explícito de cada rota `app/api/**`.

## Distribuição temporal

```
2026-02 |████████ (v1.0 scaffold + leaf components + restyle)
2026-03 |████████████████████████ (v1.0 telas + pipeline + dashboard + design system)
2026-04 |█████ (v2.0 boletim PDF + hardening + admin)
2026-05 |█████████ (v3.0 pipeline + polish + hardening final)
```

O perfil reflete o modelo de execução de TCC com cliente real (PCJ): janelas longas de validação externa (aprovação de copywriting, calibração com SSD PCJ) intercaladas com bursts de desenvolvimento solo. A produção concentrada em poucos dias é coerente com fluxo pair-programming com IA — onde o gargalo é decisão de produto, não digitação.

## Para agentes de IA

```yaml
milestones:
  - name: v1.0
    start: 2026-02-28
    end: 2026-03-11
    phases: [1, 2, 3, 4, 5, 6, 7, 8]
    sprints: 25
    commits: 111
    deliverable: "Plataforma de monitoramento em tempo real (mapa, /rio, /glossario, ingest)"
  - name: v2.0
    start: 2026-04-13
    end: 2026-04-29
    phases: [9, 10, 11, 12, 13, 14, 15, 16]
    sprints: 13
    commits: 41
    deliverable: "Gerador de boletim em PDF (@react-pdf/renderer + chartjs-node-canvas + /relatorios)"
  - name: v3.0
    start: 2026-05-06
    end: 2026-05-13
    phases: [17, 18, 19, 20, 21, 22, 23]
    sprints: 13
    commits: 36
    deliverable: "Pipeline de dados definitivo + admin observability + backfill 2016-2025"
  - name: polish
    start: 2026-05-15
    end: 2026-05-18
    phases: []
    sprints: 6
    commits: 12
    deliverable: "UI polish, A11y, sparklines, mini-viz, hardening de auth"

totals:
  commits: 279
  sprints: 57
  formal_phases: 23
  duration_days: 80
  active_dev_days: 22
  developers: 1
  developer_name: "Pedro Henrique Rocha"
  pair_programming: "AI-assisted via gsd-* commands"

extraction_method: "git log --reverse --pretty=format:'%h|%ad|%s' --date=short --since=2026-02-01"
last_commit_analyzed: b98c5df
last_commit_date: 2026-05-18
```

## Notas metodológicas

1. **Cronograma é retrospectivo**, não preditivo. Foi reconstruído a partir do histórico Git em 18/mai/2026, no fechamento do escopo entregue ao TCC.
2. O mapeamento commit → fase utiliza prefixos convencionais (`feat(NN-NN):`, `Phase NN`, `feat(milestone):`) presentes na mensagem de commit. Quando o prefixo é ausente, o commit foi alocado ao sprint mais próximo cronologicamente, respeitando dependências do `ROADMAP.md`.
3. Fases com numeração decimal (ex.: `10.1`, `14.2`) representam **inserções urgentes** descobertas após o plano original — geralmente bugs de produção ou ajustes de fonte de dados externa.
4. O autor solo declarado é **Pedro Henrique Rocha** (Engenharia da Computação, FEAU-USJT). A assistência de IA (Claude, Gemini) está documentada como ferramenta de pair-programming nos commits e em `.planning/PROJECT.md` — não constitui co-autoria do trabalho acadêmico.
