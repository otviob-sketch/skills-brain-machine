---
name: brain-machine-config
description: Configuracao tecnica do n8n, Google Sheets, credenciais, webhooks e seguranca
---

# Brain+Machine - Configuracao Tecnica

## Instalacao do n8n

**Docker (recomendado):**
```bash
docker run -d --name n8n \
  -p 5678:5678 \
  -v n8n_data:/home/node/.n8n \
  -e N8N_ENCRYPTION_KEY=sua_chave_forte_aqui \
  -e WEBHOOK_URL=https://seu-dominio.com \
  n8nio/n8n
```

**docker-compose.yml (producao):**
```yaml
version: '3.8'
services:
  n8n:
    image: n8nio/n8n
    ports:
      - "5678:5678"
    volumes:
      - n8n_data:/home/node/.n8n
    environment:
      - N8N_ENCRYPTION_KEY=${N8N_ENCRYPTION_KEY}
      - WEBHOOK_URL=https://n8n.seudominio.com
    restart: unless-stopped
```

**NPM (local):** `npm install n8n -g && n8n start`

## Gerenciamento de credenciais

Usar o gerenciador nativo do n8n (Settings -> Credentials).

Tipos comuns:
- **OAuth2**: Google, GitHub (n8n gerencia refresh)
- **API Key**: OpenAI, Stability AI, DeepL
- **Service Account**: Google Sheets, Drive (recomendado)
- **Basic Auth**: servicos internos

Boas praticas:
1. Nunca hardcodar chaves em workflows
2. Usar nomes descritivos (ex: "OpenAI - Producao")
3. Nao compartilhar credenciais entre ambientes
4. Rotacionar chaves a cada 90 dias
5. Usar .env e nunca commitar no git

## Google Sheets via Service Account

1. Acesse console.cloud.google.com
2. Crie projeto, habilite Google Sheets API
3. Crie Service Account, baixe JSON da chave
4. Configure no n8n: Credentials -> Google Service Account
5. Compartilhe a planilha com o email da service account (permissao Editor)

**Estrutura de planilha recomendada:**
```
Aba "pendentes": id | prompt | parametro1 | parametro2 | status | tentativas
Aba "logs": timestamp | workflow_id | acao | mensagem | nivel
```

## Template universal de workflow n8n

```
[Webhook] -> [Google Sheets: Read Rows] -> [Loop/SplitInBatches]
  -> [HTTP Request / API Call] -> [Google Sheets: Update Row]
  -> [IF error: Update Row com erro] -> [End]
  -> [Response / Callback]
```

**Payload de entrada (do OpenClaw):**
```json
{
  "workflow_id": "img-gen-001",
  "tipo": "imagem",
  "sheet_id": "id-da-planilha",
  "range": "pendentes!A:F",
  "callback_url": "https://openclaw.meusite.com/webhook/resultado",
  "parametros_globais": { "modelo": "flux", "tamanho": "1024x1024" }
}
```

**Payload de saida (callback):**
```json
{
  "workflow_id": "img-gen-001",
  "status": "concluido",
  "total": 50,
  "sucesso": 48,
  "erros": 2,
  "sheet_id": "id-da-planilha",
  "detalhes_erros": [{ "linha": 23, "mensagem": "timeout na API" }]
}
```

## Tratamento de erros

- Tentar 3x com backoff exponencial (5s, 15s, 45s)
- Marcar como "erro_permanente" apos 3 falhas
- Coluna "tentativas" na planilha para rastreio
- Workflow processa apenas linhas com status "pendente" (reinicio natural)

## Checklist de seguranca

- [ ] N8N_ENCRYPTION_KEY forte (32+ caracteres)
- [ ] HTTPS ativo (Let's Encrypt)
- [ ] Firewall bloqueia portas nao utilizadas (exceto 5678 so se necessario)
- [ ] Token secreto no header das chamadas webhook
- [ ] Rate limiting no n8n
- [ ] Backup regular do volume n8n_data
- [ ] .env nunca versionado no git
