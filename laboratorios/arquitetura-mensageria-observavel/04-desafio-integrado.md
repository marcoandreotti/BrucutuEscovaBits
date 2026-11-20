# 4. Desafio integrado

## Missão

Faça o fluxo `publisher -> RabbitMQ -> consumer` sobreviver a falhas observáveis. Não tente construir uma plataforma inteira. O objetivo é testar hipóteses e registrar decisões.

## Etapa 1, caminho feliz

- Publique dez eventos `pedido.criado`.
- Processe todos com confirmação manual.
- Registre `message.id`, `message.type` e `trace.id`.
- Meça a duração do processamento.

**Pergunta:** como você prova que as dez mensagens foram processadas, e não apenas aceitas pelo broker?

## Etapa 2, falha transitória

Faça o destino falhar por alguns segundos.

- Escolha um limite de tentativas.
- Aplique uma política de retry com atraso.
- Evite reprocessamento infinito.
- Registre a tentativa atual e a causa da falha.

**Pergunta:** como diferenciar uma indisponibilidade temporária de uma mensagem inválida?

## Etapa 3, duplicidade

Processe duas vezes o mesmo `message.id`.

- Identifique o efeito que seria perigoso repetir.
- Defina uma chave de idempotência.
- Registre o resultado como processado, duplicado ou rejeitado.

**Pergunta:** onde fica a garantia de idempotência, no consumer, no banco ou nos dois?

## Etapa 4, investigação

Escolha uma mensagem que falhou e encontre seu caminho usando apenas os sinais produzidos.

Critério de sucesso: localizar o evento sem abrir o código e sem imprimir o payload completo nos logs.

## Checklist de encerramento

- [ ] O publisher não abre uma conexão nova para cada mensagem.
- [ ] O consumer usa confirmação manual.
- [ ] Uma falha não cria um loop infinito de reentrega.
- [ ] Há uma estratégia para mensagens não processáveis.
- [ ] O contrato da mensagem possui tipo e identificador.
- [ ] Logs são estruturados e não expõem segredos.
- [ ] Há pelo menos uma métrica de volume e uma de duração.
- [ ] O contexto de trace atravessa a fronteira publisher-consumer, ou a limitação está documentada.
- [ ] O laboratório registra trade-offs e não apenas comandos executados.

## Facilitadores para a próxima iteração

Depois que o fluxo estiver compreendido, crie um pequeno template ou método genérico para reduzir repetição. Ele pode padronizar:

- criação de envelopes de mensagem;
- publicação com headers de correlação;
- abertura de spans;
- logs de início, sucesso e falha;
- `ack`, `nack` e encaminhamento para dead-letter.

O template deve permitir configurar política de retry, nome da fila, tipo de mensagem e limites. Não esconda esses parâmetros atrás de defaults mágicos.

## Reflexão final

Mensageria resolve desacoplamento e processamento assíncrono. Telemetria registra sinais. Observabilidade organiza a investigação. Serilog melhora a qualidade dos logs. Nenhuma dessas ferramentas corrige, sozinha, um contrato ruim, uma operação não idempotente ou uma decisão arquitetural sem dono.
