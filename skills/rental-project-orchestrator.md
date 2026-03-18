# rental-project-orchestrator

Skill de orquestração: une n8n + scraping + Google Sheets + dashboard.

## Contexto completo do projeto

- **Docs:** `scraping_alugueis/docs/`
- **Schema:** `docs/google-sheets-schema.md`
- **Workflows:** `docs/n8n-workflows.md`
- **Estratégia de scraping:** `docs/scraping-strategy.md`
- **Dashboard:** `docs/dashboard-spec.md`
- **Runbook:** `docs/runbook.md`

## Fluxo completo

```
[sources cadastradas no Sheets]
    ↓
[WF-01: Coletor Principal — n8n, a cada 6h]
    ↓
[HTTP Request → extrair dados por método]
    ↓
[WF-03: Normalizador — Set + tags]
    ↓
[Deduplicação via hash MD5]
    ↓
[Google Sheets — listings_clean]
    ↓
[Dashboard — Looker Studio ou HTML]
```

## Como usar esta skill

Ao receber qualquer solicitação relacionada ao projeto de aluguéis, aplicar:

1. **Checar scraping-guardrails** antes de propor solução
2. **Seguir a ordem de scraping**: API → JSON embutido → HTML → Playwright
3. **Usar n8n-workflow-builder** para qualquer criação/edição de workflow
4. **Validar antes de deployar** (regra absoluta do N8N Builder)
5. **Documentar mudanças** nos arquivos relevantes de `docs/`

## Estado atual do projeto

| Componente | Status |
|-----------|--------|
| Google Sheets | A configurar (Passo 1 do runbook) |
| Credencial n8n-Sheets | A configurar (Passo 2) |
| WF-01 Coletor | A criar |
| WF-02 Fontes | A criar |
| WF-03 Normalizador | A criar |
| WF-04 Limpeza | A criar |
| Dashboard | A criar após Sheets |

## Próximo passo imediato
Seguir `docs/runbook.md` do Passo 1 ao 8.
