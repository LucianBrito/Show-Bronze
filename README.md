# 🥉 Show Bronze — Painel Comercial

Projeto de portfólio de **Análise de Dados e Business Intelligence**, desenvolvido a partir de dados fictícios de um e-commerce de cosméticos bronzeadores.

O projeto simula o trabalho de um Analista de Dados/BI desde a preparação e validação dos dados até a análise em SQL, exploração com Python e construção de um dashboard executivo no Power BI.

> **Dados fictícios**, gerados exclusivamente para estudo, demonstração técnica e composição de portfólio.

---

## 🎯 Objetivo do projeto

Construir uma solução analítica capaz de responder perguntas de negócio relacionadas a:

- evolução do faturamento ao longo do tempo;
- desempenho por canal e loja;
- impacto da política de descontos;
- desempenho de produtos, categorias e marcas;
- concentração de faturamento entre canais e parceiros;
- identificação de produtos de maior participação nas vendas.

---

## 📊 Resultado

O projeto foi estruturado como um relatório Power BI de **3 páginas**, com navegação lateral e foco em análise comercial:

| Página | Conteúdo |
|---|---|
| **Visão Geral** | KPIs, evolução mensal do faturamento, comparação com ano anterior e Top 5 produtos |
| **Canais e Descontos** | Faturamento por loja/canal e análise de descontos |
| **Produto e Categoria** | Treemap, drill-down, ranking e participação dos produtos dentro das categorias |

### Prévia do dashboard

![Visão Geral](Prints/Visão%20Geral.png)

![Canais e Descontos](Prints/Canais%20e%20Descontos.png)

![Produto e Categoria](Prints/Produto%20e%20Categoria.png)

---

## 🗂️ Modelo de dados

O projeto utiliza **modelo dimensional em esquema estrela**, composto por uma tabela fato e seis dimensões:

```text
                    dim_categoria
                          │
   dim_marca ──────  fato_vendas  ────── dim_loja
                          │
              ┌───────────┼───────────┐
        dim_produto    dim_data   dim_faixa_preco
```

### Tabela fato

`fato_vendas` possui **3.100 registros** e tem como grão uma linha por venda de um produto.

Principais campos:

`id_produto`, `id_categoria`, `id_marca`, `id_data`, `id_loja`, `id_faixa_preco`, `quantidade_vendida`, `preco_unitario`, `desconto_percentual`, `valor_bruto`, `valor_desconto`, `valor_liquido`, `status_estoque`.

### Dimensões

- `dim_categoria` — 13 categorias
- `dim_data` — calendário completo de 2016 a 2026
- `dim_faixa_preco` — 6 faixas de preço
- `dim_loja` — 4 lojas/canais
- `dim_marca` — 4 marcas
- `dim_produto` — 27 produtos

### Totais de referência

| Indicador | Valor |
|---|---:|
| Registros em `fato_vendas` | **3.100** |
| Quantidade vendida | **139.368** |
| Faturamento líquido | **R$ 4.082.325,28** |

---

## 🔧 Metodologia

### 1. Preparação e qualidade dos dados

- identificação e correção de aproximadamente 16 ocorrências de corrupção de encoding em textos acentuados;
- validação da integridade referencial entre fato e dimensões;
- conferência dos totais da tabela fato;
- validação dos principais indicadores antes da construção do relatório.

### 2. Modelagem dimensional

Foi construído um modelo em **esquema estrela** no Power BI.

Durante a modelagem foi identificado um relacionamento ambíguo: `dim_produto` possuía caminhos duplicados até `dim_marca` e `dim_categoria`. As relações diretas desnecessárias foram removidas para manter o modelo com um único caminho de filtro entre dimensões e fato.

### 3. Debugging de dados e locale

Durante a validação dos KPIs, os valores apresentados no Power BI estavam incorretos mesmo com as medidas DAX corretas.

A investigação identificou um problema na conversão de tipos no Power Query: o código `Table.TransformColumnTypes` não especificava a cultura/locale, fazendo com que o separador decimal do CSV fosse interpretado incorretamente.

A correção foi aplicar explicitamente o locale **`en-US`** na conversão dos campos numéricos.

Esse processo foi importante porque demonstrou uma situação real de trabalho de BI: **um resultado incorreto no dashboard nem sempre significa que o DAX está errado; a origem pode estar na ingestão ou transformação dos dados.**

### 4. Análise exploratória

O projeto também contém análises desenvolvidas em **Python/pandas** e **SQL/DuckDB**, permitindo comparar os resultados e validar os indicadores antes da publicação no Power BI.

### 5. DAX e análise de contexto

Foram desenvolvidas mais de 20 medidas DAX, organizadas por finalidade:

- Base e agregação
- Indicadores
- Inteligência temporal
- Lógica
- Contexto e hierarquia
- Medidas dinâmicas

---

## 🧮 DAX em destaque

### Participação no subtotal da categoria

```dax
% sobre o Subtotal da Categoria =
VAR TotalCategoria =
    CALCULATE(
        [Faturamento Líquido],
        ALLSELECTED(dim_produto[nome_produto])
    )
RETURN
    SWITCH(
        TRUE(),
        ISINSCOPE(dim_produto[nome_produto]),
            DIVIDE([Faturamento Líquido], TotalCategoria, 0),
        BLANK()
    )
```

Utiliza `ALLSELECTED` e `ISINSCOPE` para calcular a participação do produto dentro do contexto selecionado da categoria.

### Medida dinâmica

```dax
Medida Dinâmica - Valor =
VAR MedidaEscolhida = SELECTEDVALUE(aux_Medidas[Medida])
RETURN
    SWITCH(
        MedidaEscolhida,
        "Faturamento Líquido", [Faturamento Líquido],
        "Quantidade Vendida", [Quantidade Vendida],
        "Desconto Total", [Desconto Total],
        "Ticket Médio", [Ticket Médio],
        BLANK()
    )
```

Permite alternar a métrica apresentada em um visual por meio de um slicer, sem duplicar os visuais para cada indicador.

---

## 💡 Principais insights

- **A política de desconto não apresentou aumento relevante no volume médio vendido:** a quantidade média por venda permanece próxima de 44–46 unidades nas diferentes faixas de desconto.
- **O canal B2B representa aproximadamente 51% do faturamento**, apesar de corresponder a apenas 2 das 4 lojas/canais analisados, indicando concentração relevante em parceiros de revenda.
- **Parafinas** lidera o faturamento entre as categorias, seguida por **Óleos** e **Kits**.

> Os insights devem ser interpretados como resultados da base fictícia utilizada neste projeto e não como dados reais de mercado.

---

## 🛠️ Tecnologias e ferramentas

| Tecnologia | Aplicação |
|---|---|
| **Power BI** | Modelagem, DAX, visualização e dashboard |
| **Power Query (M)** | ETL, limpeza e transformação |
| **SQL / DuckDB** | Consultas analíticas e validação |
| **Python / pandas** | Análise exploratória e validação |
| **NumPy / Matplotlib** | Apoio à análise e visualização |
| **Jupyter Notebook** | Ambiente das análises Python e SQL |
| **Git / GitHub** | Versionamento e portfólio |

---

## 📁 Estrutura do repositório

```text
Show-Bronze/
│
├── Dados/
│   ├── dim_categoria.csv
│   ├── dim_data.csv
│   ├── dim_faixa_preco.csv
│   ├── dim_loja.csv
│   ├── dim_marca.csv
│   ├── dim_produto.csv
│   ├── fato_vendas.csv
│   └── Analise_Show_Bronze.ipynb
│
├── Desiner/
│   ├── Background.png
│   ├── Show-Bronze-Logo-Dourada.webp
│   └── show_bronze_powerbi_theme.json
│
├── Power Bi/
│   └── Analise Show Bronzebi.pbix
│
├── Prints/
│   ├── Visão Geral.png
│   ├── Canais e Descontos.png
│   └── Produto e Categoria.png
│
├── Python/
│   └── Show_Bronze_Analise_Python.ipynb
│
├── SQL/
│   └── Show_Bronze_Analise_SQL.ipynb
│
└── README.md
```

> **Nota de padronização:** os diretórios `Desiner` e `Power Bi` ainda podem ser renomeados para `Designer` e `PowerBI` em uma próxima limpeza estrutural. O conteúdo analítico não depende desses nomes, mas a padronização melhora a apresentação do repositório.

---

## 🔎 Competências demonstradas

Este projeto demonstra competências relevantes para posições de **Analista de Dados / Analista de BI Júnior**:

- análise exploratória;
- SQL analítico;
- Python/pandas;
- tratamento e qualidade de dados;
- Power Query;
- modelagem dimensional;
- DAX;
- inteligência temporal;
- análise de contexto de filtro;
- construção de KPIs;
- visualização de dados;
- storytelling de dados;
- validação cruzada dos resultados;
- documentação técnica.

---

## 👤 Autor

**Luciano Brito**

[LinkedIn](https://www.linkedin.com/in/luciano-concei%C3%A7%C3%A3o-de-brito/) · [GitHub](https://github.com/LucianBrito)

---

### 📌 Status do projeto

**Portfólio — versão consolidada em Power BI, SQL e Python.**

O projeto continua aberto a melhorias de modelagem, novas análises e refinamentos de visualização.
