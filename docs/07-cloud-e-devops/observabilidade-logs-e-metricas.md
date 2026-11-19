# Observabilidade, logs e métricas

> **Resumo:** observabilidade é a capacidade de entender o estado interno de um sistema a partir de sinais externos. Logs, métricas e traces (rastreamentos) respondem perguntas diferentes.

## Onde aplicar

- **Logs:** eventos detalhados, como falha de validação ou transação recusada.
- **Métricas:** séries numéricas, como latência, taxa de erro e fila pendente.
- **Traces:** caminho de uma requisição entre componentes.

<small>Log conta o que aconteceu, métrica mostra a febre e trace acompanha o viajante por todas as cavernas.</small>

## Exemplo em C#

```csharp
public async Task<Produto?> ObterAsync(
    Guid id, CancellationToken cancellationToken)
{
    using var atividade = atividadeSource.StartActivity("catalogo.obter_produto");
    atividade?.SetTag("produto.id", id);
    logger.LogInformation("Consultando produto {ProdutoId}", id);

    var produto = await repositorio.ObterAsync(id, cancellationToken);
    if (produto is null)
        logger.LogWarning("Produto {ProdutoId} não encontrado", id);

    return produto;
}
```

Não registre senhas, tokens ou dados pessoais sem necessidade. Prefira logs estruturados, níveis coerentes e correlação de requisições.

## Métricas úteis

- taxa e duração de requisições;
- erros por operação;
- profundidade e idade de mensagens;
- acertos e falhas de cache;
- consumo de recursos.

Métrica sem ação definida vira decoração de parede. Para cada alerta, documente o que investigar.
