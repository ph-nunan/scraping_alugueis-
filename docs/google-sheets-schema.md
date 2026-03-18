# Schema do Google Sheets

## Estrutura de Abas

| Aba | Propósito |
|-----|-----------|
| `config` | Parâmetros globais do projeto |
| `sources` | URLs/fontes a monitorar |
| `runs` | Log de execuções |
| `listings_raw` | Dados brutos de cada coleta |
| `listings_clean` | Dados normalizados e deduplificados |

---

## Aba: `config`

| Coluna | Exemplo | Descrição |
|--------|---------|-----------|
| chave | `max_aluguel` | Nome do parâmetro |
| valor | `1500` | Valor atual |
| descricao | `Aluguel máximo em R$` | Documentação |

Linhas iniciais:
- `max_aluguel` → `1500`
- `max_total` → `1800`
- `regioes` → `Águas Claras,Taguatinga`
- `quartos` → `1`
- `tipo` → `apartamento`
- `ativo` → `true`

---

## Aba: `sources`

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| id | texto | Slug único da fonte (ex: `olx-aguas-claras`) |
| nome | texto | Nome legível (ex: `OLX Águas Claras`) |
| url | URL | URL base para scraping |
| metodo | enum | `json_embutido`, `api`, `html`, `playwright` |
| ativo | boolean | `TRUE` / `FALSE` |
| intervalo_horas | número | Com qual frequência executar |
| ultima_execucao | datetime | Preenchido automaticamente |
| notas | texto | Observações sobre o portal |

---

## Aba: `runs`

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| run_id | texto | UUID da execução |
| source_id | texto | FK para sources.id |
| inicio | datetime | Início da coleta |
| fim | datetime | Fim da coleta |
| status | enum | `ok`, `erro`, `parcial` |
| total_coletados | número | Anúncios brutos |
| total_novos | número | Anúncios após deduplicação |
| erro | texto | Mensagem de erro (se houver) |

---

## Aba: `listings_raw`

Dados brutos, sem normalização. Preservar para debug.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| hash | texto | MD5 da URL (chave de dedup) |
| source_id | texto | Fonte de origem |
| run_id | texto | Execução que coletou |
| data_coleta | datetime | Quando foi coletado |
| payload_json | texto | JSON completo do anúncio original |

---

## Aba: `listings_clean`

Dados normalizados, prontos para o dashboard.

| Coluna | Tipo | Descrição |
|--------|------|-----------|
| hash | texto | MD5 da URL (PK) |
| titulo | texto | Título do anúncio |
| preco_aluguel | número | R$ só aluguel |
| condominio | número | R$ condomínio (0 se não informado) |
| total_estimado | número | aluguel + condomínio |
| bairro | texto | Bairro normalizado |
| regiao | enum | `Águas Claras` / `Taguatinga` / `Outro` |
| endereco | texto | Endereço completo (se disponível) |
| quartos | número | Nº de quartos |
| banheiros | número | Nº de banheiros |
| metragem | número | m² (0 se não informado) |
| vaga | boolean | TRUE/FALSE |
| descricao | texto | Descrição resumida |
| link | URL | Link original do anúncio |
| imagem_principal | URL | URL da primeira imagem |
| anunciante | texto | Nome do anunciante/imobiliária |
| telefone | texto | Telefone (se disponível) |
| whatsapp | texto | WhatsApp (se disponível) |
| site_origem | texto | Portal (olx, vivareal, etc.) |
| data_coleta | datetime | Primeira vez que apareceu |
| data_anuncio | datetime | Data do anúncio no portal (se disponível) |
| data_atualizacao | datetime | Última atualização do registro |
| tags | texto | CSV: `abaixo_orcamento,vaga,novo` |
| status | enum | `novo` / `visto` / `descartado` |
| destaque | boolean | Marcado manualmente pelo usuário |

---

## Lógica de Tags Automáticas

| Tag | Critério |
|-----|----------|
| `abaixo_orcamento` | preco_aluguel < 1500 |
| `total_ate_1800` | total_estimado <= 1800 |
| `sem_condominio` | condominio = 0 |
| `anuncio_novo` | data_coleta = hoje |
| `mobiliado` | "mobiliado" na descrição |
| `vaga` | vaga = TRUE |
| `boa_oportunidade` | abaixo_orcamento E vaga |

---

## Deduplicação
- Hash = MD5 da URL do anúncio
- Se hash já existe em `listings_clean` → atualizar apenas `data_atualizacao` e `preco_aluguel`
- Se hash não existe → inserir nova linha com `status = novo`
