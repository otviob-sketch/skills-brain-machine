---
name: brain-machine-outros-canais
description: Interface de comandos via Discord, Slack, Messenger, SMS, Web Chat, Email e Signal
---

# Brain+Machine - Outros Canais de Comando

## Discord

**Criar bot:** discord.com/developers/applications -> New Application -> Bot -> Reset Token

**Integracao:** n8n tem node "Discord" nativo. Ou usar webhook do Discord para n8n.

**Slash commands (recomendado):** `/gerar`, `/status`, `/listar`, `/cancelar`, `/ajuda`

**Especificidades:** Suporta embeds, botoes, rate limit 50 req/s

## Slack

**Criar app:** api.slack.com/apps -> Create New App -> From Manifest

**Slash commands:** Configurar `/gerar` com Request URL apontando para n8n

**Block Kit:** Suporta mensagens ricas com botoes, selects, campos

**Especificidades:** response_type `in_channel` (todos veem) vs `ephemeral` (so o usuario)

## Facebook Messenger

**Setup:** developers.facebook.com -> App Business -> Adicionar Messenger

**Webhook:** Configurar Callback URL no Meta apontando para n8n. Responder challenge na verificacao.

**Envio:** POST para `https://graph.facebook.com/v18.0/me/messages`

**Especificidades:** Janela de 24h para resposta livre. Suporta Persistent Menu e Get Started button.

## SMS / Twilio

**Setup:** Criar conta Twilio, comprar numero, configurar webhook no console

**Receber:** n8n webhook recebe POST com `From`, `Body`, `MessageSid`

**Enviar:** POST para API Twilio com Basic Auth (Account SID + Auth Token)

**Especificidades:** Limite ~160 caracteres, sem botoes ou formatacao. Custo ~US$0.0079 (US).

## Web Chat (HTML/JS)

**Chat simples:** HTML + JS puro, POST para webhook n8n, exibe resposta

**Template minimo:**
```html
<input id="input" placeholder="Digite seu comando...">
<button onclick="enviar()">Enviar</button>
<script>
async function enviar() {
  const res = await fetch('https://n8n.seusite.com/webhook/chat', {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({ mensagem: input.value })
  });
  // exibir data.resposta no chat
}
</script>
```

## Email

**Opcao IMAP:** n8n conecta na caixa de entrada, le emails nao lidos, responde via SMTP

**Opcao Mailgun/SendGrid:** Servico converte email recebido em webhook HTTP para n8n

**Formato:** Assunto com o comando, corpo com detalhes

**Especificidades:** Nao e tempo real (polling a cada 1-5 min). Suporta anexos.

## Signal

**Via signal-cli-rest-api:** Docker com API HTTP. Polling de mensagens, envio via POST.

**Especificidades:** Criptografia ponta a ponta. Nao tem bots oficiais. Para uso pessoal/baixo volume.

## Tabela comparativa

| Plataforma | Setup | Custo | Tempo Real | Botoes | Comandos |
|---|---|---|---|---|---|
| Telegram | Facil | Gratuito | Sim | Sim | /slash |
| WhatsApp | Medio | Varia | Sim | Nao | text |
| Discord | Medio | Gratuito | Sim | Sim | /slash |
| Slack | Medio | Gratis* | Sim | Sim | /slash |
| Messenger | Dificil | Gratuito | Sim | Sim | text |
| SMS/Twilio | Facil | $$$ | Sim | Nao | text |
| Web Chat | Facil | Gratuito | Sim** | Sim | text |
| Email | Medio | Gratuito | Nao | Nao | subject |
| Signal | Dificil | Gratuito | Nao | Nao | text |

* Plano gratis: limite 10k mensagens, 90d historico
** Com WebSocket; HTTP simples e request/resposta

## Multi-canal

O mesmo n8n + OpenClaw pode atender varios canais simultaneamente:
1. Webhooks separados para cada canal no n8n
2. Todos chamam o mesmo workflow de processamento
3. Workflow verifica "canal_origem" no payload
4. Resposta enviada de volta pelo canal correspondente
