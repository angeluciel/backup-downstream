# BackupMonitor API — Pull Request

## Descrição

<!-- Explique o que foi feito e o porquê. Ex.: 
   - Adiciona endpoint para listar jobs de backup com falha nas últimas 24h
   - Ajusta cálculo de taxa de falha por cliente
-->

## Tipo de mudança

- [ ] feat (nova funcionalidade)
- [ ] fix (correção de bug)
- [ ] chore (ajuste interno / infra)
- [ ] docs (documentação)
- [ ] test (testes)

## Checklist — Definition of Done (DoD)

- [ ] Mudança relacionada a uma história/issue em DoR.
- [ ] Código compilando localmente.
- [ ] Testes unitários relevantes criados/atualizados.
- [ ] Nenhum teste quebrado.
- [ ] Logs revisados (nível adequado, uso de `request_id`).
- [ ] Documentação mínima atualizada (se aplicável).
- [ ] Plano simples de rollback descrito, se mudança for sensível.

## Quality Gates (antes do merge)

- [ ] Pipeline de CI **verde**.
- [ ] Lint sem erros.
- [ ] Cobertura global ≥ 70%.
- [ ] SAST sem vulnerabilidades HIGH/CRITICAL pendentes.
- [ ] Pelo menos 1 revisão/aprovação de outra pessoa do time.

## Risco & Rollback

- **Impacto da mudança:** <!-- baixo / médio / alto -->
- **Plano de rollback:** <!-- Ex.: reverter deploy para versão X.Y.Z; desativar feature flag XYZ -->