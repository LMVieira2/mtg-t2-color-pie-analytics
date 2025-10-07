# 🧙‍♂️ MTG Data Lakehouse – Análise de Cartas de Magic: The Gathering

## 📘 Visão Geral

Este projeto implementa um pipeline **end-to-end** de ingestão, tratamento e análise de dados de cartas de *Magic: The Gathering* utilizando o **Azure Databricks** e o **Unity Catalog**.  
O objetivo é construir uma arquitetura **Lakehouse (Bronze → Silver → Gold)** para disponibilizar dados limpos e modelados ao **Power BI**, permitindo a criação de KPIs e análises de metajogo e design de cartas.

---

## 🧱 Arquitetura

### 🔸 Bronze Layer – Ingestão
- Dados brutos ingeridos do endpoint da [Scryfall API](https://scryfall.com/docs/api).
- Estrutura salva em formato **Delta Table** (`mtg.bronze.cards_raw`).
- Colunas complexas como `card_faces` são mantidas em JSON.
- Funções aplicadas nesta etapa:
  - Explosão de arrays (`explode`)
  - Extração de campos compostos (e.g., `mana_cost`, `colors`, `oracle_text`).
  - Tratamento de MDFCs (cartas dupla face).

### 🔸 Silver Layer – Limpeza e Padronização
- Foco em limpeza e enriquecimento dos dados.
- Tabela: `mtg.silver.cards_clean`
- Principais tratamentos:
  - Substituição de nulos (`coalesce`) em `power`, `toughness`, `loyalty`.
  - Atribuição de `"colorless"` para cartas sem cor ou identidade de cor.
  - Normalização do campo `mana_cost` (remoção de `{}`).
  - Padronização de colunas (`colors`, `color_identity`, `type_line`).

### 🔸 Gold Layer – Modelagem e Métricas
- Tabela e view final para consumo no Power BI.
- Campos disponibilizados:
name,
mana_cost,
colors,
color_identity,
set,
set_name,
type_cleaned

- `type_cleaned` é derivado de `type_line`, removendo supertipos (ex: *Legendary*) e subtipos (ex: *Human Wizard*).

- Criação de **view** no Unity Catalog:
```sql
CREATE OR REPLACE VIEW mtg.gold.vw_cards_for_powerbi AS
SELECT name, mana_cost, colors, color_identity, set, set_name, type_cleaned
FROM mtg.silver.cards_clean;
```

Principais Aprendizados

Tratamento de colunas complexas e arrays (card_faces) com PySpark.

Uso de funções condicionais (F.when, F.coalesce, F.regexp_replace).

Modelagem em camadas Delta (Bronze → Silver → Gold).

Criação de views e integração com Power BI via Unity Catalog.

Criação de medidas e colunas calculadas em DAX.

📊 Tecnologias Utilizadas
| Categoria |	Ferramenta |
| --------- | ---------- |
| Data Lakehouse | Azure Databricks + Unity Catalog |
| Linguagem |	PySpark / SQL / DAX |
| Visualização | Power BI |
| Armazenamento	| Azure Data Lake Storage Gen2 |
| API Fonte |	Scryfall |
