# EcoData — TCC

Documentação acadêmica do projeto **EcoData**, espelhando a estrutura formal de um TCC de Sistemas de Informação (referência: `3_MandouBem.pdf`).

> **Para agentes de IA**: comece em [`AGENT-CONTEXT.md`](./AGENT-CONTEXT.md) antes de tudo. O arquivo descreve o projeto, mapeia os arquivos e indica como extrair conteúdo estruturado.

## Estrutura

| Seção | Arquivo                                                              | Conteúdo                                      |
|-------|----------------------------------------------------------------------|-----------------------------------------------|
| —     | [METADATA.yaml](./METADATA.yaml)                                     | Metadados para conversão Pandoc               |
| —     | [AGENT-CONTEXT.md](./AGENT-CONTEXT.md)                               | Digest para agentes de IA                     |
| 1     | [01-introducao.md](./01-introducao.md)                               | Propósito e público-alvo                      |
| 2     | [02-visao-geral.md](./02-visao-geral.md)                             | Visão geral do produto                        |
| 3     | [03-casos-de-uso-diagramas.md](./03-casos-de-uso-diagramas.md)       | Diagramas de casos de uso                     |
| 4     | [04-casos-de-uso-descricoes.md](./04-casos-de-uso-descricoes.md)     | Descrições dos 20 casos de uso                |
| 5     | [05-diagramas-sequencia.md](./05-diagramas-sequencia.md)             | 20 diagramas de sequência                     |
| 6     | [06-arquitetura.md](./06-arquitetura.md)                             | Arquitetura do sistema                        |
| 7     | [07-banco-de-dados.md](./07-banco-de-dados.md)                       | Modelo de dados (ER)                          |
| 8     | [08-casos-de-teste.md](./08-casos-de-teste.md)                       | 20 casos de teste                             |
| 9     | [09-telas-da-solucao/](./09-telas-da-solucao/)                       | Telas (1 .md + screenshot por tela)           |
| 10    | [10-planejamento-atividades.md](./10-planejamento-atividades.md)     | Planejamento real (git log → sprints)         |
| —     | [glossario-tecnico.md](./glossario-tecnico.md)                       | Glossário expandido                           |

## Ordem de leitura sugerida

1. `AGENT-CONTEXT.md` (overview)
2. `01-introducao.md` + `02-visao-geral.md` (contexto de negócio)
3. `03-casos-de-uso-diagramas.md` (panorama funcional)
4. `06-arquitetura.md` + `07-banco-de-dados.md` (visão técnica)
5. `04` → `05` → `08` → `09` (drill por caso de uso)
6. `10-planejamento-atividades.md` (cronograma)

## Conversão para PDF/DOCX

Pandoc (instalar separadamente):

```bash
cd docs/tcc
pandoc --metadata-file=METADATA.yaml --toc --pdf-engine=xelatex \
  01-introducao.md 02-visao-geral.md 03-casos-de-uso-diagramas.md \
  04-casos-de-uso-descricoes.md 05-diagramas-sequencia.md \
  06-arquitetura.md 07-banco-de-dados.md 08-casos-de-teste.md \
  09-telas-da-solucao/README.md $(ls 09-telas-da-solucao/9.*.md | sort) \
  10-planejamento-atividades.md glossario-tecnico.md \
  -o EcoData-TCC.pdf
```
