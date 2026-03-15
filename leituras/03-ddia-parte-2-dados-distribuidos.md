# Designing Data-Intensive Applications — Parte II: Dados Distribuidos

## Introducao a Parte II

A Parte I do livro tratou de dados armazenados em uma unica maquina. A Parte II aborda o que acontece quando precisamos distribuir dados entre multiplas maquinas. Existem tres razoes principais para distribuir dados: **escalabilidade** (o volume de dados ou carga de leitura/escrita excede a capacidade de uma unica maquina), **tolerancia a falhas** (a aplicacao precisa continuar funcionando mesmo quando maquinas individuais falham), e **latencia** (usuarios estao distribuidos geograficamente e precisam de um servidor proximo).

A distribuicao de dados entre nos pode seguir duas estrategias fundamentais que frequentemente sao combinadas:

- **Replicacao**: manter copias dos mesmos dados em multiplos nos, potencialmente em localizacoes geograficas diferentes.
- **Particionamento (sharding)**: dividir um banco de dados grande em subconjuntos chamados particoes, onde cada particao e atribuida a nos diferentes.

---

## Capitulo 5 — Replicacao (Replication)

Replicacao significa manter uma copia dos mesmos dados em multiplos nos conectados via rede. As motivacoes sao: manter dados geograficamente proximos dos usuarios (reduzir latencia), permitir que o sistema continue funcionando mesmo com falha de alguns nos (aumentar disponibilidade), e escalar o numero de maquinas que podem servir leituras (aumentar throughput de leitura).

Toda a dificuldade da replicacao reside no tratamento de **mudancas nos dados replicados**. Se os dados nunca mudassem, bastaria copiar para cada no uma vez e pronto. Mas dados mudam, e precisamos de estrategias para propagar essas mudancas. O capitulo aborda tres abordagens populares: replicacao com lider unico (single-leader), replicacao com multiplos lideres (multi-leader) e replicacao sem lider (leaderless).

### 5.1 Lideres e Seguidores (Leaders and Followers)

Cada no que armazena uma copia do banco de dados e chamado de **replica**. Quando existem multiplas replicas, surge a questao inevitavel: como garantir que todas as replicas tenham os mesmos dados?

A abordagem mais comum e a **replicacao baseada em lider (leader-based replication)**, tambem conhecida como active/passive ou master/slave. Funciona assim:

1. Uma das replicas e designada como **lider (leader)**. Quando clientes querem escrever no banco, devem enviar suas requisicoes ao lider, que primeiro escreve os novos dados em seu armazenamento local.
2. As outras replicas sao chamadas de **seguidores (followers)**. Sempre que o lider escreve novos dados em seu armazenamento local, ele tambem envia a mudanca para todos os seguidores atraves de um **log de replicacao** ou **change stream**. Cada seguidor pega o log do lider e atualiza sua copia local aplicando as escritas na mesma ordem em que foram processadas pelo lider.
3. Quando um cliente quer ler dados, pode consultar qualquer replica (lider ou seguidor). Porem, escritas sao aceitas somente pelo lider.

Este modelo e usado por bancos de dados relacionais como PostgreSQL, MySQL e SQL Server, bancos NoSQL como MongoDB e RethinkDB, e ate brokers de mensagens como Kafka e RabbitMQ.

#### 5.1.1 Replicacao Sincrona vs Assincrona

Um aspecto crucial e se a replicacao acontece de forma **sincrona** ou **assincrona**.

Na **replicacao sincrona**, o lider espera a confirmacao do seguidor de que recebeu a escrita antes de reportar sucesso ao usuario e antes de tornar a escrita visivel para outras leituras. A vantagem e que o seguidor tem garantia de ter uma copia atualizada dos dados, consistente com o lider. Se o lider falhar repentinamente, os dados estao disponiveis no seguidor. A desvantagem e que se o seguidor nao responder (por crash, falha de rede ou qualquer motivo), a escrita nao pode ser processada. O lider fica bloqueado e precisa esperar ate que o seguidor esteja disponivel novamente. Qualquer no fora do ar causa a parada de todo o sistema.

Na **replicacao assincrona**, o lider envia a mensagem ao seguidor mas nao espera confirmacao. O lider continua processando escritas independentemente. A vantagem e que o lider pode continuar processando escritas mesmo que todos os seguidores tenham ficado para tras. A desvantagem e que se o lider falhar e nao for recuperavel, quaisquer escritas que nao tenham sido replicadas para os seguidores serao perdidas. A escrita nao e duravel mesmo que tenha sido confirmada ao cliente.

Na pratica, usar replicacao totalmente sincrona e impraticavel: se qualquer seguidor nao responder, todo o sistema trava. Por isso, quando se fala em "replicacao sincrona" na pratica, geralmente significa **semi-sincrona (semi-synchronous)**: um seguidor e sincrono e os demais sao assincronos. Se o seguidor sincrono ficar indisponivel ou lento, um dos seguidores assincronos e promovido a sincrono. Isso garante que pelo menos dois nos (lider e um seguidor) tenham copias atualizadas dos dados.

A maioria dos sistemas baseados em lider usa replicacao **completamente assincrona**. Isso significa que se o lider falhar e nao puder ser recuperado, escritas confirmadas ao cliente podem ser perdidas. Essa configuracao e amplamente usada, especialmente quando ha muitos seguidores ou quando estao distribuidos geograficamente, porque a alternativa (parar todo o sistema quando um no atrasa) seria inaceitavel para muitos cenarios.

#### 5.1.2 Configurando Novos Seguidores

De tempos em tempos e necessario adicionar novas replicas: para substituir nos que falharam, para aumentar o numero de replicas, ou para trocar hardware. Como garantir que o novo seguidor tenha uma copia exata dos dados do lider?

Simplesmente copiar arquivos de dados nao e suficiente, porque clientes continuam escrevendo e os dados estao constantemente mudando. Travar o banco (lock) tornaria a operacao indisponivel, violando o objetivo de alta disponibilidade. O processo conceitual e:

1. Tirar um **snapshot consistente** do banco de dados do lider em algum momento, se possivel sem travar o banco. A maioria dos bancos tem essa funcionalidade, pois ela tambem e necessaria para backups. Em alguns cenarios, ferramentas de terceiros sao usadas (como innobackupex para MySQL).
2. Copiar o snapshot para o novo no seguidor.
3. O seguidor se conecta ao lider e requisita todas as mudancas que ocorreram **desde** o snapshot. Isso exige que o snapshot esteja associado a uma posicao exata no log de replicacao do lider. No PostgreSQL, essa posicao e chamada de **log sequence number (LSN)**; no MySQL, de **binlog coordinates**.
4. Quando o seguidor processar o backlog de mudancas desde o snapshot, ele esta "emparelhado" (caught up) e pode continuar processando mudancas em tempo real.

#### 5.1.3 Lidando com Interrupcoes de Nos (Handling Node Outages)

Qualquer no no sistema pode cair (por falha ou manutencao planejada). O objetivo e manter o sistema funcionando apesar de falhas individuais e minimizar o impacto.

**Recuperacao de Seguidor: Catch-up Recovery**

Cada seguidor mantem um log das mudancas recebidas do lider no disco local. Se um seguidor cair e reiniciar, ou se a rede entre seguidor e lider for temporariamente interrompida, a recuperacao e relativamente simples: o seguidor consulta seu log, identifica a ultima transacao processada com sucesso, e requisita ao lider todas as mudancas desde aquele ponto. Ao aplicar essas mudancas, o seguidor volta ao estado atual.

**Failover do Lider: Leader Failover**

O failover do lider e significativamente mais complexo. Quando o lider falha, um dos seguidores precisa ser promovido a novo lider, clientes precisam ser reconfigurados para enviar escritas ao novo lider, e os demais seguidores precisam comecar a consumir mudancas do novo lider. Esse processo e chamado de **failover**.

O failover pode ser manual (um administrador percebe a falha e executa os passos) ou automatico. O failover automatico tipicamente segue estes passos:

1. **Determinar que o lider falhou**: nao existe um metodo infalivel. A maioria dos sistemas usa um **timeout**: nos trocam mensagens entre si, e se um no nao responde por algum periodo (por exemplo, 30 segundos), e considerado morto. Determinar o timeout correto e dificil -- muito longo significa mais tempo de inatividade; muito curto pode causar failovers desnecessarios.

2. **Eleger um novo lider**: pode ser feito por um processo de eleicao (replicas votam) ou um no controlador previamente designado pode apontar o novo lider. O melhor candidato normalmente e a replica com os dados mais atualizados (menor atraso de replicacao em relacao ao lider antigo), para minimizar perda de dados.

3. **Reconfigurar o sistema para usar o novo lider**: clientes precisam enviar escritas ao novo lider. Se o lider antigo voltar, o sistema precisa garantir que ele se torne seguidor e reconheca o novo lider. Esse passo e muitas vezes o mais delicado.

O failover e repleto de armadilhas e problemas potenciais:

- **Perda de dados com replicacao assincrona**: se o antigo lider tinha escritas que nao foram replicadas antes de falhar, e um novo lider e eleito, essas escritas sao perdidas. Se o antigo lider voltar, ele tera escritas que conflitam com o novo lider. A solucao mais comum e simplesmente descartar as escritas nao replicadas do antigo lider, o que pode violar a expectativa de durabilidade dos clientes.

- **Split brain**: em cenarios de falha, dois nos podem acreditar que sao o lider. Isso e extremamente perigoso porque ambos aceitam escritas, levando a dados divergentes ou corrompidos. Alguns sistemas tem mecanismos de seguranca (como STONITH -- Shoot The Other Node In The Head) para desligar um no a forca quando se detecta que dois lideres existem.

- **Perda de dados com efeitos colaterais externos**: em um incidente famoso do GitHub, um seguidor MySQL desatualizado foi promovido a lider. O banco usava um contador auto-incrementado para chaves primarias. O novo lider reutilizou chaves primarias que o antigo lider ja tinha alocado. Essas chaves eram usadas em um cache Redis, resultando em dados privados sendo expostos aos usuarios errados. Descartar escritas e especialmente perigoso quando outros sistemas de armazenamento fora do banco estao coordenados com o conteudo do banco.

- **Timeout correto**: se o timeout para declarar o lider morto for muito curto, pode haver failovers desnecessarios durante picos de carga ou glitches de rede temporarios. Failovers desnecessarios pioram a situacao: o novo failover causa carga adicional no sistema que ja esta sobrecarregado.

Esses problemas nao tem solucoes faceis. Algumas equipes de operacao preferem failover manual por causa dessas complexidades.

#### 5.1.4 Implementacao de Logs de Replicacao

Existem varias abordagens para como o lider envia mudancas aos seguidores:

**Replicacao Baseada em Instrucoes (Statement-based Replication)**

O lider registra cada instrucao de escrita (INSERT, UPDATE, DELETE) que executa e envia essa instrucao aos seguidores, que a executam como se tivessem recebido diretamente de um cliente.

Problemas:
- Instrucoes com funcoes nao-deterministicas como `NOW()` ou `RAND()` geram valores diferentes em cada replica.
- Instrucoes que dependem de dados existentes (como auto-incremento ou `UPDATE ... WHERE ...`) precisam ser executadas na mesma ordem exata em todas as replicas, o que limita a concorrencia.
- Instrucoes com efeitos colaterais (triggers, stored procedures, funcoes definidas pelo usuario) podem produzir resultados diferentes em cada replica.

E possivel contornar esses problemas (substituindo chamadas nao-deterministicas por valores fixos, por exemplo), mas existem tantos casos extremos que outras abordagens sao preferidas. O MySQL usou esse metodo antes da versao 5.1.

**Replicacao por Write-Ahead Log (WAL Shipping)**

Todo banco de dados usa um log de escrita antecipada (write-ahead log, WAL) para durabilidade: um log append-only contendo todos os bytes escritos no banco. No caso de bancos baseados em log-structured storage (SSTables, LSM-trees) ou B-trees (que usam WAL para recuperacao de crash), esse log pode ser usado diretamente para replicacao.

O lider envia o WAL para os seguidores, que processam o log e constroem uma copia exata das estruturas de dados do lider. Este metodo e usado pelo PostgreSQL e Oracle, entre outros.

A principal desvantagem e que o WAL descreve os dados em um nivel muito baixo: quais bytes foram mudados em quais blocos de disco. A replicacao e intimamente acoplada ao mecanismo de armazenamento (storage engine). Se o formato do banco mudar entre versoes (o que e comum), tipicamente nao e possivel ter lider e seguidores executando versoes diferentes do software. Isso impede upgrades sem downtime (zero-downtime upgrades), ja que seria necessario atualizar todos os nos simultaneamente em vez de um de cada vez.

**Replicacao Logica / Baseada em Linhas (Logical / Row-based Replication)**

Uma alternativa e usar um formato de log diferente do formato interno de armazenamento, chamado de **log logico**, para desacoplar a replicacao do storage engine.

Para bancos relacionais, o log logico tipicamente e uma sequencia de registros descrevendo escritas em granularidade de linha:
- Para uma linha inserida: os novos valores de todas as colunas.
- Para uma linha deletada: informacao suficiente para identificar a linha (chave primaria ou todos os valores das colunas se nao houver chave primaria).
- Para uma linha atualizada: informacao para identificar a linha e os novos valores das colunas alteradas.

Uma transacao que modifica varias linhas gera varios registros no log, seguidos por um registro indicando que a transacao foi comitada.

O MySQL usa esse formato em seu **binlog** quando configurado para replicacao row-based. A grande vantagem e que o log logico e desacoplado do storage engine, permitindo que lider e seguidores usem versoes diferentes do software ou ate storage engines diferentes. Alem disso, o formato logico e mais facil de ser consumido por sistemas externos (como data warehouses, caches, ou sistemas de busca), uma tecnica conhecida como **change data capture (CDC)**.

**Replicacao Baseada em Triggers (Trigger-based Replication)**

As abordagens anteriores sao implementadas pelo banco de dados. Em certos cenarios, e necessario mais flexibilidade: replicar apenas um subconjunto dos dados, replicar de um tipo de banco para outro, ou aplicar logica de resolucao de conflitos customizada.

Bancos relacionais oferecem **triggers** e **stored procedures** que permitem executar codigo customizado quando uma mudanca de dados ocorre. Ferramentas como Oracle GoldenGate ou Bucardo para PostgreSQL usam triggers para ler mudancas do log e aplicar logica personalizada antes de replicar.

A replicacao por trigger tem desvantagens significativas: maior overhead, maior propensao a bugs, e mais limitacoes do que a replicacao embutida no banco. Mas e util quando flexibilidade e necessaria.

### 5.2 Problemas com Atraso de Replicacao (Replication Lag)

A replicacao baseada em lider e atraente para cargas de trabalho com muitas leituras e poucas escritas (o padrao comum em aplicacoes web). Podemos criar muitos seguidores e distribuir leituras entre eles. Isso funciona bem com replicacao assincrona: se um seguidor atrasar, nao bloqueia o sistema. Mas se a aplicacao ler de um seguidor assincrono, pode ver dados desatualizados se o seguidor nao estiver emparelhado com o lider.

Se pararmos de escrever e esperarmos, eventualmente todos os seguidores terao os mesmos dados que o lider. Isso e chamado de **consistencia eventual (eventual consistency)**. O "eventualmente" e deliberadamente vago: nao ha limite garantido sobre quanto tempo a convergencia demora. O atraso de replicacao pode ser fracoes de segundo normalmente, mas em situacoes de sobrecarga ou problemas de rede, pode chegar a segundos ou ate minutos.

Ha tres cenarios concretos onde o atraso de replicacao causa problemas visiveis para usuarios:

#### 5.2.1 Lendo Suas Proprias Escritas (Reading Your Own Writes)

Cenario: um usuario faz uma escrita (que vai para o lider), e imediatamente depois faz uma leitura (que pode ir para um seguidor atrasado). O usuario nao ve a mudanca que acabou de fazer, pensando que seus dados foram perdidos.

A garantia necessaria e chamada **read-after-write consistency** (ou **read-your-writes consistency**): o usuario que fez a escrita sempre vera seus proprios dados atualizados, embora outros usuarios possam nao ver imediatamente.

Tecnicas para implementar:
- Se o usuario esta lendo algo que ele pode ter modificado, ler do lider. Por exemplo, o perfil do proprio usuario e sempre lido do lider; perfis de outros usuarios sao lidos de seguidores.
- Se a maioria dos dados pode ter sido editada pelo usuario (como em um aplicativo de notas), a regra anterior nao funciona. Alternativas: rastrear o timestamp da ultima escrita do usuario e, por um minuto apos a escrita, direcionar todas as leituras ao lider; ou monitorar o atraso dos seguidores e redirecionar ao lider quando o atraso excede um limiar.
- O cliente pode lembrar o timestamp da escrita mais recente. O sistema garante que qualquer leitura desse usuario sera servida por uma replica que tenha processado pelo menos ate aquele timestamp. Se nenhum seguidor estiver suficientemente atualizado, a leitura vai ao lider ou espera.

Complicacao adicional: se o usuario acessa de multiplos dispositivos (web e mobile), o timestamp da ultima escrita precisa ser centralizado (nao pode ser armazenado localmente no dispositivo). Alem disso, se os dispositivos estao em datacenters diferentes, nao ha garantia de que ambos alcancem o mesmo lider.

#### 5.2.2 Leituras Monotonicas (Monotonic Reads)

Cenario: o usuario faz duas leituras em sequencia. A primeira leitura vai para um seguidor atualizado e retorna dados recentes. A segunda leitura vai para um seguidor atrasado e retorna dados mais antigos. Para o usuario, parece que o tempo voltou: dados que existiam "desapareceram".

**Monotonic reads** e uma garantia de que isso nao acontece. Se um usuario fez uma leitura e viu dados em um determinado ponto no tempo, qualquer leitura subsequente desse usuario nao vera dados de um ponto anterior no tempo. E uma garantia mais forte que consistencia eventual, mas mais fraca que strong consistency.

Uma forma de implementar e garantir que cada usuario sempre leia da mesma replica (por exemplo, escolhendo a replica baseado em um hash do ID do usuario). Se a replica falhar, o usuario e redirecionado para outra.

#### 5.2.3 Leituras com Prefixo Consistente (Consistent Prefix Reads)

Cenario: em um banco particionado, diferentes particoes operam independentemente. Se uma sequencia causal de escritas acontece em particoes diferentes, um observador pode ver os eventos fora de ordem. Exemplo classico: em uma conversa, o observador ve a resposta antes da pergunta porque a resposta foi escrita em uma particao mais rapida.

**Consistent prefix reads** garante que se uma sequencia de escritas acontece em uma certa ordem, qualquer leitor vera essas escritas na mesma ordem. Isso e particularmente problematico em bancos particionados (sharded) que nao possuem ordenacao global de escritas.

Uma solucao e garantir que escritas causalmente relacionadas sejam escritas na mesma particao. Em alguns cenarios isso e facil; em outros, pode ser dificil ou ineficiente. Existem algoritmos que rastreiam dependencias causais, discutidos em detalhes posteriores no capitulo.

#### 5.2.4 Solucoes para Atraso de Replicacao

O atraso de replicacao e um problema real que idealmente deveria ser resolvido pelo banco de dados de forma transparente, para que desenvolvedores de aplicacao nao precisem se preocupar. E por isso que **transacoes** existem: elas fornecem garantias mais fortes ao desenvolvedor. Abandonar transacoes (como muitos bancos NoSQL fizeram inicialmente) significou transferir a complexidade para o codigo de aplicacao. O livro defende que transacoes sao a abstracao correta, e o Capitulo 7 as aborda em profundidade.

### 5.3 Replicacao Multi-Lider (Multi-Leader Replication)

Na replicacao com lider unico, o lider e um ponto unico de gargalo para escritas e um ponto unico de falha. A **replicacao multi-lider** (tambem chamada de master-master ou active/active) permite que mais de um no aceite escritas. Cada lider age simultaneamente como seguidor dos outros lideres.

#### 5.3.1 Casos de Uso

**Operacao em multiplos datacenters**: com lider unico, todas as escritas passam pelo datacenter que contem o lider. Com multi-lider, cada datacenter pode ter seu proprio lider. Dentro de cada datacenter, a replicacao segue o modelo lider-seguidor; entre datacenters, cada lider replica suas mudancas para os lideres dos outros datacenters.

Comparacao em operacao multi-datacenter:
- **Performance**: com lider unico, toda escrita cruza a internet ate o datacenter do lider (adicionando latencia significativa). Com multi-lider, escritas sao processadas no datacenter local e replicadas assincronamente para os demais.
- **Tolerancia a falha de datacenter**: com lider unico, se o datacenter do lider cair, failover promove um seguidor de outro datacenter. Com multi-lider, cada datacenter opera independentemente; quando o datacenter que caiu volta, a replicacao continua.
- **Tolerancia a problemas de rede**: o trafego entre datacenters tipicamente usa a internet publica, menos confiavel que a rede local. Replicacao sincrona entre datacenters e fragil. Multi-lider com replicacao assincrona tolera melhor problemas de rede inter-datacenter.

A grande desvantagem e que escritas concorrentes nos mesmos dados em lideres diferentes geram **conflitos de escrita** que precisam ser resolvidos. Ferramentas de replicacao multi-lider incluem BDR para PostgreSQL, Tungsten Replicator para MySQL, e GoldenGate para Oracle. Porem, essas ferramentas frequentemente apresentam bugs sutis e interagem de formas problematicas com features do banco como auto-incremento, triggers e constraints de integridade. Multi-lider e considerado perigoso e deve ser evitado se possivel.

**Clientes com operacao offline**: aplicacoes que precisam funcionar sem conexao (como calendarios em smartphones) sao efetivamente um caso de replicacao multi-lider. Cada dispositivo tem um "banco de dados local" que age como lider, e a sincronizacao acontece assincronamente quando ha conexao. CouchDB foi projetado para esse caso de uso.

**Edicao colaborativa**: ferramentas como Google Docs permitem que multiplos usuarios editem simultaneamente. Cada usuario efetivamente escreve em sua copia local e as mudancas sao replicadas assincronamente. Para evitar conflitos de edicao, pode-se usar travas por unidade de mudanca (como um paragrafo), mas isso se assemelha a replicacao com lider unico em termos de performance.

#### 5.3.2 Lidando com Conflitos de Escrita

O maior problema da replicacao multi-lider sao os conflitos de escrita: dois usuarios alteram o mesmo dado em lideres diferentes ao mesmo tempo. Cada escrita e bem-sucedida localmente, mas quando as escritas sao replicadas, ha um conflito.

**Deteccao sincrona vs assincrona**: com lider unico, o segundo escritor recebe um erro ou espera. Com multi-lider e replicacao assincrona, o conflito so e detectado depois que ambas as escritas foram aceitas, tornando muito tarde para pedir ao usuario que resolva. Seria possivel tornar a deteccao sincrona (esperar a replicacao antes de confirmar), mas isso eliminaria a principal vantagem do multi-lider.

**Evitacao de conflitos (Conflict Avoidance)**: a estrategia mais simples e a mais recomendada. Se todas as escritas de um determinado registro passam pelo mesmo lider, conflitos nao existem para aquele registro. Por exemplo, escritas do perfil de um usuario sempre vao para o mesmo datacenter. Porem, isso falha quando o datacenter designado cai ou o usuario muda de localizacao, necessitando reroteamento que pode causar escritas concorrentes em lideres diferentes.

**Convergencia para um estado consistente**: com lider unico, o ultimo escritor vence porque as escritas sao aplicadas em ordem sequencial. Com multi-lider, nao ha ordenacao definida das escritas. Mas todos os lideres devem convergir para o mesmo estado final. Tecnicas:

- **Last Write Wins (LWW)**: atribuir um ID unico a cada escrita (timestamp, UUID, etc.) e a escrita com maior ID vence. Isso e propenso a perda de dados, pois escritas sao silenciosamente descartadas.
- **Prioridade por replica**: cada replica recebe um ID unico, e escritas de replicas com ID maior tem precedencia. Tambem propenso a perda de dados.
- **Merge de valores**: concatenar ou fundir os valores de forma que ambas as escritas sejam preservadas (por exemplo, ordenar e juntar: "B/C" se um lider escreveu "B" e outro "C").
- **Registro do conflito**: preservar todas as informacoes em uma estrutura de dados explicita e deixar o codigo de aplicacao resolver o conflito depois (possivelmente pedindo ao usuario).

**Resolucao customizada de conflitos**: muitos bancos multi-lider permitem codigo customizado para resolucao:
- **Na escrita (on write)**: quando o banco detecta um conflito no log de replicacao, chama um handler de conflito. Bucardo para PostgreSQL permite escrever um snippet Perl.
- **Na leitura (on read)**: quando um conflito e detectado, todas as versoes conflitantes sao armazenadas. Na proxima leitura, todas sao retornadas ao aplicativo, que resolve e escreve o resultado de volta. CouchDB funciona assim.

Pesquisas academicas exploram resolucao automatica de conflitos: **CRDTs (Conflict-free Replicated Data Types)** sao estruturas de dados que podem ser modificadas concorrentemente por multiplos usuarios e resolvem conflitos automaticamente de formas sensatas. Exemplos incluem sets, counters, e textos editaveis. **Mergeable persistent data structures** rastreiam historico explicitamente (similar a Git) e fazem merge de tres vias. **Operational transformation** e o algoritmo por tras do Google Docs, projetado especificamente para edicao concorrente de listas ordenadas de itens (como caracteres em um texto).

#### 5.3.3 Topologias de Replicacao Multi-Lider

A topologia descreve os caminhos de comunicacao pelos quais escritas sao propagadas de um lider para outro.

**Circular**: cada no recebe escritas de um no e as encaminha para outro, formando um circulo. MySQL usa por padrao.

**Estrela (Star/Tree)**: um no raiz encaminha para todos os outros. Pode ser generalizada para uma arvore.

**All-to-all**: cada lider envia suas escritas para todos os outros lideres. E a topologia mais comum.

Problemas das topologias circular e estrela: se um unico no falha, o fluxo de mensagens de replicacao pode ser interrompido para todos os outros nos ate que o no seja reparado. A topologia all-to-all e mais tolerante a falhas porque mensagens podem trafegar por caminhos alternativos.

Problema da topologia all-to-all: mensagens podem chegar fora de ordem. Se o no A insere uma linha e o no B atualiza essa linha, um terceiro no C pode receber a atualizacao antes da insercao (porque a rede entre B e C pode ser mais rapida que entre A e C). Isso e um problema de **ordenacao causal**: a atualizacao depende causalmente da insercao.

Para ordenar eventos corretamente, uma tecnica e usar **version vectors**, mas muitas implementacoes multi-lider tem suporte deficiente para deteccao de ordenacao, tornando a resolucao de conflitos problematica na pratica.

### 5.4 Replicacao Sem Lider (Leaderless Replication)

Algumas arquiteturas de banco de dados abandonam completamente o conceito de lider. Qualquer replica pode aceitar escritas diretamente de clientes. Essa abordagem foi usada nos primordios dos bancos de dados, caiu em desuso durante a era dos bancos relacionais, e foi popularizada novamente pelo Dynamo da Amazon (nao confundir com DynamoDB, que usa lider unico). Riak, Cassandra e Voldemort seguem o modelo Dynamo e sao chamados de bancos "estilo Dynamo".

Em algumas implementacoes, o cliente envia escritas diretamente para varias replicas. Em outras, um no coordenador faz isso em nome do cliente, mas sem impor ordem de escritas (diferente do lider).

#### 5.4.1 Escrevendo Quando um No Esta Indisponivel

Imagine um banco com tres replicas. Uma delas esta indisponivel. Com replicacao baseada em lider, se o seguidor cair, o failover e necessario. Com replicacao sem lider, o cliente envia a escrita para todas as tres replicas em paralelo e considera a escrita bem-sucedida se duas das tres confirmarem. O no indisponivel simplesmente perde a escrita.

Quando o no indisponivel volta, clientes que lerem dele podem receber dados desatualizados. Para resolver, leituras tambem sao enviadas a multiplas replicas em paralelo. O cliente recebe respostas com numeros de versao diferentes e usa a versao mais recente.

**Read Repair**: quando o cliente faz uma leitura e detecta que uma replica tem um valor desatualizado, o cliente escreve o valor mais recente de volta naquela replica. Funciona bem para valores lidos frequentemente, mas valores raramente lidos podem ficar desatualizados por longo tempo.

**Anti-entropy process**: um processo em background constantemente procura diferencas entre replicas e copia dados faltantes. Diferente do log de replicacao baseado em lider, o processo de anti-entropia nao replica escritas em ordem especifica, e pode haver atraso significativo.

#### 5.4.2 Quoruns para Leitura e Escrita

Se existem **n** replicas, e cada escrita precisa ser confirmada por **w** nos, e cada leitura consulta **r** nos, entao enquanto **w + r > n**, esperamos obter um valor atualizado ao ler. Isso e chamado de **quorum de leitura e escrita**.

Valores comuns: com n=3, w=2, r=2. Ou com n=5, w=3, r=3. O numero de nos que precisam confirmar uma escrita ou leitura e calculado como: contanto que w + r > n, ao menos um dos nos consultados na leitura deve ter o valor mais recente.

Os valores w e r podem ser ajustados conforme a carga de trabalho. Com poucas escritas e muitas leituras, w=n e r=1 sao bons (leituras rapidas, escritas lentas). Ha um trade-off: w ou r menores significam maior disponibilidade (toleram mais nos indisponiveis), mas maior probabilidade de ler dados desatualizados.

A condicao de quorum **w + r > n** permite tolerar nos indisponiveis: com w < n, podemos escrever mesmo com nos indisponiveis; com r < n, podemos ler mesmo com nos indisponiveis. Especificamente, com n=3, w=2, r=2, toleramos um no indisponivel. Com n=5, w=3, r=3, toleramos dois nos indisponiveis.

#### 5.4.3 Limitacoes da Consistencia de Quorum

Mesmo com w + r > n, existem cenarios onde leituras retornam valores desatualizados:

- Com **sloppy quorum**, as w escritas podem cair em nos diferentes dos r nos consultados na leitura, eliminando a sobreposicao garantida.
- Se duas escritas ocorrem concorrentemente, nao ha ordem clara. A resolucao segura e fazer merge; se LWW for usado, escritas podem ser perdidas.
- Se escrita e leitura ocorrem concorrentemente, a escrita pode ter sido aplicada em apenas algumas replicas. A leitura pode ou nao retornar o valor novo.
- Se uma escrita foi bem-sucedida em menos que w replicas (algumas falharam) e nao e revertida nas que tiveram sucesso, leituras subsequentes podem ou nao retornar o valor.
- Se um no com valor novo falha e e restaurado de um no com valor antigo, o quorum pode ser quebrado.

Portanto, quoruns nao sao garantia absoluta de leitura do valor mais recente. Sao garantias probabilisticas. Para garantias mais fortes, transacoes ou consenso sao necessarios.

**Monitoramento de desatualizacao**: em replicacao baseada em lider, o atraso e facilmente mensuravel comparando posicoes no log. Em sistemas sem lider, nao ha ordem fixa de escritas, dificultando o monitoramento. Pesquisas existem para medir desatualizacao em quoruns, mas nao sao pratica comum.

#### 5.4.4 Sloppy Quorums e Hinted Handoff

Bancos com quoruns adequados toleram falhas de nos individuais sem failover ou slowdown. Sao atraentes para aplicacoes que requerem alta disponibilidade e baixa latencia, tolerando leituras ocasionalmente desatualizadas.

Porem, quoruns nao sao tao tolerantes a falhas quanto parecem. Uma rede interrompida pode facilmente cortar um cliente de um grande numero de nos. Se o cliente consegue alcancar alguns nos mas nao o suficiente para um quorum, ha uma escolha:

- Retornar erro para todas as escritas para as quais nao se consegue um quorum de w nos?
- Aceitar escritas e escreve-las nos nos acessiveis, mesmo que nao sejam os nos designados para aquele valor?

A segunda opcao e chamada de **sloppy quorum**: escritas e leituras ainda requerem w e r respostas bem-sucedidas, mas podem incluir nos que normalmente nao sao responsaveis por aquele valor. E como se, ao ser trancado para fora de casa, voce batesse na porta do vizinho e pedisse para ficar no sofa temporariamente.

Quando a rede e reparada, quaisquer escritas temporariamente aceitas por nos "errados" sao enviadas aos nos corretos. Isso e chamado de **hinted handoff**: a escrita carrega uma "dica" (hint) de qual no e o destinatario real.

Sloppy quorums aumentam a disponibilidade de escrita: o sistema aceita escritas enquanto qualquer w nos estiverem acessiveis, em qualquer lugar do cluster. Mas significa que mesmo com w + r > n, nao ha garantia de que uma leitura retornara o valor mais recente, porque o valor mais recente pode estar temporariamente em nos fora do quorum normal.

Sloppy quorums sao opcionais em Riak, habilitados por padrao em Cassandra e Voldemort.

#### 5.4.5 Detectando Escritas Concorrentes

Em bancos estilo Dynamo, conflitos podem ocorrer durante escrita multi-no e durante read repair. O problema e que eventos podem chegar em diferentes ordens em diferentes nos, e se os nos simplesmente sobrescreverem valores, as replicas divergem permanentemente.

**Last Write Wins (LWW)**: atribuir um timestamp a cada escrita e manter apenas a mais "recente". Cassandra usa esse metodo e e o unico suportado no Riak. LWW alcanca convergencia a custo de durabilidade: escritas concorrentes sao silenciosamente descartadas. Se perder dados e inaceitavel, LWW e uma escolha ruim. A unica situacao segura para LWW e quando chaves sao escritas apenas uma vez e depois sao imutaveis (como usar um UUID como chave para cada escrita).

**Relacao "Acontece-antes" (Happens-before)**: duas operacoes A e B podem ter uma de tres relacoes: A aconteceu antes de B (A causou ou influenciou B), B aconteceu antes de A, ou sao **concorrentes** (nenhuma sabia da outra). Se uma operacao aconteceu antes da outra, a posterior pode sobrescrever a anterior. Se sao concorrentes, ha um conflito que precisa ser resolvido.

Para determinar se duas operacoes sao concorrentes, o algoritmo usa numeros de versao:

1. O servidor mantem um numero de versao para cada chave, incrementado a cada escrita.
2. Quando um cliente le uma chave, o servidor retorna o valor e a versao.
3. Quando um cliente escreve, deve incluir o numero de versao da leitura anterior e merge todos os valores recebidos.
4. Quando o servidor recebe uma escrita com versao v, pode sobrescrever todos os valores com versao menor ou igual a v (o cliente ja os viu), mas deve manter valores com versao maior (escritas concorrentes que o cliente nao viu).

Quando uma escrita nao inclui numero de versao, e concorrente com todas as escritas existentes.

**Merge de valores concorrentes (siblings)**: os clientes precisam fazer merge inteligente. Para adicionar itens, a uniao de siblings funciona. Para remover itens, a simples uniao nao funciona (um item removido pode reaparecer). A solucao e usar um **tombstone** (marcador de delecao) em vez de simplesmente remover o item.

Merge de siblings em codigo de aplicacao e complexo e propenso a erros. CRDTs podem automatizar esse processo para certos tipos de dados.

**Version Vectors**: com multiplas replicas, cada replica precisa de seu proprio numero de versao. A colecao de numeros de versao de todas as replicas e chamada de **version vector**. Cada replica incrementa seu proprio numero de versao ao processar uma escrita e rastreia os numeros de versao que viu de outras replicas. O version vector permite distinguir entre sobrescrita e escritas concorrentes.

O version vector e enviado dos nos de banco de dados para clientes na leitura e de volta ao banco na escrita. O version vector permite ao banco distinguir entre escritas que substituem e escritas concorrentes. Riak usa uma variante chamada **dotted version vector**.

---

## Capitulo 6 — Particionamento (Partitioning)

Quando o volume de dados e muito grande para uma unica maquina, e necessario dividir os dados em **particoes** (tambem chamadas de **shards** no MongoDB/Elasticsearch/SolrCloud, **regions** no HBase, **tablets** no Bigtable, **vnodes** no Cassandra/Riak, ou **vBuckets** no Couchbase).

Cada particao e um pequeno banco de dados independente, embora o banco possa suportar operacoes que cruzam particoes. O principal motivo e **escalabilidade**: dados e carga de consultas sao distribuidos entre muitos discos e processadores. Consultas que operam em uma unica particao podem ser executadas independentemente, permitindo escalar o throughput adicionando mais nos.

Particionamento e geralmente combinado com replicacao: cada particao e armazenada em multiplos nos para tolerancia a falhas. Um no pode armazenar multiplas particoes, e cada particao tem um lider e seguidores (se usando replicacao baseada em lider). Cada no pode ser lider de algumas particoes e seguidor de outras.

### 6.1 Particionamento de Dados Chave-Valor

O objetivo e distribuir dados e carga de forma uniforme entre nos. Se o particionamento for desigual, algumas particoes terao mais dados ou carga que outras, criando **hotspots**. Uma particao com carga desproporcional e chamada de **skewed**. No pior caso, toda a carga vai para uma unica particao, anulando o beneficio de ter multiplos nos.

A forma mais simples de evitar hotspots e atribuir registros aleatoriamente a nos. Isso distribui uniformemente, mas tem uma desvantagem enorme: ao ler um registro, nao ha como saber em qual no ele esta, necessitando consultar todos os nos em paralelo.

#### 6.1.1 Particionamento por Faixa de Chave (Key Range Partitioning)

Atribuir a cada particao um intervalo continuo de chaves, do minimo ao maximo, similar a volumes de uma enciclopedia. As faixas nao precisam ser uniformes porque os dados podem nao ser distribuidos uniformemente.

Dentro de cada particao, chaves podem ser mantidas em ordem (usando SSTables ou B-trees). Isso torna range scans eficientes: buscar todos os registros com chaves entre X e Y requer acessar apenas as particoes relevantes.

A desvantagem e o risco de hotspots. Se a chave e um timestamp, por exemplo, todas as escritas de um dia vao para a mesma particao enquanto as demais ficam ociosas. Para evitar isso, usa-se algo diferente como primeiro elemento da chave (por exemplo, nome do sensor seguido de timestamp), mas isso exige que range queries sobre timestamps consultem cada particao separadamente.

Bancos que usam: HBase, Bigtable, RethinkDB, e MongoDB (antes da versao 2.4).

#### 6.1.2 Particionamento por Hash de Chave (Hash Partitioning)

Para evitar skew e hotspots, muitos bancos usam uma funcao hash para determinar a particao. Uma boa funcao hash transforma chaves skewed em valores distribuidos uniformemente. Cada particao recebe um intervalo de valores hash (em vez de um intervalo de chaves).

Os limites entre particoes podem ser igualmente espacados ou escolhidos pseudo-aleatoriamente (**consistent hashing**, embora o termo seja ambiguo e o livro prefira "hash partitioning").

A desvantagem e a perda de ordenacao: chaves adjacentes agora estao em particoes diferentes, tornando range queries ineficientes. No MongoDB, range queries em chaves com hash sao enviadas para todas as particoes. Riak, Couchbase e Voldemort nao suportam range queries na chave primaria.

Cassandra usa uma abordagem hibrida: a chave primaria consiste de multiplas colunas. A primeira coluna e hasheada para determinar a particao, mas as colunas restantes sao usadas como indice concatenado para ordenacao dentro da particao. Isso permite range queries eficientes dentro de uma particao (sobre as colunas nao-hasheadas) enquanto distribui escritas uniformemente.

**Hotspots remanescentes**: nenhum esquema de hash elimina hotspots completamente. No caso extremo, todas as leituras e escritas sao para a mesma chave (por exemplo, o perfil de uma celebridade no Twitter). A maioria dos bancos nao tem logica para compensar automaticamente. A aplicacao pode adicionar um numero aleatorio ao inicio ou fim da chave para distribuir escritas entre particoes, mas isso requer que leituras consultem todas as particoes e combinem os resultados -- mais trabalho para o codigo de aplicacao.

### 6.2 Particionamento e Indices Secundarios

Os esquemas acima funcionam para dados acessados por chave primaria. Indices secundarios complicam o cenario porque nao mapeiam diretamente para particoes. Indices secundarios sao a razao de ser dos bancos relacionais e dos motores de busca (Solr, Elasticsearch). Muitos bancos chave-valor (como HBase e Voldemort) os evitam pela complexidade, mas outros (como Riak, Cassandra) os adicionaram pela utilidade.

#### 6.2.1 Particionamento de Indices Secundarios por Documento (Local Index)

Cada particao mantem seus proprios indices secundarios, cobrindo apenas os documentos naquela particao. E chamado de **indice local**. Ao escrever, so a particao que contem o documento e atualizada.

A desvantagem e que uma busca por indice secundario (por exemplo, todos os carros vermelhos) precisa consultar **todas** as particoes. Essa abordagem e chamada de **scatter/gather** e torna leituras por indice secundario caras, especialmente com muitas particoes. Apesar disso, e amplamente usada: MongoDB, Riak, Cassandra, Elasticsearch, SolrCloud e VoltDB usam indices locais.

#### 6.2.2 Particionamento de Indices Secundarios por Termo (Global Index)

Em vez de cada particao ter seu proprio indice, constroi-se um **indice global** que cobre dados de todas as particoes. O indice global tambem e particionado -- mas de forma diferente dos dados. Por exemplo, carros com cores de "a" a "r" estao em uma particao do indice, e cores de "s" a "z" em outra. O indice pode ser particionado por faixa do termo (util para range scans) ou por hash do termo (distribuicao mais uniforme).

A vantagem e que leituras sao mais eficientes: em vez de scatter/gather, basta consultar a particao do indice que contem o termo desejado.

A desvantagem e que escritas sao mais lentas e complexas: um unico documento pode afetar multiplas particoes do indice (se o documento tem multiplos termos indexados em particoes diferentes). Na pratica, atualizacoes de indices globais sao **assincronas**: apos uma escrita, a mudanca no indice global pode nao ser visivel imediatamente. DynamoDB afirma atualizar indices globais em fracoes de segundo em circunstancias normais, mas pode atrasar mais durante falhas.

### 6.3 Rebalanceamento de Particoes

Com o tempo, mudancas ocorrem: throughput aumenta (mais CPUs necessarias), tamanho dos dados cresce (mais discos necessarios), ou nos falham (outros nos precisam assumir). O processo de mover dados e carga de um no para outro e chamado de **rebalanceamento**.

Requisitos do rebalanceamento: apos o rebalanceamento, a carga deve ser distribuida uniformemente; durante o rebalanceamento, o banco deve continuar aceitando leituras e escritas; nao mais dados que o necessario devem ser movidos para minimizar trafego de rede e I/O de disco.

#### 6.3.1 Estrategias de Rebalanceamento

**Anti-padrao: hash mod N**: usar `hash(key) mod N` (onde N e o numero de nos) parece intuitivo mas e pessimo para rebalanceamento. Se N muda (de 10 para 11 nos, por exemplo), a maioria das chaves precisa ser movida. E excessivamente caro.

**Numero fixo de particoes**: criar muito mais particoes do que nos (por exemplo, 1000 particoes para 10 nos, cada no recebendo ~100 particoes). Quando um no e adicionado, ele "rouba" algumas particoes de cada no existente. Quando um no e removido, suas particoes sao distribuidas entre os restantes. Apenas particoes inteiras sao movidas; as chaves dentro de uma particao nao mudam. O numero de particoes nao muda; muda apenas a atribuicao de particoes a nos. Usado por Riak, Elasticsearch, Couchbase e Voldemort.

O numero de particoes e tipicamente fixo quando o banco e configurado e nao muda depois. O numero ideal depende do tamanho total dos dados, que e dificil prever. Particoes grandes demais tornam o rebalanceamento caro; particoes pequenas demais criam overhead de gerenciamento.

**Particionamento dinamico**: bancos com key range partitioning teriam dificuldade com numero fixo de particoes porque limites pre-definidos podem resultar em particoes muito desiguais. Bancos como HBase e RethinkDB criam particoes dinamicamente: quando uma particao cresce alem de um limiar, e dividida em duas; quando encolhe, e fundida com uma adjacente. Similar a B-trees.

Vantagem: o numero de particoes se adapta ao volume de dados. Com poucos dados, poucas particoes; com muitos dados, muitas particoes. Caveat: com um banco vazio, ha apenas uma particao, e todas as escritas vao para um unico no ate que a primeira divisao ocorra. Alguns bancos mitigam com **pre-splitting**: configurar um conjunto inicial de particoes em um banco vazio.

**Particionamento proporcional ao numero de nos**: numero de particoes proporcional ao numero de nos (numero fixo de particoes por no). Quando um novo no entra, ele escolhe aleatoriamente particoes existentes para dividir e assume metade de cada. A randomizacao pode produzir divisoes desiguais, mas em media, com 256 particoes por no, o rebalanceamento e razoavel. Usado por Cassandra e Ketama.

#### 6.3.2 Operacoes: Automatico ou Manual?

Rebalanceamento pode ser totalmente automatico ou totalmente manual. Totalmente automatico e conveniente mas pode causar surpresas desagradaveis: se combinado com deteccao automatica de falhas, um no lento (sobrecarregado) pode ser declarado morto, causando rebalanceamento que gera mais carga no sistema e nos outros nos, potencialmente causando falha em cascata. Couchbase, Riak e Voldemort geram sugestoes automaticas de rebalanceamento mas requerem confirmacao de um administrador.

### 6.4 Roteamento de Requisicoes (Request Routing)

Quando um cliente quer ler ou escrever, como sabe para qual no enviar a requisicao? Essa e uma instancia do problema geral de **service discovery**. Tres abordagens:

1. **Cliente contacta qualquer no** (round-robin ou load balancer): se o no tem a particao, processa; senao, encaminha para o no correto. Cassandra e Riak usam um protocolo gossip entre nos para disseminar mudancas na atribuicao de particoes, eliminando a dependencia de um servico externo de coordenacao.

2. **Camada de roteamento**: todas as requisicoes vao para um routing tier que determina o no correto e encaminha. O routing tier nao processa requisicoes, so encaminha.

3. **Cliente ciente das particoes**: o cliente sabe qual particao esta em qual no e faz a requisicao diretamente ao no correto.

O problema central e: como o componente que toma decisoes de roteamento descobre que a atribuicao de particoes a nos mudou?

Muitos sistemas usam um servico de coordenacao como **ZooKeeper** para rastrear metadados de cluster. Cada no se registra no ZooKeeper. O ZooKeeper mantem o mapeamento autoritativo de particoes para nos. Componentes de roteamento (ou clientes) assinam essa informacao no ZooKeeper e sao notificados quando mudancas ocorrem. HBase, SolrCloud e Kafka usam ZooKeeper extensivamente. MongoDB tem uma arquitetura similar com seus proprios config servers e protocolo.

---

## Capitulo 7 — Transacoes (Transactions)

Muitas coisas podem dar errado em sistemas de dados: o banco pode falhar no meio de uma escrita, a aplicacao pode crashar a qualquer momento, interrupcoes de rede podem cortar a comunicacao, escritas concorrentes podem sobrescrever umas as outras, leituras podem ver dados parcialmente atualizados. **Transacoes** sao o mecanismo pelo qual bancos de dados simplificam esses problemas para a aplicacao. Uma transacao agrupa varias leituras e escritas em uma unica operacao logica: ou todas tem sucesso (commit) ou todas falham (abort/rollback). Se falhar, a aplicacao pode simplesmente retentar com seguranca.

Transacoes nao sao uma lei da natureza; sao uma abstracao criada para simplificar o modelo de programacao. Nem toda aplicacao precisa de transacoes, e as vezes ha vantagens em enfraquece-las ou abandona-las (por performance, disponibilidade ou escalabilidade). Para tomar essa decisao informadamente, e preciso entender exatamente o que transacoes oferecem.

### 7.1 ACID: O Significado Real

As garantias de seguranca de transacoes sao frequentemente descritas pela sigla ACID: Atomicity, Consistency, Isolation, Durability. Na pratica, as implementacoes de ACID variam enormemente entre bancos. "ACID compliance" tornou-se quase um termo de marketing. Sistemas que nao fornecem todas as garantias ACID sao as vezes chamados de BASE (Basically Available, Soft state, Eventual consistency), um acronimo ainda mais vago que ACID.

**Atomicidade (Atomicity)**: no contexto de ACID, atomicidade NAO se refere a concorrencia (que e coberta por Isolation). Atomicidade significa que se uma transacao que faz multiplas escritas falha no meio (por crash do processo, falha de rede, erro de disco, violacao de constraint), nenhuma das escritas tera efeito. A transacao nao pode ser parcialmente completada: ou todas as mudancas sao aplicadas ou nenhuma. Se falhar, a aplicacao sabe que nada mudou e pode retentar com seguranca. Talvez **abortabilidade** fosse um termo melhor.

**Consistencia (Consistency)**: no contexto de ACID, consistencia significa que certas invariantes sobre os dados devem ser verdadeiras antes e depois da transacao. Por exemplo, em um sistema contabil, creditos e debitos devem estar balanceados. Porem, essa e uma propriedade da **aplicacao**, nao do banco de dados. O banco pode ajudar com constraints (chaves estrangeiras, unicidade), mas e a aplicacao que define o que e "consistente". O C em ACID e essencialmente um termo que nao pertence ali -- foi incluido para que a sigla funcionasse. Consistencia depende de A e I para ser mantida.

**Isolamento (Isolation)**: quando multiplas transacoes acessam os mesmos dados concorrentemente, podem interferir umas nas outras. Isolamento significa que transacoes concorrentes sao isoladas umas das outras: cada transacao pode fingir que e a unica executando no banco. O resultado e o mesmo que se as transacoes tivessem executado **serialmente** (uma apos a outra). A implementacao classica e **serializabilidade** (serializability), mas na pratica, bancos usam niveis mais fracos de isolamento porque serializabilidade tem custo de performance significativo.

**Durabilidade (Durability)**: a promessa de que dados comitados nao serao perdidos, mesmo em caso de falha de hardware ou crash do banco. Tipicamente, significa que dados foram escritos em armazenamento nao volatil (disco rigido ou SSD) ou em um write-ahead log. Em bancos replicados, durabilidade pode significar que dados foram copiados para um certo numero de nos. Nenhuma implementacao de durabilidade e perfeita: se todos os discos e backups forem destruidos simultaneamente, os dados estao perdidos. Durabilidade com replicacao pode perder dados se todos os nos com dados atualizados falharem simultaneamente. Na pratica, durabilidade e uma questao de reduzir risco, nao de elimina-lo completamente.

### 7.2 Operacoes Single-Object vs Multi-Object

Transacoes multi-objeto sao necessarias quando varias operacoes de escrita em registros diferentes precisam ser coordenadas. Exemplos: inserir uma linha em uma tabela e atualizar um contador em outra; atualizar uma tabela e seus indices secundarios; atualizar documentos que referenciam uns aos outros em um banco de documentos.

Muitos bancos distribuidos abandonaram transacoes multi-objeto porque sao dificeis de implementar com particionamento (quando objetos estao em particoes ou nos diferentes) e podem prejudicar performance em cenarios de alta disponibilidade.

**Operacoes single-object**: quase todos os bancos oferecem atomicidade e isolamento para operacoes em um unico objeto. Atomicidade pode ser implementada com WAL para recuperacao de crash. Isolamento com lock no objeto. Alguns bancos oferecem operacoes atomicas mais complexas, como compare-and-set. Mas essas nao sao transacoes no sentido usual: transacoes sao mecanismos para agrupar multiplas operacoes em multiplos objetos em uma unica unidade de execucao.

### 7.3 Niveis Fracos de Isolamento

Serializabilidade (o nivel mais forte) tem custo de performance. Muitos bancos usam niveis mais fracos por padrao. Esses niveis sao muito mais dificeis de entender e causam bugs sutis. Ate bancos comerciais grandes (IBM DB2) usam niveis fracos como padrao.

#### 7.3.1 Read Committed

O nivel mais basico de isolamento de transacoes. Fornece duas garantias:

**Sem dirty reads**: ao ler, so se ve dados que foram comitados. Se uma transacao T1 escreveu algo mas nao comitou, T2 nao pode ler o valor nao comitado. Isso evita ver estados intermediarios (uma transacao atualizou um objeto mas nao outro) ou dados que serao revertidos.

**Sem dirty writes**: ao escrever, so se sobrescreve dados que foram comitados. Se duas transacoes T1 e T2 escrevem no mesmo objeto concorrentemente, dirty writes podem causar mistura de estados (um campo de um registro escrito por T1 e outro por T2). Read committed previne isso normalmente usando locks de escrita: uma transacao que quer escrever deve adquirir um lock exclusivo no objeto e manter ate commit ou rollback.

Para prevenir dirty reads, uma abordagem seria usar o mesmo lock para leituras, mas isso prejudicaria a latencia de leituras (leituras poderiam ser bloqueadas por escritas longas). Na pratica, a maioria dos bancos que usa read committed lembra o valor antigo e o novo sendo escrito pela transacao ativa. Leituras durante a transacao retornam o valor antigo; somente apos o commit, leituras retornam o novo valor.

Read committed e o padrao em Oracle 11g, PostgreSQL, SQL Server 2012, MemSQL e muitos outros.

#### 7.3.2 Snapshot Isolation e Repeatable Read

Read committed ainda permite anomalias. Exemplo classico: **nonrepeatable read** (ou **read skew**). Alice tem duas contas bancarias com $500 cada. Ela olha uma conta, que mostra $500. Uma transferencia de $100 entre contas e comitada nesse instante. Ela olha a outra conta, que mostra $400 (ja com $100 removido). Do ponto de vista de Alice, parece que $100 desapareceu (a soma e $900 em vez de $1000). Se ela recarregar a pagina, vera os valores corretos. Mas em certas situacoes, isso e intoleravel:

- **Backups**: o backup de um banco grande pode levar horas. Escritas continuam durante o backup. Partes do backup podem ter dados antigos, partes dados novos. Restaurar esse backup produziria inconsistencias permanentes.
- **Queries analiticas e verificacoes de integridade**: queries longas que escaneiam partes grandes do banco podem ver dados em estados inconsistentes, retornando resultados sem sentido.

**Snapshot isolation** resolve esse problema. A ideia: cada transacao le de um **snapshot consistente** do banco -- a transacao ve todos os dados que estavam comitados no inicio da transacao. Mesmo que dados mudem durante a execucao, a transacao ve apenas o snapshot antigo.

A implementacao mais comum e **MVCC (Multi-Version Concurrency Control)**. O banco mantem multiplas versoes de cada objeto. Cada transacao recebe um ID unico (txid) monotonicamente crescente. Quando uma transacao escreve dados, o valor escrito e marcado com o txid da transacao que o escreveu. Quando uma transacao le, uma regra de visibilidade determina quais versoes sao visiveis:

1. No inicio da transacao, o banco lista todas as transacoes em andamento. Escritas dessas transacoes sao ignoradas, mesmo que comitem depois.
2. Escritas de transacoes abortadas sao ignoradas.
3. Escritas de transacoes com txid maior (iniciadas depois) sao ignoradas.
4. Todas as outras escritas sao visiveis.

Basicamente, uma escrita e visivel se: a transacao que a fez ja tinha comitado quando a transacao leitora comecou, E a transacao que a fez nao esta na lista de transacoes em andamento quando a transacao leitora comecou.

Indices em bancos MVCC podem apontar para multiplas versoes de um objeto, e a query filtra versoes nao visiveis.

**Confusao de nomenclatura**: snapshot isolation e chamada de **Serializable** no Oracle e **Repeatable Read** no PostgreSQL e MySQL. O padrao SQL nao tem o conceito de snapshot isolation (porque foi definido antes da tecnica existir). Nao existe um padrao real de nomes entre bancos, criando grande confusao.

#### 7.3.3 Prevenindo Updates Perdidos (Lost Updates)

Lost updates ocorrem quando duas transacoes fazem read-modify-write concorrentemente. Exemplos: dois usuarios incrementam um contador, dois usuarios editam uma pagina wiki, dois processos atualizam um JSON complexo. A segunda escrita nao incorpora a mudanca da primeira, perdendo-a.

Solucoes:

**Operacoes atomicas**: muitos bancos oferecem operacoes atomicas como `UPDATE counters SET value = value + 1 WHERE key = 'foo'`. Implementadas com lock exclusivo no objeto durante a operacao, ou executadas em um unico thread. MongoDB oferece operacoes atomicas em documentos JSON, e Redis oferece operacoes atomicas em estruturas de dados. Nem sempre e possivel expressar a logica como operacao atomica.

**Lock explicito (explicit locking)**: a aplicacao explicitamente trava os objetos que vai atualizar. Usando `SELECT ... FOR UPDATE` em SQL, que retorna linhas e as trava contra escritas concorrentes. Exige cuidado do desenvolvedor para nao esquecer de adquirir o lock.

**Compare-and-Set (CAS)**: em bancos que nao oferecem transacoes, CAS permite uma atualizacao apenas se o valor nao mudou desde a leitura. `UPDATE wiki SET content = 'novo' WHERE id = 1234 AND content = 'antigo'`. Se o conteudo ja foi mudado por outra transacao, a atualizacao nao tem efeito e a aplicacao pode retentar. Porem, se o banco le de um snapshot antigo (snapshot isolation), a condicao WHERE pode ser verdadeira mesmo que outra escrita concorrente tenha ocorrido. Verificar se a implementacao de CAS do banco e segura e essencial.

**Resolucao de conflitos e replicacao**: em bancos replicados (especialmente multi-lider e leaderless), o problema e mais complexo porque escritas concorrentes podem ocorrer em nos diferentes. Locks e CAS assumem uma copia canonica dos dados (o lider). Bancos replicados permitem que escritas concorrentes criem siblings e resolvam usando aplicacao ou CRDTs.

#### 7.3.4 Write Skew e Phantoms

Write skew e uma anomalia mais sutil que lost updates. Exemplo: um hospital exige que pelo menos um medico esteja de plantao. Dois medicos estao de plantao. Ambos se sentem mal e decidem sair ao mesmo tempo. Cada um verifica que ha dois medicos de plantao, conclui que e seguro sair, e atualiza seu proprio registro para "fora de plantao". Ambas as transacoes comitam, e agora nao ha nenhum medico de plantao.

Write skew nao e dirty write nem lost update, porque as transacoes estao atualizando objetos **diferentes**. Mas o resultado viola uma invariante (pelo menos um medico de plantao).

Mais exemplos de write skew:
- **Reserva de sala de reuniao**: duas transacoes verificam que a sala esta livre e ambas reservam o mesmo horario.
- **Jogo multiplayer**: dois jogadores movem pecas para a mesma posicao.
- **Registro de username unico**: dois usuarios tentam registrar o mesmo username ao mesmo tempo.
- **Prevencao de gasto duplo**: duas transacoes tentam gastar do mesmo saldo, cada uma verificando que o saldo e suficiente.

O padrao e: (1) uma query SELECT verifica uma condicao, (2) baseado no resultado, a aplicacao decide continuar, (3) uma escrita (INSERT, UPDATE, DELETE) muda os dados de forma que invalida a condicao verificada no passo 1. Esse padrao e chamado de **phantom**: a escrita em uma transacao muda o resultado da query de busca de outra transacao.

**Materialization de conflitos**: se o problema e que nao ha objeto para adquirir lock (os phantoms sao ausencias de linhas), pode-se criar artificialmente objetos para representar o espaco de possibilidades. Por exemplo, criar linhas para cada combinacao sala/horario em uma tabela auxiliar e adquirir locks nessas linhas. Isso e chamado de **materializing conflicts**: converter um phantom em um conflito concreto sobre linhas existentes. E uma solucao de ultimo recurso; normalmente, serializabilidade e preferida.

### 7.4 Serializabilidade (Serializability)

Serializabilidade e o nivel de isolamento mais forte. Garante que, mesmo que transacoes executem em paralelo, o resultado e como se tivessem executado serialmente (uma por vez, em alguma ordem). O banco previne todas as race conditions possiveis.

Tres tecnicas implementam serializabilidade:

#### 7.4.1 Execucao Serial Real (Actual Serial Execution)

A abordagem mais simples: eliminar a concorrencia completamente. Executar cada transacao em um unico thread, uma de cada vez, em ordem serial. Embora obvio, so se tornou viavel recentemente por duas razoes:

1. **RAM ficou barata**: o dataset inteiro (ou a particao ativa) cabe em memoria, eliminando I/O de disco durante transacoes, tornando-as muito rapidas.
2. **Designers perceberam** que transacoes OLTP sao tipicamente curtas e fazem poucas leituras e escritas, diferente de queries analiticas longas (que usam snapshot isolation e nao precisam de serializabilidade).

VoltDB, Redis e Datomic implementam serial execution.

**Stored procedures**: com execucao serial, transacoes interativas (que esperam por input de rede do cliente entre operacoes) seriam desastrosas: um unico thread ficaria ocioso esperando. A solucao e submeter a transacao inteira como uma **stored procedure**: todo o codigo de transacao e enviado antecipadamente ao banco, que o executa sem pausas de rede. Stored procedures historicamente tem ma reputacao (linguagens arcaicas, dificeis de debugar, dificeis de versionar), mas implementacoes modernas usam linguagens como Java (VoltDB), Lua (Redis) ou Clojure (Datomic).

**Particionamento para performance**: em um unico thread, o throughput e limitado ao de um unico core de CPU. Para escalar, dados sao particionados, e cada particao tem seu proprio thread de transacoes. Transacoes que envolvem uma unica particao sao rapidas; transacoes cross-partition requerem coordenacao entre particoes e sao muito mais lentas (VoltDB reporta throughput ate 1000x menor para transacoes cross-partition). O esquema de particionamento precisa ser cuidadosamente projetado.

Execucao serial e viavel sob restricoes especificas: transacoes curtas e rapidas, dados cabem em memoria, throughput de escrita baixo o suficiente para um unico core (ou dados podem ser particionados sem necessidade frequente de cross-partition transactions).

#### 7.4.2 Two-Phase Locking (2PL)

Por aproximadamente 30 anos, houve apenas um algoritmo amplamente utilizado para serializabilidade: **two-phase locking (2PL)**. Nao confundir com two-phase commit (2PC).

2PL e similar a locks usados em read committed, mas muito mais forte. Regras:
- Multiplos leitores podem ler um objeto simultaneamente (shared lock).
- Se alguem quer escrever, precisa de acesso exclusivo (exclusive lock).
- Se uma transacao leu um objeto, nenhuma outra pode escrever nele ate que a primeira transacao comite ou aborte.
- Se uma transacao escreveu um objeto, nenhuma outra pode ler ou escrever nele ate que a primeira transacao comite ou aborte.

A diferenca em relacao a read committed: em read committed, escritores bloqueiam apenas escritores. Em 2PL, escritores bloqueiam leitores e leitores bloqueiam escritores. Snapshot isolation tem o mantra "leitores nunca bloqueiam escritores e escritores nunca bloqueiam leitores"; 2PL nao tem essa propriedade.

Implementacao: cada objeto tem um lock com modo compartilhado (shared) e exclusivo (exclusive). Para ler, adquire shared lock (bloqueado se alguem tem exclusive). Para escrever, adquire exclusive lock (bloqueado se alguem tem qualquer lock). Se uma transacao le e depois quer escrever, faz upgrade de shared para exclusive. Locks sao mantidos ate o fim da transacao (fase 1: adquirir locks, fase 2: liberar locks -- apenas no commit/abort).

**Performance**: o grande problema do 2PL. Significativamente pior que niveis fracos de isolamento por causa do overhead de adquirir e liberar locks e, mais importante, pela concorrencia reduzida. Se duas transacoes concorrentes tocam os mesmos dados, uma precisa esperar. Transacoes podem esperar por periods indeterminados. Deadlocks sao comuns (transacao A espera por B e B espera por A); o banco precisa detectar e abortar uma delas.

**Predicate locks**: para prevenir phantoms (e write skew), 2PL precisa de locks que cubram objetos que ainda nao existem. Predicate locks protegem todos os objetos que satisfazem uma condicao de busca. Por exemplo, "todos os registros com room_id = 123 e time entre 12:00 e 13:00". Se alguma transacao T2 quer inserir, atualizar ou deletar qualquer registro que satisfaca essa condicao, precisa esperar por T1.

**Index-range locks** (next-key locking): predicate locks nao performam bem porque verificar predicados e caro. A maioria dos bancos com 2PL implementa **index-range locks**, uma aproximacao simplificada. Em vez de travar exatamente os registros que satisfazem um predicado, trava-se um intervalo maior baseado no indice. Por exemplo, travar todas as reservas para room_id = 123 (em qualquer horario), ou todas as reservas entre 12:00 e 13:00 (para qualquer sala). Menos preciso mas muito mais eficiente. Se nao ha indice adequado, o banco pode travar a tabela inteira, o que e ruim para performance mas correto.

#### 7.4.3 Serializable Snapshot Isolation (SSI)

SSI e um algoritmo relativamente novo (descrito em 2008) que fornece serializabilidade completa com performance comparavel a snapshot isolation. Usado em PostgreSQL 9.1+ (como nivel Serializable) e no FoundationDB.

A diferenca fundamental: 2PL e **pessimista** (assume que conflitos vao acontecer e previne antecipadamente). Serial execution e extremamente pessimista (um unico thread). SSI e **otimista**: permite que transacoes executem concorrentemente sem bloqueio (usando snapshot isolation normal). Quando uma transacao quer comitar, o banco verifica se houve conflitos. Se houve, a transacao e abortada e precisa ser retentada.

SSI funciona detectando duas situacoes:

**Detectando leituras de MVCC desatualizadas (stale MVCC reads)**: quando uma transacao T2 comitou uma escrita que afeta dados que T1 leu antes de T2 comitar (mas T1 nao viu a escrita de T2 por causa das regras de snapshot). No momento do commit de T1, o banco verifica se alguma escrita ignorada (por regras de MVCC) foi comitada. Se sim, T1 e abortada.

**Detectando escritas que afetam leituras anteriores**: quando T1 leu dados e T2 subsequentemente escreveu dados que invalidam a leitura de T1. O banco usa uma estrutura similar a index-range locks, mas sem bloquear: apenas registra que certas transacoes leram dados em certo intervalo. Quando uma escrita ocorre nesse intervalo, as transacoes afetadas sao notificadas. No commit, se a transacao foi notificada e a transacao conflitante ja comitou, a transacao e abortada.

Comparado com 2PL, SSI nao bloqueia transacoes esperando por locks, resultando em latencia muito mais previsivel e menor. Transacoes de leitura longa (como backups) nao sao problemicas. A taxa de aborts e maior com alta contencao, mas o throughput geral e melhor em muitos cenarios. SSI e menos sensivel a transacoes lentas que serial execution (que bloqueia todo o sistema).

---

## Capitulo 8 — Os Problemas com Sistemas Distribuidos (The Trouble with Distributed Systems)

Este capitulo e um estudo pessimista (mas realista) de tudo que pode dar errado em sistemas distribuidos. Entender as falhas possiveis e essencial para projetar sistemas que funcionam apesar delas.

Em um computador individual, o software tipicamente funciona deterministicamente: ou funciona ou nao. Falhas de hardware sao raras e geralmente totais (o computador para completamente). Em sistemas distribuidos, a realidade e fundamentalmente diferente: **falhas parciais** sao a norma. Algumas partes do sistema podem funcionar enquanto outras falham de maneiras imprevisiveis, e nem sempre e possivel saber o que falhou. Falhas parciais sao nao-deterministicas: a mesma operacao pode funcionar as vezes e falhar outras.

### 8.1 Redes Nao Confiaveis

A maioria dos sistemas distribuidos usa redes de pacotes compartilhados assincronos. Isso significa que ao enviar uma mensagem para outro no:

1. A requisicao pode ter se perdido na rede.
2. A requisicao pode estar em uma fila esperando (o destinatario pode estar sobrecarregado).
3. O no destinatario pode ter falhado (crash).
4. O no destinatario pode ter parado de responder temporariamente (por GC ou sobrecarga).
5. O no destinatario pode ter processado a requisicao mas a resposta se perdeu na rede.
6. O no destinatario pode ter processado a requisicao mas a resposta esta atrasada.

O remetente nao tem como distinguir entre esses cenarios. A unica informacao disponivel e que uma resposta nao chegou. Se nenhuma resposta for recebida, o remetente nao sabe nem se a mensagem foi entregue.

**Timeouts e deteccao de falhas**: a abordagem usual e declarar que um no esta indisponivel apos um timeout. Mas qual timeout usar?

Timeout longo: espera mais tempo, mas o sistema demora a reagir a falhas reais.
Timeout curto: reage rapidamente a falhas, mas pode declarar nos vivos como mortos prematuramente (o no pode estar apenas lento). Declarar um no morto desnecessariamente e perigoso: se as tarefas do no "morto" sao transferidas para outros nos, isso dobra a carga; se o no ainda esta vivo, operacoes podem ser executadas em duplicata.

Nao existe um valor de timeout "correto". Na pratica, timeouts sao calibrados experimentalmente. Pesquisas como Phi Accrual Failure Detector (usado no Akka e Cassandra) medem variabilidade de tempos de resposta e ajustam o timeout automaticamente.

**Congestionamento de rede**: em redes com muitos nos, filas de pacotes podem se encher em varias camadas: switches de rede, sistema operacional do destinatario, hipervisor em ambientes virtualizados, e TCP flow control. Em ambientes de nuvem publica, recursos de rede sao compartilhados entre muitos clientes, causando variabilidade de latencia ainda maior. TCP implementa mecanismos de flow control, backpressure e retransmissao que adicionam atrasos variaveis.

**Faults de rede na pratica**: estudos mostram que problemas de rede sao surpreendentemente comuns, mesmo em datacenters controlados. Um switch mal configurado, uma atualilzacao de firmware, um tubarao mordendo um cabo submarino. Nao e necessario que a rede fique totalmente indisponivel: mesmo glitches temporarios podem causar problemas. O software precisa ser capaz de lidar com falhas de rede graciosamente (retentando, usando nos alternativos, etc).

### 8.2 Relogios Nao Confiaveis

Em sistemas distribuidos, o tempo e um assunto delicado. Cada maquina tem seu proprio relogio de quarzo, que nao e perfeitamente acurado. Protocolos como **NTP (Network Time Protocol)** permitem sincronizar relogios usando servidores de tempo, mas a precisao e limitada (tipicamente dezenas de milissegundos, e muito pior em redes congestionadas).

#### 8.2.1 Relogios de Tempo do Dia vs Monotoicos

**Time-of-day clocks (relogios de tempo do dia)**: retornam a data e hora atual segundo algum calendario (tipicamente UNIX epoch). Sincronizados via NTP. Problema: podem **pular para tras** quando o NTP detecta que o relogio local esta adiantado. Nao sao adequados para medir duracoes.

**Monotonic clocks (relogios monotoicos)**: garantem sempre avancar. Adequados para medir duracoes (diferenca entre dois instantes). O valor absoluto nao tem significado (pode ser "nanosegundos desde que o computador ligou"). Cada computador tem seu proprio relogio monotonico; nao faz sentido comparar valores de relogios monotoicos entre maquinas diferentes. NTP pode ajustar a **velocidade** de um relogio monotoico (slewing) para que avance mais rapido ou devagar, mas nunca pule.

#### 8.2.2 Problemas de Confiabilidade dos Relogios

**Confianca em relogios sincronizados**: Muitas coisas podem dar errado:
- O daemon NTP pode estar parado ou com firewall bloqueando acesso.
- O relogio pode desviar significativamente entre sincronizacoes NTP.
- Se o relogio diverge muito do servidor NTP, pode se recusar a sincronizar ou forcar um salto no tempo.
- Firewalls podem bloquear NTP acidentalmente.
- Virtualizacao pode fazer uma VM "pausar" por dezenas de milissegundos sem o relogio perceber.
- Dispositivos moveis podem ignorar NTP completamente para economizar bateria.

**Timestamps para ordenacao de eventos**: e tentador usar timestamps para determinar qual escrita aconteceu por ultimo (Last Write Wins). Mas se os relogios das maquinas estiverem dessincronizados (mesmo por milissegundos), a escrita "mais recente" pode na verdade ter acontecido primeiro. Isso pode silenciosamente descartar escritas. LWW e usado por Cassandra, Riak e outros bancos sem lider. Ate NTP bem configurado nao garante ordenacao correta de eventos que aconteceram com milissegundos de diferenca.

**Intervalos de confianca**: ao ler um relogio, a leitura nao e um ponto exato; e um intervalo com margem de incerteza. Se o NTP sincronizou 5 segundos atras e o drift maximo e 200ppm, o erro pode ser ate 1ms. Infelizmente, a maioria dos sistemas nao expoe essa incerteza. Uma excecao notavel e o **Google Spanner TrueTime API**, que retorna um intervalo [earliest, latest] para cada leitura de relogio. O Spanner espera pela duracao da incerteza antes de comitar transacoes, garantindo que timestamps reflitam causalidade. O intervalo de confianca depende de relogios atomicos e GPS em cada datacenter do Google, mantendo a incerteza tipicamente abaixo de 7ms.

**Pausas de processo**: um processo pode ser pausado por longos periodos sem perceber:
- **Garbage collector (GC)**: pode pausar todos os threads de uma aplicacao por dezenas de milissegundos ou ate segundos.
- **Maquinas virtuais**: o hipervisor pode suspender uma VM (live migration) por intervalos.
- **Swap/paging**: se o SO usa swap para disco, acessar paginas pode causar pausas.
- **Sinais UNIX**: SIGSTOP pode pausar um processo indefinidamente.
- **I/O sincrono**: pode bloquear threads em cenarios inesperados.

Exemplo de problema: um no adquire um lease (lock com expiracao). O lease expira em 10 segundos. O no adquire o lease, entao o GC pausa por 15 segundos. Quando o processo volta, o lease ja expirou e outro no o adquiriu. Mas o primeiro no nao sabe que o lease expirou e age como se ainda o tivesse. Isso pode causar violacoes de seguranca.

### 8.3 Conhecimento, Verdade e Mentiras

Em sistemas distribuidos, um no nao pode confiar em seu proprio julgamento. Um no pode achar que e o lider, que seu lease e valido, que uma operacao foi bem-sucedida -- mas estar errado.

**A verdade e definida pela maioria**: muitos algoritmos distribuidos dependem de **quorums** -- decisoes requerem votos de uma maioria de nos. Um unico no nao pode tomar decisoes unilaterais com seguranca. Se um no acredita ser o lider mas a maioria dos nos discorda, o no nao e o lider.

**Fencing tokens**: para proteger recursos contra acesso indevido por nos que erroneamente acreditam ter um lock/lease, usa-se **fencing tokens**. Cada vez que um lock e concedido, o servico de lock emite um numero monotonicamente crescente (fencing token). Quando o cliente envia uma escrita para o recurso protegido, inclui o fencing token. O recurso verifica que o token e maior ou igual ao ultimo que viu. Se um token antigo chega (de um cliente cujo lease expirou mas nao sabe), a escrita e rejeitada. ZooKeeper usa o zxid ou znode version como fencing token.

**Falhas Bizantinas**: ate agora, assumimos que nos podem falhar (crash) ou ser lentos, mas nao que nos mentem deliberadamente. Se nos podem enviar mensagens falsas (corrompidas ou maliciosas), temos **falhas bizantinas**. Um sistema e **tolerante a falhas bizantinas** se funciona corretamente mesmo com nos maliciosos. Em geral, sistemas dentro de um datacenter controlado por uma unica organizacao nao precisam de tolerancia a falhas bizantinas -- a complexidade e o custo sao enormes. Sistemas que precisam incluem blockchains (onde participantes nao confiam uns nos outros) e sistemas aeroespaciais/militares.

Porem, e razoavel proteger contra nos "fracos bizantinos": corrupcao de pacotes de rede (checksums), input de usuario malicioso (sanitizacao), servidores NTP incorretos (consultando multiplos).

#### 8.3.1 Modelos de Sistema

Para raciocinar formalmente sobre o que algoritmos distribuidos podem fazer, usamos modelos abstratos:

**Modelos de tempo**:
- **Sincrono**: assume limites conhecidos para atraso de rede, pausa de processo e desvio de relogio. Nao e realista para a maioria dos sistemas praticos.
- **Parcialmente sincrono**: assume que o sistema se comporta como sincrono na maior parte do tempo, mas pode violar limites ocasionalmente. Modelo realista para a maioria dos sistemas.
- **Assincrono**: nao assume nada sobre tempo. Nao permite usar timeouts. Muito restritivo; poucos algoritmos funcionam nesse modelo.

**Modelos de falha de nos**:
- **Crash-stop**: um no falha parando permanentemente. Nunca volta.
- **Crash-recovery**: um no pode falhar e eventualmente voltar. Pode ter armazenamento duravel que sobrevive ao crash, mas dados em memoria sao perdidos.
- **Bizantino**: nos podem fazer absolutamente qualquer coisa, incluindo mentir e tentar enganar outros nos.

O modelo mais util para a maioria dos sistemas reais e **parcialmente sincrono com crash-recovery**.

**Corretude de algoritmos**: um algoritmo e correto se satisfaz suas propriedades em todos os cenarios permitidos pelo modelo de sistema. Propriedades podem ser de **safety** ("nada ruim acontece" -- se violada, ha um ponto especifico no tempo onde foi violada e nao pode ser desfeita) ou **liveness** ("algo bom eventualmente acontece" -- pode nao ser verdade em um instante especifico, mas ha esperanca de que se torne verdade no futuro).

Para sistemas distribuidos, geralmente exigimos que propriedades de safety sejam SEMPRE mantidas (mesmo em falhas) e que propriedades de liveness sejam satisfeitas sob certas premissas (por exemplo, assumindo que nos crashados eventualmente se recuperam e a rede eventualmente entrega mensagens).

---

## Capitulo 9 — Consistencia e Consenso (Consistency and Consensus)

Este capitulo explora as garantias mais fortes que sistemas distribuidos podem fornecer e os algoritmos que as implementam. O tema central e que a forma mais fundamental de garantia em sistemas distribuidos e o **consenso**: fazer com que todos os nos concordem sobre algo.

### 9.1 Garantias de Consistencia

A maioria dos bancos de dados replicados fornece no minimo **consistencia eventual**: se parar de escrever e esperar tempo suficiente, todas as replicas convergem. Mas "eventual" e vago. Sistemas com garantias mais fortes sao mais faceis de usar corretamente, mas tipicamente tem performance pior ou sao menos tolerantes a falhas. Diferentes modelos de consistencia existem (diferente de niveis de isolamento de transacoes, embora haja sobreposicao).

### 9.2 Linearizabilidade

Linearizabilidade (tambem chamada de **consistencia atomica**, **consistencia forte**, **consistencia imediata** ou **consistencia externa**) e a garantia mais forte de consistencia single-object.

A ideia basica: fazer parecer que ha apenas uma copia dos dados e todas as operacoes sao atomicas. Mesmo que haja multiplas replicas, o sistema se comporta como se houvesse uma unica copia e toda operacao toma efeito atomicamente em algum ponto entre seu inicio e fim. Uma vez que uma leitura retorna um valor novo, todas as leituras subsequentes (de qualquer cliente) tambem devem retornar o valor novo.

Linearizabilidade e uma **garantia de recencia**: o valor lido e o mais recente, nao de um cache ou replica desatualizada.

#### 9.2.1 Quando Linearizabilidade e Necessaria

**Locking e eleicao de lider**: um sistema que usa lock para eleicao de lider (como no ZooKeeper) precisa de linearizabilidade: todos os nos devem concordar sobre quem e o lider. Se o lock nao for linearizavel, dois nos podem acreditar que sao lideres (split brain).

**Constraints de unicidade**: se o banco precisa garantir que um username e unico, ou que um assento em voo nao e vendido duas vezes, isso requer linearizabilidade. Duas requisicoes concorrentes tentando registrar o mesmo username: exatamente uma deve ter sucesso. Isso e conceitualmente identico a adquirir um lock ou a uma operacao de compare-and-set.

**Dependencias entre canais de comunicacao**: se ha canais de comunicacao adicionais (como um file storage + fila de mensagens), a falta de linearizabilidade pode causar race conditions. Exemplo: salvar uma imagem no storage e enviar uma mensagem na fila com a referencia. Se a fila e mais rapida que a replicacao do storage, o consumidor pode nao encontrar a imagem.

#### 9.2.2 Implementando Linearizabilidade

**Replicacao com lider unico**: potencialmente linearizavel (se leituras forem do lider ou de seguidores sabidamente atualizados). Na pratica, bugs de implementacao ou failover mal executado podem violar linearizabilidade. Async replication com leituras de seguidores nao e linearizavel.

**Algoritmos de consenso**: linearizaveis. Algoritmos como Raft, Paxos e Zab implementam linearizabilidade. ZooKeeper e etcd usam esses algoritmos.

**Replicacao multi-lider**: NAO linearizavel. Escritas concorrentes em lideres diferentes sao processadas assincronamente.

**Replicacao sem lider (estilo Dynamo)**: provavelmente NAO linearizavel. Mesmo com quorum estrito (w + r > n), race conditions entre escritas e leituras podem violar linearizabilidade por causa de atrasos na replicacao. Sloppy quorums sao definitivamente nao linearizaveis.

#### 9.2.3 O Custo da Linearizabilidade

**O Teorema CAP e suas nuances**: considere uma situacao com replicacao entre dois datacenters. Se a rede entre eles e interrompida:

- Com replicacao multi-lider, cada datacenter pode continuar operando normalmente. Escritas em um datacenter sao enfileiradas e replicadas quando a rede e restaurada. Nao e linearizavel, mas e disponivel.
- Com replicacao single-leader, se o lider esta em um datacenter e o outro perde conexao, o datacenter desconectado nao pode processar escritas (nem leituras linearizaveis). E linearizavel, mas nao disponivel para o datacenter desconectado.

Isso ilustra o trade-off do **Teorema CAP**: em caso de **particao de rede (P)**, o sistema deve escolher entre **consistencia linearizavel (C)** e **disponibilidade (A)**. O trade-off nao e binario entre tres opcoes: P (particao de rede) nao e uma escolha, e algo que acontece. Quando P acontece, voce escolhe entre C e A. Quando nao ha particao, voce pode ter ambos.

O CAP e frequentemente mal interpretado e tem escopo limitado (so considera um modelo de consistencia -- linearizabilidade -- e um tipo de falha -- particoes de rede). Na pratica, ha muitos trade-offs mais nuancados.

Mesmo sem particoes de rede, ha razoes para nao usar linearizabilidade: performance. Sistemas de replicacao sem lider como Dynamo sacrificam linearizabilidade por performance e disponibilidade, nao apenas para tolerar particoes de rede. A latencia adicional de coordenacao para linearizabilidade e inaceitavel para muitas aplicacoes.

### 9.3 Garantias de Ordenacao

Ordenacao e um tema recorrente: a ordem de escritas no log de replicacao, a ordem de serializacao em transacoes, timestamps em logs de eventos.

#### 9.3.1 Ordenacao e Causalidade

Ha uma conexao profunda entre ordenacao e causalidade. Causalidade impoe uma **ordem parcial**: se A causou B, entao A vem antes de B. Se A e B sao concorrentes (nenhum causou o outro), nao ha ordem definida entre eles. Isso difere de uma **ordem total**, onde quaisquer dois elementos podem ser comparados (um e anterior ao outro).

Linearizabilidade implica ordem total: em um sistema linearizavel, todas as operacoes estao em uma unica linha do tempo totalmente ordenada. Causalidade permite operacoes concorrentes (incomparaveis), portanto e uma ordem parcial.

Linearizabilidade e mais forte que causalidade. Linearizabilidade implica consistencia causal, mas nao vice-versa. Consistencia causal e o modelo de consistencia mais forte que nao sacrifica performance por causa de atrasos de rede. Muitos sistemas que parecem precisar de linearizabilidade na verdade so precisam de causalidade.

#### 9.3.2 Numeros de Sequencia e Timestamps Logicos

Rastrear toda a causalidade e impraticavel. Em vez disso, podemos usar **numeros de sequencia** ou **timestamps logicos** para criar uma **ordem total** consistente com causalidade. Em bancos single-leader, o log de replicacao define uma ordem total consistente com causalidade.

**Lamport Timestamps**: algoritmo que gera uma ordem total consistente com causalidade em um sistema distribuido sem lider. Cada no tem um ID e um contador. O Lamport timestamp e o par (contador, nodeID). Cada no incrementa seu contador a cada operacao. Quando um no recebe um timestamp maior que seu contador, atualiza seu contador. A ordenacao: maior contador vence; em caso de empate, maior nodeID vence.

Diferenca crucial entre Lamport timestamps e version vectors: Lamport timestamps definem uma **ordem total** (qualquer dois timestamps podem ser comparados), enquanto version vectors definem uma **ordem parcial** (podem determinar se dois eventos sao concorrentes ou nao). Lamport timestamps sao mais compactos mas nao capturam concorrencia: ao comparar dois Lamport timestamps, nao da para saber se os eventos foram concorrentes ou causalmente relacionados.

**Limitacao de Lamport timestamps**: embora definam uma ordem total, nao sao suficientes para resolver problemas que requerem decisoes **em tempo real**. Exemplo: dois usuarios registram o mesmo username concorrentemente. Podemos usar Lamport timestamps para determinar que um pedido veio "primeiro" e dar-lhe o username. Mas cada no so pode tomar essa decisao **depois de coletar todos os timestamps de todos os nos**, para ter certeza de que nenhum timestamp menor existe. Se outro no esta indisponivel, nao sabemos se ele processou um pedido com timestamp menor. Precisamos esperar ou usar outro mecanismo.

#### 9.3.3 Total Order Broadcast

**Total order broadcast** (ou **atomic broadcast**) e um protocolo de comunicacao que garante:

1. **Entrega confiavel (reliable delivery)**: se uma mensagem e entregue a um no, e entregue a todos os nos.
2. **Entrega totalmente ordenada (totally ordered delivery)**: mensagens sao entregues a todos os nos na mesma ordem.

E exatamente o que se precisa para replicacao de banco de dados: se cada no processa as mesmas escritas na mesma ordem, todas as replicas sao consistentes. Tambem e equivalente a implementacao de logs de replicacao.

Total order broadcast e diferente de Lamport timestamps: Lamport timestamps definem uma ordem total **apos o fato**, enquanto total order broadcast garante que mensagens sao entregues em ordem **em tempo real**, sem precisar esperar.

**Relacao com linearizabilidade**: total order broadcast e equivalente a consenso e pode ser usado para implementar linearizabilidade. Para uma leitura linearizavel em cima de total order broadcast: enviar uma mensagem de leitura no log, esperar ate que ela seja entregue de volta, e entao ler o estado naquele ponto. Ou consultar a posicao mais recente do log, esperar que todas as mensagens ate aquela posicao sejam entregues, e entao ler.

Reciprocamente, linearizabilidade pode implementar total order broadcast: usar um registro linearizavel como contador de sequencia, e cada mensagem recebe o proximo numero atomicamente.

### 9.4 Transacoes Distribuidas e Consenso

**Consenso** e o problema de fazer multiplos nos concordarem sobre algo. E um dos problemas mais fundamentais (e dificeis) em computacao distribuida.

Situacoes que requerem consenso:
- **Eleicao de lider**: todos os nos devem concordar sobre quem e o lider (para evitar split brain).
- **Atomic commit**: em transacoes que envolvem multiplos nos/particoes, todos os nos devem concordar se a transacao e comitada ou abortada (para manter atomicidade).

#### 9.4.1 Atomic Commit e Two-Phase Commit (2PC)

Em um banco single-node, atomic commit depende da ordem de escrita no disco: se o registro de commit do WAL e escrito com sucesso, a transacao esta comitada; caso contrario, e abortada.

Em um banco multi-node, nao basta que cada no comite independentemente: se alguns nos comitam e outros abortam, as replicas divergem. Uma vez comitada, a transacao nao pode ser revertida (outros podem ter agido baseados nela). E o **principio da irrevogabilidade de commit**: quando um no comitou, nao tem volta.

**Two-Phase Commit (2PC)**: algoritmo classico para atomic commit distribuido.

O 2PC usa um componente chamado **coordenador** (ou transaction manager), frequentemente implementado como biblioteca na aplicacao que solicita a transacao. O protocolo tem duas fases:

**Fase 1 - Prepare**: o coordenador envia uma requisicao de prepare para cada participante (no). Cada participante responde "sim" (prometo que posso comitar) ou "nao" (nao posso comitar). Quando um participante vota "sim", esta fazendo uma promessa irrevogavel de que comitara se instruido. Ele deve garantir que pode comitar em qualquer circunstancia futura (gravar dados em disco, verificar conflitos, etc.) antes de votar "sim".

**Fase 2 - Commit/Abort**: se TODOS os participantes votaram "sim", o coordenador escreve uma decisao de commit em seu log de transacoes (o ponto de commit) e envia commit para todos. Se QUALQUER participante votou "nao", o coordenador envia abort para todos.

O protocolo tem duas acoes irrevogaveis: quando um participante vota "sim", nao pode mudar de ideia; quando o coordenador decide commit, a decisao e definitiva.

**O problema do coordenador em duvida**: se o coordenador falha entre a fase 1 e a fase 2 (depois de receber todos os votos "sim" mas antes de enviar a decisao), os participantes ficam **em duvida (in doubt)**: votaram "sim" mas nao sabem se devem comitar ou abortar. Nao podem abortar unilateralmente (o coordenador pode ter decidido commit) nem comitar unilateralmente (outro participante pode ter votado "nao"). Devem esperar o coordenador se recuperar. Nesse meio tempo, os participantes mantem locks nos dados da transacao.

Se o coordenador nunca se recuperar (disco corrompido), os dados ficam permanentemente travados. Em teoria, um administrador pode intervir manualmente, mas e uma situacao operacional terrivel.

**Three-Phase Commit (3PC)**: foi proposto como alternativa que nao bloqueia. Porem, assume limites de atraso de rede (modelo sincrono) e nao funciona corretamente na presenca de atrasos de rede ilimitados ou pausas de processo. 2PC continua sendo usado na pratica.

#### 9.4.2 Transacoes Distribuidas na Pratica

Dois tipos de transacoes distribuidas:
- **Database-internal**: todos os nos executam o mesmo software de banco de dados. Otimizacoes internas sao possiveis (como VoltDB, MySQL Cluster, etc.)
- **Heterogeneas**: participantes sao sistemas completamente diferentes (banco de dados + message broker + servico externo). O protocolo padrao e **XA transactions** (eXtended Architecture), suportado por muitos bancos (PostgreSQL, MySQL, DB2, SQL Server) e message brokers (ActiveMQ, HornetQ, MSMQ, IBM MQ).

**Limitacoes de XA/2PC**:
- O coordenador e um ponto unico de falha. Se nao for replicado, uma falha pode bloquear participantes.
- Como o coordenador e parte da aplicacao (na maioria das implementacoes XA), a aplicacao se torna stateful (mantendo estado de transacoes), complicando deploy e escalabilidade.
- A compatibilidade entre implementacoes XA de diferentes vendors e limitada na pratica.
- Transacoes 2PC amplificam falhas: se qualquer participante falhar, toda a transacao falha. Transacoes in-doubt mantem locks que bloqueiam outras transacoes.
- 2PC nao funciona bem com deteccao de deadlocks entre sistemas diferentes.

#### 9.4.3 Consenso Tolerante a Falhas

O problema do 2PC e que o coordenador e um ponto unico de falha. Algoritmos de consenso tolerante a falhas resolvem isso: os nos votam, mas o protocolo garante progresso mesmo com falhas de nos.

Algoritmos classicos: **Viewstamped Replication**, **Paxos**, **Raft** e **Zab (ZooKeeper Atomic Broadcast)**.

Esses algoritmos satisfazem as seguintes propriedades:

- **Acordo uniforme (Uniform agreement)**: todos os nos que decidem, decidem o mesmo valor. (Safety)
- **Integridade (Integrity)**: nenhum no decide duas vezes. (Safety)
- **Validade (Validity)**: se um no decide o valor v, v foi proposto por algum no. (Safety - previne decisoes triviais/invalidas)
- **Terminacao (Termination)**: todo no que nao falhou eventualmente decide. (Liveness)

A terminacao e a parte dificil: garante que o algoritmo faz progresso e nao fica travado para sempre. Requer que menos da metade dos nos falhe. Se mais da metade falha, a terminacao nao pode ser garantida, mas as propriedades de safety (acordo e integridade) sao SEMPRE mantidas: mesmo com falhas, o sistema nunca toma decisoes incorretas ou contradictorias.

A maioria dos algoritmos de consenso nao usa votacao "um a um". Em vez disso, usam **epocas (epochs)** e distinguem lider e seguidores dentro de cada epoca. Em cada epoca, o lider e unico. Se o lider falha, uma nova epoca comeca com uma nova eleicao. Se dois lideres conflitam (por exemplo, o lider anterior nao sabe que foi deposto), o lider com epoca maior vence.

Para cada decisao, o lider propoe um valor e os seguidores votam. Um quorum (maioria) de votos e necessario para uma decisao. A votacao acontece duas vezes: uma para eleger o lider e outra para cada proposta. As duas votacoes devem ter quoruns que se sobrepoem, garantindo que pelo menos um no no quorum de votacao da proposta participou da eleicao mais recente e nao aceita propostas de lideres de epocas anteriores.

**Raft** e o algoritmo mais moderno e compreensivel: um lider e eleito, propoe entradas de log, seguidores votam e aceitam. Se o lider falha, timeout causa nova eleicao com epoca incrementada. Raft garante que o novo lider tem todas as entradas de log comitadas pelo lider anterior.

**Limitacoes do consenso**: consenso nao e gratuito. O processo de votacao e essencialmente sincrono: nos devem esperar pela resposta da maioria. Em redes com alta latencia, isso impacta performance. Consenso requer um numero fixo de nos para funcionar (3 nos toleram 1 falha, 5 toleram 2). Adicionar ou remover nos dinamicamente e possivel mas complexo. Consenso depende de timeouts para detectar falha de lider; em redes com latencia variavel, pode haver eleicoes frequentes desnecessarias que prejudicam performance.

#### 9.4.4 Servicos de Associacao e Coordenacao (Membership and Coordination Services)

**ZooKeeper** e **etcd** sao projetos frequentemente descritos como "bancos de dados de chave-valor distribuidos" ou "servicos de coordenacao". Implementam algoritmos de consenso (Zab no ZooKeeper, Raft no etcd) e sao projetados para armazenar pequenas quantidades de dados que cabem em memoria.

ZooKeeper fornece funcionalidades cruciais:

- **Linearizable atomic operations**: operacoes atomicas como compare-and-set, usadas para implementar locks distribuidos. Se multiplos nos tentam adquirir o lock concorrentemente, apenas um tera sucesso.
- **Total ordering of operations**: operacoes recebem um numero de sequencia monotonico (zxid) e timestamp, usaveis como fencing tokens.
- **Failure detection**: clientes e ZooKeeper mantem sessoes com heartbeats. Se o heartbeat para (cliente crashou ou rede particionada), o ZooKeeper declara a sessao morta. Locks adquiridos pela sessao sao liberados automaticamente (ephemeral nodes).
- **Change notifications**: clientes podem vigiar (watch) chaves e ser notificados quando mudam, sem precisar fazer polling.

Desses blocos, aplicacoes construem:
- **Eleicao de lider**: multiplos nos tentam adquirir um lock em um no do ZooKeeper. O no que adquire o lock e o lider. Se o lider falha, o ephemeral node e deletado e outra eleicao acontece.
- **Service discovery**: nos registram seus enderecos no ZooKeeper. Outros servicos consultam o ZooKeeper para encontrar enderecos.
- **Atribuicao de particoes a nos**: quando o conjunto de nos muda (nos entram ou saem), o mapeamento de particoes para nos precisa mudar. ZooKeeper pode gerenciar essa atribuicao e notificar todos os nos relevantes.

ZooKeeper, etcd e Consul sao usados extensivamente na infraestrutura distribuida. Eles fornecem "outsourced consensus": em vez de cada aplicacao implementar um algoritmo de consenso, aplicacoes usam esses servicos como blocos fundamentais.

---

## Conclusao da Parte II

A Parte II demonstra que distribuir dados entre multiplas maquinas introduz complexidade fundamental. Falhas parciais, atrasos de rede, e dessincronizacao de relogios tornam sistemas distribuidos fundamentalmente mais dificeis de raciocinar do que sistemas em uma unica maquina.

Os capitulos progridem de mecanismos fundamentais (replicacao e particionamento) para desafios conceituais (transacoes, falhas de rede e relogios) ate garantias mais fortes (linearizabilidade e consenso). O ponto crucial e que cada garantia mais forte tem um custo: performance, disponibilidade, ou complexidade operacional. Nao existe solucao universal; o engenheiro deve entender os trade-offs e escolher o modelo apropriado para cada situacao.

Conceitos como consistencia eventual, quoruns, version vectors, snapshot isolation, serializabilidade, linearizabilidade, e consenso formam o vocabulario fundamental para projetar e avaliar sistemas distribuidos. Entender esses conceitos em profundidade -- incluindo suas limitacoes e armadilhas -- e essencial para qualquer engenheiro trabalhando com sistemas de dados modernos.
