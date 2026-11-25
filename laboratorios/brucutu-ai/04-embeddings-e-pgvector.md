# 4. Embeddings e PostgreSQL com pgvector

> **Resumo:** uma busca literal procura palavras. Uma busca vetorial pode aproximar trechos relacionados mesmo quando o vocabulário muda. Aproximação não é comprovação.

## Pedra fundamental

Um **embedding (representação vetorial)** transforma uma entrada em uma sequência de números produzida por um modelo. Nesse espaço, uma medida de distância ou similaridade ajuda a encontrar conteúdos relacionados. Essa relação depende do modelo e da tarefa.

<small>“Guardar a lança” e “armazenar a ferramenta” podem morar perto no mapa, mesmo sem compartilhar a mesma placa na porta.</small>

**pgvector** é uma extensão do PostgreSQL que oferece armazenamento e busca de vetores. Ela não gera embeddings por conta própria. A aplicação obtém os vetores de um modelo e os persiste junto dos dados necessários para interpretar os resultados.

A **distância cosseno** compara a orientação dos vetores. No pgvector, o operador `<=>` calcula essa distância: menor valor significa maior proximidade segundo essa medida. Não é uma probabilidade de o conteúdo estar correto. [Referência: pgvector](https://github.com/pgvector/pgvector).

## O problema e o encaixe arquitetural

O assistente precisa localizar o procedimento sobre ferramentas guardadas, mesmo quando a pessoa pergunta por “devolver a lança”.

`Application` define portas como `IGeradorDeEmbedding` e `IBuscadorDeTrechos`. Adaptadores em `Infrastructure` integram o modelo de embeddings e o PostgreSQL. O domínio não depende de operadores SQL nem de tipos específicos da extensão.

Guardar texto, documento de origem, versão, identificação do trecho e permissões é tão importante quanto guardar o vetor. Registrar modelo, configuração e dimensão evita misturar representações incompatíveis. Mesma dimensão não significa mesmo espaço vetorial.

## Experimento proposto

**Pré-requisito:** [etapa 3](03-prompt-engineering.md), um conjunto pequeno de textos fictícios e, para executar, PostgreSQL com pgvector e um modelo de embeddings.

1. Preparar seis trechos com assuntos próximos e diferentes: estoque, empréstimo, devolução e cuidados com ferramentas.
2. Gerar os embeddings dos trechos e das perguntas com configurações compatíveis. Preservar os textos e metadados.
3. Comparar uma busca literal com uma busca vetorial exata, recuperando os três trechos mais próximos e suas distâncias.
4. Incluir uma pergunta fora do assunto e uma pessoa sem acesso a determinado documento. Aplicar as restrições na consulta autorizada, antes de enviar qualquer conteúdo ao modelo.

## Evidências e critérios de sucesso

- Uma paráfrase recupera um trecho relevante mesmo sem palavras idênticas.
- O resultado permite voltar à origem e à versão do texto.
- Documentos sem permissão não aparecem no contexto entregue ao assistente.
- A consulta fora do assunto demonstra que sempre pode existir um “mais próximo”, mesmo sem haver uma resposta útil.

## Vantagens, limitações e erros comuns

Busca vetorial ajuda com paráfrases; busca literal pode ser melhor para códigos e identificadores exatos. Combinar as duas é uma evolução a avaliar, não uma obrigação inicial.

Começar pela busca exata simplifica a comparação. Índices aproximados como **HNSW**, de *Hierarchical Navigable Small World* (grafo hierárquico para busca por vizinhança), podem reduzir latência, com custos de memória, construção e perda de recuperação. Filtros também afetam os resultados desses índices; medir com os dados e permissões reais antes de adotar.

Não misture embeddings de modelos diferentes nem interprete uma distância como certeza. Se mudar o modelo, planeje uma coleção compatível e a reindexação dos documentos. Vetores derivados de dados restritos também exigem proteção.

**Desafio da caverna:** crie uma pergunta cujo melhor resultado ainda seja ruim. Como detectar essa situação sem inventar um limiar universal de distância?

[Anterior: Prompt Engineering](03-prompt-engineering.md) · [Visão geral](README.md) · [Próxima: RAG](05-rag-com-documentos.md)
