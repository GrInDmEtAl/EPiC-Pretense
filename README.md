# EPiC-Pretense
Pretense Changelog only for server EPiC

# Changelog - Melhorias na IA Estratégica

Todas as mudanças notáveis no sistema da IA Estratégica do Pretense serão documentadas neste arquivo.

## [2026-03-25]

### Corrigido
- **TaskExtensions (AA/AG Default)**: Hardening em `setDefaultAA` e `setDefaultAG` com validações de `group/controller` antes de aplicar opções da IA, prevenindo erros por referência nula durante retask/spawn/despawn.
- **GroupMonitor (Convoys de Suprimento)**: Limpeza de `supplySpawners` ao remover/despawnar grupos, evitando acúmulo de referências órfãs.
- **Event Handlers (Framework Core)**: Isolamento de erro em handlers registrados no `world.addEventHandler` com `xpcall` e log contextual por subsistema (`CSARTracker`, `GroupMonitor`, `JTAC`, `MarkerCommands`, `MenuRegistry`, `MissionTracker`, `PlayerLogistics`, `PlayerTracker`, `ZoneCommand`), evitando que exceções pontuais interrompam fluxo de eventos.
- **IADS (`isSAM`)**: Correção dos padrões de detecção por nome com fronteiras léxicas para eliminar falso positivo por substring (ex.: `motor` acionando `tor`) no `IADS_CFG.lua`.
- **Offmap Supply (rota/seleção)**:
  - Correção da lógica de pouso em `landAtAirfield` para evitar perfil de voo que passava sobre o alvo e retornava ao ponto de origem.
  - Hardening no cálculo de distância de origem (`supplyPointRegistry`) para evitar erro com ponto nulo quando grupo de supply não está disponível no bootstrap.

### Melhorado
- **GroupMonitor (Convoys de Suprimento)**: Adicionado cooldown de retask de rota para comboios terrestres, reduzindo replanejamento excessivo e carga desnecessária no servidor.
- **Cameron (Ameaça SAM Radar)**: Fluxo de missão em presença de SAM radar mantém `SEAD` e `PATROL`, e preserva `ASSAULT` terrestre para continuidade de pressão no objetivo sem enviar CAS de forma suicida.
- **TemplateDB (init.lua)**: Refatoração incremental com helper `registerGroupTemplate(...)` para reduzir duplicação e padronizar definição de templates (SAM/EWR/defesa), melhorando manutenção.
- **SAM Sites (anti-HARM / imersão)**:
  - Ajuste de composição para maior redundância de sensores/lançadores e escolta SHORAD orgânica.
  - Aumento de dispersão (`maxDist`) em baterias de médio/longo alcance para reduzir vulnerabilidade a ataques SEAD/HARM em cadeia.
  - Remoção de dependências `CHAP_*` nos blocos SAM/EWR estratégicos, priorizando compatibilidade com assets base.
- **Doutrina de Defesa em Camadas**:
  - Introduzido `presets.defenseProfiles.red_defensive` e `presets.defenseProfiles.blue_mobile` para organizar `long/mid/short/sensor` sem quebrar chaves legadas.
  - Rebalanceamento de custo/display dos presets de defesa para refletir perfil RED mais defensivo e BLUE mais móvel.
- **Aproximação de Offmap Supply Configurável**:
  - Novos parâmetros `Config.offmapApproachMin` e `Config.offmapApproachMax` para controlar a distância de aproximação final (padrão 8-12 km) e reduzir colisões em descida tardia próximo da zona.
- **Performance (carga de loop/log)**:
  - `MissionTracker:tallyWeapon` com polling reduzido na fase terminal de bombas (`0.01 -> 0.05` e `0.1 -> 0.2`) para diminuir pressão de timers em ataques massivos.
  - `GCI` com intervalos configuráveis (`Config.gciRadarRefreshInterval`, `Config.gciReportInterval`) e desaceleração automática quando não há jogadores registrados no GCI.
  - Logs de telemetria quente em `GCI` e `GroupMonitor` agora sob flags de debug (`Config.debugGCI` e `Config.debugAI`), reduzindo I/O de log em servidor dedicado.
- **Tick Adaptativo da IA Terrestre (dedicated)**:
  - `GroupMonitor` agora desacelera grupos terrestres `enroute` que estão longe de jogadores e fora da frente ativa, mantendo tick normal para estados críticos.
  - Novas configs:
    - `Config.groundAdaptiveTickEnabled`: ativa/desativa o modo adaptativo.
    - `Config.groundActivePlayerDistance`: distância (m) para considerar grupo "ativo" por proximidade de player.
    - `Config.groundActiveFrontDistMax`: zonas com `distToFront` até este valor permanecem em tick normal.
    - `Config.groundSleepInterval`: intervalo (s) usado para grupos em sleep mode.
- **SAVE (autosave dedicado, configurável)**:
  - Autosave da missão agora usa intervalo configurável por `Config.persistenceSaveInterval` (padrão 60s).
  - `PersistenceManager` ganhou proteção de intervalo mínimo entre gravações efetivas (`Config.persistenceMinSaveGap`, padrão 30s) para evitar rajadas de escrita em disco.
  - Log `Mission state saved` só é emitido quando a gravação realmente ocorre.

---

## [2026-03-24]

### Adicionado
- **TaskExtensions**: Melhoria na lógica de pouso de helicópteros com cálculo de aproximação (offset de 3km) para aumentar a taxa de sucesso de pouso na zona e proteção contra divisão por zero em aproximações verticais (Grindmetal).
- **PlayerTracker**: Verificação e hardening contra crashes no evento de pouso (`isLanded` nil-guard para zonas).
- **PersistenceManager**: Hardening no carregamento de tabelas JSON com proteção `pcall` no `JSON:decode` contra arquivos de save corrompidos.

### Alterado
- **MIST Framework**: Atualizado para a versão `mist_128-DYNSLOTS-02.lua` visando maior estabilidade do sistema de slots dinâmicos.
- **Logística do Cameron**: Documentação da preferência por comboios terrestres por custo-benefício (4000 vs 2500 cap, 10 vs 20 cost) e alcance ilimitado via conexões de zona.

### Melhorado
- **Controle de GCI**: Refinamento do `gciGhostTime` para controle de persistência de alvos no radar pós-perda de sinal.
- **Balanceamento Estratégico**: Tunning refinado dos parâmetros `lossCompensation`, `randomBoost` e `buildSpeed` para ajuste dinâmico da dificuldade PvE conforme controle de zona.

---

## [2026-03-21]


### Adicionado
- **Logística do C-130J-30 (Módulo Oficial ASC)**: Suporte para detecção automática de portas abertas (argumentos 38, 86, 87, 88), permitindo carregamento/descarregamento de carga e suprimentos.
- **Separação de Módulos Hercules**: Lógica de portas para o C-130J oficial agora é independente do Mod Hercules original (Anubis), garantindo compatibilidade com ambos.
- **Overhaul do GCI Radar (MiniGCI)**:
    - Implementação do `MiniGCI`, um motor de rastreamento de radar mais performático e robusto.
    - **Ghost Tracks**: Rádios do GCI agora lembram a posição de contatos por até 120 segundos após a perda de sinal radar, simulando memória tática.
    - **Blindagem de Coordenadas**: Proteção contra crashes durante a destruição de unidades rastreadas, garantindo que o loop de busca de alvos não trave.
- **Equilíbrio Dinâmico PvE (BattlefieldManager)**:
    - **Modo Urgência do RED**: Ativado automaticamente quando o lado Azul conquista >70% das zonas (Configurável via `Config.redUrgencyThreshold`), concedendo um boost de produção ao RED para contra-ofensivas.
    - **Compensação de Perda Assimétrica**: O lado em desvantagem recebe multiplicadores de recursos para evitar o colapso total da missão, com limites específicos para RED e Blue.
    - **Variância de Produção Aleatória**: Introdução de fator aleatório cúbico na produção do RED para simular eficácia estratégica variável.
- **Sistema de Salvamento 2.0**: Migração dos arquivos de save para o subdiretório `Missions/Saves/` e atualização do formato para `2.0.json`.
- **Comandos Administrativos por Marcadores**:
- 
- **Logística de Emergência (Strategic AI)**:
    - Notificações de rádio inter-zonas quando uma zona de linha de frente está crítica e sob ameaça imediata.
    - Priorização automática de comboios de suprimento para estas zonas de "emergência".

### Melhorado
- **Blindagem do GroupMonitor (Anti-Stall)**:
    - O loop principal de monitoramento de IA agora está envolto em `pcall` para garantir que o erro em uma única unidade não trave o monitoramento de todo o teatro de operações.
    - Adicionada detecção e remoção automática de "grupos zumbis" (unidades ativas no framework mas sem presença física no DCS), resolvendo o problema de falta de spawns cíclicos (CAP, CAS, etc).
    - Função `getFirstUnit` aprimorada para procurar qualquer unidade viva restante no grupo, não apenas a primeira, mantendo o monitoramento ativo mesmo após perdas parciais.
- **Segurança de Voo de Suporte (AWACS & Tankers)**:
    - Aumento da profundidade estratégica preferida para zonas de suporte (distToFront 2 a 5, antes 1 a 3), reduzindo a exposição a interceptações na linha de frente.
    - Aumento do raio de órbita do AWACS de 10km para 25km para evitar invasão acidental de espaço aéreo inimigo durante a curva da órbita.
- **Hardening do TaskExtensions**: Adição de nil-guards e validações de existência de grupo em funções de missão (`executeSead`, `executeCas`, `landAtAirfield`), prevenindo erros nulos.
- **Economia de Retaguarda**: Zonas em profundidade (rear zones) agora priorizam recursos para missões em vez de defesas superficiais estáticas, a menos que detectem ameaças próximas.
- **Persistência de Veículos**: Melhoria na restauração da carga (esquadrões e suprimentos) transportada por veículos de jogadores entre sessões.
- **Tuning de Recompensas**: Ajuste fino nos ganhos de XP para ações de logística e transporte.

### Corrigido
- **Safe Group Destruction**: Adicionadas proteções de `isExist()` obrigatórias antes de tentar destruir ou acessar o tamanho de grupos terrestres e aéreos, eliminando crashes críticos.
- **Nil Guards (AIActivator)**: Adicionada verificação de existência para dados de posição durante a restauração de sessões salvas, evitando erros de "teleport to point" nulo.

---

## [2026-03-07]

### Adicionado
- **Reatribuição Dinâmica de Recursos**: Unidades ativas (incluindo as que estão voltando para a base) que ainda possuem armamento podem agora ser interceptadas pela IA e enviadas para novos objetivos.
- **Priorização por Proximidade**: A lógica de reatribuição agora calcula distâncias para priorizar o objetivo pendente mais próximo da unidade.
- **Gestão de Unidades Ociosas (Idle Management)**: Lógica automática de `sendHome` para unidades que permanecem sem objetivo por muito tempo, otimizando a reciclagem de recursos.
- **Redirecionamento por Captura de Base**: Unidades ativas agora detectam se sua base de origem foi capturada e se redirecionam automaticamente para a base aliada mais próxima.
- **Inventário de Zona para RED (Debug)**: Adicionada opção `showRedInventory` no `Config.lua` para visualizar o que a IA adversária está construindo em tempo real no mapa F10, incluindo seus pontos de recurso ([X/Y]) e progresso de construções.
- **Controle de Mensagens de Logística**: Adicionada opção `showLogisticsMessages` no `Config.lua` para habilitar ou desabilitar as notificações globais de saída e chegada de suprimentos (Convoy e Air Supply).
- **Blindagem de Loops da IA (Anti-Crash)**:
    - Loop de decisão da `StrategicAI` envolvido em `xpcall` com log de erros para garantir que o timer seja reagendado mesmo após erros inesperados.
    - Loop de atualização do `GroupMonitor` protegido com `xpcall` para evitar que erros de processamento em uma unidade parem todo o rastreamento de missões.

### Alterado
- **Robustez do AIActivator**:
    - A reatribuição agora limpa explicitamente o flag `returning`.
    - Adicionado reset automático de ROE (Rules of Engagement) para `WEAPON FREE` ao reatribuir missões.
- **Persistência de Bases**: Melhoria no `AIActivator` para preservar mudanças dinâmicas de base nas reativações de missão.

### Corrigido
- **Crash Crítico (Cameron.lua)**: Corrigido erro de "nil value" ao acessar `v.home.side` durante a fase de decisão.
- **Nil Guards (Segurança de Código)**: Adicionadas verificações de existência para grupos e unidades em diversos módulos principais antes de acessar suas propriedades.
- **Mitigação de "Spawn Burst"**: Impedido o acúmulo de missões causado por travamentos nos loops da IA.
- **Implementação de Deployment Throttle**: Adicionado limitador de ativações de missão por ciclo (Configurável via `Config.maxActivationsPerCycle`) para cadenciar o spawn de unidades após reinício do servidor.

---

## [2026-03-06]

### Melhorado
- **Refinamento de IADS e Radar**: Melhoria na detecção de ameaças pelo GCI baseada em radares SR/EWR e AWACS ativos.
- **Sobrevivência de CAP**: Ajustes na lógica de retorno por Bingo Fuel e táticas de evasão de ameaças.

## [2026-03-04]

### Adicionado
- **FARPs Dinâmicos**: Implementação de sistema que permite a jogadores criarem FARPs funcionais com zonas de rearmamento e reabastecimento modular (Munidção, Combustível, Comando).

## [2026-03-03]

### Corrigido
- **Ajuste de Posicionamento de Suporte**: AWACS e Tankers agora usam um intervalo flexível de distância do front (2 a 4 zonas), com fallback automático, garantindo que avancem conforme o território é conquistado e não fiquem presos no aeroporto de spawn.
- **Lógica de Destino de Suprimentos**: Correção de bug que enviava comboios para zonas com recursos já no limite, evitando loops logísticos infinitos.

---

## [MIST Framework - Versão mist_128-DYNSLOTS-02]


### Características Especiais
- **Suporte a Dynamic Slots (Jan 2025)**: Versão preparada para o novo recurso do DCS 2.9.6+ que permite criar slots de jogadores dinamicamente durante a missão.
- **Correção de Spawn em Pistas**: Modificação do "Zip (VEAF)" que permite o uso de `mist.dynAdd` em aeródromos e pistas, resolvendo um bloqueio antigo do MIST original.
- **Integração Nativa Pretense**: Inclusão de metadata como `hiddenOnMFD` diretamente no banco de dados do MIST, garantindo que objetos criados dinamicamente sejam reconhecidos corretamente pelo sistema de mapa e IADS do Pretense.

---
*Última atualização: 2026-03-25*

