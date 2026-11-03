# Arquitetura — Projeto diretoriacao

## Visão Geral

```
┌─────────────────────────────────────────────────────────────┐
│                    Azure (Resource Group: caodash)           │
│                                                             │
│  ┌──────────────────────┐    ┌───────────────────────────┐  │
│  │  Azure Static Web App │    │   Azure Blob Storage      │  │
│  │  (cao-dash)           │◄──►│   (sysmanagerdashcao)     │  │
│  │  polite-stone-...     │    │   container: dashcao      │  │
│  │  SPA: dash-cao/       │    │   arquivo: data.json      │  │
│  │  API: /api/*          │    └───────────────────────────┘  │
│  └──────────────────────┘                                   │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Azure Logic App (sysmanager-email-mensal)           │   │
│  │  Trigger: dias 1, 3, 5 do mês — 08:00 Brasília      │   │
│  │  Ações: 7 e-mails via Office 365 Outlook            │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Sistemas publicados (Azure SWA: polite-stone-0f0145610.7.azurestaticapps.net)

| Path | Sistema | Descrição |
|------|---------|-----------|
| `/dash-cao/` | CAO Dashboard | Dashboard executivo — KPIs por área |
| `/onboarding/` | Onboarding 90 dias | Acompanhamento de onboarding |
| `/v2mom-m4/` | V2MOM M4 | Tracker de metas de Governança |
| `/talentos-ia/` | Talentos IA | Portal de captação de profissionais IA |

## Outros sistemas (SWAs separadas)

| Sistema | URL | Descrição |
|---------|-----|-----------|
| SoftSkill | softskill SWA | 8 módulos de clima/cultura |
| ISO 42001 · SGAI | polite-grass-05458f610.7.azurestaticapps.net | Controle SGAI |
| SysAction | lemon-moss-0fbcdf910.7.azurestaticapps.net | Governança de ações da Diretoria |

## CAO Dashboard — Fluxo de dados

```
Browser
  │
  ├── GET data.json (SAS URL) ──► Blob Storage
  │       │
  │   Object.assign(FALLBACK_DATA, blobData) → DATA em memória
  │
  ├── Edição pelo usuário → DATA (memória + localStorage)
  │
  └── saveToServer(DATA)
        │
        ├── 1. GET blob atual (no-store cache)
        ├── 2. mergeWithRemote(local, remoto)  ← proteção race condition
        └── 3. PUT resultado merged → Blob Storage
```

## mergeWithRemote — Lógica de merge

| Tipo de array | Chave primária | Comportamento |
|--------------|----------------|---------------|
| Mensal | `mes + ano` | merge field-level; null local preserva remoto |
| Anual | `ano` | merge field-level; null local preserva remoto |
| custom_kpis | `id` | local vence (último editor ganha) |
| Outros arrays | — | local vence |
| Escalares | — | local vence se não-null |

## Roles de usuário

```
admin  → edita dados + gerencia user_roles no blob
user   → edita dados
viewer → somente leitura (checkWritePermission() bloqueia PUT)
```

## Logic App — Fluxo de e-mails

```
Recorrencia_Dia1 (dias 1,3,5 — 08:00 BRT)
        │
        ▼
Calcular_mes_anterior
  formatDateTime(addDays(startOfMonth(utcNow()), -1), 'MMMM/yyyy', 'pt-BR')
        │
        ├──► Email_DP         → tatyane.santos@sysmanager.com.br
        ├──► Email_DHO        → tamires.lourenco@sysmanager.com.br
        ├──► Email_RS         → caroline.daudt@sysmanager.com.br
        ├──► Email_TD         → lucas.muratori@sysmanager.com.br
        ├──► Email_Suprimentos → daniel.cesar@sysmanager.com.br
        ├──► Email_TI         → wellysson.macena@sysmanager.com.br
        └──► Email_GC         → paulo.felipe@sysmanager.com.br
```

## Infraestrutura completa

| Recurso | Nome | Região | Resource Group |
|---------|------|--------|----------------|
| Static Web App | cao-dash | Central US | caodash |
| Storage Account (dashboard) | sysmanagerdashcao | Brazil South | caodash |
| Storage Account (onboarding) | sysmanageronboarding | Central US | caodash |
| Logic App | sysmanager-email-mensal | Brazil South | caodash |
| API Connection | office365 | Brazil South | caodash |
| Communication Services | softskill-comm | Global | caodash |
| Communication Services | sysmanager-onboarding-svc | Global | caodash |

## Decisões técnicas relevantes

- **SPA sem framework:** HTML/CSS/JS puro — zero dependências de build, deploy simples
- **Blob como banco:** JSON único no blob em vez de banco relacional — suficiente para volume atual, sem custo de servidor
- **SAS token de longa duração:** válido até 2030 — simplifica operação; aceite consciente de risco vs. complexidade de rotação
- **Race condition resolvida por merge:** fetch-before-write em vez de lock — eventual consistency aceitável para o caso de uso
- **Node 18 polyfill:** `globalThis.crypto` ausente no Node 18 do SWA runtime — corrigido em `api/src/index.js`
