# Stack de testes em .NET

> **Resumo:** uma boa estratégia combina testes rápidos, claros e confiáveis. xUnit, FluentAssertions, Bogus, NSubstitute e Testcontainers resolvem partes diferentes do trabalho.

## Ferramentas

- **xUnit:** execução e organização de testes.
- **FluentAssertions:** asserções legíveis.
- **Bogus:** geração de dados; use sementes quando precisar de repetibilidade.
- **NSubstitute:** substitutos para dependências externas.
- **Testcontainers:** dependências reais isoladas em contêineres.

## Exemplo

```csharp
public sealed class CalculadoraDeFreteTests
{
    [Fact]
    public void Deve_calcular_frete_gratis_para_pedidos_acima_do_limite()
    {
        var pedido = new PedidoBuilder().ComTotal(250m).Build();
        var frete = new CalculadoraDeFrete().Calcular(pedido);
        frete.Should().Be(0m);
    }
}
```

**Testing best practices** (boas práticas de testes) incluem nomes que explicam comportamento, isolamento, determinismo e feedback rápido. Não transforme um teste em uma segunda implementação do sistema.

## Quando usar cada camada

- Teste unitário: regra pura e rápida.
- Teste de integração: banco, fila ou adaptador real.
- Testcontainers: quando um fake esconderia riscos importantes.
- Substituto: quando a dependência é lenta, instável ou irrelevante para o cenário.

## Desafio da caverna

Escreva um teste unitário para uma regra de desconto e um teste de integração para sua persistência. Compare tempo, confiança e manutenção.
