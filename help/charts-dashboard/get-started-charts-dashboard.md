---
title: Introdução a gráficos e painéis
description: Saiba como visualizar dados do Observability Insights com gráficos e painéis.
source-git-commit: 143a9567da06f0137ef690795cfbc41bda1780ca
workflow-type: tm+mt
source-wordcount: '309'
ht-degree: 1%
---

## Gráficos e painéis: visualização de dados {#charts-and-dashboards-visualizing-your-data}

As consultas encontram as informações necessárias. Gráficos e painéis tornam essas informações úteis para a sua organização. Este documento mostra como os resultados de SQL se tornam gráficos e como criar, organizar e compartilhar painéis.

### Gráficos

Cada resultado SQL pode ser renderizado como um dos seis tipos de gráfico.

| Tipo de gráfico | Use-o para | Forma de consulta |
| --------------- | --------------------------------------------------- | ----------------------------------------------- |
| **Linha** | Tendências ao longo do tempo; uma linha por valor de faceta | Agregado + `TIMESERIES` |
| **Área** | Volume ao longo do tempo, como taxa de transferência por serviço | Agregado + `TIMESERIES` |
| **Barra** | Comparação de categorias; séries temporais esparsas | Agregado + `FACET` (± `TIMESERIES`) |
| **Pizza/Rosca** | Compartilhamento de um todo (combinação de códigos de status, tráfego por camada) | Uma agregação + `FACET`, poucos valores |
| **Número** | Um único KPI, com limites de cor opcionais | Outdoor (somente agregados) |
| **Tabela** | Várias agregações por grupo ou listas de eventos | Várias agregações + `FACET` ou atributos brutos |

Depois que um widget está em um painel, você pode:

- **Edite** sua consulta, título, tipo de gráfico e unidade (ícone de lápis).
- **Duplique** para criar uma variante, como o mesmo gráfico para outra camada.
- **Baixar CSV** das linhas atuais ou **PNG** na resolução 2×.
- **Arraste-o** pela alça da alça e **redimensione-o** do canto inferior direito.

### Painéis

Os gráficos funcionam bem por conta própria, mas os painéis os reúnem. Um painel é uma coleção de **guias** com nome e escopo de locatário. Cada guia é uma grade de 12 colunas de **widgets**.

| Conceito | Significado |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Painel** | Uma coleção nomeada de 1 a 40 guias, armazenada no lado do servidor para o locatário selecionado. |
| **Guia** | Uma grade de widgets (até 200). Renomeie com o lápis, exclua-o com × e adicione um com **+**. Pelo menos uma guia deve permanecer. |
| **Widget** | Uma **única consulta**, uma **fórmula** ou um **texto** (uma observação ou um cabeçalho de seção recolhível). |
| **Variável** | Um seletor acima da grade. Seu valor é inserido como `{{name}}`. |
| **Seletor de hora** | A janela de cada widget no painel. |
