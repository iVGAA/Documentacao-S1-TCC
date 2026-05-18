# Agent Context — EcoData TCC

This file is the single entry point for AI agents loading `docs/tcc/` as context. Read this first.

## Project digest

EcoData is a real-time water-quality monitoring portal for the **PCJ basin** (Rios Piracicaba, Capivari, Jundiaí), built on **Next.js 15 + Neon Postgres + Vercel Blob**. Hourly ingestion from CETESB SIMQUA (quality) and SSD PCJ (quantity) APIs.

**Rotas reais (MVP atual):**

- Públicas: `/` (home + mapa), `/mapa` (mapa em tela cheia), `/rio/[id]` (detalhes históricos por trecho), `/relatorios` (boletins), `/glossario`, `/sobre`, `/login` (login do admin)
- Admin (autenticadas): `/admin/dashboard`, `/admin/pipeline`, `/admin/boletins`, `/admin/boletim/preview`, `/admin/atividade`, `/admin/estacoes`, `/admin/templates`
- Onboarding admin: modal + checklist injetados em `/admin/*` via `src/components/admin/WelcomeModal.tsx` + `OnboardingChecklist.tsx` (não é rota dedicada)

> **Drift conhecido**: a primeira versão deste arquivo listava rotas públicas de atividade e de estações que **não existem** no código atual. No MVP, essas superfícies só estão sob `/admin/atividade` e `/admin/estacoes`. Casos de uso UC-04, UC-08 e UC-09 ficam no índice como previstos para V2 pública; em §4 estão descritos contra a tela admin atualmente shipada.

## Tech stack
| Layer            | Tech                                            |
|------------------|-------------------------------------------------|
| Frontend         | Next.js 15 App Router, React 19, Tailwind v4    |
| UI primitives    | shadcn/ui (Radix), lucide-react                 |
| State / data     | Server Components + drizzle-orm on Neon         |
| Map              | Leaflet + OpenStreetMap basemap                 |
| PDF generation   | @react-pdf/renderer 4.x                         |
| Chart rendering  | chartjs-node-canvas (server-side PNG)           |
| Storage          | Vercel Blob (PDFs)                              |
| Auth             | Custom session-based (cookie), `getSession()`   |
| Cron             | Vercel Cron + Bearer-secret POST endpoints      |
| Hosting          | Vercel                                          |

## Domain glossary (10 essentials)

| Term         | Definition                                                                                        |
|--------------|---------------------------------------------------------------------------------------------------|
| Bacia PCJ    | Bacias hidrográficas dos rios Piracicaba, Capivari e Jundiaí                                      |
| CONAMA 357   | Resolução que classifica corpos d'água por uso e define limites de qualidade                      |
| IQA          | Índice de Qualidade da Água (NSF, 0-100)                                                          |
| OD           | Oxigênio dissolvido (mg/L) — proxy para vida aquática                                             |
| Q7,10        | Vazão mínima de 7 dias com 10 anos de recorrência — referência para outorga                       |
| SIMQUA       | Sistema de Informações da Monitoramento da Qualidade das Águas (CETESB)                          |
| SSD PCJ      | Sistema de Suporte à Decisão das Bacias PCJ — fonte telemétrica de quantidade                     |
| SPAguas      | Departamento de Águas e Energia Elétrica do estado de SP (ex-DAEE)                                |
| Boletim      | Publicação mensal integrada com dados de qualidade + quantidade por trecho de rio                 |
| Lifecycle    | Estados do boletim: draft → approved → published → archived                                       |

## File map

| File                                  | Purpose                                                        |
|---------------------------------------|----------------------------------------------------------------|
| `README.md`                           | Navegação, ordem de leitura, índice geral                      |
| `METADATA.yaml`                       | Metadados (autor, versão, instituição) para conversão Pandoc   |
| `AGENT-CONTEXT.md`                    | Este arquivo — digest para agentes de IA                       |
| `01-introducao.md`                    | Seção 1: propósito + público-alvo                              |
| `02-visao-geral.md`                   | Seção 2: visão geral do produto + modelo de operação           |
| `03-casos-de-uso-diagramas.md`        | Seção 3: diagramas de casos de uso (Mermaid)                   |
| `04-casos-de-uso-descricoes.md`       | Seção 4: descrições dos 20 casos de uso                        |
| `05-diagramas-sequencia.md`           | Seção 5: 20 diagramas de sequência (Mermaid)                   |
| `06-arquitetura.md`                   | Seção 6: arquitetura (camadas, deploy, integrações)            |
| `07-banco-de-dados.md`                | Seção 7: BD + diagrama ER (Mermaid)                            |
| `08-casos-de-teste.md`                | Seção 8: 20 casos de teste (propósito, entradas, saída)        |
| `09-telas-da-solucao/`                | Seção 9: 1 .md por tela + screenshots em `screens/`            |
| `10-planejamento-atividades.md`       | Seção 10: sprints reais derivados do git log                   |
| `glossario-tecnico.md`                | Glossário expandido (termos técnicos do projeto)               |

## Extraction patterns

- **"Como funciona X?"** → comece em `04-casos-de-uso-descricoes.md`, drill em `05-` e `06-`
- **"Qual o schema do banco?"** → `07-banco-de-dados.md`
- **"Como é testado X?"** → `08-casos-de-teste.md` (seção bate com número do caso de uso)
- **"Como é a tela de X?"** → `09-telas-da-solucao/9.NN-<slug>.md`
- **"Quando foi feito X?"** → `10-planejamento-atividades.md`
- **"O que é IQA / OD / CONAMA?"** → `glossario-tecnico.md` ou tabela acima

## Numbering convention

Seções 4, 5, 8, 9 estão **alinhadas por número**:
- Caso de uso UC-05 → descrito em §4.5, diagrama em §5.5, teste em §8.5, tela em §9.5

Mantenha esse alinhamento ao adicionar ou remover casos de uso.

## Casos de uso (índice rápido)

### Visitante (UC-01 a UC-10)
1. Visualizar mapa em tempo real
2. Visualizar popup de estação no mapa
3. Ver detalhes históricos de rio
4. Ver atividade recente do sistema
5. Buscar termo no glossário
6. Listar boletins disponíveis
7. Baixar boletim PDF
8. Listar estações monitoradas
9. Filtrar estações por bacia/classe
10. Consultar página institucional / footer

### Administrador (UC-11 a UC-20)
11. Autenticar-se
12. Visualizar dashboard administrativo
13. Monitorar pipeline de ingest
14. Disparar ingest manual
15. Listar boletins gerados
16. Gerar/regenerar boletim
17. Editar template do boletim
18. Apagar boletim
19. Visualizar atividade administrativa
20. Visualizar onboarding/checklist
