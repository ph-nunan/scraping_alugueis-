# Workflows n8n — MVP

## Visão Geral

```
[Schedule] → [Fetch Sources] → [Scrape] → [Normalize] → [Dedup] → [Write Sheets] → [Log Run]
```

## Workflows do MVP

### WF-01: Coletor Principal
**Trigger:** Schedule (a cada 6h)
**Função:** Orquestra toda a pipeline de coleta

```
Schedule Trigger
  → Google Sheets (ler sources ativas)
  → Split In Batches (processar uma source por vez)
  → HTTP Request (buscar página/API)
  → Switch (por metodo: json_embutido / api / html)
  → [extrator específico por método]
  → Set (normalizar campos)
  → Crypto (gerar hash MD5 da URL)
  → Google Sheets (verificar se hash existe)
  → IF (novo ou atualizar?)
  → Google Sheets (append ou update)
  → Google Sheets (log no runs)
```

### WF-02: Cadastro de Fontes
**Trigger:** Webhook ou manual
**Função:** Adicionar/atualizar fontes no Google Sheets

```
Webhook Trigger (ou Execute Workflow Trigger)
  → Set (validar campos obrigatórios)
  → Google Sheets (append em sources)
  → Respond to Webhook
```

### WF-03: Normalizador / Tagger
**Função:** Subchamado pelo WF-01 para normalizar dados e calcular tags
**Trigger:** Execute Workflow Trigger

```
Execute Workflow Trigger
  → Set (extrair campos padrão)
  → Number Node / IF nodes (calcular total_estimado)
  → Code Node mínimo (gerar tags CSV)
  → Return output
```

### WF-04: Limpeza / Manutenção
**Trigger:** Schedule semanal
**Função:** Remover anúncios muito antigos de listings_clean

```
Schedule Trigger (semanal)
  → Google Sheets (ler listings_clean)
  → Filter (data_coleta < hoje - 30 dias AND status != destaque)
  → Google Sheets (deletar linhas antigas)
```

## Nodes Principais Utilizados

| Node | Uso |
|------|-----|
| Schedule Trigger | Disparar coletas periódicas |
| Google Sheets | Ler/escrever todas as abas |
| HTTP Request | Buscar páginas dos portais |
| HTML Extract | Parsear HTML quando necessário |
| Set | Normalizar e mapear campos |
| IF / Switch | Branching por método de scraping |
| Split In Batches | Processar sources uma a uma |
| Code Node | Extrair __NEXT_DATA__ (apenas quando não há alternativa) |
| Crypto | Gerar hash MD5 para deduplicação |
| Wait | Delay entre requests (2s mínimo) |
| Execute Workflow | Reutilizar subworkflows |

## Notas de Implementação
- Usar `n8n_update_partial_workflow` para edições incrementais
- Validar cada workflow com `validate_workflow()` antes de deployar
- Pesquisar em `n8n-workflows Docs` por exemplos de scraping antes de construir
- Credencial Google Sheets: OAuth2 configurada na instância n8n
