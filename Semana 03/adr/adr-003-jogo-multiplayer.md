# ADR-003 — Jogo multiplayer: cliente-servidor autoritativo

- **Status:** Aceito (para o cenário acadêmico; validar com gênero e testes de rede)
- **Data:** 2026-11-16

## Contexto

O jogo fictício mantém sessões com vários participantes. As prioridades são comportamento temporal rápido, sincronização de estado e resistência a ações inválidas de clientes. Gênero, número de jogadores por sessão, regiões, orçamento de infraestrutura e latência-alvo estão **a definir**.

## Decisão

Usar modelo cliente-servidor com servidor de jogo autoritativo por sessão/região. Clientes enviam inputs e recebem o estado validado. Escolher o transporte segundo o gênero e a frequência dos updates: protocolo de baixa latência para o estado frequente quando necessário; HTTPS/WebSocket podem atender autenticação, lobby, matchmaking ou chat, sem assumir que WebSocket seja ideal para todo tráfego de gameplay. Usar matchmaking para alocar sessões e prever reconexão, sincronização e proteção contra spam/inputs inválidos.

## Alternativas consideradas

- **Peer-to-peer:** pode reduzir infraestrutura de servidor, mas amplia desafios de confiança, NAT, sincronização e saída do host.
- **Servidor autoritativo com apenas chamadas HTTP síncronas:** simples de compreender, mas inadequado para atualizações frequentes de uma partida em tempo real.
- **Servidor autoritativo com transporte especializado:** mantém autoridade, mas exige engenharia de rede e operação mais especializadas.

## Consequências

### Benefícios

- Estado da partida validado centralmente, reduzindo divergências e alguns vetores de trapaça no cliente.
- Sessões isoladas podem ser alocadas por região e distribuídas horizontalmente.
- Matchmaking e servidores permitem controlar admissão e capacidade por partida.

### Custos e riscos aceitos

- Custos contínuos de computação, rede, regiões, matchmaking e observabilidade.
- Cada servidor de partida tem limite de CPU/rede; sobrecarga aumenta latência ou afeta a sessão.
- Latência de rede não desaparece: predição/interpolação, jitter, perda de pacotes e reconexão devem ser tratados.
- Picos maiores que a capacidade provisionada podem exigir fila/espera; escala automática pode não acompanhar o crescimento instantâneo.

## Como verificar

Testar partidas simultâneas e falhas por região; medir latência/jitter/perda e custo por sessão; exercitar reconexão e inputs maliciosos. Metas numéricas: **a definir conforme gênero**.
