# Case Rocket Lab 2026 · Grupo 2 · V-Credit

Pipeline de dados da operação de atendimento da V-Credit, construído no Databricks em arquitetura medalhão. Ele integra chamados, horários, custos, pesquisas de satisfação e as bases de referência em um modelo único, com indicadores atualizados por um Job diário e um painel no Power BI.

## Arquitetura

```
CSVs → Landing_to_Bronze → Bronze_to_Silver → Silver_to_Gold → CSVs → Power BI
```

| Camada | Conteúdo |
|---|---|
| Bronze | As 8 bases como chegaram, com metadados de carga |
| Silver | Bases tratadas e tipadas, mais o log de qualidade |
| Gold | Modelo estrela (5 dimensões, 2 fatos), 3 tabelas de KPI, log de qualidade consolidado e 6 views de análise |

Detalhes das regras, do modelo e das decisões em [docs/arquitetura.md](docs/arquitetura.md).

## Estrutura do repositório

```
├── Landing_to_Bronze.ipynb
├── Bronze_to_Silver.ipynb
├── Silver_to_Gold.ipynb
├── Jobs/
│   └── Job-Case-Grupo2-SP.json
└── docs/
    ├── arquitetura.md
    └── Conferencia_Idempotencia.ipynb
```

Os notebooks ficam na raiz porque o Job os lê por esse caminho. Eles estão salvos com os outputs da última execução.

## Como executar

1. Crie o catálogo e o Volume de entrada rodando a primeira célula do `Landing_to_Bronze`, ou crie `vcredit.bronze.inputs` manualmente.
2. Envie os 8 CSVs do case para `/Volumes/vcredit/bronze/inputs/`.
3. Crie o Job a partir de `Jobs/Job-Case-Grupo2-SP.json`. Ele lê os notebooks da branch `main` deste repositório.
4. Rode o Job. As tasks `to_Bronze`, `to_Silver` e `to_Gold` são executadas em sequência, e cada uma só começa se a anterior terminar com sucesso.

Os CSVs para o Power BI ficam em `/Volumes/vcredit/gold/exports/`.

## Execuções do Job

O Job roda todos os dias às 6h (America/Sao_Paulo).

## Idempotência

Todas as tabelas são gravadas em overwrite, as views são recriadas com `CREATE OR REPLACE` e as chaves substitutas são determinísticas. Para comprovar, o Job foi executado duas vezes e, depois de cada execução, o notebook `docs/Conferencia_Idempotencia.ipynb` registrou a quantidade de linhas e um checksum do conteúdo de cada tabela e view. As colunas com o horário da carga ficam de fora, porque mudam a cada execução.

Resultado: **34 de 34 tabelas e views idênticas** entre as duas execuções (8 na Bronze, 9 na Silver, 11 na Gold e 6 views).

## Painel

O painel no Power BI consome os CSVs da Gold e segue os relacionamentos descritos no fim do `Silver_to_Gold`.

## Dados

Os arquivos de origem e os CSVs exportados são confidenciais e não fazem parte deste repositório.

## Integrantes

- Annalice
- Beatriz Cristina
- João Pereira
- Leonardo Kenzo
- Rafael Massaro