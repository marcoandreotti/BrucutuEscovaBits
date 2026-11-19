# IA na engenharia de software

> **Resumo:** ferramentas de inteligência artificial podem acelerar levantamentos, análise, implementação, testes e documentação. O ganho sustentável vem de combiná-las com contexto, limites, revisão humana e evidências executáveis.

## O que a IA pode fazer

Na engenharia de software, a IA pode apoiar:

- levantamento e decomposição de requisitos;
- leitura de bases de código existentes;
- comparação de alternativas técnicas;
- criação de planos e tarefas menores;
- geração de código e testes;
- identificação de riscos, duplicações e código fossilizado;
- atualização de documentação;
- análise de falhas e preparação de Pull Requests.

Ela não elimina a necessidade de compreender o domínio, confirmar requisitos, revisar alterações ou operar o sistema.

<small>A IA pode ajudar o Brucutu a carregar mais pedras, mas ainda precisamos decidir onde construir a ponte.</small>

## Pedra fundamental: contexto antes de resposta

Uma solicitação vaga produz uma resposta genérica. Um bom contexto informa:

1. objetivo e problema de negócio;
2. restrições técnicas e de segurança;
3. arquivos e decisões relevantes;
4. critérios de aceitação;
5. forma de validação esperada.

```text
Objetivo: adicionar consulta de produtos por categoria.
Contexto: API .NET existente; não alterar o contrato atual.
Restrições: usar os padrões do módulo de catálogo; não adicionar dependências.
Critérios: paginação, validação de categoria, testes unitários e documentação.
Validação: executar testes e revisar o diff antes do commit.
```

## Código gerado precisa de prova

Um trecho sugerido pela IA deve passar pelo mesmo ciclo de qualquer alteração:

```csharp
public async Task<IReadOnlyList<Produto>> ListarPorCategoriaAsync(
    string categoria,
    CancellationToken cancellationToken)
{
    ArgumentException.ThrowIfNullOrWhiteSpace(categoria);

    return await repositorio.ListarPorCategoriaAsync(
        categoria.Trim(), cancellationToken);
}
```

O trecho parece simples, mas ainda exige perguntas: o contrato aceita espaços? A busca é case-insensitive? Existe índice? Há autorização? Como o cancelamento é tratado? O teste confirma a regra?

## Revisão humana e segurança

Antes de aceitar uma sugestão, verifique:

- comportamento e requisitos;
- segurança, autorização e exposição de dados;
- concorrência, transações e idempotência;
- desempenho e custo operacional;
- cobertura e qualidade dos testes;
- compatibilidade com decisões já registradas.

Não forneça segredos, tokens, dados pessoais ou código confidencial a ferramentas sem autorização e controles adequados. A resposta da IA é uma sugestão não confiável até ser examinada.

## Código como verdade e documentação

Documentação gerada pode acelerar o trabalho, mas deve refletir o sistema real. Quando houver conflito entre texto e comportamento executável, registre a divergência e corrija a fonte adequada. Documentação não deve inventar endpoints, regras ou garantias que o código não sustenta.

## Desafio da caverna

Escolha uma função pequena, peça à IA três alternativas e compare-as usando critérios explícitos. Registre a alternativa escolhida, a descartada e a evidência usada para decidir.

## Resumo

IA aumenta capacidade de análise e produção, mas qualidade depende de contexto, revisão e validação. O objetivo não é aceitar mais código: é tomar decisões melhores com rastreabilidade.
