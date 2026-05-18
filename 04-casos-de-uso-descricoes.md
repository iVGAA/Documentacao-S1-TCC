---
file: 04-casos-de-uso-descricoes.md
section: 4. Descrições dos Casos de Uso
purpose: Descreve em detalhe cada um dos 20 casos de uso do EcoData
depends_on: [03-casos-de-uso-diagramas.md, AGENT-CONTEXT.md]
last_updated: 2026-05-18
status: draft
---

# 4. Descrições dos Casos de Uso

> **Para agentes de IA**: esta seção apresenta os 20 casos de uso descritos no padrão Mandou Bem (descrição, atores, pré-condições, fluxo principal, fluxos alternativos e fluxos de exceção). A numeração de §4.1 a §4.20 corresponde exatamente aos identificadores UC-01 a UC-20 do `AGENT-CONTEXT.md` e se alinha com §5.N (diagramas de sequência), §8.N (casos de teste) e §9.N (telas da solução). Cada fluxo principal foi derivado da implementação real (arquivos em `src/app/**` e `src/lib/auth.ts`), não de comportamento idealizado.

## Identificação dos atores

| #Ator   | Nome                | Descrição                                                                                                                |
|---------|---------------------|--------------------------------------------------------------------------------------------------------------------------|
| Ator-01 | Visitante / Cidadão | Usuário anônimo que acessa o portal público para consultar dados de qualidade da água, baixar boletins e estudar termos. |
| Ator-02 | Administrador       | Operador autenticado (perfil `admin`, login mock) responsável por gerar, revisar, publicar e excluir boletins.           |

> Os fluxos administrativos (UC-11 a UC-20) exigem sessão com `role = 'admin'` materializada no cookie `ecodata_session` (`src/lib/auth.ts`). Sem sessão válida, qualquer rota sob `/admin/*` redireciona para `/login` por meio de `requireAdmin()`.

---

## 4.1 Caso de uso "UC-01 Visualizar mapa em tempo real"

**Descrição:** o Visitante acessa a página inicial do EcoData (`/`) e visualiza um painel composto pelo herói com a contagem de status CONAMA, a grade-semáforo das estações monitoradas, uma prévia interativa do mapa Leaflet e os atalhos para os boletins recentes. A página é renderizada como Server Component com revalidação ISR de 3600s e duas janelas de dados pré-buscadas (24h e 7d).

**Atores:** Visitante.

**Pré-condições:** não se aplica — a página é pública e renderiza com dados fallback mesmo quando o banco está offline.

**Fluxo principal:**
1. O Visitante acessa `https://<host>/`.
2. O servidor executa `getDashboardData('24h')` e `getDashboardData('7d')` em paralelo e devolve o HTML pré-renderizado.
3. O cliente carrega `DashboardClient`, que monta sequencialmente: `HeroSection` (contagem de status), `SemaphoreGrid` (estações), `MapPreview` (Leaflet compacto), `BoletimCards` (acordeão de boletins) e `LastUpdated` (timestamp).
4. O Visitante alterna o seletor de janela entre "24h" e "7d"; o componente troca o estado local sem nova chamada de API.
5. O Visitante clica no overlay "Ver mapa completo" sobre a prévia do mapa.
6. O cliente navega para `/mapa`, onde o Leaflet ganha controles de zoom, camadas e popups completos.

**Fluxos alternativos:**
- **A1 — Sem dados frescos no banco:** quando uma consulta retorna lista vazia, a grade-semáforo exibe placeholders e o herói mostra contagem zero, preservando o layout.

**Fluxos de exceção:**
- **E1 — Falha ao consultar Postgres:** `getDashboardData` propaga erro; o Next.js renderiza a página de erro 500 padrão. O cache ISR anterior continua válido para outros visitantes até a próxima revalidação.

---

## 4.2 Caso de uso "UC-02 Visualizar popup de estação no mapa"

**Descrição:** sobre o mapa público, o Visitante clica em um marcador de estação e abre um cartão flutuante (`StationPopup`) com faixa de status edge-to-edge, identificação da estação (nome serifado, rio, município, código), lista de parâmetros com classificação CONAMA por bolinha colorida, timestamps relativos e absolutos da última leitura e um CTA "Ver detalhes da estação" que linka para `/rio/[id]`.

**Atores:** Visitante.

**Pré-condições:** o mapa em `/` ou `/mapa` deve estar carregado e a estação clicada deve estar presente em `MapStation[]`.

**Fluxo principal:**
1. O Visitante posiciona o cursor sobre um marcador `QualityStationMarker` ou `QuantityStationMarker`.
2. O Visitante clica no marcador; o Leaflet abre o popup associado.
3. O componente `StationPopup` renderiza a `StatusBand` calculando cor e rótulo a partir de `station.conamaLevel`, `station.isStale` e `station.latestMeasurements.length`.
4. O Visitante percorre a lista zebrada de `ParameterRow` com valores em fonte mono e indicador CONAMA quando a estação é de qualidade.
5. O Visitante clica em "Ver detalhes da estação".
6. O navegador segue para `/rio/<station.id>`, abrindo o detalhamento histórico (UC-03).

**Fluxos alternativos:**
- **A1 — Estação sem medições:** quando `latestMeasurements.length === 0`, a banda exibe "Sem dados" e a área de parâmetros mostra a frase "Nenhuma medição registrada nesta estação." no lugar das linhas.
- **A2 — Telemetria desatualizada:** se `station.isStale` é verdadeiro, o rodapé do popup mostra a pílula amarela "desatualizado" junto ao timestamp.

**Fluxos de exceção:** não se aplica — o popup opera sobre dados já carregados pelo cliente.

---

## 4.3 Caso de uso "UC-03 Ver detalhes históricos de rio"

**Descrição:** o Visitante consulta a página de detalhe de uma estação em `/rio/[id]`, recebendo metadados (nome, rio, município), a lista de parâmetros monitorados e séries históricas pré-carregadas para a janela de 7 dias. A página é renderizada como Server Component com ISR de uma hora e gera params estáticos via `generateStaticStationParams()`.

**Atores:** Visitante.

**Pré-condições:** existir uma estação cujo `id` numérico bata com o parâmetro de rota; caso contrário, o Next chama `notFound()`.

**Fluxo principal:**
1. O Visitante acessa `/rio/<id>` (em geral via popup do mapa ou via lista no admin/estações).
2. O servidor invoca `getStationDetail(Number(id))`; se devolver `null`, o framework dispara `notFound()` e renderiza a página 404.
3. O servidor busca `getStationParameters(station.id)` para enumerar os parâmetros monitorados.
4. Para cada parâmetro, o servidor chama `getChartData(station.id, param.id, '7d')` em paralelo e monta `initialChartData`.
5. O `StationDetailClient` recebe os dados pré-carregados e renderiza o cabeçalho, abas de parâmetros e gráficos.
6. O Visitante alterna parâmetros e janela temporal nos controles do cliente; novas séries são solicitadas sob demanda.

**Fluxos alternativos:**
- **A1 — Slug inexistente:** ao receber id inválido, a rota retorna 404 antes de qualquer renderização do client.

**Fluxos de exceção:**
- **E1 — Falha no Postgres durante `Promise.all`:** o Server Component propaga o erro e a página de erro padrão do Next é renderizada, sem corromper o cache ISR vigente.

---

## 4.4 Caso de uso "UC-04 Ver atividade recente do sistema"

**Descrição:** em `/admin/atividade` o usuário visualiza um feed cronológico que agrega eventos de boletins (criação e publicação) e grupos de alertas (`severity + parameterCode`) gerados pelos pipelines. O feed permite filtrar entre "Todos", "Alertas" e "Boletins" via query string e exibe um heatmap mensal de densidade × severidade. Esta é a única superfície de "atividade" entregue no MVP; a versão pública prevista no AGENT-CONTEXT permanece como roadmap (V2 — auditoria com auth real).

**Atores:** Administrador (interface ativa); Visitante (visão pública futura, ver §9.4).

**Pré-condições:** sessão válida com `role = 'admin'`; existir registros em `reports` e/ou `alerts`.

**Fluxo principal:**
1. O Administrador acessa `/admin/atividade`.
2. O servidor lê `searchParams.tipo` (padrão `todos`) e dispara `loadEvents(filter)`.
3. `loadEvents` busca os 20 boletins não-excluídos mais recentes e os 80 alertas ativos mais recentes; alertas similares (`severity + parameterCode`) são agrupados em `AlertGroupEvent`.
4. Os eventos são ordenados por timestamp decrescente, truncados em 30 itens e o heatmap mensal é construído por `buildHeatmap`.
5. A página renderiza o filtro segmentado, o heatmap (quando `totalEvents > 0`) e a timeline com dot, ícone, chip de tipo, título e timestamp relativo.
6. O Administrador clica em outro segmento do filtro; o Next reexecuta a rota com a query string atualizada.

**Fluxos alternativos:**
- **A1 — Feed vazio:** quando `events.length === 0`, a página exibe o placeholder "Nenhuma atividade ainda." e omite o heatmap.

**Fluxos de exceção:**
- **E1 — Falha no banco:** `loadEvents` captura o erro, registra `console.error('[admin/atividade]', err)` e retorna lista vazia, preservando o layout (cai no fluxo A1).

---

## 4.5 Caso de uso "UC-05 Buscar termo no glossário"

**Descrição:** o Visitante acessa `/glossario`, navega por categorias temáticas (parâmetros de qualidade, hidrológicos, instituições, conceitos) e utiliza a busca instantânea para localizar termos como "IQA", "vazão" ou "turbidez". Cada `GlossaryCard` expansível mostra explicação direta, micro-visualização contextual (barras IQA, régua de pH, limites de OD ou padrão sazonal de vazão), analogia do dia a dia, fonte oficial e botão "Copiar link" com âncora SEO.

**Atores:** Visitante.

**Pré-condições:** não se aplica — o glossário é estático, importado de `@/data/glossary`.

**Fluxo principal:**
1. O Visitante abre `/glossario`.
2. O cliente registra o atalho global "/" para focar o campo de busca quando não há outro input ativo.
3. O Visitante digita um termo; `normalize()` aplica `NFD` + lower-case e o `useMemo` filtra a lista por `title + explanation`.
4. A página re-renderiza a contagem de resultados e distribui os itens nas quatro categorias.
5. O Visitante clica em um cartão; o `useState` interno alterna `open`, o Framer Motion anima a expansão e exibe a explicação completa + analogia + micro-viz.
6. O Visitante clica em "Copiar link"; `handleCopyLink` grava `window.location.origin + /glossario#<id>` no clipboard e dispara o toast "Link copiado".

**Fluxos alternativos:**
- **A1 — Nenhum resultado:** quando o filtro devolve zero itens, é renderizado um estado vazio com a frase "Nenhum termo encontrado para …" e o atalho "Limpar busca".

**Fluxos de exceção:**
- **E1 — Falha de clipboard:** `navigator.clipboard.writeText` rejeita; o `catch` dispara `toast.error("Não foi possível copiar")` sem quebrar a página.

---

## 4.6 Caso de uso "UC-06 Listar boletins disponíveis"

**Descrição:** o Visitante acessa `/relatorios` e visualiza duas seções: os boletins gerados pelo EcoData (`reports` em status `ready` com `pdfUrl`) e os boletins originais do SSPCJ (`boletins` com `blobUrl`). Cada item exibe ícone PDF, mês/ano, indicador de versão (quando há mais de uma publicação no mesmo mês) e ação de download. A rota é `dynamic = 'force-dynamic'` para refletir publicação/exclusão imediatamente.

**Atores:** Visitante; Administrador (vê adicionalmente o botão de exclusão por linha).

**Pré-condições:** não se aplica.

**Fluxo principal:**
1. O Visitante abre `/relatorios`.
2. O servidor executa em paralelo `getGeneratedBulletins()`, `getOriginalBulletins()` e `getSession()`.
3. `readyBulletins` é filtrado pelos itens com `status === 'ready'` e `pdfUrl` não-nulo; `monthCounts` rastreia duplicatas para anexar `(v<N>)` na label.
4. A página renderiza a seção "Boletins Gerados pelo EcoData" como grade responsiva, e abaixo a seção "Boletins Originais SSPCJ".
5. Quando `session.role === 'admin'`, cada cartão gerado recebe o componente `DeleteBulletinButton`.
6. O Visitante clica no ícone, na label ou no botão de download; o navegador abre `pdfUrl` em nova aba.

**Fluxos alternativos:**
- **A1 — Sem boletins gerados:** a seção exibe o estado vazio "Nenhum boletim gerado ainda."
- **A2 — Sem originais com download:** a segunda seção mostra "Nenhum boletim original disponível para download."

**Fluxos de exceção:**
- **E1 — Falha nas queries Drizzle:** o `try/catch` em cada loader devolve `[]` e a página renderiza ambos os estados vazios.

---

## 4.7 Caso de uso "UC-07 Baixar boletim PDF"

**Descrição:** a partir de `/relatorios`, o Visitante baixa um PDF — gerado pelo EcoData (`pdfUrl` apontando para Vercel Blob) ou um original SSPCJ (`blobUrl`). O download abre em nova aba com `rel="noopener noreferrer"`; o arquivo permanece hospedado no blob e contabiliza o tráfego pela CDN da Vercel.

**Atores:** Visitante.

**Pré-condições:** o boletim alvo deve ter `pdfUrl` (gerado) ou `blobUrl` (original) preenchido; para os gerados, `status === 'ready'`.

**Fluxo principal:**
1. O Visitante localiza o boletim desejado em `/relatorios`.
2. O Visitante clica no ícone PDF, no título ou no ícone `Download` da linha.
3. O navegador segue o atributo `href={bulletin.pdfUrl}` (ou `blobUrl`) com `target="_blank"`.
4. A Vercel CDN devolve o PDF; o navegador baixa ou abre o arquivo conforme a preferência do usuário.

**Fluxos alternativos:**
- **A1 — Boletim original SSPCJ:** o atalho usa `blobUrl` e a label vem de `bulletin.label || formatMonthId(bulletin.monthId)`; o ícone exibido é `ExternalLink` em vez de `Download`.

**Fluxos de exceção:**
- **E1 — Blob indisponível:** se a Vercel CDN retornar 404/403, o navegador exibe a página de erro do próprio Blob; a página `/relatorios` permanece intacta.

---

## 4.8 Caso de uso "UC-08 Listar estações monitoradas"

**Descrição:** a listagem das cinco estações pareadas (qualidade + quantidade) é entregue em `/admin/estacoes`. Cada cartão mostra o código CETESB, classe CONAMA, rio e município, métricas-chave (OD médio, conformidade %, vazão média, Q7,10) e um sparkline SVG da série diária de OD com a linha pontilhada do limite CONAMA. O cartão inteiro é stretched-link para `/rio/<pair.key>` (UC-03). No MVP, esta superfície é restrita ao admin; a versão pública prevista no AGENT-CONTEXT é roadmap.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; existir dados em `monthly_stats` ou em `measurements` para o mês de referência (mês fechado anterior).

**Fluxo principal:**
1. O Administrador acessa `/admin/estacoes`.
2. `loadStations` itera `STATION_PAIRS` e, em paralelo, executa `calcStationStatistics(pair.key, { month, year })` para cada par.
3. Cada cartão classifica a estação via `classify(odAvg, conamaLimit)` em `normal / atencao / alerta / critico / sem-dados` e calcula cobertura (% de dias do mês com leitura válida) e idade da última leitura.
4. A página renderiza a grade responsiva (1, 2 ou 3 colunas) com a pílula de status, o sparkline OD, a relação cobertura/última leitura e os atalhos "No mapa" e "Ver série histórica".
5. O Administrador clica no cartão e é levado a `/rio/<pair.key>`.

**Fluxos alternativos:**
- **A1 — Estação sem dados computados:** `calcStationStatistics` lança; o `catch` devolve `{ stats: null, status: 'sem-dados', freshness: { coveragePercent: 0 } }` e o cartão exibe traços ("—") nas métricas.

**Fluxos de exceção:** não se aplica — falhas individuais são absorvidas no `try/catch` por par.

---

## 4.9 Caso de uso "UC-09 Filtrar estações por bacia/classe"

**Descrição:** o filtro real entregue no MVP do EcoData ocorre dentro do mapa público (`/mapa`) por meio do `LayerControlPanel`, que permite ligar/desligar camadas de qualidade, quantidade, rios e sub-bacias, e indiretamente em `/admin/estacoes` por meio do cabeçalho que enumera "5 pares (qualidade + quantidade) na UGRHI 05". A combinação substitui o filtro tabular previsto na ata original.

**Atores:** Visitante (no mapa); Administrador (em `/admin/estacoes`).

**Pré-condições:** estar em `/mapa` ou `/admin/estacoes`.

**Fluxo principal:**
1. O Visitante abre `/mapa`.
2. O `MapPageClient` exibe `LayerControlPanel` no topo com toggles para camadas Qualidade, Quantidade, Rios e Sub-bacias.
3. O Visitante alterna um toggle; o estado local do client filtra `MapStation[]` por `station.type` e oculta/exibe os marcadores correspondentes no Leaflet.
4. A `MapLegend` é atualizada para refletir apenas as camadas ativas.
5. O Visitante combina filtros de tipo de estação e camadas geográficas até obter a visualização desejada.

**Fluxos alternativos:**
- **A1 — Filtro vazio:** quando todas as camadas de estações são desligadas, o mapa exibe apenas rios e sub-bacias, sem marcadores.

**Fluxos de exceção:** não se aplica — o filtro opera sobre dados já carregados pelo cliente.

---

## 4.10 Caso de uso "UC-10 Consultar página institucional / footer"

**Descrição:** o `Footer` global, presente em todas as páginas públicas, agrega navegação interna (Início, Panorama, Mapa, Relatórios, Glossário, Sobre), referências às fontes de dados (SIMQUA/CETESB, SSD PCJ, Sala de Situação PCJ), links institucionais (Sobre, Metodologia, Fontes) e recursos externos (CONAMA 357/2005, CETESB, Comitê PCJ, ANA). Conta ainda com um disclaimer destacando o status experimental do projeto e a atribuição "Dados: CETESB/SIMQUA, Agência PCJ/SSD PCJ, SSPCJ". O componente é animado via GSAP com `prefers-reduced-motion` respeitado.

**Atores:** Visitante.

**Pré-condições:** não se aplica.

**Fluxo principal:**
1. O Visitante rola até o final de qualquer página pública (`/`, `/mapa`, `/rio/[id]`, `/relatorios`, `/glossario`, `/sobre`, `/login`).
2. O GSAP `ScrollTrigger` dispara a animação `gsap.from(container, { opacity: 0, y: 40 })` quando o footer entra em 80% da viewport, exceto para usuários com `prefers-reduced-motion: reduce`.
3. O Visitante percorre as cinco colunas (Brand, Navegação, Dados, Institucional, Recursos) e o disclaimer de projeto experimental.
4. O Visitante clica em um link externo; o `FooterLink` adiciona `target="_blank"` e ícone `ExternalLink`, abrindo o destino em nova aba.
5. O Visitante clica em um link interno; o `Link` do Next executa client-side navigation para a rota correspondente.

**Fluxos alternativos:**
- **A1 — Usuário com motion reduzido:** `useReducedMotion` desativa partículas decorativas e o GSAP define `opacity: 1, y: 0` sem animação.

**Fluxos de exceção:** não se aplica — o footer é estático e não depende de dados remotos.

---

## 4.11 Caso de uso "UC-11 Autenticar-se"

**Descrição:** o Administrador autentica-se em `/login` informando e-mail e senha. A implementação é um mock didático (DECISIONS K-12) com duas credenciais hard-coded em `MOCK_CREDENTIALS`: `admin@pcj.sp.gov.br / admin123` (role `admin`) e `user@example.com / user123` (role `user`). Em caso de sucesso, o cookie httpOnly `ecodata_session` armazena `base64(JSON({email, role}))` por 7 dias.

**Atores:** Administrador.

**Pré-condições:** não estar autenticado (ou desejar trocar de usuário).

**Fluxo principal:**
1. O Administrador acessa `/login` (ou é redirecionado por `requireAdmin()`).
2. O `LoginForm` envia o submit via Server Action `loginAction(prevState, formData)`.
3. A ação normaliza `email` (trim + lower-case) e recupera `password`; se algum campo estiver vazio, retorna `{ error: 'Informe e-mail e senha.' }`.
4. A ação busca em `MOCK_CREDENTIALS` o par compatível; se não houver, retorna `{ error: 'Credenciais inválidas.' }`.
5. Em caso de match, a ação grava o cookie `ecodata_session` com `httpOnly`, `sameSite='lax'`, `secure` em produção, `path='/'` e `maxAge = 60*60*24*7`.
6. Se `match.role === 'admin'`, redireciona para `/admin/dashboard`; caso contrário, redireciona para `/`.

**Fluxos alternativos:**
- **A1 — Logout:** o Administrador dispara `logoutAction()`; o cookie é removido com `store.delete(SESSION_COOKIE)` e a sessão é encerrada com `redirect('/')`.

**Fluxos de exceção:**
- **E1 — Credenciais inválidas:** o `useFormState` recebe `{ error: '...' }` e o formulário renderiza a mensagem inline sem reload.
- **E2 — Sessão corrompida:** `decodeSession` falha ao decodificar base64/JSON ou ao validar o shape; `getSession()` retorna `null` e `requireAdmin()` redireciona o usuário para `/login`.

---

## 4.12 Caso de uso "UC-12 Visualizar dashboard administrativo"

**Descrição:** em `/admin/dashboard`, o Administrador recebe uma saudação contextual (`getGreeting()` por horário), três `MetricCard` (Boletins ativos, Última leitura, Estações ativas) com sparkline opcional e delta vs mês anterior, lista dos três boletins mais recentes com `LifecycleBadge` e ações inline, tabela de status CONAMA por estação (streamada via Suspense) e timeline lateral de atividade derivada dos mesmos boletins.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`.

**Fluxo principal:**
1. O Administrador chega em `/admin/dashboard` (após login ou pela navegação do `AdminShell`).
2. O servidor obtém `session = await getSession()` e deriva `displayName` do handle do e-mail.
3. Em paralelo, executa `loadDashboardMetrics`, `loadRecentBulletins`, `loadBulletinHistory` (últimos 7 meses) e `loadMeasurementHistory` (últimos 7 dias).
4. `deriveDelta` calcula a variação % do mês corrente vs anterior para a sparkline de boletins; falha silenciosa quando há menos de 2 pontos.
5. A página renderiza os `MetricCard`, a seção "Boletins recentes" com link para edição e o botão "Novo boletim".
6. A `StationsTable` é renderizada em paralelo via `Suspense`, executando 5 chamadas `calcStationStatistics` para popular a tabela com OD médio + conformidade.

**Fluxos alternativos:**
- **A1 — Banco offline:** cada loader registra `console.warn` e retorna o `fallback` com `null`; os `MetricCard` exibem "—" e os deltas são omitidos.
- **A2 — Onboarding ainda não completo:** o componente `OnboardingTracker step="view-dashboard"` registra a etapa atual; o `OnboardingChecklist` no layout permanece visível.

**Fluxos de exceção:**
- **E1 — Sessão expirada antes da renderização:** `requireAdmin()` no layout redireciona para `/login` antes que o `page.tsx` execute.

---

## 4.13 Caso de uso "UC-13 Monitorar pipeline de ingest"

**Descrição:** em `/admin/pipeline` o Administrador recebe a observabilidade completa dos três pipelines (`simqua`, `ssd_pcj`, `monthly_stats`). A página apresenta `PipelineDiagnostics` com achados textuais, três `CronStatusCard` com último run, contagens inseridos/pulados/erros, top três motivos de skip e contexto por estação, tabela "Saúde por estação" cruzando última leitura de chuva, vazão e OD com o `stationBreakdown` mais recente, cards "Preview mensal" do mês de referência e a timeline expansível das últimas 20 execuções lidas de `ingest_logs`.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; pelo menos uma execução registrada em `ingest_logs`.

**Fluxo principal:**
1. O Administrador acessa `/admin/pipeline`.
2. O servidor calcula `targetMonthId` (mês fechado anterior em UTC) e executa, em sequência segura: `getLatestRuns()` (1 SQL com `DISTINCT ON (pipeline)`), seguido em paralelo por `getRecentRuns(20)`, `getStationHealth(latestRuns)`, `getMonthlyPreview(targetMonthId)` e `getActivityCounters()`.
3. `deriveCardStatus` aplica regras de idade (`maxAgeMin` 120 para SIMQUA/SSD PCJ, 35 dias para monthly_stats) para definir o status agregado de cada cron.
4. A página renderiza o cabeçalho com `StatusPill` global, `LivePulse` (heartbeat do último run) e contadores 7d (execuções, parciais, erros).
5. `PipelineDiagnostics` é exibido só quando há achados; o detalhe "Como funciona o pipeline" é colapsável.
6. As seções "Status dos crons", "Saúde por estação", "Preview mensal", "Ações manuais" e "Histórico de execuções" são renderizadas em sequência, com a timeline mostrando linha expansível por execução.

**Fluxos alternativos:**
- **A1 — Mobile:** a tabela "Saúde por estação" colapsa para lista de cards (`md:hidden`) preservando os mesmos pills de status.
- **A2 — Sem dados em `monthly_stats`:** cada cartão do preview exibe o estado vazio com o convite "Sem dados computados — rode Recalcular abaixo."

**Fluxos de exceção:**
- **E1 — Erro de conexão Postgres:** o `try/catch` da página captura a mensagem e renderiza um banner vermelho "Erro de conexão com o banco" com o texto truncado; `globalStatus` é forçado para `'error'`.

---

## 4.14 Caso de uso "UC-14 Disparar ingest manual"

**Descrição:** ainda em `/admin/pipeline`, o Administrador usa o painel `ManualActions` para re-executar pipelines sob demanda. Quatro ações estão disponíveis: re-executar SIMQUA, re-executar SSD PCJ, recalcular `monthly_stats` para um mês específico (default = mês fechado anterior) e "Pipeline completo · pré-boletim", que encadeia SIMQUA → SSD PCJ → compute-month do mês de referência.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; rotas `/api/admin/pipeline/run-simqua`, `/api/admin/pipeline/run-ssd-pcj` e `/api/admin/pipeline/compute-month` operacionais.

**Fluxo principal:**
1. O Administrador clica em "Re-executar" em uma das `SimpleActionRow` (SIMQUA ou SSD PCJ).
2. O cliente atualiza `ActionFeedback` para `{ state: 'running' }` e renderiza o spinner + label "Executando…".
3. `postAction(endpoint)` envia `POST` JSON; o servidor responde com `{ inserted, skipped, errors, durationMs, status }`.
4. `formatFeedback` monta a string `"<n> inseridos · <n> pulados · <n> erros → status"` e a `FeedbackPill` é renderizada em verde.
5. `router.refresh()` é disparado dentro de `startTransition`, e a página recarrega timeline e cards com o novo run.

**Fluxos alternativos:**
- **A1 — Recalcular mês específico:** o Administrador escolhe `YYYY-MM` no select de `ComputeMonthRow`; o cliente valida `monthId.match(/^\d{4}-\d{2}$/)` antes do POST.
- **A2 — Pipeline completo:** `RunAllRow` executa as três rotas em série, atualizando `stepLabel` ("SIMQUA" → "SSD PCJ" → "compute-month") e exibindo o total acumulado de inserções.

**Fluxos de exceção:**
- **E1 — Resposta HTML do CDN (504):** `postAction` faz `res.text()` e tenta `JSON.parse`; em caso de falha, devolve `{ error: <texto truncado> }` e a pílula vira vermelha.
- **E2 — Erro no encadeamento:** em `RunAllRow`, qualquer passo que retorne `ok = false` lança e o `stepLabel` corrente é incluído na mensagem de erro.

---

## 4.15 Caso de uso "UC-15 Listar boletins gerados"

**Descrição:** em `/admin/boletins`, o Administrador vê todos os boletins não-excluídos da tabela `reports`, filtráveis por lifecycle (`draft / approved / published / archived`), visibilidade (`public / private`) e período (ano + mês). A lista é renderizada como tabela em desktop e como lista de cards empilhados em mobile, com `LifecycleBadge` e o componente cliente `BoletimRowActions` por linha (editar, baixar PDF, publicar, apagar).

**Atores:** Administrador.

**Pré-condições:** sessão `admin`.

**Fluxo principal:**
1. O Administrador acessa `/admin/boletins`.
2. O servidor lê `searchParams` via `parseFilters` e monta as condições Drizzle (`isNull(reports.deletedAt)` sempre presente).
3. `loadReports` faz duas queries em paralelo: a lista (top 100, ordenada por `periodStart desc, createdAt desc`) e o `count()` total.
4. A header exibe a contagem total e o botão "Novo boletim" (CTA para `/admin/boletim/preview`).
5. A barra de filtros mostra `FilterChip` para "Todos" + cada lifecycle, separador, e chips "Público"/"Privado".
6. O Administrador clica em um chip; o `Link` força nova navegação para `/admin/boletins?lifecycle=...&isPublic=...` e a página re-renderiza com o filtro aplicado.

**Fluxos alternativos:**
- **A1 — Nenhum boletim com os filtros atuais:** a tabela mostra "Nenhum boletim encontrado com esses filtros." + atalho "Criar novo boletim →".
- **A2 — Filtro por mês:** quando `year` e `month` estão definidos, o where adiciona `gte(periodStart, start) AND lt(periodStart, end)`; apenas `year` aplica intervalo anual.

**Fluxos de exceção:**
- **E1 — Falha no banco:** o `try/catch` retorna `{ rows: [], total: 0 }` e a página cai no estado vazio (cenário A1).

---

## 4.16 Caso de uso "UC-16 Gerar/regenerar boletim"

**Descrição:** em `/admin/boletim/preview`, o Administrador cria um novo rascunho de boletim para o mês/ano selecionado (template fixo `ssdpcj-oficial`) ou, com `?reportId=<n>`, carrega um existente para edição. A publicação é feita via `POST /api/reports/[id]/publish` com `{ regenerate: true }`, o que reprocessa o PDF, atualiza `pdfUrl` no Vercel Blob, marca `lifecycle = 'published'` e exibe o boletim em `/relatorios`.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; existir dados em `monthly_stats` para o mês alvo (a rota `/api/reports/preview-data` falha caso contrário).

**Fluxo principal:**
1. O Administrador abre `/admin/boletim/preview` sem `reportId`.
2. O `CreateDraftForm` apresenta selects de mês (default = mês fechado anterior) e ano.
3. O Administrador clica "Criar rascunho"; o cliente faz `POST /api/reports` com `{ month, year, templateSlug: 'ssdpcj-oficial' }`.
4. Em caso de sucesso, `markComplete('create-bulletin')` é chamado e o router navega para `/admin/boletim/preview?reportId=<id>`.
5. O `EditDraftEditor` carrega `/api/reports/<id>` e `/api/reports/preview-data?month=&year=` em paralelo.
6. O Administrador clica "Publicar"; o `AlertDialog` confirma a ação e dispara `POST /api/reports/<id>/publish` com `{ regenerate: true }`; ao retornar `data.report`, o estado é atualizado, o banner "Publicado com sucesso. Disponível em /relatorios." aparece e `markComplete('publish-bulletin')` é registrado.

**Fluxos alternativos:**
- **A1 — Regenerar sobre boletim já publicado:** o mesmo endpoint aceita `regenerate: true` em boletins `published`, substituindo o `pdfUrl` no blob e bumpando `updatedAt` sem alterar `publishedBy`.

**Fluxos de exceção:**
- **E1 — `/api/reports` falha:** o `catch` em `createDraft` define `err` e exibe banner vermelho; `creating` volta para `false` e o botão fica clicável.
- **E2 — Publish falha:** `publish()` captura erro e renderiza banner vermelho com `err.message`; o lifecycle local permanece como `draft`.

---

## 4.17 Caso de uso "UC-17 Editar template do boletim"

**Descrição:** ainda em `/admin/boletim/preview?reportId=<n>`, o Administrador edita o conteúdo editorial do boletim: observações do analista, notas por estação e visibilidade pública. Todas as alterações são enviadas via `PATCH /api/reports/<id>` com debounce de 1.5s; o badge `SaveStatusBadge` indica `pending / saved / error`. O preview PDF à direita é regenerado on-demand com `POST /api/reports/preview-pdf`, exibindo um blob URL no iframe. A edição estrutural de templates (layout JSON) acontece em rota separada (`/admin/templates/[id]/edit-v2`).

**Atores:** Administrador.

**Pré-condições:** existir um rascunho válido com `reportId`; sessão `admin`.

**Fluxo principal:**
1. O Administrador altera o texto em "Observações do analista" ou nos campos por estação, ou alterna o toggle "Visibilidade pública".
2. `queueSave(payload)` define `dirtyRef = true`, `saving = 'pending'` e reinicia o timer de 1.5s.
3. Ao expirar, o cliente envia `PATCH /api/reports/<id>` com o payload diff.
4. Em sucesso, `dirtyRef = false`, `saving = 'saved'`; após 2s sem novas edições, volta a `'idle'`.
5. O Administrador clica em `RefreshCw` "Atualizar preview"; `refreshPdf()` faz `POST /api/reports/preview-pdf` com `{ month, year, analystNotes, stationNotes }`, recebe o blob, revoga o objectURL anterior (se for blob, não persistido) e atualiza o iframe.

**Fluxos alternativos:**
- **A1 — Boletim já publicado:** quando `report.lifecycle === 'published'` e há `pdfUrl`, o preview carrega direto do blob persistido sem regenerar; `pdfIsPersisted = true` impede `URL.revokeObjectURL`.
- **A2 — Edição estrutural de layout:** o Administrador navega para `/admin/templates/<id>/edit-v2` e altera o JSON do template via `editor-shell.tsx`; o boletim continua referenciando o `templateVersionId` salvo.

**Fluxos de exceção:**
- **E1 — `PATCH /api/reports/<id>` falha:** `saving = 'error'` e o banner exibe `err.message`.
- **E2 — Geração de preview falha:** `refreshPdf` captura o erro e exibe banner "Erro ao gerar preview"; o iframe mantém o blob anterior.

---

## 4.18 Caso de uso "UC-18 Apagar boletim"

**Descrição:** o Administrador exclui (soft-delete) um boletim a partir de duas superfícies. Em `/admin/boletins`, cada linha tem o botão `Trash2` do `BoletimRowActions` que dispara `DELETE /api/reports/<id>`. Em `/relatorios`, quando a sessão é `admin`, o `DeleteBulletinButton` aparece junto ao download. A operação marca `deletedAt = now()` na tabela `reports` e remove o boletim das listagens públicas; pode ser restaurada por SQL direto.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; boletim existente e ainda não excluído (`deletedAt IS NULL`).

**Fluxo principal:**
1. O Administrador clica no botão `Trash2` da linha em `/admin/boletins`.
2. O `BoletimRowActions.softDelete` exibe `confirm('Apagar (soft delete) este boletim? Pode ser restaurado via SQL.')`.
3. Em caso de confirmação, define `busy = 'delete'` e envia `DELETE /api/reports/<id>`.
4. A rota responde com `{ ok: true }`; o cliente dispara `router.refresh()` e a linha some da tabela.
5. Em caso de fluxo via `/relatorios`, `DeleteBulletinButton.handleDelete` confirma com `confirm('Apagar este boletim?')`, faz o mesmo DELETE e dispara `window.location.reload()`.

**Fluxos alternativos:**
- **A1 — Cancelar diálogo nativo:** o `confirm()` devolve `false` e nada é executado; a UI permanece intacta.

**Fluxos de exceção:**
- **E1 — `DELETE` retorna erro:** o `catch` mostra `alert(data.error || 'Erro ao apagar')` em `/admin/boletins` ou `alert(data.error || 'Erro ao apagar')` em `/relatorios`; o spinner é encerrado.
- **E2 — Erro de rede:** o `catch` exibe `alert('Erro de rede')` (rota pública) ou a mensagem genérica do `Error` (rota admin).

---

## 4.19 Caso de uso "UC-19 Visualizar atividade administrativa"

**Descrição:** o mesmo `/admin/atividade` descrito em UC-04 também serve à perspectiva administrativa: o Administrador filtra por "Boletins" para auditar publicações recentes (com `publishedBy` e timestamps), ou por "Alertas" para inspecionar grupos por parâmetro/severidade que vieram dos pipelines. Os filtros são compartilháveis por URL (`?tipo=alertas`) e o heatmap mensal fornece visão agregada de severidade × densidade.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`.

**Fluxo principal:**
1. O Administrador acessa `/admin/atividade?tipo=boletins`.
2. `loadEvents('boletins')` consulta apenas a tabela `reports`, agregando eventos `bulletin-created` e `bulletin-published` por boletim.
3. A timeline exibe cada evento com chip "Boletim" ou "Publicação", título no formato "Boletim <Mês/Ano> criado/publicado" e ator (`publishedBy ?? 'admin'`).
4. O Administrador clica no chip "Alertas" da barra segmentada; a página recarrega com `?tipo=alertas` e exibe apenas `AlertGroupEvent`.
5. O Administrador hover/foca em uma célula do heatmap mensal; o `aria-label` revela "dia X: N eventos, severidade Y".

**Fluxos alternativos:**
- **A1 — Sem alertas ativos:** o filtro "Alertas" retorna lista vazia e cai no estado "Nenhuma atividade ainda."

**Fluxos de exceção:**
- **E1 — Falha no banco:** mesmo comportamento de UC-04 (E1) — `console.error` + retorno `[]` + estado vazio.

---

## 4.20 Caso de uso "UC-20 Visualizar onboarding/checklist"

**Descrição:** no primeiro acesso ao painel administrativo, o Administrador recebe o `WelcomeModal` — diálogo multi-step (4 slides: Hero / Como funciona / Primeiros passos / CTA) com indicators, navegação Back/Next e Skip. Após o welcome, o `OnboardingChecklist` permanece ancorado em `bottom-right` mostrando o progresso dos cinco passos (`view-dashboard`, `create-bulletin`, `view-boletins`, `publish-bulletin`, etc.) registrados via `OnboardingTracker step="..."` em cada página. Ao completar 5/5, o componente dispara confete via `canvas-confetti` e se auto-encerra após ~2.5s.

**Atores:** Administrador.

**Pré-condições:** sessão `admin`; estado `welcomeSeen === false && dismissedAt === null` em `useOnboarding()`.

**Fluxo principal:**
1. O Administrador faz login pela primeira vez e o `AdminLayout` renderiza `<WelcomeModal />` e `<OnboardingChecklist />` ao lado dos filhos da rota.
2. O `WelcomeModal` avança pelos quatro slides (`Hero`, `Flow`, `Checklist`, `CTA`) com `ArrowRight`/`ArrowLeft`; ao final clica em "Vamos lá" e `markWelcomeSeen()` é registrado.
3. O modal fecha; o `OnboardingChecklist` aparece com `welcomeSeen === true && dismissedAt === null && progress < 5`.
4. O Administrador navega pelo admin; cada `OnboardingTracker step="..."` no topo da página marca a etapa correspondente como concluída.
5. Ao atingir 5/5, `isComplete === true` dispara duas chamadas de `canvas-confetti`, ativa `ring-2 ring-status-healthy/60` no card e agenda `dismiss()` após 3.0s (`CELEBRATION_DELAY_MS + 500`).

**Fluxos alternativos:**
- **A1 — Pular no welcome:** o Administrador clica em "Skip"; `dismiss()` define `dismissedAt = now()` e nem o welcome nem o checklist voltam a aparecer (até reativação manual via Configurações).
- **A2 — Colapsar o checklist:** o Administrador clica em `ChevronDown`; `collapsed = true` reduz a largura do card sem alterar o estado de progresso.

**Fluxos de exceção:**
- **E1 — `canvas-confetti` indisponível:** o `import('canvas-confetti')` rejeita; o `try/catch` ignora silenciosamente (comentário "confetti é opcional") e o auto-dismiss segue normalmente.

---

## Referências cruzadas

- [`03-casos-de-uso-diagramas.md`](./03-casos-de-uso-diagramas.md) — Diagramas Mermaid
- [`05-diagramas-sequencia.md`](./05-diagramas-sequencia.md) — Sequências
- [`08-casos-de-teste.md`](./08-casos-de-teste.md) — Testes alinhados
- [`09-telas-da-solucao/`](./09-telas-da-solucao/) — Telas alinhadas

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: section-04-use-case-descriptions
total_use_cases: 20
alignment: §5.N, §8.N, §9.N share numbering with §4.N
canonical_inventory: AGENT-CONTEXT.md
actors:
  - id: Ator-01
    name: Visitante / Cidadão
    scope: UC-01..UC-10 + leitura pública de UC-06/UC-07/UC-18
  - id: Ator-02
    name: Administrador
    scope: UC-04, UC-08, UC-09 (parcial), UC-11..UC-20
notes:
  - UC-04 e UC-08 estão entregues apenas na superfície admin no MVP; versões públicas ficam como roadmap (V2)
  - UC-09 cobre o filtro real entregue (toggles de camada no /mapa); o filtro tabular previsto na ata original não foi implementado
  - UC-11 é mock (DECISIONS K-12) — credenciais hard-coded em src/app/login/actions.ts
  - UC-18 expõe duas superfícies (admin/boletins e /relatorios com sessão admin)
```
<!-- /agent-extractable -->
