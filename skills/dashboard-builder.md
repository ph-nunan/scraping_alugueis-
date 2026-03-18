# dashboard-builder

Referência para construir e manter o dashboard do projeto.

## Opção primária: Looker Studio

Setup básico em 15 minutos, sem código.

### Passos
1. https://lookerstudio.google.com → Criar → Google Sheets
2. Selecionar planilha e aba `listings_clean`
3. Adicionar filtros: bairro, total_estimado, tags, status
4. Adicionar tabela com colunas essenciais
5. Adicionar scorecards: total, novos hoje, média preço

### Link para compartilhar
Looker Studio gera URL pública. Favoritar no browser.

---

## Opção secundária: HTML vanilla

Quando precisar de mais controle visual.

### Solicitar ao Claude
"Cria o dashboard HTML para o projeto de aluguéis lendo do Google Sheets {SHEET_ID}"

Claude criará `scraping_alugueis/dashboard/index.html` com:
- Filtros por bairro, preço, tags
- Cards com imagem, preço, bairro
- Botão para abrir anúncio original
- Destaque para anúncios novos

### URL do CSV do Sheets
```
https://docs.google.com/spreadsheets/d/{SHEET_ID}/gviz/tq?tqx=out:csv&sheet=listings_clean
```

### Para atualizar o dashboard
Basta fechar e reabrir o HTML — os dados são lidos frescos do Sheets a cada vez.

---

## Fora de escopo (não adicionar no MVP)
- Login/autenticação
- Edição de dados no dashboard
- Notificações push
- PWA / service workers
