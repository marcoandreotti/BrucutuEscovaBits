# 6. Function Calling

> **Resumo:** quando a pergunta depende do estoque de agora, precisamos consultar um sistema. O modelo pode sugerir a chamada; a aplicação decide se pode executá-la.

## Pedra fundamental

**Function Calling (solicitação estruturada de chamada de função)**, também chamado **tool calling (solicitação de uso de ferramenta)**, permite ao modelo indicar uma operação e seus argumentos conforme um contrato oferecido pela aplicação. O modelo não executa diretamente o método C# nem recebe autoridade para ignorar regras.

<small>O xamã pede para abrir o depósito. O pedido não transforma a barba dele em chave.</small>

O fluxo é: apresentar ferramentas, receber uma solicitação estruturada, validar e autorizar, executar, devolver o resultado ao modelo e obter a resposta ou outra solicitação. Esse ciclo precisa de limites. [Referência: chamadas de ferramentas em .NET](https://learn.microsoft.com/dotnet/ai/conceptual/calling-tools).

## O problema e o encaixe arquitetural

“Temos quantas lanças disponíveis?” pede um dado atual. O procedimento recuperado por RAG pode explicar como consultar estoque, mas não substitui a consulta ao sistema responsável.

Uma ferramenta `ConsultarEstoque` expõe uma entrada pequena e uma saída tipada. Seu adaptador invoca um caso de uso de consulta em `Application`, que aplica autorização e utiliza a porta de estoque. Não oferecer acesso irrestrito ao banco, ao terminal ou a qualquer método descoberto por reflexão.

A identidade vem do contexto autenticado da aplicação. Um argumento inventado pelo modelo, como “usuário administrador”, não altera as permissões.

## Experimento proposto

**Pré-requisito:** [etapa 5](05-rag-com-documentos.md), estoque fictício e modelo com suporte à capacidade exigida. Simulações podem verificar o contrato antes da integração real.

1. Definir nome, descrição, argumentos obrigatórios e formato de retorno da ferramenta. Começar apenas com leitura.
2. Enviar uma pergunta que precise de estoque atual. Inspecionar a chamada proposta antes de executá-la.
3. Validar tipos, limites, identificador do item e acesso do usuário. Executar somente ferramentas explicitamente permitidas e limitar a quantidade de chamadas.
4. Simular item inexistente, falta de permissão, argumento inválido e indisponibilidade. Devolver resultados controlados, sem inventar disponibilidade.

## Evidências e critérios de sucesso

- A resposta de estoque está vinculada ao resultado real da ferramenta fictícia.
- Uma chamada não autorizada falha antes de acessar o recurso.
- Ferramenta desconhecida ou argumento inválido não dispara execução genérica.
- Falhas e chamadas são correlacionadas sem expor dados restritos ou credenciais.

## Vantagens, limitações e erros comuns

Ferramentas conectam linguagem natural a operações verificáveis. Cada ferramenta amplia a superfície de acesso e consome contexto com sua descrição; descrições vagas também prejudicam a escolha.

Para futuras escritas, como `ReservarFerramenta`, usar comando com regras determinísticas, confirmação quando necessária e **idempotência (repetir uma solicitação sem duplicar seu efeito)**. Uma chave de operação deve representar a ação lógica e sobreviver às novas tentativas. Solicitar uma confirmação no prompt não substitui verificá-la na aplicação.

Se uma tela ou um endpoint direto atende melhor a tarefa, não é obrigatório colocar um modelo entre a pessoa e a consulta. E uma chamada de ferramenta, sozinha, não torna o fluxo um agente autônomo.

**Desafio da caverna:** simule uma resposta perdida depois de uma reserva bem-sucedida. Explique como impedir que a nova tentativa reserve uma segunda lança. Consulte [idempotência e duplicidade](../../docs/04-apis-e-integracoes/idempotencia-e-duplicidade.md).

[Anterior: RAG](05-rag-com-documentos.md) · [Visão geral](README.md) · [Próxima: agentes inteligentes](07-agentes-inteligentes.md)
