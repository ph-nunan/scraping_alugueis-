# Auditoria de MCPs — Projeto Scraping de Aluguéis

## MCPs Existentes (mantidos, não duplicar)

| MCP | Localização | Função | Decisão |
|-----|-------------|--------|---------|
| `n8n-mcp` | `~/.claude/settings.json` | Criar/validar/deployar/atualizar workflows n8n | Mantido — base principal |
| `n8n-workflows Docs` | `~/.claude/settings.json` | 2053 workflows de referência | Mantido — usar para exemplos |
| `context7 Docs` | `~/.claude/settings.json` | Documentação live de APIs | Mantido — usar para docs atualizadas |

## MCPs Adicionados

| MCP | Localização | Função | Decisão |
|-----|-------------|--------|---------|
| `playwright` | `.mcp.json` (raiz projeto) | Browser automation para páginas dinâmicas | Adicionado — reusável em outros projetos |
| `firecrawl` | `.mcp.json` (raiz projeto) | Crawling e extração estruturada | Adicionado — requer FIRECRAWL_API_KEY |

## MCPs Descartados

| MCP | Motivo |
|-----|--------|
| Browserbase | Playwright já cobre o caso de uso |
| Hyperbrowser | Sem necessidade prática identificada no MVP |

## Notas de Ativação

### Firecrawl
Requer chave de API em `.mcp.json`:
```json
"FIRECRAWL_API_KEY": "sua-chave-aqui"
```
Obter em: https://www.firecrawl.dev/

### Playwright
Funciona sem configuração extra. Na primeira execução instala os browsers automaticamente.
