# listing-normalizer

Regras de normalização dos dados coletados dos anúncios.

## Campos e transformações

| Campo raw | Transformação | Campo clean |
|-----------|---------------|-------------|
| Preço "R$ 1.200/mês" | extrair número | `preco_aluguel` = 1200 |
| Condomínio "R$ 350" | extrair número | `condominio` = 350 |
| Total | aluguel + condomínio | `total_estimado` |
| Bairro variado | normalizar capitalização | `bairro` |
| Quartos "1 quarto" | extrair número | `quartos` = 1 |
| Metragem "45m²" | extrair número | `metragem` = 45 |
| Garagem | verificar presença | `vaga` = TRUE/FALSE |

## Geração de tags

```javascript
const tags = [];
if (preco_aluguel < 1500) tags.push('abaixo_orcamento');
if (total_estimado <= 1800) tags.push('total_ate_1800');
if (condominio === 0) tags.push('sem_condominio');
if (isToday(data_coleta)) tags.push('anuncio_novo');
if (/mobiliado/i.test(descricao)) tags.push('mobiliado');
if (vaga) tags.push('vaga');
if (tags.includes('abaixo_orcamento') && vaga) tags.push('boa_oportunidade');
return tags.join(',');
```

## Hash de deduplicação

```
hash = MD5(link_do_anuncio)
```

Usar o node `Crypto` do n8n com algoritmo MD5.

## Campos opcionais (não bloquear se ausentes)
- telefone, whatsapp, endereço, metragem → colocar string vazia ou 0
- data_anuncio → usar data_coleta se não disponível
- imagem_principal → usar string vazia se não disponível

## Status inicial
Todo anúncio novo entra com `status = novo` e `destaque = FALSE`.
