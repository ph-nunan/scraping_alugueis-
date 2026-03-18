# source-intake

Skill de referência para cadastrar e validar fontes de scraping.

## O que é uma fonte (source)

Uma fonte é qualquer URL de listagem de imóveis que queiramos monitorar.
Ex: OLX com filtros aplicados, página de busca do Viva Real, etc.

## Campos obrigatórios ao cadastrar

```
id:          slug único (ex: olx-aguas-claras-1q)
nome:        nome legível (ex: OLX Águas Claras 1 quarto)
url:         URL completa com filtros já aplicados
metodo:      json_embutido | api | html | playwright
ativo:       TRUE
intervalo_horas: 6
```

## Processo de cadastro via Claude

1. Informar a URL ao Claude
2. Claude inspeciona a URL para determinar o método de scraping correto
3. Claude cria a linha na aba `sources` do Google Sheets via n8n
4. Claude confirma o cadastro lendo a linha de volta

## Validação antes de cadastrar

- URL deve ser acessível (HTTP 200)
- URL deve já conter os filtros desejados (quartos=1, preço máximo, etc.)
- Método deve ser o mais simples possível (seguir scraping-guardrails)

## Como atualizar uma fonte

Pedir ao Claude: "Atualiza a fonte [id] com [campo] = [novo_valor]"
Claude usará `n8n_update_partial_workflow` ou Google Sheets para atualizar.
