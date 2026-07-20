# Curadoria

A cadeia de prompts é mais eficaz do que um único prompt porque cada elo tem um objetivo restrito e produz uma saída estruturada que serve de contexto para o próximo. Isso reduz respostas genéricas e melhora a qualidade da análise.

Pontos fortes da cadeia:

* **Separação de responsabilidades:** diagnóstico, estratégia, execução, revisão e consolidação ficam isolados.
* **Contexto progressivo:** cada etapa reutiliza e aprofunda a anterior, evitando perda de informação.
* **Reutilização:** os meta-prompts podem ser aplicados a outras migrações (batch→streaming, monólito→microserviços, banco de dados etc.) apenas substituindo os parâmetros.
* **Aderência aos requisitos:** a estratégia incorpora migração incremental, execução paralela, compatibilidade com consumidores existentes, critérios de aceite e rollback em todas as fases, atendendo à exigência de evitar uma migração *big bang*.
