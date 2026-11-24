# Painel de gestão da clínica

Dashboard em HTML (arquivo único, sem dependência) + base de dados em planilha com abas. Feito para um sócio acompanhar a clínica em uma tela.

---

## 1. Arquivos

| Arquivo | O que é |
|---|---|
| `index.html` | O painel completo. Abre no navegador, funciona offline. |
| `Base_Clinica.xlsx` | Base com 9 abas: dimensões, fatos, metas e visão de consumo. |
| `dados/vw_consultas.csv` | A mesma visão de consumo em CSV, pronta para publicar. |
| `dados/*.csv` | Todas as tabelas exportadas, para conferência ou carga em outro sistema. |
| `gerador_base.py` | Script que gera os dados sintéticos (regenera para outra escala). |
| `shot-*.png` | Capturas do painel. |

---

## 2. Abrir o painel

Basta abrir o `index.html` no navegador. Sem link de planilha ele abre em **modo demonstração**, com 1.311 atendimentos de exemplo embutidos e a faixa amarela avisando.

Para publicar em qualquer hospedagem estática: arraste a pasta no Netlify Drop, ou envie o `index.html` para o `public_html`.

> Abrir por `file://` funciona para o modo demonstração. Para ler a planilha, precisa estar em `http(s)`, porque o navegador bloqueia leitura externa em arquivo local.

---

## 3. Ligar ao Google Sheets

1. Suba o `Base_Clinica.xlsx` no Google Sheets (Arquivo → Importar).
2. **Arquivo → Compartilhar → Publicar na web**.
3. Escolha a aba **`vw_consultas`** e o formato **CSV**. Publique.
4. Copie o link e cole no `index.html`:

```js
const CONFIG = {
  CSV_URL: "https://docs.google.com/spreadsheets/d/e/2PACX-.../pub?gid=...&single=true&output=csv",
  CSV_METAS: "",   // opcional: mesma coisa, com a aba "metas"
  cacheMinutos: 5
};
```

5. Opcional, para o medidor de meta e a ocupação funcionarem com dados reais: publique também a aba **`metas`** e cole o link em `CSV_METAS`.

Sem `CSV_METAS`, o painel usa as metas embutidas da demonstração — o que distorce o atingimento. Com dados reais, publique as duas abas.

---

## 4. Modelo de dados

Modelo estrela: quatro dimensões em volta de uma tabela de fatos.

```
              dim_tempo
                  │
dim_pacientes ── fato_consultas ── dim_medicos
                  │
        dim_procedimentos ── dim_convenios
                  │
                metas (por mês)
```

| Aba | Papel | Grão |
|---|---|---|
| `dim_medicos` | quem atende | 1 linha por profissional |
| `dim_pacientes` | quem é atendido | 1 linha por paciente |
| `dim_procedimentos` | o que é feito e por quanto | 1 linha por procedimento |
| `dim_convenios` | como é pago e quanto se perde | 1 linha por convênio |
| `fato_consultas` | o evento | 1 linha por atendimento agendado |
| `metas` | metas do mês | 1 linha por mês |
| `resumo_mensal` | indicadores consolidados | 1 linha por mês |
| `vw_consultas` | visão de consumo (nomes resolvidos) | 1 linha por atendimento |

### Regras

- O fato registra **todo agendamento**, inclusive falta e cancelamento. Isso é o que permite medir no-show.
- `valor_bruto` só existe em atendimento realizado. Falta e cancelamento entram com zero.
- `glosa = valor_bruto × taxa_glosa` do convênio.
- `valor_liquido = valor_bruto − glosa`.
- `margem = valor_liquido − custo_operacional` do procedimento.
- `satisfacao` é a nota 0–10 do paciente, apenas em atendimento realizado.
- `slots_ofertados` (aba `metas`) é a capacidade da agenda no mês — sem ela não há como calcular ocupação.

### Campos principais de `fato_consultas`

| Campo | Tipo | Observação |
|---|---|---|
| `id_atendimento` | texto | chave |
| `data` | data | ISO (AAAA-MM-DD) |
| `hora` | número | hora cheia do início |
| `id_paciente` / `id_medico` / `id_procedimento` / `id_convenio` | texto | chaves estrangeiras |
| `status` | texto | Realizada, Faltou, Cancelada |
| `primeira_vez` | texto | Sim/Não |
| `duracao_min` / `espera_min` | número | minutos |
| `valor_bruto` / `glosa` / `valor_liquido` / `custo` / `margem` | moeda | R$ |
| `satisfacao` | número | 0 a 10 |

---

## 5. Dicionário de indicadores

| Indicador | Como é calculado | Por que importa |
|---|---|---|
| **Receita líquida** | soma de `valor_liquido` das realizadas | o que efetivamente entra |
| **Atendimentos** | contagem de status Realizada | volume real, não agendado |
| **Ticket médio** | receita ÷ atendimentos | preço médio praticado |
| **Ocupação** | realizadas ÷ `slots_ofertados` | mede agenda ociosa |
| **No-show** | faltas ÷ total agendado | dinheiro que evapora |
| **NPS** | (% notas 9–10) − (% notas 0–6) | satisfação e indicação |
| **Glosa** | glosa ÷ valor bruto | quanto o convênio retém |
| **Margem** | (receita − custo) ÷ receita | sobra por atendimento |
| **Espera média** | média de `espera_min` | percebida como qualidade |
| **Duração média** | média de `duracao_min` | base para dimensionar agenda |

Cada cartão mostra a variação contra o período anterior de mesmo tamanho. Em "No-show", cair é bom — por isso a seta verde aparece quando o número diminui.

---

## 6. O que o painel mostra

- **6 KPIs** com variação e minigráfico: receita, atendimentos, ticket, ocupação, no-show e NPS.
- **Receita líquida e atendimentos**: área da receita, linha da meta e linha de volume em eixo duplo.
- **Atingimento da meta**: medidor semicircular com percentual do período.
- **Receita por convênio**: rosca com participação.
- **Receita por especialidade**: barras ordenadas.
- **Perdas e margem**: glosa, custo e margem em percentual.
- **Ocupação por dia e horário**: mapa de calor Seg–Sex × 8h–18h.
- **Situação da agenda**: barras empilhadas por mês (realizada, falta, cancelada).
- **Procedimentos por resultado**: tabela com volume, receita, margem e %.
- **Origem dos pacientes**: barras de aquisição.
- **Novos e recorrentes**: duas linhas por mês.
- **Experiência**: NPS, espera e duração.

Filtros de período (3, 6 ou 12 meses), especialidade e convênio recalculam tudo no navegador.

---

## 7. Cores

Paleta neutra com dois acentos, para leitura rápida sem cansar:

```css
--ink:#14181C;      /* texto e barras do menu */
--bg:#F4F6F7;       /* fundo */
--teal:#0F5E57;     /* acento principal: receita e métricas boas */
--teal-2:#2E8B7F;   /* séries secundárias */
--gold:#C8A46B;     /* meta, faltas, comparações */
--neg:#B4453A;      /* alerta: no-show e queda */
```

O verde-petróleo puxa o olho para receita e resultado; o dourado marca meta e falta; o vermelho aparece só quando o indicador piora.

---

## 8. Regenerar os dados

```bash
pip install openpyxl
python gerador_base.py
```

Gera novamente a planilha, os CSVs e o bloco de dados embutido no HTML. O gerador usa semente fixa, então o resultado é reprodutível.

Para trocar por dados reais do sistema da clínica: exporte do sistema (ou preencha a planilha) mantendo as colunas de `vw_consultas` — o painel não precisa de nada além delas.

---

## 9. Limitações

- Ocupação depende de `slots_ofertados`; se a clínica não registrar a capacidade, o indicador fica sem base.
- Ao filtrar por especialidade, os slots são rateados pela quantidade de médicos da especialidade — é uma aproximação, não a agenda real de cada um.
- Valores do fatos são históricos: não há previsão de recebimento nem contas a receber por convênio com prazo.
- O painel não grava nada: só lê o CSV e calcula no navegador.
