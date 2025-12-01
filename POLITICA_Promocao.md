# Política de Promoção — BackupMonitor API

Este documento define critérios **claros e objetivos** para promover versões da
BackupMonitor API entre os ambientes **dev → staging → prod**.

## 1. Promoção de dev → staging

Uma versão da BackupMonitor API pode ser promovida de **dev** para **staging** quando:

1. **CI na branch de integração (ex.: `develop` ou PR para `main`) está verde**:
   - install ✅
   - lint ✅
   - test ✅
   - coverage (≥ 70%) ✅
   - sast (sem HIGH/CRITICAL) ✅
   - build ✅

2. **Definition of Done cumprida** para as histórias incluídas na release:
   - testes unitários e de integração relevantes criados/atualizados;
   - logs essenciais revisados;
   - documentação mínima atualizada (se aplicável).

3. **PR revisado e aprovado** por pelo menos 1 membro do time.

4. **Migrações de banco** revisadas tecnicamente e aplicadas em dev sem erro.

## 2. Promoção de staging → produção (prod)

Uma versão pode ser promovida de **staging** para **prod** quando:

1. **Smoke tests e E2E críticos em staging passaram**:
   - consulta de status de backup por cliente;
   - listagem de jobs com falha;
   - geração de indicador de taxa de falha.

2. **Monitoramento em staging está estável** por uma janela mínima (ex.: 30 minutos):
   - taxa de erros 5xx dentro do esperado;
   - p95 de latência dentro dos limites esperados.

3. **Plano de rollout definido**:
   - percentual inicial de tráfego da canário (ex.: 10%);
   - critérios de aumento de tráfego;
   - gatilhos de rollback (conforme RUNBOOK_rollback.md).

4. **Plano de rollback validado**
   - versão anterior disponível como imagem/artefato (`backupmonitor-api:vX.Y.Z`);
   - configurações de balanceamento prontas para redirecionar tráfego.

5. **Aprovação de produto/PO (fictício)**
   - confirmação de que as mudanças fazem sentido do ponto de vista funcional
     e de valor para o usuário.


## 3. Janelas de Mudança e Freeze

- Deploys para produção devem ocorrer em **janelas combinadas** com o time
  (ex.: dias úteis em horário comercial).
- Períodos de **change freeze** podem ser definidos (ex.: véspera de grandes eventos)
  onde só correções críticas são liberadas.


## 4. Evidências de Promoção

Toda promoção de **staging → prod** deve registrar:

- Link da pipeline de deploy bem-sucedida.
- Versão/tag da imagem implantada.
- Prints ou links de dashboards de monitoramento durante a janela de observação.
- Confirmação (texto curto) de que os critérios desta política foram atendidos.

## 5. Revisão desta política

- Periodicidade: a cada 2–3 sprints, dentro da cerimônia de melhoria/retrospectiva.
- Objetivo: simplificar quando possível e tornar critérios mais objetivos
  à medida que a maturidade do time aumenta.
