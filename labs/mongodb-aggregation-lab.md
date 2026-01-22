# laboratorio: agregacións en mongodb (incremental) + solucións

## contorno
Empregarase a base de datos **sample_mflix** (coleccións típicas: `movies`, `comments`, `users`, `theaters`).
```javascript
use sample_mflix
```
---

## exercicio 1: primeira toma de contacto con $project
**obxectivo:** amosar só algúns campos e ocultar `_id`.

- recuperar 10 películas amosando só:
  - title
  - year
  - runtime



---

## exercicio 2: filtrado con $match sobre un rango
**obxectivo:** aplicar un filtro sinxelo e manter poucos campos.

- recuperar películas estreadas entre 1990 e 1999 (incluídos)
- amosar:
  - title
  - year

---

## exercicio 3: ordenación e limitación
**obxectivo:** combinar $match, $sort e $limit.

- recuperar as 5 películas máis longas (runtime maior)
- amosar:
  - title
  - runtime
- ignorar documentos sen runtime numérico


---

## exercicio 4: campo calculado con $project
**obxectivo:** crear un campo calculado.

- crear runtime_hours = runtime / 60
- amosar:
  - title
  - runtime
  - runtime_hours
- devolver 10 resultados


---

## exercicio 5: estatísticas por ano con $group
**obxectivo:** empregar acumuladores.

- para cada ano:
  - número de películas
  - duración media
- só anos con polo menos 50 películas
- ordenar por número de películas desc


---

## exercicio 6: unwind e renomeado
**obxectivo:** traballar con arrays e renomear o campo de agrupamento.

- contar películas por país
- usar $unwind sobre countries
- amosar os 10 países con máis películas


---

## exercicio 7: arrays e acumuladores
**obxectivo:** usar $push e traballar con mostras de datos.

- por cada director:
  - total de películas
  - tres títulos de exemplo
- só directores con 10 ou máis películas

---

## exercicio 8: subcampos e ordenación
**obxectivo:** filtrar por imdb.rating.

- rating >= 8.5
- ordenar por rating desc e ano desc
- amosar 10 resultados

---

## exercicio 9: lookup con comments
**obxectivo:** unir películas con comentarios.

- amosar título e número de comentarios
- só unha mostra de 10 películas

---

## exercicio 10: top películas por comentarios
**obxectivo:** join + group.

- obter as 10 películas con máis comentarios

---

## exercicio 11: $out
**obxectivo:** gardar o resultado nunha nova colección.

- crear top_commented_movies

---

## exercicio 12: $merge
**obxectivo:** actualización incremental.

- recalcular top 50 películas por comentarios
- actualizar ou inserir en top_commented_movies






