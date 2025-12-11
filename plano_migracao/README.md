# Plano de Migração e Estruturação de Arquitetura de Dados para Modelo Lakehouse

Este documento descreve o Plano de Migração da Arquitetura de Dados, abrangendo a transição do ambiente ETL monolítico em Python (legado) para a nova arquitetura ELT baseada em Airbyte, dbt, Airflow e BigQuery. O plano inclui estratégia de migração incremental, padrões arquiteturais, processos operacionais, governança, definição de camadas e organização do trabalho do time.

---

## O que eu faria a partir do momento em que assumisse essa arquitetura

### Entendimento do ambiente
- Mapear arquitetura atual, custos, criticidade de cada pipeline.
- Entender cultura, responsabilidades, habilidades do time e principais dores.
- Levantar todas as dependências entre pipelines, dashboards e sistemas.
- Mapear regras de negócio existentes nos Python scripts legados

### Avaliação técnica e riscos
- Avaliar performance e custo atual no BigQuery e no servidor legado.
- Identificar gargalos, riscos operacionais, modelos críticos e pipelines instáveis.
- Priorizar migração com base em impacto no negócio e esforço de reconstrução.

---

## Definição da plataforma alvo

### Arquitetura Lakehouse com modelo medalhão
- **Landing**: Onde os dados chegam brutos exatamente como vêm da fonte.
- **Bronze**: Ingestão padronizada: normaliza formatos, adiciona metadata, mas ainda sem grandes tratamentos.
- **Silver**: Transformações validadas: limpeza, deduplicação, tipagem correta, joins; dados prontos para análise.
- **Gold**: Camada de negócio: métricas, agregações, KPIs, modelos analíticos, dados prontos para consumo final.

---

## Ferramentas
- Airbyte para ingestão
- dbt para transformações, testes, lineage e documentação
- Airflow para orquestração
- DataHub para catálogo, governança, lineage e documentação
- Grafana observabilidade
- Git + CI/CD para versionamento e deploy

---

## Como conduziria a migração do legado para o novo modelo

### Estratégia de migração incremental – ETL - ELT
- Novo ambiente isolado do legado, novos projetos já nascem no modelo ELT.
- Cada pipeline com data de corte.
- Execução paralela até validação final.

### Processo padrão de migração
- Mapear lógica de extração, transformação e carga do legado.
- Levar extração para o Airbyte  - dados brutos no Landing.
- Reescrever transformações no dbt (bronze/silver/gold).
- Criar DAG no Airflow (novo ambiente).

### Executar pipelines em paralelo e comparar
- Métricas.
- Registros.  
- Regras de negócio.  
- Validação com BI → apontar Looker para novo dataset.  
- Desligar pipeline legado.

### Garantia contra duplicação de regras de negócio
- Validação com área de negócio.
- Conferir que nenhuma lógica seja mantida no Python após migração.

### Convivência temporária entre cron e Airflow
- Cron continua apenas para pipelines não migrados.
- Liga de um lado, desliga de outro.

---

## Como resolveria os principais desafios técnicos, operacionais

### Governança e padronização
- DataHub para catálogo, lineage e documentação centralizada.
- dbt tests (not_null, unique, …..).
- Políticas de acesso baseadas em camadas e tabelas.
- Padrões de nomenclatura e organização por domínio.

### Observabilidade
- Monitoramento integrado (Airflow, Airbyte, dbt, Grafana).
- Alertas de falha.

---

## Como organizaria a arquitetura, processos e padrões

### Arquitetura
- Landing - Bronze - Silver - Gold, com responsabilidades claras.

### Processos
- Fluxo padrão de desenvolvimento:
- Ambientes separados (dev / prod).
- Processo para novas fontes no Airbyte e novos modelos no dbt.

---

## Como trabalharia com um time júnior

### Distribuição
- Juniores: documentações, testes dbt, modelos simples no Silver, pequenas correções.
- Níveis mais experientes: pipelines críticos e arquitetura.

### Garantia de qualidade
- PR obrigatório.
- Automação: CI / CD, testes dbt.
- Templates de PR com checklist técnico.

### Escalabilidade
- Frameworks reutilizáveis: macros dbt, DAGs padrão, templates de ingestão.
- Reduzir variação entre pipelines para permitir autonomia segura.

---

## Decisões
- Arquitetura medalhão → padronização e governança.
- Transformações centralizadas no dbt → evita duplicação e facilita testes.
- Airflow como orquestrador único → reduz risco do cron e integra todo o fluxo.
- DataHub → governança, lineage e documentação vivos.
- CI/CD → garante qualidade mesmo com time júnior.
- Migração incremental → menor risco e rollback fácil.
- Cultura de documentação → escalabilidade, previsibilidade e clareza.
