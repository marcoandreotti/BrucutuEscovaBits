# 7. Agentes inteligentes

> **Resumo:** um agente pode decidir o próximo passo de uma tarefa com base no que observou. Essa flexibilidade precisa de um objetivo verificável e de limites que não dependam da boa vontade do modelo.

## Pedra fundamental

Neste laboratório, um **agente** é um componente que usa um modelo para escolher ações em um ciclo de execução orientado a um objetivo, com contexto, ferramentas e condições de parada. O termo possui usos variados; essa é a definição operacional que vamos avaliar.

<small>O explorador pode escolher a próxima trilha. O mapa de áreas proibidas e a hora de voltar continuam valendo.</small>

Um **workflow (fluxo de trabalho predefinido)** determina antecipadamente a sequência de passos. Em um fluxo com agente, parte dessa sequência é escolhida durante a execução. Ter autonomia é uma decisão de projeto, não uma melhoria automática.

## O problema e o encaixe arquitetural

A tarefa será: “Prepare uma orientação para um novo integrante retirar uma ferramenta, considerando o procedimento e a disponibilidade atual”. O assistente pode precisar buscar documentos, consultar o estoque e pedir esclarecimento.

O caso de uso `ExecutarAssistente` coordena o ciclo em `Application`. As ferramentas reutilizam consultas e comandos existentes. O agente não recebe atalhos para contornar o domínio nem credenciais adicionais por ter escolhido uma ação.

O estado registra objetivo, passos, resultados e motivo de encerramento. **Memória** é contexto preservado entre interações; ela precisa de origem, escopo de acesso, validade e possibilidade de exclusão. Histórico ilimitado não é uma estratégia de memória.

## Experimento proposto

**Pré-requisito:** busca documental e ferramentas autorizadas das [etapas 5](05-rag-com-documentos.md) e [6](06-function-calling.md).

1. Resolver primeiro a tarefa com um fluxo fixo: buscar procedimento, consultar estoque e responder. Guardar esse resultado como referência de comparação.
2. Permitir que um único agente escolha entre essas ferramentas ou peça esclarecimento. Não conceder operações de escrita neste primeiro experimento.
3. Aplicar no executor limites iniciais didáticos: até quatro rodadas de decisão, trinta segundos no total e um orçamento de tokens definido antes do teste. Esses números são hipóteses para medir, não valores universais.
4. Provocar ferramenta indisponível, pergunta ambígua e repetição de uma chamada. Encerrar por conclusão, falta de dados, cancelamento, erro ou limite atingido.

## Evidências e critérios de sucesso

- Cada ação fica registrada com ferramenta, resultado e vínculo à tarefa.
- A execução sempre encontra uma condição de parada, inclusive quando o modelo insiste.
- A resposta distingue tarefa concluída de tentativa interrompida; não anuncia sucesso após uma falha.
- A comparação mede qualidade, duração, chamadas e custo em relação ao fluxo fixo.

Não é necessário solicitar nem armazenar raciocínio interno do modelo. Argumentos, resultados, fontes e decisões observáveis sustentam a investigação operacional.

## Vantagens, limitações e erros comuns

Agentes podem adaptar o percurso quando a tarefa varia. Também ampliam caminhos possíveis, dificultam reprodução e podem repetir erros a cada rodada. Um fluxo fixo costuma ser melhor quando a sequência já é conhecida.

Evite começar com vários agentes, delegação recursiva e memória permanente. Primeiro demonstre que um agente com poucas ferramentas entrega benefício mensurável. Para ações de impacto, a autorização e a confirmação continuam sob controle da aplicação.

**Desafio da caverna:** apresente um caso em que a autonomia ajuda e outro em que o fluxo fixo é mais previsível. Defenda a escolha com as evidências.

## Referências

- [Conceitos de agentes na documentação .NET](https://github.com/dotnet/docs/blob/main/docs/ai/conceptual/agents.md).
- [Agentes, skills, memória e orquestração no Brucutu](../../docs/08-inteligencia-artificial/agentes-skills-memoria-e-orquestracao.md).

[Anterior: Function Calling](06-function-calling.md) · [Visão geral](README.md) · [Próxima: MCP Server e MCP Client](08-mcp-server-e-client.md)
