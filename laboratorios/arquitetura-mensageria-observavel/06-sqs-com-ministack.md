# 6. SQS com MiniStack

## Objetivo

Explorar uma fila acessada por API HTTP e comparar esse modelo com o broker RabbitMQ. O MiniStack permite praticar o contrato localmente, sem enviar chamadas para a AWS.

No SQS, a confirmação acontece quando o consumidor exclui a mensagem. Enquanto ela está sendo processada, permanece invisível durante o **visibility timeout**. Se o processo falhar e não excluir a mensagem, ela poderá aparecer novamente.

<small>RabbitMQ entrega pela topologia da caverna; SQS entrega por uma API e espera que o consumidor diga quando terminou.</small>

## 1. Inicie e prepare a CLI

```powershell
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml up -d

$env:AWS_ACCESS_KEY_ID = "test"
$env:AWS_SECRET_ACCESS_KEY = "test"
$env:AWS_DEFAULT_REGION = "us-east-1"
$endpoint = "http://localhost:4566"

Invoke-RestMethod "$endpoint/_ministack/health"
aws --endpoint-url=$endpoint sts get-caller-identity
```

As credenciais são fictícias e existem apenas para a API local.

## 2. Crie e consulte uma fila

```powershell
aws --endpoint-url=$endpoint sqs create-queue `
  --queue-name pedidos `
  --attributes VisibilityTimeout=30,ReceiveMessageWaitTimeSeconds=10

$queueUrl = aws --endpoint-url=$endpoint sqs get-queue-url `
  --queue-name pedidos --query QueueUrl --output text

aws --endpoint-url=$endpoint sqs get-queue-attributes `
  --queue-url $queueUrl --attribute-names All
```

O long polling de 10 segundos reduz consultas vazias. O timeout de visibilidade precisa ser maior que o tempo normal de processamento, ou ser estendido durante tarefas longas.

## 3. Publique mensagens

```powershell
aws --endpoint-url=$endpoint sqs send-message `
  --queue-url $queueUrl `
  --message-body '{"pedidoId":"ped-001","evento":"criado"}'
```

Para operações em lote, use `send-message-batch` e teste o limite de dez entradas por chamada.

## 4. Consuma e confirme

```powershell
$response = aws --endpoint-url=$endpoint sqs receive-message `
  --queue-url $queueUrl `
  --max-number-of-messages 10 `
  --wait-time-seconds 10 `
  --visibility-timeout 30 | ConvertFrom-Json

foreach ($message in @($response.Messages)) {
  try {
    $body = $message.Body | ConvertFrom-Json
    Write-Output "Processando $($body.pedidoId)"

    # Exclua somente depois do efeito desejado.
    aws --endpoint-url=$endpoint sqs delete-message `
      --queue-url $queueUrl `
      --receipt-handle $message.ReceiptHandle
  }
  catch {
    Write-Warning "Falha ao processar a mensagem: $($_.Exception.Message)"
    # Sem delete, a mensagem voltará após o visibility timeout.
  }
}
```

Não confunda receber com confirmar. O recebimento apenas torna a mensagem invisível temporariamente; a exclusão confirma o processamento.

## 5. Crie uma DLQ

Uma **dead-letter queue**, ou DLQ, recebe mensagens que falharam repetidamente. O ambiente local permite praticar o redrive sem envolver a AWS real.

```powershell
aws --endpoint-url=$endpoint sqs create-queue --queue-name pedidos-dlq

$dlqUrl = aws --endpoint-url=$endpoint sqs get-queue-url `
  --queue-name pedidos-dlq --query QueueUrl --output text

$dlqArn = (aws --endpoint-url=$endpoint sqs get-queue-attributes `
  --queue-url $dlqUrl --attribute-names QueueArn | ConvertFrom-Json).Attributes.QueueArn

$redrivePolicy = @{
  RedrivePolicy = (@{
    deadLetterTargetArn = $dlqArn
    maxReceiveCount = "3"
  } | ConvertTo-Json -Compress)
} | ConvertTo-Json -Compress

aws --endpoint-url=$endpoint sqs set-queue-attributes `
  --queue-url $queueUrl --attributes $redrivePolicy
```

DLQ não é lixeira. Ela precisa de observação, diagnóstico e procedimento de reprocessamento.

## 6. Exercite FIFO

```powershell
aws --endpoint-url=$endpoint sqs create-queue `
  --queue-name pedidos.fifo `
  --attributes FifoQueue=true,ContentBasedDeduplication=true

$fifoUrl = aws --endpoint-url=$endpoint sqs get-queue-url `
  --queue-name pedidos.fifo --query QueueUrl --output text

aws --endpoint-url=$endpoint sqs send-message `
  --queue-url $fifoUrl `
  --message-body '{"pedidoId":"ped-010"}' `
  --message-group-id pedidos `
  --message-deduplication-id ped-010
```

O `MessageGroupId` participa da ordenação e o identificador de deduplicação ajuda a evitar duplicações dentro da janela prevista pelo serviço. Isso não elimina a necessidade de idempotência no consumer.

## 7. Compare os modelos

| Pergunta | RabbitMQ | SQS no MiniStack |
| --- | --- | --- |
| Como a mensagem é roteada? | Exchange, binding e routing key | Fila acessada por API |
| Como confirmar? | `ack` ou `nack` no consumer | `delete-message` |
| Como lidar com falha? | Requeue, retry e dead-letter | Visibilidade, novas entregas e DLQ |
| Onde testar a topologia? | Painel e API do broker | CLI e API da fila |
| Quando faz sentido? | Controle fino de broker e AMQP | Integração com contrato AWS e filas gerenciadas |

## 8. Limpeza

```powershell
aws --endpoint-url=$endpoint sqs purge-queue --queue-url $queueUrl
aws --endpoint-url=$endpoint sqs delete-queue --queue-url $queueUrl
aws --endpoint-url=$endpoint sqs delete-queue --queue-url $dlqUrl

docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml down -v
```

Use `down -v` somente para volumes do laboratório.

## Referência

- [MiniStack](https://ministack.org)
- [Guia de SQS da AWS](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/welcome.html)
