O PROJETO AINDA ESTA EM DESENVOLVIMENTO - é um Saas - ABAIXO É O QUE JÁ ESTA PRONTO - Subi apenas algumas fotos porque este projeto já tem um destino e propósito. Caso interesse podemos agendar apresentação. (miltonconsultordeti@gmail.com)

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


DOCUMENTAÇÂO DE PIX

# 📘 Documentação – PIX + DICT (Simulador BK SIM)

## 1️⃣ Visão Geral

Este projeto implementa um **simulador de PIX** inspirado no funcionamento real do ecossistema brasileiro, incluindo:

* DICT (Diretório de Identificadores de Contas Transacionais)
* PIX por QR Code
* PIX por Chave
* Fluxos **on-us** (mesmo PSP)
* Processamento assíncrono via fila
* Idempotência, auditoria e rastreabilidade


---

## 2️⃣ Conceitos-Chave

### PSP (Payment Service Provider)

Instituição participante do PIX.
No simulador:

* Existe um PSP “nosso” (`eh_nosso = true`)
* Outros PSPs podem existir no DICT (off-us)

### DICT

Base central de chaves PIX → resolve:

* Para qual PSP a chave pertence
* Qual conta está associada
* Status atual da chave (ATIVA / INATIVA)

---

## 3️⃣ Modelagem de Dados (alto nível)

### 🔹 pix_keys

Tabela **interna do banco**
Representa as chaves que o cliente criou na conta.

Campos principais:

* `id`
* `conta_id`
* `tipo_chave` (EVP, EMAIL, PHONE, CPF, CNPJ)
* `chave`
* `ativa`
* `criada_em`

---

### 🔹 dict_psps

Cadastro de PSPs conhecidos pelo DICT.

Campos:

* `id`
* `nome`
* `ispb`
* `eh_nosso`
* `ativo`

---

### 🔹 dict_entries

Estado **atual** da chave no DICT.

Campos principais:

* `chave`
* `tipo_chave`
* `status` (ATIVA / INATIVA)
* `psp_id`
* `conta_id` (somente se on-us)
* dados da conta e do titular

🔴 Sempre **1 registro por chave** (estado atual).

---

### 🔹 dict_entry_histories

Histórico imutável de eventos da chave.

Eventos típicos:

* `CHAVE_CRIADA`
* `CHAVE_RECRIADA`
* `CHAVE_INATIVADA`
* `CHAVE_RESOLVIDA`

Usado para:

* Auditoria
* Futuro motor de risco
* Investigações

---

### 🔹 pix_qr_codes

QR Codes gerados pelo banco.

Campos:

* `transaction_id`
* `valor`
* `pago`
* `conta_id`

---

### 🔹 pix_recebidos

Registro definitivo de PIX **entrada confirmada**.

Campos:

* `tx_id`
* `e2e_id`
* `valor`
* `conta_id`
* `qr_code_id` (nullable → entrada por chave)
* `status`

---

### 🔹 pix_saidas

Registro de PIX **saída solicitada**.

Campos:

* `transaction_id`
* `idempotency_key`
* `status`
* dados do favorecido
* dados resolvidos do DICT (destino)

---

### 🔹 movimentacoes

Extrato financeiro da conta.

Tipos:

* `Pix Entrada`
* `Pix Saída`
* `Transferencia Entrada`
* `Transferencia Saída`
* etc.

---

### 🔹 pix_entrada_erros / pix_saida_erros

Falhas permanentes de processamento.

Usadas para:

* Diagnóstico
* Suporte
* Auditoria

---

## 4️⃣ Fluxos Implementados

## 🔄 Criação de Chave PIX

1. Cliente solicita criação
2. Validações:

   * Limite de 5 chaves ativas
   * Unicidade (PixKeys + DICT)
3. Cria em:

   * `pix_keys`
   * `dict_entries`
   * `dict_entry_histories`

✔️ Simula o **registro no DICT do Bacen**

---

## ❌ Exclusão de Chave PIX

1. Marca `pix_keys.ativa = false`
2. Atualiza `dict_entries.status = INATIVA`
3. Registra histórico `CHAVE_INATIVADA`

✔️ Histórico nunca é apagado

---

## 📥 PIX Entrada por QR Code

1. QR criado (`pix_qr_codes`)
2. Webhook simula Bacen
3. Evento enviado para fila
4. Handler:

   * Idempotência (TxId / E2E)
   * Atualiza saldo
   * Cria `pix_recebidos`
   * Marca QR como pago
   * Cria movimentação

---

## 📥 PIX Entrada por Chave (DICT)

1. API recebe requisição
2. Resolve chave no DICT

   * Deve existir
   * Deve estar ATIVA
   * Deve ser on-us
3. Publica evento na fila
4. Handler:

   * Não usa QR
   * Usa `valor + contaIdDestino`
   * Atualiza saldo
   * Cria `pix_recebidos`
   * Cria movimentação

✔️ Simula SPI / liquidação interna

---

## 📤 PIX Saída

1. Cliente envia solicitação
2. Idempotência por `idempotency_key`
3. Resolve chave no DICT
4. Registra `pix_saidas`
5. Publica evento de saída
6. (Fluxo futuro: off-us / liquidação externa)

---

## 5️⃣ Idempotência & Concorrência

* **Entrada**

  * `tx_id` → idempotência principal
  * `e2e_id` → proteção adicional
* **Saída**

  * `idempotency_key` obrigatório (GUID)

Tratamento:

* Duplicate → tratado como replay
* Não duplica saldo nem movimentação

---

## 6️⃣ Auditoria e Observabilidade

* `BankAuditLogger`
* `pix_entrada_erros`
* `pix_saida_erros`
* `dict_entry_histories`

🎯 Base pronta para **Risk Tool futuro**

---

## 7️⃣ O que NÃO foi implementado (por decisão)

* Quarentena de chaves
* Bloqueio automático por risco
* Liquidação real off-us
* SPI real / Bacen

➡️ Tudo isso foi **preparado estruturalmente**, mas propositalmente fora do escopo.

---

