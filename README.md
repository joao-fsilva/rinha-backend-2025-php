# 🥊 Rinha de Backend 2025 - PHP & Swoole  

## 🎯 Objetivo  
Construir uma API que intermedia pagamentos entre dois processadores, priorizando:  
- 💰 **Lucro**  
- ⚡ **Velocidade**  
- 🔒 **Consistência**  

🔗 [Repositório Oficial do Desafio](https://github.com/zanfranceschi/rinha-de-backend-2025)  

---

## ⚙️ Fluxo  

- **Nginx** → Distribui requisições em *round robin*  
- **App (PHP + Swoole)** → Responde rápido e enfileira pagamentos  
- **Redis** → Gerencia fila e armazenamento  
- **Workers (PHP + Swoole)** → Processam de forma assíncrona  
- **Fallback Inteligente** → Escolha dinâmica do processador  
- **Consistência** → Relatório dos pagamentos processados para verificação de consistência.  

---

## 🛠️ Stack  

| Camada          | Tecnologia           |
|-----------------|----------------------|
| Linguagem       | PHP 8.4              |
| Runtime         | Swoole (assíncrono)  |
| Banco/Fila      | Redis                |
| Load Balancer   | Nginx                |
| Infra           | Docker + Compose     |

---

## 📊 Limites de Recursos  

| Serviço  | CPU  | Memória | Função                          |
|----------|------|---------|---------------------------------|
| nginx    | 0.1  | 30MB    | Balanceador de carga            |
| app1     | 0.6  | 120MB   | API + Worker (PHP/Swoole)       |
| app2     | 0.6  | 120MB   | API + Worker (PHP/Swoole)       |
| cache    | 0.2  | 80MB    | Redis (Fila e Armazenamento)    |
| **Total**| 1.5  | 350MB   | ✅ dentro do limite              |  

---
