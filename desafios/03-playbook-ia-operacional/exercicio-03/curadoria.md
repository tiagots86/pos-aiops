# Curadoria

## Qualidade do prompt

**Pontos fortes**

* Estrutura o raciocínio para evitar respostas superficiais.
* Separa fatos, inferências e hipóteses.
* Exige correlação entre configuração, métricas e logs.
* Força uma cadeia causal, reduzindo o risco de confundir sintomas com causa-raiz.
* Produz um relatório reutilizável para diferentes incidentes, bastando trocar os artefatos de entrada.

**Limitações**

* Depende da qualidade e completude dos artefatos fornecidos.
* Não substitui validações em ambiente (por exemplo, inspeção de I/O, CPU ou estado do cluster).
* A causa-raiz pode permanecer probabilística quando faltarem evidências essenciais.

## Tratamento antes de enviar a um modelo externo

Como os dados devem ser tratados como produção, recomenda-se:

* **Remover ou pseudonimizar nomes internos** (por exemplo, "Cerebro", "Sentinel", "Aegis", "Sam Wilson"), substituindo por identificadores genéricos.
* **Generalizar nomes de índices**, shards e pods se revelarem convenções internas.
* **Eliminar identificadores de tarefas** (como `task [88123]`) e outros IDs operacionais que não agreguem à análise.
* **Revisar logs em busca de informações sensíveis**, como endereços IP, nomes de hosts, caminhos de arquivos, credenciais ou dados de clientes (não presentes neste exemplo, mas comuns em produção).
* **Manter as relações temporais e métricas**, pois são essenciais para a análise de causa-raiz e podem ser preservadas após anonimização.

Essa curadoria reduz o risco de exposição de informações operacionais sem comprometer a capacidade do modelo de identificar a cadeia causal do incidente.
