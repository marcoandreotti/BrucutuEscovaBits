# 1. API .NET integrada com LLM

> **Resumo:** a primeira conquista é pequena e importante: receber uma pergunta, consultar um modelo e devolver uma resposta sem espalhar detalhes do fornecedor pelo sistema.

## Pedra fundamental

Um **LLM**, de *Large Language Model* (modelo de linguagem de grande porte), gera conteúdo a partir de padrões aprendidos e do contexto recebido. Uma resposta convincente pode conter erros: fluência não comprova conhecimento nem acesso aos dados atuais da empresa.

<small>O contador de histórias da tribo fala bonito. Ainda precisamos conferir se o mamute realmente passou por ali.</small>

O **provedor** disponibiliza o serviço ou a execução do modelo. O **modelo** é a opção concreta usada para gerar a resposta. Trocar um não é necessariamente trocar o outro.

Um **token** é uma unidade de processamento do modelo, que pode representar parte de uma palavra, pontuação ou outros elementos. A **janela de contexto** limita a quantidade de informação considerada em uma interação; suas regras variam por modelo. Histórico e instruções também ocupam espaço.

## O problema e o encaixe arquitetural

Começamos com: “Explique para um novo integrante como registrar uma ferramenta emprestada”. Nesta etapa, o assistente produz uma explicação genérica. Ele ainda não conhece o procedimento oficial da tribo.

O adaptador HTTP valida a entrada e aciona um caso de uso em `Application`. Esse caso de uso utiliza uma porta como `IGeradorDeResposta`. Um adaptador em `Infrastructure` traduz o contrato para a API do provedor e normaliza resposta, uso reportado e falhas.

**SDK**, de *Software Development Kit* (kit de desenvolvimento), é a biblioteca que facilita essa integração. Seus tipos não precisam aparecer nos contratos HTTP ou no domínio. As abstrações `IChatClient` do ecossistema .NET são uma alternativa a avaliar no adaptador, conforme a necessidade real. [Referência: Microsoft.Extensions.AI](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai).

## Experimento proposto

**Pré-requisito:** [pressupostos do laboratório](README.md#pré-requisitos-e-pressupostos). Um adaptador simulado permite começar sem credenciais; a chamada real será uma validação adicional.

1. Definir um contrato de entrada com pergunta e um contrato de saída com texto, modelo utilizado e identificador da execução.
2. Rejeitar entrada vazia ou acima do limite antes da chamada externa. Manter a chave de acesso na configuração protegida do servidor.
3. Executar o caso de uso com um adaptador simulado. Depois, conectar um provedor e repetir a mesma pergunta.
4. Simular indisponibilidade e demora excessiva. Aplicar **timeout (tempo máximo de espera)** e propagar o cancelamento, devolvendo erro controlado e sem detalhes internos.

Um cancelamento local não garante que o provedor interrompeu o processamento ou deixou de cobrar. Essa diferença deve aparecer na ficha do experimento.

## Evidências e critérios de sucesso

- Entrada inválida não chega ao provedor.
- Trocar o adaptador não altera o caso de uso nem o contrato da API.
- Falhas são distinguíveis de respostas úteis, com correlação e sem credenciais nos registros.
- Duração, modelo e consumo disponível são registrados sem guardar o texto completo por padrão.

## Vantagens, limitações e erros comuns

A porta reduz acoplamento e facilita simulações. Ela adiciona um contrato para manter: não deve virar uma cópia de todas as opções de todos os fornecedores.

Evite tratar texto gerado como regra de negócio, reenviar um histórico ilimitado ou converter qualquer erro em uma resposta aparentemente bem-sucedida. Se a tarefa exige somente uma mensagem fixa, um texto revisado pode resolver melhor e custar menos.

**Desafio da caverna:** cancele uma solicitação e descreva o que a aplicação consegue garantir e o que depende do provedor.

[Visão geral](README.md) · [Próxima: AI Gateway](02-ai-gateway.md)
