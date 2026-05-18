---
file: 01-introducao.md
section: 1. Introdução
purpose: Apresenta o propósito do projeto EcoData e seu público-alvo
depends_on: []
last_updated: 2026-05-18
status: draft
---

# 1. Introdução

> **Para agentes de IA**: arquivo introdutório. Define o problema (acesso descentralizado e tardio a dados de qualidade da água nas Bacias PCJ) e os públicos atendidos (cidadãos, gestores, pesquisadores). É a base contextual para todas as outras seções.

## 1.1 Propósito

As Bacias Hidrográficas dos Rios Piracicaba, Capivari e Jundiaí (Bacias PCJ) compreendem 76 municípios e abastecem mais de cinco milhões de habitantes no interior paulista. O monitoramento contínuo dessas águas é conduzido por dois órgãos distintos: a Companhia Ambiental do Estado de São Paulo (CETESB), por meio do Sistema de Informações da Monitoramento da Qualidade das Águas (SIMQUA), responsável pelos parâmetros físico-químicos de qualidade; e o Sistema de Suporte à Decisão das Bacias PCJ (SSD PCJ), operado pela Agência das Bacias PCJ, que consolida dados telemétricos de quantidade — chuva, nível e vazão. Embora ambas as fontes mantenham coletas em frequência sub-diária, a publicação consolidada ao público ocorre apenas no boletim mensal integrado, distribuído em formato PDF, com latência que pode ultrapassar trinta dias entre a leitura em campo e o acesso pelo destinatário final.

O presente trabalho apresenta o **EcoData**, plataforma web experimental de caráter acadêmico que tem por finalidade consolidar e republicar, em uma única interface pública, os dados de qualidade e quantidade das águas das Bacias PCJ. O sistema executa rotinas de ingestão horária junto às fontes oficiais, aplica classificação automática contra os limites estabelecidos pela Resolução CONAMA 357/2005 (Classe 2) e disponibiliza visualizações geoespaciais, gráficos temporais, glossário técnico e geração programática de boletins em PDF. A intenção é reduzir a distância temporal e cognitiva entre a coleta da medição e a sua interpretação pelo público interessado, sem substituir as publicações oficiais, mas oferecendo um ponto de consulta complementar, pesquisável e atualizado.

Cumpre destacar que o EcoData é um projeto **experimental e não-oficial**, desenvolvido como Trabalho de Conclusão de Curso, sem vínculo institucional com a CETESB, com a Agência das Bacias PCJ ou com qualquer órgão público. Todos os dados exibidos são de origem pública governamental, e a plataforma referencia explicitamente cada fonte original, preservando a autoridade dos provedores primários.

## 1.2 Público-alvo

A plataforma EcoData foi projetada para atender três grupos distintos, cujas necessidades informacionais convergem em torno do mesmo conjunto de dados, mas exigem superfícies de consumo diferenciadas:

- **Cidadãos das Bacias PCJ.** Moradores e usuários dos rios da região que demandam respostas diretas — em linguagem acessível — sobre a situação atual da água em seu trecho de interesse. A esse grupo atendem o mapa interativo da página inicial (`/`), as fichas de rio (`/rio/[id]`) e a página de atividade recente (`/admin/atividade`), com semáforos visuais de classificação e textos contextuais que dispensam familiaridade prévia com terminologia técnica.

- **Gestores públicos e técnicos ambientais.** Servidores de prefeituras, comitês de bacia, conselhos e órgãos ambientais que necessitam de panoramas consolidados, séries históricas e documentos formatados para uso em reuniões, deliberações e relatórios. Para essa audiência, a plataforma disponibiliza a área de relatórios (`/relatorios`) com boletins mensais em PDF passíveis de download, bem como o catálogo completo de estações monitoradas (`/admin/estacoes`), filtrável por bacia e por classe.

- **Pesquisadores e jornalistas de dados.** Profissionais acadêmicos e da imprensa que demandam acesso histórico estruturado, com possibilidade de comparação temporal e referência cruzada. A combinação entre `/relatorios` (PDFs versionados) e as fichas de rio com janelas temporais ajustáveis oferece pontos de partida para investigações mais aprofundadas, ainda que a exportação de dados brutos em formato CSV permaneça como evolução prevista em trabalhos futuros.

---

## Referências cruzadas

- [`02-visao-geral.md`](./02-visao-geral.md) — Visão geral do produto e modelo de operação
- [`06-arquitetura.md`](./06-arquitetura.md) — Arquitetura do sistema
- [`AGENT-CONTEXT.md`](./AGENT-CONTEXT.md) — Digest geral do projeto

## Para agentes de IA

<!-- agent-extractable -->
```yaml
section_id: section-01-introducao
project: EcoData
problem_statement: |
  Dados de qualidade e quantidade das águas das Bacias PCJ são publicados
  por órgãos oficiais com latência de semanas, em formatos não-pesquisáveis
  (PDF mensal). Cidadãos, gestores e pesquisadores ficam sem acesso oportuno.
solution_statement: |
  Plataforma web que consolida SIMQUA + SSD PCJ em interface única, com
  atualização horária, classificação CONAMA 357 automática e visualização
  geoespacial.
audiences:
  - id: cidadao
    label: Cidadãos das Bacias PCJ
    primary_surface: "/, /rio/[id], /admin/atividade"
  - id: gestor
    label: Gestores públicos e técnicos ambientais
    primary_surface: "/relatorios, /admin/estacoes"
  - id: pesquisador
    label: Pesquisadores e jornalistas de dados
    primary_surface: "/relatorios, download de PDFs"
```
<!-- /agent-extractable -->
