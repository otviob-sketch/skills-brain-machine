---
name: brain-machine-fundamento
description: Padrao Brain+Machine - quando aplicar, arquitetura, metodologia de 6 passos e estimativa de economia
---

# Brain+Machine - Fundamento

## O que e o padrao

Separa o trabalho em dois papeis:

**CEREBRO (OpenClaw / LLM)**
- Entende intencao do usuario em linguagem natural
- Planeja, cria prompts, define parametros
- Estrutura dados para execucao
- Formata resultados para o usuario
- GASTA TOKENS -> deve pensar UMA vez

**MAQUINA (n8n)**
- Execucao deterministicas repetitivas
- Mesmo processo N vezes sem variacao
- NAO GASTA TOKENS
- Processa mais rapido que LLM
- Confiavel: se falhar na linha 23, recomeca da linha 23

**COL A (Google Sheets)**
- Camada de dados compartilhada
- OpenClaw escreve parametros de entrada
- n8n le, processa e escreve resultados

## Arquitetura de referencia

```
[Usuario] -> [OpenClaw/LLM] -> [Google Sheets] -> [n8n Webhook] -> [Google Sheets] -> [OpenClaw/LLM] -> [Usuario]
```

## Matriz de decisao

**APLIQUE quando TODOS os criterios forem verdade:**
- A tarefa envolve repeticoes (3+ iteracoes)
- Cada iteracao segue o MESMO padrao/template
- Cada iteracao NAO requer julgamento criativo unico
- Pode ser descrita em passos deterministicos
- Ha formato de dados bem definido

**NAO APLIQUE quando QUALQUER criterio for verdade:**
- Cada iteracao requer tomada de decisao contextual unica
- Resultado de uma iteracao AFETA a proxima
- Requer compreensao contextual continua
- Numero de iteracoes e imprevisivel
- Requer validacao/feedback humano entre iteracoes

## Metodologia universal - 6 passos

1. **IDENTIFICAR** a tarefa repetitiva no fluxo atual
2. **SEPARAR** cerebro (decisao) da maquina (execucao)
3. **DEFINIR** estrutura de dados na planilha (colunas de entrada/saida)
4. **CRIAR** workflow no n8n (Webhook -> Sheets Read -> Loop -> API Call -> Sheets Update)
5. **CONECTAR** via webhook
6. **TESTAR** com 3-5 itens, depois escalar

## Exemplos

- **Imagens**: OpenClaw cria prompts, n8n chama API (Stability AI/DALL-E)
- **Email**: OpenClaw cria template, n8n envia via SendGrid/SES/SMTP
- **Dados**: OpenClaw define regras, n8n aplica linha a linha
- **Traducao**: OpenClaw define tom, n8n chama API (DeepL/Google Translate)
- **Relatorios**: OpenClaw cria template, n8n puxa dados e formata

## Estimativa de economia

**50 imagens com OpenAI:**
- SEM padrao: ~35.000 tokens, ~US$0.70, ~5-10 minutos
- COM padrao: ~3.000 tokens, ~US$0.06, ~1-2 minutos
- Economia: ~90% tokens, ~80% tempo
