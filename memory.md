# Memory — Projeto diretoriacao

Registro consolidado de decisões, configurações e histórico do projeto.
Última atualização: 2026-11-03

---

## CAO Dashboard (`dash-cao/`)

### O que é
Dashboard executivo SPA com persistência em Azure Blob Storage.
URL: https://polite-stone-0f0145610.7.azurestaticapps.net/dash-cao/

### Responsáveis por aba
| Aba | Responsável | E-mail |
|-----|-------------|--------|
| Departamento Pessoal | Tatyane Santos | tatyane.santos@sysmanager.com.br |
| DHO | Tamires Lourenco | tamires.lourenco@sysmanager.com.br |
| R&S | Caroline Daudt | caroline.daudt@sysmanager.com.br |
| T&D | Lucas Muratori | lucas.muratori@sysmanager.com.br |
| Suprimentos | Daniel Cesar | daniel.cesar@sysmanager.com.br |
| TI | Wellysson Macena | wellysson.macena@sysmanager.com.br |
| GC | Paulo Felipe | paulo.felipe@sysmanager.com.br |
| Admin | Ezequiel Santos | ezequiel.santos@sysmanager.com.br |

### Blob Storage
- Conta: `sysmanagerdashcao` — Brazil South — RG `caodash`
- Arquivo: `dashcao/data.json`
- SAS token: ver `_BLOB_SAS` em `dash-cao/index.html`

### user_roles (no blob)
Dict `{ "email": "role" }`. Roles: `admin`, `user`, `viewer`.

---

## Logic App — E-mails Mensais

**Nome:** `sysmanager-email-mensal`
**Região:** Brazil South | **RG:** caodash
**Status:** ATIVO (testado 03/11/2026, 2 runs Succeeded)

**Quando dispara:** dias 1, 3 e 5 de cada mês às 08:00 (Horário de Brasília)

**O que faz:** envia e-mail a cada responsável solicitando atualização do mês anterior no CAO Dashboard

**Connection Office 365:** autorizada com ezequiel.santos@sysmanager.com.br

**Template ARM:** `logicapp-email-mensal/logicapp-email-mensal.json`

---

## Alterações implementadas (2026-11-03)

- **Race condition corrigida:** `mergeWithRemote()` + fetch-before-write em `saveToServer`
- **Logo SysManager:** header e tela de login do dashboard
- **Abas removidas:** V2MOM e SG&A
- **Custom KPI tables:** iniciam com meses do ano corrente (antes iniciavam em Jan/2023)
- **% Cumprimento:** corrigido para contar apenas `cumpriu` (antes somava `cumpriu + parcial`)
- **portal.html:** adicionados cards SysAction e ISO 42001·SGAI em Gestão & Indicadores
- **TI responsável:** wellysson.macena@sysmanager.com.br (já estava no blob)

---

## Deploy

Executar **de `~/swa-deploy-tmp`** (não de `/tmp` nem do diretório do projeto).
Token e comando completo em `CLAUDE.md`.

---

## Infraestrutura completa

| Recurso | Nome | Região | RG |
|---------|------|--------|-----|
| Static Web App | cao-dash | Central US | caodash |
| Storage Account | sysmanagerdashcao | Brazil South | caodash |
| Logic App | sysmanager-email-mensal | Brazil South | caodash |
| API Connection | office365 | Brazil South | caodash |

---

## Outros sistemas (referência rápida)

| Sistema | Path/URL | Descrição |
|---------|----------|-----------|
| Onboarding 90d | `/onboarding/` | Table Storage direto via SAS |
| V2MOM M4 | `/v2mom-m4/` | Tracker metas Governança |
| Talentos IA | `/talentos-ia/` | Portal captação IA (OAuth pendente) |
| SoftSkill | SWA separada | 8 módulos clima/cultura |
| ISO 42001·SGAI | polite-grass-05458f610... | Controle SGAI |
| SysAction | lemon-moss-0fbcdf910... | Governança ações Diretoria |
| SysFlow | syscashflow.git | Fluxo de caixa (em desenvolvimento) |

---

## Decisões técnicas chave

| Decisão | Motivo |
|---------|--------|
| SPA sem framework | Zero dependências de build, deploy simples no SWA |
| Blob JSON como banco | Suficiente para volume atual, sem custo de servidor |
| SAS token longa duração (até 2030) | Simplifica operação; aceite consciente do risco |
| Merge antes de PUT | Protege dados quando múltiplos usuários editam simultaneamente |
| Polyfill `globalThis.crypto` | Node 18 no runtime SWA não tem crypto global (Node 20+ sim) |
| Logic App Consumption | Custo zero em execuções baixas; integração nativa Office 365 |
