# Domain-Driven Design: Tackling Complexity in the Heart of Software -- Eric Evans

## Resumo Detalhado Capitulo a Capitulo

---

# PARTE I: Colocando o Modelo de Dominio em Acao

A primeira parte do livro estabelece a filosofia fundamental do DDD: o software deve ser construido em torno de um modelo rico do dominio do negocio, e esse modelo deve ser o resultado de uma colaboracao intensa entre desenvolvedores e especialistas do dominio. Evans argumenta que a complexidade mais significativa de muitos projetos de software nao esta na tecnologia, mas no proprio dominio -- nas regras de negocio, nos processos e nas politicas que o software precisa representar.

---

## Capitulo 1: Crunching Knowledge (Triturando Conhecimento)

### Conceitos Centrais

**Knowledge Crunching** e o processo iterativo e colaborativo de extrair, organizar e refinar o conhecimento do dominio ate que ele se torne um modelo util para o software. Evans compara esse processo a "mastigar" informacao bruta ate transforma-la em algo digerivel e estruturado. Nao se trata de uma fase unica no inicio do projeto, mas de uma atividade continua que permeia todo o ciclo de vida do desenvolvimento.

O processo funciona assim: desenvolvedores e especialistas do dominio sentam juntos e comecam a conversar sobre o negocio. Os desenvolvedores fazem perguntas, os especialistas explicam, e juntos eles vao construindo um modelo mental compartilhado. Esse modelo e refinado a cada iteracao -- conceitos sao adicionados, removidos, renomeados e reestruturados conforme o entendimento evolui.

**Modelagem Efetiva** nao e sobre criar diagramas UML perfeitos ou documentos de requisitos exaustivos. E sobre criar uma abstracao do dominio que capture os aspectos mais relevantes para o problema que o software precisa resolver. Um bom modelo e como um mapa: ele nao precisa representar cada detalhe do territorio, mas precisa representar os detalhes certos de forma clara e util.

Evans enfatiza que o modelo deve ser:
- **Util para a implementacao**: se o modelo nao pode ser traduzido em codigo de forma razoavel, ele nao serve.
- **Fiel ao dominio**: se o modelo nao representa o negocio de forma que os especialistas reconhecam, ele esta errado.
- **Seletivo**: um modelo que tenta capturar tudo nao captura nada bem. A arte esta em escolher o que incluir e o que deixar de fora.

**Aprendizado Continuo** e um principio que Evans considera inegociavel. O conhecimento do dominio nunca esta completo no inicio do projeto. Os proprios especialistas do dominio muitas vezes nao conseguem articular todas as regras e nuances do negocio de uma so vez. E a medida que o software evolui, novos aspectos do dominio se tornam relevantes. Portanto, a equipe deve estar permanentemente aberta a aprender mais e a revisar o que ja sabe.

### Exemplos Praticos

Evans descreve um cenario de um sistema de transporte maritimo (shipping) onde a equipe inicialmente modela o dominio de forma simplista, com navios transportando cargas de um porto a outro. Conforme as conversas com especialistas avancam, o modelo evolui: surge o conceito de "viagem" (voyage), que encapsula a rota completa de um navio; depois percebe-se que o que realmente importa para o negocio nao e o navio em si, mas o itinerario da carga -- onde ela esta, para onde vai, quando chega. Cada iteracao dessas conversas refina o modelo e o aproxima da realidade do negocio.

Imagine um sistema de e-commerce. No inicio, a equipe pode modelar "Pedido" como um objeto simples com uma lista de produtos e um total. Mas ao conversar com o time de operacoes, descobre-se que um pedido pode ser parcialmente enviado, pode ter itens substituidos, pode ter diferentes estados de pagamento para diferentes itens. O modelo inicial nao suporta essa complexidade. O knowledge crunching revela essas nuances e o modelo evolui para acomodaIas.

### Regras e Diretrizes

1. Nunca modele sozinho. O modelo deve ser construido em parceria com especialistas do dominio.
2. Use diagramas e prototipos como ferramentas de comunicacao durante as sessoes de modelagem, nao como artefatos finais.
3. Esteja preparado para jogar fora modelos inteiros quando um entendimento melhor surgir. Apego a modelos obsoletos e um dos maiores inimigos do DDD.
4. Faca perguntas "bobas". Muitas vezes as respostas mais valiosas vem de perguntas que parecem obvias.
5. Preste atencao ao vocabulario dos especialistas. As palavras que eles usam naturalmente muitas vezes revelam conceitos importantes do dominio.

### Erros Comuns

- **Analise-paralise**: gastar semanas ou meses tentando criar o modelo "perfeito" antes de escrever qualquer codigo. O modelo deve evoluir junto com o codigo.
- **Delegar a modelagem**: achar que um "analista de negocios" pode fazer o knowledge crunching sozinho e entregar um documento pronto para os desenvolvedores implementarem. Os desenvolvedores PRECISAM participar diretamente.
- **Modelar tecnologia, nao dominio**: criar um modelo que reflete a estrutura do banco de dados ou do framework, em vez de refletir os conceitos do negocio.
- **Ignorar contradicoes**: quando dois especialistas do dominio discordam, isso geralmente indica uma complexidade importante que o modelo precisa acomodar, nao um problema a ser varrido para baixo do tapete.

### Principais Licoes do Capitulo

O software mais bem-sucedido e aquele construido por equipes que investem tempo genuino em entender o dominio. Knowledge crunching nao e uma fase, e uma pratica continua. O modelo do dominio e o artefato mais valioso do projeto, e ele deve ser tratado como um ser vivo que cresce e evolui.

---

## Capitulo 2: Communication and the Use of Language (Comunicacao e o Uso da Linguagem)

### Conceitos Centrais

**Linguagem Ubiqua (Ubiquitous Language)** e possivelmente o conceito mais importante de todo o livro. Evans argumenta que a maioria dos projetos de software sofre de um problema fundamental de comunicacao: os especialistas do dominio falam uma linguagem, os desenvolvedores falam outra, e entre os dois existe uma camada de traducao que introduz distorcoes, ambiguidades e perda de informacao.

A Linguagem Ubiqua e a solucao para esse problema. Ela e um vocabulario rigoroso e compartilhado que e usado por TODOS os envolvidos no projeto -- especialistas do dominio, desenvolvedores, gerentes, testadores -- em TODAS as formas de comunicacao -- conversas, documentos, codigo, testes, diagramas. Nao existem "dois idiomas" no projeto. Existe um so.

Essa linguagem nao e inventada pelos desenvolvedores e imposta aos especialistas, nem vice-versa. Ela emerge do processo de knowledge crunching e e refinada continuamente. Quando um termo e ambiguo, ele e discutido e clarificado. Quando um conceito nao tem um nome bom, um nome e criado. Quando o significado de um termo muda, todos sao informados.

**A Linguagem Molda o Codigo** de forma direta e literal. Os nomes de classes, metodos, variaveis e modulos no codigo devem usar exatamente os mesmos termos da Linguagem Ubiqua. Se os especialistas do dominio falam em "Apólice", o codigo deve ter uma classe chamada `Apolice`, nao `InsurancePolicy` ou `Contrato`. Se o processo de negocio e chamado de "Emissao", o metodo deve se chamar `emitir()`, nao `process()` ou `create()`.

Isso tem consequencias profundas. Quando o codigo usa a mesma linguagem que o negocio, os especialistas do dominio podem ler o codigo (ou pelo menos os nomes dos conceitos) e verificar se ele esta correto. Quando ha uma discrepancia entre o codigo e a linguagem, isso e um sinal de alarme: ou o codigo esta errado, ou a linguagem precisa ser atualizada.

**A Linguagem Molda a Conversa** tambem. Evans insiste que a equipe deve se policiar para usar os termos da Linguagem Ubiqua consistentemente, mesmo em conversas informais. Se alguem diz "aquele negocio que calcula o desconto", outro membro da equipe deve perguntar "voce quer dizer a Politica de Desconto?". Essa disciplina pode parecer pedante no inicio, mas ela previne mal-entendidos que custam dias ou semanas de retrabalho.

### Exemplos Praticos

Em um sistema bancario, a equipe pode descobrir que "conta" e um termo ambiguo: pode ser Conta Corrente, Conta Poupanca, ou Conta de Investimento, e cada uma tem regras completamente diferentes. A Linguagem Ubiqua exigiria que a equipe parasse de usar "conta" genericamente e adotasse termos especificos. O codigo refletiria isso com classes distintas: `ContaCorrente`, `ContaPoupanca`, `ContaInvestimento`.

Outro exemplo: em um sistema de logistica, os especialistas usam o termo "consolidacao" para se referir ao processo de agrupar varias cargas pequenas em um unico container. Se o desenvolvedor implementa isso como `agruparCargas()`, ele esta criando uma desconexao com o dominio. O metodo deveria se chamar `consolidar()`, e a classe que gerencia esse processo deveria se chamar `Consolidacao`.

Evans tambem mostra como diagramas e documentos devem usar a Linguagem Ubiqua. Um diagrama de fluxo que usa termos diferentes dos usados no codigo ou nas conversas esta causando dano, nao ajudando.

### Regras e Diretrizes

1. A Linguagem Ubiqua deve ser usada em todas as formas de comunicacao: codigo, testes, documentacao, conversas, e-mails, quadros brancos.
2. Quando um termo gera confusao, pare tudo e resolva a ambiguidade. Nao continue trabalhando com termos confusos.
3. A linguagem deve evoluir. Nao tente definir todo o vocabulario no inicio do projeto. Novos termos surgem, velhos termos sao refinados ou aposentados.
4. Teste a linguagem: tente descrever cenarios complexos do dominio usando apenas os termos da Linguagem Ubiqua. Se nao for possivel, faltam conceitos no modelo.
5. Documente a linguagem, mas de forma leve -- um glossario simples, mantido atualizado, e suficiente.

### Erros Comuns

- **Usar jargao tecnico no dominio**: chamar um processo de negocio de "workflow" ou "pipeline" quando os especialistas usam outros termos.
- **Traduzir mentalmente**: o desenvolvedor ouve o especialista dizer "sinistro" e traduz mentalmente para "incidente" no codigo. Essa traducao silenciosa e um veneno para o projeto.
- **Linguagem de fachada**: criar nomes bonitos na camada de apresentacao (API, UI) mas usar nomes tecnicos internamente. A Linguagem Ubiqua deve penetrar todas as camadas, especialmente a camada de dominio.
- **Nao corrigir desvios**: quando alguem usa um termo errado, nao corrigir por educacao. A correcao gentil e necessaria para manter a integridade da linguagem.
- **Linguagem ambigua aceita por conveniencia**: aceitar que "pedido" signifique coisas diferentes em contextos diferentes sem criar distincoes explicitas.

### Principais Licoes do Capitulo

A Linguagem Ubiqua e o alicerce do DDD. Sem ela, o modelo do dominio e o codigo inevitavelmente divergem, e o software se torna cada vez mais dificil de entender e modificar. A disciplina de manter uma linguagem compartilhada e rigorosa e trabalhosa, mas o retorno sobre esse investimento e enorme.

---

## Capitulo 3: Binding Model and Implementation (Vinculando Modelo e Implementacao)

### Conceitos Centrais

**Model-Driven Design (Design Dirigido pelo Modelo)** e o principio de que o modelo do dominio e o codigo devem ser a mesma coisa, nao duas representacoes separadas que precisam ser sincronizadas. Evans rejeita categoricamente a ideia de que o modelo e um artefato de "analise" que depois e "traduzido" para codigo por desenvolvedores. Nessa abordagem tradicional, o modelo e o codigo inevitavelmente divergem, e o modelo se torna um documento morto que ninguem consulta.

No Model-Driven Design, cada elemento do modelo tem uma representacao direta no codigo, e cada mudanca no modelo implica uma mudanca no codigo (e vice-versa). O modelo nao e um diagrama UML pendurado na parede -- ele vive no codigo. Se voce quer entender o modelo, leia o codigo da camada de dominio.

Isso nao significa que diagramas sao proibidos. Eles sao uteis como ferramentas de comunicacao e exploracao. Mas eles nao sao o modelo "oficial". O codigo e.

**Modeladores Praticos (Hands-On Modelers)** e outro principio fundamental. Evans argumenta que as pessoas que modelam o dominio devem ser as mesmas que escrevem o codigo. A separacao classica entre "arquitetos que modelam" e "programadores que implementam" e destrutiva para o DDD. O arquiteto que nao programa perde contato com a realidade da implementacao e cria modelos impraticaveis. O programador que nao participa da modelagem nao entende o "porque" por tras do codigo e faz escolhas que corrompem o modelo.

Evans nao esta dizendo que todos precisam ter o mesmo nivel de habilidade. Mas todos os desenvolvedores que tocam na camada de dominio precisam entender o modelo e participar de sua evolucao. E os modeladores mais experientes precisam manter as maos no codigo regularmente.

**O Ciclo Virtuoso** entre modelo e implementacao funciona assim: a equipe modela o dominio, implementa o modelo em codigo, descobre que certas partes do modelo nao funcionam bem no codigo, refina o modelo, atualiza o codigo, descobre novas nuances do dominio durante a implementacao, incorpora essas nuances no modelo, e assim por diante. Esse ciclo e a essencia do DDD.

### Exemplos Praticos

Evans contrasta dois projetos. No primeiro, uma equipe de analistas criou um modelo detalhado do dominio em UML, documentou tudo em um documento de 200 paginas, e entregou para os desenvolvedores. Os desenvolvedores olharam o modelo, perceberam que ele nao mapeava bem para a tecnologia escolhida, e criaram seu proprio design "tecnico". O resultado: dois modelos separados, nenhum deles fiel ao dominio, e uma equipe que nao conseguia se comunicar.

No segundo projeto, desenvolvedores e especialistas modelaram juntos, usando quadros brancos e prototipos rapidos. Cada conceito do modelo foi imediatamente implementado em codigo. Quando a implementacao revelava problemas no modelo, o modelo era ajustado. O resultado: um codigo que os especialistas do dominio podiam ler e validar, e um modelo que os desenvolvedores podiam implementar naturalmente.

Um exemplo concreto: ao modelar um sistema de reservas de hotel, a equipe pode inicialmente criar uma classe `Reserva` com um metodo `cancelar()`. Ao implementar, descobre-se que o cancelamento tem regras complexas (prazo, taxa, tipo de quarto). O modelo evolui para incluir uma `PoliticaDeCancelamento` que encapsula essas regras. Essa evolucao so acontece quando quem modela tambem implementa.

### Regras e Diretrizes

1. O codigo da camada de dominio deve ser uma expressao direta do modelo. Se o modelo diz "um Pedido contem Itens", o codigo deve ter uma classe `Pedido` com uma colecao de `Item`.
2. Nao crie "camadas de traducao" entre o modelo e o codigo. O modelo E o codigo.
3. Todo desenvolvedor que trabalha na camada de dominio deve participar das sessoes de modelagem.
4. Mudancas no modelo devem ser refletidas no codigo imediatamente, e vice-versa. Divergencia e divida tecnica da pior especie.
5. Se o modelo nao pode ser implementado de forma clara no codigo, o problema e do modelo, nao do codigo. Refine o modelo.
6. Escolha paradigmas e tecnologias que facilitem a expressao do modelo em codigo. Linguagens orientadas a objetos sao naturalmente boas para isso, mas nao sao a unica opcao.

### Erros Comuns

- **Modelo "de analise" separado do modelo "de design"**: ter dois modelos e pior do que ter nenhum, porque eles inevitavelmente divergem e causam confusao.
- **Arquitetos de torre de marfim**: pessoas que desenham modelos lindos mas nunca implementam codigo. Seus modelos sao irreais.
- **Codigo que "funciona" mas nao reflete o modelo**: um sistema onde as regras de negocio estao espalhadas em stored procedures, scripts de integracao e logica de UI, sem uma camada de dominio coerente.
- **Escolher tecnologia antes do modelo**: deixar que o framework ou o banco de dados ditem como o dominio e representado, em vez de o dominio ditar como a tecnologia e usada.

### Principais Licoes do Capitulo

O poder do DDD vem da uniao inseparavel entre modelo e codigo. Quando os dois sao a mesma coisa, a equipe ganha clareza, comunicacao e capacidade de evolucao. Quando sao separados, o projeto sofre com ambiguidade, retrabalho e entropia. Os melhores modeladores sao aqueles que tambem programam.

---

# PARTE II: Os Building Blocks do Model-Driven Design

A segunda parte do livro apresenta os padroes taticos do DDD -- os blocos de construcao que a equipe usa para expressar o modelo do dominio em codigo. Esses padroes nao sao invenções de Evans; muitos existiam antes do DDD. Mas Evans os organiza em um sistema coerente e mostra como eles trabalham juntos para criar um design rico e expressivo.

---

## Capitulo 4: Isolating the Domain (Isolando o Dominio)

### Conceitos Centrais

**Arquitetura em Camadas (Layered Architecture)** e o padrao arquitetural que Evans recomenda como pre-requisito para o DDD. A ideia central e simples: separar o codigo em camadas com responsabilidades distintas, de forma que a logica do dominio fique isolada de preocupacoes tecnicas como interface de usuario, persistencia de dados e integracao com sistemas externos.

Evans define quatro camadas:

**1. Camada de Interface de Usuario (User Interface / Presentation Layer)**
Responsavel por apresentar informacoes ao usuario e interpretar seus comandos. Pode ser uma interface web, uma API REST, uma CLI, ou qualquer outro mecanismo de interacao. Esta camada NAO contem logica de negocio. Ela delega tudo para a camada de aplicacao.

**2. Camada de Aplicacao (Application Layer)**
Coordena as atividades da aplicacao. Ela nao contem logica de negocio, mas orquestra o uso dos objetos de dominio para realizar tarefas especificas. Pense nela como um "maestro": ela nao toca nenhum instrumento, mas diz para cada musico quando tocar. Um servico de aplicacao pode, por exemplo, buscar um Pedido no repositorio, chamar um metodo do dominio para aplicar um desconto, e depois salvar o Pedido atualizado. A logica de "como aplicar o desconto" esta no dominio, nao na aplicacao.

Esta camada e fina. Se voce encontrar muita logica aqui, provavelmente esta vazando logica de dominio para fora do dominio.

**3. Camada de Dominio (Domain Layer / Model Layer)**
O coracao do software. Aqui vivem as entidades, os objetos de valor, os servicos de dominio, as regras de negocio, as invariantes -- tudo que representa o conhecimento do dominio. Esta camada nao sabe nada sobre bancos de dados, APIs, interfaces de usuario ou frameworks. Ela e pura logica de negocio.

Esta e a camada que o DDD mais se preocupa em projetar bem. Todos os padroes dos capitulos seguintes se aplicam primariamente aqui.

**4. Camada de Infraestrutura (Infrastructure Layer)**
Fornece capacidades tecnicas genericas que suportam as camadas superiores: persistencia de dados (implementacoes de repositorios), envio de e-mails, mensageria, logging, etc. Esta camada implementa interfaces definidas pelas camadas superiores (especialmente pelo dominio), seguindo o Principio da Inversao de Dependencia.

**A Regra de Dependencia** e crucial: as dependencias devem fluir de cima para baixo (UI -> Aplicacao -> Dominio -> Infraestrutura), e nunca o contrario. Na pratica moderna, o dominio define interfaces (portas) que a infraestrutura implementa (adaptadores), permitindo que o dominio nao dependa de nada tecnico.

### Exemplos Praticos

Considere uma funcionalidade de "Realizar Pagamento" em um e-commerce:

- **UI**: recebe a requisicao HTTP com os dados do pagamento, valida o formato dos dados, e chama o servico de aplicacao.
- **Aplicacao**: `RealizarPagamentoService` busca o `Pedido` no repositorio, busca o `MeioDePagamento` do cliente, chama `pedido.realizarPagamento(meioDePagamento)`, e salva o resultado.
- **Dominio**: `Pedido.realizarPagamento()` verifica se o pedido esta em um estado que permite pagamento, calcula o valor final (com descontos, frete, etc.), cria um objeto `Pagamento`, e atualiza o estado do pedido. Se algo estiver errado, lanca uma excecao de dominio.
- **Infraestrutura**: `PedidoRepositoryImpl` salva o pedido no banco de dados. `GatewayDePagamentoImpl` se comunica com o gateway de pagamento externo.

Note como a logica de negocio ("pedido so pode ser pago se estiver no estado X", "desconto nao pode exceder Y%") esta toda na camada de dominio.

### Regras e Diretrizes

1. A camada de dominio NUNCA deve depender de camadas externas. Ela nao importa nada de infraestrutura, aplicacao ou UI.
2. A camada de aplicacao deve ser fina. Se ela tem mais de algumas linhas de orquestracao, provavelmente ha logica de dominio vazando.
3. Use inversao de dependencia para que o dominio defina suas necessidades (interfaces) e a infraestrutura as satisfaca (implementacoes).
4. Nao coloque regras de negocio em stored procedures, triggers de banco de dados, ou middlewares de framework. Elas pertencem ao dominio.
5. Testes unitarios da camada de dominio devem ser possiveis sem nenhuma infraestrutura (sem banco, sem rede, sem filesystem).

### Erros Comuns

- **Camada de dominio "anemica"**: classes de dominio que sao apenas structs com getters e setters, sem nenhuma logica. Toda a logica vive em servicos de aplicacao enormes. Isso e o chamado "Anemic Domain Model", um antipadrao que Evans condena explicitamente.
- **Logica de negocio na UI**: validacoes de negocio em JavaScript no frontend, regras em controllers, logica em templates.
- **Dominio acoplado a infraestrutura**: a classe `Pedido` que conhece o JPA/Hibernate, que importa anotacoes de banco de dados, que depende de um framework especifico.
- **Camada de aplicacao gorda**: servicos de aplicacao com centenas de linhas de logica condicional que deveriam estar no dominio.

### Principais Licoes do Capitulo

O isolamento do dominio e um pre-requisito para todo o resto do DDD. Se a logica de negocio esta espalhada por todas as camadas, nenhum dos padroes dos capitulos seguintes pode ser aplicado efetivamente. A arquitetura em camadas nao e um fim em si mesma -- ela existe para proteger e nutrir a camada de dominio.

---

## Capitulo 5: A Model Expressed in Software (Um Modelo Expresso em Software)

### Conceitos Centrais

Este e um dos capitulos mais densos e importantes do livro. Evans apresenta os quatro elementos fundamentais para expressar um modelo de dominio em codigo orientado a objetos: Entidades, Objetos de Valor, Servicos e Modulos.

---

### Entidades (Entities)

Uma Entidade e um objeto que possui uma **identidade unica** que persiste ao longo do tempo e atraves de diferentes representacoes. A caracteristica definidora de uma Entidade nao e seus atributos, mas sua identidade. Dois objetos com os mesmos atributos mas identidades diferentes sao objetos diferentes. Um objeto cujos atributos mudam completamente ao longo do tempo, mas que mantem a mesma identidade, continua sendo o mesmo objeto.

**Identidade vs. Igualdade**: este e um ponto crucial. Para Entidades, igualdade e determinada pela identidade, nao pelos atributos. Duas instancias de `Cliente` com o mesmo CPF sao o mesmo cliente, mesmo que uma tenha o endereco antigo e outra o novo. Em codigo, isso significa que o metodo `equals()` de uma Entidade compara apenas o identificador.

A identidade pode ser:
- **Natural**: um atributo do dominio que naturalmente identifica o objeto (CPF, CNPJ, numero de matricula).
- **Atribuida**: um identificador gerado pelo sistema (UUID, sequencial).
- **Composta**: uma combinacao de atributos que juntos formam a identidade.

A gestao de identidade e uma responsabilidade seria. Em sistemas distribuidos, garantir unicidade de identidade pode ser um desafio significativo.

Entidades geralmente sao mutaveis -- seus atributos mudam ao longo do tempo. Um `Pedido` muda de estado, um `Cliente` muda de endereco. A identidade permanece.

**Quando usar Entidades**: use quando o objeto precisa ser rastreado individualmente ao longo do tempo, quando existe a nocao de "este mesmo objeto" em diferentes momentos ou contextos, quando a identidade importa mais do que os atributos.

---

### Objetos de Valor (Value Objects)

Um Objeto de Valor e um objeto que descreve alguma caracteristica ou atributo, mas **nao possui identidade propria**. Ele e definido completamente pelos seus atributos. Dois Objetos de Valor com os mesmos atributos sao iguais e intercambiaveis.

Exemplos classicos: `Dinheiro` (valor + moeda), `Endereco` (rua, numero, cidade, CEP), `Periodo` (dataInicio, dataFim), `Cor` (R, G, B), `CPF` (numero).

**Imutabilidade**: Objetos de Valor devem ser imutaveis. Uma vez criado, um Objeto de Valor nao muda. Se voce precisa de um valor diferente, cria um novo objeto. Isso simplifica enormemente o raciocinio sobre o codigo: voce pode passar Objetos de Valor livremente sem se preocupar com efeitos colaterais.

**Funcoes Livres de Efeitos Colaterais (Side-Effect-Free Functions)**: como Objetos de Valor sao imutaveis, todas as suas operacoes sao funcoes puras. `dinheiro.somar(outroDinheiro)` retorna um NOVO objeto `Dinheiro` com o resultado da soma, sem modificar nenhum dos operandos. Isso torna o codigo mais previsivel e testavel.

**Igualdade por valor**: o metodo `equals()` de um Objeto de Valor compara todos os atributos. `new Dinheiro(100, "BRL").equals(new Dinheiro(100, "BRL"))` deve retornar `true`.

**Compartilhamento e copia**: como Objetos de Valor sao imutaveis, eles podem ser compartilhados livremente entre Entidades sem risco. Varios pedidos podem referenciar o mesmo Objeto de Valor `Endereco` sem problemas.

**Quando usar Objetos de Valor**: use sempre que possivel. Evans enfatiza que a maioria dos objetos em um modelo bem projetado deveria ser Objetos de Valor, nao Entidades. Se voce nao precisa rastrear a identidade de um objeto ao longo do tempo, ele provavelmente e um Objeto de Valor. Objetos de Valor sao mais simples, mais seguros e mais performaticos do que Entidades.

**O erro de tornar tudo Entidade**: um erro muito comum e modelar objetos como Entidades quando eles deveriam ser Objetos de Valor. Se `Endereco` e uma Entidade, voce precisa gerenciar sua identidade, se preocupar com mutabilidade, e lidar com complexidade desnecessaria. Se `Endereco` e um Objeto de Valor, ele e simplesmente um conjunto de dados que descreve uma localizacao.

---

### Servicos (Services)

Nem toda logica de dominio pertence naturalmente a uma Entidade ou Objeto de Valor. Quando uma operacao importante do dominio nao se encaixa em nenhum objeto especifico, ela deve ser modelada como um Servico.

Um Servico de Dominio e uma operacao **sem estado** que realiza algo significativo no dominio. Ele nao possui atributos proprios -- apenas comportamento. Ele opera sobre Entidades e Objetos de Valor, mas nao pertence a nenhum deles especificamente.

**Caracteristicas de um bom Servico de Dominio**:
- A operacao que ele realiza se refere a um conceito do dominio que nao pertence naturalmente a uma Entidade ou Objeto de Valor.
- A interface e definida em termos de outros elementos do modelo de dominio.
- A operacao e sem estado (stateless).

**Exemplo**: `TransferenciaService.transferir(contaOrigem, contaDestino, valor)`. A transferencia nao pertence nem a conta de origem nem a de destino -- e uma operacao que envolve ambas. Forcar a transferencia dentro de uma das contas (`contaOrigem.transferirPara(contaDestino, valor)`) distorce o modelo.

**Cuidado com o excesso de servicos**: se voce tem muitos servicos e poucas entidades com comportamento, provavelmente esta caindo no antipadrao do Modelo de Dominio Anemico. Servicos devem ser a excecao, nao a regra. A maioria da logica de negocio deve viver dentro de Entidades e Objetos de Valor.

**Tres tipos de servicos** (nao confundir):
- **Servicos de Dominio**: logica de negocio que nao pertence a um objeto especifico. Vivem na camada de dominio.
- **Servicos de Aplicacao**: orquestracao de casos de uso. Vivem na camada de aplicacao.
- **Servicos de Infraestrutura**: capacidades tecnicas (envio de email, integracao). Vivem na camada de infraestrutura.

---

### Modulos (Modules / Packages)

Modulos sao agrupamentos de elementos do modelo que possuem alta coesao interna e baixo acoplamento externo. Eles sao a forma como organizamos o modelo em partes gerenciaveis.

**Modulos devem refletir o dominio, nao a tecnologia**. Um modulo chamado `pedidos` que contem `Pedido`, `ItemDePedido`, `PoliticaDePreco` e `PedidoRepository` faz muito mais sentido do que modulos chamados `entities`, `repositories`, `services` que agrupam por tipo tecnico.

Evans adverte que modulos sao frequentemente negligenciados, mas sao muito importantes para a comunicacao. Os nomes dos modulos devem fazer parte da Linguagem Ubiqua. Quando alguem diz "o modulo de faturamento", todos devem saber exatamente que conceitos estao la.

### Exemplos Praticos

**Entidade vs. Objeto de Valor -- o caso do Dinheiro**:
`Dinheiro(100, "BRL")` e um Objeto de Valor. Nao importa qual nota de R$100 voce tem -- qualquer uma serve. Elas sao intercambiaveis. Mas uma `NotaDeDinheiro` com numero de serie e uma Entidade -- cada nota e unica e rastreavel.

**Servico de Dominio -- calculo de frete**:
O calculo de frete depende do endereco de entrega (do Pedido), do peso e dimensoes dos produtos (dos Itens), e das regras da transportadora (uma Politica). Nao pertence a nenhuma Entidade especifica. Um `CalculoDeFrete` como Servico de Dominio e apropriado.

### Regras e Diretrizes

1. Prefira Objetos de Valor a Entidades sempre que possivel.
2. Torne Objetos de Valor imutaveis. Sem excecoes.
3. Use Servicos de Dominio apenas quando a operacao genuinamente nao pertence a nenhum objeto.
4. Mantenha Servicos de Dominio sem estado.
5. Organize modulos por conceitos de dominio, nao por tipos tecnicos.
6. O metodo `equals()` de Entidades compara identidade; o de Objetos de Valor compara atributos.

### Erros Comuns

- **Modelo Anemico**: Entidades sem comportamento, apenas com getters/setters, toda logica em Servicos.
- **Objetos de Valor mutaveis**: permitir que um `Endereco` seja modificado apos criacao, causando bugs sutis quando o mesmo objeto e compartilhado.
- **Identidade desnecessaria**: dar IDs a tudo, mesmo quando nao ha necessidade de rastreamento individual.
- **Servicos para tudo**: criar `PedidoService`, `ClienteService`, `ProdutoService` que fazem todo o trabalho enquanto as Entidades sao cascas vazias.
- **Modulos tecnicos**: pacotes como `com.empresa.entities`, `com.empresa.services`, `com.empresa.repositories` em vez de `com.empresa.pedidos`, `com.empresa.catalogo`, `com.empresa.faturamento`.

### Principais Licoes do Capitulo

A escolha entre Entidade e Objeto de Valor e uma das decisoes mais importantes do design de dominio. Objetos de Valor, por serem imutaveis e sem identidade, sao mais simples e devem ser preferidos. Servicos existem para operacoes orfas, mas nao devem ser usados em excesso. Modulos organizam o modelo de forma que facilite a comunicacao.

---

## Capitulo 6: The Lifecycle of a Domain Object (O Ciclo de Vida de um Objeto de Dominio)

### Conceitos Centrais

Objetos de dominio nao existem em isolamento temporal. Eles sao criados, passam por transformacoes, sao armazenados, recuperados, e eventualmente descartados. Evans apresenta tres padroes para gerenciar esse ciclo de vida de forma que preserve a integridade do modelo: Agregados, Fabricas e Repositorios.

---

### Agregados (Aggregates)

O Agregado e talvez o padrao mais dificil de aplicar corretamente e o mais impactante quando aplicado bem. Um Agregado e um cluster de objetos de dominio que sao tratados como uma unidade para fins de mudanca de dados.

**Raiz do Agregado (Aggregate Root)**: todo Agregado tem uma unica Entidade que serve como ponto de entrada. Objetos externos ao Agregado so podem referenciar a raiz, nunca os objetos internos diretamente.

**Fronteira do Agregado (Aggregate Boundary)**: define quais objetos fazem parte do Agregado. Objetos dentro da fronteira podem se referenciar livremente, mas objetos fora so enxergam a raiz.

**Invariantes**: as regras de consistencia que devem ser sempre verdadeiras dentro do Agregado. A raiz e responsavel por garantir essas invariantes. Nenhuma mudanca nos objetos internos pode violar as invariantes do Agregado.

**Regras fundamentais dos Agregados**:

1. A raiz tem identidade global (pode ser referenciada de qualquer lugar do sistema). Objetos internos tem identidade local (unica apenas dentro do Agregado).
2. Nada fora do Agregado pode manter uma referencia direta a um objeto interno. A raiz pode passar referencias transitorias (para uso imediato), mas o objeto externo nao deve armazena-las.
3. Apenas a raiz pode ser obtida diretamente de um Repositorio. Objetos internos sao acessados por navegacao a partir da raiz.
4. Quando a raiz e deletada, todos os objetos internos sao deletados junto (cascata).
5. Quando qualquer objeto dentro do Agregado e modificado, todas as invariantes do Agregado inteiro devem ser satisfeitas ao final da operacao.
6. Uma transacao nao deve abranger multiplos Agregados. Cada Agregado e uma fronteira transacional.

**Exemplo classico**: `Pedido` (raiz) contem `ItensDePedido` (internos). Voce nao pode acessar um `ItemDePedido` sem passar pelo `Pedido`. Se voce precisa adicionar um item, faz `pedido.adicionarItem(produto, quantidade)` -- o Pedido valida as invariantes (ex: quantidade minima, limite de itens) e cria o item internamente. Ninguem de fora cria um `ItemDePedido` solto e o injeta no Pedido.

**Dimensionamento de Agregados**: Agregados devem ser pequenos. A tentacao e criar Agregados enormes que contem tudo que esta relacionado, mas isso causa problemas de performance (carregamento excessivo), concorrencia (locks amplos) e complexidade. Se um objeto dentro do Agregado pode existir independentemente e ser acessado por conta propria, ele provavelmente deveria ser sua propria raiz de Agregado, referenciado por ID.

**Consistencia eventual entre Agregados**: como uma transacao nao deve abranger multiplos Agregados, a consistencia entre Agregados e eventual, nao imediata. Se o processamento de um `Pagamento` precisa atualizar o estado de um `Pedido`, isso pode ser feito via eventos de dominio ou processamento assincrono.

---

### Fabricas (Factories)

Quando a criacao de um objeto (ou Agregado) e complexa, essa complexidade deve ser encapsulada em uma Fabrica. A Fabrica e responsavel por criar objetos de dominio em um estado valido e consistente.

**Por que Fabricas sao necessarias**: a criacao de um Agregado pode envolver a instanciacao de multiplos objetos, a configuracao de relacionamentos entre eles, a validacao de invariantes iniciais, e a geracao de identidade. Colocar toda essa logica no construtor da raiz poluiria a raiz com responsabilidades que nao sao suas. Colocar no codigo cliente (servico de aplicacao, por exemplo) vazaria conhecimento de dominio para fora do dominio.

**Tipos de Fabricas**:
- **Factory Method**: um metodo na propria raiz do Agregado ou em outra Entidade que cria objetos relacionados. Ex: `pedido.adicionarItem(produto, quantidade)` cria internamente um `ItemDePedido`.
- **Abstract Factory**: uma interface que define a criacao de familias de objetos relacionados.
- **Factory standalone**: uma classe dedicada a criacao de Agregados complexos.

**Regras para Fabricas**:
1. Toda Fabrica deve produzir objetos em um estado valido. Se as invariantes nao podem ser satisfeitas, a criacao deve falhar (excecao).
2. A Fabrica deve ser uma operacao atomica. Nao deve retornar objetos parcialmente construidos.
3. A Fabrica nao deve depender da infraestrutura. Ela e pura logica de dominio.
4. Para Objetos de Valor simples, um construtor e suficiente. Nao crie Fabricas desnecessarias.

---

### Repositorios (Repositories)

Um Repositorio fornece a ilusao de uma colecao em memoria de todos os objetos de um determinado tipo. Ele encapsula toda a logica de acesso a dados (persistencia, consulta, reconstrucao) atras de uma interface que faz sentido para o dominio.

**A interface do Repositorio pertence ao dominio**: a interface `PedidoRepository` com metodos como `salvar(pedido)`, `buscarPorId(id)`, `buscarPendentes()` e definida na camada de dominio. A implementacao (que usa JPA, Mongo, arquivo, o que for) vive na camada de infraestrutura.

**Repositorios sao apenas para Raizes de Agregado**: voce nao cria um `ItemDePedidoRepository`. Se voce precisa de um item, busca o Pedido e navega ate o item.

**Repositorios vs. DAOs**: embora superficialmente parecidos, sao conceitos diferentes. Um DAO e orientado a dados (CRUD generico sobre tabelas). Um Repositorio e orientado ao dominio (fornece operacoes que fazem sentido para o modelo). Um Repositorio pode usar um DAO internamente, mas sua interface e definida pelo dominio, nao pelo banco.

**Estrategias de consulta**: os metodos de consulta do Repositorio devem usar a Linguagem Ubiqua. `repositorio.buscarPedidosAtrasados()` e melhor do que `repositorio.buscar(new Criteria("status", "=", "atrasado").and("dataLimite", "<", hoje))`. Consultas complexas podem ser encapsuladas usando o padrao Specification.

**Reconstrucao**: quando um Repositorio recupera um objeto do banco de dados, ele nao esta "criando" um novo objeto -- esta "reconstruindo" um objeto que ja existe. Essa distincao e sutil mas importante. A reconstrucao pode usar uma Fabrica internamente, mas com regras diferentes (por exemplo, nao gera um novo ID, usa o existente).

### Exemplos Praticos

**Agregado de Pedido**:
```
Pedido (Raiz do Agregado)
  - id: PedidoId
  - cliente: ClienteId  (referencia por ID, nao por objeto)
  - itens: List<ItemDePedido>  (objetos internos)
  - status: StatusPedido
  - valorTotal: Dinheiro

  + adicionarItem(produto, quantidade): void  (garante invariantes)
  + removerItem(itemId): void
  + calcularTotal(): Dinheiro
  + confirmar(): void  (muda status, valida que tem itens)
```

`ItemDePedido` nao tem repositorio proprio, nao pode ser acessado diretamente de fora do Pedido, e nao tem identidade global.

**Fabrica de Pedido**:
```
PedidoFactory
  + criarPedido(clienteId, itens): Pedido
    // Gera ID
    // Cria cada ItemDePedido
    // Calcula valor total inicial
    // Valida invariantes (ex: minimo 1 item)
    // Retorna Pedido valido ou lanca excecao
```

**Repositorio de Pedido**:
```
interface PedidoRepository (no dominio):
  + salvar(pedido): void
  + buscarPorId(id): Pedido
  + buscarPorCliente(clienteId): List<Pedido>
  + buscarPendentesDeEnvio(): List<Pedido>

PedidoRepositoryImpl (na infraestrutura):
  // Implementa usando JPA, Mongo, etc.
```

### Regras e Diretrizes

1. Mantenha Agregados pequenos. Prefira referenciar outros Agregados por ID, nao por objeto.
2. Uma transacao = um Agregado. Use consistencia eventual entre Agregados.
3. A raiz do Agregado e a unica guardia das invariantes internas.
4. Todo Repositorio e para uma raiz de Agregado, nunca para objetos internos.
5. A interface do Repositorio pertence ao dominio; a implementacao pertence a infraestrutura.
6. Fabricas criam objetos novos em estado valido. Repositorios reconstroem objetos existentes.
7. Nao use Fabricas para objetos simples -- construtores bastam.

### Erros Comuns

- **Agregados gigantes**: colocar tudo dentro de um unico Agregado "porque esta relacionado". Um `Cliente` com todos os seus `Pedidos`, `Enderecos`, `Pagamentos` dentro do mesmo Agregado e um desastre de performance e concorrencia.
- **Referenciar objetos internos**: permitir que codigo externo obtenha e armazene referencias a objetos dentro do Agregado, contornando a raiz.
- **Repositorios para tudo**: criar repositorios para Objetos de Valor ou objetos internos de Agregados.
- **Logica de dominio no Repositorio**: colocar regras de negocio dentro da implementacao do repositorio (ex: aplicar desconto na hora de salvar).
- **Ignorar invariantes na criacao**: permitir que Agregados sejam criados em estados invalidos "porque serao validados depois".
- **Transacoes entre Agregados**: tentar garantir consistencia imediata entre multiplos Agregados em uma unica transacao.

### Principais Licoes do Capitulo

Agregados sao fronteiras de consistencia. Eles protegem as invariantes do dominio e definem os limites transacionais. Fabricas garantem que objetos nascem validos. Repositorios fornecem acesso a objetos persistidos sem expor detalhes de infraestrutura. Juntos, esses tres padroes gerenciam o ciclo de vida completo dos objetos de dominio de forma coerente e alinhada ao modelo.

---

## Capitulo 7: Using the Language: An Extended Example (Usando a Linguagem: Um Exemplo Estendido)

### Conceitos Centrais

Este capitulo e um exercicio pratico e estendido que aplica todos os conceitos das Partes I e II a um cenario realista: um sistema de transporte maritimo de cargas. Evans caminha pelo processo completo de modelagem, desde as primeiras conversas com especialistas ate o codigo final, mostrando como cada decisao de design e motivada pelo dominio.

O capitulo demonstra:

**A interacao entre modelo e linguagem**: a equipe começa com termos vagos e vai refinando. "Aquela coisa que rastreia a carga" se torna uma `EspecificacaoDeRota` (Route Specification), um Objeto de Valor que descreve de onde para onde a carga deve ir, sem especificar o caminho exato.

**A evolucao do modelo**: o modelo inicial tem `Carga`, `Navio` e `Viagem`. Ao refinar, a equipe percebe que o que importa nao e o navio, mas o `Itinerario` -- a sequencia de `Trechos` (Legs) que a carga percorre. O `Navio` sai do modelo principal e se torna um detalhe de infraestrutura.

**A aplicacao dos building blocks**:
- `Carga` e uma Entidade (tem identidade -- numero de rastreamento).
- `EspecificacaoDeRota` e um Objeto de Valor (descreve o destino desejado).
- `Itinerario` e um Objeto de Valor (descreve o caminho planejado).
- `EventoDeManseio` (Handling Event) e uma Entidade que registra o que aconteceu com a carga em cada ponto.
- `ServicoDeProgramacao` (Booking Service) e um Servico de Dominio que encontra itinerarios que satisfazem uma especificacao de rota.

**O padrao Specification**: Evans introduz o padrao Specification aqui de forma pratica. A `EspecificacaoDeRota` nao e apenas um Objeto de Valor passivo -- ela tem um metodo `eSatisfeitaPor(itinerario)` que verifica se um dado itinerario atende aos requisitos. Isso encapsula a logica de matching no dominio, em vez de espalha-la por servicos.

### Exemplos Praticos

O capitulo inteiro e um exemplo pratico, acompanhando a modelagem do sistema de carga desde o inicio:

1. Primeira sessao: especialistas explicam como funciona o transporte maritimo. A equipe identifica os conceitos centrais: Carga, Viagem, Porto.
2. Refinamento: descobre-se que uma Carga pode ser transferida entre viagens. Surge o conceito de Itinerario.
3. Mais refinamento: descobre-se que o cliente especifica origem e destino, mas nao o caminho. Surge a EspecificacaoDeRota como conceito distinto do Itinerario.
4. Aplicacao de Agregados: Carga se torna a raiz de um Agregado que contem seu Itinerario e EspecificacaoDeRota. EventosDeManseio sao outro Agregado, referenciando a Carga por ID.
5. Repositorios: `CargaRepository` para buscar cargas por numero de rastreamento. `ViagemRepository` para buscar viagens disponiveis.

### Regras e Diretrizes

1. Comece modelando com os especialistas, nao com diagramas de banco de dados.
2. Deixe o dominio guiar as decisoes de design, nao a tecnologia.
3. Use o padrao Specification para encapsular logica de selecao e validacao complexa.
4. Nao tenha medo de refatorar o modelo quando um entendimento melhor surgir.
5. Cada elemento do modelo (classe, metodo, modulo) deve ter um nome que faca parte da Linguagem Ubiqua.

### Erros Comuns

- **Pular direto para o codigo**: comecar a codar sem investir tempo em modelagem com os especialistas.
- **Modelo estatico**: tratar o modelo como definitivo apos a primeira sessao. O modelo deve evoluir.
- **Ignorar a Specification**: espalhar logica de matching e validacao em varios lugares em vez de encapsula-la em um objeto dedicado.

### Principais Licoes do Capitulo

A teoria dos capitulos anteriores ganha vida quando aplicada a um cenario real. O processo de modelagem e iterativo, desordenado e recompensador. Os building blocks (Entidades, Objetos de Valor, Servicos, Agregados, Fabricas, Repositorios) nao sao usados mecanicamente -- eles sao ferramentas que a equipe escolhe deliberadamente com base no entendimento do dominio.

---

# PARTE III: Refatorando em Direcao a um Insight Mais Profundo

A terceira parte do livro e sobre a maturacao do modelo de dominio. Os building blocks da Parte II dao a estrutura, mas e o refinamento continuo que transforma um modelo adequado em um modelo verdadeiramente profundo e expressivo. Evans argumenta que os maiores avancos em projetos de DDD nao vem da aplicacao mecanica de padroes, mas de momentos de insight onde a equipe descobre uma forma fundamentalmente melhor de representar o dominio.

---

## Capitulo 8: Breakthrough (Avancos)

### Conceitos Centrais

**Breakthroughs (Avancos)** sao momentos onde o modelo de dominio da um salto qualitativo. Nao sao melhorias incrementais -- sao reformulacoes fundamentais onde a equipe percebe que estava olhando para o dominio de um angulo errado e descobre um angulo muito melhor.

Evans descreve breakthroughs como experiencias quase misticas no desenvolvimento de software. A equipe pode estar lutando com um modelo por semanas, sentindo que algo esta errado mas sem conseguir identificar o que. Entao alguem faz uma observacao -- muitas vezes durante uma conversa casual com um especialista do dominio -- e de repente tudo se encaixa. O modelo se simplifica dramaticamente, problemas que pareciam intratáveis se resolvem naturalmente, e o codigo fica muito mais expressivo.

**O paradoxo do breakthrough**: esses momentos sao imprevisíveis, mas nao sao aleatorios. Eles so acontecem em equipes que estao ativamente envolvidas em knowledge crunching continuo, que mantem uma Linguagem Ubiqua disciplinada, e que estao dispostas a refatorar o modelo quando necessario. Breakthroughs sao o premio pela pratica diligente do DDD.

**O custo do breakthrough**: reformular o modelo pode ser doloroso. Significa reescrever codigo, atualizar testes, repensar integrações. Evans argumenta que o custo e quase sempre compensado pelos beneficios -- codigo mais simples, menos bugs, maior facilidade de evolucao. Mas a equipe deve estar preparada para o investimento.

**Resistencia ao breakthrough**: muitas equipes percebem que o modelo precisa mudar mas resistem porque "ja funciona". Evans avisa: adiar um breakthrough e como adiar o pagamento de uma divida tecnica com juros compostos. Quanto mais tempo o modelo errado fica, mais codigo e construido sobre ele, e mais caro fica muda-lo.

### Exemplos Praticos

Evans conta a historia de um projeto de seguros onde a equipe lutou por meses com um modelo que tinha dezenas de classes e regras condicionais complexas para calcular premios. O breakthrough veio quando alguem percebeu que os diferentes tipos de calculo podiam ser modelados como uma composicao de componentes de risco independentes (Share, o conceito central do dominio de seguros), em vez de uma hierarquia rigida de tipos de apolice. O modelo resultante era dramaticamente mais simples e flexivel.

Em outro exemplo, uma equipe trabalhando em um sistema de emprestimos bancarios tinha dificuldade em modelar os diferentes tipos de taxas de juros. O breakthrough veio quando perceberam que a "taxa de juros" nao era apenas um numero, mas uma funcao complexa que variava ao longo do tempo. Modelar a taxa como um objeto com comportamento (em vez de um campo numerico) resolveu dezenas de problemas de uma so vez.

### Regras e Diretrizes

1. Cultive o ambiente para breakthroughs: mantenha as sessoes de modelagem frequentes, misture desenvolvedores e especialistas, e esteja aberto a questionar premissas.
2. Preste atencao nos "cheiros" de que um breakthrough e necessario: excesso de condicionais, classes que crescem descontroladamente, dificuldade em explicar o modelo para novos membros.
3. Quando um breakthrough acontecer, nao hesite. Invista o tempo necessario para reformular o modelo e o codigo.
4. Documente o antes e o depois. Breakthroughs sao valiosos como aprendizado para a equipe.

### Erros Comuns

- **Confundir refatoracao incremental com breakthrough**: melhorar nomes de metodos e extrair classes e importante, mas nao e um breakthrough. Breakthroughs envolvem mudanca de paradigma no modelo.
- **Resistir ao breakthrough por medo de retrabalho**: o custo de nao mudar cresce exponencialmente.
- **Esperar o momento perfeito**: breakthroughs raramente vem quando voce esta "pronto". Eles vem quando a equipe esta imersa no dominio.

### Principais Licoes do Capitulo

Os melhores modelos de dominio nao sao projetados de uma vez -- eles emergem atraves de refinamentos iterativos pontuados por breakthroughs transformadores. A equipe deve cultivar as condicoes para que esses breakthroughs acontecam e ter a coragem de agir quando eles surgirem.

---

## Capitulo 9: Making Implicit Concepts Explicit (Tornando Conceitos Implicitos Explicitos)

### Conceitos Centrais

Uma das tecnicas mais poderosas de refinamento de modelo e identificar conceitos que estao implicitos no codigo ou nas conversas e torna-los explicitos no modelo. Quando uma regra de negocio esta escondida dentro de uma condicional complexa, quando um processo esta implicito na sequencia de chamadas de metodos, quando uma restricao esta codificada mas nao nomeada -- esses sao conceitos implicitos esperando para serem explicitados.

**Onde encontrar conceitos implicitos**:

1. **Na linguagem dos especialistas**: quando um especialista usa um termo que nao existe no modelo, provavelmente e um conceito que deveria existir. Se o especialista diz "a politica de cancelamento nao permite isso", mas o codigo nao tem uma classe `PoliticaDeCancelamento`, ha um conceito implicito.

2. **Em condicoes de guarda complexas**: se um metodo tem cinco `if` aninhados que verificam se uma operacao e permitida, provavelmente ha uma Specification ou Policy escondida ali.

3. **Em parametros que sempre andam juntos**: se tres parametros sempre sao passados juntos (ex: valor, moeda, data de conversao), provavelmente ha um Objeto de Valor pedindo para nascer.

4. **Na documentacao e nas regras de negocio**: regras escritas em documentos mas nao refletidas como objetos de primeira classe no modelo.

**Constraints (Restricoes)**: regras que limitam o que pode acontecer no dominio. "Um pedido nao pode ter mais de 50 itens" e uma restricao. Se ela esta escondida em uma linha de codigo, deveria ser explicitada como uma constante nomeada, um metodo de validacao, ou ate mesmo um objeto de restricao.

**Processes (Processos)**: sequencias de atividades que transformam algo no dominio. Muitas vezes processos sao codificados como procedimentos longos em servicos. Torna-los explicitos -- como objetos de primeira classe -- pode simplificar enormemente o modelo.

**Specification (Especificacao)**: um padrao especialmente importante. Uma Specification e um objeto que encapsula uma regra de negocio ou criterio de selecao. Ela tem um metodo (`eSatisfeitaPor(objeto)`) que retorna verdadeiro ou falso.

As Specifications podem ser usadas para:
- **Validacao**: verificar se um objeto satisfaz certos criterios.
- **Selecao**: filtrar objetos de uma colecao.
- **Criacao sob demanda**: especificar os requisitos que um objeto a ser criado deve atender.

Specifications podem ser combinadas usando logica booleana (AND, OR, NOT), criando criterios compostos sofisticados.

### Exemplos Praticos

**De implicito a explicito -- Politica de Desconto**:

Antes (implicito):
```
// Dentro de Pedido.calcularTotal()
if (cliente.tipo == "VIP" && itens.size() > 10 && dataAtual.mes == 12) {
    desconto = 0.15;
} else if (cliente.tipo == "VIP") {
    desconto = 0.10;
} else if (itens.size() > 20) {
    desconto = 0.05;
}
```

Depois (explicito):
```
PoliticaDeDescontoVIP         -- Specification
PoliticaDeDescontoNatalVIP    -- Specification
PoliticaDeDescontoVolume      -- Specification

pedido.aplicarDesconto(politicaDeDesconto);
```

**Specification para selecao**:
```
EspecificacaoDeClienteInadimplente spec =
    new EspecificacaoDeClienteInadimplente(90); // 90 dias

List<Cliente> inadimplentes =
    clienteRepository.buscarSatisfazendo(spec);
```

**Processo explicito**:
Em vez de um metodo `processarDevolucao()` com 200 linhas, criar um objeto `ProcessoDeDevolucao` que encapsula os estados, regras e transicoes do processo.

### Regras e Diretrizes

1. Fique atento a conceitos implicitos: ifs complexos, parametros que andam juntos, termos dos especialistas nao refletidos no codigo.
2. Use Specifications para encapsular regras de selecao e validacao.
3. Nomeie restricoes explicitamente, mesmo que sejam simples.
4. Quando um processo de negocio e significativo, modele-o como um objeto de primeira classe.
5. Teste se o modelo ficou mais expressivo: o codigo deve ser mais facil de ler e discutir apos a refatoracao.

### Erros Comuns

- **Over-engineering**: tornar explicito demais. Nem toda condicao precisa ser uma Specification. Use bom senso.
- **Specifications pra tudo**: usar Specifications para regras simples que ficariam melhor como metodos comuns.
- **Nao questionar o implicito**: aceitar logica condicional complexa como "normal" sem questionar se ha conceitos escondidos.

### Principais Licoes do Capitulo

Os modelos de dominio mais ricos sao aqueles que tornam explicitos os conceitos que outros modelos deixam implicitos. Cada conceito implicito que se torna uma classe, interface ou metodo nomeado e uma oportunidade de tornar o codigo mais expressivo, testavel e alinhado com o dominio.

---

## Capitulo 10: Supple Design (Design Flexivel)

### Conceitos Centrais

Evans argumenta que um bom design de dominio nao e apenas correto -- ele e "flexivel" (supple), no sentido de que e facil de usar, facil de entender e facil de evoluir. Um design flexivel convida a mudanca; um design rigido a resiste. Este capitulo apresenta um conjunto de principios e padroes que, quando aplicados juntos, criam um design flexivel.

---

**Intention-Revealing Interfaces (Interfaces que Revelam a Intencao)**

Os nomes de classes e metodos devem comunicar claramente o que fazem, nao como fazem. O usuario de uma classe deve entender o que ela faz apenas lendo os nomes, sem precisar olhar a implementacao.

Exemplo ruim: `pedido.processar(3, true, null)`
Exemplo bom: `pedido.confirmarComDescontoDe(3).semFrete()`

O metodo `processar` nao revela nada sobre sua intencao. O que significa `3`? O que significa `true`? O que acontece com `null`? Ja `confirmarComDescontoDe` e `semFrete` comunicam claramente a intencao.

Isso se aplica especialmente ao design de Objetos de Valor e Entidades. Os metodos publicos dessas classes formam uma "linguagem" que o desenvolvedor-cliente usa para expressar operacoes de dominio. Essa linguagem deve ser clara e natural.

---

**Side-Effect-Free Functions (Funcoes Livres de Efeitos Colaterais)**

Operacoes que retornam resultados sem modificar estado observavel sao muito mais faceis de raciocinar, testar e combinar. Evans recomenda maximizar o uso de funcoes puras, especialmente em Objetos de Valor.

Quando um metodo muda o estado do objeto, o desenvolvedor precisa rastrear mentalmente todas as possiveis sequencias de chamadas para entender o comportamento. Quando um metodo e puro (recebe dados, retorna dados, nao muda nada), ele pode ser invocado livremente em qualquer ordem, quantas vezes quiser.

Na pratica: separe operacoes em **comandos** (mudam estado, nao retornam nada) e **consultas** (retornam dados, nao mudam estado). Isso e o principio CQS (Command-Query Separation). Objetos de Valor devem ter apenas consultas (ja que sao imutaveis). Entidades podem ter ambos, mas devem maximizar as consultas.

---

**Assertions (Assercoes)**

Como complemento a funcoes puras, Evans recomenda que os efeitos colaterais de comandos sejam documentados atraves de **pos-condicoes** (o que e verdade depois que o comando executa) e **invariantes** (o que e sempre verdade sobre o objeto).

Se voce nao pode eliminar efeitos colaterais (Entidades mudam de estado), pelo menos torne-os explicitos e previsíveis. Contratos (pre-condicoes, pos-condicoes, invariantes) sao a forma de fazer isso.

Em linguagens que suportam, isso pode ser feito com assertions literais no codigo. Em outras, pode ser feito com documentacao rigorosa e testes que verificam as condicoes.

---

**Conceptual Contours (Contornos Conceituais)**

Decomponha o modelo de forma que cada parte corresponda a um conceito coeso do dominio. Os "contornos" das classes e metodos devem seguir os contornos naturais do dominio, nao linhas arbitrarias.

Se uma classe faz duas coisas conceitualmente distintas, ela deveria ser dividida. Se duas classes representam partes inseparaveis de um unico conceito, elas deveriam ser unidas.

Como encontrar os contornos certos: observe quais partes do modelo mudam juntas e quais mudam independentemente. Partes que mudam juntas provavelmente pertencem ao mesmo conceito. Partes que mudam independentemente provavelmente sao conceitos distintos.

---

**Standalone Classes (Classes Autonomas)**

Quanto menos dependencias uma classe tem, mais facil ela e de entender e usar. Evans recomenda minimizar dependencias e buscar, quando possivel, criar classes que possam ser entendidas completamente em isolamento.

Isso nao significa que classes nao devem colaborar -- significa que cada classe deve ser compreensível por si mesma, sem precisar entender todas as suas colaboradoras. Objetos de Valor sao frequentemente bons exemplos de classes autonomas: `Dinheiro`, `CPF`, `Periodo` podem ser entendidos sem conhecer o resto do sistema.

---

**Closure of Operations (Fecho de Operacoes)**

Quando possivel, defina operacoes que retornam o mesmo tipo dos operandos. `Dinheiro.somar(Dinheiro): Dinheiro` e um exemplo perfeito. O resultado da soma de dois objetos `Dinheiro` e outro `Dinheiro`. Isso cria uma algebra natural que permite composicao de operacoes.

Esse principio vem da matematica (grupos, aneis) e e especialmente poderoso para Objetos de Valor. Quando um tipo e "fechado" sob certas operacoes, o desenvolvedor pode combinar operacoes livremente sem se preocupar com conversoes de tipo.

### Exemplos Praticos

**Closure of Operations com Dinheiro**:
```
Dinheiro preco = new Dinheiro(100, "BRL");
Dinheiro desconto = preco.multiplicarPor(0.10);  // retorna Dinheiro
Dinheiro precoFinal = preco.subtrair(desconto);   // retorna Dinheiro
```

Cada operacao retorna `Dinheiro`, permitindo encadeamento natural e seguro.

**Intention-Revealing Interface para um Repositorio**:
Ruim: `repo.find(new Query("status=active AND type=premium"))`
Bom: `repo.buscarClientesPremiumAtivos()`

**Conceptual Contours -- dividindo uma classe inchada**:
Se `Pedido` tem metodos para calcular frete, aplicar descontos, gerar nota fiscal e processar devolucao, provavelmente ha conceitos implicitos. `CalculoDeFrete`, `PoliticaDeDesconto`, `NotaFiscal` e `ProcessoDeDevolucao` podem ser extraidos como classes separadas, cada uma com seus proprios contornos conceituais.

### Regras e Diretrizes

1. Nomeie tudo pela intencao, nao pela implementacao.
2. Maximize funcoes puras, especialmente em Objetos de Valor.
3. Use CQS: comandos nao retornam, consultas nao mudam estado.
4. Documente efeitos colaterais com pre/pos-condicoes e invariantes.
5. Siga os contornos naturais do dominio ao decidir as fronteiras de classes.
6. Minimize dependencias. Busque classes autonomas.
7. Busque fechamento de operacoes em Objetos de Valor.

### Erros Comuns

- **Nomes tecnicos em interfaces de dominio**: `processPayment()`, `executeTransfer()`, `handleOrder()` em vez de nomes que refletem o dominio.
- **Misturar comandos e consultas**: um metodo que muda estado E retorna um valor e uma fonte de confusao.
- **Classes com muitas dependencias**: se para entender uma classe voce precisa entender dez outras, o design precisa ser simplificado.
- **Contornos arbitrarios**: dividir ou agrupar classes por criterios tecnicos (tamanho do arquivo, tipo de padrao) em vez de conceituais.

### Principais Licoes do Capitulo

Um design flexivel nao e um acidente -- e o resultado da aplicacao deliberada de principios que tornam o codigo legivel, compreensível e evoluivel. Interfaces que revelam intencao, funcoes puras, assercoes, contornos conceituais, classes autonomas e fecho de operacoes trabalham juntos para criar um design que e um prazer de usar e evoluir.

---

## Capitulo 11: Applying Analysis Patterns (Aplicando Padroes de Analise)

### Conceitos Centrais

**Padroes de Analise** sao modelos reutilizaveis que capturam estruturas recorrentes em dominios de negocio. Evans referencia fortemente o trabalho de Martin Fowler ("Analysis Patterns") e mostra como padroes de analise podem acelerar o processo de modelagem.

A ideia e que muitos dominios compartilham estruturas semelhantes. Um modelo de "conta" com debitos, creditos e saldo aparece em bancos, em contabilidade, em sistemas de pontos de fidelidade, em carteiras digitais. Um modelo de "regra de negocio" com condicoes e acoes aparece em seguros, em emprestimos, em sistemas de precificacao.

**Como usar Padroes de Analise no DDD**:

1. Padroes de Analise sao **pontos de partida**, nao solucoes prontas. Eles devem ser adaptados ao dominio especifico do projeto.
2. Eles devem ser integrados ao modelo de dominio usando a Linguagem Ubiqua do projeto. Nao adianta copiar os nomes do padrao se eles nao fazem sentido para os especialistas do dominio.
3. Eles sao mais uteis como inspiracao durante sessoes de knowledge crunching. Quando a equipe esta lutando com um conceito, um padrao de analise pode sugerir uma estrutura que funciona.

Evans da o exemplo do padrao "Account" (Conta) e mostra como ele pode ser aplicado a um dominio de investimentos, onde cada "conta" pode ter multiplas "transacoes" de diferentes tipos (compra, venda, dividendo, taxa), e o "saldo" e calculado de forma diferente dependendo do tipo de conta.

### Exemplos Praticos

**Padrao Quantity (Quantidade)**:
Em vez de representar quantidades como numeros primitivos, usa-se um Objeto de Valor que associa um numero a uma unidade de medida: `Quantidade(10, "kg")`. Isso evita erros de conversao e torna explicita a unidade.

**Padrao Range (Faixa)**:
Um Objeto de Valor que representa um intervalo: `Periodo(dataInicio, dataFim)` com metodos como `contem(data)`, `sobrepoe(outroPeriodo)`, `uniao(outroPeriodo)`. Este padrao aparece em inumeros dominios.

**Padrao Money (Dinheiro)**:
Ja discutido: um Objeto de Valor que encapsula valor e moeda com aritmetica propria.

### Regras e Diretrizes

1. Estude padroes de analise existentes antes de modelar do zero. Alguem provavelmente ja resolveu problemas semelhantes.
2. Adapte o padrao ao seu dominio. Nunca force um padrao que nao se encaixa.
3. Use os nomes do padrao apenas se fizerem sentido na Linguagem Ubiqua do projeto.
4. Padroes de analise sao mais uteis em dominios complexos e maduros (financas, seguros, logistica, saude).

### Erros Comuns

- **Aplicar padroes mecanicamente**: usar um padrao so porque ele existe, sem verificar se ele realmente resolve o problema do dominio.
- **Fidelidade excessiva ao padrao**: insistir em usar todas as partes de um padrao mesmo quando so uma parte e relevante.
- **Inventar a roda**: recusar-se a consultar padroes existentes e reinventar solucoes que ja sao bem compreendidas.

### Principais Licoes do Capitulo

Padroes de Analise sao um acelerador valioso para o DDD, mas devem ser usados com discernimento. Eles sao ferramentas de inspiracao e ponto de partida, nao receitas prontas. A adaptacao ao dominio especifico e a integracao com a Linguagem Ubiqua sao essenciais.

---

## Capitulo 12: Relating Design Patterns to the Model (Relacionando Padroes de Design ao Modelo)

### Conceitos Centrais

Evans mostra como padroes de design classicos (do livro "Gang of Four") podem ser usados nao apenas como solucoes tecnicas, mas como elementos do modelo de dominio. Quando um padrao de design corresponde a um conceito real do dominio, ele ganha um significado adicional que vai alem de sua utilidade tecnica.

**Strategy (Estrategia)**

O padrao Strategy permite definir uma familia de algoritmos encapsulaIos em objetos separados e torna-los intercambiaveis. No contexto do DDD, Strategies sao frequentemente usadas para modelar politicas de dominio.

Exemplo: em um sistema de logistica, o calculo de frete pode variar conforme a politica de envio (economico, expresso, internacional). Cada politica e uma Strategy:

```
interface PoliticaDeFrete {
    Dinheiro calcular(Pedido pedido);
}

class FreteEconomico implements PoliticaDeFrete { ... }
class FreteExpresso implements PoliticaDeFrete { ... }
class FreteInternacional implements PoliticaDeFrete { ... }
```

O ponto crucial de Evans e que `PoliticaDeFrete` nao e apenas um padrao tecnico -- e um conceito do dominio. Os especialistas falam em "politicas de frete". A Strategy reflete o dominio, nao apenas a tecnica.

**Composite (Composicao)**

O padrao Composite permite tratar objetos individuais e composicoes de objetos de forma uniforme. No DDD, ele e util quando o dominio tem estruturas hierarquicas ou recursivas.

Exemplo: em um sistema de precificacao, um desconto pode ser simples (10% off) ou composto (10% off + frete gratis + brinde). Um `DescontoComposto` contem uma lista de `Desconto` e aplica todos eles. O `Desconto` individual e o `DescontoComposto` implementam a mesma interface.

Outro exemplo: uma estrutura organizacional onde um `Departamento` pode conter outros `Departamentos` e `Funcionarios`, e ambos implementam uma interface `UnidadeOrganizacional`.

### Exemplos Praticos

**Strategy no dominio de seguros**:
Uma seguradora tem diferentes estrategias de calculo de risco: `CalculoDeRiscoResidencial`, `CalculoDeRiscoAutomotivo`, `CalculoDeRiscoVida`. Cada uma encapsula regras complexas e especificas. A interface `EstrategiaDeCalculoDeRisco` e um conceito que os atuarios reconhecem e discutem.

**Composite no dominio de regras de negocio**:
Uma regra de aprovacao de credito pode ser composta de sub-regras: `RegraDeRendaMinima AND (RegraDeHistoricoPositivo OR RegraDeGarantiaSuficiente)`. Cada regra e um Specification, e as composicoes usam o padrao Composite.

### Regras e Diretrizes

1. Use padroes de design quando eles refletem conceitos do dominio, nao apenas quando resolvem problemas tecnicos.
2. Os nomes das classes que implementam o padrao devem usar termos da Linguagem Ubiqua, nao nomes tecnicos (prefira `PoliticaDeFrete` a `FreteStrategy`).
3. Composite e especialmente util para Specifications, regras de negocio e estruturas hierarquicas.
4. Strategy e especialmente util para politicas e calculos que variam.

### Erros Comuns

- **Aplicar padroes por habito tecnico**: usar Strategy, Composite e outros padroes GoF como solucoes tecnicas genericas sem verificar se eles refletem o dominio.
- **Nomes tecnicos no dominio**: chamar a classe de `FretteStrategy` em vez de `PoliticaDeFrete`.
- **Complexidade desnecessaria**: usar Composite quando uma lista simples bastaria.

### Principais Licoes do Capitulo

Padroes de design ganham um poder especial quando correspondem a conceitos do dominio. A convergencia entre um padrao tecnico e um conceito de dominio e um sinal de que o modelo esta no caminho certo.

---

## Capitulo 13: Refactoring Toward Deeper Insight (Refatorando em Direcao a um Insight Mais Profundo)

### Conceitos Centrais

Este capitulo fecha a Parte III sintetizando o processo de refinamento continuo do modelo. Evans distingue entre dois tipos de refatoracao:

**Refatoracao tecnica**: melhorar a estrutura do codigo sem mudar seu comportamento. Renomear variaveis, extrair metodos, eliminar duplicacao. Importante, mas nao transforma o modelo.

**Refatoracao de dominio**: mudar o modelo para refletir um entendimento mais profundo do dominio. Introduzir novos conceitos, reformular relacionamentos, redefinir fronteiras de Agregados. Esta e a refatoracao que Evans mais valoriza.

A refatoracao de dominio e motivada por:
- **Linguagem desalinhada**: quando os termos do codigo nao correspondem aos termos dos especialistas.
- **Rigidez**: quando mudanças simples no negocio requerem mudancas complexas no codigo.
- **Conceitos implicitos**: quando ha logica escondida que deveria ser explicita (capitulo 9).
- **Design inflexivel**: quando o design viola os principios do capitulo 10.
- **Novos insights**: quando a equipe descobre algo novo sobre o dominio.

**O processo de refatoracao profunda**:
1. Identifique a dor: que parte do codigo e dificil de mudar, entender ou discutir?
2. Busque o conceito: que conceito do dominio esta ausente ou mal representado?
3. Experimente: tente diferentes formas de modelar o conceito.
4. Escolha a melhor opcao com base em clareza, expressividade e fidelidade ao dominio.
5. Implemente incrementalmente, mantendo o sistema funcional a cada passo.

### Exemplos Praticos

Evans mostra um projeto onde uma equipe refatorou progressivamente um modelo de emprestimos bancarios. Cada iteracao descobriu conceitos mais profundos:

1. Primeira versao: `Emprestimo` com um campo `taxaDeJuros` (numero).
2. Refatoracao 1: `taxaDeJuros` se torna um Objeto de Valor `TaxaDeJuros` com metodo `calcular(valor, periodo)`.
3. Refatoracao 2: percebe-se que existem taxas fixas e variaveis. `TaxaDeJuros` se torna uma interface com implementacoes `TaxaFixa` e `TaxaVariavel` (Strategy).
4. Refatoracao 3: descobre-se que a taxa variavel depende de um indice economico que muda ao longo do tempo. Surge o conceito de `IndiceEconomico` como Entidade.
5. Refatoracao 4: o calculo de juros compostos ao longo de multiplos periodos revela a necessidade de um `PlanoDeAmortizacao` como Objeto de Valor complexo.

Cada refatoracao tornou o modelo mais fiel ao dominio e o codigo mais simples e expressivo.

### Regras e Diretrizes

1. Refatoracao de dominio e tao importante quanto refatoracao tecnica. Reserve tempo para ambas.
2. Nao espere o modelo perfeito. Refatore continuamente em direcao a um modelo melhor.
3. Mantenha os testes atualizados. Refatoracao sem testes e cirurgia sem anestesia.
4. Envolva os especialistas do dominio na refatoracao. Eles podem confirmar se o novo modelo e mais fiel.
5. Documente os insights que motivaram a refatoracao. Eles sao conhecimento valioso.

### Erros Comuns

- **Refatorar apenas tecnicamente**: melhorar a estrutura do codigo sem melhorar o modelo. O codigo fica "limpo" mas continua desconectado do dominio.
- **Refatorar demais de uma vez**: tentar mudar o modelo inteiro em uma unica iteracao. Prefira passos pequenos e seguros.
- **Nao refatorar por medo**: deixar o modelo degradar por medo de quebrar algo. Testes robustos mitigam esse medo.

### Principais Licoes do Capitulo

O caminho para um modelo de dominio profundo e feito de muitos passos pequenos de refatoracao, pontuados por breakthroughs ocasionais. A equipe deve cultivar uma cultura de refatoracao continua, motivada nao apenas por qualidade tecnica, mas por fidelidade ao dominio.

---

# PARTE IV: Design Estrategico

A quarta parte do livro eleva a perspectiva do nivel tatico (como modelar objetos individuais) para o nivel estrategico (como organizar multiplos modelos em sistemas grandes). Em sistemas reais, nao existe um unico modelo perfeito que abrange todo o dominio. Existem multiplos modelos, cada um servindo uma parte do negocio, e a questao estrategica e como esses modelos se relacionam.

---

## Capitulo 14: Maintaining Model Integrity (Mantendo a Integridade do Modelo)

### Conceitos Centrais

Este e o capitulo mais extenso e estrategicamente importante do livro. Evans apresenta os padroes que definem como multiplos modelos coexistem e se relacionam em um sistema grande.

---

**Bounded Context (Contexto Delimitado)**

O Bounded Context e a fronteira explicita dentro da qual um modelo de dominio e definido e aplicavel. Dentro de um Contexto Delimitado, cada termo da Linguagem Ubiqua tem um significado preciso e unico. Fora dessa fronteira, os mesmos termos podem ter significados completamente diferentes.

Exemplo: o termo "Produto" pode significar coisas muito diferentes no contexto de Catalogo (tem nome, descricao, fotos, categorias), no contexto de Estoque (tem SKU, quantidade, localizacao), e no contexto de Faturamento (tem codigo fiscal, aliquota de imposto). Tentar criar um unico modelo de "Produto" que sirva para todos esses contextos resultaria em uma classe monolitica, confusa e fragil.

A solucao e reconhecer que existem tres Contextos Delimitados, cada um com seu proprio modelo de "Produto", otimizado para as necessidades daquele contexto. Cada contexto tem sua propria Linguagem Ubiqua, suas proprias classes, e seu proprio ciclo de evolucao.

**Um Bounded Context nao e necessariamente um microservico**. Ele pode ser um modulo dentro de um monolito, uma biblioteca, um servico, ou qualquer outra unidade de deploy. O que define o Bounded Context e a fronteira do modelo, nao a fronteira do deploy.

---

**Context Map (Mapa de Contextos)**

O Context Map e um diagrama (ou documento) que mostra todos os Bounded Contexts do sistema e os relacionamentos entre eles. Ele e uma ferramenta de comunicacao estrategica que ajuda a equipe a entender a paisagem dos modelos.

O Context Map deve ser realista, nao idealista. Ele mostra como os contextos realmente se relacionam, incluindo relacionamentos problematicos, nao como a equipe gostaria que eles se relacionassem.

Tipos de relacionamentos entre Contextos:

---

**Shared Kernel (Nucleo Compartilhado)**

Dois Contextos Delimitados compartilham um subconjunto do modelo. Esse subconjunto e mantido em comum, e mudancas nele requerem acordo entre as equipes de ambos os contextos.

O Shared Kernel reduz duplicacao, mas cria acoplamento. Qualquer mudanca no nucleo compartilhado pode quebrar ambos os contextos. Evans recomenda que o Shared Kernel seja pequeno e que haja testes automatizados abrangentes cobrindo a integracao.

Exemplo: dois contextos (Vendas e Faturamento) compartilham o modelo de `Cliente` basico (ID, nome, documento). Mudancas nesse modelo sao coordenadas entre as equipes.

---

**Customer/Supplier (Cliente/Fornecedor)**

Uma relacao upstream/downstream onde o contexto "fornecedor" (upstream) fornece dados ou servicos que o contexto "cliente" (downstream) consome. O fornecedor tem um compromisso de atender as necessidades do cliente, mas o cliente nao influencia o modelo interno do fornecedor.

Exemplo: o contexto de Pedidos (upstream) fornece dados de pedidos para o contexto de Relatorios (downstream). A equipe de Pedidos se compromete a manter a API estavel e a considerar as necessidades de Relatorios ao fazer mudancas.

---

**Conformist (Conformista)**

Similar a Customer/Supplier, mas sem o compromisso. O contexto downstream simplesmente aceita o modelo do upstream como ele e, sem influencia-lo. Isso acontece quando o upstream e um sistema externo, um legado que nao pode ser modificado, ou quando a equipe upstream nao se dispoe a colaborar.

Ser Conformista simplifica a integracao (nao ha traducao), mas pode forcar o downstream a trabalhar com um modelo subotimo.

---

**Anticorruption Layer (Camada Anticorrupcao)**

Quando o modelo de um contexto externo (upstream) e significativamente diferente ou de baixa qualidade, o contexto downstream cria uma camada de traducao que converte entre o modelo externo e o modelo interno. Essa camada "protege" o modelo interno de ser corrompido pela influencia do modelo externo.

A Camada Anticorrupcao e composta de:
- **Facades**: simplificam a interface do sistema externo.
- **Adapters**: traduzem entre os modelos.
- **Translators**: convertem objetos de um modelo para outro.

Exemplo: ao integrar com um sistema legado de contabilidade que usa um modelo confuso com tabelas de "MOVTO_CTBL" e campos como "CD_TP_MVTO", a equipe cria uma Camada Anticorrupcao que traduz esses dados para objetos de dominio limpos como `LancamentoContabil` com atributos claros.

A Camada Anticorrupcao e essencial para manter a saude do modelo interno. Sem ela, a "sujeira" do modelo externo vaza para dentro e contamina todo o design.

---

**Separate Ways (Caminhos Separados)**

Quando dois contextos nao precisam se integrar, ou quando o custo da integracao excede o beneficio, eles simplesmente nao se integram. Cada um segue seu caminho independente, mesmo que haja alguma sobreposicao funcional.

Isso e mais comum do que parece. Nem toda duplicacao precisa ser eliminada. As vezes, dois sistemas independentes que fazem coisas parecidas sao mais simples e robustos do que um sistema integrado complexo.

---

**Open Host Service (Servico de Host Aberto)**

Quando um contexto precisa servir muitos outros contextos, ele define um protocolo aberto (API) que qualquer consumidor pode usar. Em vez de criar integrações ponto a ponto para cada consumidor, o contexto fornecedor publica um servico generico.

O protocolo deve ser bem documentado e versionado. Mudancas devem ser feitas com cuidado para nao quebrar consumidores existentes.

---

**Published Language (Linguagem Publicada)**

Complementa o Open Host Service. Em vez de cada integracao inventar seu proprio formato de dados, os contextos usam uma linguagem de intercambio comum e bem documentada. Pode ser XML com um schema definido, JSON com um schema, ou qualquer outro formato padronizado.

Exemplos de linguagens publicadas no mundo real: SWIFT para transacoes financeiras, HL7 para dados de saude, EDI para comercio eletronico.

### Exemplos Praticos

**E-commerce com multiplos contextos**:

Contextos:
- Catalogo: gerencia produtos, categorias, descricoes.
- Vendas: gerencia pedidos, carrinho, checkout.
- Estoque: gerencia disponibilidade, reservas, localizacao.
- Faturamento: gerencia notas fiscais, impostos.
- Entrega: gerencia envios, rastreamento.

Mapa de Contextos:
- Catalogo -> Vendas: Customer/Supplier (Catalogo fornece dados de produtos para Vendas).
- Vendas -> Estoque: Customer/Supplier (Vendas informa Estoque sobre pedidos; Estoque confirma disponibilidade).
- Vendas -> Faturamento: Customer/Supplier (Vendas informa Faturamento sobre pedidos confirmados).
- Faturamento -> Sistema Legado de Contabilidade: Anticorruption Layer (Faturamento traduz dados do legado).
- Entrega -> API de Transportadora Externa: Anticorruption Layer.

**Camada Anticorrupcao na pratica**:
```
// Modelo do sistema externo (confuso)
class ExternalOrderDTO {
    String ORD_NUM;
    String CUST_CD;
    List<Map<String, Object>> LINE_ITEMS;
}

// Tradutor na Camada Anticorrupcao
class TradutorDePedido {
    Pedido traduzir(ExternalOrderDTO dto) {
        ClienteId clienteId = new ClienteId(dto.CUST_CD);
        List<ItemDePedido> itens = dto.LINE_ITEMS.stream()
            .map(this::traduzirItem)
            .collect(toList());
        return PedidoFactory.criar(clienteId, itens);
    }
}

// Modelo interno (limpo)
class Pedido { ... }  // Modelo de dominio puro
```

### Regras e Diretrizes

1. Sempre identifique os Bounded Contexts no seu sistema. Eles existem quer voce os reconheca ou nao.
2. Crie um Context Map e mantenha-o atualizado. Ele e uma das ferramentas estrategicas mais valiosas do DDD.
3. Use Anticorruption Layer ao integrar com sistemas legados ou de baixa qualidade. NUNCA deixe modelos externos contaminarem seu dominio.
4. Escolha o padrao de relacionamento conscientemente. Cada padrao tem trade-offs diferentes.
5. Nao tente criar um unico modelo universal. Aceite que modelos diferentes sao necessarios para contextos diferentes.
6. O tamanho ideal de um Bounded Context depende do tamanho da equipe e da complexidade do dominio. Uma equipe deve conseguir manter a Linguagem Ubiqua consistente dentro do seu contexto.

### Erros Comuns

- **Bounded Context unico para todo o sistema**: a tentacao de criar um unico modelo "corporativo" que serve a todos. Isso leva a classes monstruosas e linguagem ambigua.
- **Nao ter Anticorruption Layer**: deixar que o modelo de um sistema externo (especialmente legado) dite a estrutura interna do seu dominio.
- **Context Map inexistente**: nao ter uma visao clara de como os contextos se relacionam, levando a integrações ad hoc e frageis.
- **Confundir Bounded Context com microservico**: um Bounded Context e uma fronteira de modelo, nao de deploy. Um microservico pode conter varios contextos (ruim) ou um contexto pode abranger varios servicos (possivel).
- **Shared Kernel grande demais**: compartilhar demais entre contextos, criando acoplamento excessivo.

### Principais Licoes do Capitulo

A integridade do modelo e mantida nao tentando criar um modelo unico para tudo, mas reconhecendo que modelos diferentes sao necessarios para contextos diferentes e gerenciando cuidadosamente os relacionamentos entre esses contextos. O Context Map e a Anticorruption Layer sao ferramentas essenciais para qualquer sistema de escala significativa.

---

## Capitulo 15: Distillation (Destilacao)

### Conceitos Centrais

**Destilacao** e o processo de identificar e isolar o que e mais essencial no dominio -- o Core Domain (Dominio Central) -- separando-o de tudo que e importante mas nao essencial.

Evans argumenta que nem todas as partes de um sistema de software tem a mesma importancia. Algumas partes sao o que diferencia o negocio da concorrencia, o que gera valor unico. Outras partes sao necessarias mas genericas -- qualquer empresa precisa delas, mas elas nao sao um diferencial competitivo. O DDD recomenda investir o melhor talento e o maior esforco de design no Core Domain.

---

**Core Domain (Dominio Central)**

O Core Domain e a parte do sistema que representa a competencia central do negocio. E onde as regras mais complexas e valiosas vivem. E onde breakthroughs no modelo geram o maior retorno.

Identificar o Core Domain requer entender o negocio profundamente. "O que nos diferencia?" "O que fazemos melhor que os concorrentes?" "Que parte do sistema, se fosse perfeita, geraria mais valor?" As respostas a essas perguntas apontam para o Core Domain.

O Core Domain deve receber:
- Os melhores desenvolvedores.
- O design mais cuidadoso.
- A maior atencao em refatoracao e evolucao.
- A maior cobertura de testes.

---

**Generic Subdomains (Subdominios Genericos)**

Partes do sistema que sao necessarias mas nao sao diferenciais. Exemplos: autenticacao, autorizacao, notificacoes, geracao de relatorios, auditoria. Esses subdominios podem ser:
- Comprados como produtos prontos.
- Implementados com frameworks existentes.
- Desenvolvidos internamente, mas sem o mesmo nivel de investimento do Core Domain.

A decisao "build vs. buy" e especialmente relevante para Generic Subdomains. Se nao e o seu diferencial competitivo, por que investir os melhores desenvolvedores nisso?

---

**Domain Vision Statement (Declaracao de Visao do Dominio)**

Um documento curto (uma pagina ou menos) que descreve o Core Domain e o valor que ele gera. Serve como bussola para decisoes de design: quando em duvida, consulte a Domain Vision Statement para lembrar o que e mais importante.

---

**Highlighted Core (Nucleo Destacado)**

Uma forma mais detalhada de comunicar o Core Domain. Pode ser:
- Um documento de poucas paginas que descreve os elementos centrais do modelo e como eles interagem.
- Anotacoes no codigo que marcam quais classes e modulos fazem parte do Core Domain.

O Highlighted Core nao substitui o codigo -- ele complementa, fornecendo uma visao de alto nivel que ajuda novos membros da equipe a entender rapidamente o que e mais importante.

---

**Cohesive Mechanisms (Mecanismos Coesos)**

Quando uma parte do dominio envolve um algoritmo ou mecanismo complexo que pode ser encapsulado em uma unidade separada, Evans recomenda extraí-lo como um Cohesive Mechanism. Isso simplifica o modelo principal, que pode se referir ao mecanismo sem precisar entender seus detalhes internos.

Exemplo: um motor de calculo de impostos e complexo, mas pode ser encapsulado em um mecanismo coeso que recebe dados e retorna o imposto calculado. O modelo de dominio nao precisa saber como o calculo e feito -- apenas que o mecanismo existe e como usa-lo.

---

**Abstract Core (Nucleo Abstrato)**

Quando o Core Domain e grande e complexo, Evans sugere criar uma versao abstrata que captura os conceitos mais fundamentais sem os detalhes de implementacao. Isso e feito com interfaces e classes abstratas que definem o vocabulario central, enquanto as implementacoes concretas vivem em modulos separados.

### Exemplos Praticos

**E-commerce -- identificando o Core Domain**:

Para uma empresa generica de e-commerce, o Core Domain pode ser o motor de recomendacao personalizada, a estrategia de precificacao dinamica, ou o algoritmo de otimizacao de estoque -- dependendo de onde esta a vantagem competitiva.

Subdominios genericos: gerenciamento de usuarios, processamento de pagamentos (use um gateway), envio de emails, geracao de PDFs.

**Declaracao de Visao do Dominio para um sistema de seguros**:
"Nosso Core Domain e o motor de avaliacao de risco e precificacao de apolices, que combina dados atuariais, historico de sinistros e perfil do segurado para gerar premios competitivos e rentaveis. Esse e o diferencial que nos permite oferecer precos melhores que a concorrencia mantendo a saude financeira da operacao."

### Regras e Diretrizes

1. Identifique o Core Domain cedo e revisit-e regularmente.
2. Coloque os melhores desenvolvedores no Core Domain.
3. Considere comprar ou usar solucoes prontas para Generic Subdomains.
4. Escreva uma Domain Vision Statement e compartilhe-a com toda a equipe.
5. Use Highlighted Core para facilitar a navegacao do modelo.
6. Extraia mecanismos complexos que obscurecem o Core Domain.

### Erros Comuns

- **Tratar tudo como Core Domain**: se tudo e igualmente importante, nada e importante. Priorize.
- **Investir no generico**: gastar meses construindo um sistema de autenticacao customizado quando existem solucoes prontas.
- **Core Domain com os piores desenvolvedores**: por ironia, muitas organizacoes colocam os melhores desenvolvedores em problemas de infraestrutura e deixam o Core Domain com juniores.
- **Nao comunicar o Core Domain**: a equipe nao sabe o que e mais importante, entao toma decisoes desalinhadas.

### Principais Licoes do Capitulo

Destilacao e sobre foco. Em um sistema grande, nao e possivel (nem desejavel) investir o mesmo esforco em todas as partes. Identifique o Core Domain, proteja-o, invista nele, e use solucoes pragmaticas para o resto.

---

## Capitulo 16: Large-Scale Structure (Estrutura em Grande Escala)

### Conceitos Centrais

Quando um sistema cresce alem de um certo tamanho, mesmo com Bounded Contexts bem definidos, pode ser dificil entender como as pecas se encaixam. Evans apresenta padroes para dar estrutura ao sistema como um todo, sem impor uma rigidez que impeca a evolucao.

---

**Evolving Order (Ordem em Evolucao)**

A estrutura de grande escala deve emergir e evoluir, nao ser imposta no inicio. Uma arquitetura rigida definida antes de o dominio ser bem compreendido e quase certamente errada. A equipe deve comecar com pouca estrutura e adicionar mais conforme o entendimento cresce.

O principio e: tanta estrutura quanto necessario, tao pouca quanto possivel.

---

**System Metaphor (Metafora do Sistema)**

Uma metafora que ajuda a equipe a pensar sobre o sistema como um todo. Por exemplo, "o sistema e como uma fabrica com linhas de montagem" ou "o sistema e como um ecossistema com organismos que interagem".

Metaforas podem ser poderosas quando funcionam, mas perigosas quando sao forcadas. Uma boa metafora ilumina; uma ruim confunde.

---

**Responsibility Layers (Camadas de Responsabilidade)**

Organizar o sistema em camadas onde cada camada tem uma responsabilidade conceitual ampla. Diferente das camadas tecnicas (UI, Application, Domain, Infrastructure), as Responsibility Layers sao camadas de dominio.

Exemplo em um sistema de logistica:
- **Camada de Operacao**: lida com eventos individuais (carga recebida, carga despachada).
- **Camada de Capacidade**: gerencia recursos e restricoes (capacidade dos navios, disponibilidade de containers).
- **Camada de Decisao/Politica**: define regras e estrategias de alto nivel (quais rotas priorizar, como alocar capacidade).
- **Camada de Compromisso**: gerencia acordos de longo prazo (contratos, SLAs).
- **Camada Potencial**: define o que a organizacao e capaz de fazer (frota de navios, rede de portos).

Cada camada depende das camadas abaixo (Decisao depende de Capacidade que depende de Operacao) mas nao o contrario.

---

**Knowledge Level (Nivel de Conhecimento)**

Quando um sistema precisa ser altamente configuravel -- quando as regras de negocio mudam frequentemente e precisam ser ajustadas sem recompilar -- Evans sugere separar o modelo em dois niveis:

- **Nivel Operacional**: os objetos que representam as entidades e processos do dia a dia (Pedidos, Clientes, Transacoes).
- **Nivel de Conhecimento**: os objetos que definem as regras e estruturas que governam o nivel operacional (tipos de produto, politicas de desconto, configuracoes de workflow).

O Nivel de Conhecimento e essencialmente metadados que controlam o comportamento do nivel operacional. Isso permite que regras de negocio sejam alteradas por configuracao em vez de por codigo.

---

**Pluggable Component Framework (Framework de Componentes Plugaveis)**

O nivel mais avancado de estrutura: um framework que permite que componentes (implementacoes de Bounded Contexts) sejam plugados e substituidos. Isso requer interfaces muito bem definidas e estáveis.

Evans adverte que este padrao e ambicioso e deve ser usado com cautela. O custo de criar e manter um framework plugavel e alto, e muitos projetos nao precisam desse nivel de flexibilidade.

### Exemplos Praticos

**Responsibility Layers em um sistema bancario**:
- Potencial: infraestrutura de agencias, ATMs, internet banking.
- Compromisso: contas abertas, emprestimos concedidos.
- Politica: regras de juros, limites de credito, politicas de compliance.
- Operacao: transacoes diarias, transferencias, pagamentos.

Cada camada usa servicos das camadas abaixo. Uma transferencia (Operacao) obedece as regras de limite (Politica), dentro do escopo de uma conta (Compromisso), usando a infraestrutura disponivel (Potencial).

**Knowledge Level em um sistema de RH**:
- Nivel de Conhecimento: define tipos de cargo, faixas salariais, politicas de ferias, regras de promocao.
- Nivel Operacional: colaboradores individuais, suas posicoes, seus historicos, seus pedidos de ferias.

O departamento de RH pode mudar as politicas de ferias (Knowledge Level) sem alterar o codigo que processa os pedidos de ferias (Operational Level).

### Regras e Diretrizes

1. Deixe a estrutura de grande escala emergir. Nao force uma arquitetura rigida no inicio.
2. Use a estrutura que fizer sentido para o dominio. Nem todo sistema precisa de Responsibility Layers ou Knowledge Level.
3. A estrutura de grande escala deve simplificar o entendimento, nao adicionar complexidade.
4. Revise a estrutura periodicamente. O que fazia sentido com 5 contextos pode nao fazer sentido com 20.
5. Pluggable Component Framework so vale a pena para sistemas muito grandes e com necessidade real de flexibilidade.

### Erros Comuns

- **Arquitetura astronauta**: definir uma estrutura de grande escala grandiosa no inicio do projeto, antes de entender o dominio.
- **Estrutura rigida**: uma estrutura que nao pode evoluir conforme o entendimento do dominio melhora.
- **Metafora forcada**: insistir em uma metafora de sistema que nao se encaixa naturalmente no dominio.
- **Knowledge Level desnecessario**: adicionar um nivel de metadados quando as regras raramente mudam e poderiam estar no codigo.

### Principais Licoes do Capitulo

A estrutura de grande escala e uma ferramenta de comunicacao e organizacao que ajuda equipes a entender sistemas complexos. Ela deve emergir do dominio, evoluir com o entendimento, e ser tao simples quanto possivel. Nem todo sistema precisa de estrutura de grande escala -- use-a quando a complexidade justificar.

---

## Capitulo 17: Bringing the Strategy Together (Unindo a Estrategia)

### Conceitos Centrais

O capitulo final sintetiza todo o design estrategico em um framework coerente. Evans argumenta que os tres grandes temas estrategicos -- Integridade de Modelo (Bounded Contexts), Destilacao (Core Domain) e Estrutura de Grande Escala -- devem trabalhar juntos.

**Decisoes estrategicas sao decisoes de equipe**: o design estrategico nao e responsabilidade de um unico arquiteto. Ele deve ser discutido e decidido colaborativamente, com input de todas as equipes que serao afetadas.

**O Context Map como ferramenta central**: o Context Map e o artefato que conecta tudo. Ele mostra os Bounded Contexts, seus relacionamentos, o Core Domain, e como a estrutura de grande escala se manifesta.

**Trade-offs sao inevitaveis**: toda decisao estrategica envolve trade-offs. Shared Kernel reduz duplicacao mas cria acoplamento. Separate Ways elimina acoplamento mas cria duplicacao. Anticorruption Layer protege o modelo mas adiciona complexidade. Nao existe solucao perfeita -- existe a solucao mais adequada para o contexto atual.

**Evolucao continua**: a estrategia nao e definida uma vez e congelada. Ela evolui conforme o negocio muda, conforme a equipe cresce, conforme o entendimento do dominio aprofunda. O Context Map deve ser revisitado regularmente.

### Exemplos Praticos

Evans apresenta um cenario de uma grande empresa com multiplas equipes trabalhando em diferentes partes de um sistema de comercio internacional. Ele mostra como as decisoes estrategicas sao tomadas:

1. Identificacao dos Bounded Contexts: Booking (reserva de carga), Tracking (rastreamento), Billing (faturamento), Scheduling (programacao de viagens).
2. Definicao do Core Domain: Booking, porque e o que gera receita e diferencia a empresa.
3. Relacionamentos: Booking -> Scheduling (Customer/Supplier), Booking -> Tracking (Shared Kernel para o conceito de Carga), Billing -> Sistema Legado (Anticorruption Layer).
4. Estrutura de grande escala: Responsibility Layers (Potencial, Compromisso, Operacao) que se aplicam transversalmente aos contextos.

### Regras e Diretrizes

1. Comece com o Context Map. Ele e a fundacao de toda estrategia.
2. Identifique o Core Domain e proteja-o.
3. Escolha os padroes de relacionamento entre contextos deliberadamente.
4. Nao sobrecarregue com estrutura. Adicione estrutura de grande escala apenas quando a complexidade justificar.
5. Revise a estrategia periodicamente.
6. Envolva todas as equipes nas decisoes estrategicas.

### Erros Comuns

- **Estrategia top-down imposta**: um arquiteto define tudo sozinho e as equipes apenas obedecem. Isso raramente funciona.
- **Ausencia de estrategia**: cada equipe faz o que quer, sem coordenacao. O resultado e um sistema fragmentado e incoerente.
- **Estrategia rigida**: definir tudo no inicio e recusar-se a mudar. O negocio muda, a estrategia deve acompanhar.
- **Foco exclusivo em microservicos**: confundir design estrategico com arquitetura de deploy. Bounded Contexts e microservicos sao conceitos diferentes.

### Principais Licoes do Capitulo

O design estrategico e o que separa equipes que constroem sistemas sustentaveis de equipes que constroem Big Balls of Mud (Grandes Bolas de Lama). Ele requer visao, colaboracao, disciplina e disposicao para evoluir. Nao existe formula magica -- existe um conjunto de ferramentas (Context Map, Bounded Context, Core Domain, padroes de relacionamento, estrutura de grande escala) que, quando usadas com sabedoria, permitem que sistemas complexos cresçam de forma saudavel.

---

# Glossario DDD

**Aggregate (Agregado)**: cluster de objetos de dominio tratados como uma unidade para fins de mudanca de dados. Tem uma raiz (Entidade principal) e uma fronteira que define quais objetos fazem parte dele. Todas as invariantes devem ser satisfeitas dentro da fronteira do Agregado.

**Aggregate Root (Raiz do Agregado)**: a Entidade principal do Agregado, unico ponto de acesso externo. Responsavel por garantir as invariantes do Agregado inteiro.

**Anticorruption Layer (Camada Anticorrupcao)**: camada de traducao que protege o modelo de dominio interno de ser contaminado por modelos externos de baixa qualidade ou incompativeis.

**Application Layer (Camada de Aplicacao)**: camada que coordena tarefas da aplicacao, delegando trabalho ao dominio. Nao contem logica de negocio.

**Assertion (Assercao)**: declaracao explicita de pre-condicoes, pos-condicoes e invariantes que documenta os efeitos colaterais de operacoes.

**Bounded Context (Contexto Delimitado)**: fronteira explicita dentro da qual um modelo de dominio e valido. Dentro do contexto, cada termo tem um significado unico e preciso.

**Closure of Operations (Fecho de Operacoes)**: principio de design onde uma operacao retorna o mesmo tipo dos operandos, permitindo composicao natural.

**Cohesive Mechanism (Mecanismo Coeso)**: algoritmo ou mecanismo complexo extraido do modelo principal e encapsulado em uma unidade separada.

**Conceptual Contour (Contorno Conceitual)**: as fronteiras naturais de um conceito de dominio que devem guiar a decomposicao de classes e metodos.

**Conformist (Conformista)**: padrao de relacionamento entre contextos onde o downstream aceita o modelo do upstream sem influencia-lo.

**Context Map (Mapa de Contextos)**: representacao visual ou documental de todos os Bounded Contexts e seus relacionamentos.

**Core Domain (Dominio Central)**: a parte mais valiosa e diferenciadora do sistema, onde as regras de negocio mais complexas e importantes vivem.

**Customer/Supplier (Cliente/Fornecedor)**: relacao entre contextos onde o upstream (fornecedor) se compromete a atender as necessidades do downstream (cliente).

**Domain Event (Evento de Dominio)**: registro de algo significativo que aconteceu no dominio. Embora Evans mencione eventos brevemente, este conceito ganhou destaque na comunidade DDD apos o livro.

**Domain Layer (Camada de Dominio)**: camada que contem toda a logica de negocio, incluindo Entidades, Objetos de Valor, Servicos de Dominio, Agregados, Fabricas e Repositorios.

**Domain Vision Statement (Declaracao de Visao do Dominio)**: documento curto que descreve o Core Domain e o valor que ele gera para o negocio.

**Entity (Entidade)**: objeto definido por sua identidade unica, que persiste ao longo do tempo. Igualdade e determinada pela identidade, nao pelos atributos.

**Evolving Order (Ordem em Evolucao)**: principio de que a estrutura de grande escala deve emergir e evoluir, nao ser imposta prematuramente.

**Factory (Fabrica)**: objeto ou metodo responsavel por criar objetos de dominio complexos em um estado valido e consistente.

**Generic Subdomain (Subdominio Generico)**: parte do sistema necessaria mas nao diferenciadora. Candidata a solucoes prontas ou investimento reduzido de design.

**Hands-On Modelers (Modeladores Praticos)**: principio de que as pessoas que modelam o dominio devem ser as mesmas que escrevem o codigo.

**Highlighted Core (Nucleo Destacado)**: documentacao concisa que identifica os elementos mais importantes do Core Domain.

**Infrastructure Layer (Camada de Infraestrutura)**: camada que fornece capacidades tecnicas (persistencia, mensageria, etc.) implementando interfaces definidas pelas camadas superiores.

**Intention-Revealing Interface (Interface que Revela a Intencao)**: design onde nomes de classes e metodos comunicam o que fazem, nao como fazem.

**Invariant (Invariante)**: regra de consistencia que deve ser sempre verdadeira para um Agregado ou objeto.

**Knowledge Crunching (Trituracao de Conhecimento)**: processo iterativo de extrair, organizar e refinar conhecimento do dominio em colaboracao com especialistas.

**Knowledge Level (Nivel de Conhecimento)**: separacao do modelo em nivel operacional (dados e processos do dia a dia) e nivel de conhecimento (regras e configuracoes que governam o operacional).

**Layered Architecture (Arquitetura em Camadas)**: organizacao do codigo em camadas (UI, Application, Domain, Infrastructure) com dependencias unidirecionais.

**Model-Driven Design (Design Dirigido pelo Modelo)**: principio de que o modelo de dominio e o codigo devem ser a mesma coisa, sem camada de traducao entre eles.

**Module (Modulo)**: agrupamento de elementos do modelo com alta coesao interna e baixo acoplamento externo. Deve refletir conceitos do dominio.

**Open Host Service (Servico de Host Aberto)**: protocolo aberto e documentado que um contexto fornecedor disponibiliza para multiplos consumidores.

**Pluggable Component Framework (Framework de Componentes Plugaveis)**: estrutura que permite que implementacoes de Bounded Contexts sejam substituidas sem afetar o resto do sistema.

**Published Language (Linguagem Publicada)**: formato de dados padronizado e documentado usado para intercambio entre contextos.

**Repository (Repositorio)**: objeto que encapsula acesso a dados, fornecendo a ilusao de uma colecao em memoria. A interface pertence ao dominio; a implementacao pertence a infraestrutura.

**Responsibility Layers (Camadas de Responsabilidade)**: organizacao de grande escala do sistema em camadas conceituais baseadas em responsabilidades de dominio.

**Separate Ways (Caminhos Separados)**: decisao deliberada de nao integrar dois contextos, aceitando alguma duplicacao em troca de independencia total.

**Service (Servico)**: operacao sem estado que realiza algo significativo no dominio e nao pertence naturalmente a nenhuma Entidade ou Objeto de Valor.

**Shared Kernel (Nucleo Compartilhado)**: subconjunto do modelo compartilhado entre dois Bounded Contexts, mantido em comum por acordo entre as equipes.

**Side-Effect-Free Function (Funcao Livre de Efeitos Colaterais)**: operacao que retorna um resultado sem modificar estado observavel. Fundamental para Objetos de Valor.

**Specification (Especificacao)**: objeto que encapsula uma regra de negocio ou criterio de selecao, com metodo que verifica se um objeto satisfaz o criterio.

**Standalone Class (Classe Autonoma)**: classe com minimas dependencias que pode ser entendida em isolamento.

**Strategy (Estrategia)**: padrao de design que encapsula algoritmos intercambiaveis. No DDD, usado para modelar politicas de dominio.

**Supple Design (Design Flexivel)**: design que e facil de entender, usar e evoluir, resultado da aplicacao de principios como Intention-Revealing Interfaces, Side-Effect-Free Functions e outros.

**System Metaphor (Metafora do Sistema)**: analogia ou metafora que ajuda a equipe a pensar sobre o sistema como um todo.

**Ubiquitous Language (Linguagem Ubiqua)**: vocabulario rigoroso e compartilhado usado por toda a equipe (desenvolvedores, especialistas, gerentes) em todas as formas de comunicacao (codigo, conversas, documentos).

**Value Object (Objeto de Valor)**: objeto imutavel definido completamente por seus atributos, sem identidade propria. Dois Objetos de Valor com os mesmos atributos sao iguais.

---

# Principais Licoes

**1. O dominio e o coracao do software.** A complexidade mais significativa da maioria dos projetos nao esta na tecnologia, esta no negocio. Investir em entender o dominio profundamente e a decisao mais impactante que uma equipe pode tomar.

**2. A Linguagem Ubiqua e inegociavel.** Sem uma linguagem compartilhada entre desenvolvedores e especialistas do dominio, o projeto esta condenado a mal-entendidos, retrabalho e entropia. A linguagem deve permear o codigo, as conversas, os documentos -- tudo.

**3. O modelo e o codigo devem ser a mesma coisa.** Modelos "de analise" separados do codigo sao documentos mortos. O codigo da camada de dominio E o modelo. Se voce quer entender o modelo, leia o codigo.

**4. Isole o dominio.** A logica de negocio deve viver em uma camada dedicada, protegida de preocupacoes tecnicas. Sem esse isolamento, os padroes do DDD nao podem ser aplicados efetivamente.

**5. Prefira Objetos de Valor a Entidades.** Imutabilidade simplifica tudo. Use Entidades apenas quando a identidade e necessaria. O restante deve ser Objetos de Valor.

**6. Agregados sao fronteiras de consistencia.** Mantenha Agregados pequenos. Uma transacao por Agregado. Consistencia eventual entre Agregados.

**7. Refatore em direcao a insights mais profundos.** O modelo inicial nunca e o melhor modelo. Refatoracao continua, motivada por entendimento cada vez mais profundo do dominio, e o caminho para a excelencia.

**8. Bounded Contexts sao inevitaveis.** Em qualquer sistema de escala razoavel, nao existe um modelo unico que serve para tudo. Reconheca os contextos, defina suas fronteiras, e gerencie seus relacionamentos conscientemente.

**9. Proteja seu Core Domain.** Identifique o que e mais valioso e invista desproporcionalmente nessa parte. Use solucoes prontas para o generico. Coloque os melhores desenvolvedores no Core Domain.

**10. O design estrategico e tao importante quanto o tatico.** Patterns de Entidade e Objeto de Valor nao salvam um projeto se os Bounded Contexts estao errados, se o Core Domain nao esta identificado, ou se as integrações entre contextos sao frageis.

**11. DDD e uma pratica, nao um destino.** Nao existe um momento em que o modelo esta "pronto". Knowledge crunching, refinamento de linguagem, refatoracao de modelo e revisao de estrategia sao atividades permanentes, nao fases.

**12. Colaboracao e a chave.** DDD funciona quando desenvolvedores e especialistas do dominio trabalham juntos, continuamente, com respeito mutuo e disposicao para aprender. Sem essa colaboracao, o DDD se reduz a um conjunto de padroes tecnicos sem alma.
