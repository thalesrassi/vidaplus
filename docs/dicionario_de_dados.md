# Dicionário de Dados — VidaPlus Horizonte 2

Encoding UTF-8 · delimitador `,` · aspas duplas em campos com vírgula · datas em `YYYY-MM-DD`.

---

## F1 · `f1_atendimentos_<ano>.csv`
**Grão:** 1 linha = 1 atendimento. **Origem simulada:** HIS, PEP, RIS, LIS (5 arquivos anuais, 1.509.481 linhas).

| Campo | Tipo | Descrição |
|---|---|---|
| `id_atendimento` | string | Chave primária. `ATD` + 10 dígitos. |
| `id_paciente_pseudo` | string | Token pseudonimizado do paciente (`PAC-` + 12 hex). Estável ao longo dos 5 anos. |
| `id_unidade` | string | FK → `dim_unidades.id_unidade`. |
| `cod_municipio_ibge` | int | Município **da unidade**. FK → `dim_municipios`. |
| `uf` | string | SP, MG, BA ou RS. |
| `id_bairro_residencia` | string | Bairro **de residência do paciente**. FK → `dim_bairros.id_bairro`. |
| `data_admissao` | date | Data de entrada. |
| `hora_admissao` | string | `HH:MM`. |
| `ano_epidemiologico` | int | Ano da semana epidemiológica (≠ ano da data em viradas de ano). |
| `semana_epidemiologica` | int | SE brasileira, 1–53. |
| `tipo_entrada` | cat | `Pronto-Socorro`, `Internacao`, `UTI`, `Ambulatorial`. |
| `classificacao_risco` | cat | Protocolo de Manchester: `Vermelho`, `Laranja`, `Amarelo`, `Verde`, `Azul`. |
| `sexo` | cat | `F`, `M`. |
| `idade` | int | Anos completos (0–103). |
| `faixa_etaria` | cat | `0-4`, `5-14`, `15-29`, `30-49`, `50-64`, `65-79`, `80+`. |
| `cid10` | string | CID-10 do diagnóstico principal. FK → `dim_cid10`. |
| `capitulo_cid10` | string | Capítulo agregado do CID-10. |
| `internacao` | 0/1 | Houve internação. |
| `uti` | 0/1 | Houve passagem por UTI (implica `internacao = 1`). |
| `ventilacao_mecanica` | 0/1 | Uso de ventilação mecânica invasiva. |
| `dias_internacao` | int | Permanência total em dias (0 se não internou). |
| `dias_uti` | int | Dias em UTI (≤ `dias_internacao`). |
| `desfecho` | cat | `Alta melhorado`, `Obito`, `Transferencia externa`, `Evasao/Alta a pedido`. |
| `data_desfecho` | date | `data_admissao + dias_internacao`. |
| `custo_estimado_brl` | float | Custo estimado do episódio, em reais correntes (inclui correção ao longo da série). |
| `reinternacao_30d` | 0/1 | Internação ocorrida ≤ 30 dias após internação anterior do mesmo paciente. |
| `notificacao_compulsoria` | 0/1 | CID sujeito a notificação (arboviroses, COVID-19, influenza, pneumonia). |
| `sistema_origem` | cat | `HIS`, `PEP`, `RIS`, `LIS` — sistema legado que originou o registro. |

---

## F2 · `f2_sinan_sivep_notificacoes.csv`
**Grão:** 1 linha = 1 notificação compulsória (220.745 linhas).

| Campo | Tipo | Descrição |
|---|---|---|
| `id_notificacao` | string | Chave primária (`NOT` + 9 dígitos). |
| `sistema_origem` | cat | `SINAN` (arboviroses) ou `SIVEP-GRIPE` (SRAG e COVID-19). |
| `agravo` | cat | `Dengue`, `Chikungunya`, `Zika`, `SRAG`, `COVID-19`. |
| `cid10` | string | CID do agravo. |
| `data_inicio_sintomas` | date | Data de início dos sintomas. |
| `data_notificacao` | date | Data da notificação (0 a 8 dias após os sintomas). |
| `ano_epidemiologico` / `semana_epidemiologica` | int | Calendário epidemiológico da notificação. |
| `cod_municipio_ibge`, `municipio`, `uf` | | Município de notificação. |
| `id_bairro_residencia` | string | FK → `dim_bairros`. |
| `id_unidade_notificante` | string | FK → `dim_unidades` **ou** o literal `EXTERNA` (~38% — unidade fora da rede). |
| `sexo`, `idade`, `faixa_etaria` | | Perfil do notificado. |
| `classificacao_final` | cat | Inclui `Descartado` — **filtrar antes de calcular incidência**. |
| `criterio_confirmacao` | cat | `Laboratorial` ou `Clinico-epidemiologico`. |
| `agente_etiologico_sorotipo` | cat | Sorotipo DENV-1..4 para dengue; subtipo viral para SRAG; `-` quando não se aplica. |
| `hospitalizacao` | 0/1 | Caso hospitalizado. |
| `evolucao` | cat | `Cura`, `Cura pos-internacao`, `Obito`. |
| `data_encerramento` | date | Encerramento da ficha. |

> **Não** existe chave de paciente comum entre F1 e F2. A conciliação é parte do desafio.

---

## F3 · `f3_clima_diario.csv`
**Grão:** município × dia (21.912 linhas).

| Campo | Tipo | Descrição |
|---|---|---|
| `data` | date | Dia da observação. |
| `cod_municipio_ibge`, `municipio`, `uf` | | Estação de referência do município. |
| `temp_media_c`, `temp_max_c`, `temp_min_c` | float | Temperatura em °C. |
| `umidade_relativa_media_pct` | float | Umidade relativa média (%). |
| `precipitacao_mm` | float | Precipitação acumulada no dia (mm). |
| `dias_consecutivos_sem_chuva` | int | Sequência de dias com precipitação ≤ 1 mm. |
| `indice_uv_max` | float | Índice UV máximo do dia (0–14). |
| `velocidade_vento_media_ms` | float | Vento médio (m/s). |
| `pressao_atmosferica_hpa` | float | Pressão (hPa). |
| `estacao` | cat | `Verao`, `Outono`, `Inverno`, `Primavera` (hemisfério sul). |
| `fonte` | string | `INMET-SIM` (marca de origem sintética). |
| `ano_epidemiologico`, `semana_epidemiologica` | int | Calendário epidemiológico. |

---

## F4a · `f4a_recursos_leitos_equipes.csv`
**Grão:** unidade × dia (21.912 linhas). Derivado do censo real de pacientes de F1 — as duas
fontes são internamente consistentes.

| Campo | Tipo | Descrição |
|---|---|---|
| `data` | date | Dia do censo. |
| `id_unidade`, `cod_municipio_ibge`, `uf` | | Identificação da unidade. |
| `leitos_clinicos_operacionais` | int | Leitos clínicos efetivamente abertos (< cadastrados: bloqueios por equipe/reforma). |
| `leitos_clinicos_ocupados` | int | Censo de pacientes internados fora da UTI. **Pode exceder os operacionais.** |
| `taxa_ocupacao_clinica` | float | `ocupados / operacionais`. Pode ser > 1,0. |
| `leitos_uti_operacionais` | int | Leitos de UTI abertos (com expansão em 2021 e crescimento gradual). |
| `leitos_uti_ocupados` | int | Censo de UTI. |
| `taxa_ocupacao_uti` | float | `ocupados / operacionais`. Pode ser > 1,0. |
| `pacientes_aguardando_leito` | int | Fila por leito (excedente + demanda represada no PS). |
| `tempo_medio_espera_leito_horas` | float | Tempo médio de espera por leito. |
| `medicos_escalados` | int | Médicos efetivamente escalados no dia. |
| `enfermagem_escalada` | int | Profissionais de enfermagem escalados. |
| `taxa_absenteismo_equipe` | float | Fração da escala ausente (0–1). |
| `horas_extras_realizadas` | int | Horas extras do dia. |
| `ventiladores_disponiveis` | int | Ventiladores mecânicos livres. |
| `ano_epidemiologico`, `semana_epidemiologica` | int | Calendário epidemiológico. |

---

## F4b · `f4b_recursos_estoque_insumos.csv`
**Grão:** unidade × insumo × dia (219.120 linhas). Consumo derivado do censo real de F1/F4a.

| Campo | Tipo | Descrição |
|---|---|---|
| `data` | date | Dia. |
| `id_unidade` | string | FK → `dim_unidades`. |
| `cod_insumo`, `nome_insumo`, `unidade_medida` | | FK → `dim_insumos`. |
| `consumo_dia` | int | Consumo do dia (função de atendimentos + censo clínico + censo UTI). |
| `quantidade_recebida` | int | Entrada de estoque no dia (0 na maioria dos dias). |
| `estoque_final` | int | Saldo ao fim do dia. |
| `ponto_de_pedido` | int | Nível que dispara o ressuprimento (consumo médio × lead time + folga). |
| `dias_de_cobertura` | float | `estoque_final / consumo médio diário`. |
| `ruptura_estoque` | 0/1 | Saldo zerado no dia (desabastecimento). |
| `custo_unitario_brl` | float | Custo unitário do insumo. |

---

## F5 · `f5_ibge_socioeconomico_bairros.csv`
**Grão:** bairro × ano (720 linhas = 144 bairros × 5 anos).

| Campo | Tipo | Descrição |
|---|---|---|
| `cod_municipio_ibge`, `municipio`, `uf` | | Município. |
| `id_bairro`, `nome_bairro` | | FK → `dim_bairros`. |
| `ano` | int | 2021 a 2025. |
| `populacao_estimada` | int | População residente estimada. **Denominador para taxas de incidência.** |
| `area_km2` | float | Área do bairro. |
| `densidade_demografica_hab_km2` | float | Habitantes por km². |
| `renda_domiciliar_media_brl` | float | Renda domiciliar média mensal. |
| `pct_domicilios_agua_encanada` | float | % de domicílios com água encanada. |
| `pct_domicilios_esgoto_sanitario` | float | % com esgotamento sanitário. |
| `pct_domicilios_coleta_lixo` | float | % com coleta regular de lixo. |
| `idhm` | float | Índice de Desenvolvimento Humano Municipal do bairro (0–1). |
| `pct_populacao_0_14`, `pct_populacao_60_mais` | float | Estrutura etária. |
| `pct_populacao_dependente_sus` | float | % da população sem plano de saúde. |
| `num_unidades_basicas_saude` | int | UBS no bairro. |
| `cobertura_estrategia_saude_familia_pct` | float | Cobertura de ESF. |
| `indice_vulnerabilidade_social` | float | IVS 0–1 (maior = mais vulnerável). |
| `pct_domicilios_armazenamento_agua_inadequado` | float | % com armazenamento inadequado de água — **proxy de criadouro de vetor**. |

---

## Dimensões

**`dim_unidades.csv`** (12) — `id_unidade`, `nome_unidade`, `cod_municipio_ibge`, `porte`
(`Grande`/`Medio`/`Pequeno`), `perfil_assistencial` (`Geral/Alta Complexidade`,
`Urgencia/Trauma`, `Materno-Infantil`, `Geral`), `leitos_clinicos_cadastrados`,
`leitos_uti_cadastrados`, `municipio`, `uf`, `regional_saude`.

**`dim_municipios.csv`** (12) — `cod_municipio_ibge`, `municipio`, `uf`, `regional_saude`, `latitude`.

**`dim_bairros.csv`** (144) — `cod_municipio_ibge`, `municipio`, `uf`, `id_bairro`, `nome_bairro`,
`indice_vulnerabilidade_social`.

**`dim_cid10.csv`** (34) — `cid10`, `descricao_cid10`, `capitulo_cid10`.

**`dim_insumos.csv`** (10) — `cod_insumo`, `nome_insumo`, `unidade_medida`, `lead_time_dias`,
`custo_unitario_brl`.
