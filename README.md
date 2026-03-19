# Pousinox Bot — WhatsApp 24/7

Bot de atendimento automático para qualificação de leads da **Pousinox** via WhatsApp, focado no produto **fixador de porcelanato em inox**.

## Stack

- **N8N** — automação do fluxo
- **Z-API** — integração WhatsApp
- **Redis** — memória de sessão por lead
- **Google Sheets** — CRM e log de eventos
- **Melhor Envio** — cotação de frete (opcional)
- **PagBank** — link de pagamento

## Início rápido

### 1. Clonar o repositório

```bash
git clone https://github.com/SEU_USUARIO/pousinox-bot.git /opt/pousinox-bot
cd /opt/pousinox-bot
```

### 2. Configurar variáveis de ambiente

```bash
cp .env.example .env
nano .env   # preencha com suas credenciais
```

### 3. Subir os containers

```bash
docker compose up -d
```

### 4. Importar o workflow no N8N

Acesse `http://SEU_IP:5678`, vá em **Workflows → Import from file** e selecione:

```
workflows/pousinox_bot_v2.json
```

### 5. Configurar credenciais no N8N

- **Redis**: Settings → Credentials → New → Redis → `host: redis, port: 6379`
- **Google Sheets**: OAuth2 ou Service Account

### 6. Configurar webhook na Z-API

No painel Z-API, aponte o webhook para:

```
https://SEU_DOMINIO:5678/webhook/pousinox-whatsapp-v2
```

### 7. Ativar o workflow

No N8N, abra o workflow e clique no toggle **Active**.

---

## Fluxo da conversa

```
Cliente envia mensagem
  → Bot identifica o stage atual (Redis)
  → Coleta: tipo de inox → espessura → quantidade → CEP
  → Calcula orçamento + frete
  → Envia resumo ao cliente
  → Notifica consultor (handoff)
  → Salva lead no Google Sheets
```

## Comandos especiais

| Mensagem | Ação |
|---|---|
| `0` ou `menu` | Volta ao menu inicial |
| `2` / `humano` / `atendente` | Solicita atendimento humano |
| `#reset` | Reativa o bot (apenas para o consultor) |

## Desenvolvimento com Claude Code

Veja o arquivo [CLAUDE.md](CLAUDE.md) para instruções de como usar o Claude Code para editar o workflow.

---

**Pousinox** · Pouso Alegre, MG · [fixadorporcelanato.com.br](https://fixadorporcelanato.com.br)
