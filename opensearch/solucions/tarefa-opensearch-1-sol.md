## Boletín de exercicios (10) — Query DSL en OpenSearch (de menos a máis)

Obxectivo: practicar principalmente `term`, `match`, `range` e introducir `bool` de forma progresiva.  
Só 1 exercicio con agregación.

---

### Exercicio 1 — Eventos `error` (term simple)

Enunciado: Devolver só eventos con `nivel=error`.

Solución:

    GET eventos/_search
    {
      "query": {
        "term": { "nivel": "error" }
      }
    }

---

### Exercicio 2 — Eventos do servizo `payments` (term simple)

Enunciado: Devolver só eventos con `servizo=payments`.

Solución:

    GET eventos/_search
    {
      "query": {
        "term": { "servizo": "payments" }
      }
    }

---

### Exercicio 3 — Full-text: mensaxes con “deadlock” (match simple)

Enunciado: Buscar documentos cuxa `mensaxe` conteña “deadlock”.

Solución:

    GET eventos/_search
    {
      "query": {
        "match": { "mensaxe": "deadlock" }
      }
    }

---

### Exercicio 4 — Latencia superior a 800 ms (range simple)

Enunciado: Devolver eventos con `latency_ms > 800`.

Solución:

    GET eventos/_search
    {
      "query": {
        "range": {
          "latency_ms": { "gt": 800 }
        }
      }
    }

---

### Exercicio 5 — Status menor que 400 (range simple)

Enunciado: Devolver eventos con `status < 400`.

Solución:

    GET eventos/_search
    {
      "query": {
        "range": {
          "status": { "lt": 400 }
        }
      }
    }

---

### Exercicio 6 — Servizo `auth` con `status=401` (primeiro bool)

Enunciado: Buscar eventos do servizo `auth` con `status=401`.

Solución:

    GET eventos/_search
    {
      "query": {
        "bool": {
          "filter": [
            { "term": { "servizo": "auth" } },
            { "term": { "status": 401 } }
          ]
        }
      }
    }

---

### Exercicio 7 — Eventos `warn` do servizo `db` (bool con dous termos)

Enunciado: Buscar eventos con `nivel=warn` e `servizo=db`.

Solución:

    GET eventos/_search
    {
      "query": {
        "bool": {
          "filter": [
            { "term": { "nivel": "warn" } },
            { "term": { "servizo": "db" } }
          ]
        }
      }
    }

---

### Exercicio 8 — Texto “timeout” só en `db` (bool con match + filter)

Enunciado: Buscar mensaxes que conteñan “timeout” pero só no servizo `db`.

Solución:

    GET eventos/_search
    {
      "query": {
        "bool": {
          "must": [
            { "match": { "mensaxe": "timeout" } }
          ],
          "filter": [
            { "term": { "servizo": "db" } }
          ]
        }
      }
    }

---

### Exercicio 9 — Intervalo temporal + servizo (bool con range)

Enunciado: Eventos do servizo `api` entre  
`2025-01-11T08:00:00` e `2025-01-11T09:00:00`.

Solución:

    GET eventos/_search
    {
      "query": {
        "bool": {
          "filter": [
            { "term": { "servizo": "api" } },
            {
              "range": {
                "timestamp": {
                  "gte": "2025-01-11T08:00:00",
                  "lt":  "2025-01-11T09:00:00"
                }
              }
            }
          ]
        }
      }
    }

---

### Exercicio 10 — Agregación: Top 4 endpoints máis utilizados

Enunciado: Calcular os 4 valores de `endpoint` con máis eventos.

Solución:

    GET eventos/_search
    {
      "size": 0,
      "aggs": {
        "top_endpoints": {
          "terms": {
            "field": "endpoint",
            "size": 4
          }
        }
      }
    }

