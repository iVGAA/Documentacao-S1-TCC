---
file: 05-diagramas-sequencia.md
section: 5 — Diagramas de Sequência
purpose: 20 diagramas Mermaid sequenceDiagram alinhados 1-para-1 com UC-01..UC-20
depends_on: [03-casos-de-uso-diagramas.md, 04-casos-de-uso-descricoes.md]
last_updated: 2026-05-18
status: draft
---

# 5. Diagramas de Sequência

> **Para agentes de IA**: esta seção contém **20 diagramas Mermaid
> `sequenceDiagram`**, alinhados 1-para-1 com os casos de uso descritos em
> `04-casos-de-uso-descricoes.md`. A sub-seção §5.N corresponde sempre a
> UC-NN (ex.: §5.07 ↔ UC-07). Os participantes seguem a legenda do bloco
> `agent-extractable` no rodapé. Mensagens entre participantes são em
> português brasileiro e refletem o comportamento real implementado em
> `src/app/**` e `src/lib/**` — incluindo as rotas de API, as queries
> Drizzle ao Neon Postgres, o cache do Vercel Edge e o upload para o
> Vercel Blob.

Os diagramas a seguir representam a troca de mensagens entre os atores
(Visitante, Administrador), o navegador, o runtime do Next.js na Vercel,
o banco de dados Neon Postgres, o armazenamento de blobs (Vercel Blob),
as APIs externas (SSD PCJ, SIMQUA, OSM) e o Vercel Cron. A divisão
segue a fronteira lógica de execução: o que roda no cliente, o que roda
no Edge (CDN), o que roda no servidor (RSC ou Route Handler) e o que
roda em sistemas externos.

## 5.1 Diagrama "UC-01 Visualizar mapa em tempo real"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant Cache as Vercel Edge
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    V->>B: acessa "/"
    B->>Cache: GET /
    Cache->>N: cache miss (SSR/ISR)
    N->>DB: SELECT últimas medições por estação
    DB-->>N: rows (qualidade + quantidade)
    N->>DB: SELECT estações ativas com geo
    DB-->>N: rows (lat/lng/classe)
    N-->>Cache: HTML renderizado (RSC)
    Cache-->>B: HTML + headers de cache
    B-->>V: layout visível (LCP)
    B->>B: hidrata Leaflet + OSM tiles
    B-->>V: mapa interativo pronto
```

## 5.2 Diagrama "UC-02 Visualizar popup de estação"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant L as Leaflet<br/>(client)

    V->>B: clica em marcador da estação
    B->>L: marker.click()
    L->>L: lê props pré-hidratadas (estação + última leitura)
    L-->>B: monta popup HTML (nome, classe, parâmetros)
    B-->>V: popup renderizado in-place
    V->>B: clica em "ver detalhes do rio"
    B-->>V: navega para /rio/[id] (UC-03)
```

## 5.3 Diagrama "UC-03 Ver detalhes históricos de rio"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    V->>B: acessa /rio/piracicaba
    B->>N: GET /rio/[id]
    N->>DB: SELECT rio por slug/key
    DB-->>N: row (rio + metadados)
    N->>DB: SELECT estações vinculadas ao rio
    DB-->>N: rows (estações)
    N->>DB: SELECT série histórica (medições + estatísticas mensais)
    DB-->>N: rows agrupados por parâmetro
    N-->>B: HTML com cards de parâmetros + sparklines
    B-->>V: página renderizada (CONAMA 357 + IQA)
```

## 5.4 Diagrama "UC-04 Ver atividade recente do sistema"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    V->>B: acessa /admin/atividade
    B->>N: GET /admin/atividade
    N->>DB: SELECT ingest_logs ORDER BY started_at DESC LIMIT 50
    DB-->>N: rows (logs públicos)
    N->>DB: SELECT últimas publicações de boletins
    DB-->>N: rows (relatorios published)
    N-->>B: HTML com timeline + heatmap
    B-->>V: lista de execuções e publicações recentes
```

## 5.5 Diagrama "UC-05 Buscar termo no glossário"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant Cache as Vercel Edge
    participant N as Next.js<br/>(Server)

    V->>B: acessa /glossario
    B->>Cache: GET /glossario
    Cache->>N: cache miss (rota estática)
    N-->>Cache: HTML pré-renderizado (glossário inline)
    Cache-->>B: HTML + cache-hit nas próximas visitas
    B-->>V: lista de termos visível
    V->>B: digita "IQA" no campo de busca
    B->>B: filtra in-memory + atualiza âncora ?q=iqa
    B-->>V: scroll suave até cartão "IQA"
```

## 5.6 Diagrama "UC-06 Listar boletins disponíveis"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    V->>B: acessa /relatorios
    B->>N: GET /relatorios
    N->>DB: SELECT reports WHERE lifecycle = 'published' AND is_public AND deleted_at IS NULL
    DB-->>N: rows (id, mês, ano, pdf_url, capa)
    N-->>B: HTML com cards por boletim (PDF + metadados)
    B-->>V: lista de boletins disponíveis
```

## 5.7 Diagrama "UC-07 Baixar boletim PDF"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant Blob as Vercel Blob

    V->>B: clica em "Baixar PDF"
    B->>Blob: GET pdf_url (https://*.blob.vercel-storage.com/...)
    Blob-->>B: streaming binário do PDF
    B-->>V: dialog "Salvar como…" (ou abre no viewer)
    Note over B,Blob: URL pública assinada — sem<br/>chamada ao servidor Next.js
```

## 5.8 Diagrama "UC-08 Listar estações monitoradas"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    V->>B: acessa /admin/estacoes
    B->>N: GET /admin/estacoes
    N->>DB: SELECT estações + último ingest_at por origem
    DB-->>N: rows (estações + status de frescor)
    N->>DB: SELECT classes CONAMA por estação
    DB-->>N: rows (parâmetros enquadrados)
    N-->>B: HTML com tabela responsiva (stack-to-card no mobile)
    B-->>V: lista completa de estações
```

## 5.9 Diagrama "UC-09 Filtrar estações por bacia/classe"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant URL as History API

    V->>B: seleciona bacia "Piracicaba" + classe 2
    B->>B: aplica filtro in-memory sobre rows pré-hidratados
    B->>URL: pushState ?bacia=piracicaba&classe=2
    URL-->>B: URL atualizada (sem reload)
    B-->>V: tabela re-renderizada (somente matches)
    Note over B: Sem round-trip ao servidor —<br/>dados já vieram em UC-08
```

## 5.10 Diagrama "UC-10 Consultar página institucional / footer"

```mermaid
sequenceDiagram
    actor V as Visitante
    participant B as Browser
    participant Cache as Vercel Edge
    participant N as Next.js<br/>(Server)

    V->>B: acessa /sobre (ou rola até o footer)
    B->>Cache: GET /sobre
    Cache->>N: cache miss (rota estática)
    N-->>Cache: HTML institucional pré-renderizado
    Cache-->>B: HTML + cache hit subsequente
    B-->>V: página institucional + créditos + links
```

## 5.11 Diagrama "UC-11 Autenticar-se"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Server Action)
    participant DB as Neon Postgres

    A->>B: acessa /login e submete e-mail + senha
    B->>N: POST /login (Server Action loginAction)
    N->>DB: SELECT user_accounts WHERE email = $1
    DB-->>N: row (hash, role)
    N->>N: bcrypt.compare(senha, hash)
    alt credenciais válidas
        N->>DB: INSERT sessions (user_id, token, expires_at)
        DB-->>N: ok
        N-->>B: Set-Cookie session=<token>; HttpOnly; Secure
        B-->>A: redirect /admin
    else credenciais inválidas
        N-->>B: render /login com erro
        B-->>A: mensagem "Credenciais inválidas"
    end
```

## 5.12 Diagrama "UC-12 Visualizar dashboard administrativo"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    A->>B: acessa /admin
    B->>N: GET /admin (cookie session=<token>)
    N->>N: middleware → getSession() valida cookie
    N->>DB: SELECT count(reports), count(ingest_logs), uptime
    DB-->>N: agregados de saúde
    N->>DB: SELECT últimas 10 execuções de ingest
    DB-->>N: rows
    N-->>B: HTML do dashboard (KPIs + sparklines)
    B-->>A: dashboard renderizado
```

## 5.13 Diagrama "UC-13 Monitorar pipeline de ingest"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    A->>B: acessa /admin/pipeline
    B->>N: GET /admin/pipeline (cookie session)
    N->>N: getSession() + role check
    N->>DB: SELECT ingest_logs ORDER BY started_at DESC LIMIT 200
    DB-->>N: rows (source, status, rows_inserted, duration_ms, error)
    N->>DB: SELECT última execução por source
    DB-->>N: heads (SIMQUA, SSD PCJ, retention)
    N-->>B: HTML com timeline + tabela detalhada
    B-->>A: pipeline observável (status de cada cron)
```

## 5.14 Diagrama "UC-14 Disparar ingest manual"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Route Handler)
    participant ExtAPI as SSD PCJ / SIMQUA
    participant DB as Neon Postgres

    A->>B: clica "Rodar SSD PCJ agora"
    B->>N: POST /api/admin/pipeline/run-ssd-pcj
    N->>N: getSession() + verifica role admin
    N->>DB: INSERT ingest_logs (status='running')
    DB-->>N: log_id
    N->>ExtAPI: GET endpoints telemétricos (paralelo por estação)
    ExtAPI-->>N: JSON com séries de vazão / chuva
    N->>DB: UPSERT measurements (batched)
    DB-->>N: rows_inserted
    N->>DB: UPDATE ingest_logs SET status='success', duration_ms
    DB-->>N: ok
    N-->>B: 200 { success: true, rows: N }
    B-->>A: toast "Ingest concluído (N linhas)"
```

## 5.15 Diagrama "UC-15 Listar boletins gerados"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    A->>B: acessa /admin/boletins
    B->>N: GET /admin/boletins (cookie session)
    N->>N: getSession() + role admin
    N->>DB: SELECT reports WHERE deleted_at IS NULL ORDER BY year DESC, month DESC
    DB-->>N: rows (todos os lifecycles: draft, approved, published, archived)
    N-->>B: HTML com tabela administrativa (filtros por lifecycle)
    B-->>A: lista completa com ações (gerar, editar, apagar, publicar)
```

## 5.16 Diagrama "UC-16 Gerar/regenerar boletim"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Route Handler)
    participant DB as Neon Postgres
    participant Blob as Vercel Blob

    A->>B: clica "Gerar boletim de abril/2026"
    B->>N: POST /api/reports/generate-bulletin { month, year }
    N->>N: getSession()/Bearer + role admin
    N->>DB: SELECT measurements agregadas por par de estações (5 pares)
    DB-->>N: rows (séries diárias)
    N->>N: calcStationStatistics + chartjs-node-canvas (30 charts)
    N->>N: @react-pdf/renderer → buffer PDF (7 páginas)
    N->>Blob: PUT bulletin-YYYY-MM.pdf
    Blob-->>N: pdf_url assinada
    N->>DB: INSERT reports (lifecycle='draft', pdf_url, metadata)
    DB-->>N: report_id
    N-->>B: 200 { reportId, pdfUrl, stationsProcessed }
    A->>B: clica "Publicar"
    B->>N: POST /api/reports/[id]/publish
    N->>DB: UPDATE reports SET lifecycle='published', is_public=true, published_at=now()
    DB-->>N: ok
    N-->>B: 200 { published: true }
    B-->>A: card atualizado como "Publicado"
```

## 5.17 Diagrama "UC-17 Editar template do boletim"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Route Handler)
    participant DB as Neon Postgres

    A->>B: acessa /admin/boletim/[id]/editor
    B->>N: GET /api/reports/[id]
    N->>DB: SELECT report (overrides, station_notes, analyst_notes)
    DB-->>N: row
    N-->>B: JSON com overrides
    B-->>A: editor com campos pré-preenchidos
    A->>B: edita textos e clica "Pré-visualizar"
    B->>N: POST /api/admin/bulletin/render-template { overrides }
    N->>N: renderiza template com merge de overrides
    N-->>B: HTML pré-visualizado (iframe)
    A->>B: clica "Salvar"
    B->>N: PATCH /api/reports/[id] { overrides, analystNotes }
    N->>DB: UPDATE reports SET overrides=$1, analyst_notes=$2, updated_at=now()
    DB-->>N: ok
    N-->>B: 200 { saved: true }
    B-->>A: toast "Template salvo"
```

## 5.18 Diagrama "UC-18 Apagar boletim"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Route Handler)
    participant DB as Neon Postgres

    A->>B: clica no ícone "Apagar" do card do boletim
    B-->>A: modal de confirmação ("Esta ação é reversível pelo DBA")
    A->>B: confirma exclusão
    B->>N: DELETE /api/reports/[id]
    N->>N: getSession() + role admin
    N->>DB: UPDATE reports SET deleted_at=now() WHERE id=$1
    DB-->>N: ok (soft delete)
    N->>N: revalidatePath('/admin/boletins')
    N-->>B: 200 { deleted: true }
    B-->>A: card removido da listagem
    Note over Blob: PDF no Vercel Blob permanece —<br/>limpeza periódica via cron de retenção
```

## 5.19 Diagrama "UC-19 Visualizar atividade administrativa"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant N as Next.js<br/>(Server)
    participant DB as Neon Postgres

    A->>B: acessa /admin/atividade
    B->>N: GET /admin/atividade (cookie session)
    N->>N: getSession() + role admin
    N->>DB: SELECT ingest_logs + audit trail (usuário, ação, alvo) últimas 100
    DB-->>N: rows
    N->>DB: SELECT reports com eventos de lifecycle (criação, publicação, exclusão)
    DB-->>N: rows
    N-->>B: HTML com feed cronológico de atividade
    B-->>A: feed completo (auditoria interna)
```

## 5.20 Diagrama "UC-20 Visualizar onboarding/checklist"

```mermaid
sequenceDiagram
    actor A as Admin
    participant B as Browser
    participant LS as localStorage

    A->>B: primeiro login → carrega /admin
    B->>LS: leitura "ecodata:onboarding:state"
    LS-->>B: undefined (primeira visita)
    B-->>A: renderiza checklist (5 passos)
    A->>B: marca passo "explorei o pipeline"
    B->>LS: write "ecodata:onboarding:state" = { step1: true }
    LS-->>B: ok
    B-->>A: progresso atualizado (UI client-side)
    Note over B,LS: Sem chamada ao servidor —<br/>estado persistido localmente
```

---

## Referências cruzadas

- `03-casos-de-uso-diagramas.md` — visão geral dos atores e relações `<<include>>`
- `04-casos-de-uso-descricoes.md` — descrição textual (fluxos, pré/pós-condições) de cada UC
- `06-arquitetura.md` — camadas, deploy e integrações (contexto dos participantes)
- `07-banco-de-dados.md` — schema e tabelas acessadas pelos diagramas (`reports`, `ingest_logs`, `measurements`, `stations`, `sessions`)
- `08-casos-de-teste.md` — casos de teste alinhados 1-para-1

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: section-05-sequence-diagrams
total_diagrams: 20
alignment: 5.N matches UC-NN in AGENT-CONTEXT.md
participants_legend:
  V: Visitante
  Admin: Administrador
  B: Browser
  N: Next.js Server
  DB: Neon Postgres
  Cache: Vercel Edge CDN
  Blob: Vercel Blob
  Cron: Vercel Cron
  ExtAPI: SSD PCJ / SIMQUA / OSM
  LS: localStorage (client)
  L: Leaflet (client)
  URL: History API (client)
diagrams:
  - id: 5.1
    use_case: uc-01
    participants: [V, B, Cache, N, DB]
  - id: 5.2
    use_case: uc-02
    participants: [V, B, L]
  - id: 5.3
    use_case: uc-03
    participants: [V, B, N, DB]
  - id: 5.4
    use_case: uc-04
    participants: [V, B, N, DB]
  - id: 5.5
    use_case: uc-05
    participants: [V, B, Cache, N]
  - id: 5.6
    use_case: uc-06
    participants: [V, B, N, DB]
  - id: 5.7
    use_case: uc-07
    participants: [V, B, Blob]
  - id: 5.8
    use_case: uc-08
    participants: [V, B, N, DB]
  - id: 5.9
    use_case: uc-09
    participants: [V, B, URL]
  - id: 5.10
    use_case: uc-10
    participants: [V, B, Cache, N]
  - id: 5.11
    use_case: uc-11
    participants: [Admin, B, N, DB]
  - id: 5.12
    use_case: uc-12
    participants: [Admin, B, N, DB]
  - id: 5.13
    use_case: uc-13
    participants: [Admin, B, N, DB]
  - id: 5.14
    use_case: uc-14
    participants: [Admin, B, N, ExtAPI, DB]
  - id: 5.15
    use_case: uc-15
    participants: [Admin, B, N, DB]
  - id: 5.16
    use_case: uc-16
    participants: [Admin, B, N, DB, Blob]
  - id: 5.17
    use_case: uc-17
    participants: [Admin, B, N, DB]
  - id: 5.18
    use_case: uc-18
    participants: [Admin, B, N, DB]
  - id: 5.19
    use_case: uc-19
    participants: [Admin, B, N, DB]
  - id: 5.20
    use_case: uc-20
    participants: [Admin, B, LS]
```
<!-- /agent-extractable -->
