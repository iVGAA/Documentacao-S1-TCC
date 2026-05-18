---
file: 08-casos-de-teste.md
section: 8 — Casos de teste
purpose: Casos de teste manuais (propósito, pré-condições, entradas, saída esperada) para os 20 casos de uso do EcoData
depends_on:
  - 04-casos-de-uso-descricoes.md
  - 05-diagramas-sequencia.md
  - 07-banco-de-dados.md
last_updated: 2026-05-18
status: draft
---

# 8. Casos de teste

> **Para agentes de IA**: esta seção contém **20 sub-seções** (uma por
> caso de uso, numeradas de §8.1 a §8.20 em correspondência direta com
> UC-01..UC-20). Cada sub-seção tem entre **2 e 3 casos de teste**, no
> formato `Caso de Teste N.M`, com quatro campos fixos: **Propósito**,
> **Pré-condições**, **Lista de Entradas** (tabela markdown) e **Saída
> Aguardada** (lista). O alinhamento numérico é estável — §8.5 testa
> UC-05, §8.16 testa UC-16, etc. Os testes cobrem caminho feliz, falha
> esperada e, quando aplicável, edge case relevante. Use o bloco YAML
> final para indexar a estrutura.

Esta seção descreve os roteiros de teste manual aplicados ao EcoData
durante o desenvolvimento e antes de cada release. O escopo é
funcional: cobre o que o sistema deve fazer da perspectiva do usuário
final (Visitante e Administrador), não cobre testes unitários de
componentes nem testes de carga. Cada caso de teste é nomeado pelo
identificador do caso de uso que valida (UC-XX) e segue a estrutura
padrão de propósito, pré-condições, lista de entradas e saída
aguardada.

---

## 8.1 Teste "UC-01 Visualizar mapa em tempo real"

### Caso de Teste 1.1

- **Propósito:** garantir que o mapa inicial carrega com a camada base
  do OpenStreetMap e os marcadores das estações monitoradas.
- **Pré-condições:** sistema implantado; tabela `stations` populada
  com pelo menos uma estação ativa por bacia (Piracicaba, Capivari,
  Jundiaí); tabela `monthly_stats` com pelo menos um par de meses
  recentes para o cálculo do IQA exibido nos popups.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/` (ou `/mapa`)                            |
| Sessão                    | anônima (sem cookie `ecodata_session`)      |
| Navegador                 | Chrome/Firefox em viewport 1440×900         |

- **Saída Aguardada:**
  - Mapa Leaflet centralizado na Bacia PCJ é renderizado em até 3 s.
  - Camada base do OpenStreetMap aparece com atribuição visível.
  - Marcadores das estações são exibidos em cores classificadas por
    estado de OD (verde para conforme, amarelo para atenção, vermelho
    para inconforme).
  - Controles de zoom `+/-` estão funcionais.

### Caso de Teste 1.2

- **Propósito:** validar comportamento do mapa quando não há dados
  recentes de qualidade.
- **Pré-condições:** estação cadastrada em `stations`, mas sem
  registros em `water_quality_readings` nos últimos 30 dias.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/mapa`                                     |
| Estação alvo              | uma estação sem dados recentes              |

- **Saída Aguardada:**
  - Marcador da estação é exibido em cor neutra (cinza) indicando
    ausência de dado recente.
  - Tooltip ou legenda do mapa identifica visualmente o estado "sem
    dados".

---

## 8.2 Teste "UC-02 Visualizar popup de estação no mapa"

### Caso de Teste 2.1

- **Propósito:** confirmar que o clique em um marcador exibe popup
  com OD, pH, turbidez e link para detalhes do rio.
- **Pré-condições:** estação `BAR-01` (Rio Atibaia, Itatiba) com
  leitura recente em `water_quality_readings`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/mapa`                                     |
| Ação                      | clique sobre o marcador `BAR-01`            |

- **Saída Aguardada:**
  - Popup é aberto sobre o marcador.
  - Cabeçalho do popup mostra nome da estação e nome do rio.
  - Métricas exibidas: OD (mg/L), pH, turbidez (UNT), com timestamp
    da última leitura.
  - Link "Ver detalhes do rio" navega para `/rio/[id]`.

### Caso de Teste 2.2

- **Propósito:** verificar fechamento do popup ao clicar fora ou em
  outro marcador.
- **Pré-condições:** ao menos duas estações com leituras recentes.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/mapa`                                     |
| Ação 1                    | abrir popup da estação A                    |
| Ação 2                    | clicar em estação B                         |

- **Saída Aguardada:**
  - Popup da estação A é fechado automaticamente.
  - Popup da estação B é aberto com as métricas correspondentes.
  - Não há sobreposição de dois popups simultâneos.

---

## 8.3 Teste "UC-03 Ver detalhes históricos de rio"

### Caso de Teste 3.1

- **Propósito:** validar exibição da página de detalhes de rio com
  gráfico histórico e tabela de estações associadas.
- **Pré-condições:** rio `piracicaba` em `rivers`; ao menos uma
  estação associada; `monthly_stats` com 6+ meses de dados de OD.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/rio/piracicaba`                           |
| Período                   | últimos 12 meses (default)                  |

- **Saída Aguardada:**
  - Página renderiza nome do rio, classe CONAMA e Q7,10 no header.
  - Gráfico histórico de OD é desenhado com pelo menos 6 pontos.
  - Tabela de estações associadas mostra `name`, `municipality`,
    `lastReadingAt`.
  - Cards de IQA e OD agregados aparecem com valores numéricos.

### Caso de Teste 3.2

- **Propósito:** garantir resposta correta para rio inexistente.
- **Pré-condições:** chave `inexistente` ausente em `rivers`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/rio/inexistente`                          |

- **Saída Aguardada:**
  - Página retorna HTTP 404 com mensagem "Rio não encontrado".
  - Link de retorno para `/mapa` é exibido.

### Caso de Teste 3.3

- **Propósito:** confirmar comportamento com rio cadastrado mas sem
  histórico em `monthly_stats`.
- **Pré-condições:** rio cadastrado; nenhum registro em
  `monthly_stats` para suas estações.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/rio/<key-sem-dados>`                      |

- **Saída Aguardada:**
  - Página renderiza header normalmente.
  - Área do gráfico exibe estado vazio com mensagem "Sem dados
    históricos para o período selecionado".

---

## 8.4 Teste "UC-04 Ver atividade recente do sistema"

### Caso de Teste 4.1

- **Propósito:** validar que a página `/admin/atividade` lista eventos
  recentes do sistema (ingestão, geração de boletim, publicação) em
  ordem cronológica decrescente.
- **Pré-condições:** `ingest_logs` com pelo menos 5 entradas dos
  últimos 7 dias; `reports` com pelo menos 1 boletim publicado.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/atividade`                                |
| Sessão                    | anônima                                     |

- **Saída Aguardada:**
  - Lista cronológica reversa de eventos é exibida.
  - Cada item mostra tipo de evento, fonte (SIMQUA, SSD PCJ), status
    (`ok`, `partial`, `error`) e timestamp formatado em pt-BR.
  - Eventos de boletim mostram link para `/relatorios`.

### Caso de Teste 4.2

- **Propósito:** verificar estado vazio quando o sistema acabou de
  ser implantado.
- **Pré-condições:** `ingest_logs` vazia; `reports` vazia.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/atividade`                                |

- **Saída Aguardada:**
  - Página renderiza header e mensagem "Nenhuma atividade registrada
    ainda".
  - Link explicativo para `/sobre` é apresentado.

---

## 8.5 Teste "UC-05 Buscar termo no glossário"

### Caso de Teste 5.1

- **Propósito:** validar que a busca textual destaca termos cujo
  título ou descrição contenha a query.
- **Pré-condições:** glossário com ao menos 10 termos cadastrados
  incluindo "IQA", "OD" e "CONAMA".
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/glossario`                                |
| Campo busca               | `IQA`                                       |

- **Saída Aguardada:**
  - Cards filtrados mostram apenas termos com correspondência (IQA e
    eventuais variações como "Índice de Qualidade da Água").
  - Contador de resultados é atualizado.
  - URL recebe parâmetro `?q=IQA` para compartilhamento.

### Caso de Teste 5.2

- **Propósito:** confirmar copiar link de termo via âncora.
- **Pré-condições:** termo `oxigenio-dissolvido` cadastrado.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/glossario`                                |
| Ação                      | clicar no ícone "copiar link" do termo OD   |

- **Saída Aguardada:**
  - Clipboard recebe URL `…/glossario#oxigenio-dissolvido`.
  - Toast de confirmação aparece por 2 s.
  - Recarregar a URL leva direto à âncora correspondente.

### Caso de Teste 5.3

- **Propósito:** validar feedback para busca sem resultados.
- **Pré-condições:** glossário populado.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Campo busca               | `zxzxzx`                                    |

- **Saída Aguardada:**
  - Lista de cards é esvaziada.
  - Estado vazio mostra mensagem "Nenhum termo encontrado para
    'zxzxzx'" e sugere limpar o filtro.

---

## 8.6 Teste "UC-06 Listar boletins disponíveis"

### Caso de Teste 6.1

- **Propósito:** confirmar listagem pública de boletins publicados
  com `isPublic = true`.
- **Pré-condições:** `reports` com pelo menos 2 registros publicados
  (`lifecycle = 'published'`, `isPublic = true`, `deletedAt = null`).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/relatorios`                               |
| Sessão                    | anônima                                     |

- **Saída Aguardada:**
  - Página lista boletins em ordem cronológica decrescente por
    `periodStart`.
  - Cada card mostra mês/ano de referência, data de publicação,
    link "Baixar PDF" e badge "Publicado".
  - Boletins com `isPublic = false` ou `lifecycle != 'published'`
    não aparecem.

### Caso de Teste 6.2

- **Propósito:** validar exclusão lógica em `/relatorios`.
- **Pré-condições:** boletim com `deletedAt != null`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/relatorios`                               |

- **Saída Aguardada:**
  - Boletim soft-deletado **não** aparece na listagem pública.
  - Demais boletins permanecem visíveis.

---

## 8.7 Teste "UC-07 Baixar boletim PDF"

### Caso de Teste 7.1

- **Propósito:** validar download de PDF gerado e armazenado em
  Vercel Blob.
- **Pré-condições:** boletim com `pdfUrl` válido apontando para
  Vercel Blob; arquivo acessível publicamente.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/relatorios`                               |
| Ação                      | clicar em "Baixar PDF" de um boletim        |

- **Saída Aguardada:**
  - Navegador abre o PDF em nova aba (ou inicia download).
  - PDF tem ao menos uma página renderizada com cabeçalho, sumário
    e tabela de estações.
  - Resposta HTTP é 200 com `content-type: application/pdf`.

### Caso de Teste 7.2

- **Propósito:** verificar resiliência quando o blob foi removido
  externamente.
- **Pré-condições:** registro de boletim com `pdfUrl` apontando para
  URL inexistente (404).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Ação                      | clicar em "Baixar PDF" do boletim quebrado  |

- **Saída Aguardada:**
  - Browser exibe erro 404 do Vercel Blob.
  - Card do boletim sinaliza visualmente "PDF indisponível" na
    próxima visita à página (após detecção pelo sistema).

---

## 8.8 Teste "UC-08 Listar estações monitoradas"

### Caso de Teste 8.1

- **Propósito:** validar listagem pública das estações com nome,
  rio, município e bacia.
- **Pré-condições:** tabela `stations` com pelo menos 5 estações
  ativas distribuídas entre as 3 bacias.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/estacoes`                                 |

- **Saída Aguardada:**
  - Tabela ou cards exibem `name`, `river`, `municipality`,
    `waterClass`, `lastReadingAt`.
  - Ordenação default é por bacia e nome.
  - Contador total de estações é visível.

### Caso de Teste 8.2

- **Propósito:** verificar a coluna de última leitura para estações
  defasadas.
- **Pré-condições:** estação sem leitura há mais de 7 dias.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/estacoes`                                 |

- **Saída Aguardada:**
  - Coluna "Última leitura" exibe data formatada e badge "Defasada"
    quando o intervalo for superior a 7 dias.
  - Ordenação por essa coluna agrupa estações defasadas no topo
    quando ascendente.

---

## 8.9 Teste "UC-09 Filtrar estações por bacia/classe"

### Caso de Teste 9.1

- **Propósito:** validar filtro por bacia hidrográfica.
- **Pré-condições:** estações distribuídas entre Piracicaba (3),
  Capivari (2) e Jundiaí (2).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/estacoes`                                 |
| Filtro Bacia              | `Piracicaba`                                |

- **Saída Aguardada:**
  - Listagem exibe apenas as 3 estações da bacia Piracicaba.
  - URL recebe parâmetro `?bacia=piracicaba`.
  - Contador é recalculado para "3 de 7 estações".

### Caso de Teste 9.2

- **Propósito:** validar combinação de filtros bacia + classe
  CONAMA.
- **Pré-condições:** estações com `waterClass` distintos (2, 3, 4).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Filtro Bacia              | `Piracicaba`                                |
| Filtro Classe             | `2`                                         |

- **Saída Aguardada:**
  - Listagem exibe apenas estações da bacia Piracicaba com
    `waterClass = 2`.
  - URL recebe ambos parâmetros: `?bacia=piracicaba&classe=2`.

### Caso de Teste 9.3

- **Propósito:** garantir reset dos filtros via botão "Limpar".
- **Pré-condições:** filtros aplicados conforme caso 9.2.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Ação                      | clicar em "Limpar filtros"                  |

- **Saída Aguardada:**
  - Todos os parâmetros são removidos da URL.
  - Listagem volta a exibir todas as estações ativas.

---

## 8.10 Teste "UC-10 Consultar página institucional / footer"

### Caso de Teste 10.1

- **Propósito:** validar conteúdo institucional na página `/sobre`.
- **Pré-condições:** página `sobre` em produção.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/sobre`                                    |

- **Saída Aguardada:**
  - Página renderiza seções: hipótese, fontes de dados, créditos
    institucionais.
  - Cards de fontes (SIMQUA, SSD PCJ, SPAguas) mostram logos e
    links externos com `target="_blank"`.
  - Disclaimer "operação experimental, sem vínculo institucional
    formal" é visível.

### Caso de Teste 10.2

- **Propósito:** verificar presença de links navegáveis no footer
  em todas as páginas públicas.
- **Pré-condições:** footer global ativo no `RootLayout`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/`, `/glossario`, `/relatorios`            |

- **Saída Aguardada:**
  - Footer aparece em todas as páginas.
  - Links para `/sobre`, `/glossario`, `/relatorios` e GitHub do
    projeto estão presentes e funcionais.

---

## 8.11 Teste "UC-11 Autenticar-se"

### Caso de Teste 11.1

- **Propósito:** validar login bem-sucedido como administrador.
- **Pré-condições:** sistema implantado com credenciais mock
  configuradas; cookie `ecodata_session` ausente.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/login`                                    |
| E-mail                    | `admin@pcj.sp.gov.br`                       |
| Senha                     | `admin123`                                  |
| Ação                      | submeter formulário                         |

- **Saída Aguardada:**
  - Servidor retorna `Set-Cookie: ecodata_session=<base64>` com
    flags `httpOnly`, `sameSite=lax`, `maxAge=7d`.
  - Redirecionamento para `/admin/dashboard`.
  - Cabeçalho administrativo mostra e-mail logado.

### Caso de Teste 11.2

- **Propósito:** verificar mensagem de erro para senha incorreta.
- **Pré-condições:** e-mail `admin@pcj.sp.gov.br` cadastrado nas
  credenciais mock.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| E-mail                    | `admin@pcj.sp.gov.br`                       |
| Senha                     | `senhaErrada123`                            |

- **Saída Aguardada:**
  - Formulário exibe alerta acessível (`role="alert"`) com a
    mensagem "Credenciais inválidas.".
  - Nenhum cookie `ecodata_session` é definido.
  - Usuário permanece em `/login`.

### Caso de Teste 11.3

- **Propósito:** confirmar rejeição de e-mail não cadastrado.
- **Pré-condições:** sistema implantado.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| E-mail                    | `naoexiste@pcj.sp.gov.br`                   |
| Senha                     | `qualquer123`                               |

- **Saída Aguardada:**
  - Mesma mensagem genérica "Credenciais inválidas." (sem
    revelar se o e-mail existe).
  - Nenhum cookie é definido.
  - Tentativa não cria registro em nenhuma tabela.

---

## 8.12 Teste "UC-12 Visualizar dashboard administrativo"

### Caso de Teste 12.1

- **Propósito:** validar carregamento do dashboard com KPIs e
  sparklines.
- **Pré-condições:** logado como admin; `monthly_stats` populada;
  `reports` com pelo menos 2 boletins publicados.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/dashboard`                          |
| Sessão                    | cookie de admin válido                      |

- **Saída Aguardada:**
  - KPIs renderizam (estações ativas, boletins publicados,
    ingestões nos últimos 7 dias).
  - Sparklines mostram série temporal recente para cada KPI.
  - Cards de pipeline, boletim e estações têm links navegáveis.

### Caso de Teste 12.2

- **Propósito:** verificar redirecionamento para `/login` quando
  usuário não autenticado tenta acessar o dashboard.
- **Pré-condições:** sem cookie `ecodata_session`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/dashboard`                          |
| Sessão                    | nenhuma                                     |

- **Saída Aguardada:**
  - Servidor responde 307 com `Location: /login`.
  - Página renderizada é a de login, não o dashboard.

---

## 8.13 Teste "UC-13 Monitorar pipeline de ingest"

### Caso de Teste 13.1

- **Propósito:** validar listagem de execuções recentes do pipeline
  com status e timestamps.
- **Pré-condições:** logado como admin; `ingest_logs` com pelo
  menos 1 entrada dos últimos 7 dias para cada fonte (SIMQUA,
  SSD PCJ).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/pipeline`                           |
| Filtro Período            | `Últimos 7 dias` (default)                  |

- **Saída Aguardada:**
  - Lista mostra cada ingestão com fonte, status (`ok`, `partial`,
    `error`), `inserted`, `skipped`, `errors`, `durationMs` e
    timestamp em pt-BR.
  - Live pulse no topo indica a próxima janela de cron prevista.
  - Cards de diagnóstico exibem cobertura percentual por fonte.

### Caso de Teste 13.2

- **Propósito:** verificar comportamento quando não há execuções
  no período filtrado.
- **Pré-condições:** logado como admin; `ingest_logs` sem entradas
  nos últimos 7 dias.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/pipeline`                           |

- **Saída Aguardada:**
  - Lista exibe estado vazio com mensagem "Nenhuma execução nos
    últimos 7 dias".
  - Botão "Re-executar SIMQUA" continua disponível para
    intervenção manual.

---

## 8.14 Teste "UC-14 Disparar ingest manual"

### Caso de Teste 14.1

- **Propósito:** validar disparo manual de re-execução do ingest
  SIMQUA com sucesso.
- **Pré-condições:** logado como admin; CETESB SIMQUA acessível
  publicamente; `stations` populada.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/pipeline`                           |
| Ação                      | clicar em "Re-executar SIMQUA"              |

- **Saída Aguardada:**
  - Botão entra em estado "Executando…" com spinner.
  - Após retorno, painel mostra feedback `N inseridos · M pulados
    → ok` com duração em ms.
  - Novo registro aparece em `ingest_logs` com `source = 'simqua'`,
    `triggeredBy = 'manual'` e `triggeredByEmail` igual ao e-mail
    da sessão.
  - Lista de execuções é atualizada via `router.refresh()`.

### Caso de Teste 14.2

- **Propósito:** validar disparo de recálculo mensal
  (`compute-month`) com seleção de mês fechado.
- **Pré-condições:** logado como admin; `water_quality_readings` e
  `flow_readings` com dados para o mês selecionado.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Seletor Mês               | `2026-04` (mês anterior)                    |
| Ação                      | clicar em "Calcular monthly_stats"          |

- **Saída Aguardada:**
  - Painel exibe `N pares calculados · M pulados`.
  - Tabela `monthly_stats` ganha (ou atualiza por upsert) registros
    para o par `(station, month)` informado.
  - Novo registro em `ingest_logs` com `source = 'compute-month'`.

### Caso de Teste 14.3

- **Propósito:** validar rejeição de input inválido no seletor
  de mês.
- **Pré-condições:** logado como admin.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Seletor Mês               | `2026-13` (mês fora do range)               |

- **Saída Aguardada:**
  - Feedback exibe "Formato inválido — use YYYY-MM".
  - Nenhuma chamada POST é realizada.
  - `ingest_logs` permanece inalterado.

---

## 8.15 Teste "UC-15 Listar boletins gerados"

### Caso de Teste 15.1

- **Propósito:** validar listagem completa de boletins (todos os
  lifecycles) na área admin.
- **Pré-condições:** logado como admin; `reports` com boletins em
  `draft`, `approved` e `published`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/boletins`                           |

- **Saída Aguardada:**
  - Tabela lista boletins de todos os lifecycles, com badges
    coloridos por estado.
  - Colunas exibem período, lifecycle, isPublic, pdfUrl e ações.
  - Ações disponíveis por linha: editar, baixar PDF (se existir),
    publicar (se não publicado) e apagar.

### Caso de Teste 15.2

- **Propósito:** verificar filtro por lifecycle.
- **Pré-condições:** boletins em pelo menos 3 lifecycles distintos.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/boletins?lifecycle=draft`           |

- **Saída Aguardada:**
  - Tabela exibe apenas boletins com `lifecycle = 'draft'`.
  - Contagem total é atualizada.
  - Boletins `deletedAt != null` continuam ocultos.

---

## 8.16 Teste "UC-16 Gerar/regenerar boletim"

### Caso de Teste 16.1

- **Propósito:** validar criação de um novo rascunho de boletim
  para um mês fechado.
- **Pré-condições:** logado como admin; estações com dados
  agregados em `monthly_stats` para o mês anterior; template
  `ssdpcj-oficial` semeado com pelo menos uma `templateVersion`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/boletim/preview`                    |
| Mês                       | `4` (Abril)                                 |
| Ano                       | `2026`                                      |
| Ação                      | clicar em "Criar rascunho"                  |

- **Saída Aguardada:**
  - POST `/api/reports` responde 201 com `report.id`.
  - Novo registro em `reports` com `lifecycle = 'draft'`,
    `status = 'pending'`, `isPublic = false`, `periodStart` =
    primeiro dia do mês, `periodEnd` = último dia do mês.
  - Redirecionamento para `/admin/boletim/preview?reportId=<id>`.
  - Banner experimental "Recurso em construção" visível no topo.

### Caso de Teste 16.2

- **Propósito:** validar geração do PDF e publicação do boletim.
- **Pré-condições:** rascunho criado conforme caso 16.1; analyst
  notes preenchidas.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/boletim/preview?reportId=<id>`      |
| Analyst notes             | "Mês marcado por chuvas acima da média."    |
| Toggle isPublic           | ativado                                     |
| Ação                      | clicar em "Publicar"                        |

- **Saída Aguardada:**
  - POST `/api/reports/<id>/publish` responde 200.
  - `reports.lifecycle` muda para `published`, `publishedAt` é
    preenchido, `publishedBy` recebe e-mail do admin.
  - `pdfUrl` recebe URL válida do Vercel Blob (`https://*.public.blob.vercel-storage.com/…`).
  - Iframe à direita atualiza para o PDF final.
  - Boletim aparece em `/relatorios` no fluxo público.

### Caso de Teste 16.3

- **Propósito:** validar rejeição de mês inválido na criação.
- **Pré-condições:** logado como admin.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Mês                       | `13`                                        |
| Ano                       | `2026`                                      |

- **Saída Aguardada:**
  - API retorna HTTP 400 com `{ error: "Mês inválido: 13" }`.
  - Nenhum registro é inserido em `reports`.
  - Formulário exibe a mensagem de erro ao usuário.

---

## 8.17 Teste "UC-17 Editar template do boletim"

### Caso de Teste 17.1

- **Propósito:** validar criação de nova versão de template a
  partir da última versão existente.
- **Pré-condições:** logado como admin; template `ssdpcj-oficial`
  com pelo menos uma versão semeada.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/templates/<id>`                     |
| Ação                      | editar título da capa e clicar em "Salvar"  |

- **Saída Aguardada:**
  - Novo registro em `template_versions` com `version` incrementado.
  - `templates.currentVersionId` aponta para a nova versão.
  - Boletins existentes não são alterados (mantêm
    `templateVersionId` original).

### Caso de Teste 17.2

- **Propósito:** verificar pré-visualização do template antes de
  salvar.
- **Pré-condições:** logado como admin; em modo de edição.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Ação                      | clicar em "Pré-visualizar"                  |

- **Saída Aguardada:**
  - Iframe renderiza o PDF com as alterações em memória.
  - Nenhuma escrita em `template_versions` ocorre até o "Salvar".

---

## 8.18 Teste "UC-18 Apagar boletim"

### Caso de Teste 18.1

- **Propósito:** validar soft delete de boletim a partir da listagem
  admin.
- **Pré-condições:** logado como admin; boletim existente com
  `deletedAt = null`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/boletins`                           |
| Ação                      | clicar no ícone de lixeira da linha alvo     |
| Confirmação               | aceitar o `confirm()` do navegador          |

- **Saída Aguardada:**
  - DELETE `/api/reports/<id>` responde 200.
  - `reports.deletedAt` é preenchido com timestamp atual.
  - Linha do boletim desaparece da listagem `/admin/boletins`.
  - Mesmo boletim deixa de aparecer em `/relatorios`.

### Caso de Teste 18.2

- **Propósito:** validar soft delete via botão público em
  `/relatorios` (interface alternativa).
- **Pré-condições:** logado como admin; boletim publicado visível
  em `/relatorios`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/relatorios`                               |
| Ação                      | clicar em `DeleteBulletinButton` do card    |
| Confirmação               | aceitar "Apagar este boletim?"              |

- **Saída Aguardada:**
  - DELETE `/api/reports/<id>` responde 200.
  - Página recarrega; o card do boletim deixa de existir na
    listagem pública.
  - `reports.deletedAt` registrado no banco.

### Caso de Teste 18.3

- **Propósito:** verificar comportamento ao cancelar a confirmação
  de exclusão.
- **Pré-condições:** logado como admin; boletim existente.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Ação                      | clicar no botão de apagar                   |
| Confirmação               | cancelar o `confirm()`                      |

- **Saída Aguardada:**
  - Nenhuma chamada DELETE é disparada.
  - `reports.deletedAt` permanece `null`.
  - Boletim continua visível em ambas as listagens.

---

## 8.19 Teste "UC-19 Visualizar atividade administrativa"

### Caso de Teste 19.1

- **Propósito:** validar listagem de eventos administrativos com
  filtros por tipo.
- **Pré-condições:** logado como admin; `ingest_logs` com ações
  manuais (`triggeredBy = 'manual'`); `reports` com ao menos um
  evento de publicação.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/atividade`                          |

- **Saída Aguardada:**
  - Timeline lista eventos administrativos com timestamp, autor
    (e-mail), tipo de ação (ingest manual, publicação, exclusão)
    e detalhes.
  - Filtro por tipo de evento atualiza a listagem sem recarregar a
    página inteira.

### Caso de Teste 19.2

- **Propósito:** verificar paginação de eventos antigos.
- **Pré-condições:** mais de 50 eventos registrados em
  `ingest_logs`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/atividade?page=2`                   |

- **Saída Aguardada:**
  - Página exibe a segunda página de resultados.
  - Controles "Anterior/Próxima" funcionam.
  - Total de eventos é apresentado no cabeçalho.

---

## 8.20 Teste "UC-20 Visualizar onboarding/checklist"

### Caso de Teste 20.1

- **Propósito:** validar exibição do checklist de onboarding no
  primeiro login.
- **Pré-condições:** logado como admin pela primeira vez;
  `localStorage` sem entrada `ecodata_onboarding_done`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/dashboard`                          |
| Sessão                    | admin recém-logado                          |

- **Saída Aguardada:**
  - Modal/checklist aparece com 4-6 etapas (rodar ingest, gerar
    boletim, publicar, conferir glossário).
  - Cada etapa tem CTA navegável para a tela correspondente.
  - Botão "Dispensar" oculta o checklist e grava
    `ecodata_onboarding_done = true`.

### Caso de Teste 20.2

- **Propósito:** confirmar que o checklist não reaparece após
  dispensa.
- **Pré-condições:** `localStorage.ecodata_onboarding_done = "true"`.
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| URL                       | `/admin/dashboard`                          |

- **Saída Aguardada:**
  - Checklist não é renderizado na carga inicial.
  - Botão discreto "Reabrir tour" continua disponível no header.

### Caso de Teste 20.3

- **Propósito:** validar atualização do progresso conforme
  etapas concluídas.
- **Pré-condições:** checklist visível; admin executou ingest
  manual (UC-14).
- **Lista de Entradas:**

| Campo                     | Valor                                       |
|---------------------------|---------------------------------------------|
| Ação                      | concluir etapa "Rodar ingest manual"        |

- **Saída Aguardada:**
  - Etapa correspondente recebe estado `checked`.
  - Barra de progresso avança proporcionalmente.
  - Próxima etapa é destacada como atual.

---

## Referências cruzadas

- `04-casos-de-uso-descricoes.md` — descrição textual de cada UC
  citado nas pré-condições e saídas esperadas.
- `05-diagramas-sequencia.md` — diagrama de sequência por UC,
  útil para reproduzir as chamadas HTTP descritas nos testes.
- `07-banco-de-dados.md` — schema das tabelas `reports`,
  `ingest_logs`, `monthly_stats`, `stations` e `template_versions`
  referenciadas pelas saídas esperadas.
- `09-telas-da-solucao/` — screenshots das telas em que os testes
  são executados.

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: section-08-test-cases
total_use_cases: 20
test_cases_per_uc: 2-3
alignment: 8.N matches UC-NN
```
<!-- /agent-extractable -->
