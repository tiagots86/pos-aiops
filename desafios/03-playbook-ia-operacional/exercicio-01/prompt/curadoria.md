# Curadoria

**Framework utilizado:** Role + Task + Constraints + Output Format (estrutura inspirada em práticas de meta-prompting e engenharia de prompts baseada em papéis e instruções explícitas).

### Técnica aplicada

O prompt foi inicialmente gerado por meta-prompting, solicitando à IA um prompt reutilizável para triagem de pods. Em seguida, foi refinado para:

* separar claramente **papel**, **entrada**, **processo de análise** e **formato de saída**;
* obrigar o cruzamento entre `kubectl get pods`, `describe`, `events` e `logs`, evitando respostas que apenas repitam o STATUS;
* restringir inferências às evidências presentes no snapshot, reduzindo alucinações;
* padronizar a resposta em um formato operacional adequado para uso recorrente pela equipe de SRE.

O resultado é um prompt parametrizável, reutilizável e consistente, adequado para compor a biblioteca versionada de prompts da Aegis.