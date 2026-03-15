# Designing Data-Intensive Applications — Martin Kleppmann

## Informacoes do Livro
- Autor: Martin Kleppmann
- Ano: 2017
- Por que este livro e o mais importante do roadmap: Este livro e considerado a referencia definitiva para engenheiros que trabalham com sistemas que precisam armazenar, processar e transmitir dados em escala. Diferentemente de livros que ensinam uma tecnologia especifica, o DDIA ensina os principios fundamentais que governam todos os sistemas de dados — desde bancos de dados relacionais ate sistemas distribuidos. Compreender esses fundamentos permite que um engenheiro tome decisoes arquiteturais informadas, avalie trade-offs entre diferentes tecnologias e entenda por que determinados sistemas se comportam da forma que se comportam. Para quem esta fazendo a transicao de frontend para fullstack/backend, este livro preenche a lacuna mais critica: o entendimento profundo de como os dados fluem, sao armazenados e recuperados nos sistemas que sustentam qualquer aplicacao moderna.

## Parte I — Fundamentos de Sistemas de Dados

A Parte I do livro estabelece os alicerces conceituais para todo o restante da obra. Kleppmann comeca definindo o que caracteriza uma aplicacao "data-intensive" (intensiva em dados) em oposicao a uma aplicacao "compute-intensive" (intensiva em computacao). A maioria das aplicacoes modernas nao e limitada pela capacidade de processamento da CPU, mas sim pela complexidade dos dados: a quantidade de dados, a complexidade dos dados e a velocidade com que os dados mudam. Essas aplicacoes tipicamente precisam de bancos de dados, caches, indices de busca, processamento de streams, processamento em batch, entre outros componentes. O desafio nao e apenas usar essas ferramentas, mas entender profundamente como elas funcionam para escolher a ferramenta certa para cada problema e combina-las de forma eficaz.

---

## Capitulo 1 — Reliable, Scalable, and Maintainable Applications

### Visao Geral

O primeiro capitulo estabelece tres principios fundamentais que devem guiar o design de qualquer sistema de dados: confiabilidade (reliability), escalabilidade (scalability) e manutenibilidade (maintainability). Kleppmann argumenta que esses tres atributos sao os mais importantes para a maioria dos sistemas de software e que muitas decisoes de design se resumem a trade-offs entre eles.

### 1.1 Reliability (Confiabilidade)

Confiabilidade significa, intuitivamente, que o sistema continua funcionando corretamente mesmo quando as coisas dao errado. Mais precisamente, o sistema deve continuar executando a funcao que o usuario espera, com o nivel de desempenho esperado, mesmo diante de falhas. Kleppmann faz uma distincao crucial entre **fault** (falha de um componente) e **failure** (falha do sistema como um todo). O objetivo nao e prevenir todas as falhas — isso e impossivel — mas sim construir sistemas **tolerantes a falhas** (fault-tolerant), que continuam operando corretamente mesmo quando componentes individuais falham.

#### 1.1.1 Hardware Faults (Falhas de Hardware)

Falhas de hardware sao a primeira categoria analisada. Discos rigidos falham, a RAM apresenta defeitos, a energia eletrica e interrompida, cabos de rede sao desconectados. Kleppmann cita dados concretos: discos rigidos tem um tempo medio ate falha (MTTF — Mean Time To Failure) de aproximadamente 10 a 50 anos. Em um cluster com 10.000 discos, isso significa que, estatisticamente, cerca de um disco falha por dia.

A abordagem tradicional para lidar com falhas de hardware e a **redundancia**: discos em RAID, fontes de alimentacao duplas, CPUs com hot-swap, geradores de backup para o datacenter. Quando um componente falha, o componente redundante assume. Para a maioria das aplicacoes, essa abordagem era suficiente, pois a probabilidade de dois componentes redundantes falharem simultaneamente e muito baixa.

Porem, com o crescimento dos volumes de dados e das demandas computacionais, mais aplicacoes passaram a usar um numero maior de maquinas, o que aumenta proporcionalmente a taxa de falhas de hardware. Alem disso, plataformas de nuvem como a AWS sao projetadas para priorizar flexibilidade e elasticidade em detrimento da confiabilidade de maquinas individuais — e bastante comum que maquinas virtuais se tornem indisponiveis sem aviso previo. Por essas razoes, ha uma tendencia crescente de usar tecnicas de tolerancia a falhas baseadas em software, alem da redundancia de hardware. Sistemas que toleram a perda de maquinas inteiras podem ser atualizados com rolling upgrades (um no de cada vez), sem downtime para o usuario.

#### 1.1.2 Software Errors (Erros de Software)

Erros de software sao mais insidiosos do que falhas de hardware porque tendem a ser **sistematicos** e **correlacionados**. Enquanto falhas de hardware sao geralmente independentes (o disco de uma maquina falhar nao causa a falha do disco de outra), bugs de software podem afetar todos os nos do sistema simultaneamente. Exemplos citados por Kleppmann incluem:

- Um bug no kernel do Linux que, em 30 de junho de 2012, causou o travamento simultaneo de muitos servidores devido a um problema com o leap second (segundo intercalar).
- Um processo que consome excessivamente um recurso compartilhado (CPU, memoria, disco, largura de banda).
- Um servico do qual o sistema depende que se torna lento, nao responsivo ou comeca a retornar respostas corrompidas.
- Falhas em cascata (cascading failures), onde a falha de um componente desencadeia falhas em outros componentes, que por sua vez causam mais falhas.

Nao ha uma solucao simples para bugs de software. Kleppmann sugere varias praticas que ajudam: pensar cuidadosamente nas suposicoes e interacoes do sistema, testes extensivos (incluindo testes unitarios, testes de integracao e testes end-to-end), isolamento de processos, permitir que processos se reiniciem apos crashes, medir e monitorar o comportamento do sistema em producao, e implementar alertas que detectem quando o sistema viola garantias esperadas (por exemplo, se o numero de mensagens recebidas difere do numero de mensagens enviadas).

#### 1.1.3 Human Errors (Erros Humanos)

Mesmo nos melhores sistemas de software, seres humanos sao a principal fonte de interrupcoes. Kleppmann cita um estudo que mostra que erros de configuracao feitos por operadores sao a causa principal de interrupcoes em servicos de internet, enquanto falhas de hardware causam apenas 10-25% das interrupcoes. Os humanos sao notoriamente nao confiaveis, mas nao podemos simplesmente remove-los do processo.

Para minimizar erros humanos, Kleppmann sugere:

- **Projetar sistemas que minimizem oportunidades para erro**: APIs bem projetadas, abstracoes que incentivam o uso correto e dificultam o uso incorreto. Porem, se as interfaces forem muito restritivas, as pessoas vao contorna-las, entao e necessario equilibrar restricao com flexibilidade.
- **Desacoplar ambientes de exploracao de ambientes de producao**: fornecer ambientes de sandbox onde as pessoas possam experimentar com seguranca usando dados reais, sem afetar usuarios.
- **Testar extensivamente em todos os niveis**: testes unitarios, testes de integracao, testes end-to-end e testes manuais. Testes automatizados sao particularmente valiosos para cobrir corner cases que raramente surgem em operacao normal.
- **Permitir recuperacao rapida de erros humanos**: facilitar o rollback rapido de mudancas de configuracao, implementar rolling releases graduais, fornecer ferramentas para recalcular dados caso a versao antiga tenha dado resultados incorretos.
- **Configurar monitoramento detalhado**: metricas de desempenho e taxas de erro (telemetria). O monitoramento pode funcionar como um sistema de alerta precoce e e inestimavel para diagnosticar problemas.
- **Implementar boas praticas de gerenciamento e treinamento**: isso e importante, mas esta alem do escopo do livro.

### 1.2 Scalability (Escalabilidade)

Escalabilidade nao e um rotulo binario — nao faz sentido dizer que um sistema "e escalavel" ou "nao e escalavel". A pergunta correta e: "Se o sistema crescer de uma forma especifica, quais sao nossas opcoes para lidar com esse crescimento?" e "Como podemos adicionar recursos computacionais para lidar com a carga adicional?"

#### 1.2.1 Describing Load (Descrevendo a Carga)

Antes de discutir escalabilidade, e necessario descrever a carga atual do sistema de forma concisa. A carga pode ser descrita com alguns numeros chamados **load parameters** (parametros de carga). A escolha dos parametros depende da arquitetura do sistema: pode ser o numero de requisicoes por segundo a um web server, a proporcao de leituras para escritas em um banco de dados, o numero de usuarios ativos simultaneos em uma sala de chat, a taxa de cache hits, entre outros.

Kleppmann usa o Twitter como estudo de caso detalhado. Em novembro de 2012, o Twitter tinha duas operacoes principais:

1. **Post tweet**: um usuario publica uma nova mensagem para seus seguidores (4.600 requisicoes/segundo em media, com picos de mais de 12.000 requisicoes/segundo).
2. **Home timeline**: um usuario visualiza tweets postados pelas pessoas que segue (300.000 requisicoes/segundo).

O desafio do Twitter nao esta no volume de tweets — 12.000 escritas por segundo e relativamente simples. O desafio e o fan-out: cada usuario segue muitas pessoas, e cada usuario e seguido por muitas pessoas. Ha duas abordagens:

**Abordagem 1 — Pull (fan-out on read)**: Quando um usuario solicita seu home timeline, o sistema busca todos os tweets de todas as pessoas que esse usuario segue, faz o merge e ordena por tempo. Isso requer um JOIN complexo no banco de dados para cada requisicao de timeline.

**Abordagem 2 — Push (fan-out on write)**: Quando um usuario posta um tweet, o sistema busca todos os seguidores desse usuario e insere o tweet na timeline cache de cada seguidor. A leitura da timeline entao se torna barata — e apenas uma consulta a um cache pre-computado.

O Twitter inicialmente usou a abordagem 1, mas migrou para a abordagem 2 porque a carga de leituras de timeline (300.000/s) era muito maior que a carga de escritas de tweets (4.600/s). Porem, a abordagem 2 tem um problema: usuarios com muitos seguidores (celebridades com mais de 30 milhoes de seguidores) tornam a escrita extremamente cara — um unico tweet pode resultar em mais de 30 milhoes de escritas em caches. O Twitter acabou adotando uma **abordagem hibrida**: tweets de usuarios comuns sao distribuidos via push (fan-out on write), enquanto tweets de celebridades sao buscados via pull (fan-out on read) e mesclados na timeline no momento da leitura.

Esse exemplo ilustra perfeitamente que a distribuicao de seguidores por usuario (possivelmente seguindo uma distribuicao power-law) e um parametro de carga critico para determinar a escalabilidade do sistema.

#### 1.2.2 Describing Performance (Descrevendo o Desempenho)

Uma vez descrita a carga, podemos investigar o que acontece quando ela aumenta. Isso pode ser visto de duas formas:

1. Se os recursos (CPU, memoria, rede) permanecem fixos e a carga aumenta, como o desempenho e afetado?
2. Se queremos manter o desempenho constante quando a carga aumenta, quantos recursos adicionais sao necessarios?

Para responder a essas perguntas, precisamos de numeros de desempenho. Em sistemas de processamento batch (como Hadoop), a metrica principal e o **throughput** (vazao): o numero de registros processados por segundo, ou o tempo total para processar um dataset de tamanho fixo. Em sistemas online, a metrica principal e o **response time** (tempo de resposta): o tempo entre o cliente enviar uma requisicao e receber a resposta.

**Latencia vs. Response Time**: Kleppmann faz uma distincao importante. Latencia e a duracao em que uma requisicao esta aguardando para ser tratada (esta latente, aguardando servico). Response time e o que o cliente observa: inclui o tempo de processamento da requisicao, atrasos de rede, atrasos de filas, entre outros. Sao conceitos relacionados mas distintos.

#### 1.2.3 Percentis de Latencia

O tempo de resposta nao e um numero fixo — e uma **distribuicao de valores**. A mesma requisicao pode levar tempos diferentes em ocasioes diferentes. Kleppmann argumenta que nao devemos pensar em termos de media (mean), mas sim de **percentis**.

- **Mediana (p50)**: se voce ordenar todos os tempos de resposta, a mediana e o valor no meio. Metade das requisicoes e mais rapida e metade e mais lenta. Isso indica o tempo de resposta tipico.
- **Percentis altos (tail latencies)**: para entender o pior caso, olhamos os percentis superiores. O **p95** (percentil 95) significa que 95% das requisicoes sao mais rapidas que esse valor. O **p99** significa que 99% sao mais rapidas. O **p999** (percentil 99.9) significa que apenas 1 em cada 1.000 requisicoes e mais lenta que esse valor.

Tail latencies (latencias de cauda) sao especialmente importantes porque frequentemente afetam os usuarios mais valiosos. Na Amazon, por exemplo, os clientes que experimentam os tempos de resposta mais lentos frequentemente sao aqueles com mais dados em suas contas — porque fizeram muitas compras — e sao, portanto, os clientes mais valiosos. A Amazon observou que um aumento de 100 milissegundos no tempo de resposta resulta em uma queda de 1% nas vendas. Um atraso de 1 segundo resulta em uma reducao de 16% na satisfacao do cliente.

**SLOs e SLAs**: Service Level Objectives (SLOs) e Service Level Agreements (SLAs) sao definidos usando percentis. Por exemplo, um SLO pode exigir que a mediana do tempo de resposta seja inferior a 200ms e que o p99 seja inferior a 1 segundo. Um SLA e um contrato que estipula consequencias (como reembolso) caso o servico nao atinja os objetivos definidos. Um SLA tipico pode estipular que o servico sera considerado "up" se tiver mediana inferior a 200ms e p99 inferior a 1s, e que o cliente tem direito a reembolso se o uptime for inferior a 99.9%.

**Head-of-Line Blocking**: Mesmo que apenas uma pequena porcentagem de requisicoes seja lenta no servidor, se o cliente faz varias requisicoes simultaneas, a probabilidade de que pelo menos uma delas seja lenta aumenta significativamente. Isso e chamado de tail latency amplification. Alem disso, em sistemas onde as requisicoes sao processadas sequencialmente (por exemplo, um numero limitado de threads de processamento), uma requisicao lenta pode bloquear o processamento de requisicoes subsequentes que seriam rapidas — isso e o **head-of-line blocking**. Por isso, e importante medir os tempos de resposta do lado do cliente, nao do lado do servidor.

#### 1.2.4 Approaches for Coping with Load (Abordagens para Lidar com Carga)

**Scaling up (vertical scaling)**: mover para uma maquina mais poderosa. Simplicidade, mas tem limites fisicos e custo exponencial.

**Scaling out (horizontal scaling)**: distribuir a carga entre muitas maquinas menores. Tambem chamado de shared-nothing architecture. Mais complexo de implementar, mas sem limites teoricos de escalabilidade.

Na pratica, bons sistemas usam uma combinacao pragmatica de ambas as abordagens. Por exemplo, usar algumas maquinas poderosas e mais simples e barato do que usar um grande numero de maquinas pequenas.

**Elastic vs. Manual scaling**: Alguns sistemas sao **elasticos** — adicionam recursos automaticamente quando detectam aumento de carga. Outros requerem que um humano analise a capacidade e adicione maquinas manualmente. Sistemas elasticos sao uteis quando a carga e altamente imprevisivel, mas sistemas com escalonamento manual sao mais simples e podem ter menos surpresas operacionais.

Kleppmann observa que, ate recentemente (2017), a sabedoria convencional era manter bancos de dados em um unico no (scaling up) e distribuir apenas a camada de aplicacao stateless. Porem, ferramentas e abstracoes para sistemas distribuidos estavam melhorando, tornando o scaling out de dados mais viavel.

Um ponto crucial: **nao existe uma arquitetura generica e one-size-fits-all para escalabilidade**. Um sistema que lida com 100.000 requisicoes/segundo de 1 KB cada e muito diferente de um que lida com 3 requisicoes/minuto de 2 GB cada, mesmo que ambos tenham o mesmo throughput total. A arquitetura deve ser projetada para os parametros de carga especificos do sistema.

### 1.3 Maintainability (Manutenibilidade)

A maior parte do custo de software nao esta no desenvolvimento inicial, mas na manutencao continua: corrigir bugs, manter o sistema operacional, investigar falhas, adaptar a novas plataformas, modificar para novos casos de uso, pagar divida tecnica e adicionar novos recursos. Kleppmann identifica tres principios de design que minimizam a dor da manutencao:

#### 1.3.1 Operability (Operabilidade)

Facilitar a vida da equipe de operacoes. Boas praticas incluem: fornecer visibilidade sobre o estado interno do sistema (monitoramento), suportar automacao e integracao com ferramentas padrao, evitar dependencia de maquinas individuais (permitir que maquinas sejam retiradas de servico para manutencao), boa documentacao, bons defaults com possibilidade de override por administradores, comportamento previsivel e auto-recuperacao quando possivel.

#### 1.3.2 Simplicity (Simplicidade)

Conforme os sistemas crescem, tornam-se mais complexos. A complexidade retarda o desenvolvimento, aumenta o custo de manutencao e aumenta a probabilidade de bugs. Kleppmann distingue complexidade **essencial** (inerente ao problema que o software resolve) de complexidade **acidental** (resultado da implementacao). A principal ferramenta para remover complexidade acidental e a **abstracao**. Uma boa abstracao esconde detalhes de implementacao atras de uma fachada limpa e simples. Exemplos: linguagens de alto nivel sao abstracoes sobre codigo de maquina; SQL e uma abstracao sobre estruturas de dados complexas em disco.

#### 1.3.3 Evolvability (Evolucionabilidade)

Tambem chamada de extensibility, modifiability ou plasticity. Os requisitos de um sistema mudam constantemente: novas funcionalidades, mudancas regulatorias, crescimento inesperado, novas plataformas. A facilidade com que um sistema pode ser modificado para atender a novos requisitos e sua evolucionabilidade. Sistemas simples sao geralmente mais faceis de modificar. Padroes como Agile fornecem frameworks para adaptacao a mudancas, mas Kleppmann foca nas propriedades do sistema em si que facilitam essa adaptacao.

### Conclusoes do Capitulo 1

O capitulo 1 nao oferece respostas definitivas, mas estabelece o vocabulario e os frameworks conceituais para pensar sobre sistemas de dados. Confiabilidade, escalabilidade e manutenibilidade sao objetivos que frequentemente entram em conflito, e o trabalho do engenheiro e encontrar o equilibrio correto para cada situacao.

---

## Capitulo 2 — Data Models and Query Languages

### Visao Geral

O capitulo 2 explora uma das questoes mais fundamentais do design de software: como representamos dados? Modelos de dados afetam nao apenas como o software e escrito, mas como pensamos sobre o problema que estamos resolvendo. A maioria das aplicacoes e construida empilhando camadas de modelos de dados: os desenvolvedores modelam o mundo real em objetos/estruturas de dados, que sao expressos em termos de um modelo de dados generico (tabelas, documentos, grafos), que sao representados em bytes na memoria/disco/rede, e que, por sua vez, sao representados em correntes eletricas, pulsos de luz, campos magneticos.

### 2.1 Relational Model vs. Document Model

#### 2.1.1 Historia e Evolucao

O **modelo relacional** foi proposto por Edgar Codd em 1970 e implementado comercialmente no inicio dos anos 1980. Dados sao organizados em **relacoes** (tabelas), onde cada relacao e uma colecao desordenada de **tuplas** (linhas). O modelo relacional foi originalmente projetado para business data processing: processamento de transacoes (vendas, operacoes bancarias, reservas de voos) e processamento em batch (relatorios, folha de pagamento).

O modelo relacional enfrentou concorrentes ao longo das decadas: bancos de dados de rede (modelo CODASYL) e bancos de dados hierarquicos nos anos 1970 e 1980, bancos de dados orientados a objetos nos anos 1980 e 1990, bancos de dados XML nos anos 2000. Nenhum conseguiu destrona-lo.

O movimento **NoSQL** surgiu por volta de 2010, impulsionado por varios fatores: necessidade de maior escalabilidade (datasets muito grandes ou throughput de escrita muito alto), preferencia por software open-source em detrimento de produtos comerciais, frustacao com restricoes do modelo relacional para queries especializadas, e desejo por modelos de dados mais dinamicos e expressivos. Kleppmann observa que o nome "NoSQL" foi apenas uma hashtag cativante para um meetup no Twitter, sem significado profundo, e que retroativamente foi reinterpretado como "Not Only SQL".

#### 2.1.2 The Object-Relational Mismatch

A maioria do desenvolvimento de aplicacoes modernas e feito em linguagens de programacao orientadas a objetos. Quando dados sao armazenados em tabelas relacionais, e necessaria uma camada de traducao entre os objetos da aplicacao e o modelo de tabelas/linhas/colunas do banco de dados. Essa desconexao e conhecida como **impedance mismatch** (um termo emprestado da engenharia eletrica).

Frameworks ORM (Object-Relational Mapping), como ActiveRecord e Hibernate, reduzem o boilerplate dessa traducao, mas nao eliminam completamente as diferencas entre os modelos.

Kleppmann usa o exemplo de um curriculo (resume/CV) para ilustrar a diferenca. Um curriculo tem uma estrutura que se mapeia naturalmente a um documento JSON: um usuario tem um nome, informacoes de contato, e listas de posicoes profissionais, formacao academica e informacoes de contato. Em um banco relacional, isso requer multiplas tabelas (users, positions, education, contact_info) com foreign keys. Em um banco de documentos, o curriculo inteiro pode ser armazenado como um unico documento JSON, o que e mais natural para esse caso de uso.

O modelo de documento tem vantagens para esse tipo de dado: **data locality** (todos os dados relevantes estao em um unico lugar, sem necessidade de multiplos JOINs) e proximidade com as estruturas de dados usadas pela aplicacao.

#### 2.1.3 Many-to-One and Many-to-Many Relationships

Um curriculo pode referenciar regiao geografica ou industria. Voce poderia armazenar esses valores como texto livre ("Greater Seattle Area") ou como um ID padronizado que referencia uma lista. Usar IDs padronizados tem vantagens: consistencia de estilo, evitar ambiguidade, facilidade de atualizacao (se a regiao muda de nome, atualiza-se em um unico lugar), suporte a internacionalizacao e melhor busca.

Armazenar IDs em vez de texto e a essencia da **normalizacao** em bancos de dados relacionais. Porem, normalizar dados requer relacoes **many-to-one** (muitos curriculos referenciam a mesma regiao), e relacoes many-to-one nao se encaixam bem no modelo de documentos. Em um banco relacional, e natural juntar tabelas via JOINs. Em um banco de documentos, o suporte a JOINs e frequentemente fraco ou inexistente, forcando a aplicacao a fazer multiplas queries e juntar os dados no codigo da aplicacao (application-level join).

A medida que funcionalidades sao adicionadas (por exemplo, recomendacoes entre usuarios, ou mencoes a organizacoes/escolas como entidades de primeira classe), surgem relacoes **many-to-many**, que se tornam ainda mais problematicas no modelo de documentos.

Kleppmann traca um paralelo historico fascinante: o debate entre modelos hierarquicos e relacionais nos anos 1970 e surpreendentemente similar ao debate entre modelos de documentos e relacionais hoje. O modelo hierarquico do IMS (Information Management System da IBM, de 1968) representava dados como arvores aninhadas — muito similar ao modelo de documentos JSON. O modelo hierarquico tinha o mesmo problema com relacoes many-to-many, e duas solucoes surgiram: o modelo relacional (que venceu) e o modelo de rede (CODASYL).

No modelo de rede (CODASYL), um registro podia ter multiplos pais, permitindo relacoes many-to-one e many-to-many. Porem, acessar um registro requeria seguir um caminho (access path) atraves dessas ligacoes, tornando as queries extremamente complexas e o codigo fragil.

O modelo relacional, em contraste, organiza todos os dados em tabelas planas e deixa o query optimizer decidir automaticamente como executar a query — qual indice usar, em que ordem fazer JOINs, etc. Isso e uma grande vantagem: o programador declara o que quer, nao como obter.

#### 2.1.4 Document Model: Schema Flexibility and Data Locality

O modelo de documentos oferece **schema flexibility**. Na verdade, Kleppmann argumenta que o termo "schemaless" e enganoso. Os dados tem uma estrutura implicita — a aplicacao assume que os dados tem certos campos de certos tipos. A distincao real e entre:

- **Schema-on-read** (modelo de documentos): a estrutura dos dados e implicita e interpretada apenas quando os dados sao lidos. E analogo a tipagem dinamica em linguagens de programacao.
- **Schema-on-write** (modelo relacional): o schema e explicito e o banco de dados garante que todos os dados escritos estejam em conformidade. E analogo a tipagem estatica.

A abordagem schema-on-read e vantajosa quando:
- Ha muitos tipos diferentes de objetos e nao e pratico ter uma tabela para cada tipo.
- A estrutura dos dados e determinada por sistemas externos que voce nao controla e que podem mudar a qualquer momento.

A analogia com tipagem dinamica vs. estatica e instrutiva: nao ha uma resposta universalmente correta, e a escolha depende do contexto.

Sobre **data locality**: se a aplicacao frequentemente precisa acessar o documento inteiro, ha vantagem de desempenho em ter todos os dados em um unico bloco continuo no armazenamento (um unico documento), em vez de espalhados por multiplas tabelas que requerem multiplas leituras de disco. Porem, essa vantagem so se aplica quando se precisa de grandes partes do documento de uma vez. Se apenas uma pequena parte e acessada, o banco ainda precisa carregar o documento inteiro, o que e desperdicado. Alem disso, atualizacoes em um documento geralmente requerem a reescrita do documento inteiro (exceto para modificacoes que nao mudam o tamanho do dado codificado). Por essas razoes, e recomendado manter documentos relativamente pequenos e evitar escritas que aumentem o tamanho do documento.

Kleppmann observa que a ideia de data locality nao e exclusiva de bancos de documentos. O Google Spanner oferece data locality em um banco relacional permitindo que tabelas filhas sejam interleaved (intercaladas) dentro de tabelas pai. O Oracle permite multi-table index cluster tables. A column-family do Bigtable (usada no Cassandra e HBase) tem um conceito similar.

#### 2.1.5 Convergencia entre Modelos

Kleppmann observa que os modelos relacional e de documentos estao convergindo. Bancos relacionais como PostgreSQL (desde a versao 9.2), MySQL (desde a versao 5.7) e IBM DB2 suportam documentos JSON. Bancos de documentos como RethinkDB suportam joins, e MongoDB permite document references que funcionam de forma similar a JOINs (embora resolvidos no lado do cliente). Com essa convergencia, a escolha entre os modelos depende cada vez mais das caracteristicas especificas dos dados da aplicacao, e e possivel que um hibrido dos dois modelos seja o futuro.

### 2.2 Graph-Like Data Models

Quando os dados tem muitas relacoes many-to-many, o modelo de grafos se torna a escolha mais natural. Um grafo consiste em **vertices** (nos, entidades) e **edges** (arestas, relacionamentos). Muitos tipos de dados podem ser modelados como grafos: grafos sociais (pessoas conectadas por relacoes de amizade), a web (paginas conectadas por hyperlinks), redes de estradas e transporte, entre outros.

Kleppmann destaca que grafos nao se limitam a dados homogeneos. Um unico grafo pode conter vertices representando pessoas, locais, eventos, check-ins e comentarios, com arestas representando diferentes tipos de relacionamentos entre eles. Essa flexibilidade e uma das maiores vantagens do modelo de grafos.

#### 2.2.1 Property Graphs

No modelo de property graph (usado pelo Neo4j, Titan, InfiniteGraph), cada vertice tem um identificador unico, um conjunto de arestas de saida (outgoing edges), um conjunto de arestas de entrada (incoming edges) e uma colecao de propriedades (pares chave-valor). Cada aresta tem um identificador unico, o vertice de origem (tail), o vertice de destino (head), um label que descreve o tipo de relacionamento e uma colecao de propriedades.

Aspectos importantes do property graph:
- Qualquer vertice pode ter uma aresta para qualquer outro vertice. Nao ha schema que restrinja quais tipos de vertices podem ser conectados.
- Dado um vertice, voce pode eficientemente encontrar tanto as arestas de entrada quanto as de saida, e assim percorrer o grafo em qualquer direcao.
- Usando labels diferentes para diferentes tipos de relacionamentos, voce pode armazenar varios tipos de informacao em um unico grafo.

Kleppmann mostra como um property graph pode ser representado em um banco relacional com duas tabelas (vertices e edges), mas consultas que envolvem percorrer o grafo por um numero arbitrario de arestas sao extremamente dificeis de expressar em SQL — a profundidade do percurso precisa ser conhecida antecipadamente. Em SQL:1999, queries recursivas (WITH RECURSIVE) foram adicionadas, mas a sintaxe e desajeitada comparada a linguagens de query especificas para grafos.

#### 2.2.2 Triple Stores

O modelo de triple store e muito similar ao property graph, mas usa uma terminologia diferente. Toda informacao e armazenada na forma de **triples** (triplas) de tres partes: (sujeito, predicado, objeto). O sujeito e equivalente a um vertice. O objeto pode ser um valor primitivo (nesse caso a tripla e equivalente a uma propriedade do vertice: chave = predicado, valor = objeto) ou outro vertice (nesse caso a tripla e equivalente a uma aresta: sujeito -[predicado]-> objeto).

O modelo de triple store tem raizes na **Semantic Web** e no **Resource Description Framework (RDF)**. Embora a Semantic Web como visao grandiosa nunca tenha se materializado completamente, as ferramentas e modelos de dados desenvolvidos para ela sao uteis independentemente.

#### 2.2.3 Datalog

Kleppmann tambem discute o Datalog, uma linguagem de query ainda mais antiga que precedeu SPARQL e Cypher. O Datalog e baseado em logica formal e representa dados como **predicados** na forma predicado(sujeito, objeto). A abordagem de query e baseada em regras: voce define regras que derivam novos predicados a partir de predicados existentes, e essas regras podem referenciar umas as outras recursivamente. Embora menos conhecido, o Datalog e fundamental para a teoria de bancos de dados e implementado em sistemas como Datomic e Cascalog.

### 2.3 Query Languages

#### 2.3.1 SQL

SQL e uma linguagem **declarativa**: voce especifica o padrao dos dados que deseja (quais condicoes os resultados devem satisfazer, como os dados devem ser transformados), mas nao como obte-los. O sistema de banco de dados (especificamente, o query optimizer) decide quais indices usar, em que ordem fazer joins, e em que ordem executar as partes da query.

A vantagem da abordagem declarativa e enorme: ela esconde a complexidade da implementacao e permite que o banco de dados melhore o desempenho sem necessidade de mudancas na query. Quando um novo indice e adicionado, o query optimizer pode automaticamente tirar proveito dele. Alem disso, linguagens declarativas se prestam naturalmente a execucao paralela, porque especificam o resultado, nao o algoritmo.

Kleppmann contrasta isso com a abordagem **imperativa** do IMS e CODASYL, onde o programador precisava especificar exatamente como percorrer os dados. Ele tambem usa o exemplo de CSS (declarativo) vs. manipulacao DOM com JavaScript (imperativo) para ilustrar a diferenca no contexto web.

#### 2.3.2 MapReduce

MapReduce e um modelo de programacao para processar grandes volumes de dados em clusters de maquinas, popularizado pelo Google. Nao e nem puramente declarativo nem puramente imperativo — a logica e expressa em trechos de codigo que sao chamados repetidamente pelo framework.

O modelo consiste em duas funcoes fornecidas pelo programador: **map** (ou mapper) e **reduce** (ou reducer). A funcao map e chamada uma vez para cada documento/registro de entrada e emite zero ou mais pares chave-valor. O framework agrupa todos os pares com a mesma chave e chama a funcao reduce para cada chave com a colecao de valores correspondentes.

Kleppmann mostra um exemplo de contagem de observacoes de tubaroes por mes. A funcao map e chamada para cada observacao, emitindo o mes/ano como chave e a especie como valor. A funcao reduce recebe todas as observacoes de um determinado mes e conta quantas ha. Essas funcoes devem ser **funcoes puras** — sem efeitos colaterais, sem estado externo — o que permite que o framework as execute em qualquer ordem e reexecute em caso de falha.

Um problema do MapReduce e que escrever duas funcoes JavaScript coordenadas e frequentemente mais dificil do que escrever uma unica query SQL. MongoDB adicionou suporte ao **aggregation pipeline**, uma linguagem declarativa inspirada em SQL que e mais facil de usar que MapReduce puro.

#### 2.3.3 Cypher

Cypher e uma linguagem de query declarativa criada para o banco de grafos Neo4j. Ela permite expressar padroes em grafos de forma concisa. Por exemplo, para encontrar todas as pessoas que emigraram dos Estados Unidos para a Europa, voce pode escrever um padrao que busca um vertice "person" com uma aresta "BORN_IN" para um local dentro dos "US" e uma aresta "LIVING_IN" para um local dentro de "Europe".

A beleza do Cypher e que voce nao precisa especificar como percorrer o grafo — o query optimizer decide a estrategia de execucao.

#### 2.3.4 SPARQL

SPARQL e uma linguagem de query para triple stores usando o modelo RDF. E sintaticamente similar ao Cypher e muito antes do Cypher. SPARQL e um padrao W3C e suportado por muitos bancos de dados de grafos e triple stores.

### Conclusoes do Capitulo 2

Kleppmann conclui que diferentes modelos de dados sao adequados para diferentes tipos de aplicacoes. O modelo relacional continua sendo a escolha padrao para a maioria das aplicacoes. O modelo de documentos e adequado quando os dados tem uma estrutura de arvore (relacoes one-to-many) e quando o documento inteiro e frequentemente acessado de uma vez. O modelo de grafos e adequado quando as relacoes many-to-many sao predominantes e quando voce precisa percorrer caminhos arbitrarios entre entidades. Cada modelo vem com suas linguagens de query proprias, e a escolha do modelo de dados e uma das decisoes mais consequentes no design de uma aplicacao.

---

## Capitulo 3 — Storage and Retrieval

### Visao Geral

O capitulo 3 e provavelmente o mais tecnico da Parte I. Kleppmann mergulha nos mecanismos internos de como bancos de dados armazenam e recuperam dados. A premissa e simples: um banco de dados precisa fazer duas coisas — armazenar dados quando voce os fornece e devolve-los quando voce os solicita. Mas como ele faz isso internamente tem enorme impacto no desempenho.

Kleppmann argumenta que um desenvolvedor de aplicacoes precisa entender esses mecanismos, mesmo que nao va implementar seu proprio banco de dados, porque esse conhecimento permite escolher o banco de dados correto para sua carga de trabalho e tunar seu desempenho. A diferenca entre um storage engine otimizado para cargas transacionais (OLTP) e um otimizado para cargas analiticas (OLAP) e enorme, e entender por que requer entender as estruturas de dados subjacentes.

O capitulo comeca com o banco de dados mais simples possivel: duas funcoes bash. Uma funcao `db_set` que faz append de uma chave e valor em um arquivo texto, e uma funcao `db_get` que busca a ultima ocorrencia de uma chave no arquivo. A funcao `db_set` e muito rapida — append em arquivo e uma das operacoes mais eficientes possivel. Mas a funcao `db_get` e lenta — precisa escanear o arquivo inteiro do inicio ao fim (O(n)). Para tornar as leituras eficientes, precisamos de **indices**.

Um indice e uma estrutura de dados adicional derivada dos dados primarios. Manter indices adiciona overhead nas escritas (a cada escrita, o indice precisa ser atualizado), mas acelera as leituras. Esse e um trade-off fundamental em bancos de dados: indices bem escolhidos aceleram leituras, mas toda escrita fica mais lenta. Por isso, bancos de dados geralmente nao indexam tudo por padrao, e cabe ao desenvolvedor (ou DBA) escolher quais indices criar, baseado nos padroes de query da aplicacao.

### 3.1 Hash Indexes

A forma mais simples de indice e um **hash map em memoria** (hash table) que mapeia cada chave para o byte offset no arquivo de dados onde o valor esta armazenado. Quando voce escreve um novo par chave-valor, o valor e adicionado ao final do arquivo e o hash map e atualizado com o novo offset. Quando voce busca um valor, o hash map fornece o offset e o valor e lido diretamente do disco — apenas uma operacao de I/O de disco.

Essa e essencialmente a abordagem usada pelo **Bitcask**, o storage engine padrao do Riak. O Bitcask oferece leituras e escritas de alta performance, com a restricao de que todas as chaves devem caber na memoria RAM disponivel. Os valores nao precisam caber na memoria — podem ser lidos do disco com uma unica busca. Esse e um modelo ideal para situacoes onde o numero de chaves distintas e limitado, mas cada chave e atualizada frequentemente (por exemplo, a URL de um video com o contador de visualizacoes).

#### Compaction e Segmentos

Se apenas adicionamos dados ao final de um arquivo, o arquivo crescera indefinidamente. A solucao e dividir o log em **segmentos** de tamanho fixo. Quando um segmento atinge um certo tamanho, ele e fechado e um novo segmento e aberto para escritas. Segmentos fechados podem entao passar por **compaction**: descartar chaves duplicadas, mantendo apenas a atualizacao mais recente para cada chave. Alem disso, segmentos compactados podem ser **merged** (fundidos) em um novo segmento, ja que a compactacao geralmente reduz significativamente o tamanho. A compactacao e merge sao feitos em um thread de background enquanto o sistema continua aceitando leituras e escritas nos segmentos existentes.

Cada segmento tem seu proprio hash map em memoria. Para buscar uma chave, consultamos primeiro o hash map do segmento mais recente, depois o do segundo mais recente, e assim por diante. Como o merge mantem o numero de segmentos pequeno, essa busca nao requer muitas consultas.

#### Detalhes Praticos de Implementacao

Kleppmann lista varios detalhes praticos que fazem a diferenca entre uma ideia simples e uma implementacao robusta:

- **Formato do arquivo**: usar um formato binario que codifica primeiro o tamanho da string em bytes, seguido pela string raw, e mais eficiente que CSV.
- **Delecao de registros**: para deletar uma chave, escreva um registro especial chamado **tombstone** (lapide). Quando segmentos sao mesclados, o tombstone indica que os valores anteriores para essa chave devem ser descartados.
- **Crash recovery**: se o banco de dados e reiniciado, os hash maps em memoria sao perdidos. Podemos reconstrui-los escaneando todos os segmentos, mas isso e lento. O Bitcask acelera esse processo salvando snapshots dos hash maps em disco.
- **Registros parcialmente escritos**: se o banco de dados falha no meio de uma escrita, o registro pode estar corrompido. O Bitcask usa checksums para detectar e ignorar partes corrompidas do log.
- **Concorrencia**: como escritas sao append-only, um unico thread de escrita e suficiente. Leituras podem ser feitas concorrentemente por multiplos threads, ja que segmentos sao imutaveis apos serem escritos.

#### Por que Append-Only e Melhor que Update-in-Place?

Kleppmann lista varias razoes pelas quais logs append-only sao preferidos a abordagens que atualizam o arquivo no lugar:

- Append e escrita sequencial, que e muito mais rapida que escrita aleatoria, especialmente em discos magneticos e, em certa medida, tambem em SSDs.
- Concorrencia e crash recovery sao muito mais simples se segmentos sao imutaveis ou append-only. Nao e preciso se preocupar com o caso de um crash ocorrer enquanto um valor esta sendo parcialmente sobrescrito.
- Merge de segmentos evita a fragmentacao de arquivos que ocorre com updates in-place ao longo do tempo.

#### Limitacoes do Hash Index

- A tabela hash deve caber inteiramente na memoria. Se voce tem um numero muito grande de chaves, isso e problematico. Teoricamente, poderiamos manter o hash map em disco, mas hash maps em disco tem desempenho ruim: requerem muitas operacoes de I/O aleatorio, sao caros de expandir quando ficam cheios e requerem tratamento sofisticado de colisoes de hash.
- Range queries (consultas de intervalo) nao sao eficientes. Voce nao pode facilmente buscar todas as chaves entre "kitty00000" e "kitty99999" — precisa consultar cada chave individualmente no hash map.

### 3.2 SSTables e LSM-Trees

#### SSTables (Sorted String Tables)

Uma mudanca simples mas poderosa no formato de log de segmentos: exigir que os pares chave-valor sejam **ordenados por chave**. Esse formato e chamado **Sorted String Table** ou SSTable. Alem disso, exigimos que cada chave apareca no maximo uma vez em cada segmento (a compactacao ja garante isso).

SSTables tem varias vantagens sobre segmentos com hash index:

1. **Merge eficiente**: o merge de segmentos e simples e eficiente, mesmo quando os arquivos sao maiores que a memoria disponivel. A abordagem e similar ao merge sort: lemos os segmentos de entrada lado a lado, olhamos a primeira chave de cada segmento, copiamos a menor chave para o arquivo de saida e avancamos. Se a mesma chave aparece em multiplos segmentos, mantemos o valor do segmento mais recente.

2. **Indice esparso em memoria**: nao e mais necessario manter um indice de todas as chaves em memoria. Como as chaves sao ordenadas, basta manter um indice esparso: por exemplo, uma entrada para cada poucos kilobytes de dados. Para encontrar uma chave, consultamos o indice esparso para encontrar o offset do bloco que pode conter a chave, e entao escaneamos esse bloco. Isso reduz drasticamente o uso de memoria.

3. **Compressao de blocos**: como leituras precisam escanear blocos inteiros (entre entradas do indice esparso), esses blocos podem ser comprimidos antes de serem escritos em disco. Isso economiza espaco em disco e reduz a largura de banda de I/O necessaria.

#### Construindo SSTables a partir de Memtables

Manter chaves ordenadas em disco e possivel com merge sort, mas manter chaves ordenadas em memoria e facil: basta usar uma **arvore balanceada** (red-black tree, AVL tree ou similar). A estrutura em memoria e chamada **memtable**.

O fluxo de operacao e:

1. Escritas sao adicionadas a memtable (arvore em memoria), que mantem as chaves ordenadas.
2. Quando a memtable ultrapassa um certo tamanho (tipicamente alguns megabytes), ela e escrita em disco como um arquivo SSTable. Isso e eficiente porque a arvore ja mantem as chaves ordenadas. Enquanto a SSTable esta sendo escrita, novas escritas vao para uma nova memtable.
3. Para atender uma leitura, primeiro consulta a memtable, depois o segmento SSTable mais recente, depois o proximo mais recente, e assim por diante.
4. De tempos em tempos, um processo de background executa merge e compaction dos segmentos SSTable.

Ha um problema: se o banco de dados falha, o conteudo da memtable (que esta apenas em memoria) e perdido. Para evitar isso, toda escrita tambem e adicionada a um **write-ahead log** (WAL ou log de escrita antecipada) — um arquivo append-only nao ordenado em disco. O WAL e usado apenas para recuperacao apos falhas. Quando a memtable e escrita como SSTable em disco, o WAL correspondente pode ser descartado.

#### LSM-Trees (Log-Structured Merge-Trees)

O algoritmo descrito acima — memtable com flush para SSTables e compaction em background — e essencialmente o que e usado pelo **LevelDB** e pelo **RocksDB**, bibliotecas de storage engine projetadas para serem embutidas em outras aplicacoes. Storage engines similares sao usados no **Cassandra** e no **HBase**, ambos inspirados no Bigtable do Google (que introduziu os termos SSTable e memtable).

Originalmente, essa estrutura de indexacao foi descrita por Patrick O'Neil et al. sob o nome **Log-Structured Merge-Tree** (ou LSM-Tree). Storage engines baseados nesse principio sao chamados de **LSM storage engines**.

O **Lucene**, o motor de indexacao usado pelo Elasticsearch e pelo Apache Solr, usa uma estrutura similar para seu indice de termos (term dictionary). O indice de full-text e muito mais complexo que um indice chave-valor, mas a ideia basica e similar: dado um termo de busca, encontrar todos os documentos que contem esse termo. Isso e implementado como um mapeamento de termo para uma lista de IDs de documentos (posting list). No Lucene, esse mapeamento de termo para posting list e armazenado em arquivos SSTable-like que sao mergeados em background.

#### Otimizacoes de Performance

- **Bloom Filters**: quando uma chave nao existe no banco de dados, o LSM-tree precisa verificar a memtable e todos os segmentos SSTable ate o mais antigo antes de confirmar que a chave nao existe. Isso pode ser lento. Um **Bloom filter** e uma estrutura de dados probabilistica eficiente em memoria que pode dizer se uma chave definitivamente nao existe no conjunto (com 100% de certeza) ou se possivelmente existe (com uma pequena probabilidade de falso positivo). Isso evita leituras desnecessarias de disco para chaves inexistentes.

- **Estrategias de compaction**: as duas estrategias mais comuns sao:
  - **Size-tiered compaction** (usada pelo HBase e Cassandra como opcao): SSTables mais novas e menores sao progressivamente mescladas em SSTables mais antigas e maiores.
  - **Leveled compaction** (usada pelo LevelDB, RocksDB e Cassandra como opcao): a faixa de chaves e dividida em SSTables menores, e dados mais antigos sao movidos para "niveis" separados. A compaction acontece de forma mais incremental, usando menos espaco em disco.

### 3.3 B-Trees

A estrutura de indexacao mais amplamente usada e a **B-tree**, introduzida em 1970 por Bayer e McCreight e presente em praticamente todos os bancos de dados relacionais, alem de muitos bancos nao relacionais.

Como SSTables, B-trees mantem pares chave-valor ordenados por chave, o que permite buscas eficientes por chave e range queries. Mas a semelhanca para ai. O design fundamental e muito diferente.

#### Estrutura

B-trees dividem o banco de dados em blocos de tamanho fixo (tipicamente 4 KB), chamados **pages** (paginas), e leem ou escrevem uma pagina de cada vez. Esse design esta mais alinhado com o hardware subjacente, ja que discos tambem sao organizados em blocos de tamanho fixo.

Cada pagina pode ser identificada pelo seu endereco, permitindo que uma pagina referencie outra — similar a um ponteiro, mas em disco. Uma pagina e designada como a **raiz** da B-tree. Quando voce busca uma chave, comeca pela raiz. A pagina contem varias chaves e referencias a paginas filhas. As chaves delimitam os intervalos cobertos por cada subarvore. Por exemplo, se uma pagina contem as chaves 100, 200 e 300, a referencia entre 100 e 200 aponta para a subarvore que contem todas as chaves entre 100 e 200.

As paginas folha (leaf pages) contem os valores diretamente (ou referencias aos locais onde os valores estao armazenados).

O **branching factor** (fator de ramificacao) de uma B-tree e o numero de referencias a paginas filhas em cada pagina. Tipicamente e de algumas centenas. A profundidade da arvore e logaritmica: uma B-tree de 4 niveis com fator de ramificacao 500 pode armazenar ate 256 TB de dados (500^4 * 4KB).

#### Atualizacoes e Insercoes

Para atualizar o valor de uma chave existente: busque a pagina folha que contem a chave, modifique o valor nessa pagina e escreva a pagina de volta ao disco. A estrutura da arvore nao muda.

Para inserir uma nova chave: busque a pagina folha cujo intervalo inclui a nova chave e adicione a chave/valor. Se nao ha espaco suficiente na pagina, a pagina e **dividida** (split) em duas meias paginas, e a pagina pai e atualizada para refletir a nova divisao.

Esse algoritmo garante que a arvore permaneca **balanceada**: uma B-tree com n chaves sempre tem profundidade O(log n). A maioria dos bancos de dados cabe em uma B-tree de 3 a 4 niveis de profundidade.

#### Write-Ahead Log (WAL)

A operacao fundamental de uma B-tree e sobrescrever uma pagina em disco com novos dados. Isso e fundamentalmente diferente de LSM-trees, que nunca modificam arquivos — apenas adicionam e eventualmente criam novos arquivos via merge.

Sobrescrever paginas e perigoso: se o banco de dados falha no meio de uma operacao de split (que requer escrever multiplas paginas), a arvore pode ficar corrompida. Para tornar o banco de dados resiliente a falhas, B-trees geralmente usam um **write-ahead log (WAL)**, tambem chamado **redo log**. E um arquivo append-only onde cada modificacao e registrada antes de ser aplicada as paginas da arvore. Apos uma falha, o WAL e usado para restaurar a B-tree a um estado consistente.

#### Latches (Travas de Concorrencia)

Se multiplas threads acessam a B-tree simultaneamente, e necessario controle de concorrencia cuidadoso — caso contrario, uma thread pode ver a arvore em um estado inconsistente. Isso e tipicamente feito com **latches** (travas leves), que sao locks de curta duracao nas paginas da arvore. Isso e mais complexo em B-trees do que em LSM-trees, onde toda a compactacao acontece em background sem interferir com queries, e onde as escritas de memtable sao thread-safe por design.

#### Otimizacoes de B-Trees

Kleppmann lista varias otimizacoes que bancos de dados modernos aplicam:

- **Copy-on-write**: em vez de sobrescrever paginas e manter um WAL, algumas implementacoes (como LMDB) escrevem paginas modificadas em novos locais e criam uma nova versao da arvore. Isso e util para concorrencia e snapshots.
- **Abreviacao de chaves**: em vez de armazenar a chave completa nas paginas internas, armazena-se apenas informacao suficiente para funcionar como separador entre paginas adjacentes. Isso permite um fator de ramificacao maior e menos niveis na arvore.
- **Layout sequencial em disco**: tentar organizar as paginas folha em ordem sequencial no disco. Porem, conforme a arvore cresce, manter essa ordem se torna dificil. LSM-trees tem vantagem aqui, pois reescrevem grandes segmentos de forma sequencial durante a compactacao.
- **Ponteiros entre folhas irmas**: paginas folha podem ter ponteiros para as folhas irmas a esquerda e a direita, permitindo escanear chaves em ordem sem voltar as paginas pai.

### 3.4 B-Trees vs. LSM-Trees: Comparacao Detalhada

Kleppmann dedica uma secao consideravel a comparacao entre essas duas familias de storage engines, enfatizando que nao ha um vencedor claro — depende da carga de trabalho.

#### Write Amplification

**Write amplification** e o fenomeno onde uma unica escrita logica (um INSERT ou UPDATE) resulta em multiplas escritas fisicas no disco ao longo do tempo.

Em B-trees, uma escrita pode requerer: escrever no WAL, escrever a pagina da arvore modificada e, em caso de split, escrever paginas adicionais. Mesmo uma pequena mudanca requer a reescrita de uma pagina inteira (tipicamente 4 KB).

Em LSM-trees, uma escrita tambem pode ser amplificada: o dado e escrito na memtable (e no WAL), depois na SSTable quando a memtable e descarregada, e depois reescrito multiplas vezes durante compaction e merge.

Em cargas de trabalho com muitas escritas, a write amplification tem impacto significativo no desempenho, especialmente em SSDs, que tem um numero limitado de ciclos de escrita por bloco.

#### Vantagens de LSM-Trees

- **Throughput de escrita mais alto**: LSM-trees tipicamente sustentam throughput de escrita maior, especialmente em discos magneticos, porque escritas sequenciais (append) sao muito mais rapidas que escritas aleatorias (sobrescrita de paginas). Essa diferenca e menos pronunciada em SSDs, mas ainda significativa.
- **Melhor compressao**: LSM-trees geralmente produzem arquivos menores em disco. B-trees sofrem de **fragmentacao interna**: quando uma pagina e dividida ou uma linha nao preenche exatamente o espaco disponivel, parte do espaco da pagina e desperdicado. Como LSM-trees reescrevem SSTables periodicamente e nao sao baseadas em paginas de tamanho fixo, tem menos fragmentacao. Com leveled compaction, a compressao e ainda melhor.
- **Menor write amplification em SSDs**: em alguns benchmarks, LSM-trees apresentam menor write amplification, o que e importante para a longevidade de SSDs.

#### Vantagens de B-Trees

- **Leituras mais rapidas**: cada chave existe em exatamente um local no indice B-tree. Em LSM-trees, uma leitura pode precisar verificar multiplas SSTables em diferentes estagios de compactacao. Para cargas de trabalho dominadas por leituras, B-trees costumam ser mais rapidas.
- **Desempenho mais previsivel**: B-trees oferecem latencias mais previsiveis. LSM-trees podem ter picos de latencia durante compactacao, que compete por largura de banda de I/O com leituras e escritas normais.
- **Propriedades transacionais mais simples**: em B-trees, cada chave existe em exatamente um lugar, o que simplifica a implementacao de locks de transacao (range locks no indice).
- **Compaction nao acompanha a carga**: em cargas de trabalho com throughput de escrita muito alto, pode acontecer de a compactacao nao conseguir acompanhar a taxa de escritas. O numero de SSTables nao compactadas cresce, as leituras ficam progressivamente mais lentas (porque precisam verificar mais SSTables) e o disco pode encher. B-trees nao tem esse problema, pois cada escrita atualiza dados in-place sem acumular dados nao compactados.

### 3.5 Other Indexing Structures

Ate agora, discutimos indices key-value (chave primaria). Porem, indices **secundarios** sao extremamente comuns e fundamentais para joins eficientes. Um indice secundario mapeia um valor de coluna (que pode nao ser unico) para os registros que contem esse valor.

#### Storing Values within the Index

O valor armazenado no indice pode ser:
- **Referencia (heap file)**: o indice armazena apenas a chave e um ponteiro para a linha no **heap file** (arquivo onde as linhas sao armazenadas em ordem arbitraria). Multiplos indices secundarios podem apontar para o mesmo local no heap file. Vantagem: evita duplicacao de dados. Desvantagem: uma leitura extra para obter o registro.
- **Clustered index**: o indice armazena a linha completa diretamente. No MySQL (InnoDB), a chave primaria e sempre um clustered index e os indices secundarios referenciam a chave primaria (nao um heap file pointer). No SQL Server, voce pode especificar uma tabela como clustered.
- **Covering index (index with included columns)**: armazena algumas colunas da tabela dentro do indice. A query pode ser respondida diretamente pelo indice se necessitar apenas dessas colunas, evitando a ida ao heap file. Isso e chamado de "covering" a query.

#### Multi-Column Indexes

Um indice simples mapeia uma unica chave para valores. Para queries que filtram ou ordenam por multiplas colunas simultaneamente, existem:

- **Concatenated index**: o indice concatena multiplas colunas em uma unica chave (por exemplo, sobrenome + nome). Util para queries que filtram pela primeira coluna, ou pela primeira e segunda, mas nao apenas pela segunda.
- **Multi-dimensional indexes**: para queries que filtram em multiplas dimensoes simultaneamente. O exemplo classico e busca geografica: encontrar todos os restaurantes dentro de um retangulo definido por latitude/longitude. Uma B-tree padrao pode responder eficientemente a filtros em uma dimensao, mas nao em duas simultaneamente. Abordagens incluem: R-trees (usados pelo PostGIS, extensao geoespacial do PostgreSQL), space-filling curves que mapeiam 2D para 1D, e grid-based approaches. Multi-dimensional indexes nao se limitam a dados geograficos — podem ser usados para buscar por faixas de cor (RGB) ou por combinacoes de data e temperatura em dados meteorologicos.

#### Full-Text Search and Fuzzy Indexes

Todos os indices discutidos ate aqui assumem busca exata. Para busca full-text, o indice precisa lidar com variantes de palavras (plurais, conjugacoes), sinonimos, analise gramatical e busca fuzzy (encontrar palavras similares a uma dada distancia de edicao). O Lucene usa uma estrutura similar a uma SSTable para seu dicionario de termos. A memtable e uma finite state automaton (similar a um trie), e os arquivos SSTable usam um finite state transducer. Essa estrutura pode ser combinada com uma distancia de edicao (Levenshtein automaton) para busca fuzzy eficiente.

#### In-Memory Databases

Todos os storage engines discutidos ate agora lidam com as limitacoes do disco. Com a queda do preco da RAM e a disseminacao de configuracoes com dezenas ou centenas de gigabytes de RAM, manter datasets inteiramente em memoria se tornou viavel para muitos casos de uso.

Exemplos de bancos in-memory: **Memcached** (cache apenas, dados perdidos em reinicio), **VoltDB**, **MemSQL**, **Oracle TimesTen** (bancos relacionais in-memory com garantias de durabilidade), **RAMCloud** (key-value com durabilidade), **Redis** e **Couchbase** (fracos em durabilidade por padrao, mas com configuracoes de persistencia).

A vantagem de bancos in-memory nao e apenas evitar leituras de disco — um storage engine baseado em disco tambem pode ser rapido se tiver RAM suficiente para cache do sistema operacional. A verdadeira vantagem e que bancos in-memory podem oferecer **estruturas de dados que seriam dificeis de implementar eficientemente em disco**, como priority queues e sets. Alem disso, como nao precisam serializar dados para formato de disco, podem oferecer modelos de dados mais ricos.

Trabalhos recentes (como o anti-caching do VoltDB) exploram a direcao oposta: comecar com banco in-memory e, quando a memoria nao e suficiente, mover dados menos usados para disco, funcionando como um sistema operacional de gerenciamento de memoria virtual, mas com granularidade de registros individuais.

### 3.6 Transaction Processing vs. Analytics (OLTP vs. OLAP)

Kleppmann introduz uma distincao fundamental entre dois padroes de acesso a dados:

**OLTP (Online Transaction Processing)**: e o padrao de acesso tipico de aplicacoes web. Caracteristicas:
- Cada query le ou escreve um pequeno numero de registros (por chave ou condicao simples).
- Escritas sao aleatorias, baseadas em input do usuario.
- O dataset pode ser grande, mas cada query toca apenas uma pequena fracao dele.
- Latencia baixa e critica (usuarios esperam resposta rapida).
- O padrao de acesso e dominado por buscas por chave (seek pattern).

**OLAP (Online Analytical Processing)**: e o padrao de acesso tipico de analistas de negocios e data warehouses. Caracteristicas:
- Queries leem um numero enorme de registros, mas apenas algumas colunas de cada.
- Agregacoes (COUNT, SUM, AVG, GROUP BY) sao predominantes.
- Dados sao escritos em bulk (ETL ou streams), nao por transacoes individuais.
- O volume de dados e muito grande (terabytes a petabytes).
- Throughput (nao latencia) e a metrica principal.
- O padrao de acesso e dominado por varredura sequencial (scan pattern).

#### Data Warehousing

Empresas grandes frequentemente separam suas cargas OLTP e OLAP em sistemas diferentes. O banco de dados OLTP de producao precisa ser altamente disponivel e processar transacoes com baixa latencia. Executar queries analiticas pesadas nesse banco poderia degradar o desempenho das transacoes.

Um **data warehouse** e um banco de dados separado, otimizado para queries analiticas. Dados sao extraidos dos bancos OLTP (via ETL — Extract, Transform, Load), transformados em um schema adequado para analise e carregados no data warehouse. O data warehouse pode ser indexado e otimizado para queries analiticas sem afetar a operacao normal dos bancos OLTP.

O schema mais comum em data warehouses e o **star schema** (schema estrela), tambem chamado dimensional modeling. No centro esta a **fact table** (tabela de fatos), onde cada linha representa um evento (uma venda, um clique, um pageview). As colunas da fact table incluem atributos do evento e foreign keys para **dimension tables** (tabelas de dimensao), que contem informacoes detalhadas sobre o "quem", "o que", "onde", "quando" e "como" do evento. Por exemplo, uma fact table de vendas pode ter foreign keys para tabelas de dimensao de produto, cliente, loja, data e promotor.

Uma variacao e o **snowflake schema**, onde as dimensoes sao ainda mais normalizadas (uma dimensao referencia outras sub-dimensoes). O star schema e geralmente preferido por analistas por ser mais simples de trabalhar.

Fact tables podem ser enormes — empresas grandes podem ter trilhoes de linhas e petabytes de dados. Queries tipicas filtram por uma ou mais dimensoes e agregam valores da fact table.

### 3.7 Column-Oriented Storage

Em cargas OLAP tipicas, uma query acessa muitas linhas (potencialmente todas), mas apenas algumas colunas. Por exemplo: "qual foi a receita total por produto na primeira semana de janeiro?" Essa query precisa das colunas data, produto e receita, mas nao de dezenas de outras colunas como endereco do cliente, numero do pedido, etc.

Em bancos orientados a linhas (row-oriented), todos os valores de uma linha sao armazenados juntos. Mesmo que voce precise apenas de 3 colunas de uma tabela com 100 colunas, o banco precisa carregar todas as 100 colunas de cada linha do disco para a memoria e depois filtrar as desnecessarias. Isso e muito ineficiente.

Em **column-oriented storage** (armazenamento orientado a colunas), todos os valores de uma coluna sao armazenados juntos. Cada coluna e armazenada em um arquivo separado. Se voce precisa de apenas 3 colunas, apenas 3 arquivos sao lidos. A chave e que a n-esima entrada em cada arquivo de coluna pertence a mesma linha.

Exemplos de bancos colunares: **Vertica**, **Apache Parquet** (formato de arquivo colunar), **Apache ORC**, o C-Store academico (que deu origem ao Vertica), o Dremel do Google, o **Amazon Redshift**, e o Apache Kudu.

#### Column Compression

Armazenamento colunar permite compressao muito mais eficiente porque todos os valores em um arquivo de coluna sao do mesmo tipo e frequentemente tem um numero limitado de valores distintos (baixa cardinalidade). A tecnica de compressao mais eficaz para colunas de baixa cardinalidade e **bitmap encoding**.

Se uma coluna tem n valores distintos, cria-se n bitmaps separados, um para cada valor distinto. Cada bitmap tem um bit para cada linha na tabela: 1 se a linha tem esse valor, 0 caso contrario. Se n e grande (alta cardinalidade), os bitmaps serao esparsos (muitos zeros) e podem ser eficientemente comprimidos com **run-length encoding** (codificacao por comprimento de sequencia).

Bitmaps sao particularmente poderosos para queries com condicoes AND e OR: basta fazer operacoes bitwise AND e OR nos bitmaps, que sao extremamente eficientes em hardware moderno.

#### Sort Order in Column Storage

Em um banco colunar, a ordem das linhas nao precisa ser a ordem de insercao. O administrador pode escolher ordenar por uma ou mais colunas, similar a um indice. Isso beneficia queries que filtram pela coluna de ordenacao e melhora drasticamente a compressao: se a tabela e ordenada pela coluna de data, todos os valores de data iguais ficam juntos, e o run-length encoding e extremamente eficaz.

O Vertica usa uma abordagem sofisticada: manter os mesmos dados armazenados em **multiplas ordens de ordenacao** diferentes. E como ter multiplos indices secundarios em um banco orientado a linhas, mas sem a penalidade de pointers para heap file. Cada copia e armazenada em uma ordem de ordenacao diferente, e o query optimizer escolhe a copia mais adequada para cada query.

#### Materialized Views e Data Cubes

Quando queries analiticas frequentemente calculam as mesmas agregacoes (SUM, COUNT, etc.), pode ser util pre-calcular e armazenar esses resultados. Uma **materialized view** e uma tabela derivada cujo conteudo e o resultado de uma query. Diferentemente de uma view virtual (que e apenas uma abreviacao para uma query), a materialized view e uma copia real dos dados em disco.

Quando os dados subjacentes mudam, a materialized view precisa ser atualizada. Isso torna escritas mais caras, razao pela qual materialized views sao mais comuns em data warehouses (leitura intensiva) do que em bancos OLTP.

Um caso especial de materialized view e o **data cube** (ou OLAP cube). E uma grade de agregacoes agrupadas por diferentes dimensoes. Por exemplo, um data cube 2D poderia ter datas em um eixo e produtos no outro, com cada celula contendo a soma das vendas daquele produto naquela data. As bordas do cubo contem as agregacoes ao longo de uma dimensao inteira (total de vendas por data, total de vendas por produto). Cubos com mais dimensoes sao possiveis.

A vantagem do data cube e que certas queries se tornam extremamente rapidas (uma unica leitura). A desvantagem e que nao tem a mesma flexibilidade que queries sobre dados brutos — voce so pode responder perguntas que foram pre-definidas pelas dimensoes e metricas do cubo.

### Conclusoes do Capitulo 3

O capitulo 3 fornece uma base solida para entender por que diferentes bancos de dados tem caracteristicas de desempenho tao diferentes. A escolha entre B-trees e LSM-trees, entre armazenamento orientado a linhas e orientado a colunas, entre OLTP e OLAP, nao sao questoes de "melhor" ou "pior", mas de adequacao a carga de trabalho especifica. Entender as estruturas de dados subjacentes permite fazer escolhas informadas e prever o comportamento do sistema sob carga.

---

## Capitulo 4 — Encoding and Evolution

### Visao Geral

O capitulo 4 aborda um problema pratico e fundamental: como os dados sao codificados (serializados) para armazenamento e transmissao, e como manter compatibilidade quando o formato dos dados evolui. Em qualquer sistema nao trivial, mudancas sao inevitaveis: novos campos sao adicionados, formatos mudam, servicos sao atualizados. Kleppmann argumenta que a facilidade com que um sistema pode evoluir depende criticamente de como os dados sao codificados.

Em aplicacoes do lado do servidor, frequentemente fazemos **rolling upgrades** (ou staged rollouts): implantamos uma nova versao do codigo em alguns nos de cada vez, verificamos que tudo funciona e gradualmente atualizamos todos os nos. Isso significa que, durante o rollout, nos antigos e nos novos coexistem. Nos dispositivos dos usuarios (aplicativos moveis, por exemplo), os usuarios podem nao atualizar por muito tempo.

Isso implica que tanto dados antigos quanto novos precisam coexistir. Para isso, precisamos de:
- **Backward compatibility** (compatibilidade retroativa): codigo novo pode ler dados escritos por codigo antigo.
- **Forward compatibility** (compatibilidade prospectiva): codigo antigo pode ler dados escritos por codigo novo.

Backward compatibility e geralmente facil de garantir (o autor do codigo novo conhece o formato antigo). Forward compatibility e mais sutil: requer que codigo antigo ignore graciosamente campos ou valores adicionados por codigo novo.

### 4.1 Formats for Encoding Data

Programas trabalham com dados em pelo menos duas representacoes:

1. **Em memoria**: dados em objetos, structs, listas, hash maps, arvores, etc. Otimizados para acesso e manipulacao eficientes pela CPU (frequentemente usando ponteiros).
2. **Para escrita em arquivo ou envio pela rede**: dados precisam ser codificados como uma sequencia de bytes. Como ponteiros nao fazem sentido fora do espaco de enderecamento do processo, essa representacao e muito diferente.

A traducao de representacao em memoria para bytes e chamada **encoding** (tambem serialization ou marshalling). O inverso e chamado **decoding** (deserialization, unmarshalling, parsing).

#### 4.1.1 Language-Specific Formats

Muitas linguagens possuem mecanismos built-in de serializacao: `java.io.Serializable` em Java, `pickle` em Python, `Marshal` em Ruby. Kleppmann lista varias razoes pelas quais esses formatos sao geralmente uma ma escolha:

- **Acoplamento a linguagem**: dados codificados em formato Java so podem ser lidos por Java. Isso impede integracao com outros sistemas e linguagens.
- **Seguranca**: para decodificar, o processo precisa instanciar objetos arbitrarios, o que pode ser explorado para execucao remota de codigo.
- **Versionamento negligenciado**: esses formatos frequentemente nao suportam evolucao de schema (forward/backward compatibility).
- **Eficiencia**: Java Serializable e notoriamente ruim em performance e tamanho de encoding.

#### 4.1.2 JSON, XML e Binary Variants

JSON, XML e CSV sao formatos textuais amplamente usados e suportados por praticamente todas as linguagens. Porem, possuem problemas sutis:

- **Ambiguidade em numeros**: XML e CSV nao distinguem entre strings e numeros (exceto por schema externo). JSON distingue strings de numeros, mas nao distingue inteiros de ponto flutuante, e nao especifica precisao. Isso e problematico para numeros maiores que 2^53, que perdem precisao em JavaScript (que usa IEEE 754 double-precision). O Twitter, por exemplo, inclui IDs de tweets tanto como numero JSON quanto como string JSON, para evitar esse problema.
- **Suporte a dados binarios**: JSON e XML nao suportam sequencias de bytes brutas. A solucao comum e codificar em Base64, que aumenta o tamanho em 33%.
- **Schemas opcionais**: XML e JSON tem linguagens de schema (XML Schema, JSON Schema), mas muitas ferramentas nao as usam. CSV nao tem schema formal.

Apesar desses problemas, JSON, XML e CSV sao suficientes para muitos propositos, especialmente como formato de intercambio entre organizacoes diferentes.

**Binary variants de JSON/XML**: como JSON e XML sao texto, o encoding e relativamente verboso. Existem variantes binarias que visam reduzir o tamanho:

- Para JSON: **MessagePack**, **BSON** (usado internamente pelo MongoDB), **BJSON**, **UBJSON**, **SMILE**, **Amazon Ion**.
- Para XML: **WBXML**, **Fast Infoset**.

Kleppmann mostra um exemplo detalhado de como o MessagePack codifica um documento JSON de 81 bytes em 66 bytes. A economia e modesta porque os nomes dos campos (strings como "userName", "favoriteNumber") ainda precisam ser incluidos. Essa limitacao e endereacada por formatos com schema explicito como Protocol Buffers e Thrift.

### 4.2 Thrift e Protocol Buffers

**Apache Thrift** (originalmente desenvolvido no Facebook) e **Protocol Buffers** (protobuf, desenvolvido no Google) sao bibliotecas de encoding binario que requerem um **schema** definido em uma linguagem de definicao de interface (IDL). A partir do schema, ferramentas de code generation produzem classes em varias linguagens de programacao.

Exemplo de schema Protocol Buffers:
```
message Person {
    required string user_name = 1;
    optional int64 favorite_number = 2;
    repeated string interests = 3;
}
```

Os numeros (1, 2, 3) sao **field tags** (tags de campo). Eles identificam os campos no encoding binario — os nomes dos campos nao aparecem nos dados codificados. Isso torna o encoding muito mais compacto que JSON/MessagePack.

Thrift tem dois formatos de encoding binario:
- **BinaryProtocol**: cada campo e precedido pelo tipo e field tag. O exemplo de 81 bytes em JSON e codificado em 59 bytes.
- **CompactProtocol**: usa empacotamento mais eficiente (field tags e tipos combinados em um unico byte, inteiros com codificacao de comprimento variavel). O mesmo exemplo e codificado em 34 bytes.

Protocol Buffers e similar ao CompactProtocol do Thrift. O mesmo exemplo e codificado em 33 bytes.

#### Schema Evolution em Thrift e Protocol Buffers

A grande vantagem do uso de field tags numericos: campos podem ser renomeados livremente (os nomes sao apenas para o codigo gerado, nao aparecem no encoding). Novos campos podem ser adicionados com novas field tags.

**Forward compatibility**: codigo antigo encontra dados com field tags que nao reconhece. Como o encoding inclui o tipo de cada campo, o parser pode calcular quantos bytes pular para o campo desconhecido. Assim, codigo antigo simplesmente ignora campos novos.

**Backward compatibility**: codigo novo pode ler dados antigos. Campos adicionados apos o deployment inicial devem ser `optional` ou ter um valor default, pois dados antigos nao os conterao.

**Regras criticas**:
- Voce nao pode mudar a field tag de um campo existente — isso quebraria todos os dados ja codificados.
- Voce pode adicionar novos campos a qualquer momento, desde que sejam optional.
- Voce pode remover um campo, desde que nunca reutilize sua field tag (para nao confundir com dados antigos que ainda usam essa tag). Apenas campos optional podem ser removidos.
- Mudar o tipo de um campo e possivel com restricoes (por exemplo, int32 para int64 — codigo novo le o campo maior, codigo antigo trunca).

Thrift tem um tipo `list` com tipagem do elemento, enquanto Protocol Buffers usa `repeated` (que e internamente uma lista). Protocol Buffers permite mudar um campo `optional` para `repeated` (ou vice-versa): codigo antigo ve apenas o ultimo elemento se `repeated`, codigo novo ve um `optional` como lista de um elemento.

### 4.3 Avro

**Apache Avro** e outro formato de encoding binario que surgiu como subprojeto do Hadoop. Avro tambem usa schemas, mas a abordagem de schema evolution e fundamentalmente diferente de Thrift e Protocol Buffers.

No schema Avro, nao ha field tags. O schema simplesmente lista os campos com seus nomes e tipos:

```json
{
    "type": "record",
    "name": "Person",
    "fields": [
        {"name": "userName", "type": "string"},
        {"name": "favoriteNumber", "type": ["null", "long"], "default": null},
        {"name": "interests", "type": {"type": "array", "items": "string"}}
    ]
}
```

O encoding e extremamente compacto: nao ha field tags, nao ha tipo — apenas os valores concatenados em sequencia. O mesmo exemplo e codificado em apenas 32 bytes. O decodificador precisa saber exatamente o schema para interpretar os bytes.

#### Writer's Schema e Reader's Schema

A chave para entender o Avro e a distincao entre **writer's schema** (o schema usado para codificar os dados) e **reader's schema** (o schema que o codigo de leitura espera). Esses dois schemas nao precisam ser identicos — apenas precisam ser **compativeis**.

Quando o Avro decodifica dados, ele compara o writer's schema com o reader's schema lado a lado e resolve as diferencas:
- Se um campo esta no writer's schema mas nao no reader's schema, ele e ignorado.
- Se um campo esta no reader's schema mas nao no writer's schema, o valor default e usado (se definido).
- Se um campo esta em ambos mas com nomes diferentes, aliases podem ser usados.

A ordem dos campos nao importa — a resolucao e feita por nome de campo, nao por posicao.

**Forward compatibility**: um novo writer's schema (com novos campos) pode ser lido por codigo com um reader's schema antigo — os campos novos sao ignorados.

**Backward compatibility**: um novo reader's schema (esperando novos campos) pode ler dados com um writer's schema antigo — os campos ausentes recebem valores default.

Para manter compatibilidade, voce so pode adicionar ou remover campos que tenham valores default.

#### Como o Reader Conhece o Writer's Schema?

Se o writer's schema nao esta embutido nos dados (como estaria em JSON), como o reader sabe qual schema foi usado? Kleppmann descreve varios contextos:

1. **Arquivos grandes (Hadoop)**: o writer's schema e incluido uma vez no inicio do arquivo. Cada arquivo Avro (container file) contem o schema no cabecalho, e todas as entradas no arquivo usam esse schema.
2. **Banco de dados com registros individuais**: cada registro pode ter sido escrito em um momento diferente com um schema diferente. O registro inclui um numero de versao do schema, e um **schema registry** (registro de schemas) mapeia versoes para schemas completos.
3. **Comunicacao em rede**: ao negociar a conexao, o remetente e o receptor trocam schemas (por exemplo, no protocolo RPC do Avro).

#### Schema Registry

O **schema registry** e um conceito central no ecossistema Avro. E um servico que armazena todas as versoes do schema e fornece o schema correspondente a um dado numero de versao. O Confluent Schema Registry (para uso com Apache Kafka) e um exemplo popular. Ao escrever dados, o produtor registra o schema e inclui o ID do schema nos dados. Ao ler, o consumidor busca o schema pelo ID.

#### Vantagens do Avro

- **Schemas dinamicos**: como nao ha field tags numericas, o schema pode ser gerado automaticamente a partir de metadados (por exemplo, schema de um banco de dados relacional). Se uma coluna e adicionada ou removida, um novo schema Avro pode ser gerado automaticamente e o sistema continua funcionando. Com Thrift ou Protocol Buffers, a atribuicao manual de field tags tornaria esse processo mais trabalhoso.
- **Encoding muito compacto**: sem field tags ou tipos nos dados, o encoding e o mais compacto entre os formatos discutidos.

#### Schemas como Documentacao e Governanca

Kleppmann argumenta que schemas (em qualquer formato) sao valiosos alem do encoding eficiente:
- Funcionam como documentacao sempre atualizada dos dados.
- O schema registry mantem um historico de todas as versoes, permitindo verificar automaticamente forward e backward compatibility antes de implantar mudancas.
- Para linguagens com tipagem estatica, code generation a partir de schemas permite type checking em tempo de compilacao.

Comparado a JSON auto-descritivo, schemas binarios oferecem encoding mais compacto, documentacao embutida e garantias de compatibilidade — vantagens significativas para comunicacao entre servicos e armazenamento de dados.

### 4.4 Modes of Dataflow (Modos de Fluxo de Dados)

Dados fluem entre processos de diversas formas. Kleppmann categoriza os principais modos:

#### 4.4.1 Dataflow Through Databases

Quando um processo escreve dados codificados em um banco de dados, o processo que lera esses dados pode ser o mesmo processo no futuro, ou um processo diferente. Se o schema dos dados muda entre a escrita e a leitura, temos o equivalente a enviar uma mensagem para o "eu futuro".

Um cenario sutil: durante um rolling upgrade, nos antigos e novos coexistem. Um no novo pode escrever um registro com campos novos, e depois um no antigo pode ler esse registro, modifica-lo e reescreve-lo. Se o no antigo nao preservar os campos desconhecidos, eles serao perdidos silenciosamente. Isso e equivalente a uma falha de forward compatibility no nivel da aplicacao, nao do encoding.

Outro desafio: dados em bancos de dados podem persistir por anos ou decadas. Ao contrario de codigo (que pode ser atualizado em minutos via rolling upgrade), dados antigos permanecem em seu formato original indefinidamente — "data outlives code". Migrar dados retroativamente (reescrever todos os registros) e possivel mas extremamente cara em bancos grandes. Bancos relacionais permitem ALTER TABLE para adicionar colunas com valores default, e o banco preenche o default sob demanda na leitura, sem reescrever dados existentes. Schemas que suportam evolucao (como Avro) tambem lidam com isso naturalmente.

#### 4.4.2 Dataflow Through Services: REST and RPC

Quando processos precisam se comunicar pela rede, a abordagem mais comum e ter **clientes** e **servidores**. O servidor expoe uma API, e os clientes fazem requisicoes a essa API. A API do servidor e chamada de **servico**.

A web funciona assim: o navegador (cliente) faz requisicoes HTTP ao servidor web. A API consiste em URLs e parametros padronizados. Respostas sao tipicamente HTML, CSS, JavaScript, imagens.

No lado do servidor, uma arquitetura orientada a servicos (ou **microservices**) significa decompor uma aplicacao grande em servicos menores. Um servico pode ser cliente de outro servico. Servicos sao similares a bancos de dados no sentido de que permitem que clientes enviem e recebam dados, mas servicos expoe uma API especifica do dominio que permite apenas queries e operacoes pre-determinadas pela logica de negocios.

**REST** nao e um protocolo, mas uma filosofia de design baseada nos principios do HTTP. Enfatiza URLs para identificar recursos, uso de metodos HTTP (GET, PUT, POST, DELETE), cache e autenticacao via mecanismos HTTP padrao. Ganhou popularidade especialmente no formato de APIs "RESTful".

**SOAP** e um protocolo XML para requisicoes de rede. Evita recursos HTTP e usa seu proprio padrao para quase tudo. A API e descrita em WSDL (Web Services Description Language), permitindo code generation. WSDL nao e human-readable e as mensagens SOAP sao complexas demais para construcao manual, exigindo ferramentas pesadas. Caiu em desuso em favor de REST, especialmente em organizacoes menores.

#### RPC (Remote Procedure Call)

A ideia do RPC e fazer uma chamada de rede parecer uma chamada de funcao local. Kleppmann argumenta que essa abstracao e fundamentalmente falha, citando o artigo "A Note on Distributed Computing" de Waldo et al. (1994).

**Problemas fundamentais do RPC comparado a chamadas locais**:
- **Imprevisibilidade**: uma chamada local e previsivel (sucede ou falha com excecao). Uma chamada de rede pode falhar por timeout, e voce nao sabe se a requisicao chegou — pode ter sido processada mas a resposta se perdeu.
- **Timeouts**: uma chamada de rede lenta pode ficar esperando indefinidamente, diferentemente de uma chamada local que retorna em tempo previsivel.
- **Idempotencia**: se voce reenvia uma requisicao que deu timeout (sem saber se foi processada), pode causar duplicacao. Chamadas locais nao tem esse problema.
- **Latencia variavel**: chamadas de rede tem latencia variavel e imprevisivel, muito maior que chamadas locais.
- **Serializacao de parametros**: objetos complexos precisam ser serializados para bytes. Uma chamada local pode simplesmente passar um ponteiro.

Apesar desses problemas, frameworks RPC continuam sendo desenvolvidos: **gRPC** (usa Protocol Buffers), **Finagle** (usa Thrift), **Rest.li** (usa JSON sobre HTTP). Frameworks modernos nao tentam esconder que a chamada e remota — muitos usam futures/promises para resultados asincronos e suportam streams.

**Evolucao de APIs RPC**: e preciso assumir que servidores sao atualizados primeiro e clientes depois (porque voce controla o servidor mas nao os clientes). Compatibilidade do encoding (como discutido para Thrift/Protobuf/Avro) resolve parte do problema. Para APIs publicas REST, e comum usar versionamento na URL ou no header Accept.

#### 4.4.3 Dataflow Through Asynchronous Message Passing

Um modo intermediario entre RPC (comunicacao sincrona direta) e bancos de dados (armazenamento persistente): **message brokers** (corretores de mensagens) ou **message queues**.

O cliente (produtor) envia uma mensagem para uma fila ou topico, nao diretamente para o receptor. O broker armazena a mensagem temporariamente e a entrega a um ou mais consumidores. A comunicacao e **assincrona** e **unidirecional**: o produtor nao espera resposta.

**Vantagens sobre RPC direto**:
- **Buffer**: se o consumidor esta indisponivel ou sobrecarregado, o broker armazena as mensagens ate que o consumidor esteja pronto, melhorando a resiliencia.
- **Reentrega**: se um consumidor falha ao processar uma mensagem, o broker pode reentregar a mensagem para outro consumidor.
- **Desacoplamento**: o produtor nao precisa saber o endereco IP ou porta do consumidor.
- **Fan-out**: uma mensagem pode ser enviada para multiplos consumidores.

Exemplos de message brokers: **RabbitMQ**, **Apache ActiveMQ**, **HornetQ**, **Apache Kafka**, **NATS**. Kafka se diferencia por ser um log distribuido com retencao duravel, mais proximo de um banco de dados do que de um broker tradicional.

O formato de encoding das mensagens pode ser qualquer um (JSON, Protobuf, Avro, etc.). Se produtor e consumidor sao desenvolvidos independentemente, as mesmas consideracoes de compatibilidade (forward e backward) se aplicam.

#### Distributed Actor Frameworks

O **modelo de atores** e um modelo de programacao para concorrencia em um unico processo. Em vez de lidar com threads, locks e condicoes de corrida diretamente, a logica e encapsulada em **atores**. Cada ator e uma entidade com estado local que se comunica com outros atores enviando mensagens asincronas. Nao ha compartilhamento de memoria — atores processam uma mensagem de cada vez.

**Distributed actor frameworks** (como **Akka**, **Orleans** e **Erlang OTP**) estendem esse modelo para multiplas maquinas. A mesma abstracao de envio de mensagens e usada independentemente de os atores estarem no mesmo processo ou em maquinas diferentes. O framework cuida transparentemente do encoding, roteamento de rede e reentrega.

Porem, a transparencia de localizacao tem os mesmos problemas fundamentais do RPC: comunicacao local e remota sao fundamentalmente diferentes em termos de latencia e modos de falha. O modelo de atores pelo menos ja assume comunicacao assincrona e possibilidade de perda de mensagens, o que e mais honesto que o RPC tradicional.

Para schema evolution em actor frameworks, e necessario cuidado especial: como mensagens podem ser enviadas entre nos com versoes diferentes de codigo, as mesmas regras de forward e backward compatibility se aplicam. O suporte a rolling upgrades precisa ser planejado desde o inicio.

### Conclusoes do Capitulo 4

O capitulo 4 conecta o tema de encoding ao tema mais amplo de evolucao de sistemas. A escolha do formato de encoding tem implicacoes profundas na facilidade com que o sistema pode ser atualizado e evoluido ao longo do tempo. Formatos com schema explicito (Thrift, Protocol Buffers, Avro) oferecem encoding compacto, documentacao embutida e garantias de compatibilidade que formatos textuais (JSON, XML) nao oferecem nativamente. Os diferentes modos de dataflow (bancos de dados, servicos, mensageria) apresentam desafios diferentes para evolucao, mas todos se beneficiam de encoding com boas propriedades de compatibilidade.

---

## Reflexoes Finais sobre a Parte I

A Parte I do DDIA estabelece um vocabulario e um conjunto de frameworks conceituais que sao pre-requisitos para o restante do livro. Os quatro capitulos cobrem:

1. **Capitulo 1**: Os tres objetivos fundamentais (confiabilidade, escalabilidade, manutenibilidade) e como pensar sobre eles.
2. **Capitulo 2**: Como modelar dados (relacional, documento, grafo) e como consulta-los (SQL, MapReduce, Cypher, SPARQL, Datalog).
3. **Capitulo 3**: Como dados sao fisicamente armazenados e recuperados (hash indexes, LSM-trees, B-trees, armazenamento colunar) e as implicacoes para OLTP vs. OLAP.
4. **Capitulo 4**: Como dados sao codificados para transmissao e armazenamento (JSON, Protobuf, Avro) e como garantir evolucao compativel.

O tema unificador e que nao existem solucoes universais — apenas trade-offs. Cada decisao de design (modelo de dados, storage engine, formato de encoding, modo de comunicacao) tem vantagens e desvantagens que dependem do contexto especifico da aplicacao. O papel do engenheiro e entender essas trade-offs profundamente para fazer escolhas informadas. A Parte II (Distributed Data) e a Parte III (Derived Data) se constroem sobre esses fundamentos para abordar os desafios adicionais que surgem quando os dados sao distribuidos entre multiplas maquinas.
