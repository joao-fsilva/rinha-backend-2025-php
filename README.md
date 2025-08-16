# Rinha de Backend 2025 - PHP & Swoole

## Sobre o Desafio

Desenvolver uma API que intermedia pagamentos entre dois proessadores. Ganha quem lucrar mais, processar mais rápido e ter consistência. Link para o desafio: [Rinha de Backend 2025](https://github.com/zanfranceschi/rinha-de-backend-2025)

### 🔄 Fluxo de Processamento

1.  **Recebimento**: Um Load Balancer (Nginx) recebe as requisições HTTP e as distribui em modo Round Robin entre duas instâncias da aplicação PHP.
2.  **Enfileiramento Rápido**: A API responde ao cliente o mais rápido possível e enfileira o pagamento em uma fila no Redis para processamento assíncrono.
3.  **Processamento Assíncrono**: Workers (em PHP/Swoole) consomem a fila de pagamentos de forma contínua.
4.  **Lógica de Fallback Inteligente**: O worker verifica a latência dos processadores de pagamento e utiliza uma lógica de custo-benefício.
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

## Recursos (Limite: 1.5 CPU + 350MB RAM)

| Serviço | CPU   | Memória   | Função                               |
| :------ | :---- |:----------| :----------------------------------- |
| `nginx` | 0.1   | 30MB      | Load Balancer                        |
| `app1`  | 0.6   | 120MB     | API Server + Worker (PHP/Swoole)     |
| `app2`  | 0.6   | 130MB     | API Server + Worker (PHP/Swoole)     |
| `cache` | 0.2   | 70MB      | Redis (Fila e Armazenamento)         |
| **Total** | **1.5** | **350MB** | ✅                                   |

## 🚀 Como Executar a Aplicação

### Pré-requisitos

-   Docker
-   Docker Compose

### Execução

```bash
# Clone o repositório (caso ainda não tenha feito)
# git clone https://github.com/zanfranceschi/rinha-de-backend-2025
# cd rinha-backend-2025-php

# Construa e inicie os contêineres em modo de produção
docker compose up --build -d
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