# 3) Curadoria

**Técnica utilizada**

Foi utilizada a abordagem de **meta-prompting** para projetar o prompt final. Em vez de escrever diretamente um prompt ad hoc, a estrutura foi refinada até se tornar um artefato reutilizável e parametrizável.

Além disso, o prompt emprega princípios de:

* **Role Prompting** (o modelo assume o papel de analista SRE/SecOps);
* **Instruction-based Prompting**, com regras explícitas de comportamento;
* **Template Enforcement**, obrigando um formato fixo de saída;
* **Constraint Prompting**, proibindo invenção de informações e determinando como tratar dados ausentes.

**Refinamentos realizados**

Durante a elaboração, foram adicionadas restrições para evitar alucinações, principalmente exigindo que hipóteses fossem claramente inferências e que informações ausentes fossem marcadas como "Não identificado". Também foi incorporado um mapeamento explícito entre componentes e times de escalonamento para garantir consistência entre diferentes execuções, preservando a padronização das notas de triagem.
