# Semana 03 — Defesa arquitetural e ADRs

**Estudo de caso (parte 2):** refinar requisitos não funcionais, formalizar decisões arquiteturais em ADRs e defendê-las em linguagem de negócio. O material é teórico; não contém implementação.

## Premissas do estudo

Os três produtos são cenários fictícios. Não foram informados volumes absolutos, SLOs, orçamento, tamanho/experiência da equipe, regiões de operação ou requisitos regulatórios específicos. Por isso, prioridades são comparativas e as metas quantitativas ficam **a definir**. As decisões aceitas abaixo valem para esse cenário acadêmico; não são prescrições universais.

## 1. Requisitos não funcionais refinados e prioridade

| Sistema | Prioridade | RNF | Qualidade relacionada na ISO/IEC 25010 | Como avaliar (sem inventar metas) |
|---|---:|---|---|---|
| E-commerce | P0 | Escalar elasticamente em picos e manter disponibilidade do fluxo essencial de compra. | Eficiência de desempenho (capacidade/comportamento temporal) e confiabilidade (disponibilidade). | Testes de carga em catálogo/checkout, taxa de sucesso, latência e recuperação; valores-alvo **a definir**. |
| E-commerce | P1 | Conter falhas parciais; manter integridade de pagamento/estoque e consistência eventual apenas onde aceitável. | Confiabilidade; segurança (integridade); eficiência de desempenho. | Testes de falha, idempotência, reconciliação de pedidos e atraso de eventos; limites **a definir**. |
| App bancário | P0 | Proteger dados e operações; restringir acesso; manter trilha auditável e integridade dos lançamentos. | Segurança (confidencialidade, integridade, autenticidade, responsabilização/rastreabilidade). | Revisão de controles, testes de autorização, trilhas de auditoria e resposta a incidentes; critérios regulatórios **a definir**. |
| App bancário | P0 | Fazer transferências de forma atômica e consistente, sem saldo/lançamento parcial. | Confiabilidade e adequação funcional (correção); propriedade transacional ACID como requisito técnico. | Testes concorrentes, invariantes de saldo e recuperação; volume e latência aceitáveis **a definir**. |
| App bancário | P1 | Manter disponibilidade e recuperar operações sem duplicidade. | Confiabilidade (disponibilidade, tolerância a falhas, recuperabilidade). | Exercícios de recuperação e reconciliação; RTO/RPO **a definir**. |
| Jogo multiplayer | P0 | Manter baixa latência e atualizações frequentes para partidas responsivas. | Eficiência de desempenho (comportamento temporal/capacidade). | Medir latência por região, jitter, perda de pacotes e taxa de atualização; valores-alvo dependem do gênero e estão **a definir**. |
| Jogo multiplayer | P0 | Sincronizar estado e impedir que clientes imponham resultados inválidos. | Confiabilidade e segurança (integridade/autenticidade). | Testes de divergência, reconexão e inputs inválidos; política de tolerância **a definir**. |
| Jogo multiplayer | P1 | Admitir aumento de partidas simultâneas e conter falha de uma sessão/região. | Eficiência de desempenho e confiabilidade (disponibilidade/tolerância a falhas). | Testes com sessões simultâneas e falhas regionais; capacidade-alvo **a definir**. |

A ISO/IEC 25010 é um modelo de qualidade, não uma lista de metas automáticas. Nesta análise, **desempenho** é associado à característica *performance efficiency*; **disponibilidade** é subcaracterística de *reliability*; e segurança inclui dimensões como confidencialidade, integridade, autenticidade e responsabilização. A consistência ACID é propriedade técnica desejada, não o nome de uma característica ISO.

## 2. Decisões finais resumidas

1. **E-commerce:** microsserviços por domínio com eventos assíncronos seletivos; isolar escala por serviço e amortecer picos, aceitando complexidade de operação e consistência distribuída.
2. **App bancário:** monólito modular no núcleo transacional; preservar uma fronteira ACID controlada e módulos auditáveis, aceitando escala independente limitada e disciplina de modularidade.
3. **Jogo multiplayer:** cliente-servidor com servidor de partida autoritativo e transporte em tempo real escolhido conforme o gênero; controlar o estado e a latência por sessão/região, aceitando custo de infraestrutura e capacidade finita por servidor.

Os ADRs individuais, com status **Aceito**, alternativas consideradas e consequências, estão em [`adr/`](./adr/).

## 3. C4 — contexto do jogo multiplayer

O diagrama [`diagramas/c4-jogo-multiplayer.svg`](./diagramas/c4-jogo-multiplayer.svg) é uma **visão C4 de containers (nível 2)** da Plataforma do Jogo, com o jogador, provedor externo de identidade e relações entre containers. É conceitual; protocolos e limites são ilustrativos.

## 4. Simulação de crise — crescer 10× em um mês

“10×” é um cenário de estresse, não uma garantia. Sem medições de base e testes, não podemos prometer que a arquitetura absorverá esse crescimento sem degradação.

- **E-commerce:** escalar Catálogo e leitura de forma independente; cachear leituras seguras; desacoplar trabalho secundário por filas/eventos; limitar chamadas e aplicar retentativas/idempotência. Pagamentos e estoque exigem controles contra duplicidade. Se a criação de instâncias ou o banco virarem gargalos, aplicar fila/controle de admissão e comunicar espera; operar benchmarks e capacidade reservada antes da campanha.
- **App bancário:** primeiro preservar consistência e segurança, não aceitar escrita sem controle para aparentar disponibilidade. Dimensionar a camada de aplicação e leituras separadamente, medir o banco transacional e proteger operações com limites/rate limiting. Se o núcleo não acompanhar 10×, enfileirar somente ações cujo contrato permita, reduzir funções não essenciais e apresentar a limitação; não dividir saldos entre serviços sem redesenhar a fronteira contábil.
- **Jogo:** aumentar servidores de partida por sessão/região e distribuir matchmaking; limitar vagas por instância e escalar com antecedência. Como provisionamento pode ser mais lento que o pico, manter headroom, fila/espera e comunicar disponibilidade. Se faltar capacidade, preservar partidas ativas em vez de sobrecarregá-las; medir latência, saturação e custo por sessão.

## 5. Reflexão

Um diagrama mostra blocos e relações; o ADR preserva o contexto, as alternativas, o motivo da decisão e o custo aceito. Sem esse “porquê”, uma equipe futura pode repetir uma discussão já resolvida ou remover um limite que protegia um requisito importante. O ADR transforma decisão em conhecimento verificável, inclusive quando a decisão precisa ser revista.

Para defender a arquitetura a alguém focado em dinheiro e prazo, traduza atributos em efeito de negócio: “isolamos catálogo e checkout para aumentar capacidade na campanha sem dimensionar todos os serviços pelo pico do catálogo; em troca, financiamos monitoramento e operação distribuída”. Para o banco: “mantemos transferências numa transação controlada para reduzir risco de inconsistência, mesmo que isso limite escala independente”. Para o jogo: “servidores por região ajudam a controlar a latência; isso tem custo de infraestrutura por partida”. A escolha deve comparar investimento, risco e impacto, sem prometer resultado que não foi medido.

## Apresentação oral

Os slides em [`slides/apresentacao-semana-03.pdf`](./slides/apresentacao-semana-03.pdf) apresentam prioridades, decisões, consequências, um C4 de contexto e a crise 10×. Use-os para defender os efeitos no negócio; nomes dos integrantes e duração ficam **a definir**.
