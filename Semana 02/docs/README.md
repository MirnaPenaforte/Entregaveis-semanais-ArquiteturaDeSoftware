# Semana 02 — Escolha de estilos arquiteturais

**Estudo de caso teórico:** selecionar uma abordagem arquitetural para um e-commerce, um aplicativo bancário e um jogo multiplayer. As escolhas são propostas para os cenários fictícios descritos, não recomendações universais.

## 1. Sistemas e requisitos críticos

| Sistema | Contexto considerado | Requisitos não funcionais prioritários |
|---|---|---|
| E-commerce | Catálogo, carrinho, pedidos, pagamentos e promoções sazonais. | Escalar picos (por exemplo, Black Friday), tolerar falhas parciais, proteger pagamentos e manter consistência eventual aceitável em algumas leituras do catálogo. |
| App bancário | Consultas de saldo, transferências e histórico financeiro. | Segurança e privacidade elevadas, integridade e consistência forte nas operações financeiras, auditoria e rastreabilidade. |
| Jogo multiplayer | Partidas online com vários jogadores compartilhando uma sessão. | Baixa latência, atualização frequente do estado, sincronização entre clientes e autoridade contra estados inválidos/cheating. |

São premissas do estudo; volumes, orçamento, tamanho da equipe, stack e requisitos regulatórios específicos estão **a definir**.

## 2. Decisões arquiteturais

### E-commerce — microsserviços com eventos assíncronos onde adequado

**Decisão:** separar domínios como Catálogo, Pedidos, Pagamentos e Estoque em serviços com responsabilidades claras; usar eventos para propagar mudanças que não precisam bloquear a resposta síncrona do pedido.

**Justificativa técnica:** catálogo e checkout têm perfis de carga diferentes e podem precisar escalar separadamente. Eventos ajudam a desacoplar etapas, como reservar estoque após criação do pedido, e reduzem dependências síncronas em fluxos secundários. Dados distribuídos exigem consistência eventual em partes do processo e mecanismos de compensação/retentativa (por exemplo, saga) quando uma etapa falha; pagamentos devem ser idempotentes para evitar cobranças duplicadas.

**Justificativa de negócio e trade-off:** equipes de domínio podem evoluir e implantar serviços com mais independência durante campanhas de pico. Em contrapartida, microserviços aumentam custos operacionais: observabilidade distribuída, automação de implantação, segurança entre serviços, tratamento de falhas de rede e maior demanda por equipe experiente. Se a organização for pequena, começar como monólito modular pode ser mais barato; a escolha aqui assume capacidade operacional suficiente, premissa a confirmar.

### App bancário — monólito modular para o núcleo transacional

**Decisão:** organizar o núcleo financeiro como monólito modular, com módulos de Contas, Transferências, Autenticação/Autorização e Auditoria, usando transações ACID nas operações que alteram saldos. Integrações externas podem ser adaptadores isolados; eventos podem alimentar notificações ou analytics depois do commit, não substituir a transação financeira.

**Justificativa técnica:** manter a atualização de origem/destino e os lançamentos contábeis sob uma fronteira transacional controlada simplifica consistência forte, validações e rastreabilidade. Módulos explícitos reduzem acoplamento lógico sem introduzir, desde o início, chamadas de rede entre cada parte crítica.

**Justificativa de negócio e trade-off:** favorece controle, auditoria e entrega inicial mais simples do que uma malha de serviços. Não elimina a necessidade de segurança, alta disponibilidade, redundância ou escalabilidade. Um monólito pode limitar escala independente e tornar mudanças arriscadas se os limites modulares forem ignorados; crescimento, autonomia de equipes ou requisitos de disponibilidade podem justificar separar serviços mais tarde, mantendo claras as fronteiras transacionais.

### Jogo multiplayer — cliente-servidor autoritativo com comunicação em tempo real

**Decisão:** usar servidores de partida autoritativos, com clientes enviando comandos/inputs e recebendo atualizações do estado. Usar um transporte de baixa latência adequado ao gênero para atualizações frequentes; WebSocket pode servir para lobby/chat ou jogos por turnos, mas não é uma decisão automática para todo tráfego de gameplay.

**Justificativa técnica:** a autoridade do servidor reduz divergências entre clientes e dificulta trapaças baseadas em declarar localmente um estado vencedor. Servidores por região/sessão ajudam a limitar latência percebida e permitem distribuir partidas. A sincronização deve prever perda/atraso de pacotes, interpolação/predição e reconexão, conforme o gênero e a tolerância a atraso.

**Justificativa de negócio e trade-off:** favorece partidas justas e experiência consistente para jogadores sem exigir que pares confiem uns nos outros. O custo é operar servidores de jogo, alocação regional, matchmaking e mecanismos de reconexão; o servidor pode ser gargalo. Peer-to-peer pode reduzir custo de servidor em alguns jogos, mas complica confiança, NAT, sincronização e tratamento do host que sai. O gênero, número de jogadores e orçamento estão **a definir**.

## 3. Diagrama bônus — jogo multiplayer

O diagrama de blocos está em [`diagramas/jogo-multiplayer.svg`](../diagramas/jogo-multiplayer.svg). Ele mostra clientes conectados ao matchmaking e a um servidor de partida autoritativo; o serviço de contas/autorização apoia o acesso. É um modelo conceitual, não um desenho de implantação completa.

## 4. Reflexão — não existe uma “bala de prata”

Nenhum estilo arquitetural atende melhor a todos os problemas porque cada um otimiza qualidades diferentes e impõe custos. Microsserviços podem permitir escala e implantação independente, mas exigem automação, monitoramento e tolerância a falhas de rede. Um monólito modular pode ser mais simples e barato para iniciar, mas requer disciplina para preservar módulos e pode dificultar escalabilidade ou autonomia quando cresce. Cliente-servidor facilita autoridade e controle em partidas, mas custa infraestrutura de servidor e precisa de estratégia para latência.

Orçamento determina quanto a organização consegue investir em infraestrutura e operação; experiência e tamanho da equipe limitam a complexidade que pode ser sustentada; prazo de mercado favorece soluções mais simples no início, desde que não violem requisitos críticos. Por isso, a escolha deve partir do contexto e dos trade-offs, ser registrada como hipótese e revista quando negócio, escala ou equipe mudarem — não seguir uma tendência por moda.

## 5. Apresentação oral

O PDF em [`slides/apresentacao-semana-02.pdf`](../slides/apresentacao-semana-02.pdf) resume os cenários, decisões e trade-offs. Os slides são apoio visual: apresentem a justificativa com palavras próprias, não leiam o texto integralmente. Integrantes do grupo e duração da apresentação não foram informados e ficam **a definir**.

## Arquivos

- `README.md` — estudo de caso e justificativas.
- `slides/apresentacao-semana-02.pdf` — slides para apresentação oral.
- `slides/apresentacao.html` — fonte HTML dos slides.
- `diagramas/jogo-multiplayer.svg` — diagrama de blocos bônus.
