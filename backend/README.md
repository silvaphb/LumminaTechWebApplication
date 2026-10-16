# 💫 Lummina Tech -> Equipe de Desenvolvedores
A **Luminna Tech** é uma plataforma SaaS modular desenvolvida para fornecer soluções de gestão escaláveis para múltiplos nichos (comércio, serviços, entre outros), com foco inicial no controle de inventário para pequenos comércios. O sistema opera sob um modelo de assinatura recorrente (multi-tenant), permitindo:

- Gestão automatizada de acesso baseada no status da assinatura do cliente.

- Isolamento estrito de dados entre diferentes empresas (Multi-tenancy).

- Controle modular de recursos (planos e adicionais liberados via webhooks de pagamento).

- Módulo inicial de estoque: controle de produtos, categorias e movimentações de entrada/saída.

### Objetivo e Critério de Sucesso
- Validação do modelo de negócio SaaS com receita recorrente automatizada (via Asaas/Mercado Pago).

- Arquitetura de software escalável utilizando Clean Architecture e DDD (Domain-Driven Design).

- Liberação ou bloqueio instantâneo de recursos e módulos com base no pagamento do cliente, sem intervenção humana.

- Facilidade para plugar novos módulos ou nichos no futuro (expansão do SaaS) sem alterar o núcleo da aplicação.

## 🧮 Escopo do projeto
### MVP (Fase 1)
- **Módulo de Autenticação e Empresa:** Cadastro de empresas (tenants), gestão de usuários administradores e login seguro via JWT.

- **Módulo de Assinaturas & Webhooks:** Integração com gateway de pagamento para geração de cobranças mensais e atualização automática do status de acesso.

- **Módulo de Estoque:**

  - Cadastro de categorias e produtos (com código de barras/SKU, preço de custo, preço de venda e estoque mínimo).

  - Movimentações de estoque (Entrada e Saída com histórico auditável e trava contra estoque negativo).

  - Dashboard inicial com indicadores financeiros e alertas visuais de estoque baixo.

### Fase 2 — Expansão Modular e Novos Nichos
- Criação de novos módulos independentes (ex: PDV/Caixa, Financeiro, Agendamentos para serviços).

- Sistema de pacotes adicionais (Add-ons) adquiridos de forma avulsa pelo painel do cliente.

### Fora de escopo (atual)
- Aplicativo nativo mobile (o sistema será 100% responsivo via web).

- Emissão de Nota Fiscal Eletrônica (NFe) — reservada para versões futuras.

## 👥 Perfis e Permissões
O sistema opera com isolamento multi-tenant e controle hierárquico de acesso.

### Dono da Empresa (Admin)

- Acesso total às configurações da empresa, planos e dados financeiros.

- Cadastra funcionários/usuários e define permissões.

- Gerencia o estoque, categorias e movimentações.

### Funcionário / Operador

- Acesso restrito às operações do dia a dia (lançamento de entradas e saídas no estoque).

- Sem permissão para alterar dados de assinatura ou gerenciar utilizadores.

## 📎 Arquitetura da Solução
### Pilares Arquiteturais
- **Clean Architecture + DDD:** Separação rígida entre o Domínio (regras de negócio puras), Aplicação (casos de uso), Infraestrutura (bancos de dados e gateways) e Apresentação (APIs/Controllers).

- **Multi-tenancy por Banco de Dados (Shared Database, Isolated Data):** Todas as tabelas de negócio possuem a coluna empresa_id, garantindo que os dados de um cliente nunca sejam misturados com os de outro.

## 🧰 Stack Tecnológica
| Camada | Tecnologia | Motivo
|---|---|---|
|Backend|Python 3.12+ (Django / Django Ninja)|Alta performance, tipagem robusta e código limpo|
|Banco de dados|PostgreSQL|Confiabilidade, suporte a relacionamentos complexos e facilidade de isolamento por empresa_id|
|Gateway de Pagamento|Asaas ou Mercado Pago|Automação de cobranças recorrentes via PIX/Cartão e envio de Webhooks|
|Infra|Docker / Docker Compose|Conteinerização para deploy simplificado em PaaS (Render / Railway)|

## 📦 Entidades e Módulos
### Módulo `autenticacao` / `empresas` (IAM & Tenants)
Company (Empresa / Tenant)
Representa a lojinha ou negócio contratante do SaaS.

|Campo | Descrição |
|---|---|
|`id`| Identificador único da empresa|
|`name`| Nome fantasia da empresa|
|`document`| CNPJ ou CPF do proprietário|
|`plan_id`| Plano atual contratado|
|`subscription_status`| Status do pagamento (active, pending, canceled)|
|`expires_at`| Data de vencimento da assinatura atual|
|`created_at`| Data de cadastro|

### User (Usuário)
Usuários que possuem acesso ao painel da empresa.

| Campo | Descrição |
|---|---|
|`id` | Identificador único do usuário |
| `company_id` | Empresa à qual o usuário pertence |
| `name` | Nome do operador/dono |
| `email` | E-mail de login (único no sistema) |
| `password_hash` |	Senha encriptada (Bcrypt) |
| `role` | Perfil de acesso (admin, operator) |
| `active` | Status do usuário (exclusão lógica) |
| `created_at` | Data de criação |
| `Módulo assinaturas` | (Billing & Webhooks) |

### Plan (Plano)
Define os pacotes comerciais oferecidos pela Luminna Tech.

|Campo | Descrição |
|---|---|
| `id` | Identificador do plano (ex: start, pro) |
| `name` | Nome comercial do plano |
| `price` | Valor mensal da assinatura |
| `features` | Lista de módulos/permissões inclusas |

### WebhookLog (Auditoria de Pagamentos)
Guarda os eventos recebidos do gateway de pagamento. 

| Campo | Descrição |
|---|---|
| `id` | Identificador único do log |
| `gateway` | Origem (asaas, mercadopago) |
| `payload` | Dados brutos recebidos na requisição |
| `processed` | Booleano indicando sucesso no processamento |
| `created_at` | Data e hora do recebimento |

## Módulo stock (Inventário) - PRIMEIRO MÓDULO FORNECIDO PELA LUMINNA
### Category
Categorias para organização de produtos.

|Campo | Descrição |
|---|---|
| `id` | Identificador único da categoria |
| `company_id` | Empresa proprietária |
| `name` | Nome da categoria (ex: "Bebidas") |

### Product
O item físico na prateleira do cliente.

|Campo | Descrição |
|---|---|
| `id` | Identificador único do produto |
| `company_id` | Empresa proprietária |
| `category_id` | Categoria vinculada |
| `name` | Nome descritivo do produto |
| `barcode` | Código de barras ou SKU (único por empresa) |
| `cost_price` | Preço de custo pago pelo lojista |
| `sale_price` | Preço de venda ao consumidor |
| `stock_quantity` | Quantidade atual disponível |
| `min_stock` | Limite mínimo para alerta de estoque baixo |

### StockMovement
Histórico de auditoria de entradas e saídas.

| Campo | Descrição |
|---|---|
| `id` | Identificador único da movimentação |
| `company_id` | Empresa proprietária |
| `product_id` | Produto afetado |
| `type` | Tipo de operação (ENTRY ou EXIT) |
| `quantity` | Quantidade movimentada |
| `reason` | Motivo ou observação |
| `created_at` | Data e hora exata da ação |

## 📋 Casos de Uso Principais (Resumo)
- **Registo de Conta:** O lojista preenche o cadastro no site da Luminna Tech, escolhe o plano de estoque e é direcionado para gerar o pagamento inicial.

- **Confirmação via Webhook:** O gateway confirma o pagamento, dispara o webhook e o backend atualiza o subscription_status para active, liberando o acesso ao painel.

- **Validação de Acesso (Middleware):** A cada clique ou requisição nas rotas de estoque, o sistema valida se a empresa está com a assinatura em dia. Caso esteja vencida, o acesso é bloqueado com redirecionamento para regularização.

- **Gestão de Inventário (Módulo Stock):** O lojista cadastra categorias, produtos e registra entradas e saídas de mercadorias com validação automática de estoque mínimo.

