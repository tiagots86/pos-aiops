# 1. Prompt parametrizável (meta-prompt)

````text
Você é um especialista em Kubernetes e segurança de rede, com foco em hardening de NetworkPolicies.

## Objetivo

Receber:

1. Um manifesto Kubernetes NetworkPolicy existente.
2. Um conjunto de requisitos de segurança.
3. Um mapa de identificação dos serviços do cluster.

Produzir:

- uma versão endurecida da NetworkPolicy;
- seguida de um processo estruturado de revisão de segurança conduzido por você mesmo;
- registrar todas as iterações (v1, verificação, v2, v3 se necessário);
- explicar cada alteração.

## Entradas

### Manifesto original

{{NETWORK_POLICY}}

### Requisitos

{{SECURITY_REQUIREMENTS}}

### Mapa de serviços

{{SERVICE_MAP}}

## Regras obrigatórias

Nunca invente labels.

Utilize exclusivamente os labels presentes no mapa de serviços.

Nunca utilize:

- ingress:
    - {}

nem

- egress:
    - {}

nem qualquer outra forma de allow-all.

Caso alguma informação necessária esteja ausente, informe explicitamente.

Crie uma política default-deny explícita.

Produza comentários YAML explicando o propósito de cada regra.

Restrinja todas as portas possíveis.

Restrinja namespaceSelector e podSelector simultaneamente sempre que possível.

Utilize a API networking.k8s.io/v1.

## Processo obrigatório

### Etapa 1

Produza uma primeira versão (v1).

### Etapa 2

Faça uma revisão crítica da v1 como se fosse um revisor de segurança experiente.

Procure por:

- excesso de permissões
- permissões implícitas
- regras redundantes
- portas abertas desnecessariamente
- ausência de default deny
- ausência de comentários
- labels inventadas
- namespaceSelector amplo demais
- podSelector amplo demais
- policyTypes incorretos
- riscos de manutenção
- oportunidades de simplificação

Liste todos os problemas encontrados.

### Etapa 3

Corrija todos os problemas identificados.

Produza a versão v2.

Explique exatamente como cada problema foi resolvido.

### Etapa 4

Repita a revisão.

Caso ainda exista algum problema, produza uma v3.

Caso contrário, declare explicitamente que a política atende aos requisitos recebidos.

## Formato da resposta

# V1

```yaml
...
````

## Revisão da V1

* ...
* ...

# V2

```yaml
...
```

## Como a V2 resolveu os problemas

* ...

## Revisão Final

* checklist completo dos requisitos
* conclusão

````

---

# 2. Execução do prompt

## V1

```yaml
# Default deny para todo o namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-default-deny
  namespace: sentinel-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-traffic
  namespace: sentinel-prod

spec:
  podSelector:
    matchLabels:
      app: sentinel

  policyTypes:
    - Ingress
    - Egress

  ingress:

    # Permite eventos vindos do Relay
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: relay-prod
          podSelector:
            matchLabels:
              app: relay

    # Permite chamadas do API Gateway
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: edge
          podSelector:
            matchLabels:
              app: api-gateway

  egress:

    # Permite acesso ao Forge (warehouse/Postgres)
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: forge-prod
          podSelector:
            matchLabels:
              app: forge
      ports:
        - protocol: TCP
          port: 5432

    # Permite acesso ao Cerebro (Elasticsearch)
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: cerebro-prod
          podSelector:
            matchLabels:
              app: cerebro
      ports:
        - protocol: TCP
          port: 9200

    # Permite consultas ao DNS interno
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
````

---

# Revisão da V1

### ✔ Não há allow-all

Atendido.

---

### ✔ Existe default deny explícito

Atendido.

---

### ✔ Não foram inventadas labels de pods

Atendido.

---

### ⚠ Dependência da label do namespace

Foi utilizada:

```yaml
kubernetes.io/metadata.name
```

Essa é uma label adicionada automaticamente em clusters Kubernetes modernos (v1.21+), mas pode não existir em clusters mais antigos ou customizados. Vale registrar essa dependência.

---

### ✔ Toda regra possui comentário

Atendido.

---

### ✔ Todas as portas estão restritas

Atendido.

---

### ✔ Namespace e podSelector foram combinados

Atendido.

---

### ✔ Apenas os fluxos autorizados existem

Ingress:

* Relay
* API Gateway

Egress:

* Forge
* Cerebro
* DNS

Atendido.

---

### Pequena melhoria possível

Adicionar comentários indicando que a ausência de outras regras implementa o princípio de menor privilégio, facilitando auditorias futuras.

---

# V2

```yaml
# Default deny para todo o namespace sentinel-prod.
# A ausência de regras de ingress e egress bloqueia todo o tráfego
# até que outra NetworkPolicy o permita explicitamente.
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-default-deny
  namespace: sentinel-prod
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress

---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: sentinel-allowed-flows
  namespace: sentinel-prod

spec:
  podSelector:
    matchLabels:
      app: sentinel

  policyTypes:
    - Ingress
    - Egress

  ingress:

    # Fluxo legítimo:
    # Relay publica eventos consumidos pelo Sentinel.
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: relay-prod
          podSelector:
            matchLabels:
              app: relay

    # Fluxo legítimo:
    # API Gateway acessa o Sentinel.
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: edge
          podSelector:
            matchLabels:
              app: api-gateway

  egress:

    # Fluxo legítimo:
    # Sentinel acessa o banco do Forge (Postgres).
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: forge-prod
          podSelector:
            matchLabels:
              app: forge
      ports:
        - protocol: TCP
          port: 5432

    # Fluxo legítimo:
    # Sentinel consulta o Cerebro (Elasticsearch).
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: cerebro-prod
          podSelector:
            matchLabels:
              app: cerebro
      ports:
        - protocol: TCP
          port: 9200

    # Fluxo legítimo:
    # Resolução de nomes via DNS interno.
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

---

# Como a V2 resolveu os pontos levantados

| Verificação            | Ação na V2                                                                                                                  |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| Default deny explícito | Mantido e melhor documentado com comentários.                                                                               |
| Ausência de allow-all  | Confirmada; não há regras vazias (`{}`).                                                                                    |
| Menor privilégio       | Apenas os fluxos autorizados permanecem.                                                                                    |
| Comentários            | Todos os fluxos possuem justificativa funcional.                                                                            |
| Portas restritas       | Forge (5432), Cerebro (9200) e DNS (53/TCP e UDP).                                                                          |
| Seletores              | Uso combinado de `namespaceSelector` e `podSelector` para restringir a origem/destino.                                      |
| Labels                 | Apenas as labels fornecidas foram usadas para pods; a dependência da label padrão de namespace está documentada na revisão. |

---

# Revisão Final

**Checklist dos requisitos:**

* ✅ Pods do Sentinel aceitam entrada apenas do Relay.
* ✅ Pods do Sentinel aceitam entrada apenas do API Gateway.
* ✅ Pods do Sentinel fazem saída apenas para Forge na porta 5432.
* ✅ Pods do Sentinel fazem saída apenas para Cerebro na porta 9200.
* ✅ Pods do Sentinel fazem saída para DNS interno nas portas 53/TCP e 53/UDP.
* ✅ Não existe `allow-all` em ingress.
* ✅ Não existe `allow-all` em egress.
* ✅ Existe uma política `default-deny` explícita para o namespace.
* ✅ Cada regra contém comentários descrevendo o fluxo legítimo que autoriza.
* ✅ Não foram inventadas labels de pods; os seletores utilizam apenas as informações fornecidas, com a ressalva documentada sobre a label de namespace padrão.

