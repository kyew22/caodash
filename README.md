# CAO Dashboard

Dashboard executivo da Diretoria SysManager, publicado como Azure Static Web App.

**URL:** https://polite-stone-0f0145610.7.azurestaticapps.net/dash-cao/

---

## Visão Geral

SPA (Single Page Application) em HTML/CSS/JS puro com persistência em Azure Blob Storage. Não requer back-end próprio — dados são lidos e gravados diretamente no blob via SAS URL definida em `index.html` (`_BLOB_SAS`).

## Abas e Responsáveis

| Aba | Responsável | E-mail |
|-----|-------------|--------|
| Departamento Pessoal | Tatyane Santos | tatyane.santos@sysmanager.com.br |
| DHO | Tamires Lourenco | tamires.lourenco@sysmanager.com.br |
| R&S | Caroline Daudt | caroline.daudt@sysmanager.com.br |
| T&D | Lucas Muratori | lucas.muratori@sysmanager.com.br |
| Suprimentos | Daniel Cesar | daniel.cesar@sysmanager.com.br |
| TI | Wellysson Macena | wellysson.macena@sysmanager.com.br |
| GC | Paulo Felipe | paulo.felipe@sysmanager.com.br |

## Infraestrutura

| Recurso | Nome | Tipo |
|---------|------|------|
| Static Web App | cao-dash | Azure SWA (Consumption) |
| Blob Storage | sysmanagerdashcao | Azure Storage Account (Brazil South) |
| Resource Group | caodash | centralus |
| E-mail automático | sysmanager-email-mensal | Azure Logic App (Brazil South) |

## Persistência de Dados

- **Arquivo:** `https://sysmanagerdashcao.blob.core.windows.net/dashcao/data.json`
- **Acesso:** via SAS token definido em `index.html` (variável `_BLOB_SAS`)
- **Proteção contra race condition:** `mergeWithRemote()` faz fetch-then-merge antes de cada PUT
- **Fallback:** `localStorage` quando blob indisponível

## Roles de Usuário

Gerenciados no campo `user_roles` do `data.json`:
- `admin` — acesso total, gerencia usuários
- `user` — pode editar dados
- `viewer` — somente leitura

## E-mails Automáticos

Logic App `sysmanager-email-mensal` envia lembrete a cada responsável nos **dias 1, 3 e 5** de cada mês às 08:00 (Brasília), solicitando atualização do mês anterior.

## Deploy

**ATENÇÃO:** Sempre fazer backup do blob antes de redeploy. Ver `../CLAUDE.md` para o comando completo.

## Arquivos Principais

```
dash-cao/
├── index.html          # SPA principal (toda a lógica)
├── logoSys.png         # Logo SysManager (header + login)
└── README.md           # Este arquivo

../logicapp-email-mensal/
├── logicapp-email-mensal.json   # ARM template do Logic App
└── DEPLOY.md                    # Guia de deploy do Logic App
```
