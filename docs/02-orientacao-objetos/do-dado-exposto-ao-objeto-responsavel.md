# Do dado exposto ao objeto responsável

> **Resumo:** uma classe útil não é apenas um recipiente de propriedades. Este artigo mostra como reconhecer classes anêmicas, proteger invariantes com encapsulamento e escolher entre classe, `record`, entidade e serviço conforme a identidade e o comportamento do conceito.

## Público-alvo e pré-requisitos

- **Público-alvo:** pessoas que já escrevem classes em C# e querem modelar responsabilidades com mais clareza.
- **Pré-requisitos:** noções de tipos, métodos, propriedades e construtores.

## Objetivos de aprendizagem

Ao final, a pessoa leitora deverá conseguir:

- reconhecer quando uma classe está apenas transportando dados;
- proteger uma regra de negócio no lugar em que ela pode ser mantida;
- diferenciar entidade, objeto de valor, `record` e serviço de aplicação;
- avaliar quando uma classe deve ser fechada para extensão ou quando a composição é mais adequada.

## Problema que queremos resolver

É comum começar com uma classe cheia de propriedades públicas e colocar toda a lógica em um serviço externo. O código funciona, mas qualquer parte da aplicação pode criar estados inválidos e ninguém sabe claramente onde uma regra deve morar.

Isso não significa que toda classe com propriedades seja ruim. DTOs (*Data Transfer Objects*, objetos de transferência de dados), modelos de entrada e respostas de API frequentemente existem justamente para transportar dados. O problema aparece quando um conceito que possui regras importantes é tratado como uma sacola pública de campos.

## Pedra fundamental: dados, estado e comportamento

Uma classe deve encapsular o estado que precisa permanecer consistente e oferecer operações que expressem as decisões permitidas. **Encapsulamento** não é esconder tudo por reflexo: é controlar como o estado pode ser observado e alterado.

<small>Propriedade pública com `set` em toda esquina é como deixar a porta da caverna aberta e torcer para nenhum dinossauro mexer no estoque.</small>

Uma **invariante** é uma condição que deve continuar verdadeira. Para um pedido, por exemplo, a soma dos itens não deve ser negativa e um pedido já cancelado não deve ser enviado. A classe responsável pelo conceito é um bom lugar para proteger essas condições quando elas não dependem de infraestrutura externa.

## Classe anêmica: o cheiro, não o veredito

Uma **classe anêmica** armazena dados, mas quase não possui comportamento. O termo descreve um possível *code smell* (sinal de design que merece investigação), não uma condenação automática.

```csharp
public class Pedido
{
    public List<ItemPedido> Itens { get; set; } = [];
    public string Status { get; set; } = "Aberto";
    public decimal Total { get; set; }
}

public class PedidoService
{
    public void Cancelar(Pedido pedido)
    {
        if (pedido.Status == "Enviado")
            throw new InvalidOperationException("Pedido enviado não pode ser cancelado.");

        pedido.Status = "Cancelado";
        pedido.Total = 0;
    }
}
```

O exemplo permite que qualquer consumidor altere `Status`, `Itens` e `Total` sem respeitar as relações entre eles. O serviço também pode se multiplicar em várias versões, cada uma aplicando uma regra ligeiramente diferente.

Uma classe anêmica pode ser correta quando seu propósito é transportar dados entre fronteiras. Para um objeto de domínio com invariantes, ela merece uma pergunta: **quem deveria ser responsável por impedir este estado inválido?**

## Evolução: comportamento perto da regra

```csharp
public enum StatusPedido
{
    Aberto,
    Enviado,
    Cancelado
}

public sealed class Pedido
{
    private readonly List<ItemPedido> itens = [];

    public IReadOnlyCollection<ItemPedido> Itens => itens.AsReadOnly();
    public StatusPedido Status { get; private set; } = StatusPedido.Aberto;
    public decimal Total => itens.Sum(item => item.Preco * item.Quantidade);

    public void AdicionarItem(ItemPedido item)
    {
        ArgumentNullException.ThrowIfNull(item);

        if (Status != StatusPedido.Aberto)
            throw new InvalidOperationException("Só é possível alterar um pedido aberto.");

        itens.Add(item);
    }

    public void Cancelar()
    {
        if (Status == StatusPedido.Enviado)
            throw new InvalidOperationException("Pedido enviado não pode ser cancelado.");

        Status = StatusPedido.Cancelado;
    }
}

public sealed record ItemPedido
{
    public string Produto { get; }
    public decimal Preco { get; }
    public int Quantidade { get; }

    public ItemPedido(string produto, decimal preco, int quantidade)
    {
        if (string.IsNullOrWhiteSpace(produto))
            throw new ArgumentException("Produto é obrigatório.", nameof(produto));
        if (preco < 0)
            throw new ArgumentOutOfRangeException(nameof(preco));
        if (quantidade <= 0)
            throw new ArgumentOutOfRangeException(nameof(quantidade));

        Produto = produto;
        Preco = preco;
        Quantidade = quantidade;
    }
}
```

Este é um exemplo didático simplificado, não uma implementação pronta para produção. A mudança importante é conceitual: `Pedido` controla transições e calcula o total; `ItemPedido` protege sua própria validade; consumidores recebem uma visão de leitura dos itens.

## Record, classe e identidade

`record` é uma boa opção quando a igualdade por valor representa o conceito. Dois `ItemPedido` com os mesmos dados podem ser considerados equivalentes para o caso de uso. Ele também oferece sintaxe concisa para dados imutáveis, embora não elimine a necessidade de validar invariantes.

Uma classe tradicional costuma ser mais natural quando o objeto possui identidade própria, ciclo de vida ou transições. Um pedido com o mesmo identificador continua sendo o mesmo pedido mesmo que seu status mude. Essa é uma decisão de modelagem, não apenas uma preferência de sintaxe.

| Conceito | Pergunta principal | Forma comum |
| --- | --- | --- |
| DTO | Preciso transportar dados entre fronteiras? | `record` ou classe simples |
| Objeto de valor | Igualdade por valor faz sentido? | `record` com validação |
| Entidade | O conceito tem identidade e ciclo de vida? | Classe com comportamento |
| Serviço | A operação coordena objetos ou depende de infraestrutura? | Classe com dependências explícitas |

## O que significa uma classe “fechada”

Há duas ideias próximas, mas diferentes:

- **Fechada por encapsulamento:** seus detalhes internos não podem ser alterados livremente. Isso é obtido com campos privados, coleções somente leitura, construtores válidos e métodos que representam operações permitidas.
- **Fechada para extensão:** não se deseja permitir herança. Em C#, `sealed` torna essa decisão explícita, reduz superfícies de extensão e protege invariantes contra subclasses que poderiam quebrar expectativas.

“Fechada” não significa “imutável” e nem “melhor em qualquer situação”. Um tipo pode ser `sealed` e ainda ter estado mutável; também pode ser extensível e continuar bem encapsulado. Feche uma classe quando a herança não fizer parte do contrato ou quando a composição comunicar melhor a intenção.

## Comportamento, colaboração e limites

Colocar comportamento no objeto não significa empurrar toda a aplicação para dentro dele. Uma regra que depende de relógio, banco de dados, rede ou vários agregados pode pertencer a um serviço de aplicação ou de domínio. O critério é a responsabilidade:

- regra sobre o próprio estado: tende a ficar no objeto;
- coordenação de vários objetos: tende a ficar em um serviço;
- transporte entre camadas: tende a usar DTOs;
- persistência e efeitos externos: ficam atrás de portas ou adaptadores apropriados.

O resultado desejado é uma colaboração explícita, não um objeto “deus” que conhece tudo.

## Trade-offs (compensações e consequências) e quando **não** utilizar

- Encapsulamento forte protege regras, mas pode exigir mais métodos e decisões de modelagem.
- `record` reduz cerimônia para objetos por valor, mas igualdade estrutural pode ser inadequada para entidades.
- `sealed` reduz extensibilidade por herança, mas composição, interfaces e decoradores continuam disponíveis.
- Métodos no domínio aproximam regra e estado, mas não substituem serviços quando há coordenação ou efeitos externos.
- DTOs anêmicos são apropriados nas fronteiras; tentar transformá-los em entidades pode misturar contratos de transporte com regras de negócio.

## Erros comuns

- Trocar `public set` por `private set` e continuar permitindo qualquer transição por métodos genéricos.
- Recalcular um valor em um serviço e também armazená-lo na entidade, criando duas fontes de verdade.
- Usar `record` apenas porque a sintaxe é curta, sem decidir se igualdade por valor é desejada.
- Criar interfaces e hierarquias antes de existir uma necessidade de substituição.
- Esconder efeitos externos dentro de um método de entidade, tornando testes e transações difíceis de compreender.

## Desafio da caverna

Escolha uma classe do seu código que tenha pelo menos três propriedades públicas mutáveis.

1. Liste suas invariantes, por exemplo, combinações de estado que nunca deveriam ocorrer.
2. Identifique quais alterações pertencem ao próprio objeto e quais exigem coordenação externa.
3. Remova uma fonte de mutação direta e crie uma operação com nome de negócio.
4. Escreva um teste que prove uma transição válida e outro que rejeite uma transição inválida.

Critério de sucesso: uma pessoa que não conhece a implementação consegue descobrir, pelos métodos públicos, quais operações o objeto permite.

## Resumo e referências

- Classe anêmica é um sinal para investigar, não um pecado universal.
- Encapsulamento protege invariantes e torna as transições explícitas.
- `record` comunica igualdade por valor; entidades normalmente dependem de identidade.
- Classes fechadas para extensão são uma decisão de contrato, não um substituto para encapsulamento.
- A responsabilidade deve ficar próxima dos dados que ela protege, sem absorver coordenação e infraestrutura indevidamente.

Referências:

- [C# Reference: record types](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record)
- [C# Reference: sealed](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/sealed)
- [Martin Fowler, Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html)

## Próximos estudos

- Conteúdo relacionado já disponível: [SOLID e Object Calisthenics](../01-boas-praticas/solid-e-object-calisthenics.md).
- Conteúdo planejado: composição versus herança, objetos de valor e colaboração entre objetos.
