# Blueprint do Downstream — BackupMonitor API

## 1. Contexto

A **BackupMonitor API** é um sistema que centraliza informações de jobs de backup
de diferentes clientes/ambientes, permitindo consultar status, falhas recentes
e tendência de saúde dos backups.

Este documento descreve **como o time leva mudanças em código até produção**
com qualidade, previsibilidade e maturidade leve, conectando:

- Git + Pull Requests
- Testes + lint + SAST
- CI/CD com quality gates
- Ambientes (dev, staging, prod)
- Releases seguras + rollback
- Observabilidade + métricas DORA + PDCA/Kaizen


## 2. Fluxo de Git & Pull Requests

### 2.1 Estratégia de Branch

- Branch principal: ` main `
- Branches curtas a partir de ` main `, com convenção:

  - `feat/<descricao-curta>` – nova funcionalidade  
    Ex.: `feat/relatorio-falhas-ultimas-24h`
  - `fix/<descricao-curta>` – correção  
    Ex.: `fix/calc-taxa-falha-backup`
  - `chore/<descricao-curta>` – ajustes não funcionais  
    Ex.: `chore/atualiza-dependencias`

- Regra: branch deve ser pequena o suficiente para ser revisada em até 15–20 minutos.

### 2.2 Convenção de Commits

Usaremos um padrão inspirado em **Conventional Commits**:

- `feat: ...` — nova funcionalidade
- `fix: ...` — correção
- `chore: ...` — ajustes internos
- `docs: ...` — documentação
- `test: ...` — testes

Exemplo:
```text
feat(backup-jobs): add endpoint to list failed jobs in the last 24 hours
```
### 2.3 Pull Request (PR)

Todo trabalho passa por PR e segue o template [PULL_REQUEST_TEMPLATE.md](PULL_REQUEST_TEMPLATE.md).

**Obrigatório para merge em main:**

- PR vinculado a uma issue/história do bakclog em DoR.
- Checklists de DoD preenchidos.
- Pipeline de CI verde.
- Pelo menos 1 revisão de outro membro do time.


## 3. Definition of Done (DoD) & Quality Gates

### 3.1 Definition of Done (DoD) do Time

Um item só é considerado **“done”** quando:

1. Código versionado em branch própria.
2. PR aberto com descrição clara do porquê e do o quê.
3. Testes unitários relevantes criados/atualizados.
4. Cobertura global ≥ 70% (meta inicial).
5. Lint sem erros bloqueantes.
6. SAST sem vulnerabilidades HIGH/CRITICAL pendentes.
7. Logs essenciais da nova funcionalidade revisados (correlation id, nível adequado).
8. Plano de rollback simples descrito no PR, se mudança for sensível.
9. PR revisado e aprovado.

### 3.2 Quality Gates

- CI falhou em qualquer estágio (install, lint, test, coverage, SAST, build) → sem merge.
- Cobertura < 70% → sem merge (limiar inicial; pode subir com maturidade).
- SAST encontrou vulnerabilidade HIGH/CRITICAL não tratada → sem merge.
- PR sem aprovação de pelo menos 1 revisor → sem merge.


## 4. Qualidade & Testes

### 4.1 Pirâmide de Testes

Para a BackupMonitor API:

- **Testes Unitários (base da pirâmide)**
    - Focam em funções como:
        - cálculo de taxa de falha de backup por cliente;
        - classificação de severidade de alerta (normal, warning, critical);
        - parsing de respostas de APIs de terceiros (ex.: Veeam, etc.).
    - Rápidos, sem acessar rede nem banco.
- **Testes de Integração (meio da pirâmide)**
  - Verificam a integração com:
    - banco de dados (consulta de jobs e estados);
    - fila de eventos (quando houver);
    - serviços internos de agregação de métricas.
  - Podem rodar em container de banco fake ou banco de teste.
- **Testes End-to-End (E2E) (topo da pirâmide)**
  - Poucos cenários críticos:
    - consulta de painel de status de backup de um cliente;
    - fluxo “novo job de backup falha → alerta aparece na API”.
  - Preferencialmente via testes de API (ex.: usando HTTP client automatizado).

### 4.2 Meta de Cobertura

- **Meta inicial: ≥ 70% de cobertura global.**
- **Arquivos críticos (ex.: cálculo de indicadores de falha) devem ter cobertura > 80%.**
- **Conforme o time ganha maturidade, a meta pode subir para 75–80%.**

### 4.3 Ferramentas de Qualidade
- **Lint**: ESLint (JavaScript/TypeScript)
- **Testes + Cobertura:** Jest
- **SAST**: CodeQL ou Semgrep

**Quando viram blocking**

- Lint com erro → falha o job de lint → bloqueia PR.
- SAST com HIGH/CRITICAL → falha o job de sast → bloqueia PR.
- Cobertura global < 70% → falha o job de coverage → bloqueia PR.

## 5. CI/CD & Ambientes

### 5.1 Pipeline Mínimo (CI)

[Pipeline](.github/workflows/ci.yml) com os estágios:

1. **install** – instalação de dependências.
2. **lint** – execução do ESLint.
3. **test** – execução dos testes unitários e de integração.
4. **coverage** – verificação se cobertura ≥ 70%.
5. **sast** – análise estática de segurança.
6. **build** - build da aplicação / geração de artefatos.

### 5.2 Ambientes

- **dev**
    - Uso principal: desenvolvimento diário.
    - Dados fictícios ou anonimizados.
    - Logs mais verbosos (nível debug permitido).
    - Flags mais livres para experimentos.
  - **staging**
    - Espelho de produção.
    - Testes E2E e validação fim a fim:
      - carga mínima (smoke/perf leve);
      - validação de migrações de banco.
    - A única forma de chegar aqui é via pipeline (sem deploy manual).
  - **prod**
    - Uso real pelos usuários.
    - Configurações e segredos por ambiente (ex.: variáveis de ambiente).
    - Deploy somente via pipeline aprovado, seguindo [a política de promoção](POLITICA_Promocao.md).

## 6. Releases Seguras & Rollback

### 6.1 Estratégia de Release

Adotaremos canary release:

- Nova versão da BackupMonitor API é liberada:
  - inicialmente para 10% do tráfego;
  - se estável por uma janela de tempo (ex.: 15–30 minutos);
  - então gradual aumento para 50% → 100%.

Para mudanças de alto risco, podemos usar também **feature flags** para ativar/desativar partes da funcionalidade sem novo deploy.

### 6.2 Gatilhos de Rollback

Acionam o [RUNBOOK_rollback.md](./RUNBOOK_rollback.md).
- Taxa de erros 5xx > 2% das requisições da canário por 5 minutos.
- p95 de latência HTTP > **600ms** por 5 minutos.
- Queda brusca no número de jobs de backup processados (indicando travamento).
- Aumento anormal em falhas de autenticação nos endpoints da API.

## 7. Observabilidade & DORA

### 7.1 Logs Essenciais

1. Log de requisição de API
   - Campos obrigatórios:
     - `request_id` (correlation id)
     - `cliente_id`
     - `rota`
     - `método`
     - `status_code`
     - `latência_ms`
2. Log de erro de processamento de backup
   - Campos obrigatórios:
     - `request_id`
     - `cliente_id`
     - `job_id`
     - `tipo_backup` (ex.: diário, semanal, mensal)
     - `erro` (mensagem)
     - `stack_trace` (quando aplicável)

### 7.2 Métricas Essenciais

1. Taxa de erros 5xx/minuto da API.
2. p95 de latência das requisições HTTP.
3. **Percentual de jobs de backup concluídos com sucesso por janela de tempo.**

### 7.3 Métricas DORA

- Lead time for changes
  - Medido do momento em que o PR é aberto até a release ir para produção.
- Change failure rate
  - `% de releases que precisam de rollback` / `total de releases`.

### 7.4 Rotina PDCA / Kaizen

- **Periodicidade**: quinzenal (a cada 2 semanas).
- **Participantes**: time de desenvolvimento + alguém de operação/infra (quando houver).
- **Passos**:
  1. **Plan** — escolher 1–2 problemas (ex.: alta taxa de rollback, testes demorados).
  2. **Do** — experimentar uma melhoria pequena (ex.: mais testes unitários para módulo crítico).
  3. **Check** — olhar métricas (DORA, erros, lead time) e incidentes.
  4. **Act** — padronizar o que funcionou (atualizar blueprint, runbooks, políticas).

## 8. Resumo

Este blueprint define como o time da BackupMonitor API organiza seu downstream:

- Branches curtas, PRs com DoD claro e quality gates.
- Pirâmide de testes com foco em unitários e integração, poucos E2E críticos.
- CI com stages mínimos e gates objetivos (lint, cobertura, SAST).
- Ambientes dev -> staging -> prod com promoção apenas via pipeline.
- Releases canário com gatilhos claros de rollback.
- Observabilidade enxuta (logs + métricas) e uso de métricas DORA para melhoria contínua.