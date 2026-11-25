# 3. Prompt Engineering

> **Resumo:** escrever instruções claras ajuda o modelo, mas a melhoria precisa aparecer em uma avaliação. Uma frase bonita não vale como teste de qualidade.

## Pedra fundamental

**Prompt Engineering (engenharia de instruções)** é a prática de organizar tarefa, contexto, exemplos, restrições e formato de resposta para orientar o modelo. O **prompt** é o conjunto de instruções e informações apresentado na interação.

<small>Pedir “resolva aí” para o xamã é um ótimo jeito de receber uma previsão sobre o clima quando você precisava do estoque de lanças.</small>

**Zero-shot (sem exemplos)** apresenta a tarefa diretamente. **Few-shot (com alguns exemplos)** acrescenta pares de entrada e saída para demonstrar o comportamento desejado. Esses exemplos ocupam contexto e não alteram os pesos do modelo. [Referência: conceitos de prompt engineering](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/prompt-engineering-dotnet).

## O problema e o encaixe arquitetural

Queremos explicar procedimentos em português claro, reconhecer lacunas e manter um formato previsível. “Seja útil” deixa muitas decisões implícitas.

As instruções da tarefa ficam próximas ao caso de uso em `Application`, com versão e avaliação. O adaptador converte essa intenção para os papéis e parâmetros aceitos pelo provedor. Regras como acesso a documentos e autorização de ações continuam em código.

## Experimento proposto

**Pré-requisito:** a integração da [etapa 1](01-api-dotnet-com-llm.md). Manter modelo e configuração fixos durante a comparação, mesmo que o gateway já exista.

1. Montar dez casos fictícios: seis com contexto suficiente, dois ambíguos e dois sem a informação necessária. Definir antes o que caracteriza uma resposta aceitável.
2. Comparar “Explique o procedimento” com uma instrução estruturada, como a proposta abaixo.
3. Repetir cada caso três vezes por versão. Registrar acertos, omissões, afirmações sem apoio, duração e consumo.
4. Alterar uma variável por vez e guardar uma versão que melhore os critérios acordados, incluindo as regressões encontradas.

Exemplo de instrução, para estudo:

```text
Tarefa: explicar o procedimento descrito no contexto fornecido.
Público: novo integrante da tribo.
Formato: objetivo, passos e cuidados, em português brasileiro.
Limite: não completar informações ausentes com suposições.
Se faltar informação: indicar a lacuna e pedir esclarecimento.
Contexto e pergunta serão fornecidos separadamente pela aplicação.
```

Texto do usuário e documentos são entradas não confiáveis. Separá-los das instruções facilita interpretação e revisão, mas não cria uma barreira de segurança suficiente.

## Evidências e critérios de sucesso

- A avaliação usa os mesmos casos e critérios nas duas versões.
- As respostas sem apoio e as solicitações de esclarecimento são contabilizadas.
- Formatos estruturados são validados pela aplicação; pedir JSON não garante um contrato válido.
- Resultados incluem versão do prompt, modelo e parâmetros. Poucos casos revelam problemas, mas não comprovam qualidade geral.

## Vantagens, limitações e erros comuns

Boas instruções reduzem ambiguidade com baixo custo inicial. Exemplos longos aumentam consumo, manutenção e risco de induzir respostas inadequadas fora dos casos demonstrados.

Uma temperatura menor, quando suportada, pode reduzir variação, mas não garante determinismo nem verdade. Evite testar apenas perguntas fáceis ou trocar prompt e modelo juntos sem distinguir seus efeitos.

**Prompt injection (injeção de instruções)** tenta desviar o comportamento por entradas do usuário ou conteúdo externo. O prompt não deve guardar segredos nem ser responsável por conceder permissões. [Referência: OWASP sobre prompt injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/).

**Desafio da caverna:** acrescente um caso que peça para ignorar as instruções. Registre a resposta e quais controles da aplicação continuam válidos mesmo se o modelo obedecer ao pedido indevido.

[Anterior: AI Gateway](02-ai-gateway.md) · [Visão geral](README.md) · [Próxima: embeddings e pgvector](04-embeddings-e-pgvector.md)
