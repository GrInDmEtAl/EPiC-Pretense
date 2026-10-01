# Changelog - Melhorias na IA Estratégica

Todas as mudanças notáveis no sistema da IA Estratégica do Pretense serão documentadas neste arquivo.

## [2026-10-01]

### Corrigido
- **Supply Transfer (Cameron, RED e BLUE)**: A lista de zonas amigas agora é preenchida antes do processamento de suprimentos, permitindo que a IA considere o déficit de recursos de construções em andamento ao solicitar transferências. A regra permanece limitada a transferências abstratas entre zonas amigas conectadas; o abastecimento da frente continua usando comboios/helicópteros.
- **Métricas do GroupMonitor**: Removida da rotina de relatório a referência fora de escopo a `getWorkerBucket`, evitando erro de script nos callbacks dos workers.
- **FARP dinâmica sem estoque de helicópteros**: Corrigida a leitura da estrutura da missão (`country.helicopter`) usada para localizar slots `Client/Player`; agora o framework configura e confirma o estoque de helicópteros da coalizão da FARP.

### Melhorado
- **FARP dinâmica nativa no ponto construído**: O `farp-pad` agora solicita ao DCS uma FARP nativa com `dynamicSpawn`, posicionada onde o jogador a construiu; não depende das shells do editor (`farpSpawnShells`). O status F10 `Native dynamic slot requested` indica solicitação, não confirmação de disponibilidade para todos os clientes.
- **Estoque inicial das FARPs dinâmicas**: Ao criar uma FARP, o warehouse recebe 9.999 unidades por tipo de helicóptero `Client/Player` da coalizão, 9.999 por tipo de armamento listado em `WarehouseManager.weapons` e 100.000 kg de cada tipo de combustível. Os valores são configuráveis por `Config.farpSpawnAircraftStock`, `Config.farpSpawnWeaponStock` e `Config.farpSpawnFuelStock`. São estoques altos, finitos; o DCS não oferece controle Lua confiável para ativar os flags ilimitados do warehouse de uma FARP criada em runtime.
- **Persistência de estoque FARP**: FARPs restauradas mantêm o inventário salvo; o estoque inicial de armas e combustível só é aplicado na criação ou quando não há inventário salvo.
- **Métricas de desempenho (opt-in)**: Com `Config.performanceMetricsEnabled = true`, o `GroupMonitor` e `Utils.saveTable` emitem totais agregados de grupos ativos/processados/ignorados, jogadores (somente quantidade), erros, remoções e saves. Incluem duração média/máxima quando `socket.gettime` ou `os.clock` está disponível. O intervalo é controlado por `Config.performanceMetricsInterval` (padrão 300s, mínimo efetivo 60s); não há logs individuais por grupo ou jogador.
- **Build direcionado ao arquivo da missão**: `build.ps1` gera `pretense_compiled_v2.lua` por padrão (artefato usado para teste), aceita `-OutputFile` para outros nomes e verifica exatamente o destino selecionado com `-Check`. Se a missão estiver configurada para ler `pretense_compiled.lua`, use `-OutputFile pretense_compiled.lua` tanto no build quanto na verificação.

## [2026-09-30]

### Corrigido
- **GroupMonitor (Comboios de Supply)**: O limite de comboios ativos agora considera o lado que efetivamente envia o suprimento, evitando decisões incorretas quando a zona de destino é neutra.
- **GroupMonitor (retorno de comboio)**: Um grupo só passa ao estado de retorno depois que o cooldown permite emitir a nova rota, evitando que fique marcado como retornando sem receber a ordem correspondente.

### Melhorado
- **GroupMonitor (workers)**: As listas ordenadas de grupos aéreos e terrestres agora são armazenadas em cache e reconstruídas apenas quando um grupo é registrado ou removido, evitando varredura e ordenação completas em cada tick.
- **GroupMonitor (salvage de supply)**: O mapeamento das unidades de cada grupo de suprimento agora é feito uma vez, mantendo tentativas futuras quando o grupo ainda não tem unidades válidas e preservando a limpeza ao despawn.
- **Build do framework compilado**: `build.ps1` gera o pacote num arquivo temporário, valida a presença das fontes e dos marcadores dos módulos e só substitui o arquivo de destino após sucesso.
- **Verificação de consistência**: `build.ps1 -Check` compara o arquivo de destino com os módulos de `src/` sem alterá-lo e retorna erro quando está desatualizado.
- **Codificação do pacote**: O compilado gerado é salvo como UTF-8 sem BOM, formato adequado para o carregamento Lua e estável para comparação no modo de verificação.

## [2026-04-17]

### Corrigido
- **Missões Aéreas Bloqueadas (Anti-Churn)**:
  - Missões aéreas que ficam bloqueadas no `takeoff` agora entram em cooldown antes de retornar ao pool de recursos, evitando reativação imediata em loop.
  - Adicionado contador de falhas consecutivas por produto, com cooldown estendido após reincidência (`Config.airMissionBlockedCooldown`, `Config.airMissionBlockedFailureThreshold`, `Config.airMissionBlockedExtendedCooldown`).
  - A sequência de falhas é limpa quando a missão realmente entra em `inair`, preservando missões saudáveis.

### Melhorado
- **Performance (logs quentes de jogador/combate)**:
  - Logs quentes de combate/jogador (`MissionTracker.tallyHit/tallyKill`, `PlayerTracker.kill`, `MenuRegistry` player events) agora podem ser silenciados em produção via `Config.debugPlayerEvents`, preservando XP/Rank e reduzindo I/O em combate intenso.
- **CAP Defensivo (Cameron)**:
  - Priorização de CAP defensivo passou a usar score por densidade de contatos e profundidade da frente, em vez de ordenar zonas por menor número de tracks.
  - Interceptações de defesa aérea agora ficam limitadas a zonas amigas dentro de uma faixa configurável da frente (`Config.capDefenseFrontDistMax`), reduzindo perseguição para retaguarda profunda por tracks isolados.
  - Adicionada penalidade configurável de retaguarda (`Config.capDefenseRearPenalty`) para manter CAP mais próximo da frente útil.

---

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
- **MIST 128 (compat legacy do Pretense)**:
  - Reintroduzidos metadados `hiddenOnMFD`, `allowLso` e `allowAirboss` no fluxo de DB/dynAdd/getCurrentGroupData/getGroupData da `mist_128-DYNSLOTS-02.lua`.
  - Mantido suporte a dynamic slots da MIST 128, com compatibilidade dos campos usados historicamente pelo framework.

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
- **Comandos Administrativos por Marcadores (F10)**:
    - `spawn:[template]`: Permite spawnar veículos terrestres e utilitários via marcador no mapa.
    - `addres:[valor]`: Comando administrativo para gerenciar recursos das zonas em tempo real.
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
*Última atualização: 2026-04-17*

