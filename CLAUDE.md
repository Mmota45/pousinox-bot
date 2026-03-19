# CLAUDE.md — Contexto para o Claude Code

Este arquivo instrui o Claude Code sobre o projeto. Leia antes de qualquer edição.

## O que é este projeto

Bot de atendimento WhatsApp 24/7 para qualificação de leads da **Pousinox** (fabricante de equipamentos em inox, Pouso Alegre/MG). O produto piloto é o **fixador de porcelanato em inox**.

## Stack

| Componente | Tecnologia |
|---|---|
| Automação | N8N (self-hosted, Docker) |
| WhatsApp | Z-API |
| Memória de sessão | Redis |
| CRM | Google Sheets |
| Frete | Melhor Envio API (opcional) |
| Pagamento | PagBank |
| Servidor | Hostinger VPS, Ubuntu 22.04 |

## Estrutura do repositório

```
pousinox-bot/
├── workflows/
│   └── pousinox_bot_v2.json   ← workflow principal do N8N
├── scripts/
│   ├── setup.sh               ← setup inicial do VPS
│   └── deploy.sh              ← atualização do servidor
├── docs/
│   └── configuracao.md        ← guia de configuração manual
├── docker-compose.yml          ← N8N + Redis
├── .env.example                ← variáveis necessárias (sem valores reais)
├── .gitignore
└── CLAUDE.md                   ← este arquivo
```

## Arquivo principal: workflows/pousinox_bot_v2.json

### Fluxo do workflow

```
Webhook WhatsApp (Z-API POST)
  → Normalizar Entrada        [Code] extrai phone, message, name
  → Filtro Ignorar            [Filter] descarta fromMe, grupos, sem texto
  → Buscar Estado Redis       [Redis GET] recupera sessão do lead
  → Motor Comercial V2        [Code] TODA a lógica de negócio
      ↓ (1 output, 5 destinos paralelos)
      ├─ Salvar Estado Redis  [Redis SET] persiste stage/mode/dados
      ├─ Salvar Lead Redis    [Redis SET] snapshot do lead
      ├─ Google Sheets Leads  [Sheets appendOrUpdate] CRM principal
      ├─ Google Sheets Eventos[Sheets append] log de interações
      └─ Enviar Mensagens Z-API [Code] dispara mensagens
```

### Stages da conversa (Motor Comercial)

| stage | Descrição |
|---|---|
| `menu` | Menu inicial |
| `ask_inox` | Escolha do tipo de inox (430 ou 304) |
| `ask_thickness` | Espessura do porcelanato (opção) |
| `ask_thickness_exact` | Espessura exata em mm |
| `ask_qty` | Quantidade de peças |
| `ask_cep` | CEP de entrega |
| `human` | Handoff para consultor humano |

### Comandos especiais do bot

| Mensagem | Ação |
|---|---|
| `0` ou `menu` | Volta ao menu |
| `#reset` | Reativa o bot (sai do modo humano) |
| `2`, `humano`, `atendente` | Solicita atendimento humano |

### Variáveis de ambiente usadas no workflow

```
ZAPI_INSTANCE_ID        → ID da instância Z-API
ZAPI_INSTANCE_TOKEN     → Token da instância Z-API
ZAPI_CLIENT_TOKEN       → Client token Z-API
PAGBANK_URL             → Link direto de pagamento PagBank
POUSINOX_ORIGIN_CEP     → CEP de origem para cotação de frete
MELHOR_ENVIO_TOKEN      → Token API Melhor Envio (opcional)
```

## Preços atuais (CONFIG no Motor Comercial)

```js
price430: 12.00   // Inox 430 — uso geral
price304: 15.00   // Inox 304 — ambientes litorâneos
bonusQtyMin: 50   // Acima de 50 peças → bônus de R$0,05/peça
freeShippingStates: ['MG', 'SP', 'RJ']  // Frete grátis nesses estados
freeShippingQtyAbove: 100  // ... para pedidos acima de 100 unidades
specialConditionQtyAbove: 200  // Condições especiais acima de 200 unidades
```

## Regras importantes ao editar o workflow JSON

1. **Nunca adicionar múltiplos output arrays no nó Code** — o N8N só suporta 1 output por nó Code. Use `main: [[destino1, destino2, ...]]` para fan-out.
2. **continueOnFail: true** deve estar em todos os nós Redis e Google Sheets.
3. **Google Sheets documentId.mode deve ser `"id"`** — não `"list"` (evita dependência do picker).
4. **O `stateKey` e `leadKey` devem ser propagados** pelo Motor no JSON de saída.
5. **Telefone do consultor hardcoded**: `5535999619463` — altere se mudar.

## Como importar o workflow no N8N

1. Acesse o N8N em `http://SEU_IP:5678`
2. Menu → Workflows → Import from file
3. Selecione `workflows/pousinox_bot_v2.json`
4. Configure as credenciais (Redis, Google Sheets)
5. Ative o workflow (toggle Active)

## Como usar o Claude Code neste projeto

```bash
# Instalar Claude Code (se ainda não tiver)
npm install -g @anthropic-ai/claude-code

# Entrar na pasta do projeto
cd /opt/pousinox-bot

# Iniciar sessão
claude

# Exemplos de tarefas para pedir ao Claude Code:
# "Altere o preço do Inox 430 para R$13,00 no workflow"
# "Adicione o estado RS na lista de frete grátis"
# "Crie um novo stage para coletar o nome da obra"
# "Aumente o bônus de volume para R$0,08 por peça acima de 100 unidades"
```
