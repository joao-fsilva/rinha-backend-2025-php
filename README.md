# 🏆 Rinha de Backend 2025 - PHP & Swoole

## 🎯 Sobre o Desafio

Este projeto participa da Rinha de Backend 2025, um desafio que testa a capacidade de construir sistemas altamente escaláveis e resilientes. O objetivo é desenvolver um backend que intermedie solicitações de pagamentos para serviços de processamento, maximizando o lucro através da escolha inteligente entre dois processadores de pagamento com taxas e estabilidades diferentes.

## 🏗️ Arquitetura

O sistema segue uma arquitetura distribuída com foco em alta disponibilidade e processamento assíncrono.

### 🔄 Fluxo de Processamento

1.  **Recebimento**: Um Load Balancer (Nginx) recebe as requisições HTTP e as distribui em modo Round Robin entre duas instâncias da aplicação PHP.
2.  **Enfileiramento Rápido**: A API responde ao cliente o mais rápido possível (estratégia "Fire and Forget") e enfileira o pagamento em uma fila no Redis para processamento assíncrono.
3.  **Processamento Assíncrono**: Workers (em PHP/Swoole) consomem a fila de pagamentos de forma contínua.
4.  **Lógica de Fallback Inteligente**: O worker verifica a latência dos processadores de pagamento e utiliza uma lógica de custo-benefício (considerando que o `default` é 3x mais barato) para decidir qual processador usar, maximizando o lucro.
5.  **Auditoria**: O sistema persiste um registro de todas as transações processadas com sucesso, que podem ser consultadas através de um endpoint de resumo.

## 🛠️ Tecnologias

### Backend

-   **Linguagem**: PHP 8.4
-   **Runtime Assíncrono**: Swoole
-   **Banco de Dados / Fila**: Redis

### Infraestrutura

-   **Containerização**: Docker
-   **Orquestração**: Docker Compose
-   **Load Balancer**: Nginx

## 📊 Recursos (Limite: 1.5 CPU + 350MB RAM)

| Serviço | CPU   | Memória | Função                               |
| :------ | :---- | :------ | :----------------------------------- |
| `nginx` | 0.1   | 30MB    | Load Balancer                        |
| `app1`  | 0.6   | 110MB   | API Server + Worker (PHP/Swoole)     |
| `app2`  | 0.6   | 110MB   | API Server + Worker (PHP/Swoole)     |
| `cache` | 0.2   | 100MB   | Redis (Fila e Armazenamento)         |
| **Total** | **1.5** | **350MB** | ✅                                   |

## 🎯 Estratégias de Otimização

### Performance

-   **PHP JIT & OPcache**: O OPcache é usado para manter o código pré-compilado em memória e o JIT (Just-In-Time compiler) é ativado em modo `tracing` para otimizar a execução do código em tempo real.
-   **I/O Não-Bloqueante**: As corrotinas do Swoole são usadas para todas as operações de rede (Redis, HTTP), evitando o bloqueio do processo.
-   **"Fire and Forget" na API**: A API de pagamentos responde imediatamente ao cliente e enfileira a tarefa para ser processada em segundo plano, resultando em um tempo de resposta extremamente baixo.
-   **Connection Pooling**: Um pool de conexões com o Redis é mantido para evitar o custo de criar novas conexões a cada requisição.

### Resiliência

-   **Fallback Inteligente**: A lógica para escolher entre o processador `default` e `fallback` não se baseia apenas na latência, mas também no custo, tentando sempre usar o `default` se ele não for 3x mais lento que o `fallback`.
-   **Re-enfileiramento**: Se uma transação falhar em ambos os processadores, ela é devolvida para a fila para uma nova tentativa.

## 🚀 Como Executar a Aplicação

### Pré-requisitos

-   Docker
-   Docker Compose

### Execução

```bash
# Clone o repositório (caso ainda não tenha feito)
# git clone <seu-repo>
# cd rinha-de-backend-2025

# Construa e inicie os contêineres em modo de produção
docker-compose up --build -d
```

Após a execução, a API estará disponível em `http://localhost:9999`.

## 📋 Endpoints da API

### `POST /payments`

Enfileira um novo pagamento para ser processado.

**Corpo da Requisição:**
```json
{
    "correlationId": "4a7901b8-7d26-4d9d-aa19-4dc1c7cf60b3",
    "amount": 19.90
}
```

**Resposta de Sucesso:** `200 OK`

### `GET /payments-summary`

Retorna um resumo com o total de requisições e o valor total processado por cada provedor (`default` e `fallback`).

**Query Parameters (Opcionais):**

-   `from` (string, formato `Y-m-d\TH:i:s.u\Z`): Filtra transações a partir desta data.
-   `to` (string, formato `Y-m-d\TH:i:s.u\Z`): Filtra transações até esta data.

**Exemplo de Resposta:**
```json
{
    "default": {
        "totalRequests": 12324,
        "totalAmount": 245247.6
    },
    "fallback": {
        "totalRequests": 2905,
        "totalAmount": 57809.5
    }
}
```