# laboratorio: agregacións en mongodb 
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

### solución

```javascript
db.movies.aggregate([
  { $project: { _id: 0, title: 1, year: 1, runtime: 1 } },
  { $limit: 10 }
])
```

---

## exercicio 2: filtrado con $match sobre un rango
**obxectivo:** aplicar un filtro sinxelo e manter poucos campos.

- recuperar películas estreadas entre 1990 e 1999 (incluídos)
- amosar:
  - title
  - year

### solución

```javascript
db.movies.aggregate([
  { $match: { year: { $gte: 1990, $lte: 1999 } } },
  { $project: { _id: 0, title: 1, year: 1 } }
])
```

---

## exercicio 3: ordenación e limitación
**obxectivo:** combinar $match, $sort e $limit.

- recuperar as 5 películas máis longas (runtime maior)
- amosar:
  - title
  - runtime
- ignorar documentos sen runtime numérico

### solución
```javascript
db.movies.aggregate([
  { $match: { runtime: { $type: "number" } } },
  { $sort: { runtime: -1 } },
  { $limit: 5 },
  { $project: { _id: 0, title: 1, runtime: 1 } }
])
```
---

## exercicio 4: campo calculado con $project
**obxectivo:** crear un campo calculado.

- crear runtime_hours = runtime / 60
- amosar:
  - title
  - runtime
  - runtime_hours
- devolver 10 resultados

### solución
```javascript
db.movies.aggregate([
  { $match: { runtime: { $type: "number" } } },
  { $project: {
      _id: 0,
      title: 1,
      runtime: 1,
      runtime_hours: { $round: [{ $divide: ["$runtime", 60] }, 2] }
  }},
  { $limit: 10 }
])

```
---

## exercicio 5: estatísticas por ano con $group
**obxectivo:** empregar acumuladores.

- para cada ano:
  - número de películas
  - duración media
- só anos con polo menos 50 películas
- ordenar por número de películas desc

### solución
```javascript
db.movies.aggregate([
  { $match: { year: { $type: "number" }, runtime: { $type: "number" } } },
  { $group: {
      _id: "$year",
      count: { $sum: 1 },
      avg_runtime: { $avg: "$runtime" }
  }},
  { $match: { count: { $gte: 50 } } },
  { $sort: { count: -1 } },
  { $project: {
      _id: 0,
      year: "$_id",
      count: 1,
      avg_runtime: { $round: ["$avg_runtime", 2] }
  }}
])
```

---

## exercicio 6: unwind e renomeado
**obxectivo:** traballar con arrays e renomear o campo de agrupamento.

- contar películas por país
- usar $unwind sobre countries
- amosar os 10 países con máis películas

### solución
```javascript
db.movies.aggregate([
  { $match: { countries: { $type: "array" } } },
  { $unwind: "$countries" },
  { $group: { _id: { country: "$countries" }, total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 10 },
  { $project: { _id: 0, country: "$_id.country", total: 1 } }
])
```
---

## exercicio 7: arrays e acumuladores
**obxectivo:** usar $push e traballar con mostras de datos.

- por cada director:
  - total de películas
  - tres títulos de exemplo
- só directores con 10 ou máis películas

### solución
```javascript
db.movies.aggregate([
  { $match: { directors: { $type: "array" }, title: { $type: "string" } } },
  { $unwind: "$directors" },
  { $group: {
      _id: "$directors",
      total: { $sum: 1 },
      titles: { $push: "$title" }
  }},
  { $match: { total: { $gte: 10 } } },
  { $project: {
      _id: 0,
      director: "$_id",
      total: 1,
      sample_titles: { $slice: ["$titles", 3] }
  }},
  { $sort: { total: -1 } }
])
```

---

## exercicio 8: subcampos e ordenación
**obxectivo:** filtrar por imdb.rating.

- rating >= 8.5
- ordenar por rating desc e ano desc
- amosar 10 resultados

### solución
```javascript
db.movies.aggregate([
  { $match: { "imdb.rating": { $type: "number", $gte: 8.5 } } },
  { $sort: { "imdb.rating": -1, year: -1 } },
  { $limit: 10 },
  { $project: {
      _id: 0,
      title: 1,
      year: 1,
      imdb_rating: "$imdb.rating"
  }}
])
```

---

## exercicio 9: lookup con comments
**obxectivo:** unir películas con comentarios.

- amosar título e número de comentarios
- só unha mostra de 10 películas

### solución
```javascript
db.movies.aggregate([
  { $limit: 10 },
  { $lookup: {
      from: "comments",
      localField: "_id",
      foreignField: "movie_id",
      as: "comments_info"
  }},
  { $project: {
      _id: 0,
      title: 1,
      num_comments: { $size: "$comments_info" }
  }}
])
```
---

## exercicio 10: top películas por comentarios
**obxectivo:** join + group.

- obter as 10 películas con máis comentarios

### solución
```javascript
db.comments.aggregate([
  { $group: { _id: "$movie_id", total_comments: { $sum: 1 } } },
  { $sort: { total_comments: -1 } },
  { $limit: 10 },
  { $lookup: {
      from: "movies",
      localField: "_id",
      foreignField: "_id",
      as: "movie"
  }},
  { $unwind: "$movie" },
  { $project: { _id: 0, title: "$movie.title", total_comments: 1 } }
])
```
---

## exercicio 11: $out
**obxectivo:** gardar o resultado nunha nova colección.

- crear top_commented_movies

### solución
```javascript
db.comments.aggregate([
  { $group: { _id: "$movie_id", total_comments: { $sum: 1 } } },
  { $sort: { total_comments: -1 } },
  { $limit: 10 },
  { $lookup: {
      from: "movies",
      localField: "_id",
      foreignField: "_id",
      as: "movie"
  }},
  { $unwind: "$movie" },
  { $project: {
      _id: 0,
      movie_id: "$_id",
      title: "$movie.title",
      total_comments: 1
  }},
  { $out: "top_commented_movies" }
])
```

---

## exercicio 12: $merge
**obxectivo:** actualización incremental.

- recalcular top 50 películas por comentarios
- actualizar ou inserir en top_commented_movies

### solución
```javascript
db.comments.aggregate([
  { $group: { _id: "$movie_id", total_comments: { $sum: 1 } } },
  { $sort: { total_comments: -1 } },
  { $limit: 50 },
  { $lookup: {
      from: "movies",
      localField: "_id",
      foreignField: "_id",
      as: "movie"
  }},
  { $unwind: "$movie" },
  { $project: {
      _id: 0,
      movie_id: "$_id",
      title: "$movie.title",
      total_comments: 1
  }},
  { $merge: {
      into: "top_commented_movies",
      on: "movie_id",
      whenMatched: "replace",
      whenNotMatched: "insert"
  }}
])
```
Comprobación:
```javascript
db.top_commented_movies.find().sort({ total_comments: -1 }).limit(10)
```







