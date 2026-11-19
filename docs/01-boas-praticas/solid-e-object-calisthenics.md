# SOLID e Object Calisthenics

> **Resumo:** SOLID ajuda a raciocinar sobre responsabilidades e dependências. Object Calisthenics (calistenia de objetos) usa restrições como exercícios de design, não como leis universais.

## Pedra fundamental

**SOLID** reúne responsabilidade única, aberto/fechado, substituição de Liskov, segregação de interfaces e inversão de dependência. Object Calisthenics propõe métodos pequenos, pouca indentação e encapsulamento cuidadoso para treinar esse olhar.

<small>SOLID é uma bússola, não um GPS que conhece todas as cavernas.</small>

## Código fossilizado

```csharp
public class PedidoService
{
    public void Processar(Pedido pedido)
    {
        // valida, calcula preço, salva, envia e-mail e escreve log.
    }
}
```

Uma classe com muitos motivos para mudar viola o espírito da responsabilidade única.

## Evolução

```csharp
public interface IPedidoRepository
{
    Task SalvarAsync(Pedido pedido, CancellationToken cancellationToken);
}

public interface INotificadorPedido
{
    Task NotificarCriacaoAsync(Pedido pedido, CancellationToken cancellationToken);
}

public sealed class CriarPedidoHandler(
    IPedidoRepository pedidos,
    INotificadorPedido notificador)
{
    public async Task HandleAsync(Pedido pedido, CancellationToken cancellationToken)
    {
        pedido.Validar();
        await pedidos.SalvarAsync(pedido, cancellationToken);
        await notificador.NotificarCriacaoAsync(pedido, cancellationToken);
    }
}
```

Interfaces expressam necessidades e permitem trocar infraestrutura. Em produção, ainda é preciso decidir transação, idempotência e falhas.

## Trade-offs e quando não utilizar

Interfaces demais podem esconder o fluxo. Não crie abstrações apenas para satisfazer um acrônimo: use princípios quando houver pressão real de mudança, teste ou substituição.

## Desafio da caverna

Escolha uma classe grande, liste seus motivos de mudança e separe apenas uma responsabilidade. Compare a legibilidade antes e depois.
