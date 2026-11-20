# 1. Mensageria com RabbitMQ

## O que estamos investigando

Queremos desacoplar o momento em que um evento é produzido do momento em que ele é processado. **Mensageria** permite que um componente envie uma mensagem para um broker, que a armazena e a entrega a consumidores conforme a topologia configurada.

RabbitMQ é útil quando há necessidade de processamento assíncrono, absorção de picos, integração entre processos ou distribuição de trabalho. Ele não é obrigatório para toda chamada HTTP. Para uma operação simples, síncrona e com resposta imediata, introduzir um broker pode adicionar custo operacional sem resolver um problema real.

<small>Se o vizinho precisa responder agora, gritar pela caverna pode bastar. Se a tarefa pode esperar, o mensageiro evita interromper a caçada.</small>

## Vocabulário mínimo

- **Publisher ou producer:** publica a mensagem.
- **Exchange:** recebe a mensagem e decide para quais filas ela será roteada.
- **Queue:** armazena mensagens até que um consumer as processe.
- **Consumer:** recebe e processa mensagens.
- **Routing key:** informação usada pelo exchange para roteamento.
- **Ack:** confirmação de que o processamento terminou.
- **Nack ou reject:** sinal de que a mensagem não foi processada.

Uma fila não é apenas uma lista. Ela participa de uma política de durabilidade, ordem, concorrência, reentrega e retenção. Essas escolhas pertencem ao contrato operacional da mensagem.

## Publisher didático

O exemplo usa o cliente RabbitMQ para .NET 7. Conexões e canais devem ser reutilizados em uma aplicação real, em vez de serem abertos para cada mensagem. O trecho é simplificado para destacar o fluxo.

```csharp
using System.Text;
using RabbitMQ.Client;

public sealed class PedidoPublisher(IChannel channel)
{
    public async Task PublicarAsync(
        string json,
        CancellationToken cancellationToken)
    {
        await channel.ExchangeDeclareAsync(
            exchange: "pedidos",
            type: ExchangeType.Direct,
            durable: true,
            autoDelete: false);

        var body = Encoding.UTF8.GetBytes(json);
        var propriedades = new BasicProperties
        {
            ContentType = "application/json",
            DeliveryMode = DeliveryModes.Persistent,
            MessageId = Guid.NewGuid().ToString("N")
        };

        await channel.BasicPublishAsync(
            exchange: "pedidos",
            routingKey: "pedido.criado",
            mandatory: true,
            basicProperties: propriedades,
            body: body,
            cancellationToken: cancellationToken);
    }
}
```

Este trecho ainda precisa de uma fila vinculada ao exchange. Em produção, considere publisher confirms e tratamento de mensagens não roteáveis. Publicar sem confirmar que o broker recebeu a mensagem pode criar uma falsa sensação de sucesso.

## Consumer com confirmação manual

```csharp
using System.Text.Json;
using RabbitMQ.Client;
using RabbitMQ.Client.Events;

public sealed class PedidoConsumer(IChannel channel)
{
    public async Task ExecutarAsync(CancellationToken cancellationToken)
    {
        await channel.QueueDeclareAsync(
            queue: "pedidos-processamento",
            durable: true,
            exclusive: false,
            autoDelete: false);

        await channel.QueueBindAsync(
            queue: "pedidos-processamento",
            exchange: "pedidos",
            routingKey: "pedido.criado");

        await channel.BasicQosAsync(
            prefetchSize: 0,
            prefetchCount: 10,
            global: false);

        var consumer = new AsyncEventingBasicConsumer(channel);
        consumer.ReceivedAsync += async (_, entrega) =>
        {
            try
            {
                var json = entrega.Body.ToArray();
                var pedido = JsonSerializer.Deserialize<PedidoCriado>(json)
                    ?? throw new InvalidOperationException("Mensagem vazia.");

                await ProcessarAsync(pedido, cancellationToken);
                await channel.BasicAckAsync(entrega.DeliveryTag, multiple: false);
            }
            catch (Exception)
            {
                await channel.BasicNackAsync(
                    entrega.DeliveryTag,
                    multiple: false,
                    requeue: false);
            }
        };

        await channel.BasicConsumeAsync(
            queue: "pedidos-processamento",
            autoAck: false,
            consumer: consumer,
            cancellationToken: cancellationToken);

        await Task.Delay(Timeout.Infinite, cancellationToken);
    }

    private static Task ProcessarAsync(
        PedidoCriado pedido,
        CancellationToken cancellationToken) => Task.CompletedTask;
}

public sealed record PedidoCriado(Guid PedidoId, decimal Total);
```

O exemplo usa `requeue: false` para não criar um ciclo infinito de falha. Em uma solução completa, a mensagem poderia ser encaminhada para uma dead-letter queue, classificada como erro permanente ou submetida a retry com limite.

## Decisões que o laboratório deve tornar visíveis

- **Durabilidade:** a mensagem e a fila sobrevivem à reinicialização?
- **Confirmação:** o publisher usa confirms? O consumer confirma só depois do efeito desejado?
- **Duplicidade:** o que acontece se a mensagem for entregue duas vezes?
- **Ordem:** a regra depende da ordem global ou apenas da ordem por entidade?
- **Retry:** qual falha é transitória e qual é permanente?
- **Dead-letter:** onde mensagens que não podem ser processadas serão investigadas?

## Quando não utilizar

Não introduza RabbitMQ apenas para chamar um método em outro processo. Uma fila traz broker, topologia, credenciais, monitoramento, operação, reentrega e consistência eventual. Se o problema é uma função local ou uma chamada síncrona simples, uma abstração em memória ou HTTP pode ser mais honesta.

## Referências

- [Tutorial Hello World do RabbitMQ para .NET](https://www.rabbitmq.com/tutorials/tutorial-one-dotnet)
- [Publisher confirms](https://www.rabbitmq.com/tutorials/tutorial-seven-dotnet)
