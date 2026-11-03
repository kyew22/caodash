# CLAUDE.md — Diretrizes para o projeto diretoriacao

## Regra crítica de segurança de dados

**SEMPRE** fazer backup do blob antes de qualquer redeploy.
O SAS token completo está na variável `_BLOB_SAS` dentro de `dash-cao/index.html`.

```bash
# Substituir <SAS_TOKEN> pelo valor de _BLOB_SAS no index.html
curl -o /tmp/blob_backup_$(date +%Y%m%d).json \
  "https://sysmanagerdashcao.blob.core.windows.net/dashcao/data.json<SAS_TOKEN>"
```

## Comando de redeploy

Executar **sempre de `~/swa-deploy-tmp`** — não de `/tmp` (EACCES RustDesk) nem do diretório do projeto (artifact folder conflict):

```bash
cd ~/swa-deploy-tmp && swa deploy /Users/ezequielfdossantos/diretoriacao \
  --api-location /Users/ezequielfdossantos/diretoriacao/api \
  --api-language node \
  --api-version 18 \
  --deployment-token <TOKEN_NO_MEMORY_project_deploy.md> \
  --env production
```

## Blob Storage

- **URL base:** `https://sysmanagerdashcao.blob.core.windows.net/dashcao/data.json`
- **SAS token:** ver variável `_BLOB_SAS` em `dash-cao/index.html`
- **user_roles** no blob é um dict `{ "email": "role" }`, não array
- Para adicionar usuário: editar `user_roles` no blob via PUT com o SAS

## Estrutura de dados

Arrays mensais têm chave `{ mes, ano }` como PK (ex: `{ mes: "Jan", ano: 2026 }`).
Arrays anuais têm chave `{ ano }`.
A função `mergeWithRemote(local, remote)` faz merge field-level: campos `null` no local preservam o valor remoto.

## Proteção contra race condition

`saveToServer` faz fetch-then-merge antes do PUT:
1. GET blob atual
2. `mergeWithRemote(DATA_local, blob_remoto)`
3. PUT resultado merged

## Azure Functions / Node 18

O runtime SWA usa Node 18.20.8 onde `globalThis.crypto` não existe. O polyfill está em `api/src/index.js`:
```js
try { if (!globalThis.crypto) globalThis.crypto = require('node:crypto').webcrypto; } catch(e){}
```
**Não remover este polyfill.**

## Resource Group principal

`caodash` (centralus) — contém SWA, Storage Account e Logic App.

## Logic App de e-mails

`sysmanager-email-mensal` (Brazil South, Resource Group `caodash`):
- Dispara dias 1, 3 e 5 de cada mês às 08:00 Brasília
- 7 destinatários: DP, DHO, R&S, T&D, Suprimentos, TI, GC
- Connection `office365` autorizada com ezequiel.santos@sysmanager.com.br
- Template ARM em `logicapp-email-mensal/logicapp-email-mensal.json`
