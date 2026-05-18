---
file: 02-visao-geral.md
section: 2 — Visão geral do produto
purpose: Apresentar hipótese, proposta, atores, modelo de operação, fontes de dados e contraste com soluções existentes do EcoData.
depends_on:
  - 01-introducao.md
last_updated: 2026-05-18
status: draft
---

# 2. Visão geral do produto

> **Para agentes de IA**: este documento define o escopo conceitual do
> EcoData: a hipótese central, a proposta de produto (7 capacidades
> numeradas), os dois atores do sistema (Visitante/Cidadão e
> Administrador), o modelo de operação (experimental, sem fins
> comerciais, sem vínculo institucional formal), as quatro fontes de
> dados consumidas e o contraste com três soluções pré-existentes. Use
> esta seção para extrair os identificadores estáveis `actors`,
> `operating_model`, `data_sources` e `key_features`, documentados no
> bloco YAML ao final.

---

## 2.1 Hipótese

O EcoData parte da hipótese de que **a apresentação contínua, pública e
classificada de dados de qualidade e quantidade de água dos rios da
Bacia PCJ produz três efeitos mensuráveis**:

1. **Ganho de consciência ambiental** — o cidadão que mora próximo a um
   rio passa a reconhecer indicadores como OD, pH e turbidez, e a
   relacioná-los com a saúde do corpo hídrico.
2. **Vigilância cívica distribuída** — a publicação contínua cria um
   histórico verificável: qualquer pessoa pode observar tendências,
   identificar anomalias e cobrar resposta dos órgãos competentes.
3. **Suporte a decisão informal** — gestores municipais, jornalistas
   locais, professores e comitês de bacia ganham uma camada de leitura
   intermediária entre o dado bruto e o boletim oficial mensal.

A hipótese sustenta-se na lacuna documentada na Seção 1: os dados
existem em sistemas governamentais técnicos, mas a linguagem, a forma
de acesso e a ausência de classificação visual impedem que cumpram
função informativa para o público leigo.

---

## 2.2 Proposta

O EcoData materializa a hipótese em sete capacidades concretas:

1. **Consolida CETESB SIMQUA + SSD PCJ.** Os dois sistemas oficiais —
   qualidade pelo SIMQUA (CETESB) e quantidade pelo SSD PCJ (Agência
   das Bacias PCJ) — são reunidos em um banco normalizado referenciado
   pelo código oficial CETESB de cada estação.
2. **Atualiza-se de hora em hora via cron Vercel.** Endpoints
   protegidos por Bearer secret (`/api/cron/ingest-*`) garantem que a
   interface pública nunca exiba dados com mais de 60–90 minutos de
   defasagem em relação à origem.
3. **Classifica leituras (OD, pH, turbidez) por CONAMA 357.** Uma
   função pura aplica os limites da Classe 2 da Resolução CONAMA
   357/2005 a cada medição e devolve um nível discreto (Normal,
   Atenção, Alerta, Crítico) com mensagem contextual em pt-BR.
4. **Exibe malha hídrica em mapa interativo (GeoJSON do OSM).** Um
   mapa Leaflet sobre OpenStreetMap apresenta as estações como
   marcadores coloridos pelo nível CONAMA atual, com camadas de rios e
   sub-bacias derivadas do Overpass API e da base SEMIL-SP 1:50k.
5. **Mantém histórico mensal por trecho (`/rio/[id]`).** Cada trecho
   monitorado tem página própria com séries históricas (7 d / 30 d /
   90 d), barras de conformidade por parâmetro, exportação CSV e
   ligação direta para o boletim mensal mais recente.
6. **Gera boletim PDF mensal (versão experimental, identificada como
   não-oficial).** Todo dia 5 do mês um cron externo dispara a geração
   do *Boletim Integrado de Qualidade e Quantidade das Águas*,
   reproduzindo o layout oficial das Bacias PCJ. **Todos os PDFs
   trazem carimbo "PROJETO EXPERIMENTAL — Dados não oficiais — Dados
   não validados"**.
7. **Glossário técnico em `/glossario`.** Mais de vinte termos (OD,
   IQA, CONAMA 357, turbidez, Q7,10, balneabilidade, etc.) com
   definição formal, analogia do cotidiano, fonte oficial e link
   âncora copiável.

---

## 2.3 Atores

O sistema reconhece dois atores. Toda funcionalidade do EcoData pertence
ao escopo de um deles.

| Ator    | Rótulo                | Autenticação | Responsabilidades                                                                                                                   |
|---------|-----------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------|
| Ator-01 | Visitante / Cidadão   | Pública      | Consulta mapa em tempo real, navega por rios, lê boletins, busca termos no glossário e visualiza atividade recente sem login.       |
| Ator-02 | Administrador         | Autenticado  | Monitora o pipeline de ingestão, dispara cargas manuais, gera/regenera boletins, edita o template do PDF e auditora a atividade.   |

O Ator-01 corresponde a aproximadamente cinco milhões de habitantes das
Bacias PCJ, jornalistas regionais, professores e gestores municipais. O
Ator-02 é interno à operação do TCC (autor + orientadores) e opera sob
`/admin/*` com sessão por cookie e proteção por *getSession()*.

---

## 2.4 Modelo de operação

O EcoData é um **projeto experimental, sem fins comerciais e sem
vínculo institucional formal** com a CETESB, com a Agência das Bacias
PCJ, com a SEMIL-SP ou com qualquer órgão público. Trata-se de Trabalho
de Conclusão de Curso (TCC) submetido à PUC-Campinas, executado por um
único discente com orientação acadêmica.

Decorrem desse modelo três compromissos não negociáveis:

- **Disclaimer permanente em todo boletim PDF.** Cada PDF traz, em
  local visível, o carimbo **"PROJETO EXPERIMENTAL — Dados não
  oficiais — Dados não validados"** na capa e no rodapé de cada
  página, embutido no template `bulletin-pdf.tsx`.
- **Disclaimer permanente na interface web.** Rodapé global, *eyebrow*
  do hero e cards em `/relatorios` reiteram a natureza não-oficial dos
  dados, ainda que as leituras brutas sejam copiadas integralmente das
  fontes oficiais.
- **Hospedagem gratuita.** O custo de operação é nominalmente zero:
  Vercel (Hobby), Neon (plano free `sa-east-1`) e cron-job.org (plano
  gratuito). Não há receita, patrocínio ou contrato comercial.

---

## 2.5 Fontes de dados

O EcoData consome quatro fontes públicas, sem qualquer acesso
privilegiado:

| ID       | Tipo                    | Endpoint / origem                                    | Dados extraídos                                                                                | Frequência       |
|----------|-------------------------|------------------------------------------------------|------------------------------------------------------------------------------------------------|------------------|
| DS-01    | SSD PCJ — séries        | `https://sspcj.agenciapcj.org.br/api/data`           | Séries temporais de chuva, nível e vazão por estação telemétrica, com agregação configurável.  | Horária (cron)   |
| DS-02    | SSD PCJ — última leitura | `https://sspcj.agenciapcj.org.br/api/lastdata`       | Snapshot da medição mais recente de cada estação SSD PCJ (chuva, nível, vazão).                | Horária (cron)   |
| DS-03    | CETESB SIMQUA / QUALAR   | Scraping HTML do portal QUALAR (CETESB)              | Medições de qualidade (OD, pH, turbidez, condutividade, temperatura) das estações SIMQUA.       | Horária (cron)   |
| DS-04    | OpenStreetMap — Overpass | `https://overpass-api.de/api/interpreter`            | Geometrias dos rios da Bacia PCJ em GeoJSON para a camada hidrográfica do mapa.                 | Sob demanda      |

A integração SAISP (SP-Águas direto) está deliberadamente fora do
escopo: o SSD PCJ já agrega os dados da SP-Águas, e duplicar a
captura significaria implementar *scraping* JSP para obter exatamente o
mesmo conjunto de leituras.

---

## 2.6 Contraste com soluções existentes

O EcoData não substitui as fontes oficiais; **posiciona-se como camada
de mediação** entre dado bruto e leitura cidadã. Os três sistemas a
seguir já existem e cumprem propósitos distintos:

### 2.6.1 Sistema SSD PCJ original

Plataforma técnica voltada a operadores de comitê de bacia. Apresenta
séries telemétricas em formato tabular e gráficos densos, sem
classificação visual CONAMA 357 e sem tradução de termos. É fonte de
dados de quantidade do EcoData, não concorrente.

### 2.6.2 Site QUALAR (CETESB)

Interface de consulta das séries de qualidade. Exporta planilhas, não
oferece mapa interativo, não classifica medições e não publica
boletins. Função arquivística e de transparência primária; não
desenhada para consumo cidadão.

### 2.6.3 Boletim oficial integrado (SSPCJ)

PDF mensal com layout consolidado, mapas e tabelas por trecho —
referência editorial reproduzida pelo PDF experimental do EcoData, com
duas limitações práticas: **latência de 30 a 60 dias entre o mês de
referência e a publicação** e formato estático (sem interatividade,
sem busca, sem âncoras por estação).

O EcoData ocupa a faixa entre o dado bruto (SSD PCJ / QUALAR) e o
documento mensal consolidado (boletim oficial): atualização horária,
classificação visual contínua, navegação por rio e glossário aberto,
sempre identificado como produto experimental.

---

## Referências cruzadas

- `01-introducao.md` — contexto da Bacia PCJ e justificativa do projeto.
- `03-casos-de-uso-diagramas.md` — diagramas dos casos de uso por ator.
- `06-arquitetura.md` — camadas técnicas que materializam a proposta.
- `glossario-tecnico.md` — definições formais dos termos citados (OD, pH, CONAMA, IQA).

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: section-02-product-overview
actors:
  - id: ator-01
    label: Visitante / Cidadão
    auth: public
  - id: ator-02
    label: Administrador
    auth: authenticated
operating_model:
  commercial: false
  institutional_affiliation: false
  data_validation: "nenhuma — projeto experimental"
  hosting:
    platform: vercel
    plan: hobby
    cost_per_month_brl: 0
  mandatory_disclaimer: "PROJETO EXPERIMENTAL — Dados não oficiais — Dados não validados"
data_sources:
  - id: ds-01
    type: ssd-pcj-series
    base_url: https://sspcj.agenciapcj.org.br/api/data
    metrics: [chuva, nivel, vazao]
  - id: ds-02
    type: ssd-pcj-lastdata
    base_url: https://sspcj.agenciapcj.org.br/api/lastdata
    metrics: [chuva, nivel, vazao]
  - id: ds-03
    type: cetesb-simqua-qualar
    base_url: https://qualar.cetesb.sp.gov.br/qualar/
    metrics: [od, ph, turbidez, condutividade, temperatura]
  - id: ds-04
    type: osm-overpass
    base_url: https://overpass-api.de/api/interpreter
    metrics: [geojson-rios-bacia-pcj]
key_features:
  - mapa-em-tempo-real
  - classificação-conama-357
  - boletim-pdf-mensal
  - glossário-técnico
  - séries-históricas-por-rio
  - admin-pipeline-observability
relationships:
  - from: ator-01
    to: key_features
    nature: consumes
  - from: ator-02
    to: data_sources
    nature: operates-ingestion
```
<!-- /agent-extractable -->
