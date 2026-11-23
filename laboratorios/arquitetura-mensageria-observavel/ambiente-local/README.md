# Ambiente local de mensageria

Este ambiente fornece duas alternativas para explorar mensageria sem depender de uma conta AWS ou de um broker compartilhado:

| Serviço | Modelo | Porta local | Quando observar |
| --- | --- | --- | --- |
| RabbitMQ Management | AMQP, exchanges, bindings e filas | `5672` e `15672` | Topologia, roteamento e consumo pelo broker |
| MiniStack | API AWS, filas SQS e DLQ | `4566` | Código que será integrado ao Amazon SQS |

RabbitMQ e SQS não são a mesma tecnologia. O MiniStack também não transforma RabbitMQ em SQS: ele oferece uma API local para praticar operações compatíveis com serviços AWS.

## Pré-requisitos

- Docker Desktop com Docker Compose v2;
- AWS CLI somente para exercitar a API local do MiniStack;
- PowerShell ou outro terminal compatível.

## Subir o RabbitMQ

A partir da raiz do repositório:

```powershell
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/rabbitmq/docker-compose.yml up -d
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/rabbitmq/docker-compose.yml ps
```

- Painel: <http://localhost:15672>
- Usuário local: `app`
- Senha local: `app-secret`
- AMQP: `amqp://app:app-secret@localhost:5672/`

## Subir o MiniStack

```powershell
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml up -d
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml ps
Invoke-RestMethod http://localhost:4566/_ministack/health
```

Configure credenciais fictícias para a AWS CLI local:

```powershell
$env:AWS_ACCESS_KEY_ID = "test"
$env:AWS_SECRET_ACCESS_KEY = "test"
$env:AWS_DEFAULT_REGION = "us-east-1"
$endpoint = "http://localhost:4566"

aws --endpoint-url=$endpoint sts get-caller-identity
```

Essas credenciais existem apenas para satisfazer a cadeia de autenticação local. Não use credenciais reais neste laboratório.

## Parar e limpar

Para parar os serviços e preservar volumes:

```powershell
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/rabbitmq/docker-compose.yml down
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml down
```

Para remover também filas, mensagens, configurações e estado persistido:

```powershell
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/rabbitmq/docker-compose.yml down -v
docker compose -f laboratorios/arquitetura-mensageria-observavel/ambiente-local/ministack/docker-compose.yml down -v
```

Use `down -v` apenas quando tiver confirmado que os volumes pertencem ao laboratório.
