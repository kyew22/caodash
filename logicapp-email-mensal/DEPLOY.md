# Deploy – Logic App: E-mails Mensais CAO Dashboard

## O que faz
Envia e-mail a cada responsável nos **dias 1, 3 e 5** de cada mês, solicitando atualização do mês anterior.

| Área | Responsável | E-mail |
|------|-------------|--------|
| Departamento Pessoal | Tatyane | tatyane.santos@sysmanager.com.br |
| DHO | Tamires | tamires.lourenco@sysmanager.com.br |
| R&S | Caroline | caroline.daudt@sysmanager.com.br |
| T&D | Lucas | lucas.muratori@sysmanager.com.br |
| Suprimentos | Daniel | daniel.cesar@sysmanager.com.br |
| TI | Wellysson | wellysson.macena@sysmanager.com.br |
| GC | Paulo | paulo.felipe@sysmanager.com.br |

## Pré-requisitos
- Acesso ao Azure Portal com permissão de Contributor no Resource Group
- Conta Office 365 / Exchange Online para envio dos e-mails

---

## Deploy via Azure Portal (recomendado)

### Passo 1 — Abrir o Custom Deployment
1. Acesse: https://portal.azure.com
2. Pesquise "Deploy a custom template" na barra de busca
3. Clique em **"Build your own template in the editor"**
4. Cole o conteúdo do arquivo `logicapp-email-mensal.json`
5. Clique em **Save**

### Passo 2 — Preencher parâmetros
- **Subscription**: sua assinatura Azure
- **Resource Group**: mesmo do SWA (ex.: `sysmanager-rg` ou similar)
- **Location**: Brazil South (ou onde seus recursos já estão)
- **Logic App Name**: `sysmanager-email-mensal` (ou nome de sua preferência)
- **Office 365 Connection Name**: `office365` (padrão)

### Passo 3 — Deploy
Clique em **Review + Create** → **Create**. Aguarde ~1 minuto.

### Passo 4 — Autorizar a conexão Office 365 (OBRIGATÓRIO)
1. No Azure Portal, vá em **Resource Groups** → seu RG → **office365** (Connection)
2. Clique em **"This connection is not authenticated"**
3. Clique em **Authorize** e faça login com a conta que enviará os e-mails
4. Clique em **Save**

### Passo 5 — Testar
1. Vá ao Logic App `sysmanager-email-mensal`
2. Clique em **"Run Trigger"** → **"Run"**
3. Verifique o histórico de execução — todos os 7 e-mails devem aparecer como ✅

---

## Deploy via Azure CLI (alternativo)

```bash
az deployment group create \
  --resource-group SEU_RESOURCE_GROUP \
  --template-file logicapp-email-mensal.json \
  --parameters logicAppName=sysmanager-email-mensal
```

---

## Alterar o horário de envio
No arquivo JSON, localize `"startTime": "2024-01-01T08:00:00"` e ajuste o horário.
O fuso já está configurado para **Horário de Brasília** (`E. South America Standard Time`).

## Pausar os e-mails
No Azure Portal → Logic App → clique em **Disable**.
