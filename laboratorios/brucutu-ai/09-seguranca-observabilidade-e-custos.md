# 9. Segurança, observabilidade e custos

> **Resumo:** o experimento final reúne controles introduzidos desde o começo. Uma resposta rápida e barata só é útil se estiver correta o suficiente para a tarefa e respeitar as permissões.

## Pedra fundamental

**Segurança** controla acesso, exposição de dados e efeitos das operações. **Observabilidade** permite investigar o comportamento do sistema a partir de seus sinais. **Gestão de custos** conecta consumo, orçamento e valor entregue. Em IA, essas três preocupações também passam pela qualidade das respostas.

<small>A fogueira está bonita. Precisamos saber quem pode alimentá-la, de onde vem a fumaça e quanto de lenha sobra.</small>

**Logs (registros de eventos)** descrevem ocorrências, **métricas** agregam medidas e **traces (rastros de execução)** conectam etapas de uma operação. OpenTelemetry oferece padrões e ferramentas para instrumentar e transportar esses sinais; o armazenamento e a análise dependem do destino escolhido. [Referência: sinais do OpenTelemetry](https://opentelemetry.io/docs/concepts/signals/).

## O problema e o encaixe arquitetural

O assistente consulta documentos, usa ferramentas e pode realizar várias chamadas ao modelo. Uma resposta final não mostra sozinha se houve acesso indevido, desperdício ou falha escondida.

O adaptador de entrada valida identidade. Casos de uso e serviços responsáveis aplicam permissões. Adaptadores de saída preservam isolamento, limitam chamadas e registram sinais. Identificadores de organização ou usuário vêm do contexto autenticado, nunca de uma sugestão do modelo.

Conteúdo de documentos, retornos de ferramentas e respostas do modelo permanece não confiável. Aplicar validação determinística, menor privilégio e confirmação para ações de impacto reduz consequências mesmo quando uma instrução maliciosa influencia o modelo. [Referência: controles contra prompt injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/).

## Experimento integrado

**Pré-requisito:** os contratos e roteiros das etapas anteriores. Definir as metas e o orçamento antes da futura execução.

1. Criar duas organizações fictícias com documentos e estoque separados. Tentar acessar dados cruzados por pergunta, identificador e ferramenta. Repetir com cache (armazenamento temporário), se houver, e por MCP.
2. Inserir instruções maliciosas em um documento de teste. Verificar quais ações o modelo pede e quais o executor bloqueia. Não colocar segredos reais no experimento.
3. Simular demora, limite de requisições, provedor indisponível e repetição de ferramenta. Correlacionar entrada, busca, gateway, modelo e execução sem registrar os textos completos por padrão.
4. Executar o conjunto de avaliação da etapa 3, ampliado com fontes, permissões e ferramentas. Medir qualidade e consumo de todas as tentativas, inclusive falhas.

## O que medir

| Dimensão | Evidência útil |
| --- | --- |
| Qualidade | Respostas sustentadas por fontes, erros factuais, recusas adequadas e tarefas concluídas |
| Recuperação | Presença dos trechos esperados entre os resultados autorizados |
| Desempenho | Duração total e por etapa, incluindo novas tentativas; percentil 95 com amostra suficiente |
| Confiabilidade | Falhas por categoria, cancelamentos e encerramentos por limite |
| Segurança | Acessos negados e tentativas de ação fora das permissões |
| Consumo | Tokens reportados, chamadas, embeddings, armazenamento e execução de ferramentas |

Manter identificadores de alta cardinalidade, como o de uma execução, nos registros e rastros apropriados, em vez de criar uma série de métricas para cada solicitação. Definir acesso e retenção dos sinais; observabilidade não pode virar um vazamento organizado.

## Custos sem adivinhação

Uma estimativa simples, quando o fornecedor cobra por tokens, é: **custo de geração = tokens de entrada × tarifa de entrada + tokens de saída × tarifa de saída**. Usar a unidade correta, por exemplo por milhão de tokens, e somar todas as chamadas.

Acrescentar embeddings, busca, banco, ferramentas e infraestrutura. Modelos locais também consomem recursos. Consultar as tarifas do provedor escolhido no momento do experimento; este roteiro não fixa preços ou moedas.

Reservar orçamento antes da chamada e reconciliar o uso depois. Limitar tamanho, concorrência, tempo e número de rodadas. Um limite de tokens isolado não garante um teto financeiro exato, especialmente com chamadas em andamento ou uso reportado com atraso. Consumo não informado deve ficar marcado como desconhecido ou estimado, nunca como zero.

## Evidências e critérios de sucesso

- Nenhum dado da outra organização chega ao contexto ou à ferramenta executada no conjunto de testes.
- Uma operação pode ser investigada por correlação sem revelar documentos, credenciais ou perguntas completas.
- Falhas e orçamento esgotado interrompem o fluxo com motivo explícito.
- A comparação apresenta custo por tarefa concluída com qualidade aceitável, além do custo por chamada.

## Vantagens, limitações e desafio final

Mais sinais ajudam a investigar, mas aumentam armazenamento, exposição e manutenção. Mais filtros podem bloquear usos legítimos. Modelos menores podem reduzir custo e piorar qualidade, provocando novas tentativas que anulam a economia. Medir as compensações antes de escolher.

**Desafio da caverna:** demonstrar o assistente em um caso bem-sucedido e em três falhas: falta de evidência, acesso negado e limite atingido. Apresentar os resultados reais, as limitações conhecidas e uma decisão arquitetural que simplifique a solução.

Ao encerrar a futura execução, seguir a [limpeza do laboratório](README.md#limpeza-na-futura-execução) e manter apenas evidências necessárias, com dados fictícios.

## Próximos estudos

- [Observabilidade, logs e métricas](../../docs/07-cloud-e-devops/observabilidade-logs-e-metricas.md).
- [Stack de testes em .NET](../../docs/05-testes/stack-de-testes-dotnet.md).

[Anterior: MCP Server e MCP Client](08-mcp-server-e-client.md) · [Visão geral do Brucutu.AI](README.md) · [Todos os laboratórios](../README.md)
