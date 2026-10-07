# Painel de Abastecimento

Pipeline de dados que cruza a telemetria dos equipamentos de distribuição de água (VRPs, boosters, estações elevatórias e reservatórios) com as manifestações de clientes, as manobras de rede e os eventos de energia, para identificar automaticamente por que uma região ficou sem água e quais reclamações precisam de avaliação técnica.

## O problema

Quando uma rua ou bairro passa a receber várias manifestações de falta d'água (duas, três, quatro ou mais), é preciso descobrir o porquê. A causa pode estar em diferentes pontos do sistema:

- **VRP** (válvula redutora de pressão) com queda de pressão ou aumento anormal de vazão
- **Booster** ou **estação elevatória (EEAT)** parado ou travado
- **Reservatório** com nível baixando ou vazão de saída subindo muito, o que pode indicar **vazamento**
- **Queda de energia**, que para bombas e boosters
- **Manobra de rede** programada ou emergencial

Hoje essa correlação é feita manualmente, consultando sistemas diferentes e copiando informações de um para outro. O processo é lento, sujeito a erro e não gera histórico para análise.

Este projeto automatiza esse cruzamento, guarda o histórico e usa machine learning para estimar o risco de travamento de cada equipamento antes que ele aconteça.

> Os dados de telemetria, manifestações, manobras e eventos de energia são **simulados** e imitam a estrutura de um ambiente real de saneamento. Os dados de clima (INMET) e de bairros (IBGE) são públicos. Nenhum dado real de cliente ou de empresa é utilizado.

## Quem usa

| Usuário | Necessidade |
|---|---|
| Operador do centro de controle | Saber rapidamente se uma manifestação tem causa conhecida |
| Coordenação regional | Ver quais bairros, VRPs, boosters e estações elevatórias concentram mais problemas |
| Equipe de manutenção | Priorizar para onde enviar técnicos |

## Perguntas de negócio

| # | Pergunta | Prioridade |
|---|---|---|
| 1 | Quais VRPs, boosters e estações elevatórias tiveram queda de pressão ou travamento, em que dia e horário? | P1 |
| 2 | Quantas manifestações chegaram por bairro, por rua e por hora? | P1 |
| 3 | Qual % das manifestações tem causa conhecida (falha de equipamento, manobra ou queda de energia)? | P1 |
| 4 | Quais manifestações estão sem causa conhecida e precisam de avaliação técnica no setor? | P1 |
| 5 | Quais ruas acumularam várias manifestações em pouco tempo (possível vazamento ou falha não registrada)? | P1 |
| 6 | Quanto tempo passa entre a falha do equipamento e a primeira reclamação? | P2 |
| 7 | Quedas de energia explicam quantas paradas de boosters e elevatórias? | P2 |
| 8 | Calor e falta de chuva aumentam as reclamações? | P2 |
| 9 | Quais bairros têm mais reclamações por mil habitantes? | P2 |
| 10 | Quais equipamentos têm maior risco de travar nas próximas 24 horas? | ML |

**P1** = essencial · **P2** = complementar · **ML** = respondida pelo modelo de machine learning

## Requisitos

### Funcionais
- Ingerir a telemetria de VRPs, boosters, estações elevatórias e reservatórios a cada 30 minutos (pressão, vazão, nível e status)
- Ingerir manifestações, manobras e eventos de queda de energia diariamente
- Detectar anomalias simples: queda de pressão, aumento de vazão, queda de nível, equipamento parado
- Agrupar manifestações por rua e janela de tempo para identificar concentrações
- Cruzar manifestações com falhas de equipamento, manobras e quedas de energia por área e horário
- Classificar cada manifestação como "com causa conhecida" ou "sem causa conhecida"
- Exibir os resultados em um painel
- Guardar o histórico completo da telemetria
- Estimar, com machine learning, o risco de travamento de cada equipamento nas próximas 24 horas e exibir no painel

### Não funcionais
- Ambiente completo sobe com um único comando (Docker Compose)
- Testes de qualidade nos dados (sem duplicados, sem pressão negativa etc.)
- Apenas dados simulados e públicos
- Documentação suficiente para qualquer pessoa reproduzir o projeto

### Fora do escopo (v1.0)
- Processamento em tempo real (streaming)
- Deploy completo em nuvem

## Manutenção preditiva (machine learning)

Equipamentos costumam dar sinais antes de travar: a pressão começa a oscilar, a vazão sobe aos poucos, o nível do reservatório cai fora do padrão. Os dados simulados incluem esses sinais, e o modelo aprende com o histórico a reconhecê-los.

| Etapa | O que é feito |
|---|---|
| Variáveis | Médias móveis e variações de pressão, vazão e nível por equipamento |
| Rótulo | "O equipamento travou nas próximas 24 horas?" (sim/não) |
| Treino | Modelo de classificação no Databricks, com experimentos registrados no MLflow |
| Avaliação | Precisão e recall: quantos alertas acertam e quantos travamentos o modelo pega |
| Uso | Risco de travamento por equipamento exibido no painel |

## Práticas de engenharia (padrão corporativo)

| Prática | Como é aplicada |
|---|---|
| LGPD | Dados pessoais simulados (nome, CPF, endereço) mascarados antes da camada analítica |
| Qualidade de dados | Testes automáticos no dbt bloqueiam dados inválidos |
| Governança | Catálogo, dicionário de dados e linhagem no Unity Catalog (Databricks) |
| CI/CD | GitHub Actions executa os testes a cada alteração |
| Documentação | Arquitetura, dicionário de dados e guia de execução neste repositório |

## Critério de sucesso

O painel responde às 5 perguntas P1 e mostra o risco de travamento de cada equipamento, com dados atualizados automaticamente pelo orquestrador, e o projeto roda do zero em outra máquina seguindo este README.

## Stack planejada

Docker · Python · MinIO · PostgreSQL · dbt · Apache Airflow · Databricks · MLflow · scikit-learn · Metabase · GitHub Actions

## Roadmap

- [x] Fase 0: problema e requisitos
- [ ] Fase 1: arquitetura
- [ ] Fase 2: ambiente com Docker
- [ ] Fase 3: dados simulados
- [ ] Fase 4: ingestão (camada bronze)
- [ ] Fase 5: transformação com dbt (silver e gold)
- [ ] Fase 6: orquestração com Airflow
- [ ] Capítulo Databricks: pipeline em PySpark e Delta Lake
- [ ] Fase 7: painel
- [ ] Fase 8: manutenção preditiva com machine learning
- [ ] Fase 9: documentação final e publicação

## Autor

Tiago Fernandes · [GitHub](https://github.com/TiagoMFernandes)
