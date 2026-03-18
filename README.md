# Scraping Aluguéis — Monitoramento Automatizado

Sistema n8n que monitora **11 portais imobiliários** a cada 6 horas, filtra apartamentos por critério e salva no Google Sheets.

**Critérios:** Aptos de 1 quarto em Águas Claras ou Taguatinga/DF, aluguel ≤ R$1.500, total ≤ R$1.800.

## Status Atual (Março 2026)
- **Workflow:** `1QES1eVsmJgQzFIR` — deployado em `https://n8n.paulonunan.com`
- **Fontes ativas:** 11 (OLX×2, ZAP×2, DF Imóveis×2, Rentola×1, ImovelWeb×2, QuintoAndar×2)
- **Paginação:** até 5 páginas por fonte (~55 requests por execução)
- **Bypass Cloudflare:** Firecrawl API para ZAP e ImovelWeb

## Como Usar

### Ativar o workflow
1. Abra `https://n8n.paulonunan.com`
2. Abra o workflow "Scraping Alugueis - Coletor Principal"
3. Clique em **"Execute workflow"** para teste manual
4. Ligue o toggle para execução automática a cada 6h

### Adicionar nova fonte
Basta adicionar uma linha no Google Sheets na aba `sources`:
- `nome`, `url`, `metodo` (`json_embutido` ou `html_json_ld`), `regiao`, `ativo = TRUE`

### Importar em outra instância n8n
1. Importe `workflows/coletor-principal.json`
2. Configure credencial Google Sheets OAuth2
3. Substitua `SEU_SHEET_ID_AQUI` pelo ID real da planilha
4. Substitua `SUA_FIRECRAWL_KEY_AQUI` pela sua chave Firecrawl

## Estrutura

```
scraping_alugueis/
├── docs/
│   ├── analise-completa-2026-03.md  ← análise completa, bugs e aprendizados
│   ├── project-goal.md              ← objetivo e escopo
│   ├── scraping-strategy.md         ← como raspar cada portal
│   ├── google-sheets-schema.md      ← schema das abas
│   ├── n8n-workflows.md             ← descrição dos workflows
│   ├── dashboard-spec.md            ← spec do dashboard
│   └── runbook.md                   ← passo a passo completo
├── workflows/
│   └── coletor-principal.json       ← workflow n8n exportado (sem credenciais)
└── sources_inicial.csv              ← fontes com URLs e métodos de extração
```

## Stack
- **Orquestrador:** n8n self-hosted
- **Storage:** Google Sheets
- **Bypass anti-bot:** Firecrawl API (`rawHtml` format)
- **Extração:** `__NEXT_DATA__` (OLX) + JSON-LD schema.org (demais portais)
- **Dashboard:** Looker Studio (a configurar)
