# Código limpo, DRY e YAGNI

> **Resumo:** código limpo é código fácil de ler, alterar e testar. DRY e YAGNI ajudam a controlar repetição e complexidade, mas não são desculpas para abstrações prematuras.

## Público-alvo e pré-requisitos

Para quem já escreve pequenos programas em C# e quer melhorar legibilidade e manutenção.

## Objetivos de aprendizagem

- reconhecer sinais de código difícil de manter;
- diferenciar duplicação real de semelhança superficial;
- adiar complexidade sem requisito concreto.

## Pedra fundamental

**Clean Code** (código limpo) busca facilitar leitura, mudança e teste. **DRY**, de *Don't Repeat Yourself* (não repita conhecimento), mantém uma regra importante em um único lugar. **YAGNI**, de *You Aren't Gonna Need It* (você não vai precisar disso agora), recomenda não construir extensões especulativas.

<small>O código limpo é como uma caverna bem sinalizada: ninguém precisa bater a cabeça em três pedras antes de encontrar a saída.</small>

## Código fossilizado

```csharp
public decimal CalcularTotal(Pedido pedido, bool clienteVip)
{
    var total = pedido.Itens.Sum(item => item.Preco * item.Quantidade);
    if (clienteVip) total *= 0.9m;

    // A mesma regra apareceu em outro método.
    if (pedido.Itens.Count > 100) total *= 0.95m;
    return total;
}
```

O método mistura cálculo, regras e números mágicos. Se uma regra mudar, alguém precisará procurar todas as cópias.

## Evolução

```csharp
public decimal CalcularTotal(Pedido pedido, TipoCliente cliente)
{
    var subtotal = pedido.Itens.Sum(item => item.ObterSubtotal());
    var desconto = politicaDeDesconto.Calcular(subtotal, cliente, pedido.Itens.Count);
    return subtotal - desconto;
}

public sealed class PoliticaDeDesconto
{
    public decimal Calcular(decimal subtotal, TipoCliente cliente, int quantidadeDeItens)
    {
        var vip = cliente == TipoCliente.Vip ? 0.10m : 0m;
        var volume = quantidadeDeItens > 100 ? 0.05m : 0m;
        return subtotal * Math.Min(vip + volume, 0.20m);
    }
}
```

O exemplo concentra a política e dá nome às regras. Em produção, adicione testes, arredondamento explícito e validações de domínio.

## Trade-offs e quando não utilizar

Duas linhas parecidas podem representar conhecimentos diferentes. Abstrair cedo demais cria acoplamento. Elimine duplicação quando intenção e motivo de mudança forem os mesmos. YAGNI não significa ignorar requisitos conhecidos.

## Desafio da caverna

Encontre uma duplicação, escreva um teste para o comportamento e extraia a menor abstração que preserve a leitura.

## Resumo

Código limpo favorece leitura e mudança. DRY protege conhecimento duplicado; YAGNI controla complexidade especulativa. Ambos dependem de contexto.

## Próximos estudos

- [SOLID e Object Calisthenics](../01-boas-praticas/solid-e-object-calisthenics.md)
- [O que é arquitetura de software?](../03-arquitetura/o-que-e-arquitetura-de-software.md)
