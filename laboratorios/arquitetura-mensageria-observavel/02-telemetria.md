# 2. Telemetria: deixando pegadas no fluxo

## Telemetria não é sinônimo de observabilidade

**Telemetria** é a coleta e emissão de sinais sobre o comportamento do sistema. **Observabilidade** é a capacidade de investigar perguntas sobre esse comportamento usando os sinais e o contexto disponível.

Os sinais mais comuns são:

- **Logs:** eventos discretos com contexto;
- **Métricas:** medidas agregadas ao longo do tempo;
- **Traces:** caminho de uma operação por vários componentes.

Sem telemetria, o sistema pode até funcionar, mas a investigação depende de reprodução local e adivinhação. Com telemetria mal planejada, produzimos muito ruído e pouca resposta.

## Trace de uma mensagem

Uma requisição HTTP pode iniciar um trace. O publisher cria um span de publicação; o consumer continua o contexto e cria outro span de processamento. O `traceparent` pode viajar nos headers da mensagem.

```csharp
using System.Diagnostics;

public static class InstrumentacaoPedidos
{
    public static readonly ActivitySource Source =
        new("Brucutu.Pedidos");
}

using var atividade = InstrumentacaoPedidos.Source.StartActivity(
    "rabbitmq.publish",
    ActivityKind.Producer);

atividade?.SetTag("messaging.system", "rabbitmq");
atividade?.SetTag("messaging.destination.name", "pedidos");
atividade?.SetTag("messaging.operation.type", "publish");
atividade?.SetTag("pedido.id", pedidoId);
```

Os nomes das tags devem seguir convenções semânticas quando possível. Não coloque o corpo inteiro da mensagem em tags ou logs: isso aumenta custo e pode expor dados sensíveis.

## Propagação entre publisher e consumer

O contexto precisa ser serializado no header da mensagem e extraído no consumer. A implementação exata depende do propagator configurado, mas a ideia é esta:

```csharp
// Ilustração conceitual. O carrier real deve adaptar a coleção de headers
// do cliente RabbitMQ ao propagator usado pela aplicação.
var headers = new Dictionary<string, string>();
PropagarContextoAtual(headers);

// No consumer:
var contextoPai = ExtrairContexto(headers);
using var atividade = InstrumentacaoPedidos.Source.StartActivity(
    "rabbitmq.process",
    ActivityKind.Consumer,
    contextoPai);
```

O laboratório deve primeiro observar o fluxo sem propagação e depois comparar com a propagação habilitada. A diferença é a capacidade de conectar causa e efeito, não apenas gerar mais identificadores.

## Métricas úteis

Comece com poucas métricas que suportam decisões:

- `messages.published`, contador de mensagens publicadas;
- `messages.processed`, contador de processamentos concluídos;
- `messages.failed`, contador de falhas;
- `message.processing.duration`, duração do processamento;
- `queue.depth`, quantidade de mensagens pendentes, quando disponível.

Uma métrica precisa de uma pergunta associada. Se ninguém sabe qual ação tomar quando a fila cresce, o número é decoração de parede.

## Instrumentação mínima com `Meter`

```csharp
using System.Diagnostics.Metrics;

public static class MetricasPedidos
{
    public static readonly Meter Meter =
        new("Brucutu.Pedidos");

    public static readonly Counter<long> Processadas =
        Meter.CreateCounter<long>("messages.processed");

    public static readonly Counter<long> Falhas =
        Meter.CreateCounter<long>("messages.failed");
}

MetricasPedidos.Processadas.Add(1,
    new KeyValuePair<string, object?>("message.type", "pedido.criado"));
```

## O que observar durante o experimento

1. Publique uma mensagem válida e acompanhe o trace.
2. Force uma falha no processamento e compare log, métrica e trace.
3. Desligue o consumer e observe o crescimento da fila.
4. Reinicie o consumer e observe a recuperação.
5. Publique várias mensagens e compare duração, concorrência e ordem.

## Trade-offs

Instrumentar cada linha aumenta custo, volume e distração. Prefira pontos de decisão, fronteiras entre processos e operações lentas. Para mensagens, `message.id`, `message.type`, `delivery.tag` e `trace.id` costumam ser mais úteis do que despejar o payload.

## Próximos passos

O próximo arquivo mostra como transformar esses sinais em logs estruturados e uma experiência de investigação mais consistente com Serilog.
