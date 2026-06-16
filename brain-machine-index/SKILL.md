---
name: brain-machine-index
description: Indice e referencia rapida do conjunto de habilidades Brain+Machine (OpenClaw + n8n)
---

# Brain+Machine - Index

## Conjunto de 5 habilidades

| Habilidade | Conteudo |
|---|---|
| `brain-machine-fundamento` | Fundamento, arquitetura, matriz de decisao, 6 passos, exemplos |
| `brain-machine-config` | Setup tecnico: n8n, Google Sheets, credenciais, seguranca |
| `brain-machine-telegram-whatsapp` | Comandos via Telegram e WhatsApp, parsing de linguagem natural |
| `brain-machine-outros-canais` | Discord, Slack, Messenger, SMS, Web Chat, Email, Signal |

## Padrao Brain+Machine

**CEREBRO (OpenClaw/LLM) + MEMORIA (Google Sheets) + MAQUINA (n8n)**

1. Usuario envia comando via chat
2. OpenClaw interpreta intencao e cria dados estruturados
3. OpenClaw escreve dados na planilha Google Sheets
4. OpenClaw dispara webhook do n8n
5. n8n executa tarefa repetitiva deterministicamente
6. n8n escreve resultados na planilha
7. OpenClaw formata e envia resposta ao usuario

## Comandos padrao

- `/gerar <descricao>` - Inicia novo workflow
- `/status <id>` - Verifica status
- `/listar` - Lista workflows
- `/cancelar <id>` - Cancela workflow
- `/ajuda` - Mostra ajuda

## Referencia rapida

**Instalar n8n:** `docker run -d --name n8n -p 5678:5678 -v n8n_data:/home/node/.n8n -e N8N_ENCRYPTION_KEY=sua_chave n8nio/n8n`

**Workflow n8n padrao:** Webhook -> Google Sheets Read -> Loop -> HTTP Request -> Google Sheets Update -> Callback

**Google Sheets:** Criar Service Account no Google Cloud, habilitar Sheets API, compartilhar planilha com email da service account

**Estrutura de planilha:** `id | entrada | parametros | status | resultado | erro | timestamps`

**Credenciais:** Usar gerenciador nativo do n8n (Settings -> Credentials). Nunca hardcodar chaves.
