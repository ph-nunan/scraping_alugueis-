# sheets-schema-manager

Referência para manter o schema do Google Sheets consistente.

## Sheet ID
Guardar o ID da planilha no arquivo de configuração ou informar ao Claude quando necessário.
Formato: `docs.google.com/spreadsheets/d/{SHEET_ID}/`

## Abas e ordem
1. `config` — sempre primeira aba
2. `sources`
3. `runs`
4. `listings_raw`
5. `listings_clean` — principal, deve ser pública para leitura

## Como criar estrutura inicial
Pedir ao Claude: "Cria um workflow n8n para inicializar o schema do Google Sheets"
Claude criará um workflow que gera os cabeçalhos em todas as abas.

## Como verificar integridade
Pedir ao Claude: "Verifica se o schema do Sheets está correto"
Claude lê a primeira linha de cada aba e compara com o schema em `docs/google-sheets-schema.md`.

## Acessos necessários no n8n
- Credencial: Google Sheets OAuth2
- Escopos necessários: `spreadsheets` (leitura e escrita)

## Limites práticos do Sheets
- Máximo 10 milhões de células por planilha
- Para este projeto: não há risco de atingir o limite no MVP
- Se `listings_raw` crescer muito: deletar registros com mais de 60 dias
