# Idempotência e duplicidade de registros

> **Resumo:** uma operação idempotente pode ser repetida sem efeitos adicionais indevidos. Isso é essencial quando redes falham e mensagens são reentregues.

## Pedra fundamental

**Idempotência** garante que repetir a mesma intenção não crie dois pedidos, pagamentos ou cadastros. A segunda execução pode consultar o resultado já registrado.

<small>Se o mensageiro tropeçar e entregar a mesma tabuleta duas vezes, a tribo não deve construir duas casas.</small>

## Exemplo em C#

```csharp
public async Task<Pedido> CriarAsync(
    CriarPedidoCommand command, CancellationToken cancellationToken)
{
    var existente = await pedidos.BuscarPorChaveIdempotenciaAsync(
        command.ChaveIdempotencia, cancellationToken);

    if (existente is not null) return existente;

    var pedido = Pedido.Criar(command);
    await pedidos.SalvarComIndiceUnicoAsync(
        pedido, command.ChaveIdempotencia, cancellationToken);
    return pedido;
}
```

Em produção, use índice ou restrição única no banco, trate corridas entre requisições e defina expiração de chaves.

## Desafio da caverna

Simule duas requisições simultâneas com a mesma chave e prove que somente um registro é criado.
