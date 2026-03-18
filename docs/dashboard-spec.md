# Especificação do Dashboard

## Opção Recomendada: Looker Studio (Google Data Studio)

**Por quê:** Conecta direto ao Google Sheets sem código, é gratuito, suporta filtros, cards e tabelas.

### Setup
1. Acessar https://lookerstudio.google.com
2. Criar novo relatório
3. Conectar fonte: Google Sheets → aba `listings_clean`
4. Criar os componentes abaixo

### Componentes do Dashboard

**Filtros (no topo)**
- Bairro/Região (dropdown)
- Faixa de preço total (slider ou dropdown: até 1500 / até 1800 / acima)
- Tags (multi-select: vaga, mobiliado, novo, boa_oportunidade)
- Status (novo / visto / descartado)

**Cards de métricas**
- Total de anúncios ativos
- Novos hoje
- Média de preço total
- Mais baratos (top 5)

**Tabela principal**
Colunas: imagem, título, bairro, aluguel, condomínio, total, quartos, vaga, tags, link, data

**Mapa (opcional)**
Se endereços estiverem preenchidos — Google Maps chart no Looker Studio

---

## Opção Alternativa: HTML simples + Sheets CSV

Para quem prefere uma página local sem depender do Looker Studio.

### Arquivo: `scraping_alugueis/dashboard/index.html`

**Funcionamento:**
- Lê o Google Sheets como CSV público
- Renderiza cards com filtros via JavaScript vanilla
- Sem frameworks, sem build step

### URL para CSV público do Sheets
```
https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?tqx=out:csv&sheet=listings_clean
```

### Funcionalidades mínimas
- Filtro por bairro, preço, tags
- Cards: imagem + preço + bairro + link para anúncio
- Destaque visual para `status = novo`
- Botão "ver anúncio" abrindo link em nova aba
- Ordenação por total_estimado ASC

### Atualização
- Sem deploy necessário — abre o HTML localmente
- Dados atualizados automaticamente a cada abertura (lê CSV fresh)

---

## Decisão MVP
Começar com **Looker Studio** (zero código, zero manutenção).
Migrar para HTML se precisar de customizações que o Looker não suporta.
