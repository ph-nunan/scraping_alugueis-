# scraping-guardrails

Regras de proteção contra overengineering no projeto de scraping.

## Checklist antes de implementar qualquer scraper

- [ ] Tentei a API pública do portal?
- [ ] Procurei JSON embutido (`__NEXT_DATA__`, `window.__STATE__`) na página?
- [ ] Tentei HTTP Request + HTML Extract antes de usar Playwright?
- [ ] Estou adicionando delay mínimo de 2s entre requests?
- [ ] A solução cabe em no máximo 5 nodes n8n?

## Proibido neste projeto

- Banco de dados (Postgres, MySQL, MongoDB) — Google Sheets é suficiente
- Filas (Redis, RabbitMQ) — n8n gerencia isso
- Proxies rotativos — volume é baixo, não precisa
- APIs de pagamento para scraping — tentar métodos gratuitos primeiro
- Frameworks frontend (React, Vue) para dashboard — HTML vanilla basta

## Quando usar Playwright
Somente se:
1. O site exige JavaScript para renderizar dados
2. E não há JSON embutido na página
3. E o HTML estático não contém os dados

## Tamanho máximo de workflow
- Cada workflow: máximo 15 nodes
- Se crescer além disso: dividir em subworkflows com Execute Workflow

## Lembrete
> "A solução mais simples que funciona é a correta."
