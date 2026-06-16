---
name: brain-machine-telegram-whatsapp
description: Interface de comandos via Telegram e WhatsApp seguindo o padrao Brain+Machine
---

# Brain+Machine - Comandos via Telegram e WhatsApp

## Fluxo geral

```
[Usuario]
  | comando: "gerar 50 imagens de frutas"
  v
[Telegram / WhatsApp]
  | webhook envia para OpenClaw ou n8n
  v
[OpenClaw / LLM]
  | 1. Interpreta intencao
  | 2. Cria prompts estruturados
  | 3. Escreve na planilha Google Sheets
  | 4. Chama webhook do n8n
  v
[n8n]
  | 5. Le planilha
  | 6. Executa para cada linha
  | 7. Escreve resultados na planilha
  | 8. Chama callback
  v
[OpenClaw / LLM]
  | 9. Formata resposta
  v
[Usuario] <--- "Pronto! 50 imagens geradas."
```

## Telegram

**Criar bot:** BotFather -> `/newbot` -> obter TOKEN

**Opcao A - Direto no n8n:** Usar node "Telegram Trigger" no n8n (gerencia webhook automaticamente)

**Opcao B - LLM gerencia:** POST para `https://api.telegram.org/bot<TOKEN>/sendMessage` com `chat_id` e `text`

**Comandos (registrar via `/setcommands` no BotFather):**
- `/start` - Boas-vindas
- `/gerar <descricao>` - Iniciar workflow
- `/status <id>` - Verificar status
- `/listar` - Listar workflows
- `/cancelar <id>` - Cancelar workflow
- `/ajuda` - Ajuda

## WhatsApp

**Opcao 1 - WhatsApp Business API (recomendado producao):** Meta Business Suite, configurar webhook, enviar via API Graph

**Opcao 2 - Baileys:** `npm install baileys`, autentica via QR code, gratuito porem menos estavel

**Opcao 3 - Evolution API:** Projeto open-source que gerencia Baileys com API REST, facil integracao com n8n via HTTP

## Parsing de intencao natural

Ao receber um comando, extrair:
- **Intencao:** "gerar imagens", "enviar email", "traduzir", "processar dados", "relatorio"
- **Parametros:** quantidade, descricao, estilo/tom, destino, prazo
- **Acao:** Confirmar com usuario, estruturar dados na planilha, criar ID, disparar webhook, responder

**Exemplo:**
```
Usuario: "gerar 50 imagens de cafe gourmet estilo aquarela"
-> workflow_imagem, 50, "cafe gourmet", "aquarela"
-> Prompt: "cafe gourmet [variacao], estilo aquarela, fundo neutro"
```

## Templates de resposta

**Confirmacao:** "Recebi! ID: {ID}. {N} itens sendo processados..."
**Conclusao:** "Pronto! {N}/{N} itens concluidos. Link: {URL}"
**Erro parcial:** "{X} OK, {Y} erros. Quer reiniciar os erros?"
**Erro total:** "Nao foi possivel processar. Erro: {MSG}"

## Seguranca

- Whitelist de user_ids (Telegram) ou numeros (WhatsApp)
- Rate limiting (max X comandos/min/usuario)
- Log de todos os comandos recebidos
- Validar quantidade maxima de itens (ex: max 500)
