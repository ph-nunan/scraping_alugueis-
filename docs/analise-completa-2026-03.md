# Análise Completa — Scraping de Aluguéis
**Data:** Março 2026 | **Sessões:** ~5 | **Status:** Em produção (workflow ativo no n8n)

---

## 1. O Que Foi Construído

Sistema de monitoramento automatizado de apartamentos para aluguel cobrindo **11 fontes** em Águas Claras e Taguatinga (Brasília/DF). Executa a cada 6 horas, filtra por critério (aluguel ≤ R$1.800) e salva no Google Sheets.

### Stack Final
| Componente | Tecnologia |
|---|---|
| Orquestrador | n8n self-hosted (`https://n8n.paulonunan.com`) |
| Storage | Google Sheets (ID: `1PA9M2xp8w_l8AFSbjQLCCpdXd8zHZl3KshdMMsT5ZAk`) |
| Bypass Cloudflare | Firecrawl API (`api.firecrawl.dev/v1/scrape`) |
| MCP de controle | n8n-mcp (criar/validar/deployar workflows via Claude) |

### Workflow Único — `1QES1eVsmJgQzFIR`
```
A cada 6h
  → Ler Fontes (Google Sheets: aba "sources")
  → Filtrar Ativas (ativo != FALSE)
  → Gerar Paginas (Code: até 5 páginas por fonte, exceto QuintoAndar=1)
  → Buscar Pagina (HTTP GET com headers browser)
  → Combinar Fonte (Set: junta HTML + metadados da fonte)
  → Bloqueado? (IF: status 403 ou contém "Cloudflare")
      TRUE  → Firecrawl API → HTML do Firecrawl
                                        ↘
      FALSE ──────────────────────────── Juntar Streams (Merge append)
  → Extrair e Normalizar (Code: extrai, dedup, filtra ≤ R$1.800)
  → Salvar no Sheets (aba "listings_clean")
```

### Fontes Monitoradas (11 ativas)
| Portal | Regiões | Método | Paginação |
|---|---|---|---|
| OLX | AC + TAG | `json_embutido` (`__NEXT_DATA__`) | `&o=N` |
| ZAP Imóveis | AC + TAG | `html_json_ld` (via Firecrawl) | `&pagina=N` |
| DF Imóveis | AC + TAG | `html_json_ld` | `?pagina=N` |
| Rentola | AC | `html_json_ld` | `?page=N` |
| ImovelWeb | AC + TAG | `html_json_ld` (via Firecrawl) | `-pagina-N.html` |
| QuintoAndar | AC + TAG | `html_json_ld` | sem paginação (SPA) |

---

## 2. Bugs Críticos Encontrados e Corrigidos

### Bug #1 — Filtro de preço antes da deduplicação (MAIS IMPACTANTE)
**Sintoma:** Quase todos os sites retornavam `preco_aluguel = 0` no Sheets.

**Causa raiz:** Sites como ZAP e ImovelWeb retornam o mesmo imóvel em **dois lugares** no JSON-LD da mesma página:
- `ItemList → itemListElement[].item` (Apartment com `offers.price = 2200`)
- `RealEstateListing.mainEntity[]` (mesmo Apartment, sem preço individual = 0)

O filtro `if (preco > 1800) continue` rodava **antes** da deduplicação por URL. Resultado:
1. Apartment com preço R$2.200 → filtrado, nunca entra no `byUrl`
2. Mesmo Apartment com preço R$0 → URL não existe no `byUrl` → entra com preço R$0
3. Resultado no Sheets: preço R$0 (mas o apartamento era R$2.200, corretamente excluído pelo filtro — exceto que aparecia como "gratuito")

**Fix:** Mover o filtro `PRECO_MAX` para **depois** da deduplicação. Dedup primeiro, filtro depois.

```javascript
// ERRADO (antes)
for raw items:
  if (preco > 1800) continue;  // ← filtro aqui remove o item com preço real
  byUrl[url] = item;           // ← só o item com preco=0 sobra

// CORRETO (depois)
for raw items:
  byUrl[url] = item (highest price);  // ← dedup primeiro, sem filtro

for urlKeys in byUrl:
  if (preco <= 0 || preco > 1800) continue;  // ← filtro aqui, após dedup
```

---

### Bug #2 — Firecrawl retornando HTML sem `<script>` tags
**Sintoma:** Sites via Firecrawl retornavam 0 listagens (JSON-LD não encontrado).

**Causa raiz:** Usado `formats: ["html"]` que retorna o **DOM renderizado** — `<script>` tags são removidas pelo processamento. O JSON-LD fica dentro de `<script type="application/ld+json">`.

**Fix:** Usar `formats: ["rawHtml"]` que retorna o HTML original sem processamento, preservando todos os `<script>` tags.

```javascript
// ERRADO
{ url: ..., formats: ["html"] }

// CORRETO
{ url: ..., formats: ["rawHtml"], onlyMainContent: false }
```

---

### Bug #3 — QuintoAndar: preço em `potentialAction`, não em `offers.price`
**Sintoma:** QuintoAndar retornava imóveis mas com `preco_aluguel = 0`.

**Causa:** QuintoAndar usa schema.org `Apartment` com preço em:
```json
"potentialAction": [{ "@type": "RentAction", "price": 1500, "priceCurrency": "BRL" }]
```
E **não** em `offers.price` (campo padrão).

**Fix:** Função `getAction()` que percorre `potentialAction[]` buscando qualquer ação com campo `price`:
```javascript
function getAction(obj) {
  const arr = Array.isArray(obj.potentialAction) ? obj.potentialAction : [obj.potentialAction];
  for (const a of arr) {
    if (a.price !== undefined && a.price !== null) return toNum(a.price);
  }
  return 0;
}
```

---

### Bug #4 — ZAP: `numberOfBathrooms` com nome diferente
**Sintoma:** Campo de banheiros sempre vinha 0 para ZAP.

**Causa:** ZAP usa `numberOfBathroomsTotal` (não `numberOfBathrooms` ou `numberOfFullBathrooms`).

**Fix:** Verificar múltiplos nomes:
```javascript
parseInt(item.numberOfFullBathrooms || item.numberOfBathroomsTotal || item.numberOfBathrooms || 0)
```

---

### Bug #5 — ImovelWeb: campo `type` em minúsculo
**Sintoma:** Imóveis do ImovelWeb não eram reconhecidos como `RealEstateListing`.

**Causa:** ImovelWeb usa `type` (minúsculo) em vez de `@type` (padrão schema.org):
```json
{ "type": "RealEstateListing", ... }  // ImovelWeb
{ "@type": "RealEstateListing", ... } // padrão
```

**Fix:**
```javascript
const type = getStr(obj, '@type') || getStr(obj, 'type');
```

---

### Bug #6 — n8n partial update revertendo para versão antiga
**Sintoma:** Após `n8n_update_partial_workflow`, o workflow voltou a ter 11 nodes (perdeu o node "Gerar Paginas").

**Causa:** O MCP aplica o diff sobre uma versão cacheada do workflow, não sobre a versão atual na API. Após context rollover entre sessões, o cache tinha uma versão antiga.

**Fix:** Usar `n8n_update_full_workflow` ao invés de partial update quando houver dúvida sobre o estado da versão cacheada. Full update é idempotente e seguro.

---

### Bug #7 — Nomes com acento quebrando partial update
**Sintoma:** Operações `removeConnection` e `moveNode` falhavam com erro de parsing.

**Causa:** Nomes de nodes com caracteres acentuados (ex: "Gerar Páginas" com acento em "Páginas") quebravam o parser do MCP em operações que referenciam nodes por nome.

**Fix:** Renomear todos os nodes para nomes sem acento: "Gerar Paginas", "Buscar Pagina", etc.

---

### Bug #8 — `WorkflowHasIssuesError` ao executar
**Sintoma:** Workflow recusava executar com erro "WorkflowHasIssues".

**Causas encontradas (4 erros de configuração):**
1. `range` ausente no node Google Sheets (`"A:Z"` era necessário)
2. `mode` com valor incorreto no Code node (`"runOnceForEachItem"` em vez de `"runOnceForAllItems"`)
3. URL e método ausentes no HTTP Request (Firecrawl)
4. `onError` dentro de `parameters` em vez de no nível do node

**Fix:** Corrigir cada campo via `n8n_update_partial_workflow` com `updates` (não `changes`).

---

## 3. Aprendizados Técnicos por Portal

### OLX ✅ (funciona perfeitamente)
- Usa Next.js — dados em `<script id="__NEXT_DATA__">` → `props.pageProps.ads[]`
- Preço: `a.priceValue` (número limpo, sem formatação)
- Data: `a.listTime` (Unix timestamp em segundos → `* 1000` para JS)
- Paginação: `&o=N` (parâmetro na query string)
- **Não precisa Firecrawl** — HTTP direto funciona

### ZAP Imóveis ✅
- Retorna **2 JSON-LD na mesma página** (é o bug #1):
  - `ItemList → Apartment` (com preços individuais)
  - `RealEstateListing.mainEntity[]` (sem preços individuais)
- Campo de banheiros: `numberOfBathroomsTotal`
- **Precisa Firecrawl** (Cloudflare)
- Paginação: `&pagina=N`

### DF Imóveis ✅
- JSON-LD: `ItemList → Product[]`
- Preço em `offers.price`, URL em `offers.url`
- **Não precisa Firecrawl** — HTTP direto funciona
- Paginação: `?pagina=N`

### Rentola ✅
- JSON-LD: `RealEstateListing` (schema.org mais rico)
- Campos disponíveis: `geo` (lat/lng), `postalCode`, `streetAddress`, `numberOfBedrooms`, `numberOfBathrooms`
- **Não precisa Firecrawl** — HTTP direto funciona
- Paginação: `?page=N`

### ImovelWeb ✅
- JSON-LD: `RealEstateListing.mainEntity[]`
- Usa `type` em minúsculo (não `@type`)
- Sem preços individuais nos items — fallback para `priceFromDesc()` (regex na descrição)
- Regex para preço na descrição: `/[Aa]luguel[^:]*:\s*R\$\s*([\d.,]+)/`
- **Precisa Firecrawl** (Cloudflare)
- Paginação: `-pagina-N.html` (inserido antes de `.html`)

### QuintoAndar ✅ (com ressalva)
- JSON-LD: `Apartment` individual (SSR, Next.js)
- Preço em `potentialAction[{@type:"RentAction"}].price` (não em `offers.price`)
- Endereço como string: `"Rua X, Bairro, Cidade"` → necessário `split(',')[1]` para bairro
- `floorSize`: número puro (não `{value: N, unitCode: "MTK"}`)
- Sem paginação via URL (SPA com scroll infinito)
- **Não precisa Firecrawl** — SSR completo via HTTP direto

---

## 4. Aprendizados sobre n8n e MCP

### n8n
- **`runOnceForAllItems`** é obrigatório no Code node quando se processa múltiplos items de uma vez (ex: todas as páginas juntas no "Extrair e Normalizar")
- **`pairedItem`** deve ser retornado por Code nodes para manter o contexto de item pai (necessário para `$('NodeName').item.json.*` funcionar downstream)
- **`onError: "continueRegularOutput"`** no HTTP Request permite que o workflow não pare em erros HTTP (404, timeout, etc.)
- Nomes de nodes COM acento em caracteres especiais causam bugs em partial updates — usar apenas ASCII simples
- O Merge node (Juntar Streams) recebe a stream "direta" no input 0 e a stream Firecrawl no input 1

### n8n MCP
- `n8n_update_partial_workflow` opera sobre versão CACHEADA — pode reverter se o cache estiver desatualizado após context rollover
- `n8n_update_full_workflow` é sempre seguro — substitui o workflow inteiro, requer campo `name`
- O validador MCP gera falsos positivos para Code nodes: "Cannot return primitive values" (funções auxiliares), "Avoid exec()" (RegExp.exec), "Invalid $ usage" (`$input.all()` válido)
- `valid: false` do MCP validator NÃO necessariamente impede a execução — só erros de configuração estrutural (campos required) impedem
- `operationsApplied: 1` no retorno não garante que o estado final está correto — sempre verificar com `n8n_get_workflow`

### Firecrawl
- `formats: ["html"]` = DOM renderizado (sem `<script>` tags) — **INÚTIL para JSON-LD**
- `formats: ["rawHtml"]` = HTML original (preserva tudo) — **CORRETO para JSON-LD**
- `onlyMainContent: false` é necessário para capturar scripts que ficam no `<head>`
- Cloudflare detection: checar `error_status === 403` OU `data.includes('Cloudflare')`

---

## 5. Melhorias Possíveis (Backlog)

### Alta prioridade
- [ ] **WF-04 Limpeza Semanal** — remover duplicatas acumuladas no Sheets, marcar anúncios antigos como "expirado"
- [ ] **Deduplicação cross-run** — antes de salvar, checar se o `hash` já existe no Sheets para evitar duplicatas entre execuções
- [ ] **Dashboard Looker Studio** — conectar no Sheets e criar visualização filtrável por bairro, preço, tags
- [ ] **Apps Script para formatação** — colorir linhas por tag (`boa_oportunidade` = verde, `preco_indefinido` = laranja)
- [ ] **Alertas WhatsApp** — quando novo anúncio `abaixo_orcamento` + `vaga` aparecer, notificar via API WhatsApp (já temos a infra da Scala)

### Média prioridade
- [ ] **Condomínio extraído** — tentar extrair valor de condomínio da descrição via regex e calcular `total_estimado` real (hoje = aluguel puro)
- [ ] **Quartos no filtro** — alguns sites retornam studios junto com qtos 1 quarto; adicionar filtro `quartos === 1`
- [ ] **WiMóveis via Playwright** — portal desativado por usar React puro (sem SSR). Ativar com MCP Playwright quando os outros estiverem estáveis
- [ ] **Teste de integridade pós-salvamento** — verificar via n8n se o Sheets recebeu os dados após cada run

### Baixa prioridade
- [ ] **Ranking automático** — score por quartos+metragem+preço+localização para ordenar melhores oportunidades
- [ ] **Histórico de preços** — registrar quando o preço de um anúncio muda (requer comparação de hash entre runs)
- [ ] **Múltiplas cidades** — arquitetura já suporta (basta adicionar fontes no Sheets)
- [ ] **Notificação por e-mail diário** — resumo dos novos anúncios do dia

---

## 6. Limitações Conhecidas

| Limitação | Impacto | Solução proposta |
|---|---|---|
| QuintoAndar sem paginação | Perde imóveis além da 1ª tela | Playwright com scroll infinito |
| Condomínio não extraído | `total_estimado` = aluguel puro (subestimado) | Regex na descrição + campo manual |
| Sem dedup cross-run | Duplicatas entre execuções acumulam no Sheets | WF-04 + hash check antes do append |
| ImovelWeb: preço por regex | Funciona apenas se descrição segue o padrão | Fallback para `lowPrice` agregado |
| Sites sem data de publicação | `data_anuncio` vazio | Usar `data_coleta` como proxy |
| Firecrawl rate limit | Pode falhar em muitas páginas simultâneas | Sleep/throttle entre requests |

---

## 7. Arquitectura de Dados — Google Sheets

### Aba `listings_clean` (25 colunas)
```
hash | titulo | preco_aluguel | condominio | total_estimado
bairro | regiao | endereco | quartos | banheiros | metragem | vaga
descricao | link | imagem_principal | anunciante | telefone | whatsapp
site_origem | data_coleta | data_anuncio | data_atualizacao | tags | status | destaque
```

### Tags geradas automaticamente
| Tag | Condição |
|---|---|
| `abaixo_orcamento` | preco ≤ R$1.500 |
| `total_ate_1800` | preco ≤ R$1.800 (sempre presente nos resultados) |
| `vaga` | título/descrição contém "vaga" ou "garagem" |
| `mobiliado` | título/descrição contém "mobiliado/a" |
| `boa_oportunidade` | preco ≤ R$1.500 E tem vaga |
| `anuncio_novo` | sempre presente na inserção inicial |

---

## 8. Decisões de Design

### Por que um único workflow em vez de um por portal?
Simplicidade de manutenção. Com fontes configuráveis no Sheets (`sources`), adicionar um novo portal é só adicionar uma linha — sem mexer no n8n. Funciona para 11+ fontes sem problema de performance.

### Por que Firecrawl e não Playwright para Cloudflare?
Firecrawl é stateless, barato e já inclui bypass de bot detection. Playwright exigiria browser headless rodando na VPS — mais recursos, mais complexidade. Firecrawl como serviço gerenciado é suficiente para o caso de uso.

### Por que não API pública dos portais?
Nenhum dos 11 portais oferece API documentada e gratuita para busca de imóveis. OLX tem uma API interna não documentada — mas os dados já estão disponíveis via `__NEXT_DATA__` sem necessidade de autenticação.

### Por que Google Sheets e não banco de dados?
Zero infraestrutura, zero custo, filtros/ordenação embutidos, compartilhável, visível sem dashboard. Para um uso pessoal de busca de apartamento, Sheets é mais do que suficiente.

---

## 9. Próximos Passos Imediatos

1. **Testar o workflow atual** — rodar manualmente e confirmar:
   - Preços aparecem corretamente em todos os sites (não mais 0)
   - Todos os registros têm `preco_aluguel` entre R$1 e R$1.800
   - QuintoAndar extrai preço via `potentialAction`
   - Data de publicação aparece quando disponível

2. **Ativar o schedule** — ligar o toggle no n8n para rodar a cada 6h automaticamente

3. **Criar WF-04 Limpeza Semanal** — deduplica o Sheets semanalmente

4. **Configurar dashboard** — Looker Studio conectado na aba `listings_clean`

---

## 10. Referências e IDs

| Recurso | Valor |
|---|---|
| Workflow n8n | `1QES1eVsmJgQzFIR` |
| Google Sheet ID | `1PA9M2xp8w_l8AFSbjQLCCpdXd8zHZl3KshdMMsT5ZAk` |
| Credencial n8n Sheets | `UeSaLFF10d9utrmA` (Google Sheets - Paulo) |
| n8n instance | `https://n8n.paulonunan.com` |
| Firecrawl API | `https://api.firecrawl.dev/v1/scrape` |
