# Fluxo rastreável de desenvolvimento com IA

> **Resumo:** um fluxo seguro conecta demanda, análise, especificação, implementação, testes, documentação, commits, branches e Pull Requests. Cada etapa deve deixar evidência suficiente para outra pessoa entender o que aconteceu.

## Fluxo recomendado

```text
Jira/demanda
  → levantamento e perguntas
  → Open Spec / decisão
  → branch feature/*
  → implementação assistida
  → testes e análise do diff
  → documentação atualizada
  → commit semântico
  → Pull Request para develop
  → validação em Desenvolvimento
  → Homologação
  → release para main / Release
```

Os nomes dos ambientes podem variar; o importante é que promoção e responsabilidade estejam explícitas.

## Portões de segurança

- **Antes do código:** requisitos, escopo, pressupostos e critérios.
- **Antes do commit:** diff pequeno, testes e revisão da solução.
- **Antes do Pull Request:** documentação, logs de validação e riscos conhecidos.
- **Antes da homologação:** evidência de comportamento em ambiente controlado.
- **Antes da release:** aprovação, plano de retorno e rastreio da versão.

<small>Commit é a marca na parede; Pull Request é a conversa antes de abrir a passagem; release é deixar a tribo atravessar.</small>

## Exemplo de registro de trabalho

```markdown
## Alteração

Adicionar consulta de produtos por categoria.

## Decisão

Reutilizar o repositório existente e manter a API síncrona.

## Evidências

- testes unitários executados;
- diff revisado;
- documentação atualizada;
- risco: consulta exige índice em produção.

## Próximo passo

Abrir Pull Request para `develop`.
```

## Documentação gerada

A IA pode rascunhar changelogs, decisões, contratos e guias. A pessoa responsável deve conferir o texto contra o código, testes e configuração publicados. Se o código for a verdade, a documentação deve acompanhar sua evolução — e não competir com ela.

## Desafio da caverna

Pegue uma demanda fictícia, percorra todas as etapas e produza um registro com decisão, commit, testes, Pull Request e plano de promoção.
