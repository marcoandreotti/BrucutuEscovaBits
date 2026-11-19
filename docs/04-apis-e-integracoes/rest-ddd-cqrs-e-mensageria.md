# RESTful, DDD, CQRS e mensageria

> **Resumo:** APIs RESTful, modelagem orientada ao domínio e mensageria resolvem problemas diferentes. Combiná-las exige entender contratos, consistência e falhas.

## Pedra fundamental

- **RESTful:** estilo de API que usa recursos, representações, verbos HTTP e status coerentes.
- **DDD**, *Domain-Driven Design* (design orientado ao domínio): organiza o modelo em torno do problema de negócio.
- **CQRS**, *Command Query Responsibility Segregation* (separação entre comandos e consultas): separa modelos ou fluxos quando essa diferença traz benefício.
- **Mensageria:** comunicação assíncrona por mensagens entre produtores e consumidores.

<small>REST é a placa da caverna, DDD é o mapa do território, CQRS separa “mude a parede” de “meça a parede” e a mensageria é o mensageiro que não precisa esperar na porta.</small>

## RESTful em C#

```csharp
[ApiController]
[Route("api/produtos")]
public sealed class ProdutosController(IProdutoCatalogo catalogo) : ControllerBase
{
    [HttpGet("{id:guid}")]
    public async Task<ActionResult<ProdutoResponse>> Obter(
        Guid id, CancellationToken cancellationToken)
    {
        var produto = await catalogo.ObterAsync(id, cancellationToken);
        return produto is null ? NotFound() : Ok(ProdutoResponse.From(produto));
    }
}
```

Não transforme cada método em um verbo inventado nem esconda comandos importantes em `GET`.

## Mensageria: quando e qual

| Opção | Quando considerar | Cuidado principal |
| --- | --- | --- |
| API síncrona | Resposta imediata e contrato direto | Timeout e acoplamento temporal |
| Webhook | Notificar outro sistema | Assinatura, retries (repetições) e idempotência |
| RabbitMQ | Filas e roteamento flexível | Reentrega e operação |
| Kafka | Alto volume e replay (releitura) | Partições, ordenação e retenção |
| Amazon SQS | Fila gerenciada na AWS | Duplicidade e fila de mensagens mortas |

DDD não exige microsserviços. CQRS também não deve ser adotado antes de existir uma diferença real entre leitura e escrita.

## Desafio da caverna

Modele o cadastro de preço como API síncrona e comando assíncrono. Registre latência, falha, reprocessamento e consistência esperada.
