# 5. RAG com documentos próprios

> **Resumo:** o assistente passa a consultar o acervo da tribo antes de responder. Precisamos avaliar tanto a busca quanto o uso das evidências encontradas.

## Pedra fundamental

**RAG**, de *Retrieval-Augmented Generation* (geração aumentada por recuperação), combina a recuperação de informações com a geração de uma resposta. A aplicação seleciona conteúdo relevante e o inclui no contexto enviado ao modelo. Isso não treina novamente o modelo nem garante que a resposta respeitará as fontes.

<small>Antes de contar a história da caçada, o xamã consulta o registro da tribo. Ainda precisamos conferir se ele leu a pedra certa.</small>

Há dois fluxos: **ingestão**, que prepara e indexa os documentos, e **consulta**, que recupera trechos para responder. **Chunking (divisão em trechos)** recorta o conteúdo em unidades pesquisáveis, preservando contexto suficiente para interpretação. [Referência: RAG em aplicações .NET](https://learn.microsoft.com/en-us/dotnet/ai/conceptual/rag).

## O problema e o encaixe arquitetural

“Como registrar o empréstimo?” agora deve ser respondido com base no procedimento oficial fictício, incluindo sua versão. Uma explicação genérica já não basta.

O comando `ImportarDocumento` coordena extração, divisão, embeddings e persistência. A consulta `BuscarTrechos` recupera conteúdo autorizado. O caso de uso de resposta monta o contexto, chama o modelo e devolve referências verificáveis. Adaptadores cuidam dos formatos de arquivo, banco e provedor.

Começar com texto ou Markdown evita misturar o aprendizado de RAG com extração de PDFs digitalizados. Conteúdo mal extraído produz busca ruim, mesmo com um modelo excelente.

## Experimento proposto

**Pré-requisito:** a recuperação da [etapa 4](04-embeddings-e-pgvector.md).

1. Preparar três documentos fictícios com identificador, versão, vigência e permissão de acesso. Definir uma regra explícita para versões substituídas.
2. Dividir por seções, preservar a relação com a origem e registrar a configuração usada. Uma nova importação da mesma versão não deve duplicar trechos ativos.
3. Fazer perguntas com resposta presente, ausente, conflitante e restrita. Recuperar trechos permitidos e montar um contexto com tamanho limitado.
4. Solicitar resposta com referências aos trechos. Verificar se as referências existem e se realmente sustentam as afirmações. Retirar um documento e repetir a consulta.

## Evidências e critérios de sucesso

- A resposta identifica fontes e versões, sem inventar referências.
- A falta de evidência produz uma indicação de insuficiência, com avaliação dos casos em que o modelo falhar nesse comportamento.
- Conteúdo removido ou sem permissão não é enviado ao modelo, inclusive se houver cópias em cache (armazenamento temporário).
- Uma falha de recuperação é distinguida de uma resposta que distorceu trechos corretos.

## Vantagens, limitações e erros comuns

RAG permite atualizar o acervo sem alterar os pesos do modelo. Em troca, exige manutenção da ingestão, das permissões, da qualidade da busca e das avaliações.

Trechos pequenos demais perdem significado; grandes demais carregam ruído e consomem contexto. Sobreposição entre trechos pode ajudar a preservar continuidade, mas aumenta duplicação. Escolher tamanho e quantidade com perguntas representativas.

Documentos são dados não confiáveis: um trecho pode conter uma instrução maliciosa. Recuperar conteúdo não deve conceder poderes adicionais ao assistente. Para um acervo minúsculo, enviar diretamente um texto autorizado pode bastar; para regras exatas, uma consulta estruturada pode ser melhor.

**Desafio da caverna:** inclua duas versões com orientações diferentes. Demonstre qual delas deve valer e como o sistema evita responder com uma mistura das duas.

[Anterior: embeddings e pgvector](04-embeddings-e-pgvector.md) · [Visão geral](README.md) · [Próxima: Function Calling](06-function-calling.md)
