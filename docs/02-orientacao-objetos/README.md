# Orientação a objetos

Área para modelagem, responsabilidades, encapsulamento e colaboração entre objetos.

Orientação a objetos não é um concurso para ver quem cria mais classes. O objetivo é representar decisões e regras de negócio de modo que cada parte tenha uma responsabilidade compreensível, preserve seus invariantes e colabore com as demais sem conhecer detalhes desnecessários.

## Como explorar esta área

1. Comece entendendo a diferença entre dados expostos e comportamento responsável.
2. Use encapsulamento para proteger regras, não apenas para transformar campos públicos em propriedades privadas.
3. Escolha entre entidade, objeto de valor, serviço e `record` conforme a identidade e o comportamento do conceito.
4. Avalie composição, herança e classes fechadas diante do problema real, considerando legibilidade, teste e evolução.

## Conteúdo disponível

- [Do dado exposto ao objeto responsável](do-dado-exposto-ao-objeto-responsavel.md): classes anêmicas, encapsulamento, comportamentos, `record`, identidade e colaboração.

## Mapa de estudos

Os próximos artigos podem aprofundar:

- **Abstrações e polimorfismo:** contratos úteis, substituição e os limites da herança.
- **Composição versus herança:** como montar comportamentos sem criar hierarquias frágeis.
- **Objetos de valor e invariantes:** conceitos como dinheiro, período, endereço e identificadores.
- **Colaboração entre objetos:** mensagens, dependências, serviços de domínio e testes.
- **Modelagem orientada ao domínio:** entidades, agregados e fronteiras de consistência.

Os exemplos usam .NET/C# como linguagem de demonstração, conforme a explicação centralizada no [README do projeto](../../README.md). Os conceitos são agnósticos de linguagem.
