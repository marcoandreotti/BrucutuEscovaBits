# Git Flow e commits semânticos

> **Resumo:** um fluxo simples de branches e mensagens consistentes torna mudanças rastreáveis. Git Flow é uma convenção, não uma cerimônia obrigatória.

## Fluxo adotado

- `main`: conteúdo estável e protegido.
- `develop`: integração do próximo conjunto de mudanças.
- `feature/<nome>`: uma mudança pequena e verificável.

```text
feature/artigo-testes ── Pull Request ──> develop ── Pull Request ──> main
```

## Commit semântico

**Commits semânticos** usam um prefixo que comunica intenção:

```text
docs: adicionar artigo sobre testes em .NET
feat: criar laboratório de idempotência
fix: corrigir link da trilha intermediária
chore: ajustar validação de Markdown
```

Prefira verbo no infinitivo, escopo pequeno e uma mudança coerente.

<small>Um commit bom é uma tabuleta que outra pessoa consegue ler sem chamar o arqueólogo.</small>

## Desafio da caverna

Crie uma branch para um artigo, faça dois commits semânticos pequenos e abra um Pull Request para `develop`.
