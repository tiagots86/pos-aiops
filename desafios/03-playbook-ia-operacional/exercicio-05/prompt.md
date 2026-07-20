# Elo 1 — Diagnóstico

### Meta-prompt

```text
Você é um arquiteto de software especializado em migrações de pipelines de dados.

Objetivo:
Analisar o estado atual de um pipeline e produzir um diagnóstico técnico.

Entrada
<SISTEMA_ATUAL>

Saída obrigatória
1. Resumo da arquitetura atual
2. Componentes críticos
3. Gargalos
4. Dependências
5. Riscos da migração
6. Aspectos que precisam permanecer compatíveis

Não proponha soluções ainda.
```

---

### Modelo

GPT-5.5

---

### Entrada

(Cenário do Forge)

---

### Output

**Arquitetura**

* ingestão batch via cron
* Spark com 14 estágios
* processamento leva cerca de 40 minutos
* saída em tabelas particionadas por hora

**Componentes críticos**

* forge-batch-ingest
* Spark
* Data Warehouse

**Dependências**

* Sentinel
* Cerebro
* Pepper Billing

**Gargalos**

* processamento concentrado em uma janela
* acúmulo de volume após falhas
* alta latência

**Riscos**

* quebra de consumidores
* inconsistência entre batch e streaming
* perda de eventos
* duplicidade

**Compatibilidade obrigatória**

* manter formato das tabelas
* preservar contratos existentes
* manter processamento correto durante toda a migração

---

# Elo 2 — Estratégia de migração

### Meta-prompt

```text
Você é um arquiteto de migração.

Utilize apenas o diagnóstico abaixo.

<DIAGNOSTICO>

Produza uma estratégia incremental.

Requisitos:

- sem big bang
- reversível
- cada etapa deve entregar valor
- definir critérios de sucesso
- indicar rollback para cada etapa

Não detalhe implementação.
```

---

### Modelo

GPT-5.5

---

### Entrada

Saída do Elo 1.

---

### Output

## Etapa 1

Criar pipeline streaming paralelo consumindo Relay.

Rollback

Desligar consumidor.

Critério

Streaming funcionando sem impacto.

---

## Etapa 2

Executar batch e streaming simultaneamente.

Comparar resultados.

Rollback

Continuar usando somente batch.

Critério

Resultados equivalentes.

---

## Etapa 3

Migrar Cerebro para consumir saída streaming.

Rollback

Retornar para batch.

Critério

Sem divergências.

---

## Etapa 4

Migrar Sentinel.

Rollback

Voltar leitura antiga.

Critério

Dashboards corretos.

---

## Etapa 5

Migrar Billing.

Rollback

Executar relatórios via batch.

Critério

Valores idênticos.

---

## Etapa 6

Desligar batch.

Rollback

Reativar processamento batch.

Critério

Operação estável por período definido.

---

# Elo 3 — Plano executável

### Meta-prompt

```text
Você é um Tech Lead.

Receba a estratégia abaixo.

<ESTRATEGIA>

Transforme-a em um plano executável.

Para cada etapa informe:

- atividades técnicas
- entregáveis
- monitoramento
- rollback
- riscos
- critérios de aceite

Considere produção.
```

---

### Modelo

GPT-5.5

---

### Entrada

Saída do Elo 2.

---

### Output

## Etapa 1

### Atividades

* criar consumidor Relay
* definir micro-batches
* persistência temporária
* métricas

### Entregáveis

* pipeline streaming ativo

### Monitoramento

* throughput
* lag
* erros
* latência

### Rollback

Desativar consumidor.

### Aceite

Pipeline estável.

---

## Etapa 2

### Atividades

* executar processamento duplo
* reconciliar dados
* medir diferenças

### Entregáveis

Relatório de equivalência.

### Monitoramento

* divergências
* duplicação
* perda

### Rollback

Interromper streaming.

### Aceite

Equivalência validada.

---

## Etapas seguintes

Mesmo padrão para Cerebro, Sentinel, Pepper e desligamento gradual do batch.

---

# Elo 4 — Validação

### Meta-prompt

```text
Você é um revisor de arquitetura.

Analise o plano abaixo.

<PLANO>

Identifique:

- riscos não tratados
- lacunas
- melhorias
- dependências ocultas

Priorize por impacto.
```

---

### Modelo

GPT-5.5

---

### Entrada

Plano do Elo 3.

---

### Output

**Alta prioridade**

* garantir processamento idempotente
* estratégia para eventos fora de ordem
* mecanismo de replay
* DLQ para falhas
* versionamento de esquema

**Média prioridade**

* observabilidade fim a fim
* SLOs
* testes de carga

**Baixa prioridade**

* automação do rollback
* documentação operacional

---

# Elo 5 — Plano final

### Meta-prompt

```text
Utilize:

- diagnóstico
- estratégia
- plano
- revisão

Produza um plano consolidado.

Formato:

- arquitetura alvo
- roadmap
- cronograma lógico
- riscos
- rollback global
- checklist de conclusão
```

---

### Modelo

GPT-5.5

---

### Entrada

Resultados dos elos anteriores.

---

### Output

## Arquitetura alvo

```
Relay
    │
Streaming Consumer
    │
Micro-batches
    │
Transformações
    │
DW
    ├── Sentinel
    ├── Cerebro
    └── Pepper
```

### Roadmap

1. Pipeline paralelo.
2. Execução em paralelo (batch + streaming).
3. Validação de consistência.
4. Migração progressiva dos consumidores.
5. Desativação controlada do batch.

### Rollback global

* batch permanece operacional até a validação completa;
* consumidores podem voltar ao batch independentemente;
* desligamento definitivo apenas após estabilidade comprovada.

### Checklist

* Consumo contínuo do Relay.
* Micro-batches funcionando.
* Equivalência validada.
* Monitoramento implantado.
* Rollback testado.
* Consumidores migrados.
* Batch desativado.

---

