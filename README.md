# 🥉 Show Bronze — Painel Comercial

Análise de vendas de um e-commerce fictício de cosméticos bronzeadores, com dashboard interativo construído no Power BI. Projeto de portfólio para Analista de Dados/BI.

> **Dados fictícios**, gerados para fins de estudo e demonstração de competências técnicas.

---

## 📌 Sobre o projeto

A Show Bronze vende produtos bronzeadores (óleos, parafinas, kits, esfoliantes, protetores solares) através de 4 canais: e-commerce próprio, marketplace (Mercado Livre) e 2 revendedores B2B. Este projeto simula o trabalho de um Analista de BI contratado para responder às principais perguntas de negócio da empresa:

- Quanto a empresa está faturando, e como isso evolui ao longo do tempo?
- Quais canais e lojas performam melhor?
- A política de desconto está realmente gerando mais volume de venda?
- Quais categorias e produtos lideram o faturamento?

## 🗂️ Modelo de dados

Esquema estrela — uma tabela fato e 6 dimensões:

```
                    dim_categoria
                          │
   dim_marca ──────  fato_vendas  ──────  dim_loja
                          │
              ┌───────────┼───────────┐
        dim_produto   dim_data   dim_faixa_preco
```

`fato_vendas` — grão: uma linha por venda de um produto. Colunas: `id_produto, id_categoria, id_marca, id_data, id_loja, id_faixa_preco, quantidade_vendida, preco_unitario, desconto_percentual, valor_bruto, valor_desconto, valor_liquido, status_estoque`.

## 🔧 Metodologia

1. **Limpeza de dados** — identificação e correção de ~16 ocorrências de corrupção de encoding em textos acentuados (ex: `Ó³LEOS` → `ÓLEOS`), validação de integridade referencial entre fato e dimensões.
2. **Modelagem** — construção de esquema estrela no Power BI; identificação e correção de um relacionamento ambíguo (`dim_produto` tinha caminho duplicado até `dim_marca`/`dim_categoria`, criando risco de filtro incorreto).
3. **Debugging de locale no Power Query** — os cards do relatório mostravam valores incorretos mesmo com fórmulas corretas. Investigação levou à causa raiz: a etapa de conversão de tipo (`Table.TransformColumnTypes`) não especificava o parâmetro de cultura/locale, fazendo o Power Query interpretar o separador decimal do CSV (`.`) como separador de milhar. Corrigido especificando `"en-US"` explicitamente no código M.
4. **Criação de +20 medidas DAX**, organizadas em 6 grupos por finalidade (Base/Agregação, Indicadores, Tempo, Lógica, Contexto/Hierarquia, Medidas Dinâmicas).
5. **Construção do relatório** — 3 páginas com navegação lateral customizada (botões com ação de navegação de página).

## 🧮 Medidas DAX em destaque

```dax
% sobre o Subtotal da Categoria =
VAR TotalCategoria = CALCULATE([Faturamento Líquido], ALLSELECTED(dim_produto[nome_produto]))
RETURN
SWITCH(TRUE(),
    ISINSCOPE(dim_produto[nome_produto]), DIVIDE([Faturamento Líquido], TotalCategoria, 0),
    BLANK()
)
```
Calcula a participação de cada produto dentro do total da sua categoria — só retorna valor quando a hierarquia está no nível de produto (`ISINSCOPE`), evitando que o número "vaze" pro nível de categoria sem sentido.

```dax
Medida Dinâmica - Valor =
VAR MedidaEscolhida = SELECTEDVALUE(aux_Medidas[Medida])
RETURN
SWITCH(MedidaEscolhida,
    "Faturamento Líquido", [Faturamento Líquido],
    "Quantidade Vendida", [Quantidade Vendida],
    "Desconto Total", [Desconto Total],
    "Ticket Médio", [Ticket Médio],
    BLANK()
)
```
Permite trocar a métrica exibida num visual através de um slicer, sem precisar de um gráfico por métrica.

## 📊 Páginas do relatório

| Página | Conteúdo |
|---|---|
| **Visão Geral** | KPIs principais, evolução mensal do faturamento (com comparação ano anterior), eficácia do desconto, Top 5 produtos |
| **Canais e Descontos** | Faturamento por loja e por tipo de canal, desconto médio por canal |
| **Produto e Categoria** | Treemap de faturamento por categoria com drill-down a produto, matriz de produtos com ranking e participação na categoria |

## 💡 Principais insights

- **Desconto não aumenta o volume médio de venda** — a quantidade média por venda fica estável entre 44 e 46 unidades, independente de o desconto ser 0% ou 20%. Isso sugere que a política de desconto atual pode estar corroendo margem sem gerar volume incremental — vale um teste controlado antes de ampliar a estratégia.
- **O canal B2B concentra ~51% do faturamento**, mesmo sendo apenas 2 das 4 lojas ativas — indica dependência de poucos parceiros de revenda.
- A categoria **Parafinas** lidera o faturamento (R$ 858 mil), seguida por Óleos (R$ 703 mil) e Kits (R$ 437 mil).

## 🛠️ Tecnologias

- **Power BI Desktop** — modelagem, DAX, relatório
- **Power Query (M)** — transformação e limpeza de dados
- **Python (pandas)** — geração e validação inicial dos dados fictícios

## 👤 Autor

Luciano Brito
[LinkedIn](https://www.linkedin.com/in/luciano-concei%C3%A7%C3%A3o-de-brito/) · [GitHub](https://github.com/LucianBrito))

