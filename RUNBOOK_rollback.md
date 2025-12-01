# RUNBOOK de Rollback — BackupMonitor API

## Objetivo

Definir um passo a passo **curto, reversível e testável** para reverter uma release
da BackupMonitor API quando os gatilhos de erro forem atingidos.

## Gatilhos para iniciar rollback

Acione este runbook quando, na release canário:

- Taxa de erros 5xx > 2% por 5 minutos, OU
- p95 de latência > 600ms por 5 minutos, OU
- Queda brusca na taxa de jobs de backup concluídos com sucesso, OU
- Alertas críticos de integridade de dados ou autenticação.

## Passo a passo

1. **Congelar aumento de tráfego para a versão canário**
   - Pausar qualquer etapa da pipeline que esteja aumentando o percentual de tráfego.
   - Manter canário no percentual atual (ex.: 10%).

2. **Redirecionar tráfego de volta para a versão estável**
   - Atualizar configuração do balanceador de carga para enviar 100% do tráfego
     para a última versão estável (ex.: `backupmonitor-api:vX.Y.Z`).
   - Se estiver usando feature flags, desativar as flags relacionadas à nova funcionalidade.

3. **Verificar saúde após rollback**
   - Monitorar por pelo menos **10 minutos**:
     - taxa de erros 5xx;
     - p95 de latência;
     - taxa de jobs de backup concluídos com sucesso.
   - Confirmar que os indicadores voltaram aos níveis normais.

4. **Comunicação**
   - Registrar no canal do time (ex.: Slack/Teams) que o rollback foi executado:
     - qual versão foi revertida;
     - horário;
     - sintomas observados.
   - Avisar stakeholders relevantes (professor/produto fictício).

5. **Registro para PDCA**
   - Criar item de retrospectiva com:
     - causa aparente do problema;
     - impacto;
     - decisão (rollback);
     - ações de follow-up (testes adicionais, refatoração, melhoria de monitoramento).

## Verificação periódica deste runbook

- A cada 2–3 sprints, revisar este documento durante a cerimônia de melhoria/retrospectiva.
- Atualizar conforme novas lições aprendidas em incidentes simulados.
