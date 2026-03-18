# Runbook Completo — Monitoramento de Aluguéis

**Para iniciantes. Cada passo tem exatamente o que fazer.**

---

## SITUAÇÃO ATUAL
- [x] Google Sheets criado com aba `sources_inicial`
- [ ] Ajustar estrutura do Sheets
- [ ] Configurar credencial n8n → Google Sheets
- [ ] Criar workflows n8n
- [ ] Testar coleta
- [ ] Configurar dashboard

---

## PASSO 1 — Ajustar o Google Sheets

### 1.1 — Renomear a aba importada

1. Abra o seu Google Sheets (o arquivo `sources_inicial`)
2. Na parte de baixo da tela, você verá a aba chamada **"sources_inicial"**
3. Clique com o **botão direito** nessa aba
4. Clique em **"Renomear"**
5. Digite: `sources`
6. Pressione Enter

### 1.2 — Criar as outras 4 abas

Para cada aba abaixo, clique no **"+"** no canto inferior esquerdo da tela,
depois clique com botão direito na nova aba → Renomear:

- `config`
- `runs`
- `listings_raw`
- `listings_clean`

### 1.3 — Cabeçalhos da aba `config`

Clique na aba **config**. Na linha 1, coloque:

| A1 | B1 | C1 |
|----|----|----|
| chave | valor | descricao |

Nas linhas 2 em diante, preencha:

| A | B | C |
|---|---|---|
| max_aluguel | 1500 | Aluguel máximo em R$ |
| max_total | 1800 | Total com condomínio máximo em R$ |
| regioes | Águas Claras,Taguatinga | Regiões monitoradas |
| quartos | 1 | Número de quartos desejado |
| tipo | apartamento | Tipo de imóvel |
| ativo | true | Sistema ativo ou não |

### 1.4 — Cabeçalhos da aba `runs`

Clique na aba **runs**. Na linha 1, coloque cada um em uma célula:

`run_id` | `source_id` | `inicio` | `fim` | `status` | `total_coletados` | `total_novos` | `erro`

### 1.5 — Cabeçalhos da aba `listings_raw`

Clique na aba **listings_raw**. Na linha 1:

`hash` | `source_id` | `run_id` | `data_coleta` | `payload_json`

### 1.6 — Cabeçalhos da aba `listings_clean`

Clique na aba **listings_clean**. Na linha 1, coloque estes 25 cabeçalhos (um por célula, A1 até Y1):

`hash` | `titulo` | `preco_aluguel` | `condominio` | `total_estimado` | `bairro` | `regiao` | `endereco` | `quartos` | `banheiros` | `metragem` | `vaga` | `descricao` | `link` | `imagem_principal` | `anunciante` | `telefone` | `whatsapp` | `site_origem` | `data_coleta` | `data_anuncio` | `data_atualizacao` | `tags` | `status` | `destaque`

### 1.7 — Copiar o ID da planilha

1. Olhe a URL do Sheets no navegador:
   ```
   https://docs.google.com/spreadsheets/d/XXXXXXXXXXXXXXXX/edit
   ```
2. Copie tudo entre `/d/` e `/edit`
3. **Salve esse ID** — você vai precisar nos próximos passos

### 1.8 — Tornar a planilha pública (para o dashboard)

1. Clique no botão **"Compartilhar"** (azul, canto superior direito)
2. Em "Acesso geral" → mude **"Restrito"** para **"Qualquer pessoa com o link"**
3. Mantenha como **"Leitor"**
4. Clique em **"Concluído"**

---

## PASSO 2 — Configurar Credencial no n8n

### 2.1 — Acessar o n8n

Abra: **https://n8n.paulonunan.com**

### 2.2 — Criar credencial do Google Sheets

1. Menu lateral esquerdo → clique em **"Credentials"** (ícone de chave/cadeado)
2. Botão **"+ Add credential"** (canto superior direito)
3. Na busca, digite: `Google Sheets`
4. Clique em **"Google Sheets OAuth2 API"**
5. Clique em **"Sign in with Google"**
6. Escolha sua conta Google (a mesma do Sheets)
7. Clique em **"Allow"** / **"Permitir"**
8. Volte ao n8n — deve aparecer verde (conectado)
9. **Nome da credencial:** `Google Sheets - Paulo` (ou qualquer nome)
10. Clique em **"Save"**
11. **Guarde o nome** que você deu — vai precisar nos próximos passos

---

## PASSO 3 — Criar os Workflows n8n

Com o Sheets configurado e a credencial criada, você vai pedir ao Claude
para criar cada workflow. Claude usa o MCP do n8n para criar diretamente.

**Substitua os valores e cole no chat:**

**Workflow 1 — Normalizador (criar primeiro):**
```
Cria o WF-03 Normalizador no n8n.
Sheet ID: [COLE_SEU_SHEET_ID_AQUI]
Credencial Sheets: [NOME_DA_CREDENCIAL]
```

**Workflow 2 — Coletor Principal:**
```
Cria o WF-01 Coletor Principal no n8n.
Sheet ID: [COLE_SEU_SHEET_ID_AQUI]
Credencial Sheets: [NOME_DA_CREDENCIAL]
```

**Workflow 3 — Limpeza Semanal:**
```
Cria o WF-04 Limpeza Semanal no n8n.
Sheet ID: [COLE_SEU_SHEET_ID_AQUI]
Credencial Sheets: [NOME_DA_CREDENCIAL]
```

---

## PASSO 4 — Testar a Coleta

1. No n8n, abra o **WF-01 Coletor Principal**
2. Clique em **"Test workflow"** (botão no topo)
3. Aguarde terminar (pode demorar 1-2 minutos)
4. Verifique no Google Sheets:
   - Aba `listings_raw` → deve ter linhas novas
   - Aba `listings_clean` → deve ter anúncios normalizados
   - Aba `runs` → deve ter um registro da execução

**Se aparecer erro:** Informe ao Claude:
```
O WF-01 deu erro no node [nome do node vermelho]. Mensagem: [mensagem de erro]
```

---

## PASSO 5 — Ativar o Agendamento

Quando o teste funcionar:

1. Abra o WF-01 no n8n
2. Canto superior direito: clique no **toggle (interruptor)**
3. Workflow vai ficar **ativo** e executar a cada 6 horas automaticamente

---

## PASSO 6 — Dashboard

### Opção A — Looker Studio (recomendado, zero código)

1. Acesse: **https://lookerstudio.google.com**
2. Clique em **"Criar"** → **"Relatório"**
3. Escolha **"Planilhas Google"** como fonte
4. Selecione sua planilha e a aba **listings_clean**
5. Clique em **"Adicionar ao relatório"**
6. Adicione uma tabela com: titulo, preco_aluguel, total_estimado, bairro, quartos, tags, link
7. Adicione filtros para: bairro, status, tags

### Opção B — Dashboard HTML (pedir ao Claude)

```
Cria o dashboard HTML para o projeto de aluguéis.
Sheet ID: [SEU_SHEET_ID]
```

Isso cria um arquivo HTML local. Você abre no browser sem precisar de servidor.

---

## REFERÊNCIA RÁPIDA

| Precisa fazer | Fale ao Claude |
|---------------|----------------|
| Adicionar nova fonte | "Adiciona a fonte [nome] com URL [url]" |
| Ver execuções com erro | "Mostra as últimas execuções com erro do WF-01" |
| Atualizar um workflow | "Atualiza o WF-01 para [mudança]" |
| Pausar coletas | Desativar WF-01 no n8n (toggle) |
| Site parou de funcionar | "O scraper do [portal] parou, corrige o WF-01" |
