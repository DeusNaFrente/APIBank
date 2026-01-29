O PROJETO AINDA ESTA EM DESENVOLVIMENTO - ABAIXO É O QUE JÁ ESTA PRONTO - Subi apenas algumas fotos porque este projeto já tem um destino e propósito. Caso interesse podemos agendar apresentação. (miltonconsultordeti@gmail.com)

🧠 Linguagem & Plataforma

C#

.NET 8 (ASP.NET Core Web API)
→ Controllers, BackgroundService, DI, middleware, logging

🌐 API / Backend

ASP.NET Core Web API

Controllers (PixController, PixWebhookController, ContasController, MovimentacaoController)

Rotas REST

DTOs para entrada de dados

Validação de ModelState

Responses padronizados (returnCode, status, etc.)

🗄️ Persistência / Banco de Dados

Entity Framework Core

Pomelo.EntityFrameworkCore.MySql

MySQL

Recursos usados:

DbContext

DbSet<>

Migrations (implícito)

Transactions (BeginTransactionAsync)

AsNoTracking

Índices únicos (ex: idempotency_key)

Relacionamentos (ForeignKey)

Controle de saldo com consistência transacional

🔁 Mensageria / Assíncrono

RabbitMQ

RabbitMQ.Client (Async)

Conceitos aplicados:

Filas dedicadas (pix_entrada_queue, pix_saida_queue)

Publisher

Consumer

Workers (BackgroundService)

Processamento assíncrono

Idempotência no consumidor

Retry controlado (não requeue em erro lógico)

Desacoplamento API ⇄ processamento financeiro

🧩 Arquitetura / Design

Arquitetura em camadas

Controllers

Application (Handlers)

Messaging

Models

DTOs

Event-driven

CQRS leve

API grava estado → publica evento

Worker processa e confirma

Idempotência

idempotencyKey obrigatório

Índice único no banco

Transações financeiras seguras

Débito + crédito + movimentação no mesmo transaction

💸 Domínio Bancário / PIX

Simulação de banco digital (estilo Iugu / PSP)

PIX Entrada

QR Code estático

Webhook de pagamento

Detecção de duplicidade (EndToEndId)

Registro de erro (pix_entrada_erros)

PIX Saída

Processamento assíncrono

Status (RECEIVED, CONFIRMADO, ERRO)

Idempotência

PIX Interno

Identificação de chave interna

Crédito automático na conta destino

Duas movimentações (entrada + saída)

Extrato bancário consistente

Cálculo correto de saldo histórico

Crédito / Débito

Ordenação correta

Vinculação por TransferenciaId

📦 Outras Bibliotecas / Utilidades

System.Text.Json – serialização de eventos

QRCoder – geração de QR Code Pix

LINQ

Async / Await

Dependency Injection

Insomnia – testes de API

Swagger (via Swashbuckle)

🔐 Boas Práticas que você já aplicou

Idempotência obrigatória

Transações ACID para saldo

Não confiar no payload externo para valor

Processamento assíncrono de dinheiro

Evitar duplicidade de PIX

Separação clara entre:

“Receber requisição”

“Processar dinheiro”

Mensagens de erro coerentes

Logs claros de domínio financeiro
