# 2. AI Gateway com múltiplos provedores

> **Resumo:** quando existem alternativas de modelos, precisamos de um lugar explícito para decidir qual pode atender cada solicitação e sob quais condições.

## Pedra fundamental

Um **AI Gateway (ponto de controle de acesso à IA)** concentra políticas como seleção de provedor, limites de consumo, verificação de capacidades e tratamento de falhas. Ele pode começar como um componente dentro da aplicação; um serviço separado só se justifica quando houver necessidade operacional.

<small>O porteiro da caverna escolhe o mensageiro. Não precisa fundar outra aldeia para organizar a fila.</small>

**Roteamento** escolhe o destino de uma solicitação. **Fallback (alternativa de contingência)** tenta outra opção quando uma condição previamente definida acontece. **Retry (nova tentativa)** repete uma operação, normalmente diante de uma falha transitória. São decisões diferentes, com custos e riscos diferentes.

## O problema e o encaixe arquitetural

O assistente pode precisar de um modelo econômico para explicar um procedimento e de outro com suporte a ferramentas. A escolha também depende dos dados que cada destino está autorizado a receber.

`Application` declara a necessidade da operação. A política do gateway seleciona entre adaptadores registrados em `Infrastructure`. Credenciais, tradução de mensagens e erros particulares continuam nos adaptadores. O domínio não conhece nomes comerciais de modelos.

Uma interface comum reduz diferenças mecânicas; ela não elimina diferenças de contexto, formatos de saída ou suporte a ferramentas. Recursos específicos precisam ser declarados e verificados. [Referência: abstrações de IA em .NET](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai).

## Experimento proposto

**Pré-requisito:** o fluxo e as falhas controladas da [etapa 1](01-api-dotnet-com-llm.md).

1. Definir dois adaptadores, inicialmente simulados, e uma matriz com capacidades, destinos de dados permitidos e limites.
2. Criar uma política simples: escolher apenas destinos autorizados que suportem os recursos exigidos.
3. Simular falha transitória no destino principal e permitir uma única alternativa dentro de um tempo total limitado.
4. Simular credencial inválida, entrada inválida e bloqueio por política. Nesses casos, devolver falha controlada sem tentar contornar o problema com outro fornecedor.

Para verificar comportamento real com múltiplos provedores, repetir depois com dois destinos reais autorizados. Simulação comprova a política, não a compatibilidade de serviços externos.

## Evidências e critérios de sucesso

- O registro da execução explica a escolha e identifica todas as tentativas.
- Um destino sem suporte a ferramentas não recebe uma operação que as exige.
- Dados restritos não são encaminhados a um provedor fora da política, mesmo durante falhas.
- O orçamento de tentativas é finito; resposta parcial não provoca troca silenciosa de modelo.

## Vantagens, limitações e erros comuns

O gateway facilita governança e substituição. Em troca, exige testes de compatibilidade e pode concentrar falhas caso seja mal operado.

Evite empilhar novas tentativas no SDK, no adaptador e no gateway: a multiplicação aumenta latência e cobrança. Depois de uma ferramenta alterar estado, repetir a interação pode repetir o efeito; a [etapa 6](06-function-calling.md) trata dessa proteção.

Com um único provedor e poucas políticas, um adaptador direto pode ser suficiente. A abstração deve crescer junto com a necessidade.

**Desafio da caverna:** o destino principal caiu, mas o segundo não pode receber os documentos. Qual resposta preserva o contrato com o usuário?

[Anterior: API .NET](01-api-dotnet-com-llm.md) · [Visão geral](README.md) · [Próxima: Prompt Engineering](03-prompt-engineering.md)
