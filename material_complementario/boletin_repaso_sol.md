# exame / boletín — mongodb + opensearch (10 puntos)

## indicacións xerais
- responder en formato **comandos** (mongosh / curl / yaml) e unha liña curta explicando que fai cada apartado.
- cando se queira usar unha base de datos e colección, asumir:
  - bd: `bd_exam`
  - colección: `events`
- documento tipo (exemplo) en `events`:

    { "_id": 1, "ts": ISODate("2026-03-04T10:00:00Z"), "user": "u1", "type": "click", "app": "web", "value": 3, "tags": ["promo"], "geo": { "provincia": "ourense" } }

---

## Exercicio 1 — mongodb: consultas básicas sobre unha colección de mostra (2 apartados) (1 punto)

Neste exercicio empregarase unha colección de mostra de MongoDB para practicar consultas básicas.

Colección de mostra (a escoller unha das dúas opcións):
- Opción A: base de datos `sample_mflix`, colección `movies`
- Opción B: base de datos `sample_training`, colección `companies`

En todos os apartados debe indicarse explicitamente que colección se está a usar (A ou B).

### 1.a (0,5) filtros + proxección
Escribir unha consulta que devolva as películas que:
- teñan `year` maior ou igual a 2010
- e `imdb.rating` maior ou igual a 7.5

Debe devolverse só:
- `title`
- `year`
- `imdb.rating`

e ordenarse por `imdb.rating` descendente, limitando a 10 resultados.

#### solución 1.a
    use sample_mflix
    db.movies.find(
      { year: { $gte: 2010 }, "imdb.rating": { $gte: 7.5 } },
      { _id: 0, title: 1, year: 1, "imdb.rating": 1 }
    ).sort({ "imdb.rating": -1 }).limit(10)

### 1.b (0,5) array + condición adicional
Escribir unha consulta que devolva as películas que:
- teñan o xénero `"Comedy"` no array `genres`
- e que o campo `runtime` sexa maior ou igual a 90

Debe devolverse só:
- `title`
- `genres`
- `runtime`

Ordenar por `runtime` descendente e limitar a 10 resultados.

#### solución 1.b
    use sample_mflix
    db.movies.find(
      { genres: "Comedy", runtime: { $gte: 90 } },
      { _id: 0, title: 1, genres: 1, runtime: 1 }
    ).sort({ runtime: -1 }).limit(10)

---

## Exercicio 2. mongodb — actualizacións (2 puntos)

### inserción

### 2.a (0,33)
Inserir **un** documento novo co seguinte contido mínimo:
- `user="u99"`, `type="purchase"`, `app="mobile"`, `value=20`, `geo.provincia="pontevedra"`, `tags=["vip","promo"]`
- `ts` debe ser a data actual (usar `new Date()`)

#### solución 2.a
    use bd_exam
    db.events.insertOne({
      ts: new Date(),
      user: "u99",
      type: "purchase",
      app: "mobile",
      value: 20,
      tags: ["vip", "promo"],
      geo: { provincia: "pontevedra" }
    })

### 2.b (0,33)
Inserir **varios** documentos (mínimo 3) nunha soa operación, con distintos valores de `type` e `value`.

#### solución 2.b
    use bd_exam
    db.events.insertMany([
      { ts: new Date(), user: "u2", type: "click",    app: "web",    value: 1, tags: ["promo"], geo: { provincia: "ourense" } },
      { ts: new Date(), user: "u3", type: "view",     app: "web",    value: 2, tags: [],        geo: { provincia: "ourense" } },
      { ts: new Date(), user: "u4", type: "purchase", app: "mobile", value: 9, tags: ["vip"],   geo: { provincia: "lugo" } }
    ])

### borrado

### 2.c (0,33)
Borrar **un** documento calquera que cumpra `type="click"` e `value < 2`.

#### solución 2.c
    use bd_exam
    db.events.deleteOne({ type: "click", value: { $lt: 2 } })

### 2.d (0,33)
Borrar **todos** os documentos onde `app="web"` e `geo.provincia="pontevedra"`.

#### solución 2.d
    use bd_exam
    db.events.deleteMany({ app: "web", "geo.provincia": "pontevedra" })

### modificación

### 2.e (0,33)
Actualizar **un** documento onde `user="u1"` para:
- engadir un campo `plan="pro"`
- incrementar `value` en 1

#### solución 2.e
    use bd_exam
    db.events.updateOne(
      { user: "u1" },
      { $set: { plan: "pro" }, $inc: { value: 1 } }
    )

### 2.f (0,35)
Actualizar **varios** documentos onde `type="click"` para:
- engadir a etiqueta `"reviewed"` ao array `tags` (sen duplicados)
- e establecer `app="web"` (se xa está, non pasa nada)

#### solución 2.f
    use bd_exam
    db.events.updateMany(
      { type: "click" },
      { $addToSet: { tags: "reviewed" }, $set: { app: "web" } }
    )

---

## Exercicio 3 — mongodb: aggregation pipeline sobre unha colección de mostra (4 apartados) (2 puntos)

Neste exercicio empregarase a mesma colección de mostra do exercicio 1 (recoméndase `sample_mflix.movies`) para construír un pipeline de agregación.

### 3.a (0,5) $match
Construír un pipeline que filtre películas onde:
- `year` estea entre 2000 e 2015 (incluídos)
- e `imdb.rating` non sexa nulo

#### solución 3.a
    use sample_mflix
    db.movies.aggregate([
      { $match: { year: { $gte: 2000, $lte: 2015 }, "imdb.rating": { $ne: null } } }
    ])

### 3.b (0,5) $group
Engadir unha fase `$group` para agrupar por `year` e calcular:
- `num_movies`: número de películas dese ano
- `avg_rating`: media de `imdb.rating`

#### solución 3.b
    use sample_mflix
    db.movies.aggregate([
      { $match: { year: { $gte: 2000, $lte: 2015 }, "imdb.rating": { $ne: null } } },
      {
        $group: {
          _id: "$year",
          num_movies: { $sum: 1 },
          avg_rating: { $avg: "$imdb.rating" }
        }
      }
    ])

### 3.c (0,5) $sort + $limit
Engadir fases para:
- ordenar por `avg_rating` descendente
- limitar o resultado a 5 anos

#### solución 3.c
    use sample_mflix
    db.movies.aggregate([
      { $match: { year: { $gte: 2000, $lte: 2015 }, "imdb.rating": { $ne: null } } },
      {
        $group: {
          _id: "$year",
          num_movies: { $sum: 1 },
          avg_rating: { $avg: "$imdb.rating" }
        }
      },
      { $sort: { avg_rating: -1 } },
      { $limit: 5 }
    ])

### 3.d (0,5) $project (formato final)
Modificar o pipeline para que a saída teña exactamente estes campos:
- `year`
- `num_movies`
- `avg_rating` redondeado a 2 decimais

Non debe aparecer `_id` na saída.

#### solución 3.d
    use sample_mflix
    db.movies.aggregate([
      { $match: { year: { $gte: 2000, $lte: 2015 }, "imdb.rating": { $ne: null } } },
      {
        $group: {
          _id: "$year",
          num_movies: { $sum: 1 },
          avg_rating: { $avg: "$imdb.rating" }
        }
      },
      { $sort: { avg_rating: -1 } },
      { $limit: 5 },
      {
        $project: {
          _id: 0,
          year: "$_id",
          num_movies: 1,
          avg_rating: { $round: ["$avg_rating", 2] }
        }
      }
    ])

---

## Exercicio 4. opensearch — query dsl (2 puntos)

Contexto: existe un índice chamado `events`.

### 4.a (0,5) match + filtro
Construír unha consulta Query DSL que devolva documentos onde:
- `type` sexa `"click"`
- `geo.provincia` sexa `"ourense"`
Debe devolver só: `ts`, `user`, `value`.

#### solución 4.a
    GET events/_search
    {
      "_source": ["ts", "user", "value"],
      "query": {
        "bool": {
          "filter": [
            { "term": { "type.keyword": "click" } },
            { "term": { "geo.provincia.keyword": "ourense" } }
          ]
        }
      }
    }

### 4.b (0,5) rango + ordenación
Construír unha consulta que devolva documentos con:
- `value` entre 2 e 5
Ordenar por `ts` descendente e limitar a 5.

#### solución 4.b
    GET events/_search
    {
      "size": 5,
      "sort": [{ "ts": { "order": "desc" } }],
      "query": {
        "range": {
          "value": { "gte": 2, "lte": 5 }
        }
      }
    }

### 4.c (0,5) agregación terms
Construír unha consulta con agregación que devolva:
- número de documentos por `type` (terms agg)
Non é necesario devolver hits (pódese poñer `size: 0`).

#### solución 4.c
    GET events/_search
    {
      "size": 0,
      "aggs": {
        "by_type": {
          "terms": { "field": "type.keyword" }
        }
      }
    }

### 4.d (0,5) bool con must + must_not
Construír unha consulta bool:
- must: `app="web"`
- must_not: `type="purchase"`
- filtro: `geo.provincia="pontevedra"`

#### solución 4.d
    GET events/_search
    {
      "query": {
        "bool": {
          "must": [
            { "term": { "app.keyword": "web" } }
          ],
          "must_not": [
            { "term": { "type.keyword": "purchase" } }
          ],
          "filter": [
            { "term": { "geo.provincia.keyword": "pontevedra" } }
          ]
        }
      }
    }

---

## Exercicio 5. data prepper — construír un pipeline (1 punto)

Obxectivo: definir un pipeline sinxelo en Data Prepper que:
- lea de Kafka (topic `api_events`)
- parse o contido como json
- escriba en OpenSearch no índice `events`

### 5.a (1,0)
Escribir un `pipelines.yaml` mínimo con:
- `source`: kafka
- `bootstra_servers`: "kafka-9092"
- `topic`: "logs"
- `sink`: opensearch

#### solución 5.a
    entry-pipeline:
      source:
        kafka:
          bootstrap_servers:
            - "kafka:9092"
          topics:
            - name: "api_events"
          # opcional (se se usa en clase):
          # group_id: "data-prepper"
          # auto_offset_reset: "earliest"
      processor:
        - parse_json: {}
      sink:
        - opensearch:
            hosts: [ "http://opensearch:9200" ]
            index: "events"
            username: "admin"
            password: "admin"

---

## Exercicio 6 — sharding e replicación en MongoDB: completar e arranxar o cluster (2 puntos)

Neste exercicio entrégase un cluster sharded en Docker (router + config server + 2 shards con 2 réplicas cada un), pero está **incompleto** a propósito.

O obxectivo é demostrar que se entende:
- que compoñentes existen (mongos, configsvr, shardsvr)
- como se inicializa a replicación (rs.initiate)
- como se engaden shards ao router (sh.addShard)
- e como se habilita sharding nunha base de datos e colección (sh.enableSharding / sh.shardCollection)

### Material entregado
- `docker-compose.yml`
- `configserver-init.js`
- `shard1-init.js`
- `shard2-init.js`
- `router-init.js`

---

### 6.a (0,75) completar o docker-compose (placeholders)
Modificar o `docker-compose.yml` para substituír por placeholders estes valores:

1) Portos expostos:
- `router1` debe expoñer `${ROUTER_PORT}:27017`

2) nomes dos replica sets:
- config server: `${CFG_REPLSET}`
- shard 1: `${SHARD1_REPLSET}`
- shard 2: `${SHARD2_REPLSET}`

3) conexión do router ao config server:
- a opción `--configdb` do `mongos` debe quedar como:

  `${CFG_REPLSET}/configsvr1:27017`

(é dicir, inclúe o nome do replset e os hosts)

Entregar o `docker-compose.yml` modificado.

#### solución 6.a (fragmentos)
    services:
      router1:
        ports:
          - ${ROUTER_PORT}:27017
        entrypoint:
          - /usr/bin/mongos
          - --port
          - "27017"
          - --configdb
          - ${CFG_REPLSET}/configsvr1:27017
          - --bind_ip_all

      configsvr1:
        entrypoint:
          - /usr/bin/mongod
          - --port
          - "27017"
          - --configsvr
          - --replSet
          - ${CFG_REPLSET}
          - --bind_ip_all

      mongo-shard1a:
        entrypoint:
          - /usr/bin/mongod
          - --port
          - "27017"
          - --shardsvr
          - --bind_ip_all
          - --replSet
          - ${SHARD1_REPLSET}

      mongo-shard1b:
        entrypoint:
          - /usr/bin/mongod
          - --port
          - "27017"
          - --shardsvr
          - --bind_ip_all
          - --replSet
          - ${SHARD1_REPLSET}

      mongo-shard2a:
        entrypoint:
          - /usr/bin/mongod
          - --port
          - "27017"
          - --shardsvr
          - --bind_ip_all
          - --replSet
          - ${SHARD2_REPLSET}

      mongo-shard2b:
        entrypoint:
          - /usr/bin/mongod
          - --port
          - "27017"
          - --shardsvr
          - --bind_ip_all
          - --replSet
          - ${SHARD2_REPLSET}

---

### 6.b (0,75) completar os scripts de init (replica sets + addShard)
Modificar os scripts para que empreguen os placeholders anteriores.

1) `configserver-init.js`:
- debe iniciar o replset co `_id: "${CFG_REPLSET}"`
- debe manter `configsvr: true`

2) `shard1-init.js` e `shard2-init.js`:
- `_id` debe ser `${SHARD1_REPLSET}` e `${SHARD2_REPLSET}`
- deben listar 2 membros cada un (a e b)

3) `router-init.js`:
- debe engadir **cada shard unha soa vez**, usando o formato correcto:
  - `sh.addShard("${SHARD1_REPLSET}/mongo-shard1a:27017,mongo-shard1b:27017")`
  - `sh.addShard("${SHARD2_REPLSET}/mongo-shard2a:27017,mongo-shard2b:27017")`

Entregar os 4 ficheiros `.js` modificados.

#### solución 6.b
    // configserver-init.js
    rs.initiate({
      _id: "${CFG_REPLSET}",
      configsvr: true,
      version: 1,
      members: [
        { _id: 0, host: "configsvr1:27017" }
      ]
    })

    // shard1-init.js
    rs.initiate({
      _id: "${SHARD1_REPLSET}",
      version: 1,
      members: [
        { _id: 0, host: "mongo-shard1a:27017" },
        { _id: 1, host: "mongo-shard1b:27017" }
      ]
    })

    // shard2-init.js
    rs.initiate({
      _id: "${SHARD2_REPLSET}",
      version: 1,
      members: [
        { _id: 0, host: "mongo-shard2a:27017" },
        { _id: 1, host: "mongo-shard2b:27017" }
      ]
    })

    // router-init.js
    sh.addShard("${SHARD1_REPLSET}/mongo-shard1a:27017,mongo-shard1b:27017")
    sh.addShard("${SHARD2_REPLSET}/mongo-shard2a:27017,mongo-shard2b:27017")

---

### 6.c (0,5) habilitar sharding nunha base de datos e colección
Engadir (no `router-init.js` ou nun novo script `enable-sharding.js`) os comandos para:

1) habilitar sharding na base de datos:

- base de datos: `bd_exam`

2) shardear a colección:

- colección: `bd_exam.events`
- clave de sharding: `user` (hashed)

É dicir, deben aparecer comandos equivalentes a:
- `sh.enableSharding("bd_exam")`
- `sh.shardCollection("bd_exam.events", { user: "hashed" })`

Entregar o script modificado/creado.

#### solución 6.c
    // enable-sharding.js (ou ao final de router-init.js)
    sh.enableSharding("bd_exam")
    sh.shardCollection("bd_exam.events", { user: "hashed" })

---

### Evidencias a entregar (obrigatorias)
- saída (ou captura) dos seguintes comandos executados contra o router:

1) estado do sharding:
- `sh.status()`

2) replica sets:
- en `configsvr1`: `rs.status()`
- en `mongo-shard1a`: `rs.status()`
- en `mongo-shard2a`: `rs.status()`

3) shards rexistrados:
- `db.adminCommand({ listShards: 1 })`

#### solución (comandos de comprobación)
    # conectar ao router
    mongosh --port ${ROUTER_PORT}

    sh.status()
    db.adminCommand({ listShards: 1 })

    # comprobar replicación (executar dentro de cada contedor, por exemplo)
    # docker exec -it configsvr1 mongosh --eval "rs.status()"
    # docker exec -it mongo-shard1a mongosh --eval "rs.status()"
    # docker exec -it mongo-shard2a mongosh --eval "rs.status()"

---

### Criterios de avaliación (2 puntos)
- (0,75) docker-compose con `--configdb` correcto e placeholders ben aplicados
- (0,75) scripts de init correctos (rs.initiate e sh.addShard co formato correcto)
- (0,5) habilitar sharding en `bd_exam.events` e evidencias (sh.status/listShards/rs.status)
---

## anexos (material dado ao alumnado)

### docker-compose.yml (base)
Partir do material dispoñible no (repositorio)[https://github.com/adbgonzalez/NoSQL/tree/main/mongodb/replicaset-sharded]