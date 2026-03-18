# Objetivo do Projeto

## Problema
Monitorar manualmente múltiplos portais imobiliários (OLX, Viva Real, Zap, etc.) em busca de apartamentos para alugar é tedioso e ineficiente.

## Solução
Sistema automatizado que:
1. Visita periodicamente as fontes cadastradas
2. Extrai anúncios de apartamentos
3. Normaliza e deduplica os dados
4. Armazena no Google Sheets
5. Exibe um dashboard leve e filtrável

## Critérios de Busca

| Campo | Valor |
|-------|-------|
| Tipo | Apartamento |
| Quartos | 1 |
| Região | Águas Claras ou Taguatinga, Brasília/DF |
| Aluguel máximo | R$ 1.500 |
| Total c/ condomínio | até R$ 1.800 (preferencial) |

## Princípios do Projeto
- Mínima complexidade possível
- Fácil manutenção por uma pessoa
- Sem banco de dados — Google Sheets como storage
- n8n como único orquestrador
- Dashboard simples, sem frameworks pesados

## Fora de Escopo (MVP)
- Alertas por e-mail/WhatsApp (pode ser adicionado depois)
- Machine learning / ranking automático
- Agendamento dinâmico por prioridade
- Múltiplas cidades
