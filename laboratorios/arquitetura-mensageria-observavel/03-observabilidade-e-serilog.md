# 3. Observabilidade e Serilog

## O problema dos logs soltos

Esta mensagem ajuda pouco:

```text
Falha ao processar pedido
```

Ela não informa qual pedido, qual mensagem, qual trace, qual tentativa ou quanto tempo o processamento levou. Logs estruturados transformam contexto em propriedades pesquisáveis.

```csharp
logger.LogError(
    exception,
    "Falha ao processar pedido {PedidoId} da mensagem {MessageId} no trace {TraceId}",
    pedidoId,
    messageId,
    Activity.Current?.TraceId.ToString());
```

O `ILogger<T>` continua sendo uma boa porta de entrada. Serilog pode funcionar como implementação e encaminhar logs para console, arquivo ou outros destinos.

## Quando Serilog ajuda

Serilog é útil quando precisamos de logs estruturados, enriquecimento consistente, múltiplos sinks (destinos de log) ou uma política de filtragem mais expressiva. Ele não substitui métricas, traces, alertas ou um backend de observabilidade.

```csharp
using Serilog;

Log.Logger = new LoggerConfiguration()
    .Enrich.FromLogContext()
    .Enrich.WithProperty("Application", "Brucutu.Pedidos")
    .WriteTo.Console()
    .CreateLogger();

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseSerilog();
```

Em uma API ASP.NET Core, o request logging pode condensar vários eventos de uma requisição em um evento com método, rota, status e duração:

```csharp
var app = builder.Build();

app.UseSerilogRequestLogging();

app.MapGet("/pedidos/{id:guid}", (Guid id) =>
    Results.Ok(new { id }));

app.Run();
```

Coloque o middleware antes dos handlers que deseja medir. Os filtros de nível devem ser ajustados para reduzir ruído do framework sem esconder falhas relevantes.

## Escopo de propriedades

Ao consumir uma mensagem, use um escopo para manter propriedades comuns em todos os eventos daquele processamento:

```csharp
using (logger.BeginScope(new Dictionary<string, object>
{
    ["MessageId"] = messageId,
    ["MessageType"] = "pedido.criado",
    ["TraceId"] = Activity.Current?.TraceId.ToString() ?? "none"
}))
{
    logger.LogInformation("Processando mensagem");
    await ProcessarAsync(cancellationToken);
    logger.LogInformation("Mensagem processada");
}
```

## Regras de segurança e qualidade

- Não registre tokens, senhas, payloads completos ou dados pessoais sem necessidade.
- Prefira identificadores técnicos e valores mascarados.
- Use níveis coerentes: `Information` para fluxo normal, `Warning` para condições recuperáveis, `Error` para falhas que exigem investigação.
- Não registre a mesma exceção dezenas de vezes em camadas diferentes.
- Inclua `trace.id`, `message.id`, tipo da mensagem e tentativa quando isso ajudar a investigação.
- Defina retenção e acesso aos logs antes de enviá-los para um serviço externo.

## Observabilidade não é instalar um painel

Um painel bonito não responde automaticamente se uma mensagem foi perdida, duplicada ou rejeitada. Antes de escolher um backend, escreva perguntas operacionais:

- Qual é a taxa de erro por tipo de mensagem?
- Qual é a idade da mensagem mais antiga na fila?
- Qual é o tempo p95 de processamento?
- Quantas mensagens foram reentregues?
- Qual trace conecta a requisição original ao erro do consumer?

Depois escolha os sinais, a retenção, os alertas e o destino que tornam essas perguntas respondíveis.

## Referências

- [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore)
- [Observabilidade, logs e métricas no Brucutu](../../docs/07-cloud-e-devops/observabilidade-logs-e-metricas.md)
