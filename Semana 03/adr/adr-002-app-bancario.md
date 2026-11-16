# ADR-002 — App bancário: monólito modular no núcleo transacional

- **Status:** Aceito (para o cenário acadêmico; revisar diante de requisitos/regulação e escala reais)
- **Data:** 2026-11-16

## Contexto

O app bancário fictício realiza consultas e transferências. Os requisitos prioritários são segurança, trilha de auditoria e consistência forte: uma transferência não deve deixar débito sem crédito correspondente. O volume, stack, regulação aplicável, RTO/RPO e distribuição das equipes estão **a definir**.

## Decisão

Manter o núcleo de contas e transferências como monólito modular com limites explícitos para Contas, Transferências, Autenticação/Autorização e Auditoria. Executar a atualização dos lançamentos financeiros dentro de transações ACID numa fronteira de persistência controlada. Usar adaptadores isolados para integrações e publicar eventos de notificação/analytics somente após o commit; eventos não substituem o registro transacional.

## Alternativas consideradas

- **Microsserviços para cada módulo:** autonomia e escala independente, mas podem exigir transação distribuída para uma transferência e aumentam custo de segurança/rede/operação.
- **Monólito sem módulos explícitos:** simples no início, mas aumenta acoplamento e risco de mudança.
- **Event-driven para o núcleo financeiro:** bom desacoplamento para tarefas assíncronas, porém não é substituto de confirmação atômica de uma transferência.

## Consequências

### Benefícios

- Fronteira ACID mais simples para regras de saldo e lançamentos relacionados.
- Auditoria e políticas de autorização podem ser aplicadas de forma consistente no núcleo.
- Menos componentes distribuídos para operar no início; módulos podem evoluir sem exigir chamadas de rede internas.

### Custos e riscos aceitos

- Escala e implantação menos independentes por domínio; mudanças podem demandar coordenação e implantação conjunta.
- O processo de build/deploy pode crescer; sem testes e limites modulares, o monólito pode degradar em acoplamento.
- Alta disponibilidade, segurança, backups, replicação e recuperação continuam obrigatórios; “monólito” não os fornece automaticamente.
- Dividir depois exige fronteiras transacionais e contábeis cuidadosamente preservadas.

## Como verificar

Testar concorrência, invariantes contábeis, autorização, auditoria, restauração e failover. Acordar volume, latência, RTO/RPO e controles regulatórios antes da implantação; metas: **a definir**.
