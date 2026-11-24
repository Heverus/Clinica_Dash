# Painel de gestão — Clínica Prado

Dashboard de indicadores para gestão de consultório clínico. Arquivo único (`index.html`), HTML + CSS + JavaScript puros, sem build e sem dependências.

**Ao vivo:** `https://heverus.github.io/clinica-painel/`

## O que tem

- 6 KPIs com variação contra o período anterior: receita líquida, atendimentos, ticket médio, ocupação, no-show e NPS
- Receita e atendimentos por mês com linha de meta e eixo duplo
- Medidor de atingimento, rosca por convênio, barras por especialidade
- Mapa de calor de ocupação por dia e horário
- Barras empilhadas de situação da agenda
- Tabela de procedimentos com receita, margem e percentual
- Origem dos pacientes e novos vs. recorrentes

Filtros de período (3, 6 ou 12 meses), especialidade e convênio recalculam tudo no navegador.

## Dados

A página roda em **modo demonstração**, com 1.311 atendimentos sintéticos embutidos no próprio HTML. Nenhum dado de paciente real está neste repositório.

Em produção, o painel lê um CSV publicado do Google Sheets (`CSV_URL` no bloco `CONFIG`), e a base completa com modelagem em estrela fica fora do repositório — justamente para não expor informação de saúde.

## Como usar

O arquivo abre direto no navegador. Para ligar a uma base real, publique a aba `vw_consultas` do Google Sheets como CSV e cole o link em `CONFIG.CSV_URL`.

## Licença

Uso livre para fins de demonstração.
