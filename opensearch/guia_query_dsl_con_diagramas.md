# Guía de Query DSL en OpenSearch (con diagramas mentais)

Esta guía amplía a introdución a **Query DSL** incorporando **diagramas mentais** que axudan a comprender a estrutura das consultas, os tipos de queries e o formato das respostas.

Está pensada como **documento de referencia** para o alumnado unha vez vistos os primeiros exemplos prácticos.

---

## 1. Visión xeral de Query DSL

Query DSL (*Domain Specific Language*) é a linguaxe nativa de OpenSearch para realizar buscas, filtros e análises sobre os datos almacenados. Emprega documentos JSON para describir **que datos se queren recuperar** e **como deben filtrarse ou ordenarse**.

Esta sección explica a estrutura xeral das consultas, os tipos de consultas máis habituais e o formato das respostas devoltas por OpenSearch.

### Diagrama mental: Query DSL no conxunto de OpenSearch

```text
                    ┌──────────────┐
                    │  Query DSL   │
                    │   (JSON)     │
                    └───────┬──────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   Lucene / KQL             SQL                 PPL
   (Discover)        (plugin SQL)        (pipeline logs)
```

Todas estas linguaxes tradúcense internamente a Query DSL.

---

## 2. Estrutura dunha consulta
A maioría das consultas a OpenSearch seguen esta estrutura básica:


```http
GET indice/_search
{
  "query": {
    ...
  }
}
```

### Diagrama mental: estrutura dunha busca

```text
Consulta
│
├── índice
│
├── endpoint (_search)
│
└── corpo (JSON)
    └── query
        └── tipo de query
```

---

## 3. Tipos principais de queries

En Query DSL falamos de **tipos de consulta** e cláusulas, non de operadores clásicos.

De forma simplificada, pódense clasificar en:

- consultas de texto (full-text queries)
- consultas exactas (term-level queries)
- consultas compostas (compound queries)
- consultas por rango (range queries)

### Diagrama mental: clasificación de queries

```text
Query DSL
│
├── Full-text
│   └── match
│
├── Term-level (exactas)
│   └── term
│
├── Range
│   └── range
│
└── Compound
    └── bool
```

---

## 4. Consultas full-text
Estas consultas analizan o texto segundo o analizador configurado (tokenización, minúsculas, etc.).
### `match`

Analiza o texto segundo o analizador do campo.

```http
GET eventos/_search
{
  "query": {
    "match": {
      "mensaxe": "login"
    }
  }
}
```

### Diagrama mental: `match`

```text
Texto buscado
│
└── analizador
    ├── tokenización
    ├── normalización
    └── comparación con índices invertidos
```
- O texto introdúcese libremente
- OpenSearch analiza o contido do campo
- Non require coincidencia exacta

>Uso típico: buscas en mensaxes, descricións, textos longos.

---

## 5. Consultas exactas (term-level)
Estas consultas **non analizan o texto** e buscan coincidencias exactas. Úsanse normalmente sobre campos `keyword`, numéricos ou datas.
### `term`

Busca coincidencia exacta (sen análise).

```http
GET eventos/_search
{
  "query": {
    "term": {
      "nivel": "warn"
    }
  }
}
```

### Diagrama mental: `term`

```text
Valor buscado
│
└── comparación directa
    └── igualdade exacta
```
- O valor debe coincidir exactamente
- Sensible a maiúsculas/minúsculas se o campo é keyword

> Uso típico: filtros por estado, nivel, servizo, usuario, etc.

---

## 6. Consultas por rango
Permiten filtrar valores dentro dun intervalo.
### `range`

```http
GET eventos/_search
{
  "query": {
    "range": {
      "latency_ms": {
        "gt": 700
      }
    }
  }
}
```

### Diagrama mental: `range`

```text
Campo numérico / data
│
├── gt / gte
└── lt / lte
```
Operadores máis habituais:

- gt (greater than)
- gte (greater than or equal)
- lt (less than)
- lte (less than or equal)
 
> Moi usado con campos numéricos e campos temporais (timestamp)
---

## 7. Consultas booleanas (`bool`)

A query máis importante para combinar condicións.

```http
GET eventos/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "mensaxe": "login" } }
      ],
      "filter": [
        { "term": { "servizo": "auth" } }
      ]
    }
  }
}
```

### Cláusulas de `bool`

```text
bool
│
├── must       → debe cumprirse (afecta relevancia)
├── filter     → debe cumprirse (sen scoring)
├── should     → opcional
└── must_not   → exclusión
```

### Diagrama mental: uso recomendado

```text
Consulta realista
│
├── must
│   └── texto (match)
│
└── filter
    ├── termos exactos
    └── rangos
```
Cláusulas principais de bool:

- must → condicións que deben cumprirse (inflúen na relevancia)
- filter → condicións obrigatorias sen cálculo de relevancia
- should → condicións opcionais
- must_not → condicións excluíntes

> Uso recomendado: `must` para texto e `filter` para condicións exactas e rangos.

---

## 8. Control de resultados

### Tamaño, ordenación e páxinas

```text
Resultados
│
├── size   → cantidade devolta
├── from   → desprazamento
└── sort   → orde
```

Exemplo:

```http
GET eventos/_search
{
  "from": 10,
  "size": 10,
  "sort": [
    { "timestamp": { "order": "desc" } }
  ],
  "query": { "match_all": {} }
}
```

Por defecto, OpenSearch devolve 10 documentos.
---

## 9. Formato das respostas
As respostas de OpenSearch tamén se devolven en formato JSON e teñen unha estrutura común.
### Estrutura xeral

```json
{
  "took": 5,
  "timed_out": false,
  "hits": {
    "total": {
      "value": 60
    },
    "hits": [
      {
        "_index": "eventos",
        "_id": "abc123",
        "_score": 1.23,
        "_source": { ... }
      }
    ]
  }
}
```

### Diagrama mental: resposta

```text
Resposta
│
├── took / timed_out
│
└── hits
    ├── total
    │   └── número de documentos
    └── hits[]
        ├── _index
        ├── _id
        ├── _score
        └── _source (documento real)
```
### Campos máis importantes
- took → tempo da consulta en milisegundos
- timed_out → indica se a consulta expirou
- hits.total.value → número total de documentos que cumpren a consulta
- hits.hits → lista de documentos devoltos
- _source → contido real do documento
---

## 10. Interpretación de `_score`

```text
_score
│
├── alto → máis relevante
├── baixo → menos relevante
└── irrelevante en filtros exactos
```

- Só é importante en consultas full-text
- Non debe empregarse como criterio de negocio

---

## 11. Resumo final

```text
Query DSL
│
├── describe consultas con JSON
├── combina filtros e texto
├── devolve documentos + metadatos
└── é a base de todas as linguaxes de OpenSearch
```

Comprender Query DSL permite:
- escribir consultas precisas
- entender KQL, SQL e PPL
- analizar logs e eventos con criterio técnico
