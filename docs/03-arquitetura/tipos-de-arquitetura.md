# Tipos de arquitetura de software

> **Resumo:** arquiteturas modular, monolítica, em camadas, hexagonal e gateway organizam responsabilidades de formas diferentes. A escolha responde a requisitos, riscos e capacidade operacional.

## O mapa da caverna

- **Monolítica:** um deploy principal; simples de operar, mas pode acumular acoplamento.
- **Monolítica modular:** um deploy com módulos e limites explícitos; boa pedra fundamental.
- **Em camadas:** separa apresentação, aplicação, domínio e infraestrutura.
- **Hexagonal:** coloca o domínio no centro e usa portas e adaptadores.
- **Gateway:** uma entrada que compõe ou protege APIs; YARP (*Yet Another Reverse Proxy*, outro proxy reverso) pode encaminhar requisições .NET.
- **Distribuída:** partes executam separadamente, adicionando rede e operação.

<small>Escolher microsserviços para um produto pequeno é como construir cinco cavernas antes de descobrir onde a tribo vai dormir.</small>

## Exemplo de gateway com YARP

```csharp
// Program.cs — exemplo didático.
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddReverseProxy()
    .LoadFromConfig(builder.Configuration.GetSection("ReverseProxy"));

var app = builder.Build();
app.MapReverseProxy();
app.Run();
```

O gateway pode centralizar roteamento, autenticação e observabilidade. Não deve esconder toda a regra de negócio.

## Trade-offs e desafio

Começar com um monólito modular reduz custo operacional. Separar serviços pode fazer sentido quando há escala independente, autonomia de equipes ou isolamento forte. Desenhe as duas opções para uma API de catálogo e liste três custos de cada uma.

## Resumo

Não existe arquitetura superior em qualquer contexto. Limites claros, dependências compreensíveis e decisões registradas valem mais que nomes sofisticados.
