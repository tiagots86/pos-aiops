# Execução

## Modelo

**GPT-5.5**

---

## Output

# Resumo executivo

A degradação foi causada principalmente pela **reindexação diária que permaneceu executando por muitas horas além da janela esperada**, elevando continuamente a carga de escrita até saturar os recursos do cluster Elasticsearch.

Essa sobrecarga provocou:

* aumento da taxa de indexação;
* pressão crescente de heap;
* redução da eficiência do query cache;
* throttling de indexação;
* filas de escrita cheias;
* pausas de GC;
* acionamento do circuit breaker;
* timeout de buscas;
* respostas parciais e, posteriormente, falha completa de consultas.

**Confiança: Alta.**

---

# Timeline

| Horário | Evento                                    | Evidência                |
| ------- | ----------------------------------------- | ------------------------ |
| 08:02   | Reindex ainda executando (38%)            | LoggingTaskListener      |
| 08:14   | Primeira pausa significativa de GC        | JvmGcMonitorService      |
| 08:30   | Heap cresce para 68%; cache começa a cair | Métricas                 |
| 08:41   | Indexação sofre throttling                | IndexingMemoryController |
| 09:00   | Indexação sobe para ~9800 docs/s          | Métricas                 |
| 09:03   | Fila de escrita chega a 150/200           | ThreadPool               |
| 09:12   | Old GC                                    | JvmGcMonitorService      |
| 09:20   | Reindex ainda em apenas 40%               | LoggingTaskListener      |
| 09:30   | Heap 88%; cache 41%; latência dobra       | Métricas                 |
| 09:31   | Circuit breaker próximo do limite         | HierarchyCircuitBreaker  |
| 09:44   | Slow query                                | SearchService            |
| 09:58   | Fila cheia; rejeição de bulk              | ThreadPool               |
| 09:58   | Heap 96%                                  | CircuitBreaker           |
| 09:58   | Timeout de busca                          | QueryPhase               |
| 09:58   | Resultado parcial (11/12 shards)          | SearchService            |
| 10:00   | Heap 94%; cache 29%; P99 6700 ms          | Métricas                 |
| 10:01   | CircuitBreakingException                  | SearchService            |
| 10:05   | All shards failed em parte das consultas  | SearchService            |

---

# Sintomas

## Observados

* aumento progressivo da latência P99;
* redução do cache hit;
* heap próximo do limite;
* GC cada vez mais frequente;
* throttling de indexação;
* rejeição de bulk;
* timeout de consultas;
* resultados parciais;
* falhas completas de busca.

---

# Cadeia causal

```
Reindex não termina

↓

Carga elevada de indexação durante horas

↓

indexed_docs/s cresce continuamente

↓

buffers de indexação saturam

↓

thread pool de escrita enche

↓

bulk é rejeitado

↓

heap cresce

↓

GC aumenta

↓

query cache sofre eviction

↓

cache hit diminui

↓

mais consultas precisam ser reexecutadas

↓

mais uso de CPU e heap

↓

Circuit Breaker dispara

↓

consultas passam do timeout

↓

shards falham

↓

resultados parciais

↓

algumas consultas falham totalmente
```

---

# Separação entre sintomas e causas

## Sintomas

* Busca lenta.
* Timeout.
* Resultados parciais.
* All shards failed.
* GC elevado.
* Heap alto.
* Cache hit baixo.

## Causas

* Reindex extremamente atrasada.
* Sobrecarga de indexação.
* Saturação do heap.
* Saturação do thread pool de escrita.
* Eviction intensa do query cache.

---

# Causa-raiz

## Principal

O job de reindexação iniciado às 02:00 permaneceu executando até pelo menos 10:00, quando normalmente deveria terminar às 03:30. Isso manteve uma carga anormal de escrita por várias horas, levando à saturação progressiva do cluster.

**Confiança: Alta**

---

## Causas contribuintes

* Heap JVM limitado a 8 GB sob carga sustentada.
* Queda do query cache hit (74% → 29%), aumentando o custo das buscas.
* Crescimento contínuo da fila de escrita.
* Throttling do shard.
* Circuit breaker de memória.

---

## Condições agravantes

* Reindex concorrendo com tráfego de produção.
* GC Old frequente.
* Heap operando acima de 90%.
* Rejeição de bulk aumentando backlog.

---

# Evidências

| Conclusão                   | Evidência                      | Confiança |
| --------------------------- | ------------------------------ | --------- |
| Reindex prolongada          | LoggingTaskListener            | Alta      |
| Carga de escrita crescente  | indexed_docs/s                 | Alta      |
| Heap saturando              | heap_used_pct + GC logs        | Alta      |
| Cache degradando            | cache_hit_pct + QueryCache log | Alta      |
| Thread pool saturado        | ThreadPool WARN                | Alta      |
| Buscas falhando por memória | CircuitBreaker + SearchService | Alta      |

---

# Mitigações imediatas

* Interromper ou pausar a reindexação.
* Reduzir concorrência do job.
* Redistribuir carga entre nós.
* Reiniciar nós apenas se a pressão de heap não diminuir após cessar a carga.
* Limitar throughput de ingestão até estabilização.

---

# Correções permanentes

* Garantir SLA para o término do reindex antes do horário comercial.
* Implementar monitoramento de duração do reindex com alerta.
* Revisar paralelismo e estratégia de reindexação.
* Avaliar aumento de heap ou expansão horizontal do cluster.
* Revisar número de shards e estratégia de indexação.
* Criar alertas para:

  * heap > 85%;
  * cache hit em queda contínua;
  * fila de escrita > 70%;
  * reindex excedendo duração esperada.

---

# Dados adicionais desejáveis

* Utilização de CPU.
* Uso de disco e I/O.
* Taxa de merge de segmentos.
* Número de nós do cluster.
* Volume de consultas por segundo.
* Estatísticas detalhadas de GC.
* Estado do cluster (green/yellow/red).
* Métricas por shard.

---

