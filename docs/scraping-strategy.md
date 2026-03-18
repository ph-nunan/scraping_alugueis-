# Estratégia de Scraping

## Regra de Ouro
**Tente na ordem. Pare assim que funcionar. Não pule etapas.**

```
1. API pública
2. JSON embutido (__NEXT_DATA__, window.__STATE__, etc.)
3. HTML estático (HTTP Request + HTML Extract)
4. Playwright (último recurso)
```

## Por Portal (investigado Mar/2026)

### OLX — `json_embutido` ✅
- **URL AC:** `https://www.olx.com.br/imoveis/aluguel/estado-df/.../ra-xx---aguas-claras?sf=1&pe=1500`
- **URL TAG:** `https://www.olx.com.br/imoveis/aluguel/estado-df/.../ra-iii---taguatinga?sf=1&pe=1500`
- **Como extrair:** `<script id="__NEXT_DATA__">` → `props.pageProps.ads[]`
- **Campos:** `subject`, `priceValue`, `listId`, `friendlyUrl`, `origListTime`
- **Filtros na URL:** `pe=1500` (preço até), `sf=1` (recentes)

### ZAP Imóveis — `html_json_ld` ✅
- **URL AC:** `https://www.zapimoveis.com.br/aluguel/apartamentos/df+aguas-claras/?tipoPagamento=aluguel&quartos=1`
- **URL TAG:** `https://www.zapimoveis.com.br/aluguel/apartamentos/df+taguatinga/?tipoPagamento=aluguel&quartos=1`
- **Como extrair:** `script[type="application/ld+json"]` → JSON-LD Product
- **Campos:** `name`, `numberOfBedrooms`, `floorSize`, `address`, `price`

### DF Imóveis — `html_json_ld` ✅
- **URL AC:** `https://www.dfimoveis.com.br/aluguel/df/aguas-claras/apartamento`
- **URL TAG:** `https://www.dfimoveis.com.br/aluguel/df/taguatinga/apartamento`
- **Como extrair:** JSON-LD `ItemList` → `itemListElement[].item`
- **Campos:** `name`, `offers.price`, `image[]`, `sku`, `offers.url`
- **Limitação:** quartos/m² na `description` (parse por regex)

### Rentola — `html_json_ld` ✅ (mais rico)
- **URL AC:** `https://rentola.com.br/alugar/aguas-claras`
- **Como extrair:** `RealEstateListing` Schema.org
- **Campos:** `price`, `floorSize`, `numberOfBedrooms`, `numberOfBathrooms`, `streetAddress`, `geo`, `postalCode`
- **Volume:** ~315 imóveis

### ImovelWeb — `html_json_ld` ✅
- **URL AC:** `https://www.imovelweb.com.br/apartamentos-aluguel-aguas-claras-df.html`
- **URL TAG:** `https://www.imovelweb.com.br/apartamentos-aluguel-taguatinga-df.html`
- **Como extrair:** JSON-LD `ItemList` com `RealEstateListing`
- **Campos:** `address`, `price`, `floorSize`, `numberOfBedrooms`, `image`

### QuintoAndar — `html_json_ld` ✅
- **URL AC:** `https://www.quintoandar.com.br/alugar/imovel/aguas-claras-brasilia-df-brasil`
- **URL TAG:** `https://www.quintoandar.com.br/alugar/imovel/taguatinga-brasilia-df-brasil/apartamento`
- **Como extrair:** JSON-LD `Product`
- **Campos:** `name`, `price`, `floorSize`, `numberOfBedrooms`, `address`, `image`

### WiMóveis — `playwright` ⚠️ (adiar para depois do MVP)
- **Tecnologia:** React com Loadable Chunks + AJAX
- **Dados não estão no HTML** — carregados via JavaScript
- **Status:** DESATIVADO no MVP. Ativar após os outros funcionarem.

## Implementação no n8n

### Nó HTTP Request (configuração base)
```
Method: GET
URL: {{$json.url}}
Headers:
  User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
  Accept: text/html,application/xhtml+xml
  Accept-Language: pt-BR,pt;q=0.9
```

### Extrair __NEXT_DATA__ (Code Node — usar somente se necessário)
```javascript
const html = $input.first().json.data;
const match = html.match(/<script id="__NEXT_DATA__" type="application\/json">(.*?)<\/script>/s);
if (!match) throw new Error('__NEXT_DATA__ não encontrado');
return [{ json: JSON.parse(match[1]) }];
```

### Quando usar Playwright
Apenas quando:
- O site renderiza conteúdo via JavaScript puro (sem Next.js/dados embutidos)
- O site exige interação (clique em botão "carregar mais", scroll infinito)
- O site bloqueia requests simples com Cloudflare/bot detection

## Respeito ao Site
- Mínimo de 2s entre requests (`Wait` node no n8n)
- Não raspar além do necessário
- Honrar `robots.txt` como boa prática
- Não escalar para centenas de requests por hora
