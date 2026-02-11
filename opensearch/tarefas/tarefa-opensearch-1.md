## Boletín de exercicios (10) — Query DSL en OpenSearch (de menos a máis)

Obxectivo: practicar principalmente `term`, `match`, `range` e introducir `bool` de forma progresiva.  
Só 1 exercicio con agregación.

---

### Exercicio 1 — Eventos `error` (term simple)

Enunciado: Devolver só eventos con `nivel=error`.

### Exercicio 2 — Eventos do servizo `payments` (term simple)

Enunciado: Devolver só eventos con `servizo=payments`.

### Exercicio 3 — Full-text: mensaxes con “deadlock” (match simple)

Enunciado: Buscar documentos cuxa `mensaxe` conteña “deadlock”.


### Exercicio 4 — Latencia superior a 800 ms (range simple)

Enunciado: Devolver eventos con `latency_ms > 800`.

### Exercicio 5 — Status menor que 400 (range simple)

Enunciado: Devolver eventos con `status < 400`.

### Exercicio 6 — Servizo `auth` con `status=401` (primeiro bool)

Enunciado: Buscar eventos do servizo `auth` con `status=401`.

### Exercicio 7 — Eventos `warn` do servizo `db` (bool con dous termos)

Enunciado: Buscar eventos con `nivel=warn` e `servizo=db`.

### Exercicio 8 — Texto “timeout” só en `db` (bool con match + filter)

Enunciado: Buscar mensaxes que conteñan “timeout” pero só no servizo `db`.

### Exercicio 9 — Intervalo temporal + servizo (bool con range)

Enunciado: Eventos do servizo `api` entre  
`2025-01-11T08:00:00` e `2025-01-11T09:00:00`.

### Exercicio 10 — Agregación: Top 4 endpoints máis utilizados

Enunciado: Calcular os 4 valores de `endpoint` con máis eventos.

