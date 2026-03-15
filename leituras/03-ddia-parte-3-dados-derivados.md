# Designing Data-Intensive Applications -- Parte III: Dados Derivados

**Autor:** Martin Kleppmann
**Ano:** 2017
**Editora:** O'Reilly Media

## Visao Geral da Parte III

As duas primeiras partes do livro trataram dos fundamentos de armazenamento e recuperacao de dados (Parte I) e dos desafios de distribuir dados entre multiplas maquinas (Parte II). A Parte III muda o foco para algo igualmente fundamental: como **derivar novos dados a partir de dados existentes**. A distincao central aqui e entre **sistemas de registro** (systems of record), que sao a fonte autoritativa de dados, e **dados derivados** (derived data), que sao o resultado de transformar ou processar dados existentes. Um cache, um indice de busca, um data warehouse e um modelo de machine learning sao todos exemplos de dados derivados -- se voce os perder, pode recria-los a partir dos dados originais.

Essa distincao e crucial porque permite uma arquitetura onde multiplos sistemas especializados coexistem: o banco de dados principal armazena a verdade, enquanto sistemas derivados sao otimizados para padroes de acesso especificos. A Parte III explora as duas grandes familias de mecanismos para criar dados derivados: **processamento em lote** (batch processing) e **processamento de streams** (stream processing).

---

## Capitulo 10 -- Batch Processing (Processamento em Lote)

### Introducao: Tres Tipos de Sistemas

Kleppmann abre o capitulo distinguindo tres categorias amplas de sistemas:

1. **Servicos (online systems):** Esperam por requisicoes de clientes, processam e retornam respostas o mais rapido possivel. A metrica principal e o tempo de resposta (latencia). Exemplos: servidores web, APIs REST.

2. **Sistemas de processamento em lote (batch processing systems):** Recebem um grande volume de dados de entrada, executam um job que os processa, e produzem dados de saida. Nao ha usuario esperando por uma resposta imediata. A metrica principal e o throughput (quantidade de dados processados por unidade de tempo). Exemplos: MapReduce, Spark, jobs ETL noturnos.

3. **Sistemas de processamento de streams (stream processing systems):** Operam sobre eventos a medida que acontecem, com latencia entre a chegada do evento e a producao do resultado. Estao em algum ponto entre online e batch. Exemplos: Apache Kafka Streams, Apache Flink.

O capitulo 10 foca inteiramente no processamento em lote, comecando pela forma mais simples -- ferramentas Unix -- e escalando ate frameworks distribuidos como MapReduce e alem.

### Processamento em Lote com Ferramentas Unix

#### A Filosofia Unix

Kleppmann comeca com um exemplo aparentemente trivial: analisar logs de acesso de um servidor web. Usando uma cadeia de comandos Unix como `awk`, `sort`, `uniq` e `head`, e possivel extrair informacoes como "as 5 URLs mais acessadas" em segundos, mesmo para arquivos de log com gigabytes de dados. Essa eficiencia surpreendente revela principios profundos.

A **filosofia Unix**, articulada por Doug McIlroy em 1964, baseia-se em quatro ideias fundamentais:

1. **Faca cada programa fazer uma coisa bem feita.** Se uma nova tarefa surge, escreva um novo programa em vez de complicar um existente com novas funcionalidades. Isso leva a ferramentas pequenas, focadas e altamente composiveis.

2. **Espere que a saida de todo programa se torne a entrada de outro programa, ainda desconhecido.** Isso significa nao poluir a saida com informacoes estranhas. Evite formatos de entrada rigidos. Evite saida interativa sempre que possivel.

3. **Projete e construa software, mesmo sistemas operacionais, para ser experimentado cedo, idealmente em semanas.** Nao hesite em jogar fora partes desajeitadas e reconstrui-las. Este principio valoriza a iteracao rapida.

4. **Use ferramentas em vez de ajuda nao-qualificada para aliviar uma tarefa de programacao.** Mesmo que precise desviar para construir a ferramenta, e que jogue fora algumas delas depois de terminar de usa-las.

O mecanismo que viabiliza essa composicao e o **pipe (|)** do Unix, que conecta stdout de um processo a stdin de outro. Essa interface uniforme -- um fluxo de bytes -- permite que programas completamente independentes cooperem de maneiras nao previstas por seus criadores.

#### stdin e stdout como Interface Uniforme

A razao pela qual as ferramentas Unix compoem tao bem e que todas compartilham a mesma interface: **stdin** (entrada padrao) e **stdout** (saida padrao). Qualquer programa que le de stdin e escreve em stdout pode ser inserido em qualquer pipeline. Essa e uma forma de **acoplamento frouxo** (loose coupling) -- cada ferramenta nao precisa saber nada sobre a ferramenta antes ou depois dela no pipeline.

Essa abordagem tem um paralelo direto com sistemas distribuidos modernos. Quando voce define uma interface padronizada (um contrato claro entre componentes), voce habilita a composicao e a substituicao de partes. Um programa `sort` nao se importa se seus dados vem de `grep` ou de `cat` -- ele simplesmente le linhas de stdin.

Kleppmann observa que essa interface simples tem limitacoes: funciona apenas para um unico fluxo de entrada e um unico fluxo de saida. Se voce precisa de multiplas entradas ou saidas, precisa recorrer a arquivos ou outros mecanismos.

#### Separacao de Logica e Ligacao (Wiring)

Uma caracteristica elegante da abordagem Unix e a **separacao entre a logica do programa e a ligacao de entrada/saida**. O programa `sort` nao sabe e nao se importa de onde seus dados vem ou para onde vao -- ele simplesmente ordena o que recebe. A decisao de onde conectar a entrada e a saida e feita no momento da invocacao, pelo usuario ou script que monta o pipeline. Isso e analogo ao conceito de **inversao de controle** ou **injecao de dependencia** em engenharia de software -- a logica de negocio e separada da infraestrutura.

#### Transparencia e Experimentacao

Outra virtude do processamento em lote ao estilo Unix e a facilidade de **experimentacao e depuracao**:

- Voce pode executar um pipeline e inspecionar a saida em cada estagio simplesmente encurtando o pipeline.
- Voce pode redirecionar a saida de qualquer estagio para um arquivo e examina-lo.
- Se algo da errado, voce pode reexecutar o pipeline inteiro -- os arquivos de entrada nao sao modificados (imutabilidade).
- Voce pode adicionar ou remover estagios sem afetar os demais.

Essa capacidade de experimentar sem medo e extremamente valiosa, e Kleppmann argumenta que os melhores sistemas de processamento de dados mantiveram essas propriedades.

### MapReduce e Sistemas de Arquivos Distribuidos

#### De Unix para o Mundo Distribuido

As ferramentas Unix funcionam em uma unica maquina. Quando o volume de dados excede a capacidade de uma maquina, precisamos de algo que distribua o processamento entre muitas maquinas, mantendo (idealmente) as virtudes da filosofia Unix. O **MapReduce**, publicado pelo Google em 2004 e implementado de forma open-source pelo projeto **Hadoop**, e essa transicao.

A analogia e direta:

| Unix | MapReduce |
|---|---|
| stdin / arquivos locais | HDFS (sistema de arquivos distribuido) |
| pipe entre processos | Shuffle/Sort entre Map e Reduce |
| Programas individuais (sort, grep, awk) | Funcoes Map e Reduce |
| Um pipeline de comandos | Um job MapReduce (ou cadeia de jobs) |

#### HDFS -- Hadoop Distributed File System

O HDFS e baseado no **Google File System (GFS)**, descrito em um artigo de 2003. Seus principios de design sao:

- **Armazenamento em blocos:** Cada arquivo e dividido em blocos (tipicamente 64 MB ou 128 MB). Blocos grandes reduzem overhead de metadados e permitem transferencias sequenciais eficientes.

- **Replicacao:** Cada bloco e replicado em multiplas maquinas (por padrao, 3 replicas). Isso fornece tolerancia a falhas -- se um disco ou maquina falhar, os dados ainda estao disponiveis em outras copias. Tambem permite **localidade de dados**: o framework pode agendar computacao na maquina que ja possui os dados, evitando transferencia pela rede.

- **Arquitetura NameNode/DataNode:** Um NameNode central mantem os metadados (quais blocos compoem cada arquivo e onde eles estao armazenados). DataNodes armazenam os blocos reais. O NameNode e um ponto unico de falha, mitigado por mecanismos como standby NameNodes e journals compartilhados.

- **Otimizado para throughput, nao latencia:** HDFS e projetado para leitura sequencial de grandes volumes de dados, nao para acesso aleatorio de baixa latencia. Isso o torna ideal para batch processing mas inadequado como banco de dados de producao.

O modelo "shared-nothing" do HDFS -- onde cada maquina usa seus proprios discos locais -- contrasta com abordagens de armazenamento compartilhado (SAN/NAS) e permite escalar para milhares de maquinas com hardware commodity.

#### Como o MapReduce Funciona

Um job MapReduce consiste em quatro fases principais:

**1. Fase Map (Mapeamento):**

O framework le os dados de entrada (do HDFS), divide-os em "splits" (pedacos) e atribui cada split a um **mapper**. O mapper e uma funcao que o programador define. Para cada registro de entrada, o mapper emite zero ou mais pares chave-valor. O mapper nao mantem estado entre registros -- ele processa cada registro independentemente.

Exemplo: Se estamos contando palavras em um corpus de documentos, o mapper recebe uma linha de texto e emite pares como ("hello", 1), ("world", 1), ("hello", 1).

O framework tenta executar o mapper na mesma maquina (ou rack) onde os dados de entrada estao armazenados, explorando a **localidade de dados** para minimizar transferencia de rede. Esse principio -- "mova a computacao para os dados, nao os dados para a computacao" -- e fundamental para o desempenho de sistemas de big data.

**2. Fase Shuffle e Sort (Embaralhamento e Ordenacao):**

Esta e a fase mais complexa e e inteiramente gerenciada pelo framework (o programador nao escreve codigo para ela). O objetivo e agrupar todos os valores que compartilham a mesma chave, de forma que cada reducer receba todos os valores para as chaves que lhe foram atribuidas.

O processo funciona assim:
- A saida de cada mapper e **particionada** por chave. Uma funcao de particionamento (geralmente hash da chave modulo numero de reducers) determina qual reducer recebera cada par chave-valor.
- Os dados sao **ordenados por chave** dentro de cada particao.
- Os dados particionados sao escritos no disco local do mapper.
- Os reducers buscam ("pull") as particoes relevantes de todos os mappers pela rede. Esse passo e chamado de **shuffle** e geralmente e o gargalo do job, pois envolve transferencia massiva de dados pela rede.
- Cada reducer faz um **merge sort** das multiplas particoes recebidas, produzindo um fluxo ordenado por chave.

**3. Fase Reduce (Reducao):**

O reducer recebe uma chave e um iterador sobre todos os valores associados a essa chave. O programador define a funcao reduce que combina esses valores. No exemplo de contagem de palavras, o reducer soma todos os 1s para cada palavra.

O reducer pode emitir zero ou mais registros de saida. A saida e escrita de volta no HDFS.

**4. Saida:**

Os resultados dos reducers sao escritos como arquivos no HDFS. Cada reducer produz um arquivo de saida. Esses arquivos podem ser a entrada de outro job MapReduce, formando uma cadeia de processamento.

#### Execucao Distribuida

O framework MapReduce (como o Hadoop) gerencia automaticamente:

- **Paralelismo:** Multiplos mappers e reducers executam simultaneamente em diferentes maquinas do cluster.
- **Agendamento:** O framework decide quais maquinas executam quais tarefas, levando em conta a localidade de dados.
- **Tolerancia a falhas:** Se um mapper ou reducer falha (crash de maquina, disco corrompido, etc.), o framework reexecuta a tarefa em outra maquina. Isso e possivel porque os dados de entrada sao imutaveis e estao replicados no HDFS. O MapReduce e projetado para tolerar falhas de forma transparente, sem que o programador precise se preocupar com isso.
- **Especulacao:** Se uma tarefa esta demorando muito mais que as outras (um "straggler"), o framework pode lancar uma copia especulativa da mesma tarefa em outra maquina e usar o resultado de quem terminar primeiro.

#### Joins no MapReduce

Uma das operacoes mais importantes e complexas em processamento de dados e o **join** -- combinar registros de diferentes datasets baseado em alguma chave comum. O MapReduce suporta varias estrategias de join, cada uma com diferentes trade-offs.

##### Reduce-Side Joins (Joins no Lado do Reducer)

Nos reduce-side joins, ambos os datasets de entrada sao processados por mappers que emitem a chave de join como chave do MapReduce. O framework shuffle/sort garante que todos os registros com a mesma chave de join cheguem ao mesmo reducer, que entao pode combina-los.

**Sort-Merge Join:**

Esta e a estrategia padrao. Funciona assim:

1. Os mappers processam ambos os datasets. Para cada registro, emitem a chave de join como a chave MapReduce. Os mappers tambem "tagueiam" (marcam) os registros para indicar de qual dataset eles vieram.

2. O framework ordena todos os registros por chave de join (e, dentro da mesma chave, pelo tag de dataset, usando **ordenacao secundaria**). Isso garante que, ao processar uma chave, os registros de um dataset (por exemplo, o "lado um" de uma relacao um-para-muitos) aparecem antes dos registros do outro dataset (o "lado muitos").

3. O reducer itera sobre os registros de cada chave. Para cada chave de join, ele primeiro le o registro do "lado um" e depois itera sobre todos os registros do "lado muitos", combinando-os.

Essa tecnica funciona bem porque toda a complexidade da coordenacao e tratada pela fase shuffle/sort do framework. O programador so precisa definir mappers simples e um reducer que combina os registros agrupados.

**Trazendo Dados Relacionados Para Junto (Bringing Related Data Together):**

O principio fundamental dos reduce-side joins e que o particionamento e a ordenacao do framework garantem que todos os dados relacionados a uma mesma chave de join acabam no mesmo lugar (a mesma chamada de reducer). Isso transforma um problema distribuido (dados espalhados por muitas maquinas) em um problema local (todos os dados relevantes estao disponiveis no mesmo processo).

Essa ideia e profunda e recorrente em sistemas distribuidos: quando voce precisa processar dados relacionados, a maneira mais eficiente e **co-localiza-los** -- colocar todos no mesmo lugar antes de processa-los.

**GROUP BY:**

Operacoes de agrupamento (GROUP BY em SQL) seguem um padrao identico. O mapper emite a chave de agrupamento, e o reducer recebe todos os registros com a mesma chave de agrupamento, podendo calcular agregacoes (soma, contagem, media, etc.).

**Tratamento de Skew (Assimetria de Dados):**

Um problema serio em joins e a **assimetria** (skew ou data hotspot). Em muitos datasets reais, algumas chaves sao muito mais frequentes que outras. Por exemplo, em uma rede social, celebridades podem ter milhoes de seguidores enquanto a maioria dos usuarios tem dezenas. Se todo o processamento de uma "chave quente" vai para um unico reducer, esse reducer se torna um gargalo -- ele demora muito mais que os outros, e o job inteiro espera por ele.

Solucoes para skew:

- **Pig's Skewed Join:** O framework identifica chaves quentes (hot keys) antecipadamente (por amostragem ou configuracao explicita). Para essas chaves, os registros sao distribuidos aleatoriamente entre multiplos reducers, cada um recebendo uma copia completa do outro lado do join. Isso permite processar chaves quentes em paralelo, ao custo de replicar dados.

- **Hive's Skewed Join Tables:** Similar ao Pig, permite especificar quais chaves estao distorcidas e trata-las de forma especial.

- **Amostragem para particionamento:** Em vez de usar hash para particionar, usa-se amostragem dos dados para determinar limites de particao que distribuem a carga de forma mais uniforme.

O tratamento de skew ilustra um tema recorrente no livro: solucoes "ingenuas" frequentemente funcionam para dados uniformes, mas quebram quando encontram distribuicoes enviesadas do mundo real.

##### Map-Side Joins (Joins no Lado do Mapper)

Se certas condicoes sobre os dados de entrada sao satisfeitas, e possivel realizar joins inteiramente na fase map, eliminando a custosa fase shuffle/sort. Esses joins sao significativamente mais rapidos, mas exigem premissas mais fortes.

**Broadcast Hash Join:**

Quando um dos datasets e pequeno o suficiente para caber na memoria de cada mapper, pode-se carregar o dataset pequeno inteiramente em uma tabela hash na memoria. Cada mapper le o dataset pequeno do HDFS (ou de um cache distribuido) e constroi uma tabela hash em memoria. Entao, ao processar cada registro do dataset grande, o mapper consulta a tabela hash para encontrar correspondencias.

Nao ha necessidade de fase reduce -- toda a logica do join acontece no mapper. A palavra "broadcast" refere-se ao fato de que o dataset pequeno e enviado (broadcast) para todos os mappers.

Este join e extremamente eficiente para joins entre uma tabela grande e uma tabela de dimensao pequena (um padrao muito comum em data warehousing, como joins entre uma tabela de fatos e uma tabela de produtos ou clientes).

No ecossistema Hadoop, frameworks como Pig ("replicated join"), Hive ("MapJoin"), Cascading e Crunch suportam essa otimizacao.

**Partitioned Hash Join (Bucket Map Join):**

Se ambos os datasets sao particionados da mesma maneira (mesma chave de particionamento e mesmo numero de particoes), cada mapper precisa carregar apenas a particao correspondente do dataset pequeno, nao o dataset inteiro. Isso reduz drasticamente o uso de memoria.

Por exemplo, se ambos os datasets sao particionados por user_id em 100 particoes, o mapper que processa a particao 37 do dataset grande so precisa carregar a particao 37 do dataset pequeno.

**Map-Side Merge Join:**

Se ambos os datasets sao particionados da mesma maneira E ordenados pela chave de join dentro de cada particao, o mapper pode fazer um merge join (similar ao merge sort) incrementalmente, sem carregar nenhum dataset inteiro na memoria. Isso e o mais eficiente de todos, mas exige que os dados estejam pre-organizados de forma especifica.

Essa tecnica funciona lendo os dois datasets em paralelo, avancando um ou outro conforme as chaves se alinham. E a mesma logica de um merge de dois arrays ordenados, mas aplicada a datasets potencialmente enormes.

#### A Saida do Batch Processing

Os jobs de batch processing tipicamente produzem saidas que sao usadas de diversas formas:

- **Construcao de indices de busca:** O Google originalmente usou MapReduce para construir seus indices de busca. O processo le todos os documentos, extrai termos e cria indices invertidos que mapeiam termos a documentos.

- **Pares chave-valor para consulta:** A saida pode ser um banco de dados somente-leitura (como arquivos de dados para RocksDB ou LevelDB) que e depois servido por servidores de consulta.

- **Entrada para outro job:** A saida de um job frequentemente e a entrada de outro, formando **workflows** de multiplos estagios.

O principio fundamental e que a saida do batch processing e **imutavel** -- uma vez produzida, nao e modificada. Se algo da errado, voce pode reexecutar o job (ja que as entradas tambem sao imutaveis), ou voce pode manter a saida anterior ate ter confianca na nova. Essa filosofia e herdada diretamente da abordagem Unix e e extremamente valiosa para confiabilidade.

### Alem do MapReduce

#### Problemas com Materializacao de Estado Intermediario

Embora o MapReduce tenha sido revolucionario, ele tem deficiencias significativas quando usado para workflows complexos (cadeias de multiplos jobs). O problema principal e a **materializacao de estado intermediario**.

Em um workflow MapReduce, cada job escreve sua saida completa no HDFS antes que o proximo job possa comecar a le-la. Isso significa:

1. **Escrita redundante no HDFS:** Dados intermediarios sao replicados (tipicamente 3 vezes) no HDFS, mesmo que sejam temporarios e so existam para serem lidos pelo proximo job. Isso desperdicea I/O de disco e rede.

2. **Latencia de materializacao:** O proximo job so pode comecar quando o job anterior termina completamente. Mesmo que o proximo job pudesse comecar a processar dados parciais, ele precisa esperar.

3. **Mappers muitas vezes sao redundantes:** Se a saida de um reducer ja esta particionada e ordenada da maneira que o proximo job precisa, o mapper do proximo job e essencialmente uma funcao identidade -- ele le os dados e os reemite sem transformacao. Isso e overhead puro.

Compare com o pipe do Unix, onde os dados fluem incrementalmente de um programa para outro sem serem materializados em disco. O MapReduce perdeu essa propriedade elegante.

#### Dataflow Engines: Spark, Tez e Flink

Para resolver essas limitacoes, surgiu uma nova geracao de frameworks de processamento chamados **dataflow engines** (motores de fluxo de dados). Os mais notaveis sao:

- **Apache Spark:** Desenvolvido na UC Berkeley. Introduziu o conceito de **Resilient Distributed Datasets (RDDs)** -- colecoes imutaveis de dados que podem ser transformadas por operacoes como map, filter, reduce, join, etc. O RDD pode existir em memoria sem ser materializado em disco.

- **Apache Tez:** Desenvolvido pela Hortonworks como um substrato de execucao para Hive e Pig. Permite expressar DAGs complexos de operacoes.

- **Apache Flink:** Originado na TU Berlin. Trata batch processing como um caso especial de stream processing com limites definidos (bounded streams).

Esses frameworks compartilham uma arquitetura fundamentalmente diferente do MapReduce:

1. **Modelo de execucao baseado em DAG (Directed Acyclic Graph):** Em vez de uma sequencia rigida de map-shuffle-sort-reduce, o programador define um **grafo dirigido aciclico** de operacoes. Cada no do grafo e um operador (que pode ser um map, um reduce, um join, um sort, etc.), e as arestas representam fluxo de dados entre operadores.

2. **Eliminacao de shuffle/sort desnecessarios:** O framework so insere etapas de shuffle e sort quando elas sao realmente necessarias (por exemplo, para um join ou group-by), nao a cada "estagio" como no MapReduce.

3. **Pipeline de dados em memoria:** Quando possivel, dados fluem diretamente de um operador para o proximo em memoria, sem serem escritos em disco. Isso e muito mais rapido que a materializacao no HDFS.

4. **Operadores generalizados:** Nao ha distincao rigida entre "mappers" e "reducers". Um operador pode fazer qualquer coisa -- filtrar, transformar, agregar, fazer join. Isso elimina os mappers triviais do MapReduce.

5. **Otimizacao do plano de execucao:** O framework pode reordenar operadores, fundir operadores adjacentes, e escolher diferentes estrategias de join (broadcast, sort-merge, hash) com base nas caracteristicas dos dados.

#### Tolerancia a Falhas: Recomputacao vs. Checkpointing

A tolerancia a falhas e mais complexa em dataflow engines do que no MapReduce, precisamente porque os dados intermediarios nao sao materializados no HDFS.

No MapReduce, se um reducer falha, o framework simplesmente reexecuta a tarefa, lendo os dados intermediarios que ja estao no disco local dos mappers. Se um mapper falha, ele tambem e reexecutado, ja que sua entrada esta no HDFS. A materializacao de todo o estado intermediario, embora custosa, torna a recuperacao simples.

Dataflow engines usam duas estrategias principais:

**Recomputacao (Lineage-based Recovery -- Spark):**

O Spark rastreia a **linhagem** (lineage) de cada RDD -- a sequencia de transformacoes que o produziu a partir dos dados de entrada. Se uma particao de um RDD e perdida (por falha de maquina), o Spark pode recomputar apenas aquela particao, reexecutando a cadeia de transformacoes a partir dos dados de entrada (que estao no HDFS) ou de checkpoints intermediarios.

Vantagem: Em condicoes normais (sem falhas), nao ha overhead de checkpoint.
Desvantagem: Em caso de falha, a recomputacao pode ser cara se a cadeia de transformacoes for longa.

**Checkpointing (Flink):**

O Flink periodicamente cria **snapshots** (checkpoints) do estado de todos os operadores, salvando-os em armazenamento duravel. Em caso de falha, o sistema reverte para o ultimo checkpoint e reprocessa os dados a partir daquele ponto.

Vantagem: Recuperacao rapida e previsivel.
Desvantagem: O checkpointing consome recursos continuamente, mesmo quando nao ha falhas.

A escolha entre recomputacao e checkpointing depende do workload. Para transformacoes simples e rapidas, recomputacao e geralmente preferivel. Para jobs com operacoes caras (como joins grandes), checkpointing pode ser mais eficiente.

#### Grafos e Processamento Iterativo

Muitos algoritmos importantes operam sobre grafos: PageRank, deteccao de comunidades, caminhos mais curtos, etc. Esses algoritmos sao tipicamente **iterativos** -- aplicam repetidamente a mesma operacao ate convergir. O MapReduce e notoriamente ruim para processamento iterativo, porque cada iteracao requer um job completo (com toda a materializacao em HDFS).

**O Modelo Pregel/BSP (Bulk Synchronous Parallel):**

Uma abordagem alternativa e o modelo **Bulk Synchronous Parallel (BSP)**, implementado pelo **Pregel** do Google e por frameworks open-source como **Apache Giraph** e **Spark GraphX**.

No modelo BSP:

1. **Vertice-centrico:** A computacao e expressa da perspectiva de um unico vertice do grafo. Cada vertice tem um estado (por exemplo, sua estimativa atual de PageRank).

2. **Supersteps:** A execucao procede em iteracoes chamadas "supersteps". Em cada superstep:
   - Cada vertice processa as mensagens recebidas no superstep anterior.
   - Cada vertice atualiza seu estado baseado nessas mensagens.
   - Cada vertice envia mensagens para seus vizinhos (ou qualquer outro vertice).

3. **Barreira de sincronizacao:** Todos os vertices devem completar um superstep antes que o proximo comece. Isso simplifica o raciocinio sobre o programa -- dentro de um superstep, vertices operam de forma independente.

4. **Convergencia:** O algoritmo termina quando nenhum vertice envia mensagens em um superstep, indicando que o estado estabilizou.

A eficiencia depende fortemente do particionamento do grafo -- idealmente, vertices que trocam muitas mensagens devem estar na mesma maquina para minimizar comunicacao de rede. O particionamento otimo de grafos e um problema NP-dificil, entao heuristas sao usadas na pratica.

O modelo BSP tem a vantagem de expressar algoritmos iterativos de grafos de forma natural, sem o overhead de inicializar um novo job MapReduce a cada iteracao.

### APIs de Alto Nivel e Linguagens

#### Movimento em Direcao a Linguagens Declarativas

Os primeiros usuarios de MapReduce escreviam programas Java explicitamente definindo funcoes map e reduce. Isso era tedioso e propenso a erros para operacoes comuns como joins e agregacoes. Rapidamente surgiram abstracoes de nivel mais alto:

- **Pig Latin (Apache Pig):** Uma linguagem de fluxo de dados que compila para jobs MapReduce (e depois Tez/Spark). Oferece operadores relacionais como JOIN, GROUP, FILTER, ORDER sem exigir codigo Java.

- **HiveQL (Apache Hive):** Uma linguagem semelhante a SQL que compila para jobs MapReduce/Tez/Spark. Permite que analistas familiarizados com SQL consultem dados no HDFS sem aprender MapReduce.

- **Cascading, Crunch, FlumeJava:** Frameworks Java que oferecem APIs de alto nivel para definir pipelines de dados, internamente traduzidos para MapReduce.

- **Spark SQL, DataFrame API:** APIs que permitem expressar queries declarativamente e se beneficiam de otimizacoes automaticas.

A tendencia e clara: o processamento em lote esta se movendo em direcao a **abstracoes declarativas**, onde o usuario especifica **o que** quer computar e o framework determina **como** executa-lo eficientemente. Isso permite que o framework aplique otimizacoes como:

- Reordenacao de joins para minimizar dados intermediarios.
- Escolha automatica de estrategia de join (broadcast, sort-merge, hash).
- Fusao de operadores para reduzir materializacao.
- Exploracao de indices e estatisticas de dados.

#### Especializacao para Diferentes Dominios

Alem das APIs gerais, surgiram frameworks especializados construidos sobre as mesmas infraestruturas:

- **Mahout, MLlib (Spark):** Machine learning distribuido.
- **GraphX (Spark), Giraph:** Processamento de grafos.
- **SparkR, PySpark:** Integracao com linguagens de analise de dados.

Kleppmann argumenta que essa especializacao e saudavel -- diferentes dominios tem padroes distintos, e APIs especializadas podem oferecer tanto conveniencia ao programador quanto oportunidades de otimizacao para o framework.

### Licoes-Chave do Capitulo 10

1. O processamento em lote tem raizes profundas na filosofia Unix -- componentes simples, composiveis, com interfaces uniformes e dados imutaveis.

2. O MapReduce trouxe essas ideias para o mundo distribuido, mas introduziu overhead significativo com a materializacao de estado intermediario.

3. Dataflow engines como Spark e Flink superaram o MapReduce ao permitir grafos arbitrarios de operacoes com dados fluindo em memoria.

4. A tolerancia a falhas em batch processing e viabilizada pela imutabilidade dos dados de entrada -- voce sempre pode recomputar.

5. Joins sao a operacao mais desafiadora em processamento distribuido, com multiplas estrategias que envolvem trade-offs entre uso de memoria, transferencia de rede e requisitos sobre os dados.

6. A tendencia e em direcao a APIs declarativas que permitem ao framework otimizar a execucao automaticamente.

---

## Capitulo 11 -- Stream Processing (Processamento de Streams)

### Introducao

O processamento em lote opera sobre um conjunto **limitado** (bounded) de dados -- voce tem todos os dados de entrada disponivel quando o job comeca. Mas na realidade, dados chegam continuamente: usuarios clicam em links, sensores reportam medicoes, transacoes financeiras acontecem a cada segundo. Esperar ate o final do dia para processar os dados em um job batch introduz atraso inaceitavel para muitos casos de uso.

O **processamento de streams** opera sobre dados **ilimitados** (unbounded) -- um fluxo contínuo de eventos que nao tem um fim definido. Um evento (event) e tipicamente um registro pequeno e imutavel com um timestamp, contendo informacao sobre algo que aconteceu. Eventos sao gerados por um **produtor** (producer/publisher/sender) e processados por **consumidores** (consumers/subscribers/recipients).

### Transmitindo Streams de Eventos

#### Sistemas de Mensagens (Messaging Systems)

A forma mais direta de entregar eventos de produtores a consumidores e atraves de um **sistema de mensagens**. Mas ha muitas abordagens com trade-offs muito diferentes.

**Mensagens Diretas (Direct Messaging):**

Abordagens como UDP multicast, brokerless messaging libraries (ZeroMQ), ou mesmo HTTP/RPC direta entre produtores e consumidores. Vantagens: baixa latencia, sem intermediario. Desvantagens:

- Se o consumidor esta offline ou sobrecarregado, mensagens podem ser perdidas.
- O produtor precisa saber quem sao os consumidores (acoplamento).
- Sem buffer para absorver picos de carga.
- Sem possibilidade de replay (reprocessar mensagens antigas).

**Message Brokers / Message Queues:**

Um intermediario (broker) como **RabbitMQ**, **ActiveMQ**, **Amazon SQS** recebe mensagens de produtores, armazena-as temporariamente, e as entrega a consumidores. O broker desacopla produtores de consumidores e absorve picos de carga.

Diferencas fundamentais em relacao a bancos de dados:

- Brokers normalmente **deletam mensagens** apos entrega-las (o dado e transitorio, nao permanente).
- A maioria dos brokers assume que o **working set e pequeno** -- as filas devem ser curtas. Se consumidores estao lentos e a fila cresce, o desempenho degrada.
- Brokers suportam **assinatura por topico** (topic subscription), nao queries ad-hoc.
- Brokers **nao suportam indices** no conteudo das mensagens.

#### Multiplos Consumidores

Quando multiplos consumidores estao inscritos no mesmo topico, ha dois padroes principais:

**Load Balancing (Balanceamento de Carga):**

Cada mensagem e entregue a **exatamente um** consumidor (dentro de um grupo). Isso permite paralelizar o processamento -- se o volume de mensagens e alto, voce adiciona mais consumidores ao grupo. O broker distribui mensagens entre consumidores (por exemplo, round-robin ou baseado em particao).

**Fan-out:**

Cada mensagem e entregue a **todos** os consumidores. Cada consumidor recebe uma copia independente do stream completo. Isso permite que diferentes consumidores facam coisas diferentes com os mesmos dados -- por exemplo, um consumidor atualiza um cache, outro atualiza um indice de busca, outro envia notificacoes.

Esses dois padroes podem ser combinados: multiplos grupos de consumidores (fan-out entre grupos), com balanceamento de carga dentro de cada grupo.

#### Acknowledgments e Reentrega

Quando um consumidor pode falhar antes de processar uma mensagem completamente, como garantir que a mensagem nao seja perdida? O mecanismo padrao e o **acknowledgment** (ack):

1. O broker envia a mensagem ao consumidor.
2. O consumidor processa a mensagem.
3. O consumidor envia um ack ao broker, confirmando o processamento.
4. Somente apos receber o ack, o broker remove a mensagem (ou marca como processada).
5. Se o broker nao recebe o ack dentro de um timeout, ele reenvia a mensagem (possivelmente para outro consumidor).

Um problema sutil e a **reordenacao de mensagens**. Considere:

- Mensagem A e enviada ao consumidor 1.
- Mensagem B e enviada ao consumidor 2.
- Consumidor 2 processa B e envia ack.
- Consumidor 1 falha. Mensagem A e reenviada ao consumidor 2.

O resultado e que o consumidor 2 processou B antes de A, mesmo que A tenha sido enviada primeiro. Se a ordem importa, isso e um problema. Message brokers tradicionais geralmente so garantem ordem dentro de uma mesma fila/particao.

#### Message Brokers Baseados em Log

**Apache Kafka** e o exemplo canônico. Combina a durabilidade de um log (como um commit log de banco de dados) com a funcionalidade de um message broker.

**Como funciona:**

- Um **topico** e dividido em **particoes** (partitions). Cada particao e um log append-only armazenado em disco.
- Cada mensagem dentro de uma particao recebe um **offset** sequencial monotonicamente crescente.
- Produtores escrevem no final de uma particao. Consumidores leem a partir de um offset e avancam sequencialmente.
- Cada particao pode estar em uma maquina diferente, permitindo escalabilidade horizontal.

**Vantagens sobre message brokers tradicionais:**

1. **Throughput muito alto:** Escrita sequencial em disco (append-only) e extremamente rapida. Leitura sequencial tambem.

2. **Durabilidade:** Mensagens sao replicadas entre multiplos brokers (similar ao HDFS). Nao sao deletadas apos consumo -- sao retidas por um periodo configuravel (dias, semanas) ou por limite de tamanho.

3. **Replay:** Um consumidor pode voltar e reler mensagens antigas simplesmente reposicionando seu offset. Isso e revolucionario -- permite reprocessar dados quando ha bugs, quando novos consumidores sao adicionados, ou para fins de auditoria.

4. **Ordem garantida dentro de uma particao:** Como cada particao e um log sequencial, a ordem das mensagens dentro de uma particao e preservada perfeitamente.

5. **Consumer offsets:** O progresso de cada consumidor e rastreado apenas por um offset (um numero inteiro). Isso e muito mais eficiente que rastrear acks individuais para cada mensagem.

**Logs vs. Mensageria Tradicional:**

Kleppmann compara os dois modelos detalhadamente:

| Aspecto | Broker Tradicional (RabbitMQ) | Log-based (Kafka) |
|---|---|---|
| Modelo de entrega | Mensagem deletada apos ack | Mensagem retida apos consumo |
| Ordem | Nao garantida entre consumidores | Garantida dentro de particao |
| Replay | Nao possivel | Possivel (rebobinar offset) |
| Paralelismo | Numero arbitrario de consumidores | Limitado pelo numero de particoes |
| Overhead por mensagem | Alto (tracking de ack individual) | Baixo (apenas offset) |
| Caso de uso ideal | Muitas filas pequenas, processamento individual | Alto throughput, multiplos consumidores, event sourcing |

Uma limitacao importante do modelo log-based: o numero de consumidores paralelos em um grupo e limitado pelo numero de particoes (cada particao e atribuida a exatamente um consumidor em um grupo). Se voce precisa de mais paralelismo, precisa de mais particoes. Alem disso, se o processamento de uma unica mensagem e muito lento (por exemplo, envolve uma chamada de rede), o modelo log-based pode ser menos eficiente que um broker tradicional que permite maior fanout.

**Consumer Offsets:**

O offset do consumidor funciona como um **bookmark** -- marca ate onde o consumidor ja leu. O consumidor periodicamente comita (commit) seu offset (para o Kafka ou para um armazenamento externo). Se o consumidor reiniciar, ele retoma do ultimo offset comitado.

Isso cria uma semantica de **at-least-once** por padrao: se o consumidor processa uma mensagem mas falha antes de comitar o offset, a mensagem sera reprocessada apos o reinicio. Para **exactly-once**, sao necessarios mecanismos adicionais (discutidos na secao de tolerancia a falhas).

### Bancos de Dados e Streams

Kleppmann faz uma observacao profunda: ha uma conexao fundamental entre bancos de dados e streams de eventos. Um banco de dados pode ser visto como o resultado acumulado de um stream de eventos (insercoes, atualizacoes, delecoes) aplicados a um estado. O **log de escrita** (write-ahead log, WAL) de um banco de dados e, literalmente, um stream de eventos.

#### Mantendo Sistemas em Sincronia

Na maioria das aplicacoes, dados precisam estar em multiplos sistemas: o banco de dados principal, um cache (Redis), um indice de busca (Elasticsearch), um data warehouse. Manter esses sistemas sincronizados e surpreendentemente dificil:

- **Escrita dupla (dual writes):** A aplicacao escreve em ambos os sistemas. Problema: se a escrita no banco de dados funciona mas a escrita no indice de busca falha, os sistemas ficam inconsistentes. Transacoes distribuidas (2PC) poderiam resolver, mas sao lentas e frageis.

- **Batch ETL:** Um job periodico extrai dados do banco de dados e recria o indice/cache. Problema: os dados derivados ficam desatualizados entre execucoes do job.

Change Data Capture oferece uma solucao mais elegante.

#### Change Data Capture (CDC)

CDC e o processo de **observar todas as alteracoes feitas em um banco de dados e extrair essas alteracoes em um formato que outros sistemas possam consumir**, tipicamente em tempo quase-real.

**Implementando CDC:**

A forma mais elegante de implementar CDC e usando o **log de replicacao** do banco de dados. A maioria dos bancos de dados ja mantem um log detalhado de todas as alteracoes para fins de replicacao e recuperacao:

- **PostgreSQL:** Logical replication / logical decoding permite extrair alteracoes em formato estruturado.
- **MySQL:** binlog pode ser lido por ferramentas como **Debezium** ou **Maxwell**.
- **MongoDB:** oplog pode ser "tailed" (lido continuamente).
- **Oracle, SQL Server:** Tem funcionalidades nativas de CDC.

Ferramentas como **Debezium**, **LinkedIn's Databus**, **Facebook's Wormhole**, e **Yahoo's Sherpa** implementam CDC para diversos bancos de dados, publicando alteracoes em topicos Kafka.

**Snapshot Inicial (Initial Snapshot):**

Quando um novo consumidor de CDC comeca, ele precisa de uma copia completa dos dados atuais do banco (o log nao contem todo o historico, apenas alteracoes recentes). O procedimento tipico e:

1. Tirar um snapshot consistente do banco de dados.
2. Registrar a posicao no log correspondente ao snapshot.
3. Carregar o snapshot no sistema de destino.
4. Comecar a consumir o log de alteracoes a partir da posicao registrada.

Isso e analogo ao processo de adicionar uma nova replica a um banco de dados.

**Log Compaction (Compactacao de Log):**

Para evitar a necessidade de snapshots separados, pode-se usar **compactacao de log** (como o Kafka suporta). A compactacao mantem apenas a versao mais recente de cada chave no log, descartando versoes anteriores. Isso significa que o log compactado contem um snapshot completo dos dados atuais, seguido de alteracoes em tempo real.

Com log compaction, um novo consumidor pode simplesmente ler o log compactado desde o inicio para obter o estado completo, sem precisar de um snapshot separado.

#### Event Sourcing

**Event sourcing** e um padrao de design de aplicacao (popularizado pela comunidade Domain-Driven Design) que leva a ideia de CDC ao extremo. Em vez de armazenar o estado atual (como um banco de dados convencional), voce armazena **todos os eventos que levaram ao estado atual**. O estado e derivado aplicando os eventos em ordem.

Exemplos:

- Em vez de armazenar "saldo da conta = R$ 1000", armazena-se "deposito de R$ 500", "deposito de R$ 800", "saque de R$ 300". O saldo e derivado somando os eventos.
- Em vez de atualizar "endereco do cliente = Rua Nova", armazena-se "cliente mudou endereco para Rua Nova em 2025-01-15". O endereco atual e derivado do evento mais recente de mudanca de endereco.

**Diferencas entre CDC e Event Sourcing:**

- **CDC** opera no nivel do banco de dados -- extrai alteracoes (INSERT, UPDATE, DELETE) do log de replicacao. Os eventos sao de baixo nivel (linhas de tabela).
- **Event sourcing** opera no nivel da aplicacao -- os eventos representam acoes de dominio significativas (como "PedidoRealizado", "ItemAdicionadoAoCarrinho"). Os eventos sao de alto nivel e expressivos.
- Em event sourcing, os eventos sao o **registro primario**; views materializadas do estado atual sao derivados. Em CDC, o banco de dados relacional e o registro primario, e o stream de alteracoes e derivado.

**Commands vs. Events:**

Uma distincao sutil mas importante:

- Um **command** e uma requisicao que pode ser rejeitada (por exemplo, "adicionar item ao carrinho" pode ser rejeitado se o item esta fora de estoque). O command precisa ser validado.
- Um **event** e um fato que ja aconteceu e nao pode ser rejeitado. "Item adicionado ao carrinho" e um evento -- ja aconteceu.

No event sourcing, a transicao de command para event deve ser sincrona e atomica: a validacao ocorre, e se aprovada, o evento e registrado. Depois disso, os consumidores processam o evento de forma assincrona, atualizando views derivadas.

#### Estado, Streams e Imutabilidade

Kleppmann faz uma observacao filosofica profunda: todo estado mutavel pode ser visto como a **projecao** de um log imutavel de eventos. Um banco de dados relacional e apenas uma view materializada do log de insercoes, atualizacoes e delecoes que o produziu.

Essa perspectiva tem consequencias praticas:

**Vantagens de Eventos Imutaveis:**

1. **Auditabilidade:** Voce tem um registro completo de tudo que aconteceu. Se um cliente fez um pedido e depois cancelou, ambos os eventos estao registrados. No modelo mutavel, o cancelamento simplesmente apaga o pedido, perdendo a informacao de que o pedido existiu.

2. **Depuracao:** Quando algo da errado, voce pode "rebobinar" e ver o que aconteceu passo a passo.

3. **Multiplas views derivadas:** A partir do mesmo log de eventos, voce pode criar diferentes views otimizadas para diferentes padroes de acesso. Um e-commerce pode ter uma view otimizada para busca de produtos, outra para recomendacoes, outra para analise de vendas -- todas derivadas do mesmo log de eventos.

4. **Separacao de escrita e leitura (CQRS):** A forma como dados sao escritos (como eventos) pode ser completamente diferente da forma como sao lidos (como views materializadas). Isso permite otimizar cada lado independentemente.

5. **Evolucao do esquema:** Se voce precisa mudar a estrutura dos dados, pode criar uma nova view derivada sem modificar os eventos originais.

**Controle de Concorrencia:**

Com events sourcing, o log de eventos pode ser usado como mecanismo de serializacao. Se todos os eventos passam por um unico log (ou um log particionado por alguma chave), a ordem dos eventos e bem definida, eliminando muitos problemas de concorrencia.

**Limitacoes da Imutabilidade:**

1. **Volume de armazenamento:** Manter todos os eventos para sempre pode exigir armazenamento massivo. Compactacao de log e retencao baseada em tempo podem mitigar isso, mas ao custo de perder parte do historico.

2. **Direito ao esquecimento (GDPR, LGPD):** Se um usuario exerce seu direito de ter dados apagados, voce nao pode simplesmente manter os eventos para sempre. Abordagens incluem criptografia (criptografar dados pessoais com uma chave por usuario e destruir a chave) ou tombstones (marcadores de exclusao).

3. **Workloads com alta taxa de atualizacao:** Para sistemas onde o mesmo registro e atualizado milhoes de vezes (como um contador de visualizacoes), manter cada atualizacao como evento individual e impratico. Compactacao frequente e essencial.

### Processamento de Streams

Agora que estabelecemos como streams sao transmitidos e armazenados, Kleppmann discute o que se pode **fazer** com eles.

#### Usos do Processamento de Streams

**Complex Event Processing (CEP):**

CEP e uma abordagem onde voce define **padroes** de eventos e o sistema busca esses padroes no stream. A relacao entre consulta e dados e invertida em relacao a bancos de dados:

- Em um banco de dados, os dados sao armazenados e consultas sao transitorias.
- Em CEP, as consultas (padroes) sao armazenadas e os dados (eventos) sao transitorios -- passam pelo sistema e sao testados contra os padroes.

Exemplo: "alerte quando um paciente teve tres leituras de pressao arterial alta em um intervalo de 10 minutos" e um padrao CEP.

Ferramentas: **Esper**, **IBM InfoSphere Streams**, **Apama**.

**Stream Analytics:**

Menos focado em padroes especificos e mais em **agregacoes e metricas** continuas:

- Taxa de eventos por segundo (throughput monitoring).
- Media movel de um valor ao longo de uma janela de tempo.
- Percentis de latencia nos ultimos 10 minutos.
- Contagem de usuarios unicos por hora.

Ferramentas: **Apache Storm**, **Spark Streaming**, **Flink**, **Kafka Streams**.

**Busca em Streams (Search on Streams):**

O inverso de busca em bancos de dados: em vez de indexar documentos e buscar com queries, voce indexa queries e busca com documentos. Cada novo evento e testado contra todas as queries armazenadas.

Exemplo: Um usuario quer ser notificado quando um imovel que atende seus criterios e listado. A query do usuario e armazenada, e cada novo anuncio (evento) e testado contra todas as queries de usuarios.

Essa abordagem e usada em sistemas de alerta, monitoramento e "prospective search" (Elasticsearch Percolator).

**Message Passing e Atores:**

O modelo de atores (Akka, Erlang/OTP, Orleans) e um modelo de programacao concorrente onde a comunicacao entre componentes se da por troca de mensagens assincronas. Embora use infraestrutura similar (message brokers), a intencao e diferente: o foco e na comunicacao entre componentes de uma aplicacao, nao no processamento de dados.

Diferencas em relacao a stream processing:
- Atores geralmente comunicam-se de forma unidirecional, sem garantia de ordem global.
- O modelo de atores e uma abstraccao de programacao; stream processing e um paradigma de processamento de dados.

#### Raciocinando sobre Tempo

O tempo e um dos aspectos mais traicoeiros do processamento de streams.

**Event Time vs. Processing Time:**

- **Event time:** O momento em que o evento realmente aconteceu (registrado pelo produtor).
- **Processing time:** O momento em que o evento e processado pelo sistema.

Esses dois tempos podem ser significativamente diferentes. Exemplos:

- Um dispositivo movel gera eventos offline e so os envia quando se reconecta -- a diferenca pode ser horas ou dias.
- Picos de carga podem causar atrasos no processamento de minutos.
- Eventos podem chegar fora de ordem se passam por caminhos de rede diferentes.

Usar processing time para logica de negocios (como "contar eventos por hora") pode produzir resultados errados: se o sistema ficou sobrecarregado e processou um backlog de eventos de 3 horas em 30 minutos, a contagem baseada em processing time estaria distorcida.

A recomendacao e usar **event time** para logica de negocios e **processing time** apenas para monitoramento do proprio sistema de processamento.

**Janelas de Tempo (Windows):**

Para computar agregacoes sobre streams ilimitados, precisamos definir **janelas** finitas. Ha quatro tipos principais:

1. **Tumbling Window (Janela Tombante):** Janelas fixas que nao se sobrepoem. Exemplo: "contar eventos a cada minuto" produz janelas [00:00-01:00), [01:00-02:00), etc. Cada evento pertence a exatamente uma janela.

2. **Hopping Window (Janela Saltitante):** Janelas fixas que se sobrepoem. Definida por tamanho e intervalo de avanço. Exemplo: janelas de 5 minutos com avanco de 1 minuto produz janelas [00:00-05:00), [01:00-06:00), [02:00-07:00), etc. Cada evento pertence a multiplas janelas. Uma janela tombante e um caso especial onde tamanho = intervalo.

3. **Sliding Window (Janela Deslizante):** Contem todos os eventos dentro de um intervalo de tempo de qualquer outro evento no stream. Exemplo: "todos os eventos nos ultimos 5 minutos de qualquer evento". Isso produz uma janela para cada combinacao possivel de eventos. Mais flexivel mas potencialmente mais caro.

4. **Session Window (Janela de Sessao):** Agrupa eventos do mesmo usuario que estao proximos no tempo, sem atividade intermediaria. Nao tem tamanho fixo -- depende do padrao de atividade do usuario. Exemplo: uma sessao termina apos 30 minutos de inatividade. Sessoes sao particularmente uteis para analisar comportamento de usuarios em websites e apps.

**O Problema de Eventos Atrasados (Stragglers):**

Como lidar com eventos que chegam depois que a janela a que pertencem ja foi "fechada" e seus resultados emitidos? Opcoes:

1. **Ignorar:** Descartar eventos atrasados. Simples, mas pode perder dados.
2. **Publicar correcao:** Emitir um resultado atualizado quando eventos atrasados chegam. Consumidores precisam lidar com atualizacoes.
3. **Watermarks:** Um mecanismo (usado extensivamente no Flink e no Google Cloud Dataflow) que estima quando todos os eventos de uma janela ja chegaram. O watermark e um timestamp que diz "acredito que todos os eventos com timestamp anterior a este ja foram recebidos". Eventos que chegam apos o watermark sao tratados como atrasados.

Os watermarks sao necessariamente heuristicos -- nao ha como saber com certeza se todos os eventos chegaram. Watermarks muito agressivos (avancando rapido) podem descartar eventos legitimamente atrasados. Watermarks muito conservadores (avancando devagar) introduzem latencia desnecessaria.

#### Joins em Streams

Joins sao complicados em batch processing; em stream processing, sao ainda mais complexos porque os dados nao sao limitados.

**Stream-Stream Join (Window Join):**

Combina eventos de dois streams baseado em alguma chave, dentro de uma janela de tempo.

Exemplo: Voce tem um stream de buscas de usuarios e um stream de cliques em resultados de busca. Voce quer juntar cada clique com a busca que o originou. Ambos os eventos tem um session_id. O join busca, para cada clique, a busca com o mesmo session_id que ocorreu nos ultimos 30 minutos.

Implementacao: O processador mantem estado (em memoria ou em disco) para ambos os streams, indexado pela chave de join. Quando um novo evento chega, ele e indexado e verificado contra o outro stream. Eventos que passam da janela de tempo sao descartados.

**Stream-Table Join (Enrichment):**

Combina um evento do stream com dados de uma tabela (banco de dados).

Exemplo: Para cada evento de compra, adicionar informacoes do produto (nome, categoria, preco) de um banco de dados de produtos.

Abordagens:
- **Consulta ao banco a cada evento:** Simples, mas lento e sobrecarrega o banco.
- **Cache local:** Manter uma copia local da tabela (ou das partes relevantes) no processador de stream. O cache pode ser populado via CDC -- um stream de alteracoes da tabela atualiza o cache continuamente.

Com a abordagem de cache via CDC, o stream-table join se reduz a dois stream-stream joins: um com o stream de eventos de negocios e outro com o stream de CDC da tabela.

**Table-Table Join (Manutencao de View Materializada):**

Combina alteracoes em duas tabelas para manter uma view materializada atualizada.

Exemplo: Em uma rede social, cada usuario tem um timeline que mostra posts de pessoas que ele segue. Quando um usuario publica um post, ou quando alguem segue/deixa de seguir alguem, a timeline precisa ser atualizada. Isso e um join entre a "tabela" de relacoes de seguimento e a "tabela" de posts.

Este e um dos padroes mais poderosos: manter caches e views derivadas automaticamente atualizadas em tempo real.

#### Tolerancia a Falhas em Stream Processing

A tolerancia a falhas e mais desafiadora em stream processing do que em batch processing. Em batch, se algo falha, voce simplesmente reexecuta o job inteiro. Em stream processing, o "job" nunca termina -- voce nao pode simplesmente "reexecutar desde o inicio".

**Microbatching:**

Usado pelo **Spark Streaming**. Divide o stream em pequenos lotes (micro-batches) de, por exemplo, 1 segundo cada. Cada micro-batch e processado como um job batch em miniatura, com as mesmas garantias de tolerancia a falhas (reexecucao em caso de falha).

Vantagem: Reutiliza o mecanismo de tolerancia a falhas do batch processing.
Desvantagem: A latencia minima e limitada pelo tamanho do micro-batch. Para latencia abaixo de 1 segundo, microbatching nao e adequado.

**Checkpointing:**

Usado pelo **Flink**. Periodicamente salva um snapshot consistente do estado de todos os operadores. Em caso de falha, reverte para o ultimo checkpoint e reprocessa os eventos a partir daquele ponto.

O Flink usa um algoritmo elegante baseado em **barreiras** (barrier) inspirado no algoritmo de snapshot de Chandy-Lamport. Barreiras sao injetadas no stream e fluem com os dados. Quando um operador recebe barreiras de todas as suas entradas, ele faz um checkpoint de seu estado.

**Transacoes Atomicas:**

Usar transacoes distribuidas (como 2PC) para garantir que todas as saidas e atualizacoes de estado sao atomicas. Se algo falha, tudo e revertido.

Desvantagem: Overhead significativo de coordenacao, reduzindo throughput e aumentando latencia.

**Idempotencia:**

Uma abordagem pragmatica: permitir que eventos sejam processados mais de uma vez, mas garantir que o resultado final e o mesmo. Por exemplo, se a operacao e "definir valor como X" (em vez de "incrementar valor em 1"), processar o evento duas vezes produz o mesmo resultado.

Para operacoes nao-naturalmente-idempotentes (como incrementos), pode-se usar um **identificador de operacao**: cada operacao tem um ID unico, e o consumidor verifica se ja processou aquele ID antes de aplica-lo. Isso transforma qualquer operacao em idempotente.

**Reconstruindo Estado Apos Falha:**

Se o operador mantem estado local (por exemplo, uma tabela hash para um join), esse estado precisa ser reconstruido apos uma falha. Opcoes:

1. **Reprocessar do stream:** Se o estado foi construido a partir do stream (por exemplo, via CDC), reprocessar o stream reconstroi o estado. Isso pode ser lento se o stream e longo.
2. **Checkpoint do estado:** Salvar periodicamente o estado em armazenamento duravel (HDFS, S3). Recuperacao rapida, mas com overhead continuo.
3. **Armazenamento local replicado:** Usar armazenamento local que sobrevive a reinicializacoes de processo (como RocksDB com log de escrita).

### Licoes-Chave do Capitulo 11

1. Stream processing preenche o gap entre batch processing (alta latencia, alta corretude) e servicos online (baixa latencia, processamento individual).

2. Log-based message brokers (Kafka) combinam o melhor de brokers tradicionais (desacoplamento, durabilidade) com o melhor de logs de banco de dados (ordenacao, replay, retencao).

3. CDC e event sourcing representam uma mudanca de paradigma: em vez de tratar o banco de dados como o centro do universo, trate o log de eventos como a fonte de verdade e bancos de dados como views derivadas.

4. A imutabilidade de eventos e uma propriedade poderosa que habilita auditabilidade, depuracao e multiplas views derivadas.

5. Raciocinar sobre tempo em stream processing e fundamentalmente dificil. A distincao entre event time e processing time e crucial.

6. Tolerancia a falhas em streams requer mecanismos especificos (microbatching, checkpointing, idempotencia) porque nao ha o luxo de "reexecutar tudo".

---

## Capitulo 12 -- The Future of Data Systems (O Futuro dos Sistemas de Dados)

### Introducao

O capitulo final sintetiza todo o livro, conectando os temas de armazenamento, replicacao, particionamento, transacoes, batch processing e stream processing em uma visao coerente de como sistemas de dados devem (e podem) ser construidos. E tambem o capitulo mais opinativo, onde Kleppmann argumenta por certas abordagens arquiteturais e levanta questoes eticas.

### Integracao de Dados

#### Combinando Ferramentas Especializadas

Uma licao recorrente do livro e que **nenhuma ferramenta atende a todos os requisitos**. Bancos de dados relacionais sao excelentes para consultas flexiveis e transacoes ACID, mas nao para busca full-text. Elasticsearch e excelente para busca, mas nao para transacoes. Redis e excelente para cache e acesso rapido, mas nao para durabilidade de longo prazo.

A realidade e que a maioria das aplicacoes usa **multiplos sistemas de dados** -- um banco principal, um cache, um indice de busca, um data warehouse. O desafio e mante-los sincronizados.

#### Raciocinando sobre Fluxo de Dados

**Dados Derivados vs. Transacoes Distribuidas:**

Ha duas abordagens para manter multiplos sistemas em sincronia:

1. **Transacoes distribuidas (2PC):** Todas as escritas sao atomicas -- ou todas acontecem ou nenhuma. Garante consistencia forte, mas e lento, fragil e dificil de escalar. Na pratica, poucos sistemas suportam transacoes distribuidas entre sistemas heterogeneos.

2. **Dados derivados via log ordenado:** Um unico sistema e a fonte de verdade. Alteracoes sao propagadas para outros sistemas via um log ordenado (como um topico Kafka). Cada sistema consumidor processa as alteracoes de forma deterministica, eventualmente convergindo para o mesmo estado.

Kleppmann argumenta fortemente pela segunda abordagem. Ela e mais escalavel, mais tolerante a falhas e mais flexivel. A consistencia e eventual (nao instantanea), mas para a maioria dos casos de uso, isso e aceitavel.

**Ordenacao de Eventos:**

A ordenacao e crucial. Se dois eventos conflitantes sao processados em ordens diferentes por sistemas diferentes, eles divergem. Um log totalmente ordenado (como uma particao Kafka) garante que todos os consumidores veem os eventos na mesma ordem.

Para obter ordenacao total em escala, usa-se particionamento: o log e dividido em particoes, e a ordenacao total so e garantida dentro de cada particao. Eventos que precisam ser ordenados entre si devem ir para a mesma particao (tipicamente particionados pela mesma chave, como user_id).

**Ordem Total com Logs:**

Kleppmann descreve como usar um log totalmente ordenado como mecanismo de serializacao:

1. Todas as escritas sao registradas em um log.
2. O log atribui um numero de sequencia a cada escrita.
3. Todos os consumidores processam as escritas na ordem do log.
4. Se todos os consumidores sao deterministicos, todos convergem para o mesmo estado.

Isso e fundamentalmente o que um banco de dados faz internamente (com seu write-ahead log), mas aplicado entre multiplos sistemas.

#### Batch e Stream Processing

**Mantendo Estado Derivado:**

Batch processing e stream processing sao complementares:

- **Batch processing** e bom para reprocessar grandes volumes de dados historicos, por exemplo, para construir um novo indice ou corrigir um bug em uma transformacao anterior.
- **Stream processing** e bom para manter o estado derivado atualizado em tempo quase-real a medida que novos dados chegam.

**Reprocessando Dados:**

Uma das vantagens mais poderosas de um log de eventos e a capacidade de **reprocessar** dados. Se voce descobre um bug na logica de processamento, pode corrigir o codigo e reprocessar todos os eventos do log para produzir resultados corretos. Essa abordagem trata o codigo como uma funcao que transforma a entrada (log de eventos) em saida (estado derivado), e quando a funcao muda, voce simplesmente a reaplicar.

**Arquitetura Lambda:**

A **lambda architecture**, proposta por Nathan Marz, combina batch e stream processing:

1. Uma camada batch reprocessa periodicamente todos os dados historicos para produzir views "corretas" (mas com atraso).
2. Uma camada stream processa eventos recentes para produzir views "rapidas" (mas potencialmente aproximadas).
3. As consultas combinam resultados de ambas as camadas.

Problemas da lambda architecture:
- Manter duas bases de codigo (uma batch, uma stream) para a mesma logica e um fardo operacional e uma fonte de bugs.
- Combinar resultados de ambas as camadas e complexo.

**Unificando Batch e Stream:**

Frameworks modernos como **Flink** e **Kafka Streams** propoe unificar batch e stream, tratando batch como um caso especial de stream (um stream com limite definido). Isso permite usar o mesmo codigo para processamento historico e em tempo real, eliminando os problemas da lambda architecture.

A abordagem de Kleppmann: use um log de eventos como fonte de verdade. Para novos dados, processe-os em tempo real via stream processing. Para reprocessamento, "rebobine" o log e reprocesse tudo com o mesmo codigo. Isso e chamado de **kappa architecture** por Jay Kreps (criador do Kafka).

### Desagregando Bancos de Dados (Unbundling Databases)

#### Compondo Tecnologias de Armazenamento de Dados

Kleppmann faz uma analogia provocante: o que um banco de dados relacional moderno faz internamente -- manter indices, views materializadas, triggers, replicacao -- e conceitualmente similar ao que estamos fazendo quando combinamos multiplos sistemas de dados com CDC e stream processing.

**Criando um Indice como Dado Derivado:**

Quando voce cria um indice em um banco de dados, o banco:
1. Escaneia todos os dados existentes e constroi o indice.
2. A partir daquele ponto, mantem o indice atualizado conforme dados sao inseridos, atualizados e deletados.

Isso e exatamente o que acontece quando voce configura CDC para popular um indice Elasticsearch: snapshot inicial + stream de atualizacoes.

**O "Meta-Database":**

Kleppmann propoe pensar na combinacao de todas as ferramentas de dados de uma aplicacao como um unico **meta-database** -- um banco de dados logico composto por multiplos componentes especializados. O log de eventos e como o WAL desse meta-database, e os diferentes sistemas de armazenamento (banco relacional, cache, indice de busca) sao como diferentes indices mantidos pelo meta-database.

**Federacao vs. Desagregacao:**

Duas abordagens para combinar multiplos sistemas:

- **Federacao (federated database / polystore):** Uma camada unificada de consulta sobre multiplos sistemas, permitindo consultar todos como se fossem um so. O desafio e fazer queries eficientes que abrangem sistemas heterogeneos.

- **Desagregacao (unbundled database):** Em vez de uma interface de consulta unificada, foca em sincronizacao de dados confiavel entre sistemas. Cada sistema mantem sua propria interface de consulta, mas os dados fluem entre eles de forma consistente via log de eventos.

Kleppmann favorece a desagregacao, argumentando que ela e mais pratica e mais flexivel. Sistemas especializados ja existem e sao bons no que fazem -- o desafio e apenas mante-los sincronizados.

**O Que Esta Faltando:**

Kleppmann observa que ainda nao temos boas abstracoes de "banco de dados desagregado". Configurar CDC, conectar Kafka a Elasticsearch, gerenciar schemas, lidar com falhas -- tudo isso ainda exige muito trabalho manual. Ele vislumbra um futuro onde frameworks facilitam essa composicao, da mesma forma que Unix pipes facilitam a composicao de programas.

#### Projetando Aplicacoes em Torno de Fluxo de Dados

**Codigo de Aplicacao como Funcao de Derivacao:**

Em uma arquitetura baseada em fluxo de dados, o codigo da aplicacao funciona como um **operador** que transforma dados de entrada (eventos de um log) em dados de saida (estado derivado em algum armazenamento). A aplicacao nao e mais "uma coisa que responde a requisicoes HTTP" -- e "uma funcao que transforma streams de eventos em views derivadas".

Essa perspectiva muda como pensamos sobre o design da aplicacao:

- Quando a logica de negocio muda, voce atualiza a funcao de derivacao e reprocessa o log para produzir o novo estado.
- Diferentes partes da aplicacao podem ser operadores independentes, cada um consumindo e produzindo streams.
- O estado da aplicacao e sempre recuperavel a partir do log de eventos.

**Separacao de Codigo de Aplicacao e Estado:**

Kleppmann argumenta por uma separacao mais clara entre o **codigo** (logica de processamento) e o **estado** (dados armazenados). Bancos de dados tradicionalmente fundem os dois -- triggers e stored procedures sao codigo que vive dentro do banco de dados. Em uma arquitetura baseada em fluxo de dados:

- O estado vive em sistemas de armazenamento especializados.
- O codigo vive em servicos de stream processing.
- Os dois sao conectados por logs de eventos.

Essa separacao permite escalar, atualizar e fazer deploy de codigo e estado independentemente.

**Dataflow vs. Request/Response:**

O modelo dominante de comunicacao em aplicacoes web e **request/response**: o cliente envia uma requisicao, o servidor processa e retorna uma resposta. Kleppmann argumenta que um modelo baseado em **fluxo de dados** (dataflow) e mais natural para muitos cenarios:

- Em vez de o frontend fazer polling para verificar se algo mudou, o backend pode "empurrar" atualizacoes para o frontend via WebSockets, Server-Sent Events, ou mecanismos similares.
- Em vez de o servidor consultar o banco de dados a cada requisicao, uma view materializada e mantida atualizada por stream processing e servida diretamente.

**Observando Estado Derivado:**

Levando o fluxo de dados ao extremo, Kleppmann descreve como **estender o stream ate o dispositivo do usuario**. Se uma view materializada e atualizada por stream processing, e se o usuario esta vendo essa view em seu navegador ou app, por que nao enviar as atualizacoes da view diretamente ao dispositivo?

Isso e o conceito por tras de tecnologias como:
- **Firebase Realtime Database** e **Firestore**: bancos de dados que notificam clientes automaticamente quando dados mudam.
- **Meteor**: framework web que sincroniza estado entre servidor e cliente via DDP (Distributed Data Protocol).
- **GraphQL Subscriptions**: extensao do GraphQL para receber atualizacoes em tempo real.

A ideia e que a "view materializada" se estende ate a tela do usuario, e toda a cadeia de derivacao -- do evento original, passando pelo processamento, ate a view na tela -- e continua e automatica.

### Buscando Corretude

#### O Argumento End-to-End

O **end-to-end argument** (argumento ponta-a-ponta) e um principio classico de design de sistemas, originalmente articulado por Saltzer, Reed e Clark em 1984. A ideia e que certas propriedades de corretude so podem ser garantidas de ponta a ponta (da origem ao destino final), e tentar garanti-las em camadas intermediarias e insuficiente, embora possa melhorar o desempenho.

Exemplo classico: checksums de rede. Mesmo que cada hop da rede verifique checksums, erros podem ocorrer em memoria, no disco ou no software em cada extremidade. Somente uma verificacao end-to-end (o remetente calcula um checksum dos dados originais, o destinatario verifica) garante a integridade.

**Execucao Exactly-Once:**

O conceito de "exactly-once processing" em sistemas distribuidos e frequentemente mal interpretado. O que realmente se deseja e **exactly-once semantics** -- que o efeito observavel de processar cada evento seja como se tivesse sido processado exatamente uma vez, mesmo que internamente haja retentativas.

Mecanismos para alcançar isso:

**Supressao de Duplicatas:**

Se um evento pode ser entregue mais de uma vez (at-least-once), o consumidor precisa detectar e ignorar duplicatas. Abordagens:

- **Deduplicacao baseada em ID:** Cada evento tem um ID unico. O consumidor mantem um registro de IDs ja processados e ignora duplicatas. O desafio e quanto tempo manter esse registro (nao pode ser infinito).
- **Sequencia de offset:** Em log-based brokers, o offset pode servir como mecanismo de deduplicacao -- se o offset ja foi processado, a mensagem e uma duplicata.

**Identificadores de Operacao:**

Kleppmann propoe que cada operacao que um usuario inicia (como "fazer um pagamento") receba um **ID unico de operacao** gerado pelo cliente. Esse ID flui por todo o sistema. Cada componente verifica se ja processou aquele ID antes de aplica-lo. Isso garante idempotencia end-to-end, independentemente de quantas retentativas ocorram em camadas intermediarias.

**O Argumento End-to-End Aplicado a Bancos de Dados:**

Mesmo bancos de dados com garantias ACID nao oferecem exactly-once semantics end-to-end. Se a aplicacao envia uma requisicao ao banco, o banco executa a transacao, mas a resposta se perde na rede, a aplicacao nao sabe se a transacao foi executada e pode retentar -- executando a transacao duas vezes. Somente um ID de operacao end-to-end resolve isso.

**Garantindo Restricoes (Constraints):**

**Unicidade:** A restricao de unicidade (por exemplo, dois usuarios nao podem ter o mesmo email) e particularmente dificil em sistemas distribuidos. Opcoes:

1. **Particao unica:** Rotear todas as requisicoes que afetam a mesma restricao de unicidade para a mesma particao. Dentro da particao, o processamento e serial e a unicidade pode ser verificada localmente.

2. **Log de consenso:** Usar um log totalmente ordenado. Todas as solicitacoes de criacao de usuario com email X vao para a mesma particao do log. O processador processa em ordem e rejeita duplicatas.

3. **Restricao probabilistica:** Para algumas aplicacoes, uma verificacao probabilistica (aceitar temporariamente e detectar/resolver conflitos depois) e aceitavel.

**Verificacao Assincrona de Restricoes:**

Kleppmann observa que nem todas as restricoes precisam ser verificadas sincronamente. Em muitos domínios, uma violacao temporaria de restricao e aceitavel se for detectada e corrigida rapidamente. Exemplo: overbooking de voos -- companhias aereas intencionalmente permitem overbooking e lidam com as consequencias quando necessario.

**Processamento de Requisicoes Multi-Particao:**

Quando uma unica requisicao de usuario envolve multiplas particoes (por exemplo, uma transferencia bancaria entre duas contas em particoes diferentes), coordenacao e necessaria.

Abordagens:
- **Transacao distribuida (2PC):** Correta mas lenta.
- **Particionamento por operacao:** Registrar a operacao em um unico log (particionado por ID de operacao), e os efeitos nas diferentes particoes sao processados assincronamente.
- **Compensacao (Saga pattern):** Cada etapa e uma transacao local. Se uma etapa falha, transacoes compensatorias revertem as etapas anteriores.

**Coordenacao entre Particoes:**

A coordenacao e o maior inimigo da escalabilidade e do desempenho em sistemas distribuidos. Kleppmann argumenta que devemos minimizar a coordenacao sempre que possivel:

- Use particoes de forma que a maioria das operacoes seja local a uma unica particao.
- Relaxe restricoes quando possivel (consistencia eventual em vez de forte).
- Use mecanismos assincronos (compensacao, deteccao de conflitos) em vez de sincronos (2PC).

#### Tempestividade e Integridade (Timeliness and Integrity)

Kleppmann faz uma distincao crucial entre dois aspectos de "consistencia":

- **Tempestividade (timeliness):** Usuarios veem dados atualizados. Se eu atualizei meu endereco, outros usuarios veem o novo endereco rapidamente. Violacoes de tempestividade sao toleraveis -- o usuario pode ver dados desatualizados por alguns segundos.

- **Integridade (integrity):** Dados nao sao corrompidos, perdidos ou contraditórios. Se eu transferi R$ 100, o debito e o credito devem acontecer juntos. Violacoes de integridade sao muito mais graves -- podem significar perda de dinheiro ou dados.

Transacoes ACID fornecem ambos, mas com alto custo. Kleppmann propoe que muitos sistemas podem relaxar a tempestividade (usando consistencia eventual) enquanto mantêm a integridade (usando mecanismos como log de eventos, idempotencia e verificacao de restricoes).

**Corretude de Sistemas de Fluxo de Dados:**

Um sistema baseado em fluxo de dados pode oferecer garantias de integridade atraves de:

1. **Log de eventos como fonte de verdade:** Eventos sao registrados de forma duravel e ordenada.
2. **Derivacao deterministica:** Views derivadas sao funcoes deterministicas do log de eventos.
3. **Idempotencia:** O processamento de cada evento e idempotente, permitindo retentativas seguras.
4. **Verificacao de restricoes:** Restricoes sao verificadas dentro de uma particao (sem coordenacao).

Essa combinacao oferece **integridade** forte sem necessidade de transacoes distribuidas, ao custo de **tempestividade** eventual.

**Restricoes Interpretadas de Forma Flexivel:**

Kleppmann observa que muitas restricoes de negocio sao menos rigidas do que parecem. Em vez de "nao pode acontecer", elas sao "se acontecer, temos um processo para lidar". Exemplos:

- Saldo bancario negativo: Em vez de rejeitar a transacao, permita e cobre uma taxa de cheque especial.
- Dupla reserva de assento: Em vez de transacao distribuida, permita e ofereça alternativa ao segundo passageiro.

Essa perspectiva permite relaxar a necessidade de coordenacao sincrona em muitos cenarios.

**Sistemas de Dados que Evitam Coordenacao:**

O ideal e projetar sistemas onde a maioria das operacoes nao exige coordenacao:

1. Escritas sao registradas em um log particionado.
2. Consumidores derivam estado de forma assincrona e deterministica.
3. Restricoes de unicidade sao verificadas dentro de particoes.
4. Operacoes que cruzam particoes usam compensacao assincrona.

Esse design oferece alta escalabilidade (sem coordenacao) com boa corretude (integridade via log de eventos e idempotencia).

### Confie, Mas Verifique (Trust, But Verify)

#### Auditoria

Kleppmann argumenta que sistemas de dados precisam de mecanismos de **auditoria** -- a capacidade de verificar, apos o fato, que o sistema se comportou corretamente. Isso e diferente de prevencao (evitar que erros acontecam) -- e deteccao (descobrir quando erros aconteceram).

Bugs em software, falhas de hardware, e erros humanos sao inevitaveis. A questao nao e "como evitar todos os erros", mas "como detectar erros quando eles acontecem e corrigi-los".

#### Projetando para Auditabilidade

Um sistema auditavel tem as seguintes propriedades:

- **Dados imutaveis:** O historico completo de alteracoes e preservado (como em event sourcing). Nao e possivel "apagar rastros".
- **Derivacoes deterministas:** O estado atual pode ser recalculado a partir do historico de eventos. Se o recalculo produz um resultado diferente do estado atual, algo esta errado.
- **Integridade criptografica:** Hashes e assinaturas digitais podem ser usados para detectar adulteracao. Merkle trees (como as usadas em Bitcoin e Git) permitem verificar eficientemente a integridade de grandes conjuntos de dados.

#### Ferramentas para Sistemas de Dados Auditaveis

Tecnologias que suportam auditabilidade incluem:

- **Blockchain / distributed ledgers:** Logs imutaveis com consenso distribuido e verificacao criptografica. Uteis quando ha desconfianca entre as partes, mas com alto overhead.
- **Merkle trees:** Permitem verificar eficientemente se um dado foi adulterado sem verificar todo o dataset.
- **Event sourcing com snapshots verificaveis:** Periodicamente, gerar um snapshot do estado derivado e verificar que ele corresponde ao log de eventos.

#### Cultura de Verificacao vs. "Provavelmente Funciona"

Kleppmann critica a cultura prevalente de confiar que o software "provavelmente funciona" sem verificacao rigorosa. Bancos de dados relacionais, por exemplo, prometem ACID, mas quantos usuarios realmente verificam se a implementacao esta correta? Quantos testam cenarios de falha?

Ele argumenta por uma cultura de:
- Testar cenarios de falha explicitamente (chaos engineering).
- Verificar invariantes continuamente (consistencia por amostragem).
- Projetar para auditabilidade desde o inicio, nao como uma funcionalidade adicional.
- Nao confiar cegamente em nenhuma camada do stack -- verificar end-to-end.

### Fazendo a Coisa Certa (Doing the Right Thing)

#### Etica de Aplicacoes Data-Intensive

Kleppmann encerra o livro com uma secao sobre etica que se tornou cada vez mais relevante desde a publicacao. Ele argumenta que engenheiros de software tem responsabilidade etica sobre como os sistemas que constroem sao usados.

**Analise Preditiva:**

Sistemas de dados sao cada vez mais usados para tomar decisoes sobre pessoas: quem recebe um emprestimo, quem e contratado, quem ve um anuncio, quem e investigado pela policia. Essas decisoes afetam vidas reais.

Problemas:
- **Vieses algoritmicos:** Modelos treinados em dados historicos podem perpetuar e amplificar discriminacoes existentes. Se historicamente certos grupos demograficos receberam menos emprestimos, o modelo aprendera a nega-los, criando um ciclo vicioso.
- **Falta de transparencia:** Modelos complexos (como deep learning) sao frequentemente opacos -- e dificil explicar por que uma decisao foi tomada.
- **Falta de recurso:** Se um algoritmo toma uma decisao errada sobre voce, pode nao haver mecanismo efetivo para contestar.

**Privacidade:**

Kleppmann distingue entre o que e tecnicamente possivel e o que e eticamente aceitavel. So porque e possível coletar, armazenar e analisar dados pessoais nao significa que se deve. Ele aponta para o conceito de **consentimento informado** -- usuarios frequentemente nao entendem que dados estao sendo coletados e como sao usados.

A coleta massiva de dados pessoais cria **riscos assimetricos**: as empresas se beneficiam dos dados, mas os individuos arcam com os riscos (vazamentos, uso indevido, manipulacao).

**Vigilancia:**

Kleppmann observa que a infraestrutura de dados que construimos pode ser (e e) usada para vigilancia em massa. Dados coletados para fins comerciais podem ser requisitados por governos, hackeados por criminosos, ou usados de formas nao previstas originalmente.

Ele cita o conceito de **minimizacao de dados** -- coletar apenas os dados estritamente necessarios, pelo menor tempo possivel. Este principio esta incorporado em regulamentacoes como o GDPR (europeu) e a LGPD (brasileira).

**Consentimento:**

O conceito de "consentimento" para coleta de dados e problematico:
- Termos de servico sao longos e incompreensiveis.
- O "consentimento" e frequentemente coercitivo -- use o servico e aceite os termos, ou nao use.
- Dados coletados para um proposito sao frequentemente reutilizados para outros.
- Quando dados sao compartilhados entre empresas, o consentimento original nao se aplica ao novo contexto.

Kleppmann argumenta que engenheiros de software devem considerar essas questoes proativamente, nao apenas cumprir o minimo legal. Assim como engenheiros civis sao responsabilizados se uma ponte desaba, engenheiros de software devem considerar as consequencias de seus sistemas.

### Licoes-Chave do Capitulo 12

1. A integracao de dados entre sistemas especializados e o desafio central da engenharia de dados moderna.

2. Logs de eventos ordenados sao um mecanismo poderoso para manter multiplos sistemas em sincronia, mais pratico que transacoes distribuidas.

3. Batch e stream processing sao complementares e estao convergindo em frameworks unificados.

4. A "desagregacao" de bancos de dados -- compor multiplos sistemas especializados conectados por streams -- e uma tendencia arquitetural poderosa.

5. Corretude end-to-end requer mecanismos end-to-end (como IDs de operacao); confiar em camadas intermediarias e insuficiente.

6. Tempestividade e integridade sao preocupacoes separaveis -- muitos sistemas podem relaxar tempestividade sem comprometer integridade.

7. Auditabilidade deve ser um principio de design, nao uma funcionalidade tardia.

8. Engenheiros de software tem responsabilidade etica sobre como seus sistemas sao usados, especialmente quando decisoes automatizadas afetam vidas humanas.

---

## Glossario de Termos DDIA

Abaixo estao os termos mais importantes de todas as tres partes do livro, definidos de forma concisa.

**ACID:** Atomicidade, Consistencia, Isolamento, Durabilidade. Conjunto de propriedades que garantem confiabilidade em transacoes de bancos de dados.

**Acknowledgment (ACK):** Confirmacao enviada por um consumidor a um message broker indicando que uma mensagem foi processada com sucesso.

**Append-only log:** Estrutura de dados onde novos registros sao sempre adicionados ao final, nunca modificados ou deletados no lugar. Fundamental para WAL, log-based brokers e event sourcing.

**B-Tree:** Estrutura de dados de arvore balanceada amplamente usada em bancos de dados para indexacao, oferecendo busca, insercao e delecao em tempo O(log n).

**BASE:** Basically Available, Soft state, Eventually consistent. Modelo alternativo ao ACID, associado a sistemas NoSQL.

**Batch processing:** Processamento de grandes volumes de dados de uma so vez, sem interacao em tempo real. A metrica principal e throughput.

**Bloom filter:** Estrutura de dados probabilistica que permite testar se um elemento pertence a um conjunto. Pode ter falsos positivos, mas nunca falsos negativos.

**Broadcast hash join:** Estrategia de join onde o dataset menor e carregado inteiramente na memoria de cada worker.

**CAP theorem:** Teorema que afirma que um sistema distribuido nao pode simultaneamente garantir Consistencia, Disponibilidade e Tolerancia a Particao de rede. Na pratica, como particoes de rede sao inevitaveis, a escolha real e entre consistencia e disponibilidade durante uma particao.

**CDC (Change Data Capture):** Processo de capturar alteracoes feitas em um banco de dados e propaga-las para outros sistemas em tempo quase-real.

**Checkpoint:** Snapshot periodico do estado de um sistema, usado para recuperacao apos falhas.

**Column-oriented storage:** Armazenamento onde os valores de cada coluna sao armazenados juntos, otimizando consultas analiticas que acessam poucas colunas de muitas linhas.

**Compaction:** Processo de limpeza em storage engines (LSM-Trees) ou logs (Kafka) que remove dados obsoletos e consolida dados restantes.

**Consensus:** Acordo entre multiplos nos em um sistema distribuido sobre um valor ou decisao. Protocolos incluem Paxos, Raft e Zab.

**Consistencia causal:** Modelo de consistencia que garante que operacoes causalmente relacionadas sao vistas na mesma ordem por todos os nos.

**Consistencia eventual:** Modelo onde, se nenhuma nova atualizacao for feita, todos os nos eventualmente convergem para o mesmo estado.

**Consistencia forte (linearizabilidade):** Modelo onde todas as operacoes parecem executar atomicamente em alguma ordem total que respeita a ordem real no tempo.

**Consumer group:** Conjunto de consumidores que compartilham o processamento de um topico, onde cada mensagem e entregue a exatamente um consumidor do grupo.

**CRDT (Conflict-free Replicated Data Type):** Estrutura de dados projetada para ser replicada entre multiplos nos e convergir automaticamente sem coordenacao.

**DAG (Directed Acyclic Graph):** Grafo dirigido sem ciclos, usado para representar dependencias entre tarefas em dataflow engines.

**Data warehouse:** Banco de dados otimizado para consultas analiticas (OLAP), populado por processos ETL a partir de sistemas transacionais.

**Dataflow engine:** Framework de processamento (como Spark, Flink, Tez) que executa computacoes como um DAG de operadores, superando limitacoes do MapReduce.

**Denormalizacao:** Duplicacao intencional de dados para evitar joins e acelerar leituras, ao custo de complexidade nas escritas.

**Derived data (dados derivados):** Dados criados a partir de outros dados por meio de transformacao ou processamento. Caches, indices e views materializadas sao dados derivados.

**Distributed transaction:** Transacao que abrange multiplos nos ou sistemas, tipicamente coordenada por protocolo 2PC.

**Document model:** Modelo de dados onde registros sao armazenados como documentos auto-contidos (JSON, BSON), sem esquema rigido.

**Encoding/serialization:** Processo de converter estruturas de dados em memoria para um formato transmissivel (JSON, Protobuf, Avro, Thrift).

**Epoch (era/epoca):** Numero de versao incrementado a cada mudanca de lider em protocolos de consenso.

**Event sourcing:** Padrao onde o estado da aplicacao e representado como uma sequencia imutavel de eventos de dominio, em vez de um snapshot mutavel.

**Event time:** Momento em que um evento realmente ocorreu, em oposicao ao momento em que foi processado (processing time).

**Exactly-once semantics:** Garantia de que o efeito de processar cada evento e como se tivesse sido processado exatamente uma vez, mesmo com retentativas internas.

**Fan-out:** Padrao onde cada mensagem e entregue a todos os consumidores de um topico.

**Fencing token:** Numero monotonicamente crescente usado para prevenir que um no "zombie" (que perdeu a lideranca mas nao sabe) faca alteracoes.

**Graph model:** Modelo de dados que representa entidades como vertices e relacoes como arestas, otimizado para queries que navegam relacoes complexas.

**HDFS (Hadoop Distributed File System):** Sistema de arquivos distribuido inspirado no Google File System, otimizado para throughput com hardware commodity.

**Head-of-line blocking:** Situacao onde uma mensagem lenta bloqueia o processamento de mensagens subsequentes na mesma fila.

**Idempotencia:** Propriedade de uma operacao que produz o mesmo resultado se executada uma ou mais vezes. Crucial para tolerancia a falhas.

**Indice invertido:** Estrutura de dados que mapeia termos a documentos que os contem. Fundamental para motores de busca.

**Join:** Operacao que combina registros de diferentes datasets baseado em uma chave comum.

**Lambda architecture:** Arquitetura que combina camadas de batch e stream processing para oferecer views tanto corretas (batch) quanto atualizadas (stream).

**Lider (leader):** No em um sistema replicado que aceita escritas e as propaga para seguidores. Tambem chamado de master ou primary.

**Linearizabilidade:** Veja "Consistencia forte".

**Load balancing:** Distribuicao de carga entre multiplos nos ou consumidores.

**Log compaction:** Processo de reter apenas a versao mais recente de cada chave em um log, descartando versoes obsoletas.

**Log-structured storage:** Abordagem de armazenamento baseada em append-only logs e compactacao periodica (SSTable, LSM-Tree).

**LSM-Tree (Log-Structured Merge-Tree):** Estrutura de armazenamento que escreve dados sequencialmente em memtables e SSTables, otimizada para alta taxa de escrita.

**MapReduce:** Framework de processamento distribuido que divide o trabalho em fases map (transformacao) e reduce (agregacao), com shuffle/sort intermediario.

**Materialized view:** View pre-computada e armazenada, atualizada quando os dados subjacentes mudam.

**Message broker:** Intermediario que recebe mensagens de produtores e as entrega a consumidores, desacoplando os dois.

**Microbatching:** Tecnica de stream processing que divide o stream em pequenos lotes processados individualmente (usada pelo Spark Streaming).

**Multi-leader replication:** Replicacao onde multiplos nos aceitam escritas, com resolucao de conflitos quando escritas concorrentes ocorrem.

**Normalizacao:** Organizacao de dados para eliminar redundancia, usando referencias (foreign keys) entre tabelas.

**OLAP (Online Analytical Processing):** Processamento otimizado para queries analiticas complexas sobre grandes volumes de dados.

**OLTP (Online Transaction Processing):** Processamento otimizado para transacoes individuais de baixa latencia (insercoes, atualizacoes, leituras por chave).

**Offset:** Posicao sequencial de uma mensagem em uma particao de um log-based broker (como Kafka).

**Particionamento (sharding):** Divisao de dados entre multiplos nos, onde cada no e responsavel por um subconjunto dos dados.

**Particao (partition):** Subdivisao de um topico em um log-based broker, ou de um dataset em um banco de dados distribuido.

**Paxos:** Protocolo de consenso distribuido que garante acordo entre nos mesmo com falhas.

**Pregel/BSP:** Modelo de computacao para processamento de grafos baseado em supersteps sincronos e troca de mensagens entre vertices.

**Processing time:** Momento em que um evento e processado pelo sistema, em oposicao ao momento em que realmente ocorreu.

**Quorum:** Numero minimo de nos que devem concordar para uma operacao ser considerada bem-sucedida (tipicamente maioria).

**Raft:** Protocolo de consenso projetado para ser mais compreensivel que Paxos, usado em etcd e CockroachDB.

**Reduce-side join:** Join realizado na fase reduce do MapReduce, onde o framework agrupa dados pela chave de join.

**Relational model:** Modelo de dados baseado em tabelas (relacoes) com linhas (tuplas) e colunas (atributos), consultado via SQL.

**Replicacao:** Manter copias dos mesmos dados em multiplos nos para tolerancia a falhas e/ou desempenho.

**Request/response:** Modelo de comunicacao sincrono onde um cliente envia uma requisicao e espera por uma resposta.

**Schema-on-read:** Abordagem onde a estrutura dos dados e interpretada no momento da leitura (bancos de dados de documentos).

**Schema-on-write:** Abordagem onde a estrutura dos dados e validada no momento da escrita (bancos de dados relacionais).

**Serializabilidade:** Nivel de isolamento de transacoes que garante que transacoes concorrentes produzem o mesmo resultado que alguma execucao serial.

**Session window:** Janela de tempo em stream processing definida por periodos de atividade, separados por intervalos de inatividade.

**Skew (assimetria):** Distribuicao desigual de dados ou carga entre particoes, causando gargalos.

**Snapshot isolation:** Nivel de isolamento onde cada transacao ve um snapshot consistente dos dados no inicio da transacao.

**Split brain:** Situacao onde dois nos em um sistema replicado ambos acreditam ser o lider, potencialmente causando inconsistencia.

**SSTable (Sorted String Table):** Arquivo de dados onde pares chave-valor sao armazenados ordenados por chave, usado em LSM-Trees.

**Stream processing:** Processamento continuo de eventos a medida que chegam, em oposicao ao processamento em lote.

**System of record:** Sistema que contem a versao autoritativa (fonte de verdade) dos dados.

**Total order:** Ordenacao onde qualquer par de elementos pode ser comparado (um vem antes do outro), em oposicao a ordem parcial.

**Tumbling window:** Janela de tempo fixa que nao se sobrepoe a janelas adjacentes.

**Two-phase commit (2PC):** Protocolo para transacoes distribuidas com fases de preparacao e commit, coordenado por um coordenador central.

**Two-phase locking (2PL):** Mecanismo de controle de concorrencia onde transacoes adquirem locks crescentemente e os liberam apenas apos o commit.

**Vector clock:** Estrutura de dados que rastreia causalidade entre eventos em sistemas distribuidos, representando o "conhecimento" de cada no sobre os demais.

**WAL (Write-Ahead Log):** Log no qual todas as alteracoes sao registradas antes de serem aplicadas ao armazenamento principal, garantindo durabilidade.

**Watermark:** Estimativa em stream processing de que todos os eventos com timestamp anterior a certo valor ja foram recebidos.

**Window:** Intervalo de tempo finito usado para agregar eventos em stream processing (tumbling, hopping, sliding, session).

**Write skew:** Anomalia de concorrencia onde duas transacoes leem os mesmos dados, fazem decisoes baseadas neles e escrevem resultados que, combinados, violam uma invariante.

---

## Principais Licoes

As 15 licoes mais importantes de todo o livro "Designing Data-Intensive Applications":

**1. Nao existe uma ferramenta de dados universal.** Cada sistema de armazenamento e processamento faz trade-offs fundamentais. Bancos relacionais, bancos de documentos, bancos de grafos, key-value stores, motores de busca, caches -- todos existem porque otimizam para padroes de acesso diferentes. O trabalho do engenheiro e entender os trade-offs e escolher a combinacao certa para o problema.

**2. A distincao entre dados de registro e dados derivados e arquiteturalmente fundamental.** Ter clareza sobre qual sistema e a fonte de verdade e quais sistemas contem dados derivados simplifica enormemente o raciocinio sobre consistencia, recuperacao de falhas e evolucao do sistema. Dados derivados podem sempre ser reconstruidos; dados de registro nao.

**3. Codificacao e evolucao de schema sao preocupacoes de primeira classe.** Sistemas reais precisam evoluir ao longo do tempo. Formatos de codificacao (Avro, Protobuf, Thrift) que suportam compatibilidade retroativa e progressiva permitem atualizacoes graduais sem downtime e sem quebrar consumidores existentes.

**4. Replicacao e particionamento sao as duas ferramentas fundamentais de sistemas distribuidos.** Replicacao fornece tolerancia a falhas e pode melhorar o desempenho de leitura. Particionamento permite escalar alem de uma unica maquina. Mas ambos introduzem complexidade substancial -- inconsistencias temporarias, rebalanceamento, resolucao de conflitos.

**5. Transacoes distribuidas sao surpreendentemente dificeis e custosas.** Embora transacoes ACID em uma unica maquina sejam bem compreendidas, estende-las para multiplos nos (via 2PC) introduz problemas de desempenho, disponibilidade e operacao. Muitos sistemas modernos optam por alternativas como consistencia eventual com reconciliacao.

**6. A linearizabilidade e a serializabilidade sao propriedades diferentes e frequentemente confundidas.** Linearizabilidade e sobre a ordem de operacoes em objetos individuais. Serializabilidade e sobre o isolamento entre transacoes. Ambas sao caras de implementar em sistemas distribuidos.

**7. O consenso distribuido e a base de muitas garantias, mas tem custos reais.** Protocolos como Raft e Paxos permitem que nos concordem sobre valores, elejam lideres e implementem locks distribuidos. Mas exigem maiorias (quoruns) e sao sensiveis a latencia de rede.

**8. A imutabilidade e um principio de design poderoso em todos os niveis.** De append-only logs em storage engines, passando por eventos imutaveis em event sourcing, ate a filosofia de dados imutaveis em batch processing -- a imutabilidade simplifica concorrencia, habilita replay e facilita auditoria.

**9. Batch processing herda a filosofia Unix e a aplica em escala distribuida.** Componentes simples e composiveis, interfaces uniformes (arquivos no HDFS), dados imutaveis e capacidade de reexecucao sao principios que sobreviveram da decada de 1970 ate os frameworks modernos como Spark e Flink.

**10. Stream processing e o elo que conecta armazenamento e computacao em tempo real.** Log-based message brokers como Kafka permitem tratar bancos de dados como streams (via CDC) e streams como bancos de dados (via views materializadas), unificando as perspectivas de armazenamento e processamento.

**11. A separacao entre event time e processing time e uma fonte constante de bugs se ignorada.** Sistemas de stream processing devem ser projetados com consciencia de que eventos podem chegar atrasados, fora de ordem, ou duplicados. Watermarks, janelas e idempotencia sao mecanismos essenciais para lidar com essa realidade.

**12. Corretude end-to-end nao pode ser delegada a camadas intermediarias.** O argumento end-to-end de Saltzer, Reed e Clark e um dos principios mais importantes da engenharia de sistemas: garantias como exactly-once e integridade de dados so podem ser asseguradas de ponta a ponta, da origem ao destino final.

**13. Tempestividade e integridade sao preocupacoes separaveis.** Muitos sistemas podem relaxar a tempestividade (aceitar que dados podem estar alguns segundos desatualizados) sem comprometer a integridade (garantir que dados nao sao perdidos ou corrompidos). Essa separacao permite designs mais escalaveis e resilientes.

**14. Auditabilidade deve ser um principio de design, nao uma reflexao tardia.** Sistemas que preservam historico completo (event sourcing), usam derivacoes deterministicas e empregam verificacao criptografica sao fundamentalmente mais confiaveis do que sistemas que confiam em "provavelmente funciona".

**15. Engenheiros de software tem responsabilidade etica sobre os sistemas que constroem.** A capacidade de coletar, armazenar e processar dados em escala massiva traz responsabilidades proporcionais. Vieses algoritmicos, privacidade, vigilancia e consentimento informado sao preocupacoes que devem informar decisoes tecnicas, nao apenas legais.
