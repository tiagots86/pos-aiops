# Execução

## Modelo

**GPT-5.5**

---

## Entrada

```text
Relay (barramento de eventos):
- throughput sustentado: 180k msgs/s
- pico observado: 320k msgs/s durante 25 minutos
- retenção: 4h

Consumidores:
- Forge (ingestão)
- Sentinel (alerting)

Restrições:

- Sentinel não pode atrasar mais que 60 segundos
- Forge pode atrasar até 15 minutos
- orçamento está 8% acima do previsto
- perda de telemetry é inaceitável

Estratégias candidatas:

- priorização do Sentinel
- dead-letter queue
- isolamento por cliente
- auto scaling dos consumidores
```

---

# Output esperado

## Resumo

O Relay sofre sobrecarga quando clientes geram picos acima da capacidade sustentada. O principal risco é o aumento da fila, atrasando o Sentinel e violando o SLA de alertas em tempo real.

---

## Objetivos

* preservar todas as mensagens;
* manter alertas dentro de 60 segundos;
* evitar impacto entre clientes;
* controlar custos.

---

## Restrições

* nenhuma perda de telemetry;
* orçamento limitado;
* Forge pode acumular atraso;
* Sentinel não.

---

## Gargalos

O Relay suporta:

**180 mil msgs/s**

O incidente chegou a:

**320 mil msgs/s**

Isso representa aproximadamente:

**78% acima da capacidade sustentada.**

Sem mecanismos de backpressure, a fila cresce continuamente durante o pico.

---

# Estratégia 1 — Priorização do Sentinel

### Funcionamento

Consumidores do Sentinel recebem prioridade sobre o Forge.

### Vantagens

* protege o SLA crítico
* implementação relativamente simples
* não exige aumento grande de infraestrutura

### Desvantagens

* Forge acumulará backlog
* requer políticas de prioridade na fila

### Complexidade

Média

### Custo

Baixo

### Compatibilidade

Excelente.

Respeita o SLA de ambos os consumidores.

---

# Estratégia 2 — Dead-letter Queue

### Funcionamento

Eventos não processados são enviados para uma fila auxiliar para reprocessamento posterior.

### Benefícios

* evita perda de mensagens
* aumenta resiliência

### Desvantagens

* não resolve gargalo em tempo real
* aumenta complexidade operacional

### Complexidade

Média

### Custo

Baixo a médio.

### Compatibilidade

Boa, mas insuficiente sozinha.

---

# Estratégia 3 — Isolamento por cliente

### Funcionamento

Cada cliente utiliza partições ou filas independentes.

### Benefícios

* clientes barulhentos deixam de afetar outros
* melhora previsibilidade
* reduz efeito cascata

### Desvantagens

* mudança arquitetural relevante
* migração complexa

### Complexidade

Alta

### Custo

Alto inicialmente.

### Compatibilidade

Excelente para longo prazo.

---

# Estratégia 4 — Auto Scaling

### Funcionamento

Consumidores aumentam automaticamente quando a fila cresce.

### Benefícios

* absorve picos
* reduz backlog

### Desvantagens

* aumenta custos
* pode não resolver se o gargalo estiver no broker

### Complexidade

Média

### Custo

Médio a alto.

### Compatibilidade

Boa, porém limitada pelo orçamento.

---

# Comparação

| Estratégia   | SLA Sentinel | Custo      | Complexidade | Resolve pico | Longo prazo |
| ------------ | ------------ | ---------- | ------------ | ------------ | ----------- |
| Priorização  | Excelente    | Baixo      | Média        | Parcial      | Média       |
| DLQ          | Boa          | Baixo      | Média        | Não          | Boa         |
| Isolamento   | Excelente    | Alto       | Alta         | Sim          | Excelente   |
| Auto Scaling | Boa          | Médio/Alto | Média        | Sim          | Boa         |

---

# Combinação recomendada

A melhor abordagem é combinar:

* Priorização do Sentinel;
* Isolamento por cliente;
* Auto Scaling limitado;
* Dead-letter Queue para recuperação.

---

## Justificativa

A priorização garante o SLA de 60 segundos.

O isolamento evita que um único cliente degrade todos os demais.

O auto scaling absorve picos temporários sem superdimensionar permanentemente a infraestrutura.

A dead-letter queue fornece uma camada adicional de proteção contra falhas de processamento, preservando mensagens para reprocessamento.

Essa combinação atende simultaneamente aos requisitos de disponibilidade, confiabilidade e controle de custos.

---

## Premissas

A recomendação pode mudar caso:

* o Relay permita QoS/prioridade nativa;
* exista limitação física do broker;
* o throughput sustentado possa ser aumentado futuramente;
* novos consumidores sejam adicionados.

---

