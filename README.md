# n8n MDM Procurement Agent

Agente de MDM (Master Data Management) para gestão de cadastro de itens em áreas de Procurement, construído com **n8n** e **Claude AI (Anthropic)**.

O agente recebe solicitações de cadastro de novos itens via formulário, extrai os dados da proposta comercial em PDF usando a Files API do Claude, analisa os itens e gera um template padronizado para validação e entrada no sistema.

---

## O que o agente faz

1. **Recebe** a solicitação via formulário (nome do solicitante, e-mail e PDF da proposta comercial)
2. **Faz upload** do PDF para a Files API da Anthropic
3. **Chama o Claude** para extrair e padronizar os itens da proposta (descrição, conta contábil, UNSPSC, NCM e demais campos)
4. **Analisa** se há itens equivalentes já cadastrados
5. **Gera o template** de cadastro formatado e grava no Google Sheets para validação

---

## Fluxo no n8n

```
Formulário → Upload PDF (Files API) → Merge → Montar Requisição → Claude API → Parse Itens → Google Sheets
```

| Nó | Tipo | Função |
|----|------|--------|
| 01 — Formulário | Form Trigger | Coleta solicitante, e-mail e PDF |
| 02a — Salvar Campos Texto | Set | Preserva dados do formulário |
| 02b — Upload PDF | HTTP Request | Envia PDF para a Files API da Anthropic |
| 03 — Merge Campos + File ID | Merge | Combina dados do formulário com o file_id retornado |
| 04 — Montar Requisição Claude | Code | Monta o payload com o file_id e o prompt de extração |
| 05 — Chamar Claude API | HTTP Request | Envia requisição para `POST /v1/messages` |
| 06 — Parse Itens | Code | Extrai e estrutura os itens da resposta |
| 07 — Gravar no Google Sheets | Google Sheets | Registra o template para validação |

---

## Pré-requisitos

- [n8n](https://n8n.io/) self-hosted ou cloud (versão 1.x+)
- Conta na [Anthropic](https://console.anthropic.com/) com acesso à API e à **Files API** (beta)
- Conta Google com acesso ao Google Sheets (para OAuth2)

---

## Como importar

1. No n8n, acesse **Settings → Import Workflow**
2. Selecione o arquivo `flows/MDM_FilesAPI_v3.json`
3. Configure as credenciais conforme a seção abaixo
4. Ative o workflow

---

## Configuração de credenciais

O JSON **não contém nenhuma credencial**. Todos os campos sensíveis usam placeholders que precisam ser substituídos antes de usar.

### 1. Anthropic API Key

No n8n, crie uma credencial do tipo **HTTP Header Auth** com:

| Campo | Valor |
|-------|-------|
| Name | `Anthropic x-api-key` |
| Header Name | `x-api-key` |
| Header Value | `sua-chave-aqui` (obtida em [console.anthropic.com](https://console.anthropic.com/)) |

Após criar, vincule essa credencial aos nós **02b — Upload PDF** e **05 — Chamar Claude API**.

> ⚠️ A Files API requer o header `anthropic-beta: files-api-2025-04-14` — ele já está configurado nos nós HTTP Request do fluxo.

### 2. Google Sheets OAuth2

No n8n, crie uma credencial do tipo **Google Sheets OAuth2** com sua conta Google.

Após criar, vincule ao nó **07 — Gravar no Google Sheets**.

No mesmo nó, substitua o campo `documentId` pelo **ID da sua planilha** (encontrado na URL: `https://docs.google.com/spreadsheets/d/SEU_ID_AQUI/edit`).

---

## Estrutura da planilha esperada

O agente grava os seguintes campos no Google Sheets:

| Coluna | Descrição |
|--------|-----------|
| Solicitante | Nome do solicitante |
| E-mail | E-mail do solicitante |
| Descrição Padronizada | Descrição do item normalizada pelo Claude |
| Conta Contábil | Conta sugerida pelo agente |
| UNSPSC | Código de classificação UNSPSC |
| NCM | Código NCM (quando aplicável) |
| Status | `NOVO` ou `EXISTENTE` |
| Observações | Notas geradas pelo agente |

---

## Observações

- Este é um MVP funcional, desenvolvido para um contexto específico de Procurement em banco digital
- Para operações com maior volume ou criticidade, considere plataformas especializadas em MDM
- O modelo usado é `claude-sonnet-4-5` — pode ser substituído por versões mais recentes no nó 04

---

## Licença

MIT — sinta-se livre para adaptar ao seu contexto.
