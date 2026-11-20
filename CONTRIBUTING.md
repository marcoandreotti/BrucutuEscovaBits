# Contribuindo com o Brucutu Lava Bits

Obrigado por ajudar a transformar pedras em boas ideias. O projeto aceita correções, explicações, exemplos, laboratórios e melhorias de navegação.

## Antes de começar

1. Leia o [README](README.md) e o [template de artigo](docs/_templates/artigo.md).
2. Escolha uma mudança pequena e verificável.
3. Verifique se já existe conteúdo semelhante.
4. Não adicione dependências ou frameworks sem necessidade.

## Novos artigos

- Use `docs/<numero>-<area>/nome-do-artigo.md`.
- Prefira nomes em minúsculas, com hífens e sem acentos no nome do arquivo.
- Siga a estrutura de [docs/_templates/artigo.md](docs/_templates/artigo.md).
- Explique o conceito antes do código e diferencie simplificação didática de recomendação de produção.
- Inclua pré-requisitos, objetivos, erros comuns, trade-offs, desafio e referências.
- Use Mermaid para diagramas simples; mantenha os diagramas legíveis no GitHub.

## Imagens e diagramas

- Imagens devem ficar em `assets/mascote/`, `assets/banners/` ou `assets/diagramas/`, conforme seu papel.
- Use nomes descritivos e formatos adequados para publicação web.
- Registre texto alternativo significativo e a origem/licença quando houver.
- Não referencie uma imagem que ainda não existe; durante a preparação, documente o caminho esperado em um README da pasta.

## Qualidade editorial e técnica

- Revise ortografia, acentuação e consistência dos termos.
- Evite o travessão longo `—` na redação; prefira pontuação simples e frases claras.
- Ao introduzir um termo técnico em inglês, mantenha o original quando ele for útil para pesquisa, mas acrescente uma tradução ou explicação breve na primeira ocorrência.
- Use microtextos em `<small>` para analogias bem-humoradas quando ajudarem a fixar o conceito; mantenha a definição técnica fora deles e preserve a acessibilidade.
- Mostre vantagens, limitações e cenários de não utilização.
- Para exemplos de aplicação, prefira .NET/C# e informe versão, pressupostos e dependências quando forem relevantes.
- Não repita nos artigos a nota global sobre a possibilidade de aplicar os conceitos em outras linguagens; mantenha essa orientação no README.
- Evite exemplos inseguros, credenciais reais e comandos destrutivos.
- Prefira links relativos para conteúdo do próprio projeto.
- Confira se todos os links locais apontam para arquivos ou diretórios existentes.
- Se uma afirmação depender de uma fonte, cite uma referência confiável.

## Revisão

Antes de propor uma alteração, confira a renderização do Markdown no GitHub ou em um visualizador compatível, valide a sintaxe Mermaid e descreva no texto da contribuição o que mudou e como foi verificado.
