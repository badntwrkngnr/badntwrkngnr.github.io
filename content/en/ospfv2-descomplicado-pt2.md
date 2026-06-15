---
title: "x"
slug: "x" 
date: 2026-05-23
translationKey: "x"
categories: ["nx"]
math: true
draft: true
---

## Introdução

## Troubleshooting

Nesta fase, a abordagem muda da configuração para a análise do *control-plane*. O isolamento de falhas OSPF costuma dividir-se em: falha na formação de vizinhança e problemas com LSAs ou tabelas de roteamento.

### Comandos de Verificação

Durante auditorias ou exames práticos, o isolamento de anomalias depende dos comandos listados abaixo para diagnosticar as estruturas internas do OSPF:

- `show ip ospf neighbor` — A primeira linha de defesa. Analise o *State* (deve estar *Full* ou *2-Way*). Qualquer estado intermediário prolongado (Init, ExStart) denuncia falhas de negociação.
- `show ip ospf interface [brief]` — Audita as redes participantes, verifica os *Timers* (Hello/Dead), a máscara de rede embutida na interface e o *Network Type* ativado na porta.
- `show ip route ospf` — Inspeciona se as rotas processadas pelo algoritmo de Dijkstra chegaram efetivamente à RIB e valida a preferência de caminhos (O, O IA, O E1, O E2).
- `show ip ospf database` — Permite abrir o "cérebro" do protocolo. Mostra a LSDB, expondo quem originou cada LSA (Router Link, Net Link, Summary Net Link) para rastrear o caminho exato e identificar filtros inesperados.
- `show ip ospf` — Fornece estatísticas cruciais do processo global, revelando a quantidade de execuções do algoritmo SPF, que aponta se a rede está instável (flapping).
- `clear ip ospf process` — Executa um reset pesado do protocolo, limpando a LSDB e renegociando vizinhos do zero; essencial após alterar variáveis amarradas ao processo como o Router ID.
- `debug ip ospf adj` / `debug ip ospf hello` — Para dissecção em tempo real. Essencial para ler nas entrelinhas falhas silenciosas de MTU, senhas incompatíveis ou falhas de *flags* nas mensagens de descoberta.

### Problemas Comuns e Localização de Falhas

**1. Vizinhança Travada (Problemas de Control-Plane):**

- **Travado em INIT:** Indica comunicação unidirecional. Um roteador ouve os Hellos, mas seu Router ID não consta na lista de vizinhos ativos do pacote recebido. Causas: ACLs bloqueando multicast (224.0.0.5), falhas L2 ou divergências rigorosas de parâmetros.

- **Mismatched Timers / Subnets:** Hellos, Dead intervals e a subnet mask devem ser rigorosamente idênticos. O teste prático para isso é o `debug ip ospf hello`, que fará *flash* na tela alertando sobre parâmetros incompatíveis.

- **Travado em EXSTART / EXCHANGE (MTU Mismatch):** Um dos problemas operacionais mais clássicos. O OSPF recusa-se a prosseguir na negociação de banco de dados se o tamanho do payload L2 não coincidir entre os pares. O processo trava na tentativa de enviar as mensagens DBD. Teste e valide sempre verificando o tamanho do IP MTU em ambas as pontas da interface física ou túnel.

- **Falha de Area Type (Stub Flag):** Se um roteador tem a área configurada como *Stub* e o outro como *Normal*, a flag nos pacotes Hello diverge, e a adjacência é descartada imediatamente.

**2. Manipulação de Rotas e Distribuição (Problemas de Data-Plane / LSA):**

- **Roteamento Assimétrico em Redistribuição:** Redistribuir informações (de EIGRP ou BGP para o OSPF) exige extrema atenção à `seed-metric` e aos LSAs Tipo 5 ou 7. A injeção dupla sem ajustes de custo leva o algoritmo a rotear por caminhos subótimos. As rotas O E1 (somam métricas internas) e O E2 (métrica estática inalterada) possuem forte hierarquia na seleção de caminhos.

- **Rotas Fantasmas ou Discontiguous Area 0:** Uma interrupção lógica na Área 0 particiona a rede, impedindo que LSAs Tipo 3 cruzem de um lado para o outro. Testes de falha geralmente apontam rotas Inter-area sumindo misteriosamente da RIB. A solução emergencial, mas provisória, é configurar um *Virtual Link* transitando por uma área de suporte.

- **Supressão de Rotas (Filtering):** Falhas causadas pelo uso de comandos como `area range not-advertise` em um ABR, que engolem silenciosamente os LSAs Tipo 3. O troubleshooting deve começar analisando a `show ip ospf database summary` no ABR originador.

O diagnóstico reativo via linha de comando deve ser sistemático, mas a inteligência operacional se alcança com monitoramento proativo. Exportar o estado das adjacências OSPF, contadores de SPF runs e mudanças bruscas de métricas para ferramentas de observabilidade — alimentando dashboards de alta visibilidade no Grafana via Zabbix — retira o analista da postura apagadora de incêndios e permite a detecção de *flaps* no OSPF antes que virem um incidente perceptível de conectividade.
