# Redis e cache

> **Resumo:** Redis é um armazenamento em memória usado, entre outras possibilidades, para cache, sessões e contadores. Cache acelera leituras, mas cria o problema da validade dos dados.

## Evolução em C#

```csharp
public async Task<Produto?> ObterAsync(
    Guid id, CancellationToken cancellationToken)
{
    var chave = $"produto:{id}";
    var armazenado = await cache.GetStringAsync(chave, cancellationToken);

    if (armazenado is not null)
        return JsonSerializer.Deserialize<Produto>(armazenado);

    var produto = await repositorio.ObterAsync(id, cancellationToken);
    if (produto is not null)
    {
        await cache.SetStringAsync(
            chave, JsonSerializer.Serialize(produto),
            new DistributedCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            }, cancellationToken);
    }

    return produto;
}
```

**Cache hit** (acerto) evita a consulta original; **cache miss** (falha de acerto) busca a fonte. Defina invalidação, tamanho, privacidade e comportamento quando o Redis estiver indisponível.

<small>Cache é como guardar frutas perto da entrada: ótimo quando estão frescas; perigoso quando ninguém troca as antigas.</small>

## Quando não utilizar

Não use cache para esconder consultas ruins antes de medir. Não trate cache como fonte permanente sem assumir esse compromisso explicitamente.
