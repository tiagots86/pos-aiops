# Curadoria

### Pontos fortes do prompt

* Não induz uma resposta única.
* Obriga a comparar alternativas.
* Explicita trade-offs.
* Considera custos, risco e SLAs.
* Produz uma recomendação fundamentada.
* Pode ser reutilizado em outros cenários de arquitetura apenas substituindo o bloco `<CENARIO>`.

### Limitações

* A qualidade da análise depende da completude das informações fornecidas no cenário.
* Não calcula automaticamente capacidade, tempo de escoamento de backlog ou custos; para isso seria útil complementar o prompt solicitando estimativas quantitativas quando houver dados suficientes.

### Avaliação da recomendação

Para este cenário, a combinação de **priorização do Sentinel + isolamento por cliente + auto scaling controlado + DLQ** é a alternativa mais equilibrada. Ela preserva o SLA mais crítico (alertas em até 60 segundos), evita perda de telemetry, reduz o impacto de clientes com comportamento "barulhento" e limita o aumento de custos, alinhando-se às restrições apresentadas.
