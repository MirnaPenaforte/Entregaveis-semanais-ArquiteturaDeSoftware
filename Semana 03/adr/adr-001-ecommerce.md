# ADR-001 — E-commerce: microsserviços por domínio com eventos seletivos

- **Status:** Aceito (para o cenário acadêmico; revisar com métricas, equipe e orçamento reais)
- **Data:** 2026-11-16

## Contexto

O e-commerce fictício precisa lidar com picos sazonais, disponibilidade do fluxo de compra e componentes com cargas diferentes (por exemplo, catálogo e checkout). É importante conter falhas parciais e manter a integridade de pagamentos e estoque. Não há metas de volume/SLO nem estrutura operacional definidas.

## Decisão

Separar Catálogo, Pedidos, Pagamentos e Estoque em serviços de domínio, com APIs síncronas para operações que precisam de resposta imediata e eventos/fila para etapas desacopláveis. Tratar pagamentos com idempotência, tentativas controladas e reconciliação; usar consistência eventual somente em partes que tolerem atraso. A adoção pressupõe automação de implantação e observabilidade distribuída.

## Alternativas consideradas

- **Monólito modular:** mais simples de desenvolver e operar inicialmente; menos capacidade de escalar/deployar domínios independentemente.
- **Microsserviços para tudo e comunicação síncrona:** separação forte, mas amplia dependências em rede e risco de indisponibilidade em cascata.
- **Event-driven em todo fluxo:** desacopla, mas dificulta rastrear causalidade e não atende sozinho respostas imediatas de checkout.

## Consequências

### Benefícios

- Escalar catálogo, checkout ou processamento de pedidos segundo seus perfis de carga.
- Evolução/deploy por domínio com menor acoplamento organizacional.
- Eventos amortecem rajadas e desacoplam trabalho secundário.

### Custos e riscos aceitos

- Mais custo de plataforma, monitoramento, tracing, deploy, segurança entre serviços e suporte operacional.
- Falhas de rede, duplicidade e atrasos exigem timeout, retentativa limitada, idempotência, DLQ/reprocessamento e reconciliação.
- Dados distribuídos não oferecem uma única transação ACID global; consistência eventual e compensações (saga) devem ser explícitas.
- A equipe e o orçamento necessários ainda precisam ser confirmados. Se não houver maturidade operacional, a decisão deve ser reavaliada em favor de monólito modular.

## Como verificar

Realizar teste de carga e falhas com metas acordadas antes de uma campanha; observar latência, erros, fila/event lag, saturação por serviço e duplicidade de cobrança/estoque. SLOs e limiares: **a definir**.
