# Agentes, skills, memória e orquestração

> **Resumo:** agentes especializados podem dividir um problema por responsabilidade, enquanto skills (habilidades instruídas), memória e especificações compartilhadas preservam contexto. A arquitetura precisa impedir que a divisão vire uma caverna sem mapa.

## Vocabulário

- **Agente:** executor orientado a um objetivo, com contexto, ferramentas e limites.
- **Skill:** instrução reutilizável para uma abordagem, domínio ou ferramenta.
- **Memória:** contexto persistente, decisões e fatos úteis para trabalhos futuros.
- **Orquestração:** coordenação de agentes, etapas, entradas, saídas e aprovações.
- **Open Spec:** especificação aberta que organiza intenção, decisões, critérios e mudanças antes da implementação.

## Agentes por camada

Uma divisão possível é:

```text
Demanda/Jira
    ↓
Agente de requisitos ──> especificação e critérios
    ↓
Agente de arquitetura ─> decisões e impactos
    ↓
Agente de domínio ─────> regras e casos de uso
    ↓
Agente de infraestrutura -> persistência, integrações e configuração
    ↓
Agente de testes ───────> cenários, execução e evidências
    ↓
Revisão humana ─────────> decisão, commit e Pull Request
```

Separar agentes não significa separar responsabilidade humana. Uma pessoa ou equipe deve continuar respondendo pelo resultado.

## Contrato de um agente

Cada agente deve declarar:

- objetivo;
- entradas confiáveis;
- arquivos que pode alterar;
- ações proibidas;
- formato da saída;
- validações obrigatórias;
- condição de parada e quando pedir revisão.

```yaml
nome: agente-de-testes
objetivo: propor e executar testes para uma mudança pequena
entradas:
  - especificação aprovada
  - diff da implementação
saidas:
  - cenários cobertos
  - testes criados
  - comandos executados e resultados
nao_fazer:
  - alterar contrato sem registrar decisão
  - ocultar falhas
  - fazer commit sem revisão humana
```

## Memória sem virar depósito de fósseis

Memória útil deve ter fonte, data ou contexto, escopo e forma de atualização. Separe:

- fatos estáveis do projeto;
- decisões arquiteturais;
- preferências editoriais;
- estado temporário de uma tarefa;
- hipóteses ainda não confirmadas.

Memória desatualizada é um risco: um agente pode obedecer a uma decisão que já foi substituída. Toda memória importante precisa poder ser revisada ou removida.

## Orquestração e limites

Open Spec, skills e agentes devem reduzir ambiguidade, não criar uma burocracia maior que o problema. Comece com uma sequência simples, registre entradas e saídas e adicione paralelismo apenas quando houver benefício verificável.

## Desafio da caverna

Modele uma mudança pequena com dois agentes: requisitos e testes. Defina contratos de entrada e saída e identifique em qual ponto uma pessoa precisa aprovar a próxima etapa.

## Resumo

Agentes especializados podem melhorar foco e repetibilidade. Skills e memória preservam contexto; orquestração controla o fluxo. Limites, evidências e revisão humana são a argamassa dessa construção.
