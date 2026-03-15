# System Design Interview -- Alex Xu (Vol. 1 e Vol. 2)

Resumo extremamente detalhado dos dois volumes, cobrindo todos os capitulos, problemas de design, trade-offs e consideracoes de escala.

---

## Volume 1

---

### Capitulo 1: Escalar de Zero a Milhoes de Usuarios

#### Declaracao do Problema

Como evoluir uma aplicacao web desde um unico servidor atendendo poucos usuarios ate uma arquitetura distribuida capaz de suportar milhoes de requisicoes simultaneas? O capitulo apresenta uma jornada incremental, adicionando componentes conforme a demanda cresce.

#### Requisitos Funcionais

- Servir requisicoes HTTP de usuarios finais.
- Persistir dados de forma confiavel.
- Manter tempos de resposta aceitaveis sob carga crescente.

#### Requisitos Nao-Funcionais

- Alta disponibilidade (99.9%+).
- Baixa latencia (idealmente < 200ms para requisicoes comuns).
- Escalabilidade horizontal.
- Tolerancia a falhas.

#### Arquitetura Evolutiva

**Estagio 1 -- Servidor Unico**

Tudo roda em uma unica maquina: servidor web, banco de dados e aplicacao. O DNS resolve o dominio para o IP publico desse servidor. Funciona para pouquissimos usuarios, mas e um ponto unico de falha (SPOF) e nao escala.

**Estagio 2 -- Separacao de Banco de Dados**

O banco de dados e movido para um servidor dedicado. Isso permite escalar o tier web e o tier de dados independentemente. Aqui surge a decisao entre banco relacional (MySQL, PostgreSQL) e banco nao-relacional (NoSQL como Cassandra, DynamoDB, MongoDB). Bancos relacionais sao a escolha padrao por oferecerem ACID, JOINs e um ecossistema maduro. NoSQL e preferido quando: os dados nao sao relacionais, voce precisa de latencia muito baixa, os dados nao sao estruturados, ou voce precisa serializar/deserializar dados massivamente.

**Estagio 3 -- Load Balancer e Multiplos Servidores Web**

Um load balancer (como Nginx, HAProxy ou um servico gerenciado como AWS ELB) distribui trafego entre multiplos servidores web. Os servidores web ficam em uma rede privada. O load balancer possui o IP publico. Se um servidor cai, o trafego e redirecionado aos demais. Se a carga cresce, basta adicionar mais servidores. Algoritmos comuns de balanceamento: round-robin, least connections, IP hash, weighted round-robin.

**Estagio 4 -- Replicacao de Banco de Dados (Master-Slave)**

O banco de dados agora opera em modo master-slave (ou primary-replica). O master recebe todas as escritas. Os slaves recebem copias dos dados e servem leituras. Como a maioria das aplicacoes tem uma proporcao de leitura muito maior que escrita (frequentemente 10:1 ou mais), isso melhora significativamente o throughput. Se o master cai, um slave e promovido a master. Se um slave cai, leituras sao redirecionadas aos demais slaves. Beneficios: melhor performance de leitura, maior confiabilidade (dados replicados em multiplos nos), alta disponibilidade.

**Estagio 5 -- Cache**

Um cache (como Memcached ou Redis) armazena dados frequentemente acessados em memoria, reduzindo a carga no banco de dados. A estrategia mais comum e o read-through cache: a aplicacao primeiro verifica o cache; se encontra (cache hit), retorna diretamente; se nao encontra (cache miss), consulta o banco, armazena no cache e retorna. Consideracoes importantes:

- Quando usar: dados lidos frequentemente mas modificados raramente.
- Politica de expiracao: TTL muito curto causa muitos cache misses; TTL muito longo pode servir dados obsoletos.
- Consistencia: manter cache e banco sincronizados e desafiador. Em cenarios de escrita frequente, estrategias como write-through ou write-behind podem ser usadas.
- Mitigacao de falhas: um unico servidor de cache e um SPOF. Usar multiplos nos de cache com sharding.
- Politica de eviction: LRU (Least Recently Used) e a mais popular. Alternativas incluem LFU (Least Frequently Used) e FIFO.

**Estagio 6 -- CDN (Content Delivery Network)**

Uma CDN e uma rede de servidores geograficamente distribuidos que entregam conteudo estatico (imagens, CSS, JS, videos) a partir do servidor mais proximo ao usuario. Fluxo: o usuario requisita um recurso; se a CDN tem o recurso em cache, entrega diretamente (cache hit); caso contrario, busca na origem (servidor web ou storage como S3), armazena em cache e entrega. Consideracoes:

- Custo: CDNs cobram por transferencia de dados. Evite cachear recursos raramente acessados.
- TTL: equilibrio entre frescor dos dados e taxa de cache hit.
- Fallback: se a CDN falhar, a aplicacao deve ser capaz de buscar diretamente da origem.
- Invalidacao: via APIs da CDN ou usando versionamento de objetos (ex: imagem_v2.png).

**Estagio 7 -- Servidores Web Stateless**

O estado da sessao do usuario e movido dos servidores web para um armazenamento compartilhado (Redis, Memcached ou banco de dados). Isso torna cada servidor web intercambiavel -- qualquer servidor pode atender qualquer requisicao. Beneficio: auto-scaling se torna trivial, pois basta adicionar ou remover servidores conforme a carga.

**Estagio 8 -- Multiplos Data Centers**

Para atender usuarios globalmente com baixa latencia e alta disponibilidade, a aplicacao e implantada em multiplos data centers (ex: US-East e EU-West). O geoDNS direciona usuarios ao data center mais proximo. Desafios:

- Redirecionamento de trafego: em caso de falha de um data center, todo trafego e redirecionado ao outro.
- Sincronizacao de dados: replicacao multi-region do banco de dados.
- Testes e deploys: devem ser executados em todos os data centers.

**Estagio 9 -- Fila de Mensagens (Message Queue)**

Uma fila de mensagens (RabbitMQ, Kafka, SQS) desacopla componentes do sistema. Produtores publicam mensagens na fila; consumidores as processam assincronamente. Exemplo: ao enviar uma foto, o upload vai para a fila e workers processam redimensionamento em background. Beneficio: o produtor e o consumidor escalam independentemente. Se o consumidor esta lento, mensagens acumulam na fila sem impactar o produtor.

**Estagio 10 -- Sharding de Banco de Dados**

Quando um unico banco de dados nao suporta mais a carga, os dados sao particionados horizontalmente (sharding). Cada shard contem um subconjunto dos dados. Uma funcao de sharding (ex: user_id % numero_de_shards) determina em qual shard os dados residem. Desafios do sharding:

- Resharding: quando um shard atinge sua capacidade ou a distribuicao de dados fica desigual, e necessario redistribuir. Consistent hashing ajuda a minimizar a movimentacao de dados.
- Celebrity problem (hotspot): certos shards podem receber trafego desproporcional (ex: dados de celebridades). Solucao: particionar esses dados ainda mais ou alocar um shard dedicado.
- JOINs cross-shard: JOINs entre dados em shards diferentes sao extremamente complexos. A aplicacao frequentemente precisa desnormalizar dados ou executar JOINs no nivel da aplicacao.

#### Trade-offs Principais

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Escala vertical vs horizontal | Vertical e simples | Vertical tem limite fisico e e SPOF |
| SQL vs NoSQL | SQL tem ACID e JOINs | NoSQL escala horizontalmente com mais facilidade |
| Cache em memoria | Latencia muito baixa | Complexidade de consistencia e custo de RAM |
| Sharding | Escala quase ilimitada | Complexidade operacional massiva |

#### Licoes-Chave

- Comece simples e evolua conforme necessario. Nao faca over-engineering.
- Cada componente adicionado resolve um problema mas introduz complexidade.
- A separacao de concerns (web tier, data tier, cache tier) e fundamental para escalar.
- Statelessness no web tier e pre-requisito para escalabilidade horizontal.
- Sharding e o ultimo recurso -- maximize cache e replicas antes.

---

### Capitulo 2: Estimativas de Envelope (Back-of-the-Envelope Estimation)

#### Declaracao do Problema

Em entrevistas de system design, frequentemente e necessario estimar a escala do sistema: quantas requisicoes por segundo, quanto armazenamento, quanta largura de banda. Estimativas rapidas e razoaveis demonstram capacidade analitica.

#### Potencias de 2

Toda estimativa comeca com potencias de 2, pois tamanhos de dados em computacao sao baseados nelas:

- 1 KB = 10^3 bytes (2^10 = 1024, arredondado)
- 1 MB = 10^6 bytes (2^20)
- 1 GB = 10^9 bytes (2^30)
- 1 TB = 10^12 bytes (2^40)
- 1 PB = 10^15 bytes (2^50)

#### Numeros de Latencia que Todo Programador Deveria Saber

Valores aproximados (originalmente compilados por Jeff Dean):

- Referencia ao cache L1: 0.5 ns
- Referencia ao cache L2: 7 ns
- Leitura de memoria principal (RAM): 100 ns
- Envio de 1 KB pela rede em 1 Gbps: 10 us
- Leitura sequencial de 1 MB da RAM: 250 us
- Round-trip na mesma rede de data center: 500 us
- Leitura sequencial de 1 MB de SSD: 1 ms
- Seek de disco HDD: 10 ms
- Leitura sequencial de 1 MB de HDD: 20 ms
- Envio de pacote CA -> Holanda -> CA: 150 ms

Conclusoes praticas: memoria e dramaticamente mais rapida que disco; evite seeks em disco; compressao antes de enviar pela rede geralmente compensa; data centers em regioes diferentes tem latencia significativa.

#### Numeros de Disponibilidade

SLAs sao expressos em "noves":

- 99% (dois noves): ~3.65 dias de downtime/ano
- 99.9% (tres noves): ~8.76 horas/ano
- 99.99% (quatro noves): ~52.6 minutos/ano
- 99.999% (cinco noves): ~5.26 minutos/ano

#### Framework de Estimativa

1. Definir premissas claras (DAU, acoes por usuario, tamanho de cada acao).
2. Calcular QPS (Queries Per Second): DAU x acoes/usuario / 86400 segundos.
3. QPS de pico: tipicamente 2x a 5x a media.
4. Estimar armazenamento: tamanho medio do objeto x numero de objetos x periodo de retencao.
5. Estimar largura de banda: QPS x tamanho medio da resposta.
6. Estimar necessidade de cache: regra dos 80/20 -- cachear os 20% mais acessados cobre ~80% das requisicoes.

#### Exemplo Pratico: Twitter

- 300 milhoes de MAU, 50% DAU = 150 milhoes DAU.
- Cada usuario faz 2 tweets/dia: 150M x 2 / 86400 = ~3500 tweets/segundo.
- Cada usuario le 100 tweets/dia: 150M x 100 / 86400 = ~170K leituras/segundo.
- Tamanho medio do tweet com metadata: ~500 bytes.
- Armazenamento por dia: 150M x 2 x 500 bytes = 150 GB/dia.
- Em 5 anos: 150 GB x 365 x 5 = ~274 TB.

#### Licoes-Chave

- Precisao nao e o objetivo; a capacidade de raciocinar sobre escala e o que importa.
- Arredonde agressivamente (use potencias de 10).
- Escreva as premissas antes de calcular.
- Expresse resultados em unidades relevantes (QPS, TB, Gbps).

---

### Capitulo 3: Framework para Entrevistas de System Design

#### O Framework de 4 Passos

**Passo 1 -- Entender o Problema e Estabelecer Escopo do Design (3-10 min)**

Nunca comece a desenhar imediatamente. Faca perguntas para esclarecer:

- Quais sao as funcionalidades mais importantes?
- Quantos usuarios o sistema deve suportar?
- Qual e a escala esperada (DAU, QPS)?
- Qual e o stack tecnologico existente? Ha restricoes?
- Quais sao os requisitos nao-funcionais (latencia, disponibilidade, consistencia)?

O objetivo e transformar um problema vago em requisitos concretos. Um erro comum e resolver o problema errado por nao ter feito perguntas suficientes.

**Passo 2 -- Propor o Design de Alto Nivel (10-15 min)**

- Desenhe um diagrama com os componentes principais: clientes, load balancers, servidores web, servicos, banco de dados, cache, filas.
- Identifique as APIs principais (endpoints REST ou gRPC).
- Percorra os fluxos principais (happy path) para validar que o design atende aos requisitos.
- Obtenha concordancia do entrevistador antes de aprofundar.

**Passo 3 -- Deep Dive no Design (10-25 min)**

O entrevistador guiara quais componentes aprofundar. Geralmente foca-se em:

- Esquema de dados e modelagem.
- Detalhes de componentes criticos (ex: como o rate limiter funciona exatamente).
- Gargalos potenciais e como resolve-los.
- Trade-offs de cada decisao.
- Casos limites (edge cases) e cenarios de falha.

**Passo 4 -- Encerramento (3-5 min)**

- Identifique gargalos e proponha melhorias.
- Recapitule o design.
- Discuta tratamento de erros, monitoramento, metricas.
- Mencione possibilidades de evolucao futura.

#### Dicas Gerais

- Nao tente cobrir tudo; foque no que e mais importante e interessante.
- Comunique seu raciocinio continuamente; o entrevistador quer entender como voce pensa.
- Sempre considere trade-offs; nao existe solucao perfeita.
- Demonstre conhecimento de ferramentas e tecnologias reais.

---

### Capitulo 4: Design de um Rate Limiter

#### Declaracao do Problema

Projetar um componente que limita o numero de requisicoes que um cliente pode fazer a uma API em um determinado periodo de tempo.

#### Requisitos Funcionais

- Limitar requisicoes excessivas de um mesmo cliente.
- Retornar resposta adequada (HTTP 429 Too Many Requests) quando o limite e excedido.
- Permitir diferentes regras de limitacao (por usuario, por IP, por endpoint).

#### Requisitos Nao-Funcionais

- Latencia extremamente baixa (o rate limiter nao pode ser gargalo).
- Uso minimo de memoria.
- Funcionar em ambiente distribuido (multiplos servidores).
- Alta tolerancia a falhas.
- Tratamento claro de excecoes.

#### Onde Posicionar o Rate Limiter

Tres opcoes:

1. **Client-side**: facil de burlar, nao confiavel.
2. **Server-side**: implementado como middleware antes de chegar a logica de negocio.
3. **Como servico separado (API Gateway)**: solucoes como Kong, AWS API Gateway, Zuul ja incluem rate limiting. E a abordagem preferida em arquiteturas de microservicos.

A escolha depende do stack, necessidades de engenharia e recursos disponíveis. Em geral, um API Gateway e a melhor opcao se voce ja usa um.

#### Algoritmos de Rate Limiting

**1. Token Bucket**

Um "balde" com capacidade fixa de tokens. Cada requisicao consome um token. Tokens sao adicionados a uma taxa fixa. Se o balde esta vazio, a requisicao e rejeitada. Parametros: tamanho do balde (burst maximo) e taxa de reposicao (throughput sustentado). Vantagens: simples, permite bursts curtos. Desvantagem: dois parametros para ajustar.

Exemplo: balde de 4 tokens, reposicao de 4 tokens/minuto. Um burst de 4 requisicoes e permitido, depois o limite e 4/minuto.

**2. Leaking Bucket**

Similar ao token bucket, mas as requisicoes sao enfileiradas e processadas a uma taxa fixa (como agua vazando de um balde). Se a fila esta cheia, novas requisicoes sao descartadas. Vantagem: saida a taxa constante, util quando voce precisa de um fluxo estavel. Desvantagem: bursts de trafego enchem a fila com requisicoes antigas; requisicoes recentes podem ser descartadas mesmo com requisicoes antigas na fila.

**3. Fixed Window Counter**

O tempo e dividido em janelas fixas (ex: janelas de 1 minuto). Um contador rastreia quantas requisicoes ocorrem em cada janela. Se o contador excede o limite, requisicoes sao rejeitadas ate a proxima janela. Vantagem: simples, eficiente em memoria. Desvantagem: um burst na fronteira entre duas janelas pode permitir o dobro do limite (ex: 5 requisicoes no final da janela 1 + 5 no inicio da janela 2 = 10 em ~1 segundo, com limite de 5/minuto).

**4. Sliding Window Log**

Mantem um log com timestamp de cada requisicao. Ao receber uma nova requisicao, remove timestamps mais antigos que a janela e conta os restantes. Vantagem: precisao perfeita. Desvantagem: consome muita memoria, pois armazena timestamps de todas as requisicoes, mesmo as rejeitadas.

**5. Sliding Window Counter**

Combina fixed window counter com sliding window log. Usa contadores de janelas fixas adjacentes e calcula uma media ponderada baseada na posicao no tempo. Exemplo: se estamos 70% apos o inicio da janela atual, o rate = (contador_janela_anterior x 0.3) + (contador_janela_atual x 1.0). Vantagem: suaviza picos na fronteira, eficiente em memoria. Desvantagem: assume distribuicao uniforme dentro de cada janela (aproximacao).

#### Arquitetura de Alto Nivel

O rate limiter e implementado como middleware. Usa Redis como storage in-memory para contadores (comandos INCR e EXPIRE sao atomicos e extremamente rapidos). Regras de rate limiting sao armazenadas em disco (config files ou banco) e carregadas em cache.

Fluxo: requisicao chega -> middleware consulta Redis -> se abaixo do limite, incrementa contador e encaminha -> se acima, retorna HTTP 429 com headers:

- X-Ratelimit-Remaining: requisicoes restantes na janela.
- X-Ratelimit-Limit: total permitido na janela.
- X-Ratelimit-Retry-After: segundos ate poder tentar novamente.

#### Rate Limiting Distribuido

Em ambiente com multiplos servidores, dois problemas surgem:

1. **Race condition**: dois servidores leem o mesmo contador, ambos incrementam, e o valor final e 1 a menos do que deveria. Solucao: usar Lua scripts no Redis (atomicos) ou usar o comando sorted set do Redis.
2. **Sincronizacao**: cada servidor tem seu proprio rate limiter com dados diferentes. Solucao: usar um datastore centralizado (Redis) ao inves de contadores locais.

#### Trade-offs

- Hard limiting vs soft limiting: hard rejeita estritamente; soft permite um pequeno excesso.
- Rate limiting por camada: camada de aplicacao (L7) vs camada de rede (L3).
- Evitar rate limiting do seu proprio servico: usar filas internas para comunicacao entre servicos.
- Monitoramento: fundamental para ajustar regras. Se muitas requisicoes legitimas estao sendo bloqueadas, as regras estao muito restritivas.

#### Licoes-Chave

- Sliding window counter e o algoritmo mais equilibrado para a maioria dos cenarios.
- Redis e a escolha padrao para storage de rate limiting por sua atomicidade e velocidade.
- Em entrevistas, discuta os trade-offs entre os algoritmos em vez de apenas escolher um.

---

### Capitulo 5: Design de Consistent Hashing

#### Declaracao do Problema

Quando dados sao distribuidos entre N servidores usando hash simples (key % N), adicionar ou remover um servidor causa remapeamento massivo de chaves. Consistent hashing resolve isso minimizando a redistribuicao.

#### O Problema com Hashing Simples

Se temos 4 servidores e usamos serverIndex = hash(key) % 4, tudo funciona bem. Mas se um servidor cai (N=3), quase todas as chaves sao remapeadas para servidores diferentes. Em um sistema de cache, isso causa um "cache avalanche" -- quase todos os caches sao invalidados simultaneamente, gerando uma tempestade de requisicoes ao banco de dados.

#### Como Funciona o Consistent Hashing

1. **Hash Ring**: imagina-se um anel (circulo) onde o espaco de hash vai de 0 a 2^32-1 (ou outro valor grande). O ponto 0 e o ponto maximo se encontram, formando o anel.

2. **Mapeamento de servidores**: cada servidor e mapeado para um ponto no anel usando uma funcao hash (ex: hash(IP do servidor)). Servidores sao posicionados no anel.

3. **Mapeamento de chaves**: cada chave e mapeada para um ponto no anel. A chave e atribuida ao primeiro servidor encontrado percorrendo o anel no sentido horario a partir do ponto da chave.

4. **Adicao de servidor**: quando um novo servidor e adicionado, apenas as chaves entre o servidor anterior (no sentido anti-horario) e o novo servidor sao remapeadas. Todas as outras chaves permanecem onde estao.

5. **Remocao de servidor**: quando um servidor e removido, suas chaves sao redistribuidas ao proximo servidor no sentido horario. Apenas as chaves daquele servidor sao afetadas.

#### Problema da Distribuicao Desigual

Com poucos servidores, e muito provavel que a distribuicao no anel seja desigual, com alguns servidores responsaveis por particoes muito maiores que outros. Isso causa hotspots.

#### Virtual Nodes (Vnodes)

A solucao e usar nos virtuais. Cada servidor real e representado por multiplos pontos no anel (ex: servidor A tem A0, A1, A2, ..., A199). Com mais pontos, a distribuicao se torna mais uniforme. Na pratica, sistemas como Cassandra usam 256 virtual nodes por servidor fisico. Quanto mais virtual nodes, melhor a distribuicao, mas maior o uso de memoria para a tabela de lookup. E um trade-off.

#### Encontrando o Servidor para uma Chave

Dado uma chave, calcula-se seu hash e percorre-se o anel no sentido horario ate encontrar o primeiro virtual node. O servidor fisico correspondente a esse virtual node e o responsavel pela chave. Na implementacao, isso e feito com uma estrutura de dados sorted (como TreeMap em Java), permitindo busca em O(log N).

#### Aplicacoes Reais

- Amazon DynamoDB: usa consistent hashing para particionar dados.
- Apache Cassandra: distribuicao de dados entre nos do cluster.
- Akamai CDN: distribuicao de conteudo entre servidores edge.
- Discord: distribuicao de carga entre servidores.

#### Trade-offs

| Aspecto | Hash simples | Consistent Hashing |
|---|---|---|
| Redistribuicao ao escalar | Massiva (quase 100%) | Minima (K/N chaves, onde K = total de chaves e N = servidores) |
| Complexidade | Trivial | Moderada |
| Distribuicao uniforme | Boa com muitos servidores | Requer virtual nodes |
| Uso de memoria | Minimo | Proporcional ao numero de virtual nodes |

#### Licoes-Chave

- Consistent hashing e essencial em qualquer sistema distribuido que precisa escalar.
- Virtual nodes sao obrigatorios na pratica para garantir distribuicao uniforme.
- A quantidade de virtual nodes e um trade-off entre uniformidade e uso de memoria.
- Entenda que K/N e a fracao de chaves redistribuidas quando um servidor e adicionado ou removido (K = chaves totais, N = servidores).

---

### Capitulo 6: Design de um Key-Value Store

#### Declaracao do Problema

Projetar um armazenamento de chave-valor distribuido (similar a DynamoDB, Cassandra ou etcd) que suporte operacoes put(key, value) e get(key) com alta disponibilidade, escalabilidade e baixa latencia.

#### Requisitos Funcionais

- put(key, value): insere ou atualiza um par chave-valor.
- get(key): retorna o valor associado a chave.
- Tamanho do par chave-valor: chave < 10 KB, valor < 10 KB.

#### Requisitos Nao-Funcionais

- Alta disponibilidade (o sistema responde mesmo durante falhas de rede).
- Alta escalabilidade (suportar grandes volumes de dados).
- Consistencia ajustavel (o usuario pode escolher o nivel de consistencia).
- Baixa latencia.
- Tuning automatico.

#### Teorema CAP

O teorema CAP afirma que um sistema distribuido so pode garantir simultaneamente duas das tres propriedades:

- **Consistency (C)**: todos os nos veem os mesmos dados ao mesmo tempo.
- **Availability (A)**: toda requisicao recebe uma resposta (mesmo que nao seja a mais recente).
- **Partition Tolerance (P)**: o sistema continua funcionando mesmo com falhas de comunicacao entre nos.

Na pratica, particoes de rede sao inevitaveis, entao a escolha real e entre CP e AP:

- **CP (Consistency + Partition Tolerance)**: sacrifica disponibilidade. Se houver uma particao, nos que nao podem garantir dados atualizados retornam erro. Exemplo: bancos de dados tradicionais, HBase, MongoDB (configuracao padrao).
- **AP (Availability + Partition Tolerance)**: sacrifica consistencia. Todos os nos respondem, mas podem retornar dados desatualizados. Exemplo: Cassandra, DynamoDB (configuracao padrao), CouchDB.

A maioria dos key-value stores distribuidos opta por AP com consistencia eventual (eventual consistency).

#### Componentes do Sistema

**1. Particionamento de Dados**

Usa consistent hashing (capitulo 5) para distribuir dados entre nos. Cada no e responsavel por um range de chaves no hash ring. Virtual nodes garantem distribuicao uniforme.

**2. Replicacao de Dados**

Cada chave e replicada em N nos (ex: N=3). Apos mapear a chave ao seu no principal via consistent hashing, os dados sao replicados nos proximos N-1 nos no sentido horario do anel (pulando virtual nodes do mesmo servidor fisico). Isso garante que copias estejam em servidores fisicos distintos.

**3. Consistencia Ajustavel (Quorum)**

Parametros:

- N = numero total de replicas.
- W = quorum de escrita (quantas replicas devem confirmar uma escrita para que seja considerada bem-sucedida).
- R = quorum de leitura (quantas replicas devem responder para que a leitura seja considerada bem-sucedida).

Configuracoes tipicas:

- R=1, W=N: otimizado para leituras rapidas (sistema de leitura intensiva).
- R=N, W=1: otimizado para escritas rapidas.
- W + R > N: consistencia forte garantida (pelo menos uma replica tera o dado mais recente).
- W + R <= N: consistencia forte nao garantida, mas maior disponibilidade e latencia menor.

Exemplo com N=3: W=2, R=2 garante consistencia forte. W=1, R=1 oferece disponibilidade maxima com consistencia eventual.

**4. Modelos de Consistencia**

- **Consistencia forte**: qualquer leitura retorna o valor da escrita mais recente. Requer que todas as replicas concordem antes de responder. Sacrifica disponibilidade.
- **Consistencia eventual**: dada tempo suficiente sem novas escritas, todas as replicas convergem para o mesmo valor. Leituras podem retornar valores desatualizados temporariamente.
- **Consistencia causal**: garante que operacoes causalmente relacionadas sao vistas na ordem correta.

**5. Resolucao de Conflitos com Vector Clocks**

Quando replicas divergem (escritas concorrentes em nos diferentes), e necessario detectar e resolver conflitos. Um vector clock e um par [servidor, versao] associado a cada dado. Cada servidor mantem seu proprio contador. Quando um servidor modifica um dado, incrementa seu contador.

Exemplo:

1. Cliente escreve D1, tratada pelo servidor Sx: D1([Sx, 1]).
2. Cliente escreve D2 (atualizacao de D1), tratada por Sx: D2([Sx, 2]).
3. Cliente le D2 e dois servidores processam escritas concorrentes:
   - Servidor Sy produz D3([Sx, 2], [Sy, 1]).
   - Servidor Sz produz D4([Sx, 2], [Sz, 1]).
4. D3 e D4 sao conflitantes (nenhum e ancestral do outro). O cliente detecta o conflito na proxima leitura e resolve (ex: merge).

Desvantagem dos vector clocks: complexidade para o cliente e crescimento do vetor (mitigado com truncamento de pares antigos).

**6. Deteccao de Falhas com Gossip Protocol**

Em um cluster grande, nao e viavel que cada no monitore todos os outros diretamente. O gossip protocol funciona assim: cada no mantem uma lista de membros com heartbeat counters. Periodicamente, cada no envia sua lista a alguns nos aleatorios. Ao receber uma lista, o no atualiza os contadores. Se o contador de um no nao e atualizado apos um periodo, o no e considerado offline. Vantagem: descentralizado, escalavel, sem ponto unico de falha na deteccao.

**7. Tratamento de Falhas Temporarias -- Sloppy Quorum e Hinted Handoff**

Em um quorum estrito, se nos estao indisponiveis, escritas e leituras podem falhar. Sloppy quorum relaxa isso: se um no responsavel esta offline, outro no (nao responsavel) aceita a escrita temporariamente e armazena uma "hint" indicando o destino real. Quando o no original volta, os dados sao transferidos (hinted handoff). Isso aumenta a disponibilidade de escrita.

**8. Deteccao de Inconsistencias com Merkle Trees**

Para detectar inconsistencias entre replicas de forma eficiente, usa-se Merkle trees (hash trees). Cada no constroi uma arvore onde as folhas sao hashes dos dados e cada no pai e o hash dos seus filhos. Para comparar dois nos, basta comparar a raiz: se for igual, os dados sao identicos. Se for diferente, desce a arvore para encontrar os dados especificos que divergem. Isso minimiza a quantidade de dados transferida durante sincronizacao.

**9. Arquitetura de Storage -- Write Path e Read Path**

Write path: escrita chega -> dados sao escritos no commit log (append-only, para durabilidade) -> dados sao escritos na memtable (estrutura em memoria, geralmente uma red-black tree ou skip list) -> quando a memtable atinge um limiar, e flushed para disco como uma SSTable (Sorted String Table).

Read path: leitura chega -> verifica a memtable -> se nao encontra, verifica SSTables do mais recente ao mais antigo -> usa Bloom filters para evitar buscas desnecessarias em SSTables que nao contem a chave. Bloom filter e uma estrutura probabilistica que pode dizer "definitivamente nao esta aqui" ou "possivelmente esta aqui".

#### Trade-offs Principais

| Decisao | CP | AP |
|---|---|---|
| Cenario ideal | Dados financeiros, inventario | Redes sociais, contadores |
| Latencia de escrita | Maior (espera quorum) | Menor (escrita local) |
| Disponibilidade | Menor durante particoes | Maior |
| Resolucao de conflitos | Desnecessaria | Obrigatoria (vector clocks, CRDTs) |

#### Licoes-Chave

- O teorema CAP nao e uma escolha binaria; e um espectro ajustavel via quorums.
- Vector clocks sao poderosos mas complexos; na pratica, muitos sistemas usam last-write-wins (LWW) por simplicidade, aceitando perda de dados em conflitos.
- Gossip protocol e o padrao de facto para membership e deteccao de falhas em clusters grandes.
- Merkle trees sao elegantes para sincronizacao anti-entropia.
- O write path (commit log + memtable + SSTables) e o padrao LSM-tree, usado por Cassandra, HBase, LevelDB e RocksDB.

---

### Capitulo 7: Design de um Gerador de IDs Unicos em Sistema Distribuido

#### Declaracao do Problema

Em um sistema distribuido, auto-increment de banco de dados unico nao funciona (nao escala, e SPOF). Precisamos gerar IDs unicos globalmente, de forma distribuida e eficiente.

#### Requisitos Funcionais

- IDs devem ser unicos globalmente.
- IDs devem ser numericos (apenas digitos).
- IDs devem caber em 64 bits.
- IDs devem ser ordenados por tempo (IDs gerados depois sao maiores).
- O sistema deve gerar 10.000+ IDs por segundo.

#### Abordagens Analisadas

**1. Multi-Master Replication**

Cada servidor de banco de dados gera IDs usando auto-increment, mas com incremento igual ao numero de servidores. Ex: com 2 servidores, servidor 1 gera 1, 3, 5, 7... e servidor 2 gera 2, 4, 6, 8... Vantagens: simples, sem ponto unico de falha. Desvantagens: IDs nao sao ordenados por tempo globalmente; nao escala bem ao adicionar/remover servidores (precisa recalcular incrementos); dificil em ambientes multi-data-center.

**2. UUID (Universally Unique Identifier)**

UUIDs sao numeros de 128 bits gerados independentemente por cada servidor, sem coordenacao. Probabilidade de colisao e astronomicamente baixa. Vantagens: simples, cada servidor gera independentemente, escalabilidade perfeita. Desvantagens: 128 bits (nao cabe em 64 bits), nao sao ordenados por tempo, nao sao numericos puros (contem letras em hex), nao sao eficientes como indices de banco de dados (fragmentacao de B-tree).

**3. Ticket Server (abordagem do Flickr)**

Um servidor centralizado (ou par de servidores para redundancia) gera IDs sequenciais. Implementado com auto-increment em banco de dados centralizado. Vantagens: IDs numericos, facil de implementar, funciona bem para aplicacoes de escala media. Desvantagens: ponto unico de falha (SPOF); mesmo com multiplos ticket servers, a sincronizacao e desafiadora.

**4. Twitter Snowflake (abordagem escolhida)**

IDs de 64 bits divididos em segmentos:

- Bit 0 (1 bit): reservado (sempre 0), para garantir IDs positivos.
- Timestamp (41 bits): milissegundos desde uma epoca customizada. 41 bits = ~69 anos de timestamps.
- Datacenter ID (5 bits): suporta ate 32 data centers.
- Machine ID (5 bits): suporta ate 32 maquinas por data center.
- Sequence number (12 bits): incrementado para IDs gerados no mesmo milissegundo na mesma maquina. Suporta ate 4096 IDs/ms/maquina. Reseta a 0 a cada novo milissegundo.

Throughput total teorico: 32 datacenters x 32 maquinas x 4096 IDs/ms = ~4.2 milhoes IDs/ms.

Vantagens: IDs de 64 bits, ordenados por tempo (timestamp e o segmento mais significativo), altamente escalaveis, sem coordenacao entre nos (cada no gera independentemente), extremamente rapido (operacao local, sem rede). Desvantagens: depende de clock sincronizado entre servidores (NTP). Clock skew pode causar IDs fora de ordem ou duplicados. Na pratica, mitigado com monitoramento de NTP e logica de espera quando o clock retrocede.

#### Deep Dive no Snowflake

- A epoca customizada (custom epoch) e escolhida para maximizar a vida util do timestamp. Se a epoca for "01/01/2020 00:00:00", o sistema funciona ate ~2089.
- O sequence number e mantido em memoria por cada instancia do gerador. E thread-safe (atomico).
- Se todas as 4096 sequencias forem usadas em um milissegundo, o gerador espera ate o proximo milissegundo.

#### Trade-offs

| Abordagem | Unicidade | Ordenacao temporal | Escalabilidade | Complexidade |
|---|---|---|---|---|
| Multi-master | Sim | Nao | Media | Baixa |
| UUID | Sim (pratico) | Nao | Alta | Baixa |
| Ticket Server | Sim | Sim | Baixa | Baixa |
| Snowflake | Sim | Sim | Alta | Media |

#### Licoes-Chave

- Snowflake e o padrao da industria para IDs distribuidos com ordenacao temporal.
- A sincronizacao de clocks (NTP) e um requisito critico; clock drift deve ser monitorado.
- Os bits podem ser ajustados conforme necessidade: mais bits para timestamp = maior vida util; mais bits para sequence = maior throughput por maquina.

---

### Capitulo 8: Design de um URL Shortener

#### Declaracao do Problema

Projetar um servico como bit.ly ou TinyURL que converte URLs longas em URLs curtas e redireciona usuarios da URL curta para a URL original.

#### Requisitos Funcionais

- Dada uma URL longa, gerar uma URL curta unica.
- Ao acessar a URL curta, redirecionar para a URL original.
- Permitir URLs customizadas (opcional).
- URLs expiram apos um periodo configuravel.

#### Requisitos Nao-Funcionais

- Alta disponibilidade (se o servico cai, todos os links curtos param).
- Baixa latencia no redirecionamento.
- URLs curtas nao devem ser previsiveis (seguranca).

#### Estimativas

- 100 milhoes de URLs geradas por dia.
- QPS de escrita: 100M / 86400 = ~1160/s. Pico: ~2300/s.
- Proporcao leitura:escrita = 10:1, entao QPS de leitura = ~11600/s.
- Armazenamento por 10 anos: 100M x 365 x 10 = 365 bilhoes de registros.
- Tamanho medio por registro: ~500 bytes. Total: ~182 TB.

#### APIs

- POST /api/v1/shorten: recebe {longUrl: "..."}, retorna {shortUrl: "..."}.
- GET /api/v1/{shortUrl}: retorna redirecionamento HTTP para a URL original.

#### 301 vs 302 Redirect

- **301 Moved Permanently**: o browser cacheia o redirecionamento. Proximas requisicoes vao direto a URL original sem passar pelo servidor do shortener. Reduz carga no servidor, mas impede rastreamento de analytics (cliques).
- **302 Found (Temporary Redirect)**: o browser nao cacheia. Toda requisicao passa pelo servidor do shortener. Permite rastreamento completo de analytics, mas aumenta carga. A maioria dos servicos usa 302 para poder rastrear cliques.

#### Geracao da URL Curta

O comprimento da URL curta precisa ser determinado. Usando base62 (0-9, a-z, A-Z = 62 caracteres):

- 7 caracteres: 62^7 = ~3.5 trilhoes de combinacoes. Suficiente para 365 bilhoes de URLs.

**Abordagem 1: Hash + Colisao**

Aplica-se uma funcao hash (CRC32, MD5, SHA-1) a URL longa e pega-se os primeiros 7 caracteres. Se houver colisao (outra URL ja gerou o mesmo hash curto), concatena-se uma string predefinida a URL original e recalcula-se. Usa-se Bloom filter para verificacao rapida de colisoes (evita consultar o banco a cada geracao). Desvantagem: colisoes requerem re-hashing.

**Abordagem 2: Gerador de IDs Unicos + Base62**

Usa-se o gerador de IDs unicos (Snowflake, do capitulo 7) para gerar um ID numerico unico. Converte-se esse ID para base62. Vantagens: zero colisoes (IDs sao unicos por definicao), nao depende da URL original. Desvantagem: URLs curtas sao incrementais e previsiveis (pode ser mitigado com obfuscacao). Esta e a abordagem preferida.

Conversao base62: o ID numerico e dividido sucessivamente por 62; os restos mapeiam para os caracteres 0-9, a-z, A-Z.

#### Esquema de Dados

Tabela simples: id (PK, bigint), shortUrl (varchar, indexed), longUrl (varchar), createdAt (datetime), expiresAt (datetime).

#### Arquitetura de Alto Nivel

1. Cliente envia URL longa.
2. Servidor web verifica se URL ja existe no banco (ou Bloom filter).
3. Se nao existe: gera ID unico, converte para base62, salva no banco, retorna URL curta.
4. Para redirecionamento: cliente acessa URL curta, servidor busca URL original no cache (Redis) ou banco, retorna 301/302.

#### Otimizacoes e Escala

- **Cache**: URLs mais acessadas ficam em cache Redis (LRU eviction). Taxa de cache hit alta (distribuicao de acessos segue power law -- poucos links tem a maioria dos cliques).
- **Read replicas**: para suportar alto QPS de leitura.
- **Sharding de banco**: por hash da shortUrl para distribuicao uniforme.
- **Rate limiting**: para prevenir abuso (geracao massiva de URLs).
- **Analytics**: armazenar dados de cliques (timestamp, user-agent, referrer, IP) para analise posterior. Pode usar Kafka + processamento batch.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Hash + colisao | URL curta derivada do conteudo | Colisoes requerem re-hashing |
| ID unico + base62 | Zero colisoes | URLs previsiveis sem obfuscacao |
| 301 redirect | Menor carga no servidor | Sem analytics |
| 302 redirect | Analytics completo | Maior carga |

#### Licoes-Chave

- O URL shortener parece simples mas toca em varios topicos fundamentais: hashing, IDs distribuidos, caching, escala de banco.
- Base62 e preferido a base64 pois evita caracteres especiais (+, /, =) que podem causar problemas em URLs.
- Bloom filters sao extremamente uteis para verificacao rapida de existencia em conjuntos grandes.
- A escolha 301 vs 302 depende de se analytics sao necessarios.

---

### Capitulo 9: Design de um Web Crawler

#### Declaracao do Problema

Projetar um rastreador web (web crawler) que sistematicamente navega pela internet, baixando paginas web para indexacao (como o Googlebot). O crawler deve ser escalavel, educado (polite) e robusto.

#### Requisitos Funcionais

- Dado um conjunto de URLs semente (seed URLs), rastrear a web seguindo links.
- Baixar o conteudo HTML de cada pagina.
- Extrair e seguir hyperlinks encontrados nas paginas.
- Armazenar as paginas baixadas para processamento posterior.

#### Requisitos Nao-Funcionais

- Escalabilidade: rastrear bilhoes de paginas.
- Robustez: lidar com paginas malformadas, loops infinitos, conteudo duplicado, servidores lentos.
- Polidez (politeness): nao sobrecarregar servidores individuais.
- Extensibilidade: facil adicionar novos tipos de conteudo para processar.
- Conformidade: respeitar robots.txt e regras de rastreamento.

#### Estimativas

- 1 bilhao de paginas por mes.
- QPS: 1B / (30 x 86400) = ~400 paginas/segundo.
- Pico: ~800 paginas/segundo.
- Tamanho medio da pagina: 500 KB.
- Armazenamento mensal: 1B x 500KB = 500 TB/mes.
- Em 5 anos: 30 PB.

#### Arquitetura de Alto Nivel

Os componentes principais sao:

**1. Seed URLs**: ponto de partida do rastreamento. Estrategias: URLs de sites populares por categoria, ou URLs de diretorio web. A escolha das seeds impacta a cobertura.

**2. URL Frontier**: a fila de URLs a serem rastreadas. Nao e uma fila simples -- e o componente mais sofisticado do crawler. Contem duas funcionalidades criticas:

- **Priorizacao**: URLs mais importantes (sites populares, paginas atualizadas frequentemente) devem ser rastreadas primeiro. Implementado com multiplas filas com prioridades diferentes. Um seletor de prioridade determina em qual fila cada URL entra.
- **Polidez (Politeness)**: o crawler nao deve enviar muitas requisicoes ao mesmo host em curto periodo. Implementado com filas por host -- cada host tem sua propria fila FIFO, e um worker dedicado processa cada fila com delays entre requisicoes.

**3. HTML Downloader**: faz as requisicoes HTTP. Usa pool de conexoes para eficiencia. Antes de baixar, verifica o robots.txt do host (cacheado para evitar requisicoes repetidas). Robots.txt define quais paths podem ou nao ser rastreados.

**4. DNS Resolver**: resolve nomes de dominio para IPs. DNS e frequentemente um gargalo por sua latencia. Solucao: manter cache DNS local e fazer resolucoes em batch.

**5. Content Parser**: valida e parseia o HTML. Deve rodar em servidor separado para nao desacelerar o crawling. Detecta paginas malformadas e as descarta.

**6. Content Seen? (Deduplicacao de Conteudo)**: compara o hash do conteudo (usando simhash ou fingerprinting) para detectar paginas duplicadas ou quase duplicadas. Paginas com conteudo identico ja visto sao descartadas. Isso e critico pois ~30% das paginas web sao duplicatas.

**7. URL Extractor**: extrai links do HTML parseado. Converte URLs relativos em absolutos.

**8. URL Filter**: filtra URLs indesejados (extensoes de arquivo como .jpg, .exe; dominios bloqueados; URLs com patterns de armadilha).

**9. URL Seen? (Deduplicacao de URL)**: verifica se a URL ja foi visitada ou ja esta na frontier. Usa Bloom filter para verificacao eficiente em memoria. Sem isso, o crawler entraria em loops infinitos.

**10. URL Storage**: armazena URLs ja visitadas. Pode usar banco de dados em disco com Bloom filter em memoria como camada de verificacao rapida.

#### Fluxo Completo

1. Seeds sao adicionadas a URL Frontier.
2. Worker retira URL da frontier (respeitando polidez e prioridade).
3. Verifica robots.txt (cacheado).
4. Baixa a pagina via HTTP.
5. Parseia o conteudo.
6. Verifica se conteudo e duplicado (content seen?).
7. Se novo, armazena e extrai links.
8. Links extraidos passam por filtro, deduplicacao (URL seen?) e sao adicionados a frontier.
9. Ciclo repete.

#### Armadilhas (Spider Traps)

Paginas que geram URLs infinitos (ex: calendarios com links para datas futuras infinitas, URLs com parametros que mudam mas conteudo e o mesmo). Solucoes: limitar profundidade maxima de rastreamento por dominio, detectar patterns de URL suspeitos, blacklist manual de dominios problematicos.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| BFS vs DFS | BFS garante amplitude de cobertura | BFS usa mais memoria para a frontier |
| Bloom filter para dedup | O(1) verificacao, pouca memoria | Falsos positivos (URL nao rastreada pois parece duplicada) |
| Cache de robots.txt | Evita requisicoes repetidas | Pode estar desatualizado |

#### Licoes-Chave

- A URL Frontier e o cerebro do crawler -- priorizacao e polidez sao criticos.
- Deduplicacao (tanto de URL quanto de conteudo) economiza volumes massivos de recursos.
- Bloom filters sao indispensaveis para conjuntos de bilhoes de elementos.
- Robots.txt deve ser respeitado por questoes legais e eticas.
- Escalabilidade e alcancada distribuindo workers geograficamente e particionando a frontier por dominio.

---

### Capitulo 10: Design de um Sistema de Notificacoes

#### Declaracao do Problema

Projetar um sistema que envia notificacoes push (iOS, Android), SMS e e-mails para usuarios, suportando milhoes de notificacoes por dia com alta confiabilidade.

#### Requisitos Funcionais

- Suportar multiplos tipos: push notification (iOS APNs, Android FCM), SMS, e-mail.
- Notificacoes podem ser triggered por eventos do sistema ou agendadas.
- Suportar soft real-time (pequeno atraso e aceitavel).
- Usuarios podem configurar preferencias (opt-out de certos tipos).
- Suportar dispositivos multiplos por usuario.

#### Requisitos Nao-Funcionais

- Alta confiabilidade: notificacoes nao devem ser perdidas.
- Escala: milhoes de notificacoes por dia.
- Latencia aceitavel: menos de 1 segundo para push, minutos para email/SMS.
- Taxa de entrega alta.

#### Tipos de Notificacao e Seus Provedores

**Push iOS**: envia para Apple Push Notification Service (APNs). Requer device token do dispositivo, payload (JSON com titulo, corpo, badge), e autenticacao com certificado da Apple.

**Push Android**: envia para Firebase Cloud Messaging (FCM). Similar ao APNs mas com API do Google.

**SMS**: envia via provedores como Twilio ou Nexmo. Custos por mensagem sao significativos.

**E-mail**: envia via servicos como SendGrid, Amazon SES ou Mailchimp. Precisa cuidar de reputacao de IP, SPF/DKIM para evitar spam.

#### Arquitetura de Alto Nivel

**Componentes:**

1. **Servicos chamadores**: microservicos internos ou jobs que triggeram notificacoes (ex: servico de billing, servico de amizade).

2. **Notification Service**: ponto central que recebe requisicoes de notificacao, valida dados, consulta preferencias do usuario e enfileira mensagens. Expoe APIs e e stateless para escalar horizontalmente.

3. **Banco de preferencias de usuario**: armazena configuracoes do usuario (quais tipos de notificacao aceita, device tokens, e-mail, telefone). Consultado antes de cada envio.

4. **Filas de mensagens**: filas separadas por tipo (fila de push iOS, fila de push Android, fila de SMS, fila de e-mail). Separacao garante que lentidao em um canal nao afete outros.

5. **Workers**: consumidores das filas que fazem o envio efetivo para os provedores (APNs, FCM, Twilio, SendGrid). Podem escalar independentemente por canal.

6. **Banco de logs/tracking**: armazena historico de notificacoes enviadas, status de entrega, timestamps.

#### Confiabilidade e Prevencao de Perda

- **Persistencia**: notificacoes sao salvas em banco de dados antes do envio. Se o worker falha, a notificacao e reenviada.
- **Retry com backoff exponencial**: se o provedor retorna erro temporario (5xx), o worker tenta novamente com intervalos crescentes (1s, 2s, 4s, 8s...).
- **Dead letter queue**: apos N tentativas falhas, a mensagem vai para uma fila de mensagens mortas para investigacao manual.
- **Deduplicacao**: usa um ID unico por notificacao (event ID). Antes de enviar, verifica se o ID ja foi processado. Garante idempotencia -- a mesma notificacao nao e enviada duas vezes.

#### Rate Limiting

Protecao contra envio excessivo ao mesmo usuario. Um usuario nao deve receber mais do que X notificacoes por dia. Implementado como rate limiter (capitulo 4) por usuario, por tipo de notificacao.

#### Monitoramento e Analytics

Metricas essenciais: taxa de entrega, taxa de abertura (para push e e-mail), taxa de clique, taxa de opt-out. Dashboards monitoram a saude do sistema: tamanho das filas, latencia de envio, taxa de erros por provedor.

#### Template System

Notificacoes frequentemente seguem templates predefinidos com variaveis (ex: "Ola {nome}, seu pedido {id} foi enviado!"). Usar um sistema de templates evita duplicacao e facilita manutencao. Templates sao cacheados.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Filas separadas por canal | Isolamento de falhas | Mais infra para gerenciar |
| Retry com backoff | Resiliencia a falhas temporarias | Pode aumentar latencia |
| Rate limiting por usuario | Evita spam e opt-outs | Notificacoes importantes podem ser bloqueadas |

#### Licoes-Chave

- Filas de mensagens sao essenciais para desacoplar producao e envio de notificacoes.
- Idempotencia (deduplicacao via event ID) e critica -- duplicatas irritam usuarios.
- Cada canal (push, SMS, e-mail) tem suas peculiaridades e deve ser tratado separadamente.
- Monitoramento proativo evita que problemas silenciosos afetem milhoes de usuarios.

---

### Capitulo 11: Design de um Sistema de News Feed

#### Declaracao do Problema

Projetar o sistema de news feed (timeline) de uma rede social como Facebook ou Twitter. O feed mostra posts de amigos/seguidos em ordem cronologica ou por relevancia.

#### Requisitos Funcionais

- Usuarios podem publicar posts (texto, imagens, video).
- O feed agrega posts de amigos/seguidos.
- O feed e personalizado por usuario.
- Suportar ordenacao cronologica (simplicidade) ou por relevancia (ranking).

#### Requisitos Nao-Funcionais

- Baixa latencia: o feed deve carregar em menos de 500ms.
- Alta disponibilidade.
- Consistencia eventual e aceitavel (um post pode demorar poucos segundos para aparecer no feed de todos os amigos).

#### APIs

- POST /api/v1/feed/publish: publica um post (body: conteudo, imagens, etc).
- GET /api/v1/feed?user_id=X: retorna o feed personalizado do usuario X.

#### Arquitetura: Publicacao (Feed Publishing)

Quando um usuario publica um post:

1. Post e salvo no banco de dados e no cache de posts.
2. O servico de fanout distribui o post para os feeds dos seguidores.

**Fanout on Write (Push Model)**:

O post e pre-computado e inserido diretamente no cache do news feed de cada seguidor no momento da publicacao. Fluxo: usuario publica -> servico busca lista de amigos/seguidores -> para cada seguidor, insere o post_id no cache do feed desse seguidor.

Vantagens: leitura do feed e extremamente rapida (o feed ja esta pronto, basta buscar no cache). Latencia de leitura e minima. E a abordagem ideal para usuarios com poucos seguidores.

Desvantagens: para usuarios com muitos seguidores (celebridades com milhoes), o fanout e extremamente caro e lento (chamado de "hotkey problem"). Se o usuario raramente e ativo, computar o feed proativamente desperdica recursos.

**Fanout on Read (Pull Model)**:

O feed e gerado on-demand quando o usuario o solicita. O sistema busca os posts mais recentes de todos os amigos/seguidos e os ordena.

Vantagens: nenhum trabalho desperdicado para usuarios inativos. Publicacao e instantanea (sem fanout).

Desvantagens: leitura do feed e lenta pois requer merge de posts de muitos amigos em tempo real.

**Abordagem Hibrida (recomendada)**:

Para a maioria dos usuarios: fanout on write (feed pre-computado). Para usuarios com muitos seguidores (celebridades): fanout on read (os posts da celebridade sao buscados em tempo real e mesclados ao feed pre-computado na hora da leitura). Isso evita o custo explosivo de fazer fanout para milhoes de seguidores.

#### Arquitetura: Leitura do Feed (News Feed Retrieval)

1. Usuario solicita o feed.
2. Servico busca lista de post_ids do cache do news feed do usuario.
3. Para cada post_id, busca dados completos (texto, imagens, info do autor) do cache de posts.
4. Para celebridades seguidas (fanout on read), busca os posts mais recentes e faz merge.
5. Aplica ranking (se aplicavel) e retorna.

#### Cache Architecture

O cache e multicamada:

- **News Feed Cache**: armazena os IDs dos posts no feed de cada usuario. Estrutura: user_id -> lista ordenada de post_ids.
- **Content Cache**: armazena dados completos dos posts (hot content).
- **Social Graph Cache**: armazena relacoes de amizade/follow para lookup rapido.
- **Action Cache**: armazena acoes do usuario (likes, comentarios) para personalizar o feed.
- **Counters Cache**: armazena contadores (likes, shares, comentarios) com atualizacao assincrona.

#### Trade-offs

| Aspecto | Fanout on Write | Fanout on Read |
|---|---|---|
| Latencia de leitura | Muito baixa (pre-computado) | Alta (merge em tempo real) |
| Custo de publicacao | Alto para celebridades | Baixo |
| Recursos desperdicados | Sim (usuarios inativos) | Nao |
| Consistencia | Feed pode ficar levemente desatualizado | Sempre atualizado |

#### Licoes-Chave

- A abordagem hibrida (fanout on write para usuarios normais, fanout on read para celebridades) e o padrao da industria.
- Cache multicamada e essencial para performance.
- O grafo social e consultado intensamente -- cachea-lo e obrigatorio.
- Ranking e um problema de ML que envolve sinais como afinidade, tempo, tipo de conteudo.

---

### Capitulo 12: Design de um Sistema de Chat

#### Declaracao do Problema

Projetar um sistema de chat em tempo real similar ao WhatsApp, Facebook Messenger ou Slack, suportando mensagens 1:1 e em grupo, indicadores de presenca online e sincronizacao de mensagens entre dispositivos.

#### Requisitos Funcionais

- Chat 1:1 entre dois usuarios.
- Chat em grupo (ate centenas de membros).
- Indicador de presenca online (online/offline/ultimo acesso).
- Suporte a multiplos dispositivos por usuario.
- Notificacoes push para mensagens quando o usuario esta offline.
- Persistencia de historico de mensagens.

#### Requisitos Nao-Funcionais

- Latencia muito baixa para entrega de mensagens (< 100ms entre usuarios online).
- Alta disponibilidade.
- Ordenacao de mensagens garantida.
- Consistencia eventual e aceitavel entre dispositivos, mas dentro de uma conversa a ordem deve ser preservada.

#### Protocolo de Comunicacao

**HTTP (polling, long polling) vs WebSocket**:

- **Polling**: cliente pergunta ao servidor repetidamente "tem mensagem nova?". Ineficiente -- a maioria das requisicoes retorna vazio, desperdicando recursos.
- **Long polling**: cliente faz requisicao e o servidor segura a conexao ate ter dados ou timeout. Melhor que polling, mas o servidor que recebeu a mensagem pode nao ser o que tem a conexao aberta com o destinatario. Alem disso, manter muitas conexoes long-poll abertas consome recursos do servidor.
- **WebSocket**: conexao bidirecional persistente. Apos um handshake HTTP inicial, a conexao e "upgraded" para WebSocket. O servidor pode enviar mensagens ao cliente a qualquer momento sem que o cliente precise perguntar. E a escolha ideal para chat. Eficiente em recursos, baixa latencia, bidirecional.

Na pratica, WebSocket e usado para mensagens em tempo real. HTTP tradicional e mantido para funcionalidades que nao exigem tempo real (login, signup, gerenciamento de perfil, upload de midias).

#### Arquitetura de Alto Nivel

**Componentes:**

1. **Chat Servers (Stateful)**: mantem conexoes WebSocket com clientes. Cada servidor mantem um mapeamento de quais usuarios estao conectados a ele. Sao stateful por natureza (a conexao persistente e estado). Escalar requer balanceamento inteligente.

2. **API Servers (Stateless)**: lidam com funcionalidades nao real-time: autenticacao, perfil de usuario, gerenciamento de grupos, busca de historico.

3. **Notification Service**: envia push notifications para usuarios offline.

4. **Key-Value Store para Mensagens**: armazena historico de mensagens. Key-value stores como HBase ou Cassandra sao preferidos sobre bancos relacionais porque: o padrao de acesso e sequencial (buscar mensagens recentes de uma conversa), write-heavy, e a escala pode ser enorme. Chave: (conversation_id, message_id) com message_id ordenado por tempo.

5. **Service Discovery**: servico como Zookeeper que informa qual chat server o cliente deve se conectar, baseado em criterios como localizacao geografica e carga do servidor.

6. **Presence Servers**: gerenciam o status online/offline dos usuarios.

#### Chat 1:1 -- Fluxo

1. Usuario A envia mensagem para Usuario B.
2. A mensagem vai pela conexao WebSocket do Usuario A ate o Chat Server 1.
3. O Chat Server 1 gera um message_id (usando gerador de IDs local, ordenavel por tempo).
4. A mensagem e persistida no key-value store.
5. Se Usuario B esta online: o servico de roteamento encontra em qual Chat Server o Usuario B esta conectado e encaminha a mensagem.
6. Se Usuario B esta offline: a mensagem e enviada ao Notification Service para push notification.

#### Chat em Grupo -- Fluxo

Para grupos pequenos/medios (ate ~100 membros, como no WeChat):

1. Usuario A envia mensagem ao grupo.
2. A mensagem e copiada na fila de mensagens de cada membro do grupo.
3. Cada membro recebe a mensagem de sua propria fila.

Essa abordagem (fanout) funciona para grupos pequenos. Para grupos muito grandes (milhares de membros, como channels do Slack), a mensagem e persistida uma vez e os membros a buscam por pull.

#### Message ID e Ordenacao

IDs de mensagem devem ser:
- Unicos dentro de uma conversa (nao precisam ser globalmente unicos).
- Ordenaveis por tempo (mensagens mais recentes tem IDs maiores).

Abordagem: usar auto-increment local por conversa (mais simples que Snowflake global) ou usar Snowflake adaptado com sequence number local.

#### Indicador de Presenca Online

Mecanismo de heartbeat: o cliente envia um heartbeat ao Presence Server a cada X segundos (ex: 5s). Se o servidor nao recebe heartbeat por Y segundos (ex: 30s), marca o usuario como offline.

Para notificar amigos sobre mudancas de status: em grupos pequenos, fanout direto. Para usuarios com muitos amigos, usar um modelo sob demanda -- o status e atualizado apenas quando o amigo abre a lista de contatos ou o chat.

#### Sincronizacao entre Dispositivos

Cada dispositivo mantem um cur_max_message_id (o ID da mensagem mais recente que ja recebeu). Ao se reconectar, busca todas as mensagens com ID > cur_max_message_id para cada conversa. Isso garante que nenhuma mensagem e perdida, mesmo que o dispositivo esteja offline por horas.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| WebSocket | Baixa latencia, bidirecional | Stateful, mais complexo de escalar |
| Key-value store para mensagens | Alta performance de escrita, escala horizontal | Sem JOINs, queries complexas sao dificeis |
| Fanout para grupos | Simples, baixa latencia | Nao escala para grupos muito grandes |
| Heartbeat para presenca | Simples e eficiente | Pode ter delay na deteccao de offline |

#### Licoes-Chave

- WebSocket e o protocolo padrao para comunicacao real-time bidirecional.
- Chat servers sao inerentemente stateful -- isso complica escalabilidade e requer service discovery.
- A escolha de storage (key-value) e ditada pelo padrao de acesso write-heavy e leituras sequenciais.
- Presenca online via heartbeat e simples mas deve ser otimizada para nao gerar trafego excessivo de fanout.

---

### Capitulo 13: Design de um Sistema de Search Autocomplete

#### Declaracao do Problema

Projetar o sistema de autocomplete/typeahead de uma barra de busca (como o Google Search), que sugere os termos mais populares conforme o usuario digita.

#### Requisitos Funcionais

- Conforme o usuario digita, retornar as top-K sugestoes que comecam com o prefixo digitado.
- Sugestoes baseadas em popularidade (frequencia historica de busca).
- Resposta rapida o suficiente para parecer instantanea.

#### Requisitos Nao-Funcionais

- Latencia ultra-baixa: < 100ms por sugestao (idealmente < 50ms).
- Alta disponibilidade.
- Escalabilidade: bilhoes de queries por dia.
- Sugestoes devem ser razoavelmente atualizadas (nao precisam ser real-time).

#### Estimativas

- 10 milhoes de DAU, 10 buscas/dia/usuario = 100 milhoes queries/dia.
- Cada query tem ~4 palavras, ~20 caracteres. Cada caractere dispara uma requisicao de autocomplete.
- QPS de autocomplete: 100M x 20 / 86400 = ~24000 QPS. Pico: ~48000 QPS.

#### Estrutura de Dados: Trie (Prefix Tree)

A trie e a estrutura ideal para busca por prefixo. Cada no representa um caractere. Caminhos da raiz ate nos folha formam strings completas. Cada no que representa o fim de uma query armazena a frequencia (popularidade) daquela query.

**Trie basica**: para encontrar top-K sugestoes dado um prefixo, percorre-se a trie ate o no do prefixo, e entao busca-se todas as subarvores para encontrar as K queries mais populares. Problema: percorrer toda a subarvore e lento (pode ter milhoes de nos).

**Trie otimizada**: cada no armazena diretamente as top-K queries mais populares em sua subarvore. Assim, ao chegar no no do prefixo, as sugestoes ja estao prontas, sem necessidade de percorrer a subarvore. Trade-off: mais memoria (cada no armazena K queries), mas lookup e O(comprimento do prefixo).

#### Coleta de Dados (Data Gathering)

Em tempo real nao e pratico atualizar a trie a cada query (bilhoes de atualizacoes/dia seriam inviáveis). Em vez disso:

1. **Analytics Logs**: toda query e logada com timestamp.
2. **Aggregation Service**: periodicamente (ex: semanalmente) agrega os logs, contabilizando a frequencia de cada query.
3. **Workers**: reconstroem ou atualizam a trie com base nos dados agregados.
4. A nova trie e publicada (snapshot) e os servidores a adotam.

Essa abordagem batch e suficiente pois as tendencias de busca nao mudam drasticamente em minutos (exceto trending topics, que podem ter tratamento especial).

#### Storage da Trie

A trie e grande demais para caber na memoria de um unico servidor. Duas abordagens:

1. **Serializar e armazenar em banco/storage**: a trie e serializada (DFS), armazenada em banco de dados, e carregada em memoria no startup. Snapshots semanais.
2. **Sharding por prefixo**: dividir a trie entre multiplos servidores. Ex: servidor 1 cuida de prefixos "a" a "f", servidor 2 de "g" a "l", etc. A distribuicao pode ser desigual (mais queries comecam com certas letras), entao sharding baseado em dados reais e preferivel ao alfabetico uniforme.

#### Otimizacoes no Cliente

- Nao enviar requisicao a cada tecla; usar debounce (esperar ~150ms apos a ultima tecla).
- Cachear resultados no browser. Se o usuario digitou "din" e recebeu sugestoes, ao digitar "dine" pode filtrar localmente as sugestoes de "din".
- Enviar a requisicao somente se o prefixo tiver comprimento minimo (ex: >= 2 caracteres).

#### Filtragem de Conteudo Inapropriado

Queries ofensivas, perigosas ou protegidas por direitos autorais devem ser filtradas. Implementado com uma camada de filtro entre a trie e a resposta. Queries em uma blacklist sao removidas das sugestoes.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Trie com top-K em cada no | Lookup O(prefixo) | Mais memoria |
| Atualizacao batch (semanal) | Simples, eficiente | Nao captura trending topics imediatamente |
| Sharding por prefixo | Escala horizontal | Distribuicao desigual de carga |

#### Licoes-Chave

- Tries sao a estrutura perfeita para busca por prefixo, mas precisam de otimizacao (pre-computar top-K por no).
- A coleta de dados e a atualizacao da trie sao desacopladas da leitura (CQRS natural).
- Otimizacoes no cliente (debounce, cache local) reduzem drasticamente a carga no servidor.
- A trie em si e um "snapshot" que pode ser versionado e distribuido como um artefato.

---

### Capitulo 14: Design do YouTube

#### Declaracao do Problema

Projetar um sistema de compartilhamento de videos como YouTube ou Netflix, focando em upload de video, streaming e funcionalidades core.

#### Requisitos Funcionais

- Upload de videos.
- Streaming suave de videos.
- Suporte a multiplas resolucoes e formatos.
- Suporte a dispositivos diversos (mobile, desktop, smart TV).
- Listagem e busca de videos.

#### Requisitos Nao-Funcionais

- Alta disponibilidade.
- Upload rapido (o usuario nao deve esperar demais).
- Streaming sem buffering (baixa latencia de inicio, throughput sustentado).
- Escalabilidade: centenas de milhoes de videos, bilhoes de views por dia.
- Custo eficiente em infra (video e extremamente storage-intensive e bandwidth-intensive).

#### Estimativas

- 5 milhoes de videos assistidos por dia.
- 10% dos usuarios fazem upload: taxa de upload = ~1650 videos/dia.
- Tamanho medio de video original: 300 MB.
- Armazenamento diario de upload: ~500 GB. Anual: ~180 TB.
- CDN cost e o custo dominante (streaming de video e 90%+ do trafego).

#### Arquitetura de Alto Nivel

Tres fluxos principais: upload de video, processamento de video, streaming de video.

**Upload de Video:**

1. Cliente faz upload do video original para o upload service.
2. O video e armazenado em storage distribuido (ex: S3, GCS, Azure Blob).
3. O servico de transcodificacao e notificado (via fila de mensagens).
4. Metadata do video (titulo, descricao, tags, thumbnail, URL) e salva no metadata DB e cache.

**Processamento/Transcodificacao de Video:**

O video original e convertido em multiplos formatos e resolucoes. Isso e crucial pois dispositivos e conexoes diferentes precisam de formatos diferentes.

Componentes:

- **Preprocessing**: extrai metadata do video (codec, resolucao, duracao).
- **DAG Scheduler**: o processamento de video e modelado como um DAG (Directed Acyclic Graph) de tarefas. Exemplo: video split -> encode em paralelo para 360p, 720p, 1080p -> merge -> gerar thumbnails -> gerar watermark.
- **Task Workers**: executam as tarefas do DAG. Workers especializados para encoding, thumbnail generation, etc.
- **Encoded Storage**: videos transcodificados sao armazenados em blob storage.
- **Completion Handler**: ao finalizar, atualiza metadata DB e notifica o usuario.

Encoding:

- Containers: .avi, .mov, .mp4 (container encapsula video, audio e metadata).
- Codecs: H.264, VP9, HEVC (codec comprime e descomprime o video).
- Bitrate adaptativo: o mesmo video e codificado em multiplas bitrates. O player escolhe dinamicamente baseado na largura de banda do usuario (ex: DASH, HLS).

**Streaming de Video:**

Videos nao sao baixados inteiramente antes de serem reproduzidos. Sao transmitidos em streaming -- o player baixa e reproduz chunks sequencialmente.

Protocolos de streaming:
- MPEG-DASH (Dynamic Adaptive Streaming over HTTP).
- Apple HLS (HTTP Live Streaming).
- Microsoft Smooth Streaming.

Todos funcionam de forma similar: o video e dividido em segmentos pequenos (2-10 segundos cada) em multiplas qualidades. Um manifest file lista os segmentos disponiveis. O player baixa segmentos adaptativamente.

Videos sao servidos via CDN. CDN e absolutamente essencial para video -- sem CDN, o servidor de origem seria sobrecarregado e a latencia para usuarios distantes seria inaceitavel.

#### Otimizacoes

- **Upload com resumption**: se o upload falha no meio, o cliente resume de onde parou (upload em chunks com checkpoints). Protocolo como tus.
- **Upload paralelo**: dividir o video em chunks e fazer upload de multiplos chunks simultaneamente.
- **Pre-signed URLs**: o upload vai direto do cliente para o blob storage (S3) usando URLs pre-assinadas, sem passar pelo application server. Reduz carga e latencia.
- **CDN com origin fallback**: videos populares no cache da CDN; videos raramente acessados buscados da origem.
- **Cost optimization**: colocar videos populares na CDN e videos antigos/raros em storage mais barato. Usar CDN regional apenas onde ha demanda. Videos virais podem usar CDN global.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Transcodificacao para multiplas resolucoes | Suporta todos os dispositivos | Custo de processamento e storage multiplicado |
| CDN para streaming | Baixa latencia, alta throughput | Custo alto (maior despesa operacional) |
| DAG para processamento | Flexivel, paralelizavel | Complexidade de orquestracao |
| Upload direto ao blob storage | Descarrega application servers | Complexidade de autorizacao (pre-signed URLs) |

#### Licoes-Chave

- CDN e o custo dominante e a decisao arquitetural mais impactante em sistemas de video.
- Transcodificacao e um pipeline complexo melhor modelado como DAG.
- Streaming adaptativo (DASH/HLS) e obrigatorio para experiencia de usuario aceitavel.
- Videos sao servidos como segmentos, nao como arquivos inteiros.

---

### Capitulo 15: Design do Google Drive

#### Declaracao do Problema

Projetar um servico de armazenamento e sincronizacao de arquivos em nuvem como Google Drive, Dropbox ou OneDrive.

#### Requisitos Funcionais

- Upload e download de arquivos.
- Sincronizacao automatica entre dispositivos.
- Versionamento de arquivos (historico de revisoes).
- Compartilhamento de arquivos com outros usuarios.
- Notificacoes quando arquivos sao modificados.

#### Requisitos Nao-Funcionais

- Confiabilidade: dados nunca devem ser perdidos (durabilidade).
- Sincronizacao rapida e consistente.
- Eficiencia de largura de banda (minimizar dados transferidos).
- Suportar arquivos grandes (GBs).
- Alta disponibilidade.

#### Estimativas

- 50 milhoes de usuarios, 10 milhoes DAU.
- Cada usuario: 10 GB de storage, ~500 arquivos.
- Total: 500 PB de storage.
- Upload QPS: se 2 atualizacoes por usuario por dia: 10M x 2 / 86400 = ~230 QPS.

#### Arquitetura de Alto Nivel

**Componentes:**

1. **Block Servers**: recebem arquivos dos clientes. Em vez de fazer upload do arquivo inteiro, o arquivo e dividido em blocos (blocks) de tamanho fixo (ex: 4 MB). Apenas blocos modificados sao transferidos (delta sync). Cada bloco e comprimido e encriptado antes do upload.

2. **Cloud Storage (Blob Storage)**: armazena os blocos de dados (S3, GCS). Altamente duravel (11 noves de durabilidade).

3. **Metadata Database**: armazena metadados de arquivos (nome, tamanho, dono, permissoes, versao), informacoes de blocos (quais blocos compoem cada arquivo, hash de cada bloco), informacoes de workspace/compartilhamento. Banco relacional (MySQL com sharding) pela necessidade de consistencia forte e ACID.

4. **Metadata Cache**: Redis para cache de metadados frequentemente acessados.

5. **Upload/Download Service**: coordena o fluxo de upload e download. Para upload: recebe blocos, armazena no blob storage, atualiza metadata. Para download: consulta metadata para encontrar blocos necessarios e os entrega.

6. **Notification Service**: notifica outros dispositivos do mesmo usuario (e usuarios com compartilhamento) quando um arquivo e modificado. Usa long polling ou WebSocket. Long polling e preferido aqui por ser mais simples e a frequencia de atualizacoes nao ser tao alta quanto chat.

7. **Offline Backup Queue**: quando um dispositivo esta offline, as mudancas pendentes sao armazenadas em uma fila. Quando o dispositivo reconecta, as mudancas sao sincronizadas.

#### Delta Sync e Block-Level Deduplication

Em vez de enviar o arquivo inteiro a cada modificacao, apenas os blocos alterados sao transferidos. Mecanismo:

1. O arquivo e dividido em blocos de tamanho fixo.
2. Cada bloco recebe um hash (SHA-256).
3. Ao modificar o arquivo, recalcula-se os hashes dos blocos.
4. Apenas blocos cujo hash mudou sao enviados ao servidor.

Isso economiza drasticamente largura de banda. Para um arquivo de 1 GB onde apenas 1 bloco de 4 MB mudou, transferem-se apenas 4 MB em vez de 1 GB.

**Block-level deduplication**: se dois usuarios tem o mesmo bloco (mesmo hash), ele e armazenado apenas uma vez no blob storage. Extremamente eficiente para arquivos padrao (templates, libraries).

#### Resolucao de Conflitos

Quando dois usuarios editam o mesmo arquivo simultaneamente:

1. O primeiro a salvar "ganha" -- sua versao e salva normalmente.
2. O segundo recebe um conflito. O sistema cria duas versoes: a "vencedora" e a "conflitante".
3. O usuario e notificado e deve resolver manualmente (merge ou escolha de versao).

Cada versao e mantida no historico. Versoes antigas podem ser restauradas.

#### Fluxo de Sincronizacao

1. Cliente detecta mudanca local em um arquivo.
2. Divide o arquivo em blocos, calcula hashes.
3. Compara hashes com os armazenados no servidor (via metadata).
4. Envia apenas blocos novos/modificados.
5. Servidor atualiza metadata com novos blocos e nova versao.
6. Notification service avisa outros dispositivos.
7. Outros dispositivos baixam apenas os blocos novos/modificados.

#### Confiabilidade e Durabilidade

- Blob storage com replicacao multi-region (3+ copias em regioes diferentes).
- Metadata DB com replicacao sincrona (master-slave).
- Backups regulares de metadata.
- Checksums em cada bloco para detectar corrupcao.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Delta sync (block-level) | Economiza largura de banda drasticamente | Complexidade de calculo de diff |
| Long polling para notificacoes | Simples, funcional | Menos eficiente que WebSocket para alta frequencia |
| Versionamento de arquivos | Seguranca contra perda de dados | Custo de storage para multiplas versoes |
| Consistencia forte em metadata | Evita conflitos silenciosos | Maior latencia em operacoes de metadata |

#### Licoes-Chave

- Delta sync no nivel de blocos e a inovacao chave que torna a sincronizacao viavel para arquivos grandes.
- Deduplicacao de blocos por hash economiza storage significativo.
- A separacao entre metadata (banco relacional, consistencia forte) e dados (blob storage, alta durabilidade) e um pattern arquitetural fundamental.
- Resolucao de conflitos deve ser delegada ao usuario quando merge automatico nao e possivel.

---

## Volume 2

---

### Proximity Service (Yelp)

#### Declaracao do Problema

Projetar um servico que, dada a localizacao do usuario (latitude, longitude), retorna estabelecimentos proximos (restaurantes, lojas, hospitais) dentro de um raio especificado.

#### Requisitos Funcionais

- Dado lat/long e um raio, retornar lista de negocios proximos.
- Donos de negocios podem adicionar, atualizar e remover informacoes (operacoes nao real-time).
- Usuarios podem visualizar detalhes de um negocio.

#### Requisitos Nao-Funcionais

- Baixa latencia (usuarios esperam resultados instantaneos em apps de mapa).
- Alta disponibilidade.
- Escala: centenas de milhoes de buscas por dia.
- Dados de negocios nao mudam em tempo real (eventual consistency e aceitavel para atualizacoes de negocios).

#### Abordagens de Indexacao Geoespacial

O desafio central: busca por proximidade nao funciona bem com indices tradicionais de banco de dados. Indexar latitude e longitude separadamente nao ajuda eficientemente.

**1. Geohashing**

Divide a superficie da Terra em uma grade, codificando cada celula como uma string. O geohash e gerado intercalando bits de latitude e longitude, resultando em uma string base32. Quanto mais longa a string, menor a celula (maior a precisao).

- Geohash de 4 caracteres: celula de ~39km x 20km.
- Geohash de 5 caracteres: ~5km x 5km.
- Geohash de 6 caracteres: ~1.2km x 600m.

Propriedade chave: celulas com prefixos iguais estao proximas geograficamente (na maioria dos casos). Buscar negocios proximos = buscar por prefixo do geohash.

Problema de fronteira: dois pontos muito proximos podem ter geohashes completamente diferentes se estiverem em lados opostos de uma fronteira de celula. Solucao: buscar nao apenas na celula do usuario, mas tambem nas 8 celulas adjacentes.

**2. Quadtree**

Estrutura de arvore que recursivamente subdivide o espaco 2D em 4 quadrantes. A subdivisao continua ate que cada quadrante tenha um numero aceitavel de negocios (ex: < 100). Quadrantes em areas densas (centros urbanos) sao muito pequenos; em areas rurais, sao grandes.

Construcao: a quadtree inteira e construida em memoria no startup do servidor. Para 200 milhoes de negocios, a arvore usa ~1.7 GB de RAM. E read-only durante operacoes normais. Atualizacoes sao feitas por rebuild periodico (ou incremental com cuidado).

Busca: partir do no raiz, descer ate o quadrante que contem a posicao do usuario, retornar negocios nesse quadrante. Se nao ha negocios suficientes, expandir para quadrantes vizinhos.

**3. Outras abordagens**: R-tree (usado pelo PostGIS), S2 (biblioteca do Google que usa celulas em uma esfera), KD-tree.

#### Arquitetura de Alto Nivel

1. **Location-Based Service (LBS)**: servico stateless que recebe lat/long e raio, consulta o indice geoespacial e retorna negocios proximos. Read-heavy, QPS alto. Escalado com multiplas replicas.

2. **Business Service**: CRUD para dados de negocios. Write QPS baixo. Atualizacoes propagadas para o LBS periodicamente.

3. **Database**: banco relacional (MySQL/PostgreSQL) para dados de negocios. Tabela de geohash com indice no campo geohash para buscas eficientes.

4. **Cache**: Redis para dados frequentemente acessados (detalhes de negocios populares, resultados de buscas comuns).

#### Fluxo de Busca com Geohash

1. Cliente envia lat/long e raio desejado.
2. Servidor converte lat/long para geohash com precisao adequada ao raio.
3. Busca no banco/cache por negocios com geohash correspondente e nas 8 celulas adjacentes.
4. Filtra por distancia real (calculo de distancia usando formula de Haversine) para remover falsos positivos.
5. Ordena por distancia e retorna.

#### Trade-offs

| Abordagem | Vantagem | Desvantagem |
|---|---|---|
| Geohash | Simples, indice padrao de BD, facil de shardear | Problema de fronteira (mitigavel com vizinhos) |
| Quadtree | Adapta-se a densidade variavel | Complexo, construcao em memoria, updates dificeis |
| Geohash + BD | Funciona com bancos existentes | Menos otimizado que quadtree para buscas de raio |

#### Licoes-Chave

- Geohashing e a abordagem mais pratica e a mais usada em producao por sua simplicidade e compatibilidade com bancos de dados existentes.
- Sempre busque nas celulas adjacentes para mitigar o problema de fronteira.
- A quadtree e excelente quando a densidade de pontos varia enormemente.
- O calculo final de distancia (Haversine) e sempre necessario como filtro pos-busca.

---

### Nearby Friends

#### Declaracao do Problema

Projetar uma feature que mostra amigos proximos em tempo real em um app de mapa/rede social (como o "Nearby Friends" do Facebook).

#### Requisitos Funcionais

- Mostrar amigos que estao proximos (dentro de um raio, ex: 5 milhas) com suas localizacoes atualizadas.
- A lista atualiza a cada poucos segundos.
- Usuarios podem ativar/desativar a feature.

#### Requisitos Nao-Funcionais

- Baixa latencia para atualizacoes de localizacao (quasi real-time).
- Escala: centenas de milhoes de usuarios.
- Eficiencia: nao desperdicar bateria/dados em mobile.
- Eventual consistency e aceitavel (posicoes podem ter poucos segundos de atraso).

#### Abordagem com WebSocket e Redis Pub/Sub

**Fluxo:**

1. O app do usuario envia atualizacoes de localizacao periodicamente (ex: a cada 30 segundos) via WebSocket para o servidor.

2. O WebSocket Handler Server recebe a atualizacao e:
   - Armazena a localizacao no Location Cache (Redis). Chave: user_id, valor: {lat, long, timestamp}. TTL curto (ex: 1 minuto) para auto-limpeza de usuarios que param de enviar.
   - Publica a atualizacao em um canal Redis Pub/Sub dedicado ao usuario (ex: canal "user_123_location").

3. Os amigos do usuario que ativaram "nearby friends" estao inscritos (subscribed) no canal desse usuario. Quando a localizacao e publicada, eles recebem a atualizacao.

4. O servidor do amigo calcula a distancia. Se o usuario esta dentro do raio, a localizacao e exibida no app. Se esta fora, e ignorada.

**Problema de escala do Redis Pub/Sub**: cada usuario ativo cria um canal. Com 100 milhoes de usuarios, sao 100 milhoes de canais. Um unico servidor Redis nao suporta. Solucao: shardear o Redis Pub/Sub. Usar consistent hashing para mapear user_id ao servidor Redis responsavel pelo seu canal. Cada servidor Redis gerencia um subconjunto dos canais.

**Calculo de quem e amigo**: antes de inscrever em canais, o servidor busca a lista de amigos do usuario. Nao faz sentido inscrever-se em canais de nao-amigos. A lista de amigos e cacheada.

#### Otimizacoes

- **Frequency capping**: em vez de enviar localizacao a cada segundo, enviar a cada 30 segundos. Reduz carga e consumo de bateria.
- **Distancia minima de mudanca**: so enviar atualizacao se o usuario se moveu mais de X metros desde a ultima atualizacao.
- **Quadtree para matching eficiente**: alternativamente, manter uma quadtree de usuarios ativos para buscar proximos sem pub/sub por usuario.

#### Licoes-Chave

- Redis Pub/Sub e eficiente para fan-out de localizacao a um grupo pequeno (lista de amigos).
- WebSocket e essencial para comunicacao bidirecional e atualizacoes quasi real-time.
- O Location Cache com TTL curto serve como indicador implicito de atividade.
- Sharding do Pub/Sub e necessario para escala.

---

### Google Maps

#### Declaracao do Problema

Projetar um sistema de navegacao e mapas como Google Maps, Waze ou Apple Maps, com funcionalidades de renderizacao de mapas, geocoding e roteamento.

#### Requisitos Funcionais

- Renderizacao de mapa com zoom e pan.
- Geocoding (endereco -> lat/long) e reverse geocoding (lat/long -> endereco).
- Roteamento: dado origem e destino, encontrar a melhor rota (menor tempo ou distancia).
- Estimativa de tempo de chegada (ETA).

#### Requisitos Nao-Funcionais

- Baixa latencia para renderizacao de mapa (sensacao de fluidez).
- Precisao de rotas e ETA.
- Alta disponibilidade.
- Escala global.

#### Renderizacao de Mapa com Tiling

O mapa do mundo inteiro e pre-renderizado como imagens (tiles) em multiplos niveis de zoom. Cada nivel de zoom tem tiles de tamanho fixo (256x256 pixels). Zoom 0 = 1 tile (mundo inteiro). Zoom 1 = 4 tiles. Zoom N = 4^N tiles.

Tiles sao servidas via CDN. O cliente calcula quais tiles precisa baseado na posicao e zoom level, e busca cada tile por URL (ex: /tile/{zoom}/{x}/{y}.png). Como tiles sao estaticos, caching via CDN e extremamente eficiente.

Para mapas vetoriais (modernos): em vez de imagens, tiles contem dados vetoriais (geometrias de ruas, edificios). O cliente renderiza localmente. Vantagem: menor tamanho, customizacao visual, rotacao fluida. Desvantagem: maior processamento no cliente.

#### Modelagem da Rede de Estradas como Grafo

Estradas sao modeladas como um grafo direcionado ponderado:

- Nos: intersecoes e pontos de referencia.
- Arestas: segmentos de estrada entre dois nos.
- Peso: tempo estimado de viagem (baseado em distancia, limite de velocidade, condicoes de trafego).

#### Algoritmo de Roteamento

**Dijkstra e A-star** funcionam para grafos pequenos mas nao escalam para um grafo com bilhoes de nos (todas as estradas do mundo).

**Abordagem real: routing com grafos hierarquicos**

O grafo e pre-processado em multiplas camadas de hierarquia. Estradas locais formam a camada mais baixa. Rodovias e estradas principais formam camadas mais altas. Para rotas longas, o algoritmo sobe rapidamente para camadas superiores (rodovias), viaja rapidamente e desce para camadas inferiores proximo ao destino.

Tecnicas especificas:

- **Contraction Hierarchies**: pre-processamento que cria "atalhos" no grafo. Nos menos importantes sao "contraidos" (removidos) e substituidos por arestas diretas entre seus vizinhos. Na busca, o algoritmo sobe a hierarquia a partir da origem e desce a partir do destino, encontrando-se no meio.
- **A-star com particionamento**: divide o mapa em regioes. Rotas dentro de uma regiao usam grafo detalhado; entre regioes, usam grafo simplificado.

**Condicoes de trafego em tempo real**: dados de trafego (velocidade media atual) sao sobrepostos ao grafo como pesos atualizados. Fontes: dados de GPS de usuarios, sensores de estrada, historico. O roteamento usa pesos em tempo real para ETAs precisas.

#### Geocoding

Processo de converter endereco em coordenadas. Usa uma combinacao de:

- Banco de dados de enderecos (postal databases).
- Indice de texto para busca fuzzy.
- Interpolacao para enderecos nao exatos.

Reverse geocoding: dado lat/long, buscar o endereco mais proximo. Usa indice geoespacial (similar ao Proximity Service).

#### Arquitetura

1. **Tile Service**: CDN com tiles pre-renderizados.
2. **Routing Service**: recebe origem/destino, executa algoritmo de roteamento no grafo pre-processado, retorna rota com ETA.
3. **Geocoding Service**: converte enderecos em coordenadas e vice-versa.
4. **Traffic Service**: agrega dados de trafego em tempo real, atualiza pesos do grafo.
5. **Navigation Service**: fornece instrucoes turn-by-turn, recalcula rota se o usuario desvia.

#### Licoes-Chave

- Tiling com CDN e o que torna a renderizacao de mapas viavel em escala global.
- Contraction Hierarchies sao o estado da arte para roteamento em grafos de estradas.
- Dados de trafego em tempo real sao essenciais para ETAs precisas.
- O mapa e essencialmente um problema de grafos massivos com pre-processamento pesado.

---

### Distributed Message Queue

#### Declaracao do Problema

Projetar uma fila de mensagens distribuida como Apache Kafka, RabbitMQ ou Amazon SQS, que permite comunicacao assincrona entre produtores e consumidores.

#### Requisitos Funcionais

- Produtores publicam mensagens em topicos.
- Consumidores se inscrevem em topicos e recebem mensagens.
- Suportar entrega em ordem dentro de uma particao.
- Suportar consumer groups (paralelismo de consumo).
- Mensagens persistidas para durabilidade e replay.

#### Requisitos Nao-Funcionais

- Alta throughput (milhoes de mensagens por segundo).
- Baixa latencia de producao e consumo.
- Alta disponibilidade e durabilidade (mensagens nao devem ser perdidas).
- Escalabilidade horizontal.
- Entrega configuravel: at-least-once, at-most-once, exactly-once.

#### Conceitos Fundamentais

**Topico (Topic)**: categoria logica de mensagens. Produtores publicam em topicos; consumidores leem de topicos.

**Particao (Partition)**: cada topico e dividido em particoes. Cada particao e uma sequencia ordenada e imutavel de mensagens (append-only log). Particoes permitem paralelismo: multiplos consumidores podem ler particoes diferentes simultaneamente.

**Offset**: cada mensagem em uma particao tem um offset (posicao sequencial). Consumidores rastreiam seu offset -- permite replay (reprocessar mensagens antigas) e resumption (continuar de onde parou).

**Broker**: servidor que armazena particoes. Um cluster tem multiplos brokers. Cada particao reside em um broker lider (leader) com replicas em outros brokers (followers).

**Consumer Group**: grupo de consumidores que cooperativamente consomem um topico. Cada particao e atribuida a exatamente um consumidor no grupo. Se ha 4 particoes e 2 consumidores no grupo, cada consumidor processa 2 particoes. Se um consumidor falha, suas particoes sao redistribuidas (rebalancing).

**Replicacao**: cada particao tem um fator de replicacao (ex: 3). O lider recebe escritas e propaga para followers. Se o lider falha, um follower e promovido. ISR (In-Sync Replicas) e o conjunto de replicas que estao atualizadas. Acknowledgment configuravel: acks=0 (sem confirmacao), acks=1 (lider confirmou), acks=all (todas ISR confirmaram).

#### Arquitetura de Alto Nivel

1. **Producers**: publicam mensagens. Escolhem a particao via: round-robin (distribuicao uniforme), chave de particionamento (hash(key) % num_partitions, garante que mensagens com a mesma chave vao para a mesma particao e mantem ordem), ou particionamento customizado.

2. **Brokers**: armazenam particoes em disco. Usam append-only logs (writes sequenciais sao extremamente rapidos em disco). Mensagens sao escritas em segmentos de arquivo. Segmentos antigos sao deletados ou compactados baseado em politica de retencao (ex: 7 dias ou 1 TB).

3. **Consumers**: leem mensagens de particoes. Mantêm offset (armazenado no broker ou externamente). Podem ler de qualquer offset (permite replay).

4. **Coordination Service (Zookeeper ou equivalente)**: gerencia metadata do cluster, eleicao de lideres, membership de consumer groups, rebalancing.

#### Semanticas de Entrega

- **At-most-once**: produtor envia sem confirmacao (acks=0). Mensagem pode ser perdida. Consumidor commita offset antes de processar. Caso de uso: metricas onde perda ocasional e aceitavel.
- **At-least-once**: produtor espera confirmacao (acks=all). Se nao recebe, reenvia (pode causar duplicatas). Consumidor processa antes de commitar offset. Caso de uso: maioria dos sistemas (com idempotencia no consumidor).
- **Exactly-once**: combinacao de producao idempotente (producer ID + sequence number) com consumo transacional (processar + commitar offset atomicamente). Kafka suporta via Transactional API. Caso de uso: dados financeiros, contadores exatos.

#### Armazenamento em Disco

Mensagens sao armazenadas em disco, nao em memoria. Parece contra-intuitivo, mas escritas sequenciais em disco moderno sao extremamente rapidas (300+ MB/s). O OS usa page cache eficientemente -- dados recentes estao na memoria sem gerenciamento explicito. Tecnica zero-copy: dados sao enviados do disco ao socket de rede sem copiar para o espaco do usuario (sendfile syscall).

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Append-only log em disco | Throughput altissimo, duravel | Delecao por retencao, nao por mensagem individual |
| Particoes para paralelismo | Escala horizontal | Ordem garantida apenas dentro da particao |
| Pull-based (consumidor puxa) | Consumidor controla ritmo | Pode ter latencia se consumidor nao polla rapidamente |
| Push-based (broker empurra) | Baixa latencia | Pode sobrecarregar consumidor lento |

#### Licoes-Chave

- O modelo de append-only log com offsets e a inovacao chave do Kafka.
- Particoes sao a unidade de paralelismo e escala.
- Replicacao com ISR garante durabilidade sem sacrificar demais a performance.
- Exactly-once e possivel mas tem custo de performance.

---

### Metrics Monitoring and Alerting System

#### Declaracao do Problema

Projetar um sistema de monitoramento de metricas (como Prometheus, Datadog ou Grafana) que coleta, armazena e visualiza metricas de infraestrutura e aplicacao, com alertas automaticos.

#### Requisitos Funcionais

- Coletar metricas de diversas fontes (servidores, aplicacoes, bancos de dados, containers).
- Armazenar metricas como series temporais (time-series).
- Visualizar metricas em dashboards.
- Configurar alertas baseados em thresholds ou anomalias.
- Suportar queries agregadas (media, percentil, soma em janelas de tempo).

#### Requisitos Nao-Funcionais

- Escala: milhoes de metricas por segundo.
- Baixa latencia de ingestao (metricas devem estar disponiveis em segundos).
- Alta disponibilidade (monitoramento deve funcionar especialmente quando outras coisas falham).
- Retencao longa (meses a anos) com downsampling para dados antigos.
- Queries eficientes sobre grandes volumes de dados temporais.

#### Modelo de Dados: Time-Series

Cada metrica e uma serie temporal: uma sequencia de pares (timestamp, valor) associados a um nome e tags/labels.

Exemplo: {metrica: "cpu_usage", host: "server01", regiao: "us-east"} -> [(t1, 72%), (t2, 85%), (t3, 64%), ...]

Tags permitem filtragem e agregacao multidimensional.

#### Coleta de Metricas: Push vs Pull

**Pull model (Prometheus)**: o servidor de monitoramento periodicamente "puxa" metricas de cada alvo (scraping). Cada alvo expoe um endpoint HTTP (ex: /metrics). Vantagens: o servidor controla o ritmo de coleta; facil detectar se um alvo esta down (falha no scrape). Desvantagens: requer service discovery para saber quais alvos existem; nao funciona bem quando alvos estao atras de firewalls.

**Push model (Datadog, StatsD)**: cada agente/app envia (push) metricas para o collector. Vantagens: funciona atras de firewalls; melhor para metricas de curta duracao (batch jobs). Desvantagens: pode sobrecarregar o collector; mais dificil detectar se um agente parou de enviar.

Na pratica, muitos sistemas suportam ambos. A coleta passa por um Metrics Collector que bufferiza e encaminha.

#### Armazenamento: Time-Series Database (TSDB)

Bancos de dados tradicionais nao sao otimizados para time-series. TSDBs sao especializados:

- Otimizados para escritas append-only com timestamp.
- Compressao eficiente de series temporais (delta encoding, gorilla compression).
- Queries eficientes por range de tempo.
- Downsampling automatico (dados antigos sao agregados em intervalos maiores).

Exemplos: InfluxDB, TimescaleDB (extensao PostgreSQL), OpenTSDB, Prometheus TSDB.

Escritas sao muito mais frequentes que leituras (ratio ~10:1 ou mais). O armazenamento e organizado por metrica e range de tempo.

#### Pipeline de Ingestao

1. Agentes/apps enviam metricas ao Metrics Collector (ou o collector faz scrape).
2. Collector bufferiza e envia para Kafka (fila de mensagens como buffer).
3. Consumers leem de Kafka e escrevem no TSDB.

O Kafka serve como buffer para absorver picos de ingestao e desacoplar coleta de armazenamento.

#### Queries e Visualizacao

- Linguagem de query especializada (ex: PromQL no Prometheus).
- Suporte a funcoes de agregacao: rate(), avg_over_time(), percentile(), etc.
- Dashboards (Grafana) que executam queries periodicamente e renderizam graficos.
- Cache de queries frequentes para performance.

#### Sistema de Alertas

1. Regras de alerta sao definidas (ex: "alerte se cpu_usage > 90% por mais de 5 minutos").
2. Alert Evaluator executa regras periodicamente contra o TSDB.
3. Se uma regra dispara, o alerta e enviado para o Alert Manager.
4. Alert Manager gerencia: deduplicacao (nao alertar varias vezes pelo mesmo incidente), agrupamento (agrupar alertas relacionados), silenciamento (mute durante manutencao), roteamento (enviar para o canal correto: PagerDuty, Slack, email).

#### Downsampling e Retencao

Dados recentes sao mantidos em alta resolucao (ex: 1 ponto por segundo por 7 dias). Dados mais antigos sao agregados (downsampled): 1 ponto por minuto por 30 dias, 1 ponto por hora por 1 ano. Isso reduz dramaticamente o storage sem perder a visao geral.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Pull model | Controle centralizado, deteccao de falha | Requer service discovery |
| Push model | Funciona atras de firewalls | Pode sobrecarregar collector |
| TSDB especializado | Performance otimizada para time-series | Mais uma tecnologia para operar |
| Kafka como buffer | Absorve picos, desacopla | Adiciona latencia e complexidade |

#### Licoes-Chave

- Time-series databases sao essenciais -- nao tente usar um banco relacional generico para metricas em escala.
- O pipeline Collector -> Kafka -> TSDB e o padrao da industria.
- Downsampling e obrigatorio para retencao longa de dados.
- O sistema de alertas deve tratar deduplicacao e agrupamento para evitar "alert fatigue".

---

### Ad Click Event Aggregation

#### Declaracao do Problema

Projetar um sistema que agrega eventos de cliques em anuncios em tempo real para reportar metricas como "numero de cliques por anuncio nos ultimos M minutos", necessarios para cobranca de anunciantes e otimizacao de campanhas.

#### Requisitos Funcionais

- Agregar cliques por anuncio em janelas de tempo (1 min, 5 min, 1 hora).
- Retornar contagens de cliques para queries com filtros (por anuncio, por campanha, por regiao).
- Suportar data reconciliation (comparacao entre dados real-time e batch para garantir corretude).

#### Requisitos Nao-Funcionais

- Exatidao: dados financeiros -- cliques determinam cobranca. Erros sao inaceitaveis.
- Latencia de agregacao: minutos (near real-time), nao segundos.
- Alta throughput: bilhoes de cliques por dia.
- Idempotencia e exactly-once: um clique deve ser contado exatamente uma vez.
- Fault tolerance: nenhum clique pode ser perdido.

#### Pipeline de Dados

Evento de clique raw: {ad_id, user_id, timestamp, ip, country, device, ...}

1. **Ingestion**: cliques sao enviados ao Kafka por ad servers. Kafka garante durabilidade e replay.

2. **Aggregation Service**: consumers leem de Kafka e agregam em janelas de tempo. Usa uma abordagem Map-Reduce:
   - **Map**: cada evento e mapeado por chave de agregacao (ex: ad_id).
   - **Reduce**: eventos com mesma chave sao somados dentro de cada janela de tempo.

   Na pratica, usa-se um framework de stream processing como Apache Flink, Spark Streaming ou Kafka Streams. Esses frameworks gerenciam state, checkpointing e exactly-once.

3. **Aggregated Results Database**: resultados agregados (ad_id, janela_tempo, contagem) sao escritos em banco. Pode ser um banco OLAP (ClickHouse, Druid) otimizado para queries analiticas.

4. **Query Service**: recebe queries dos dashboards e retorna dados agregados.

#### Exactly-Once Processing

Garantir que cada clique e contado exatamente uma vez e desafiador em sistemas distribuidos:

- **Deduplicacao na ingestao**: cada clique tem um ID unico. Se o produtor reenvia (por falha de rede), o Kafka deduplica via producer idempotency (producer ID + sequence number).
- **Exactly-once no stream processing**: frameworks como Flink usam checkpointing com barreiras. Se um worker falha, o estado e restaurado do ultimo checkpoint e as mensagens sao reprocessadas exatamente de onde pararam.
- **Transacoes atomicas**: processar mensagem + commitar offset + escrever resultado -- tudo atomicamente.

#### Data Reconciliation

Mesmo com exactly-once, bugs e falhas podem causar discrepancias. Solucao: rodar um batch job periodico (ex: diario) que reprocessa os dados raw do Kafka (ou de um data lake) e compara os resultados batch com os resultados real-time. Diferencas sao investigadas e corrigidas. Essa abordagem e chamada de Lambda Architecture (real-time + batch para verificacao cruzada).

#### Watermark e Late Events

Eventos podem chegar com atraso (late events) -- ex: um clique que ocorreu as 10:00 mas so chegou no servidor as 10:05 devido a latencia de rede. Como lidar:

- **Watermark**: marca de tempo que indica "todos os eventos ate este timestamp provavelmente ja chegaram". Eventos apos o watermark sao considerados atrasados.
- **Janelas com grace period**: a janela de 10:00-10:01 fica aberta ate 10:06 (grace period de 5 min) para aceitar eventos atrasados.
- **Late event handling**: eventos que chegam apos o grace period sao descartados ou processados em um fluxo separado de reconciliacao.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Stream processing (real-time) | Resultados em minutos | Complexidade de exactly-once |
| Batch reconciliation | Garantia de corretude | Resultados demoram horas |
| Lambda (ambos) | Melhor dos dois mundos | Complexidade de manter dois pipelines |
| Kafka como source of truth | Replay de eventos para reprocessamento | Custo de storage para retencao longa |

#### Licoes-Chave

- Para dados financeiros, exactly-once nao e opcional -- e obrigatorio.
- Data reconciliation (batch vs real-time) e essencial como safety net.
- Watermarks e late event handling sao fundamentais em stream processing.
- O MapReduce pattern (map por chave, reduce por agregacao) e a base de qualquer sistema de agregacao.

---

### Hotel Reservation System

#### Declaracao do Problema

Projetar um sistema de reservas de hotel como Booking.com ou Airbnb, onde usuarios podem buscar quartos disponiveis e fazer reservas.

#### Requisitos Funcionais

- Buscar hoteis por localizacao, datas e criterios.
- Visualizar detalhes e disponibilidade de quartos.
- Fazer uma reserva (selecionar quarto, datas, pagar).
- Cancelar uma reserva.

#### Requisitos Nao-Funcionais

- Consistencia: nao permitir overbooking (duas pessoas reservando o mesmo quarto para as mesmas datas).
- Alta disponibilidade (especialmente durante periodos de pico).
- Latencia baixa para buscas.
- Suportar picos de trafego (black friday de viagens, feriados).

#### O Problema Central: Concorrencia e Race Conditions

O maior desafio nao e a busca -- e a reserva. Se dois usuarios tentam reservar o mesmo quarto ao mesmo tempo, o sistema deve garantir que apenas um tenha sucesso.

**Cenario de race condition**: Usuario A e B veem o mesmo quarto disponivel. Ambos clicam "Reservar" simultaneamente. Sem controle, ambos teriam sucesso, causando overbooking.

**Solucoes:**

**1. Pessimistic Locking (SELECT FOR UPDATE)**

Ao iniciar a reserva, o sistema faz SELECT FOR UPDATE na linha do inventario. Isso bloqueia a linha no banco ate o commit da transacao. Outro usuario tentando reservar o mesmo quarto ficara bloqueado ate o primeiro terminar.

Vantagens: impede conflitos completamente. Desvantagens: pode causar deadlocks; limita throughput (requisicoes sao serializadas); lock pode durar demais se a transacao e lenta.

**2. Optimistic Locking (versioning)**

Cada registro de inventario tem um campo "version". Ao reservar:
1. Leia o registro com version atual (ex: version = 5).
2. Processe a logica de negocio.
3. UPDATE ... SET available = available - 1, version = version + 1 WHERE id = X AND version = 5.
4. Se o UPDATE afetou 0 linhas, significa que outro usuario modificou o registro. Retry.

Vantagens: sem locks explicitos, melhor throughput. Desvantagens: retries podem ser frequentes em cenarios de alta contencao (muitos usuarios tentando o mesmo quarto); nao e ideal para hotspots.

**3. Database Constraints**

Usar UNIQUE constraint no banco: (room_id, date) deve ser unico na tabela de reservas. Se dois inserts ocorrem para o mesmo par, o segundo falha com constraint violation. Simples e eficiente.

#### Idempotencia

O usuario pode clicar "Reservar" varias vezes (por impaciencia ou bug do frontend). Cada clique gera uma requisicao. Sem idempotencia, multiplas reservas seriam criadas.

Solucao: o cliente gera um idempotency_key (UUID) antes de enviar a requisicao. O servidor verifica se o key ja foi processado. Se sim, retorna o resultado anterior. Se nao, processa e armazena o resultado associado ao key. Esse key pode ser uma UNIQUE constraint na tabela de reservas.

#### Modelo de Dados

Tabelas principais:
- hotels: id, nome, endereco, descricao.
- rooms: id, hotel_id, tipo, preco, amenidades.
- room_inventory: room_id, date, total_inventory, total_reserved.
- reservations: id, user_id, room_id, check_in, check_out, status, idempotency_key.

A verificacao de disponibilidade: SELECT de room_inventory WHERE total_inventory - total_reserved > 0 para cada data do periodo solicitado.

#### Arquitetura de Alto Nivel

1. **Hotel Service**: CRUD de hoteis e quartos. Write QPS baixo.
2. **Inventory Service**: gerencia disponibilidade. Usa cache para leituras, banco com locking para escritas.
3. **Reservation Service**: processa reservas. Valida disponibilidade, cria reserva atomicamente.
4. **Payment Service**: integra com gateway de pagamento.
5. **Search Service**: busca por localizacao, datas, filtros. Usa Elasticsearch. Dados sincronizados do banco principal via CDC (Change Data Capture) ou messaging.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Pessimistic locking | Garantia absoluta contra overbooking | Baixo throughput, risco de deadlock |
| Optimistic locking | Melhor throughput | Retries em alta contencao |
| DB constraint | Simples, confiavel | Menos flexivel para logica complexa |

#### Licoes-Chave

- Concorrencia e o problema central de sistemas de reserva -- escolha o mecanismo de locking adequado ao nivel de contencao.
- Idempotencia e obrigatoria em qualquer operacao financeira ou de estado.
- A separacao entre servico de busca (Elasticsearch, eventual consistency) e servico de reserva (banco relacional, strong consistency) e o padrao arquitetural.

---

### Distributed Email Service

#### Declaracao do Problema

Projetar um servico de e-mail distribuido como Gmail, Outlook ou Yahoo Mail, suportando envio, recebimento, armazenamento e busca de e-mails.

#### Requisitos Funcionais

- Enviar e receber e-mails.
- Listar e-mails por pasta (inbox, sent, drafts, spam, etc).
- Ler um e-mail especifico.
- Buscar e-mails por palavras-chave.
- Suportar anexos.

#### Requisitos Nao-Funcionais

- Alta disponibilidade (e-mail e infraestrutura critica).
- Alta durabilidade (e-mails nao podem ser perdidos).
- Escala: bilhoes de e-mails por dia.
- Busca eficiente em caixas de entrada com milhares de e-mails.
- Consistencia: um e-mail marcado como lido deve refletir em todos os dispositivos.

#### Protocolos

- **SMTP (Simple Mail Transfer Protocol)**: protocolo para envio de e-mails entre servidores. Porta 25 (ou 587 com TLS). Funciona como um sistema store-and-forward: o servidor de envio se conecta ao servidor MX (Mail Exchanger) do destinatario e entrega o e-mail.
- **IMAP (Internet Message Access Protocol)**: protocolo para leitura de e-mails. Mantêm e-mails no servidor. Permite acesso de multiplos dispositivos com sincronizacao de estado (lido, pastas, flags).
- **POP3 (Post Office Protocol)**: protocolo mais antigo que baixa e-mails para o dispositivo local. Menos usado hoje.

#### Fluxo de Envio

1. Cliente (web/mobile) envia e-mail via API (HTTP) ao Outgoing Email Service.
2. O servico valida, aplica virus scan, spam check no conteudo de saida.
3. Armazena o e-mail na pasta "Sent" do remetente.
4. Enfileira o e-mail para envio via SMTP.
5. SMTP sender se conecta ao servidor MX do dominio do destinatario e entrega.
6. Se falha, retry com backoff (SMTP define codigos de erro e mecanismo de retry).

#### Fluxo de Recebimento

1. E-mail externo chega via SMTP no Incoming Email Service.
2. Validacao: SPF, DKIM, DMARC (autenticacao do remetente).
3. Virus scan e spam filtering (ML-based: analise de conteudo, headers, reputacao do remetente).
4. E-mail e armazenado na caixa de entrada do destinatario.
5. Se o usuario esta online (WebSocket/push), notificacao em tempo real.
6. Se offline, push notification via servico de notificacoes.

#### Armazenamento

E-mails nao sao bem servidos por bancos relacionais tradicionais (muito grandes, schema variavel com headers, anexos variados). Opcoes:

- **Distributed key-value/column store**: Cassandra, HBase, Bigtable. Chave: (user_id, folder, email_id). Otimizado para leituras sequenciais por usuario/pasta.
- **Blob storage para anexos**: anexos grandes sao armazenados separadamente em S3/GCS. O e-mail contem apenas referencia (URL/pointer) ao anexo.
- **Metadata store**: metadata de e-mails (subject, from, to, date, flags) em banco otimizado para queries e ordenacao.

#### Busca (Search Indexing)

Busca full-text em e-mails e essencial. Usa-se Elasticsearch ou um indice invertido customizado. O indice e construido por usuario (cada usuario tem seu proprio indice ou particao). Desafios:

- Indexar novos e-mails em near real-time.
- Manter o indice atualizado quando e-mails sao deletados ou movidos.
- Suportar queries complexas (from:, to:, subject:, has:attachment, date range).

Pipeline: quando um e-mail e armazenado, um evento e enviado ao Kafka. Um consumer atualiza o indice Elasticsearch.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Column store para e-mails | Escala horizontal, leituras sequenciais eficientes | Queries complexas sao limitadas |
| Elasticsearch para busca | Full-text poderoso, queries complexas | Indice adicional para manter sincronizado |
| Blob storage para anexos | Custo eficiente, escala | Latencia adicional para buscar anexos |

#### Licoes-Chave

- E-mail envolve protocolos legados (SMTP, IMAP) que devem ser suportados.
- A separacao entre metadata, corpo do e-mail e anexos permite otimizar storage e performance para cada tipo.
- Spam filtering e autenticacao (SPF/DKIM/DMARC) sao criticos para confiabilidade.
- Busca full-text e uma necessidade fundamental que requer indice especializado.

---

### S3-like Object Storage

#### Declaracao do Problema

Projetar um servico de armazenamento de objetos distribuido similar ao Amazon S3, que armazena objetos (arquivos) de tamanho arbitrario com alta durabilidade e disponibilidade.

#### Requisitos Funcionais

- Criar e deletar buckets (containers logicos de objetos).
- Upload de objetos (PUT) em buckets.
- Download de objetos (GET) por chave.
- Listagem de objetos em um bucket.
- Versionamento de objetos (opcional).

#### Requisitos Nao-Funcionais

- Durabilidade extrema: 11 noves (99.999999999%) -- dados praticamente nunca sao perdidos.
- Alta disponibilidade: 99.99%.
- Escala: trilhoes de objetos, exabytes de dados.
- Objetos de 1 KB ate varios TB.
- Consistencia: strong consistency apos escrita (read-after-write consistency).

#### Separacao de Concerns: Metadata vs Data

A arquitetura fundamental de um object store separa completamente:

1. **Metadata Store**: armazena informacoes sobre cada objeto (nome, tamanho, ACL, versao, localizacao dos dados). Hospedado em banco de dados distribuido (tipo DynamoDB ou banco relacional shardado). Requer strong consistency. Escala: bilhoes de registros.

2. **Data Store**: armazena os bytes do objeto em si. Nao e um banco de dados -- e um storage engine distribuido customizado. Cada objeto e armazenado como um ou mais data chunks em data nodes.

3. **API Service**: expoe a interface REST (PUT, GET, DELETE, LIST). Autentica requisicoes, consulta metadata, coordena leitura/escrita com data store.

#### Data Store -- Arquitetura Interna

Objetos sao armazenados em data nodes. Cada data node gerencia um disco local. Escritas:

1. Objeto e dividido em chunks (se grande) ou armazenado como um unico chunk (se pequeno).
2. Cada chunk e replicado em N data nodes (ex: N=3) em racks e availability zones diferentes.
3. Replicacao pode ser sincrona (para durabilidade maxima) ou assincrona com quorum.

Para eficiencia, objetos pequenos nao sao armazenados como arquivos individuais no filesystem (overheard de inodes seria enorme). Em vez disso, multiplos objetos pequenos sao empacotados em um unico arquivo grande (similar a como WAL funciona). Um indice mapeia (object_id -> arquivo, offset, tamanho).

**Erasure Coding vs Replicacao**:

- Replicacao (3 copias): simples, rapido para leitura, mas usa 3x o storage.
- Erasure coding (ex: Reed-Solomon 8+4): divide dados em 8 fragmentos de dados + 4 fragmentos de paridade. Tolera perda de ate 4 fragmentos. Usa apenas 1.5x o storage (vs 3x com replicacao). Desvantagem: maior uso de CPU para codificacao/decodificacao, maior latencia de leitura (precisa ler de multiplos nos).

Na pratica, S3 e similares usam replicacao para dados "quentes" (frequentemente acessados) e erasure coding para dados "frios" (arquivados).

#### Metadata Store

Esquema simplificado:
- Tabela de buckets: bucket_name, owner, created_at, ACL.
- Tabela de objetos: bucket_id, object_key, version_id, size, etag (hash), acl, data_location (ponteiro para chunks no data store), created_at.

Sharding por bucket_id + object_key para distribuicao uniforme. Indice no object_key para listagem e busca.

#### Consistencia

Para garantir read-after-write consistency:
- Apos um PUT bem-sucedido, o metadata e atualizado antes de retornar sucesso ao cliente.
- Leituras subsequentes consultam o metadata atualizado e encontram o objeto.
- Isso requer consistencia forte no metadata store.

Para o data store, a consistencia e garantida pela replicacao sincrona (quorum de escrita).

#### Garbage Collection

Quando um objeto e deletado, os dados nao sao removidos imediatamente (a delete marca o metadata como deletado). Um processo de garbage collection roda periodicamente, encontra dados orfaos (sem referencia no metadata) e libera espaco. Isso evita o custo de delecao sincrona e permite "undelete" temporario.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Replicacao 3x | Simples, baixa latencia de leitura | 3x custo de storage |
| Erasure coding | ~1.5x custo de storage | Maior latencia, CPU intensivo |
| Metadata separado de dados | Escala independentemente | Consistencia entre ambos e complexa |
| Garbage collection lazy | Delecao rapida, permite undelete | Storage nao e liberado imediatamente |

#### Licoes-Chave

- A separacao metadata/data e o principio arquitetural fundamental de qualquer object store.
- Erasure coding e a tecnologia chave que torna viavel armazenar exabytes a custo razoavel.
- Empacotamento de objetos pequenos em arquivos grandes e essencial para eficiencia de I/O e filesystem.
- 11 noves de durabilidade requer replicacao entre availability zones e data centers.

---

### Real-time Gaming Leaderboard

#### Declaracao do Problema

Projetar um leaderboard em tempo real para um jogo online, mostrando o ranking de jogadores por pontuacao, atualizando conforme jogadores ganham/perdem pontos.

#### Requisitos Funcionais

- Atualizar pontuacao de um jogador em tempo real.
- Retornar o top-K jogadores (ex: top 10, top 100).
- Retornar o rank de um jogador especifico.
- Retornar jogadores em uma faixa de ranking (ex: posicoes 50-60).

#### Requisitos Nao-Funcionais

- Atualizacoes de pontuacao refletidas no leaderboard em tempo real (< 1 segundo).
- Baixa latencia para queries (< 100ms).
- Escala: milhoes de jogadores.
- Alta disponibilidade.

#### Por Que Nao Usar um Banco Relacional?

A query "SELECT rank FROM players ORDER BY score DESC" para determinar o rank de um jogador requer full table scan e sort -- O(N log N) para cada query. Com milhoes de jogadores e milhares de queries por segundo, isso e inviavel.

#### Solucao: Redis Sorted Sets

Redis Sorted Sets (ZSET) sao a estrutura de dados perfeita para leaderboards. Cada membro tem um score associado, e o Redis mantem os membros ordenados por score internamente (usando uma skip list + hash table).

Operacoes e suas complexidades:

- ZADD player_id score: adiciona/atualiza pontuacao. O(log N).
- ZREVRANK player_id: retorna o rank do jogador (ordem decrescente). O(log N).
- ZREVRANGE start stop: retorna jogadores em uma faixa de ranking. O(log N + M), onde M e o tamanho da faixa.
- ZREVRANGEBYSCORE max min LIMIT offset count: retorna jogadores com score em uma faixa.
- ZSCORE player_id: retorna a pontuacao do jogador. O(1).

Com N = 10 milhoes de jogadores, log N = ~23 operacoes. Extremamente rapido.

#### Arquitetura

1. **Game Service**: processa logica do jogo. Quando um jogador ganha pontos, envia atualizacao ao Leaderboard Service.

2. **Leaderboard Service**: recebe atualizacoes de pontuacao e executa ZADD no Redis. Responde queries de ranking (top-K, rank individual). Stateless, pode escalar horizontalmente.

3. **Redis Cluster**: armazena o sorted set. Para um unico jogo, um unico Redis node provavelmente suporta (sorted set de 10M membros usa ~1 GB de RAM). Para escala massiva, sharding por leaderboard_id (cada jogo/temporada/modo tem seu proprio sorted set).

4. **Storage persistente**: periodicamente, o leaderboard e salvo em banco de dados para durabilidade e historico. Redis tambem tem persistencia (RDB snapshots, AOF), mas um banco externo serve como backup.

#### Cenarios Especiais

**Empate de pontuacao**: Redis ordena por score e, em caso de empate, por ordem lexicografica do membro. Para desempate por tempo (quem atingiu o score primeiro), pode-se usar um score composto: score_real * 10^13 + (MAX_TIMESTAMP - timestamp). Isso garante que, para o mesmo score_real, quem atingiu primeiro tem score composto maior.

**Leaderboards por periodo**: leaderboards semanais ou mensais usam sorted sets diferentes. No inicio do periodo, cria-se um novo sorted set. No final, arquiva-se.

**Escala alem de um unico Redis**: se o sorted set nao cabe em um unico no, pode-se usar sharding por range de score ou por hash do player_id com merge no application layer. Porem, sharding de sorted sets complica queries de ranking global. Alternativa: usar multiplos sorted sets por "shard de jogadores" e merge no query time.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Redis Sorted Set | O(log N) para tudo, extremamente rapido | Dados em memoria (custo de RAM) |
| Banco relacional | Persistente, queries flexiveis | O(N log N) para ranking, inviavel em escala |
| Score composto para desempate | Desempate deterministico | Complexidade na logica de score |

#### Licoes-Chave

- Redis Sorted Sets sao a solucao padrao da industria para leaderboards.
- A complexidade O(log N) por operacao e o que viabiliza leaderboards real-time com milhoes de jogadores.
- Para persistencia e historico, complemente o Redis com um banco de dados.
- Desempates requerem encoding inteligente do score.

---

### Payment System

#### Declaracao do Problema

Projetar um sistema de pagamentos para uma plataforma de e-commerce, suportando multiplos metodos de pagamento, processamento confiavel e reconciliacao financeira.

#### Requisitos Funcionais

- Processar pagamentos (cartao de credito, debito, carteiras digitais).
- Suportar reembolsos.
- Rastrear status de pagamentos.
- Gerar relatorios financeiros.

#### Requisitos Nao-Funcionais

- Confiabilidade absoluta: dinheiro nunca pode "sumir" (nao processar pagamento e nao entregar produto, ou entregar produto e nao cobrar).
- Consistencia: o estado do pagamento deve ser consistente em todos os componentes.
- Idempotencia: reprocessar o mesmo pagamento nao deve cobrar duas vezes.
- Auditabilidade: todo evento financeiro deve ser rastreavel.
- Disponibilidade: o sistema deve funcionar 24/7.

#### Integracao com PSP (Payment Service Provider)

O sistema nao processa pagamentos diretamente -- integra com PSPs (Stripe, PayPal, Adyen, etc) que lidam com a complexidade de redes de cartoes, compliance PCI-DSS, etc.

Fluxo tipico:

1. Usuario seleciona itens e clica "Pagar".
2. Payment Service cria um payment intent/order no PSP.
3. PSP retorna uma URL ou token para o checkout.
4. Usuario completa o pagamento na interface do PSP (ou via API se PCI-compliant).
5. PSP notifica o sistema via webhook (callback) sobre o resultado (sucesso, falha, pendente).
6. Payment Service atualiza o status do pagamento.

#### Arquitetura de Alto Nivel

1. **Payment Service**: orquestra o fluxo de pagamento. Recebe requisicao do cliente, cria registros no banco, interage com PSP, processa webhooks. Implementa retry e idempotencia.

2. **PSP Integration Layer**: abstrai a comunicacao com diferentes PSPs. Cada PSP tem uma API diferente; essa camada normaliza.

3. **Ledger (Razao Contabil)**: sistema de contabilidade de partidas dobradas (double-entry bookkeeping). Cada transacao gera no minimo dois registros: um debito e um credito. A soma de todos os debitos deve igualar a soma de todos os creditos. Exemplo: pagamento de R$100 -> debito de R$100 na conta do comprador, credito de R$100 na conta do vendedor.

4. **Wallet Service**: gerencia saldos de contas internas (se a plataforma mantiver saldos, como um marketplace).

5. **Reconciliation Service**: compara registros internos com extratos do PSP e do banco para garantir que todos os valores batem. Roda diariamente (ou mais frequentemente). Discrepancias sao investigadas.

#### Double-Entry Ledger (Contabilidade de Partidas Dobradas)

Toda transacao financeira e registrada como pelo menos duas entradas em um ledger:

| Transacao | Conta Debitada | Conta Creditada | Valor |
|---|---|---|---|
| Pagamento | Buyer Liability | PSP Receivable | R$100 |
| Settlement | PSP Receivable | Seller Revenue | R$100 |
| Reembolso | Seller Revenue | Buyer Liability | R$100 |

A regra fundamental: total de debitos = total de creditos. Sempre. Se nao bate, ha um bug.

Beneficios: rastreabilidade completa, deteccao de erros, conformidade com normas contabeis.

#### Idempotencia em Pagamentos

Absolutamente critica. Um timeout na chamada ao PSP nao significa que o pagamento falhou -- pode ter sido processado. Sem idempotencia, retry pode cobrar o usuario duas vezes.

Implementacao: cada requisicao de pagamento carrega um idempotency_key unico (gerado pelo cliente). O servidor armazena o resultado associado ao key. Em caso de retry com o mesmo key, retorna o resultado anterior sem reprocessar.

O PSP tambem suporta idempotency_key para evitar duplicatas do lado deles.

#### Reconciliacao

Tres fontes de dados devem concordar:

1. Registros internos (banco de dados do sistema).
2. Registros do PSP (extratos do Stripe, PayPal, etc).
3. Registros bancarios (extratos bancarios).

O Reconciliation Service compara os tres diariamente. Discrepancias comuns: pagamento registrado internamente mas nao no PSP (falha silenciosa), pagamento no PSP mas nao registrado internamente (webhook perdido), valores divergentes.

#### Tratamento de Falhas

- **PSP timeout**: nao assuma falha. Consulte o status via API do PSP. Se inconclusivo, marque como "pending" e reconcilie depois.
- **Webhook perdido**: PSPs geralmente reenviam webhooks. Alem disso, o sistema pode fazer polling periodico do PSP para transacoes pendentes.
- **Falha parcial**: se o pagamento foi processado mas o webhook nao chegou, o Reconciliation Service detecta a discrepancia.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| PSP externo | Sem compliance PCI, rapido de implementar | Dependencia de terceiro, taxas |
| Double-entry ledger | Rastreabilidade, auditoria | Complexidade de implementacao |
| Reconciliacao diaria | Detecta discrepancias | Problemas so sao detectados com delay |

#### Licoes-Chave

- Idempotencia e a propriedade mais importante em sistemas de pagamento.
- Double-entry ledger nao e opcional para sistemas financeiros serios.
- Reconciliacao e a rede de seguranca final -- nenhum sistema e 100% confiavel sem ela.
- Nunca assuma que um timeout significa falha em contexto financeiro.

---

### Digital Wallet

#### Declaracao do Problema

Projetar uma carteira digital (como Apple Pay, Google Pay, ou carteira interna de um marketplace) que permite usuarios armazenar saldo, fazer transferencias entre carteiras e manter historico completo de transacoes.

#### Requisitos Funcionais

- Depositar dinheiro na carteira.
- Transferir dinheiro entre carteiras.
- Verificar saldo.
- Visualizar historico de transacoes.

#### Requisitos Nao-Funcionais

- Consistencia perfeita: saldo deve refletir exatamente todas as transacoes. Dinheiro nao pode "aparecer" ou "desaparecer".
- Alta disponibilidade.
- Auditabilidade completa: toda mudanca de saldo deve ser rastreavel.
- Performance: milhares de transacoes por segundo.
- Exatidao: aritmética de ponto flutuante nao e aceitavel para dinheiro. Usar inteiros (centavos) ou tipos decimais de precisao fixa.

#### Abordagem Tradicional (e seus problemas)

Abordagem naive: tabela "wallets" com campo "balance". Transferencia = transacao que decrementa saldo de A e incrementa saldo de B.

Problemas:
- **Consistencia em banco distribuido**: se o saldo de A e B estao em shards diferentes, uma transacao ACID distribuida e necessaria (two-phase commit), que e lenta e fragil.
- **Auditabilidade**: atualizar diretamente o saldo perde o historico. Por que o saldo e R$150? Quais transacoes levaram a esse valor?

#### Event Sourcing

Em vez de armazenar o saldo atual, armazene todos os eventos que afetam o saldo. O saldo e derivado (computado) a partir da sequencia de eventos.

Eventos: "deposito de R$100", "transferencia de R$30 para usuario B", "recebimento de R$50 de usuario C", etc.

O saldo atual = soma de todos os eventos aplicados em ordem.

Vantagens:
- **Auditabilidade perfeita**: o historico completo esta nos eventos. Voce pode reconstruir o saldo de qualquer momento no tempo.
- **Imutabilidade**: eventos nunca sao alterados ou deletados. Sao append-only. Isso simplifica concorrencia e replicacao.
- **Correcao de erros**: em vez de alterar um evento, cria-se um evento compensatorio (ex: "estorno de R$30").
- **Reproductibilidade**: pode-se "replayar" todos os eventos para reconstruir o estado do zero, verificar corretude, ou migrar para um novo sistema.

Desvantagens:
- Computar o saldo a partir de milhoes de eventos e lento. Solucao: manter snapshots periodicos do saldo (ex: a cada 1000 eventos) e computar apenas a partir do ultimo snapshot.
- Mais complexo de implementar que CRUD simples.

#### CQRS (Command Query Responsibility Segregation)

Separar o modelo de escrita (commands) do modelo de leitura (queries):

- **Command side**: recebe comandos (depositar, transferir), valida, e emite eventos para o event store.
- **Query side**: mantem uma view materializada otimizada para leituras (ex: tabela "wallet_balances" com o saldo atual). Essa view e atualizada assincronamente a partir dos eventos.

Beneficio: o modelo de leitura pode ser desnormalizado e otimizado para queries especificas sem impactar o modelo de escrita. Pode ter multiplas views para diferentes necessidades (saldo atual, historico, relatorios).

Trade-off: consistencia eventual entre escrita e leitura (a view materializada pode estar poucos milissegundos atrasada).

#### Transferencia entre Carteiras

Fluxo com event sourcing:

1. Comando "transferir R$50 de A para B".
2. Validacao: A tem saldo suficiente? Transacao e valida?
3. Dois eventos sao gerados atomicamente: "debito de R$50 em A" e "credito de R$50 em B".
4. Eventos sao persistidos no event store.
5. View materializada atualiza saldos de A e B.

Para garantir atomicidade quando A e B estao em shards diferentes, pode-se usar o pattern de "saga":
1. Debita R$50 de A (evento 1).
2. Credita R$50 em B (evento 2).
3. Se o credito falha, emite evento compensatorio: "estorno de R$50 em A" (evento 3).

#### Audit Trail

O event store e, por definicao, um audit trail completo. Cada evento contém: ID do evento, timestamp, tipo de operacao, carteira afetada, valor, carteira de origem/destino, metadata adicional.

Reguladores financeiros podem auditar qualquer transacao, reconstruir saldos historicos, e verificar corretude.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Event sourcing | Auditabilidade, imutabilidade | Complexidade, snapshots necessarios |
| CQRS | Leituras otimizadas, escrita desacoplada | Consistencia eventual entre command/query |
| Saldo direto (CRUD) | Simples | Sem auditoria, problemas de concorrencia |
| Saga para transacoes distribuidas | Funciona sem 2PC | Complexidade de compensacao |

#### Licoes-Chave

- Event sourcing e o padrao gold standard para sistemas financeiros por sua auditabilidade e imutabilidade.
- CQRS complementa event sourcing permitindo views otimizadas sem comprometer o modelo de escrita.
- Snapshots periodicos sao essenciais para performance quando a lista de eventos cresce.
- Sagas com eventos compensatorios substituem transacoes distribuidas em arquiteturas de microservicos.

---

### Stock Exchange (Bolsa de Valores)

#### Declaracao do Problema

Projetar o motor de matching de ordens (order matching engine) de uma bolsa de valores, onde compradores e vendedores enviam ordens e o sistema faz o casamento (matching) em tempo real.

#### Requisitos Funcionais

- Receber ordens de compra e venda (limit orders, market orders).
- Fazer matching de ordens compativeis (comprador disposto a pagar >= preco do vendedor).
- Notificar participantes sobre execucoes (trades).
- Manter o order book (livro de ofertas) atualizado.

#### Requisitos Nao-Funcionais

- Latencia ultra-baixa: microsegundos a poucos milissegundos. Cada microsegundo importa -- participantes pagam premios por co-location (servidores fisicamente proximos ao matching engine).
- Throughput: centenas de milhares de ordens por segundo.
- Determinismo: dada a mesma sequencia de ordens, o resultado deve ser identico. Essencial para auditoria e regulamentacao.
- Disponibilidade: downtime durante horario de mercado e inaceitavel.
- Fairness: ordens devem ser processadas na ordem exata de chegada (time priority).

#### O Order Book

O order book e a estrutura central. Contem todas as ordens ativas (nao executadas) para um ativo:

- **Bid side (compra)**: ordens de compra ordenadas por preco decrescente (melhor bid = preco mais alto no topo).
- **Ask side (venda)**: ordens de venda ordenadas por preco crescente (melhor ask = preco mais baixo no topo).

Um trade ocorre quando o melhor bid >= melhor ask.

Estrutura de dados: para cada nivel de preco, uma fila FIFO de ordens (time priority). Implementado com uma combinacao de ordered map (red-black tree ou similar) para niveis de preco e linked lists para ordens no mesmo preco.

#### Matching Algorithm

**Price-Time Priority (FIFO)**:

1. Nova ordem de compra chega com preco P.
2. Verifica se P >= melhor ask. Se sim, match.
3. Executa contra as ordens de venda no melhor ask, em ordem FIFO (quem chegou primeiro e atendido primeiro).
4. Se a ordem de compra nao foi totalmente preenchida, move para o proximo nivel de preco ask.
5. Se nao ha mais matches, a ordem remanescente e inserida no bid side do order book.

Para market orders: executa ao melhor preco disponivel, percorrendo o order book ate preencher.

#### Sequencer

Para garantir determinismo e fairness, todas as ordens devem ser processadas em uma unica sequencia deterministica. O sequencer e o componente que:

1. Recebe ordens de multiplos gateways.
2. Atribui um sequence number unico e monotonicamente crescente a cada ordem.
3. Encaminha ordens ao matching engine na ordem do sequence number.

O sequencer e um ponto unico que define a "verdade" sobre a ordem dos eventos. E critico: se o sequencer falha, o mercado para. Por isso, usa-se replicacao sincrona (hot standby) com failover rapido.

#### Otimizacao de Latencia

Bolsas modernas operam em microsegundos. Tecnicas:

- **Processamento single-threaded**: o matching engine processa ordens em um unico thread. Elimina overhead de locks e context switches. Parece contra-intuitivo, mas um unico thread bem otimizado em CPU moderna processa milhoes de operacoes por segundo.
- **Memoria pre-alocada**: evita alocacao dinamica de memoria durante processamento (sem garbage collection).
- **Kernel bypass**: usa DPDK ou similar para receber pacotes de rede diretamente na aplicacao, pulando o kernel do OS.
- **Lock-free data structures**: onde multithreading e necessario (ex: I/O), usa estruturas lock-free.
- **Co-location**: servidores fisicamente no mesmo data center da bolsa para minimizar latencia de rede.
- **FPGA/ASIC**: algumas bolsas implementam o matching engine em hardware dedicado para latencia sub-microsegundo.

#### Arquitetura de Alto Nivel

1. **Gateway**: recebe ordens dos participantes via protocolo FIX (Financial Information eXchange) ou API proprietaria. Valida formato, autentica participante, faz rate limiting.

2. **Sequencer**: ordena todas as ordens em uma sequencia global.

3. **Matching Engine**: processa ordens na ordem do sequencer. Mantem o order book. Gera trades. E o componente mais critico e otimizado.

4. **Order Manager**: rastreia estado de cada ordem (new, partially filled, filled, cancelled).

5. **Market Data Publisher**: publica o order book atualizado e trades executados para todos os participantes (via multicast UDP para baixa latencia).

6. **Reporter**: gera relatorios regulatorios, confirmacoes de trade, e dados para clearing house.

#### Confiabilidade e Recovery

- O sequencer grava todas as ordens em um log duravel (evento sourcing).
- Em caso de falha, o matching engine pode ser reconstruido reprocessando o log de ordens desde o ultimo snapshot.
- Hot standby: um segundo matching engine processa as mesmas ordens em paralelo (shadow). Se o primario falha, o secundario assume sem perda de dados.

#### Trade-offs

| Decisao | Vantagem | Desvantagem |
|---|---|---|
| Single-threaded matching | Determinismo, sem locks | Limitado a um core de CPU |
| Sequencer centralizado | Ordem global garantida | SPOF (mitigado com replicacao) |
| Event sourcing para recovery | Reconstrucao completa do estado | Log cresce continuamente |
| Kernel bypass | Latencia minima | Complexidade de desenvolvimento |

#### Licoes-Chave

- O matching engine e um dos sistemas de mais baixa latencia na industria de software.
- Single-threaded processing e a abordagem preferida para determinismo e simplicidade.
- O sequencer garante fairness -- a ordem de chegada das ordens e sagrada.
- Event sourcing e o padrao natural para auditabilidade e recovery em bolsas.
- Latencia e medida em microsegundos, nao milissegundos, neste dominio.

---
---

## Principais Licoes

Apos estudar todos os capitulos de ambos os volumes, as licoes que permeiam todo o conteudo sao:

1. **Comece simples, escale incrementalmente.** Nenhum sistema nasce distribuido. Comece com o minimo viavel e adicione complexidade conforme a demanda exige. Over-engineering cedo e um dos erros mais caros em engenharia de software.

2. **Entenda os trade-offs antes de escolher.** Nao existe solucao perfeita. Cada decisao arquitetural troca uma propriedade por outra: consistencia vs disponibilidade (CAP), latencia vs durabilidade, simplicidade vs escalabilidade. A habilidade de articular trade-offs e o que diferencia um engenheiro senior.

3. **Requisitos antes de design.** Nunca desenhe sem antes entender profundamente o que o sistema precisa fazer, para quantos usuarios, com que latencia e disponibilidade. Perguntas de esclarecimento economizam horas de retrabalho.

4. **Estimativas de envelope sao poderosas.** Calculos rapidos de QPS, storage e bandwidth revelam as verdadeiras restricoes do sistema e guiam decisoes arquiteturais (ex: "com 100K QPS, precisamos de cache" ou "com 500 TB, precisamos de sharding").

5. **Cache e fila de mensagens sao os dois multiplicadores de forca.** Cache reduz latencia e carga no banco. Filas desacoplam componentes e absorvem picos. Juntos, resolvem a maioria dos problemas de escala.

6. **Idempotencia nao e opcional.** Em qualquer sistema distribuido, requisicoes podem ser duplicadas (retries por timeout). Sem idempotencia, duplicatas causam correcao de dados, cobrancas duplas, ou estados inconsistentes.

7. **Dados sao o ativo mais valioso.** Durabilidade, replicacao, backup, reconciliacao -- tudo gira em torno de nao perder dados. Em sistemas financeiros, event sourcing e double-entry ledger sao obrigatorios.

8. **Consistencia e um espectro, nao binario.** Consistencia forte, eventual, causal -- cada nivel tem seu custo e beneficio. A maioria dos sistemas pode tolerar consistencia eventual para leituras enquanto mantem consistencia forte para escritas criticas.

9. **Monitoramento e observabilidade nao sao opcionais.** Um sistema que voce nao pode observar e um sistema que voce nao pode operar. Metricas, logs, traces e alertas sao tao importantes quanto a feature em si.

10. **O design mais elegante e o mais simples que atende aos requisitos.** Complexidade desnecessaria e o maior inimigo da manutenibilidade, operabilidade e evolucao do sistema.

---

## Checklist para System Design Interviews

### Antes de Comecar (3-5 minutos)

- [ ] Faca perguntas para esclarecer requisitos funcionais.
- [ ] Defina requisitos nao-funcionais (latencia, disponibilidade, consistencia, escala).
- [ ] Identifique as restricoes (DAU, QPS, storage).
- [ ] Faca estimativas de envelope (QPS, storage, bandwidth).
- [ ] Confirme o escopo com o entrevistador.

### Design de Alto Nivel (10-15 minutos)

- [ ] Identifique as APIs principais (REST endpoints).
- [ ] Desenhe o diagrama com: clientes, load balancer, web servers, servicos, banco de dados, cache, filas.
- [ ] Percorra os fluxos principais (write path, read path).
- [ ] Identifique o modelo de dados (tabelas, esquemas, escolha de banco).
- [ ] Valide com o entrevistador antes de aprofundar.

### Deep Dive (10-25 minutos)

- [ ] Aprofunde nos componentes que o entrevistador indicar.
- [ ] Discuta trade-offs para cada decisao (nao existe resposta unica correta).
- [ ] Considere cenarios de falha: o que acontece se X cai?
- [ ] Considere escala: e se o trafego dobrar? 10x? 100x?
- [ ] Discuta consistencia: o sistema precisa de strong ou eventual?
- [ ] Considere data partitioning (sharding) e replication.
- [ ] Discuta caching strategy (o que cachear, TTL, invalidacao).
- [ ] Considere rate limiting e protecao contra abuso.

### Encerramento (3-5 minutos)

- [ ] Identifique gargalos e proponha melhorias.
- [ ] Mencione monitoramento, alertas e metricas.
- [ ] Discuta possibilidades de evolucao futura.
- [ ] Mencione single points of failure e como elimina-los.
- [ ] Resuma os trade-offs principais do design.

### Tecnologias para Referenciar

- **Cache**: Redis, Memcached.
- **Filas**: Kafka, RabbitMQ, SQS.
- **Banco relacional**: MySQL, PostgreSQL.
- **NoSQL**: Cassandra, DynamoDB, MongoDB, HBase.
- **Busca**: Elasticsearch.
- **Time-series**: InfluxDB, TimescaleDB.
- **Object storage**: S3, GCS.
- **CDN**: CloudFront, Akamai, Cloudflare.
- **Load balancer**: Nginx, HAProxy, AWS ELB/ALB.
- **Service discovery**: Zookeeper, etcd, Consul.
- **Stream processing**: Flink, Spark Streaming, Kafka Streams.
- **Monitoramento**: Prometheus, Grafana, Datadog.
- **Comunicacao real-time**: WebSocket, gRPC streaming.

### Patterns Recorrentes

- **Consistent Hashing**: para particionamento de dados e distribuicao de carga.
- **Event Sourcing**: para auditabilidade e reconstrucao de estado.
- **CQRS**: para separar modelos de leitura e escrita.
- **Fanout on Write vs Read**: para feeds e notificacoes.
- **Bloom Filter**: para verificacao rapida de existencia.
- **Write-Ahead Log (WAL)**: para durabilidade de escritas.
- **LSM-Tree**: para storage engines write-heavy (Cassandra, RocksDB).
- **Merkle Tree**: para deteccao de inconsistencias entre replicas.
- **Gossip Protocol**: para membership e deteccao de falhas.
- **Saga Pattern**: para transacoes distribuidas sem 2PC.
- **Idempotency Key**: para operacoes seguras contra retry.
- **Circuit Breaker**: para protecao contra falhas em cascata.
- **Backpressure**: para protecao contra sobrecarga de consumidores.
- **Sharding**: para escala horizontal de dados.
- **Replicacao (Leader-Follower)**: para disponibilidade e throughput de leitura.
- **Quorum (W+R>N)**: para consistencia ajustavel.
