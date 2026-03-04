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

### 1.b (0,5) array + condición adicional
Escribir unha consulta que devolva as películas que:
- teñan o xénero `"Comedy"` no array `genres`
- e que o campo `runtime` sexa maior ou igual a 90

Debe devolverse só:
- `title`
- `genres`
- `runtime`

Ordenar por `runtime` descendente e limitar a 10 resultados.

---


## Exercicio 2. mongodb — actualizacións (2 puntos)

### inserción

### 2.a (0,33)
Inserir **un** documento novo co seguinte contido mínimo:
- `user="u99"`, `type="purchase"`, `app="mobile"`, `value=20`, `geo.provincia="pontevedra"`, `tags=["vip","promo"]`
- `ts` debe ser a data actual (usar `new Date()`)

### 2.b (0,33)
Inserir **varios** documentos (mínimo 3) nunha soa operación, con distintos valores de `type` e `value`.

### borrado

### 2.c (0,33)
Borrar **un** documento calquera que cumpra `type="click"` e `value < 2`.

### 2.d (0,33)
Borrar **todos** os documentos onde `app="web"` e `geo.provincia="pontevedra"`.

### modificación

### 2.e (0,33)
Actualizar **un** documento onde `user="u1"` para:
- engadir un campo `plan="pro"`
- incrementar `value` en 1

### 2.f (0,35)
Actualizar **varios** documentos onde `type="click"` para:
- engadir a etiqueta `"reviewed"` ao array `tags` (sen duplicados)
- e establecer `app="web"` (se xa está, non pasa nada)

---

## Exercicio 3 — mongodb: aggregation pipeline sobre unha colección de mostra (4 apartados) (2 puntos)

Neste exercicio empregarase a mesma colección de mostra do exercicio 1 (recoméndase `sample_mflix.movies`) para construír un pipeline de agregación.

### 3.a (0,5) $match
Construír un pipeline que filtre películas onde:
- `year` estea entre 2000 e 2015 (incluídos)
- e `imdb.rating` non sexa nulo

### 3.b (0,5) $group
Engadir unha fase `$group` para agrupar por `year` e calcular:
- `num_movies`: número de películas dese ano
- `avg_rating`: media de `imdb.rating`

### 3.c (0,5) $sort + $limit
Engadir fases para:
- ordenar por `avg_rating` descendente
- limitar o resultado a 5 anos

### 3.d (0,5) $project (formato final)
Modificar o pipeline para que a saída teña exactamente estes campos:
- `year`
- `num_movies`
- `avg_rating` redondeado a 2 decimais

Non debe aparecer `_id` na saída.

---

## Exercicio 4. opensearch — query dsl (2 puntos)

Contexto: existe un índice chamado `events`.
```http
DELETE eventos

PUT eventos
{
  "mappings": {
    "properties": {
      "timestamp":   { "type": "date" },
      "mensaxe":     { "type": "text" },
      "servizo":     { "type": "keyword" },
      "nivel":       { "type": "keyword" },
      "usuario":     { "type": "keyword" },
      "ip":          { "type": "ip" },
      "endpoint":    { "type": "keyword" },
      "status":      { "type": "integer" },
      "latency_ms":  { "type": "integer" },
      "request_id":  { "type": "keyword" }
    }
  }
}
```
```http
POST _bulk
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:00:00","servizo":"auth","nivel":"info","usuario":"ana","ip":"192.168.1.10","endpoint":"/login","status":200,"latency_ms":42,"request_id":"r-0001","mensaxe":"login correcto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:01:00","servizo":"auth","nivel":"warn","usuario":"ana","ip":"192.168.1.11","endpoint":"/login","status":401,"latency_ms":55,"request_id":"r-0002","mensaxe":"login fallido: credenciais incorrectas" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:02:00","servizo":"api","nivel":"info","usuario":"pepe","ip":"10.0.0.5","endpoint":"/v1/items","status":200,"latency_ms":63,"request_id":"r-0003","mensaxe":"consulta de items correcta" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:03:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":900,"request_id":"r-0004","mensaxe":"timeout na lectura da base de datos" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:04:00","servizo":"payments","nivel":"info","usuario":"laura","ip":"172.16.0.2","endpoint":"/pay","status":201,"latency_ms":120,"request_id":"r-0005","mensaxe":"pago procesado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:05:00","servizo":"api","nivel":"warn","usuario":"martin","ip":"10.0.0.6","endpoint":"/v1/items","status":429,"latency_ms":20,"request_id":"r-0006","mensaxe":"rate limit excedido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:06:00","servizo":"auth","nivel":"info","usuario":"noa","ip":"192.168.1.12","endpoint":"/token","status":200,"latency_ms":35,"request_id":"r-0007","mensaxe":"token emitido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:07:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":650,"request_id":"r-0008","mensaxe":"latencia alta en escritura" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:08:00","servizo":"payments","nivel":"error","usuario":"laura","ip":"172.16.0.2","endpoint":"/pay","status":502,"latency_ms":780,"request_id":"r-0009","mensaxe":"fallo do provedor de pagamento" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T08:09:00","servizo":"api","nivel":"info","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/orders","status":200,"latency_ms":88,"request_id":"r-0010","mensaxe":"listado de pedidos" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:00:00","servizo":"auth","nivel":"info","usuario":"pepe","ip":"192.168.1.13","endpoint":"/login","status":200,"latency_ms":48,"request_id":"r-0011","mensaxe":"login correcto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:01:00","servizo":"auth","nivel":"warn","usuario":"pepe","ip":"192.168.1.14","endpoint":"/login","status":401,"latency_ms":52,"request_id":"r-0012","mensaxe":"login fallido: password incorrecto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:02:00","servizo":"api","nivel":"info","usuario":"noa","ip":"10.0.0.8","endpoint":"/v1/items","status":200,"latency_ms":71,"request_id":"r-0013","mensaxe":"consulta items ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:03:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":1100,"request_id":"r-0014","mensaxe":"deadlock detectado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:04:00","servizo":"payments","nivel":"warn","usuario":"martin","ip":"172.16.0.3","endpoint":"/refund","status":409,"latency_ms":210,"request_id":"r-0015","mensaxe":"reembolso duplicado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:05:00","servizo":"api","nivel":"info","usuario":"laura","ip":"10.0.0.9","endpoint":"/v1/orders","status":200,"latency_ms":95,"request_id":"r-0016","mensaxe":"consulta pedidos ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:06:00","servizo":"auth","nivel":"error","usuario":"system","ip":"192.168.1.1","endpoint":"/token","status":500,"latency_ms":600,"request_id":"r-0017","mensaxe":"erro ao asinar token" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:07:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":720,"request_id":"r-0018","mensaxe":"cola de escritura saturada" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:08:00","servizo":"payments","nivel":"info","usuario":"ana","ip":"172.16.0.4","endpoint":"/pay","status":201,"latency_ms":140,"request_id":"r-0019","mensaxe":"pago procesado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T09:09:00","servizo":"api","nivel":"warn","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/items","status":404,"latency_ms":30,"request_id":"r-0020","mensaxe":"recurso non atopado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:00:00","servizo":"auth","nivel":"info","usuario":"laura","ip":"192.168.1.15","endpoint":"/login","status":200,"latency_ms":44,"request_id":"r-0021","mensaxe":"login correcto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:01:00","servizo":"api","nivel":"info","usuario":"martin","ip":"10.0.0.6","endpoint":"/v1/items","status":200,"latency_ms":67,"request_id":"r-0022","mensaxe":"consulta items correcta" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:02:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":980,"request_id":"r-0023","mensaxe":"timeout conexión DB" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:03:00","servizo":"payments","nivel":"error","usuario":"pepe","ip":"172.16.0.5","endpoint":"/pay","status":502,"latency_ms":850,"request_id":"r-0024","mensaxe":"gateway error no pagamento" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:04:00","servizo":"api","nivel":"warn","usuario":"noa","ip":"10.0.0.8","endpoint":"/v1/orders","status":429,"latency_ms":22,"request_id":"r-0025","mensaxe":"rate limit excedido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:05:00","servizo":"auth","nivel":"warn","usuario":"noa","ip":"192.168.1.12","endpoint":"/token","status":403,"latency_ms":40,"request_id":"r-0026","mensaxe":"token denegado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:06:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":690,"request_id":"r-0027","mensaxe":"latencia alta en commit" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:07:00","servizo":"payments","nivel":"info","usuario":"martin","ip":"172.16.0.3","endpoint":"/refund","status":200,"latency_ms":160,"request_id":"r-0028","mensaxe":"reembolso procesado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:08:00","servizo":"api","nivel":"info","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/items","status":200,"latency_ms":75,"request_id":"r-0029","mensaxe":"consulta items ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-10T10:09:00","servizo":"auth","nivel":"error","usuario":"system","ip":"192.168.1.1","endpoint":"/login","status":500,"latency_ms":520,"request_id":"r-0030","mensaxe":"erro interno en autenticación" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:00:00","servizo":"api","nivel":"info","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/items","status":200,"latency_ms":66,"request_id":"r-0031","mensaxe":"consulta items ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:01:00","servizo":"api","nivel":"warn","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/items","status":404,"latency_ms":28,"request_id":"r-0032","mensaxe":"recurso non atopado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:02:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":1050,"request_id":"r-0033","mensaxe":"erro de conexión DB" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:03:00","servizo":"payments","nivel":"info","usuario":"laura","ip":"172.16.0.2","endpoint":"/pay","status":201,"latency_ms":135,"request_id":"r-0034","mensaxe":"pago procesado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:04:00","servizo":"auth","nivel":"warn","usuario":"pepe","ip":"192.168.1.14","endpoint":"/login","status":401,"latency_ms":58,"request_id":"r-0035","mensaxe":"login fallido: credenciais incorrectas" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:05:00","servizo":"auth","nivel":"info","usuario":"pepe","ip":"192.168.1.13","endpoint":"/login","status":200,"latency_ms":46,"request_id":"r-0036","mensaxe":"login correcto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:06:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":740,"request_id":"r-0037","mensaxe":"cola de escritura saturada" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:07:00","servizo":"payments","nivel":"error","usuario":"martin","ip":"172.16.0.3","endpoint":"/refund","status":502,"latency_ms":820,"request_id":"r-0038","mensaxe":"gateway error no reembolso" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:08:00","servizo":"api","nivel":"warn","usuario":"noa","ip":"10.0.0.8","endpoint":"/v1/orders","status":429,"latency_ms":25,"request_id":"r-0039","mensaxe":"rate limit excedido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T08:09:00","servizo":"auth","nivel":"error","usuario":"system","ip":"192.168.1.1","endpoint":"/token","status":500,"latency_ms":610,"request_id":"r-0040","mensaxe":"erro ao asinar token" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:00:00","servizo":"api","nivel":"info","usuario":"laura","ip":"10.0.0.9","endpoint":"/v1/orders","status":200,"latency_ms":92,"request_id":"r-0041","mensaxe":"listado de pedidos" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:01:00","servizo":"api","nivel":"info","usuario":"martin","ip":"10.0.0.6","endpoint":"/v1/items","status":200,"latency_ms":70,"request_id":"r-0042","mensaxe":"consulta items correcta" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:02:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":990,"request_id":"r-0043","mensaxe":"deadlock detectado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:03:00","servizo":"payments","nivel":"warn","usuario":"ana","ip":"172.16.0.4","endpoint":"/pay","status":409,"latency_ms":230,"request_id":"r-0044","mensaxe":"pago duplicado detectado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:04:00","servizo":"auth","nivel":"info","usuario":"noa","ip":"192.168.1.12","endpoint":"/token","status":200,"latency_ms":34,"request_id":"r-0045","mensaxe":"token emitido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:05:00","servizo":"auth","nivel":"warn","usuario":"noa","ip":"192.168.1.12","endpoint":"/token","status":403,"latency_ms":38,"request_id":"r-0046","mensaxe":"token denegado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:06:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":710,"request_id":"r-0047","mensaxe":"latencia alta en commit" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:07:00","servizo":"payments","nivel":"info","usuario":"pepe","ip":"172.16.0.5","endpoint":"/pay","status":201,"latency_ms":150,"request_id":"r-0048","mensaxe":"pago procesado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:08:00","servizo":"api","nivel":"warn","usuario":"ana","ip":"10.0.0.7","endpoint":"/v1/items","status":429,"latency_ms":21,"request_id":"r-0049","mensaxe":"rate limit excedido" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T09:09:00","servizo":"auth","nivel":"error","usuario":"system","ip":"192.168.1.1","endpoint":"/login","status":500,"latency_ms":540,"request_id":"r-0050","mensaxe":"erro interno en autenticación" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:00:00","servizo":"api","nivel":"info","usuario":"noa","ip":"10.0.0.8","endpoint":"/v1/items","status":200,"latency_ms":64,"request_id":"r-0051","mensaxe":"consulta items ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:01:00","servizo":"api","nivel":"warn","usuario":"martin","ip":"10.0.0.6","endpoint":"/v1/orders","status":404,"latency_ms":29,"request_id":"r-0052","mensaxe":"recurso non atopado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:02:00","servizo":"db","nivel":"error","usuario":"system","ip":"10.0.0.10","endpoint":"db:read","status":500,"latency_ms":1020,"request_id":"r-0053","mensaxe":"timeout conexión DB" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:03:00","servizo":"payments","nivel":"error","usuario":"laura","ip":"172.16.0.2","endpoint":"/pay","status":502,"latency_ms":830,"request_id":"r-0054","mensaxe":"gateway error no pagamento" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:04:00","servizo":"auth","nivel":"info","usuario":"ana","ip":"192.168.1.10","endpoint":"/login","status":200,"latency_ms":41,"request_id":"r-0055","mensaxe":"login correcto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:05:00","servizo":"auth","nivel":"warn","usuario":"ana","ip":"192.168.1.11","endpoint":"/login","status":401,"latency_ms":57,"request_id":"r-0056","mensaxe":"login fallido: password incorrecto" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:06:00","servizo":"payments","nivel":"warn","usuario":"pepe","ip":"172.16.0.5","endpoint":"/refund","status":409,"latency_ms":205,"request_id":"r-0057","mensaxe":"reembolso duplicado" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:07:00","servizo":"db","nivel":"warn","usuario":"system","ip":"10.0.0.10","endpoint":"db:write","status":503,"latency_ms":735,"request_id":"r-0058","mensaxe":"cola de escritura saturada" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:08:00","servizo":"api","nivel":"info","usuario":"laura","ip":"10.0.0.9","endpoint":"/v1/orders","status":200,"latency_ms":90,"request_id":"r-0059","mensaxe":"consulta pedidos ok" }
{ "index": { "_index": "eventos" } }
{ "timestamp":"2025-01-11T10:09:00","servizo":"auth","nivel":"error","usuario":"system","ip":"192.168.1.1","endpoint":"/token","status":500,"latency_ms":605,"request_id":"r-0060","mensaxe":"erro ao asinar token" }
```


### 4.a (0,5) match + filtro
Construír unha consulta Query DSL que devolva documentos onde:
- `type` sexa `"click"`
- `geo.provincia` sexa `"ourense"`
Debe devolver só: `ts`, `user`, `value`.

### 4.b (0,5) rango + ordenación
Construír unha consulta que devolva documentos con:
- `value` entre 2 e 5
Ordenar por `ts` descendente e limitar a 5.

### 4.c (0,5) agregación terms
Construír unha consulta con agregación que devolva:
- número de documentos por `type` (terms agg)
Non é necesario devolver hits (pódese poñer `size: 0`).

### 4.d (0,5) bool con must + must_not
Construír unha consulta bool:
- must: `app="web"`
- must_not: `type="purchase"`
- filtro: `geo.provincia="pontevedra"`

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

---

### Criterios de avaliación (2 puntos)
- (0,75) docker-compose con `--configdb` correcto e placeholders ben aplicados
- (0,75) scripts de init correctos (rs.initiate e sh.addShard co formato correcto)
- (0,5) habilitar sharding en `bd_exam.events` e evidencias (sh.status/listShards/rs.status)
---

## anexos (material dado ao alumnado)

### docker-compose.yml (base)
Partir do material dispoñible no (repositorio)[https://github.com/adbgonzalez/NoSQL/tree/main/mongodb/replicaset-sharded]