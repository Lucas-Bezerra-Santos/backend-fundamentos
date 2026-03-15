# Clean Code: A Handbook of Agile Software Craftsmanship

**Autor:** Robert C. Martin ("Uncle Bob")
**Ano:** 2008
**Editora:** Prentice Hall

## Por que este livro importa?

"Clean Code" e considerado uma das obras fundamentais da engenharia de software moderna. O livro parte de uma premissa simples, mas poderosa: **codigo e lido muito mais vezes do que e escrito**. Portanto, escrever codigo limpo nao e um luxo ou vaidade -- e uma responsabilidade profissional. O custo de manter codigo ruim cresce exponencialmente ao longo do tempo, e equipes inteiras podem ser paralisadas por bases de codigo que ninguem consegue entender ou modificar com seguranca. Este livro ensina principios, padroes e praticas que transformam codigo bagunçado em codigo profissional.

---

## Capitulo 1 -- Codigo Limpo (Clean Code)

### Conceitos-chave

O capitulo abre com uma discussao sobre o **custo do codigo ruim**. Martin argumenta que a produtividade de equipes de desenvolvimento tende a cair drasticamente ao longo do tempo quando o codigo nao e mantido limpo. Ele chama isso de "pantano" (wading) -- a sensacao de que cada mudanca simples exige esforco desproporcional.

Martin entrevista varios programadores renomados pedindo suas definicoes de "codigo limpo":

- **Bjarne Stroustrup** (criador do C++): Codigo limpo e elegante e eficiente. A logica deve ser direta para dificultar bugs. Dependencias devem ser minimas. Tratamento de erros deve ser completo. O desempenho deve ser proximo do otimo.

- **Grady Booch** (autor de "Object-Oriented Analysis and Design"): Codigo limpo e simples e direto. Le-se como prosa bem escrita. Nunca obscurece a intencao do designer e esta cheio de abstracoes nitidas e linhas de controle objetivas.

- **Dave Thomas** (fundador da OTI): Codigo limpo pode ser lido e melhorado por outro desenvolvedor alem do autor original. Possui testes unitarios e de aceitacao. Tem nomes significativos. Oferece uma unica maneira (e nao varias) de fazer cada coisa. Tem dependencias minimas, explicitamente definidas, e fornece uma API clara e minima.

- **Michael Feathers** (autor de "Working Effectively with Legacy Code"): Codigo limpo parece ter sido escrito por alguem que se importa. Nao ha nada obvio que voce possa fazer para melhora-lo.

- **Ron Jeffries** (autor de "Extreme Programming Installed"): Codigo limpo passa em todos os testes, nao contem duplicacao, expressa todas as ideias de design do sistema e minimiza o numero de entidades (classes, metodos, funcoes).

### A Regra do Escoteiro

Martin introduz a **Regra do Escoteiro aplicada ao codigo**: "Deixe o acampamento mais limpo do que voce encontrou." Isso significa que toda vez que voce tocar em um arquivo, deve deixa-lo um pouco melhor do que estava. Renomear uma variavel mal nomeada, quebrar uma funcao grande, eliminar um pequeno trecho duplicado. Pequenas melhorias continuas evitam a degradacao do codigo.

### Licoes principais

- Codigo ruim gera mais codigo ruim. A "podridao" se espalha.
- Escrever codigo limpo e uma questao de disciplina profissional.
- Nao existe "depois eu limpo". O "depois" nunca chega.
- A responsabilidade pelo codigo limpo e do programador, nao do gerente ou do prazo.

---

## Capitulo 2 -- Nomes Significativos (Meaningful Names)

### Conceitos-chave

A escolha de nomes e uma das atividades mais frequentes e mais impactantes na programacao. Um bom nome elimina a necessidade de comentarios e torna o codigo auto-documentado.

### Regras detalhadas

**1. Use nomes que revelem a intencao (Use Intention-Revealing Names)**

O nome de uma variavel, funcao ou classe deve responder: por que existe, o que faz e como e usada. Se um nome precisa de comentario, ele nao revela a intencao.

```java
// Ruim
int d; // dias decorridos

// Bom
int diasDecorridos;
int diasDesdeModificacao;
int diasDesdeACriacao;
```

**2. Evite desinformacao (Avoid Disinformation)**

Nao use nomes que possam confundir. Nao chame uma variavel de `listaDeProdutos` se ela nao for realmente uma `List`. Nao use nomes que diferem em detalhes sutis, como `controladorParaManipulacaoEficienteDeStrings` e `controladorParaManipulacaoDeStrings`.

**3. Faca distincoes significativas (Make Meaningful Distinctions)**

Nao crie nomes apenas para satisfazer o compilador. Se voce tem `Produto`, `ProdutoInfo` e `ProdutoDados`, a distincao e inexistente para quem le o codigo.

```java
// Ruim -- qual a diferenca?
getContaAtiva()
getContasAtivas()
getInformacaoContaAtiva()

// Bom -- distincao clara
getContaAtivaPorId(int id)
listarContasAtivas()
obterResumoContaAtiva(int id)
```

**4. Use nomes pronunciaveis (Use Pronounceable Names)**

Humanos sao bons com palavras. Se voce nao consegue pronunciar um nome, nao consegue discuti-lo sem parecer ridiculo.

```java
// Ruim
Date genymdhms; // generation year, month, day, hour, minute, second

// Bom
Date dataGeracaoTimestamp;
```

**5. Use nomes pesquisaveis (Use Searchable Names)**

Nomes de uma unica letra e constantes numericas sao dificeis de localizar no codigo. `MAX_ALUNOS_POR_TURMA` e muito mais pesquisavel que o numero `30`.

**6. Evite codificacoes (Avoid Encodings)**

Nao use notacao hungara, prefixos de membro (`m_`) ou prefixos de interface (`I`). IDEs modernas tornam essas convencoes desnecessarias e elas adicionam ruido visual.

**7. Nomes de classes devem ser substantivos**

Classes e objetos devem ter nomes que sao substantivos ou frases substantivas: `Cliente`, `PaginaWiki`, `Conta`, `AnalisadorDeEnderecos`. Evite palavras genericas como `Manager`, `Processor`, `Data`, `Info`.

**8. Nomes de metodos devem ser verbos**

Metodos devem ter nomes que sao verbos ou frases verbais: `salvar`, `deletarPagina`, `enviarMensagem`. Accessors, mutators e predicates devem seguir o padrao javabean: `getValor`, `setValor`, `estaVazio`.

**9. Escolha uma palavra por conceito (Pick One Word per Concept)**

Escolha uma palavra para cada conceito abstrato e mantenha-a. E confuso ter `buscar`, `recuperar` e `obter` como metodos equivalentes em classes diferentes. Escolha um e use consistentemente.

**10. Use nomes do dominio do problema e do dominio da solucao**

Use termos tecnicos quando o conceito e tecnico (`Observer`, `Visitor`, `Queue`, `Factory`). Use termos do dominio do negocio quando o conceito e do negocio (`ContaCorrente`, `Pedido`, `NotaFiscal`).

### Licoes principais

- Nomes sao a ferramenta de comunicacao mais fundamental do codigo.
- Gaste tempo escolhendo bons nomes. O retorno compensa.
- Nao tenha medo de renomear. IDEs modernas facilitam refatoracao.

---

## Capitulo 3 -- Funcoes (Functions)

### Conceitos-chave

Funcoes sao os blocos de construcao fundamentais de qualquer programa. Este capitulo estabelece regras claras para escrever funcoes que sejam faceis de ler, entender e manter.

### Regras detalhadas

**1. Funcoes devem ser pequenas (Small!)**

A primeira regra e que funcoes devem ser pequenas. A segunda regra e que **devem ser ainda menores**. Martin sugere que funcoes idealmente nao devem ter mais que 20 linhas, e muitas vezes podem ter menos de 10. Blocos dentro de `if`, `else` e `while` devem ter idealmente uma unica linha -- geralmente uma chamada de funcao.

**2. Faca apenas uma coisa (Do One Thing)**

Uma funcao deve fazer uma coisa, faze-la bem e faze-la unicamente. Mas como definir "uma coisa"? Martin oferece este teste: se voce consegue extrair outra funcao com um nome que nao e meramente uma reafirmacao da implementacao, entao a funcao original faz mais de uma coisa.

```java
// Faz varias coisas
public void processarPagamento(Pedido pedido) {
    validarPedido(pedido);
    calcularDesconto(pedido);
    cobrarCartao(pedido);
    enviarEmailConfirmacao(pedido);
    atualizarEstoque(pedido);
}

// Cada funcao faz uma coisa
public void validarPedido(Pedido pedido) { ... }
public void calcularDesconto(Pedido pedido) { ... }
public void cobrarCartao(Pedido pedido) { ... }
```

**3. Um nivel de abstracao por funcao (One Level of Abstraction per Function)**

Misturar niveis de abstracao dentro de uma funcao e confuso. Detalhes de alto nivel (como `obterPaginaHtml()`) nao devem aparecer ao lado de detalhes de baixo nivel (como `String caminhoCompleto = caminho + "/" + arquivo`).

**4. A Regra Stepdown (Reading Code from Top to Bottom)**

O codigo deve ser lido como uma narrativa de cima para baixo. Cada funcao deve ser seguida pelas funcoes do proximo nivel de abstracao. Isso permite ler o programa descendo um nivel de abstracao por vez.

**5. Argumentos de funcao (Function Arguments)**

O numero ideal de argumentos e zero (niladico). Depois vem um (monadico), dois (diadico). Tres argumentos (triadico) devem ser evitados. Mais de tres exigem justificativa muito especial e nao devem ser usados de qualquer forma.

Razoes:
- Argumentos dificultam o entendimento. Cada argumento forca o leitor a interpreta-lo.
- Argumentos dificultam testes. Cada argumento multiplica os casos de teste.
- Argumentos de saida sao confusos. Use retorno em vez de modificar argumentos.

```java
// Ruim -- muitos argumentos
Circulo criarCirculo(double x, double y, double raio, String cor, boolean preenchido);

// Bom -- encapsular em objeto
Circulo criarCirculo(Ponto centro, double raio, EstiloVisual estilo);
```

**6. Sem efeitos colaterais (No Side Effects)**

Se a funcao promete fazer uma coisa, ela nao deve fazer coisas escondidas. Uma funcao `verificarSenha` que tambem inicializa a sessao tem um efeito colateral perigoso.

**7. Separacao comando-consulta (Command Query Separation)**

Uma funcao deve ou fazer algo (comando) ou responder algo (consulta), mas nao ambos.

```java
// Ruim -- faz e responde ao mesmo tempo
if (definir("usuario", "lucas")) { ... }

// Bom -- separado
if (atributoExiste("usuario")) {
    definirAtributo("usuario", "lucas");
}
```

**8. Prefira excecoes a codigos de erro**

Codigos de retorno de erro forcam o chamador a lidar com o erro imediatamente, levando a estruturas aninhadas. Excecoes permitem separar o codigo feliz do tratamento de erros.

```java
// Ruim -- codigos de erro
if (deletarPagina(pagina) == E_OK) {
    if (registrar.log("pagina deletada") == E_OK) {
        // ...
    }
}

// Bom -- excecoes
try {
    deletarPagina(pagina);
    registrar.log("pagina deletada");
} catch (Exception e) {
    logger.error(e.getMessage());
}
```

**9. DRY -- Nao se repita (Don't Repeat Yourself)**

Duplicacao e a raiz de todo mal no software. Toda vez que um algoritmo e repetido, qualquer mudanca exige alteracao em multiplos lugares, multiplicando a chance de erros.

### Licoes principais

- Funcoes pequenas, com nomes descritivos e poucos argumentos sao o alicerce do codigo limpo.
- Extraia funcoes sem medo. Funcoes pequenas com bons nomes substituem comentarios.
- Efeitos colaterais sao bugs esperando para acontecer.

---

## Capitulo 4 -- Comentarios (Comments)

### Conceitos-chave

Martin tem uma posicao forte: **comentarios sao, na melhor das hipoteses, um mal necessario**. O uso adequado de comentarios e para compensar nosso fracasso em nos expressar no codigo. Se o codigo precisar de comentarios, provavelmente pode ser reescrito de forma mais clara.

### Comentarios bons (aceitaveis)

**1. Comentarios legais:** Avisos de copyright e licenca no topo dos arquivos.

**2. Comentarios informativos:** Quando fornecem informacao util que nao pode ser expressa no codigo.

```java
// Formato esperado: dd/MM/yyyy
Pattern formatoData = Pattern.compile("\\d{2}/\\d{2}/\\d{4}");
```

**3. Explicacao de intencao:** Quando o "por que" nao e obvio, um comentario pode explicar a decisao por tras da implementacao.

```java
// Usamos TreeMap em vez de HashMap porque a ordem de insercao
// importa para o relatorio mensal que depende deste dado
Map<String, Venda> vendasMensais = new TreeMap<>();
```

**4. Alerta de consequencias:** Avisos sobre consequencias que nao sao obvias.

```java
// Este teste leva 30 minutos para rodar.
// Execute apenas antes de fazer merge para main.
@Test
public void testeDeDesempenhoCompleto() { ... }
```

**5. Comentarios TODO:** Tarefas que precisam ser feitas mas nao podem ser feitas agora. Devem ser temporarios e revisados regularmente.

**6. Javadocs em APIs publicas:** Documentacao de APIs publicas e valiosa e necessaria.

### Comentarios ruins (a evitar)

**1. Murmuro:** Comentarios escritos sem cuidado, que nao comunicam nada util.

**2. Comentarios redundantes:** Dizem exatamente o que o codigo ja diz.

```java
// Retorna o dia do mes
public int getDiaDoMes() {
    return diaDoMes;
}
```

**3. Comentarios enganosos:** Comentarios que nao refletem com precisao o que o codigo faz. Sao piores que a ausencia de comentarios.

**4. Comentarios obrigatorios:** Regras que exigem Javadoc para toda funcao ou variavel geram ruido sem valor.

**5. Comentarios de jornal (changelog):** Logs de alteracao no inicio do arquivo. Sistemas de controle de versao (Git) fazem isso melhor.

**6. Codigo comentado:** Nunca deixe codigo comentado. Ninguem tem coragem de deletar porque pensa "talvez seja importante". Use controle de versao -- o historico esta no Git.

**7. Comentarios de fechamento:** Comentarios no final de blocos longos (`} // fim do while`) indicam que o bloco e grande demais e deve ser quebrado em funcoes menores.

### Licoes principais

- O melhor comentario e aquele que voce conseguiu nao escrever, porque o codigo se explica sozinho.
- Codigo comentado (desativado) deve ser deletado, nao mantido.
- Comentarios mentem com o tempo -- o codigo muda, mas os comentarios frequentemente nao sao atualizados.

---

## Capitulo 5 -- Formatacao (Formatting)

### Conceitos-chave

Formatacao de codigo nao e uma questao cosmetica -- e uma questao de comunicacao. E a comunicacao e a principal atividade de um desenvolvedor profissional. A formatacao que voce define hoje afeta a manutenibilidade do codigo no futuro.

### Regras detalhadas

**1. Formatacao vertical**

- **Tamanho do arquivo:** Arquivos menores sao geralmente mais faceis de entender. Martin mostra que projetos bem-sucedidos como JUnit e FitNesse tem a maioria dos arquivos com menos de 200 linhas, sendo que poucos ultrapassam 500.

- **Metafora do jornal:** Um arquivo de codigo deve ser como um artigo de jornal. O nome (titulo) deve ser simples mas descritivo. As partes superiores devem fornecer conceitos e algoritmos de alto nivel. Os detalhes devem aumentar conforme descemos no arquivo.

- **Distancia vertical:** Conceitos relacionados devem estar proximos verticalmente. Variaveis devem ser declaradas o mais proximo possivel de onde sao usadas. Variaveis de instancia devem ser declaradas no topo da classe. Funcoes dependentes devem estar proximas, com a funcao que chama acima da funcao chamada.

- **Linhas em branco:** Use linhas em branco para separar conceitos distintos (entre metodos, entre blocos logicos dentro de um metodo).

**2. Formatacao horizontal**

- **Tamanho da linha:** Martin sugere um limite de 120 caracteres. Linhas nao devem exigir rolagem horizontal.

- **Espacamento horizontal:** Use espacos para associar coisas relacionadas e dissociar coisas pouco relacionadas. Operadores de atribuicao devem ter espacos. Argumentos de funcao devem ser separados por espaco apos virgula.

- **Indentacao:** A indentacao torna a hierarquia do codigo visivel. Nunca quebre a indentacao, mesmo para funcoes curtas ou `if` de uma linha.

**3. Regras de equipe**

Uma equipe de desenvolvimento deve concordar com um unico estilo de formatacao e todos devem segui-lo. Consistencia e mais importante que preferencia pessoal. Use ferramentas de formatacao automatica (linters, formatters) para garantir uniformidade.

### Licoes principais

- Codigo bem formatado comunica profissionalismo e cuidado.
- A consistencia dentro de um projeto e mais importante que qualquer regra individual.
- Configure ferramentas automaticas e nao perca tempo com formatacao manual.

---

## Capitulo 6 -- Objetos e Estruturas de Dados (Objects and Data Structures)

### Conceitos-chave

Ha uma diferenca fundamental entre objetos e estruturas de dados que afeta diretamente o design do software.

**Estruturas de dados** expoemem seus dados e nao tem funcoes significativas. **Objetos** escondem seus dados atras de abstracoes e expoemem funcoes que operam sobre esses dados.

### A Lei de Demeter

A Lei de Demeter (ou Principio do Menor Conhecimento) diz que um metodo `f` de uma classe `C` so deve chamar metodos de:
- A propria classe `C`
- Objetos criados por `f`
- Objetos passados como argumento para `f`
- Objetos armazenados em variaveis de instancia de `C`

A violacao classica e o "acidente de trem" (train wreck):

```java
// Ruim -- viola a Lei de Demeter (train wreck)
String cep = cliente.getEndereco().getCidade().getCep();

// Melhor -- pedir ao objeto que faca o trabalho
String cep = cliente.getCepResidencial();
```

### Objetos de Transferencia de Dados (DTOs)

DTOs sao estruturas de dados puras -- classes com variaveis publicas e sem funcoes. Sao uteis para comunicacao com banco de dados, parsing de mensagens, etc. Nao devem conter logica de negocio.

Active Records sao DTOs com metodos de navegacao como `save` e `find`. O erro comum e tratar Active Records como objetos, adicionando regras de negocio neles. Devem ser tratados como estruturas de dados, e a logica de negocio deve ficar em objetos separados.

### Anti-simetria entre objetos e estruturas de dados

- **Codigo procedural** (usando estruturas de dados) facilita adicionar novas funcoes sem alterar as estruturas existentes, mas dificulta adicionar novas estruturas pois todas as funcoes precisam mudar.

- **Codigo orientado a objetos** facilita adicionar novas classes sem alterar funcoes existentes (polimorfismo), mas dificulta adicionar novas funcoes pois todas as classes precisam mudar.

Bons desenvolvedores entendem essa tensao e escolhem a abordagem adequada para cada situacao.

### Licoes principais

- Objetos expoemem comportamento e escondem dados. Estruturas de dados expoemem dados e nao tem comportamento significativo.
- A escolha entre abordagem procedural e orientada a objetos deve ser consciente.
- A Lei de Demeter protege contra acoplamento excessivo.

---

## Capitulo 7 -- Tratamento de Erros (Error Handling)

### Conceitos-chave

Tratamento de erros e importante, mas se obscurece a logica do codigo, esta errado. Codigo limpo e legivel, e isso inclui o tratamento de erros.

### Regras detalhadas

**1. Use excecoes em vez de codigos de retorno**

Codigos de retorno poluem o chamador, que deve verificar o retorno imediatamente apos a chamada.

**2. Escreva blocos try-catch-finally primeiro**

Ao escrever codigo que pode lancar excecoes, comece pelo `try-catch-finally`. Isso define o escopo e ajuda a manter a consistencia do estado.

**3. Use excecoes nao-verificadas (unchecked exceptions)**

Martin argumenta contra checked exceptions do Java. Elas violam o Principio Aberto/Fechado: uma mudanca em um metodo de baixo nivel pode forcar alteracoes na assinatura de todos os metodos superiores na cadeia de chamadas.

**4. Forneca contexto nas excecoes**

Cada excecao deve fornecer contexto suficiente para determinar a fonte e a localizacao do erro. Inclua a operacao que falhou e o tipo de falha.

```java
// Ruim
throw new Exception("Erro na operacao");

// Bom
throw new OperacaoNaoPermitidaException(
    "Falha ao debitar R$" + valor + " da conta " + contaId +
    ": saldo insuficiente (saldo atual: R$" + saldoAtual + ")"
);
```

**5. Defina classes de excecao em termos das necessidades do chamador**

Classifique excecoes pela forma como serao tratadas, nao pela sua origem.

```java
// Ruim -- catch para cada tipo de excecao externa
try {
    porta.abrir();
} catch (DispositivoNaoEncontradoException e) {
    logger.error(e);
} catch (ConexaoRecusadaException e) {
    logger.error(e);
} catch (TimeoutException e) {
    logger.error(e);
}

// Bom -- wrapper que traduz excecoes
try {
    porta.abrir();
} catch (ExcecaoDePorta e) {
    logger.error(e);
}
```

**6. Nao retorne null**

Retornar `null` e um convite para `NullPointerException`. Se voce esta tentado a retornar null de um metodo, considere lancar uma excecao ou retornar um objeto especial (como uma lista vazia em vez de null).

```java
// Ruim
public List<Funcionario> getFuncionarios() {
    if (naoHaFuncionarios) return null;
}
// Obriga o chamador a verificar null em todo lugar

// Bom
public List<Funcionario> getFuncionarios() {
    if (naoHaFuncionarios) return Collections.emptyList();
}
```

**7. Nao passe null**

Passar `null` como argumento e ainda pior que retornar `null`. A menos que a API exija, nunca passe `null` para um metodo.

### O padrao Special Case (Null Object)

Em vez de tratar casos especiais com `if (x == null)`, crie uma classe que encapsula o comportamento especial:

```java
public class SalarioZerado implements Salario {
    public BigDecimal getValor() { return BigDecimal.ZERO; }
    public boolean estaAtivo() { return false; }
}
```

### Licoes principais

- Tratamento de erros nao deve dominar a logica do codigo.
- Nao retorne null. Nao passe null.
- Excecoes devem fornecer contexto rico para debugging.
- Encapsule APIs de terceiros para controlar as excecoes que voce lida.

---

## Capitulo 8 -- Limites (Boundaries)

### Conceitos-chave

Raramente controlamos todo o software de um sistema. Usamos bibliotecas de terceiros, APIs externas e codigo de outros times. Este capitulo trata de como manter limites limpos entre nosso codigo e codigo externo.

### Regras detalhadas

**1. Explorando e aprendendo limites (Learning Tests)**

Antes de usar uma API de terceiros em producao, escreva testes que exploram o comportamento da API. Martin chama esses de "learning tests". Eles servem para:
- Aprender como a API funciona
- Verificar se novas versoes da biblioteca mantém o comportamento esperado
- Documentar como voce usa a API

```java
@Test
public void testeComportamentoDoLog4j() {
    Logger logger = Logger.getLogger("MeuLogger");
    logger.addAppender(new ConsoleAppender(
        new PatternLayout("%p %t %m%n"),
        ConsoleAppender.SYSTEM_OUT));
    logger.info("mensagem de teste");
    // Verifica que o logger funciona como esperamos
}
```

**2. Encapsule codigo de terceiros**

Nao espalhe tipos de bibliotecas externas por todo o seu codigo. Encapsule-os em classes que voce controla.

```java
// Ruim -- Map espalhado por todo o codigo
Map<String, Sensor> sensores = new HashMap<>();
Sensor s = sensores.get(sensorId);

// Bom -- encapsulado
public class Sensores {
    private Map<String, Sensor> sensores = new HashMap<>();

    public Sensor buscarPorId(String id) {
        return sensores.get(id);
    }
}
```

A vantagem: se a interface `Map` mudar (ou se voce trocar a implementacao), apenas uma classe precisa ser alterada.

**3. Codigo que ainda nao existe**

Quando voce depende de uma API que ainda nao foi definida (outro time ainda esta desenvolvendo), defina a interface que voce **gostaria** de ter. Depois, use o padrao Adapter para conectar sua interface desejada a implementacao real quando ela existir.

### Licoes principais

- Limites sao pontos de alto risco para mudancas. Encapsule-os.
- Learning tests sao um investimento barato com alto retorno.
- Projete a interface que voce quer, nao a interface que voce recebe.

---

## Capitulo 9 -- Testes de Unidade (Unit Tests)

### Conceitos-chave

Este capitulo defende que **codigo de teste e tao importante quanto codigo de producao**. Testes sujos sao equivalentes a nao ter testes -- eventualmente a equipe para de manté-los, e a confianca no codigo desaparece.

### As Tres Leis do TDD

1. **Nao escreva codigo de producao** ate ter escrito um teste de unidade que falhe.
2. **Nao escreva mais de um teste de unidade** do que o suficiente para falhar (e nao compilar conta como falhar).
3. **Nao escreva mais codigo de producao** do que o suficiente para passar o teste que esta falhando.

Essas tres leis criam um ciclo de segundos: teste, codigo, teste, codigo.

### Testes limpos

O que torna um teste limpo? **Legibilidade, legibilidade e legibilidade.** As mesmas regras de codigo limpo se aplicam: clareza, simplicidade e densidade de expressao.

Martin recomenda o padrao **BUILD-OPERATE-CHECK** (ou Arrange-Act-Assert):

```java
@Test
public void deveAplicarDescontoParaComprasAcimaDeR100() {
    // Build (Arrange) -- preparar dados
    Carrinho carrinho = new Carrinho();
    carrinho.adicionar(new Produto("Livro", 120.00));

    // Operate (Act) -- executar acao
    carrinho.aplicarDescontos();

    // Check (Assert) -- verificar resultado
    assertEquals(108.00, carrinho.getTotal(), 0.01);
}
```

### Um assert por teste

Idealmente, cada funcao de teste deve ter um unico `assert`. Isso nem sempre e pratico, mas o objetivo e minimizar a quantidade de asserts por teste. Mais importante: **um conceito por teste**. Nao teste multiplos comportamentos na mesma funcao de teste.

### F.I.R.S.T. -- Propriedades de testes limpos

- **Fast (Rapidos):** Testes devem executar rapido. Testes lentos nao sao executados com frequencia.
- **Independent (Independentes):** Testes nao devem depender uns dos outros. Cada teste deve poder rodar isoladamente e em qualquer ordem.
- **Repeatable (Repetitivos):** Testes devem produzir o mesmo resultado em qualquer ambiente (producao, QA, local, sem rede).
- **Self-Validating (Auto-validaveis):** Testes devem ter saida booleana -- passam ou falham. Nao devem exigir que alguem leia um log para saber se funcionou.
- **Timely (Pontuais):** Testes devem ser escritos pouco antes do codigo de producao que os faz passar. Se voce escreve testes depois, o codigo de producao pode ser dificil de testar.

### Licoes principais

- Codigo de teste deve ser mantido com o mesmo rigor que codigo de producao.
- Testes sujos sao piores que nenhum teste: dao falsa confianca.
- Testes sao o que permite que voce mude o codigo com confianca.
- Sem testes, toda mudanca e um bug em potencial.

---

## Capitulo 10 -- Classes (Classes)

### Conceitos-chave

Depois de funcoes limpas, o proximo nivel de organizacao e a classe. Este capitulo aplica os principios de codigo limpo ao design de classes.

### Regras detalhadas

**1. Organizacao da classe**

Seguindo a convencao Java, a ordem dentro de uma classe deve ser:
1. Constantes publicas estaticas
2. Variaveis estaticas privadas
3. Variaveis de instancia privadas
4. Funcoes publicas
5. Funcoes privadas (logo apos a funcao publica que as chama)

**2. Classes devem ser pequenas**

Assim como funcoes, classes devem ser pequenas. Mas a medida de tamanho nao e linhas de codigo -- e **responsabilidades**. Uma classe deve ter uma unica responsabilidade.

Como testar: descreva a classe em 25 palavras ou menos, sem usar "e", "ou", "mas", "se". Se nao conseguir, a classe provavelmente tem responsabilidades demais.

```java
// Ruim -- responsabilidades demais
public class Funcionario {
    public void calcularSalario() { ... }
    public void salvarNoBanco() { ... }
    public void gerarRelatorioPDF() { ... }
    public void enviarEmailBoasVindas() { ... }
}

// Bom -- responsabilidade unica
public class CalculadoraSalario { ... }
public class RepositorioFuncionario { ... }
public class RelatorioFuncionario { ... }
public class NotificacaoFuncionario { ... }
```

**3. Principio da Responsabilidade Unica (SRP -- Single Responsibility Principle)**

Uma classe deve ter um, e somente um, motivo para mudar. Se uma classe pode mudar por causa de regras de negocio E por causa de mudancas no formato de relatorio, ela tem duas responsabilidades.

Muitos desenvolvedores criam classes com muitas responsabilidades porque pensam que "muitas classes pequenas" sao mais dificeis de navegar. Martin argumenta o contrario: um sistema com muitas classes pequenas e bem nomeadas e mais facil de navegar do que um com poucas classes gigantes, pelo mesmo motivo que um armario com muitas gavetas pequenas e rotuladas e mais organizado que um com poucas gavetas grandes.

**4. Coesao**

Classes devem ter alta coesao. Cada variavel de instancia deve ser usada por muitos metodos. Quando uma classe perde coesao (alguns metodos usam algumas variaveis, outros metodos usam outras), e sinal de que a classe deve ser dividida.

**5. Principio Aberto/Fechado (OCP -- Open/Closed Principle)**

Classes devem estar abertas para extensao e fechadas para modificacao. Isso e alcancado por meio de abstracoes (interfaces e classes abstratas).

```java
// Fechado para modificacao, aberto para extensao
public abstract class Forma {
    public abstract double area();
}

public class Retangulo extends Forma {
    public double area() { return largura * altura; }
}

public class Circulo extends Forma {
    public double area() { return Math.PI * raio * raio; }
}
// Adicionar Triangulo nao exige modificar Forma
```

**6. Principio de Inversao de Dependencia (DIP -- Dependency Inversion Principle)**

Classes devem depender de abstracoes, nao de implementacoes concretas. Modulos de alto nivel nao devem depender de modulos de baixo nivel. Ambos devem depender de abstracoes.

### Licoes principais

- Classes pequenas com responsabilidade unica sao a base de um bom design orientado a objetos.
- Alta coesao e baixo acoplamento caminham juntos.
- Sistemas bem organizados minimizam o impacto de mudancas.

---

## Capitulo 11 -- Sistemas (Systems)

### Conceitos-chave

Este capitulo sobe o nivel de abstracao para o design de sistemas inteiros. Assim como cidades precisam de planejamento urbano, sistemas de software precisam de arquitetura.

### Regras detalhadas

**1. Separe a construcao do uso**

A inicializacao e configuracao de um sistema e uma preocupacao ("concern") diferente da logica de execucao. O processo de construir objetos e suas dependencias deve ser separado da logica de negocio que os utiliza.

```java
// Ruim -- construcao misturada com uso
public Service getService() {
    if (service == null) {
        service = new MeuServicoImpl(...); // Dependencia hard-coded
    }
    return service;
}

// Bom -- injecao de dependencia
public class MinhaAplicacao {
    private final Service service;

    public MinhaAplicacao(Service service) {
        this.service = service; // Injetado externamente
    }
}
```

**2. Factories**

Quando a aplicacao precisa controlar quando objetos sao criados, use o padrao Abstract Factory. Isso mantém a logica de construcao separada da logica de negocio enquanto da ao codigo de aplicacao controle sobre o momento da criacao.

**3. Injecao de Dependencia (Dependency Injection)**

DI e a aplicacao do Principio de Inversao de Controle (IoC) a gestao de dependencias. Um objeto nao gerencia suas proprias dependencias -- um mecanismo externo (container) as injeta. Frameworks como Spring tornaram esse padrao popular em Java.

**4. Escale de forma incremental**

Martin argumenta que **nao e possivel acertar o sistema na primeira vez**. E mais eficaz comecar com um design simples e refatorar a medida que o sistema cresce, desde que se mantenha o codigo limpo e os testes adequados. "Big Design Up Front" (BDUF) frequentemente falha porque nao podemos prever todos os requisitos.

**5. Aspectos e preocupacoes transversais**

Preocupacoes transversais (cross-cutting concerns) como logging, seguranca, transacoes e cache permeiam varias partes do sistema. Frameworks como Spring AOP e AspectJ permitem separar essas preocupacoes do codigo de negocio, mantendo as classes focadas.

**6. Use padroes quando sao justificados**

Martin adverte contra o uso prematuro de padroes de design. Padroes adicionam complexidade. Use-os quando o beneficio for claro, nao "por precaucao".

### Licoes principais

- Sistemas limpos separam construcao de uso.
- Injecao de dependencia promove desacoplamento e testabilidade.
- Comece simples, escale incrementalmente, refatore continuamente.
- Otimize decisoes ate o ultimo momento responsavel -- nao antes.

---

## Capitulo 12 -- Emergencia (Emergence)

### Conceitos-chave

Kent Beck define quatro regras de **Design Simples** que, quando seguidas, permitem que bom design "emerja" naturalmente. Em ordem de importancia:

### As Quatro Regras de Design Simples

**1. Passa em todos os testes**

Um sistema que nao e verificavel nao deveria ser implantado. Criar testes leva a designs melhores porque classes dificeis de testar sao classes com design ruim. Testar incentiva baixo acoplamento e alta coesao.

**2. Nao contem duplicacao**

Duplicacao e o inimigo primario de um sistema bem projetado. Representa trabalho adicional, risco adicional e complexidade desnecessaria. Pequenas duplicacoes contam. Ate mesmo poucas linhas de codigo duplicadas devem ser extraidas.

```java
// Duplicacao sutil
public int calcularAreaRetangulo(int largura, int altura) {
    return largura * altura;
}

public int calcularAreaTriangulo(int base, int altura) {
    return (base * altura) / 2;
}

// Refatorado -- extrair logica comum se apropriado
// Neste caso simples, a "duplicacao" e aceitavel,
// mas em casos mais complexos, Template Method resolve.
```

**3. Expressa a intencao do programador**

A maioria do custo de um projeto de software esta na manutencao de longo prazo. Quanto mais claro o codigo, menos tempo outros gastarao tentando entende-lo. Use bons nomes, mantenha funcoes e classes pequenas, use padroes conhecidos (e nomeie-os, como `Command`, `Visitor`), e escreva testes expressivos.

**4. Minimiza o numero de classes e metodos**

Essa regra tem a menor prioridade. O objetivo e manter o sistema pequeno enquanto tambem mantém funcoes e classes pequenas. Mas nao crie abstracoes desnecessarias so por dogma. Pragmatismo prevalece.

### Licoes principais

- Testes viabilizam refatoracao, que viabiliza design emergente.
- Elimine toda duplicacao que encontrar.
- Expresse sua intencao claramente no codigo.
- Simplicidade e o objetivo final.

---

## Capitulo 13 -- Concorrencia (Concurrency)

### Conceitos-chave

Programacao concorrente e dificil. Muito dificil. Este capitulo nao tenta ensinar concorrencia em profundidade, mas oferece principios e tecnicas para manter codigo concorrente limpo.

### Mitos e verdades sobre concorrencia

- **Mito:** Concorrencia sempre melhora o desempenho.
  **Verdade:** So melhora quando ha tempo de espera significativo que pode ser compartilhado entre threads.

- **Mito:** O design nao muda ao escrever programas concorrentes.
  **Verdade:** O design de um algoritmo concorrente pode ser muito diferente do design de um single-threaded.

- **Mito:** Entender concorrencia nao e importante ao trabalhar com containers como Web ou EJB.
  **Verdade:** Voce precisa entender o que o container faz e como se proteger contra problemas de atualizacao concorrente e deadlock.

### Principios de defesa contra problemas de concorrencia

**1. Principio da Responsabilidade Unica para concorrencia**

Mantenha o codigo de concorrencia separado do restante do codigo. Codigo de concorrencia tem seu proprio ciclo de desenvolvimento, mudanca e tuning.

**2. Limite o escopo dos dados compartilhados**

Quanto mais pontos do codigo acessam dados compartilhados, mais provavel:
- Esquecer de proteger um ou mais desses pontos
- Duplicar esforco para garantir protecao
- Dificuldade de determinar a fonte de falhas

Use `synchronized` ou equivalentes para proteger secoes criticas que acessam dados compartilhados. Minimize o numero dessas secoes.

**3. Use copias de dados**

Em vez de compartilhar dados, considere copiar os dados para cada thread e coletar os resultados depois. O custo da copia pode ser menor que o custo do gerenciamento de locks.

**4. Threads devem ser tao independentes quanto possivel**

Projete threads para existirem em seu proprio mundo, com seus proprios dados (de fontes nao compartilhadas), processando independentemente.

**5. Conheca sua biblioteca**

Conheca as colecoes thread-safe da sua linguagem: `ConcurrentHashMap`, `AtomicInteger`, `ReentrantLock`, `CountDownLatch`, `Semaphore`, etc.

**6. Conheca seus modelos de execucao**

- **Producer-Consumer:** Um ou mais threads produzem trabalho em uma fila, um ou mais threads consomem.
- **Readers-Writers:** Leitores compartilham acesso, escritores precisam de acesso exclusivo. Equilibrar throughput vs. starvation.
- **Dining Philosophers:** Multiplas threads competem por multiplos recursos. Risco de deadlock.

**7. Mantenha secoes sincronizadas pequenas**

Locks sao caros e criam gargalos. Minimize o codigo dentro de blocos `synchronized`.

**8. Teste extensivamente**

- Trate falhas intermitentes como potenciais problemas de threading, nunca como "cosmicas".
- Faca o codigo nao-threaded funcionar primeiro.
- Faca o codigo threaded plugavel (configuravel para rodar com diferentes numeros de threads).
- Execute com mais threads que processadores.
- Execute em diferentes plataformas.
- Instrumentalize o codigo para forcar falhas (usando `Thread.yield()`, `Thread.sleep()` em pontos estrategicos).

### Licoes principais

- Concorrencia e inerentemente complexa. Mantenha-a isolada.
- Dados compartilhados sao a raiz da maioria dos problemas de concorrencia.
- Testes podem revelar problemas, mas nunca provar a ausencia de bugs concorrentes.

---

## Capitulo 14 -- Refinamento Sucessivo (Successive Refinement)

### Conceitos-chave

Este capitulo e um estudo de caso detalhado. Martin mostra o processo completo de desenvolvimento de um parser de argumentos de linha de comando (`Args`). O ponto central nao e a solucao final, mas **o processo de refinamento**.

### O processo demonstrado

1. **Comece com algo que funciona:** Martin mostra a primeira versao -- funcional, mas bagunçada. A classe `Args` comecou lidando apenas com argumentos booleanos. Quando suporte a strings e inteiros foi adicionado, a complexidade cresceu rapidamente e o design comecou a degradar.

2. **Reconheca quando parar de adicionar:** Em certo ponto, Martin percebeu que continuar adicionando funcionalidades ao design existente so pioraria as coisas. Ele parou e refatorou.

3. **Refatore incrementalmente:** A refatoracao nao foi uma reescrita. Martin fez pequenas mudancas, uma de cada vez, rodando os testes apos cada mudanca. Ele extraiu uma interface `ArgumentMarshaler`, criou implementacoes separadas para cada tipo (`BooleanArgumentMarshaler`, `StringArgumentMarshaler`, `IntegerArgumentMarshaler`), e moveu a logica de parsing para dentro de cada marshaler.

4. **O resultado final:** Uma estrutura limpa, extensivel e facil de entender. Adicionar um novo tipo de argumento exige apenas criar uma nova classe que implementa `ArgumentMarshaler`.

### A licao central

**Codigo limpo nao nasce limpo.** Ninguem escreve codigo limpo na primeira tentativa. O processo e:
1. Escreva codigo que funciona
2. Reconheca o momento de limpar
3. Limpe em passos pequenos, sempre com testes passando
4. Nunca deixe a degradacao continuar -- "depois" nunca chega

### Licoes principais

- Primeiro faca funcionar, depois faca certo.
- Refatoracao e um processo continuo, nao um evento.
- Testes sao a rede de seguranca que permite refatorar com confianca.
- Codigo ruim so piora com o tempo se nao for tratado.

---

## Capitulo 15 -- Internals do JUnit (JUnit Internals)

### Conceitos-chave

Martin analisa o codigo-fonte do framework JUnit, especificamente a classe `ComparisonCompactor`, que gera mensagens de falha de teste mostrando a diferenca entre valores esperados e obtidos.

### O processo de refinamento

Martin pega a classe original (que ja era codigo de qualidade, escrito por Kent Beck e Eric Gamma) e mostra como pequenas melhorias podem torna-la ainda melhor:

1. **Remocao de prefixos de membros:** Variaveis como `fExpected` foram renomeadas para `expected`. Em classes pequenas, o prefixo `f` e redundante.

2. **Encapsulamento de condicionais:** Condicionais complexos foram extraidos em metodos com nomes descritivos.

```java
// Antes
if (expected == null || actual == null || areStringsEqual())

// Depois
if (shouldNotCompact())
```

3. **Nomeacao melhorada:** Metodos e variaveis foram renomeados para expressar melhor sua intencao.

4. **Extracao de funcoes:** Blocos de logica foram extraidos em funcoes pequenas com nomes claros.

5. **Reordenacao:** Funcoes foram movidas para seguir a regra "stepdown" -- funcoes chamadoras acima das chamadas.

### Licoes principais

- Mesmo codigo bom pode ser melhorado.
- Melhorias pequenas e incrementais se acumulam.
- A Regra do Escoteiro em acao: deixe o codigo melhor do que encontrou.

---

## Capitulo 16 -- Refatorando SerialDate (Refactoring SerialDate)

### Conceitos-chave

Martin examina a classe `SerialDate` da biblioteca JCommon (de David Gilbert), uma classe que representa datas. Ele critica construtivamente e refatora o codigo.

### Principais observacoes e melhorias

1. **Renomeacao da classe:** `SerialDate` era um nome confuso (sugere serializacao). `DayDate` seria mais descritivo.

2. **Simplificacao de constantes:** Meses e dias da semana usavam constantes `int` quando enums seriam mais seguros e expressivos.

3. **Remocao de codigo morto:** Metodos nao utilizados ou redundantes foram removidos.

4. **Extracao de responsabilidades:** Logica de parsing e formatacao de datas foi movida para classes dedicadas.

5. **Melhoria de nomes:** Variaveis e metodos receberam nomes mais descritivos e consistentes.

6. **Simplificacao de logica:** Condicionais complexas foram simplificadas e comentarios redundantes foram removidos.

### O ponto crucial

Martin enfatiza que **criticar codigo nao e criticar o autor**. O codigo de David Gilbert e funcional e bem testado. Mas codigo sempre pode ser melhorado, e profissionais devem ter humildade para aceitar criticas construtivas e coragem para faze-las.

### Licoes principais

- Codigo de producao real e uma excelente fonte de aprendizado.
- Humildade e coragem sao necessarias na revisao de codigo.
- Enums sao superiores a constantes int para conjuntos fechados de valores.
- Remova codigo morto sem medo -- o Git guarda o historico.

---

## Capitulo 17 -- Cheiros e Heuristicas (Smells and Heuristics)

### Conceitos-chave

Este capitulo final e um catalogo de referencia. Martin lista "cheiros de codigo" (code smells) e heuristicas organizados por categoria. Esta e uma referencia para consulta rapida.

### Ambiente

- **A1 -- Build requer mais de um passo:** Voce deve conseguir dar checkout e fazer build com um unico comando simples.
- **A2 -- Testes requerem mais de um passo:** Voce deve conseguir rodar todos os testes com um unico comando rapido.

### Funcoes

- **F1 -- Argumentos demais:** Funcoes devem ter poucos argumentos. Mais de tres e questionavel.
- **F2 -- Argumentos de saida:** Argumentos de saida sao contra-intuitivos. Leitores esperam que argumentos sejam entradas.
- **F3 -- Argumentos flag (booleanos):** Argumentos booleanos indicam que a funcao faz mais de uma coisa. Divida em duas funcoes.
- **F4 -- Funcao morta:** Funcoes que nunca sao chamadas devem ser removidas.

### Comentarios

- **C1 -- Informacao inapropriada:** Comentarios nao devem conter informacao que pertence a outros sistemas (controle de versao, issue tracker).
- **C2 -- Comentario obsoleto:** Comentarios que envelheceram e nao sao mais precisos.
- **C3 -- Comentario redundante:** Comentarios que dizem o que o codigo ja diz claramente.
- **C4 -- Comentario mal escrito:** Se vai escrever um comentario, escreva bem. Sem gramatica ruim, sem abreviacoes confusas.
- **C5 -- Codigo comentado:** Codigo morto que foi comentado em vez de deletado. Delete-o.

### General (Geral)

- **G1 -- Multiplas linguagens em um arquivo:** Idealmente cada arquivo contem uma unica linguagem. Minimize a quantidade de linguagens em um arquivo.
- **G2 -- Comportamento obvio nao implementado:** Quando o comportamento esperado e obvio e nao esta implementado, leitores perdem confianca no codigo.
- **G3 -- Comportamento incorreto nos limites:** Nao confie na intuicao. Teste cada condicao de contorno.
- **G4 -- Segurancas desativadas:** Desligar testes que falham ou ignorar warnings do compilador e perigoso. Nao silencie alarmes.
- **G5 -- Duplicacao (DRY):** Uma das heuristicas mais importantes. Duplicacao aparece em muitas formas: blocos identicos de codigo, condicionais switch/case repetidos, algoritmos semelhantes.
- **G6 -- Codigo no nivel errado de abstracao:** Nao misture conceitos de alto e baixo nivel na mesma classe. Separe detalhes de implementacao de politicas de alto nivel.
- **G7 -- Classes base dependem de derivadas:** Classes base nao devem saber sobre suas derivadas.
- **G8 -- Informacao excessiva:** Bons modulos tem interfaces pequenas e profundas. Esconda o maximo possivel. Exponha o minimo.
- **G9 -- Codigo morto:** Codigo que nao e executado. Encontre-o e remova-o.
- **G10 -- Separacao vertical:** Variaveis e funcoes devem ser definidas perto de onde sao usadas.
- **G11 -- Inconsistencia:** Se voce faz algo de uma forma, faca todas as coisas semelhantes da mesma forma. Consistencia facilita leitura.
- **G12 -- Entulho (Clutter):** Variaveis nao usadas, funcoes nunca chamadas, comentarios sem informacao. Remova tudo que nao agrega valor.
- **G13 -- Acoplamento artificial:** Nao acople coisas que nao dependem uma da outra. Enums gerais nao devem ficar dentro de classes especificas.
- **G14 -- Feature Envy:** Metodos de uma classe que usam mais atributos e metodos de outra classe do que da propria. Mova o metodo para onde ele pertence.
- **G15 -- Argumentos seletores:** Argumentos booleanos ou enums usados para selecionar comportamento dentro de funcoes. E melhor ter funcoes separadas.
- **G16 -- Intencao obscura:** Codigo que usa tecnicas que obscurecem a intencao (expressoes complexas compactadas em uma unica linha, notacao hungara, numeros magicos).
- **G17 -- Responsabilidade mal colocada:** Codigo no lugar errado. A pergunta e: onde o leitor naturalmente esperaria encontrar esta funcionalidade?
- **G18 -- Uso inapropriado de static:** Prefira metodos de instancia a metodos estaticos. Metodos estaticos nao podem ser polimorificos e dificultam testes.
- **G19 -- Use variaveis explicativas:** Quebre calculos complexos em variaveis intermediarias com nomes significativos.

```java
// Ruim
return (int)(Math.ceil(itens.size() / (double)ITENS_POR_PAGINA));

// Bom
int totalItens = itens.size();
double paginasDecimais = totalItens / (double) ITENS_POR_PAGINA;
int totalPaginas = (int) Math.ceil(paginasDecimais);
return totalPaginas;
```

- **G20 -- Nomes de funcoes devem dizer o que fazem:** Se voce precisa ler o corpo da funcao para saber o que ela faz, o nome precisa ser melhor.
- **G21 -- Entenda o algoritmo:** Antes de considerar um metodo "pronto", tenha certeza de que entende como ele funciona. Nao apenas "parece funcionar" -- entenda de verdade.
- **G22 -- Torne dependencias logicas em fisicas:** Se um modulo depende de outro, essa dependencia deve ser explicita (passada como argumento ou injetada), nao implicita (assumindo que algo existe no contexto).
- **G23 -- Prefira polimorfismo a if/else ou switch/case:** Quando a mesma condicional aparece repetidamente, use polimorfismo. A regra "ONE SWITCH": pode haver no maximo um switch para cada tipo de selecao, e ele deve criar objetos polimorficos, nao ser repetido.
- **G24 -- Siga convencoes padrao:** A equipe deve ter um padrao de codificacao e todos devem segui-lo.
- **G25 -- Substitua numeros magicos por constantes nomeadas:**

```java
// Ruim
double circunferencia = raio * 2 * 3.14159;
if (idade > 18) { ... }

// Bom
double circunferencia = raio * 2 * Math.PI;
if (idade > IDADE_MINIMA_VOTACAO) { ... }
```

- **G26 -- Seja preciso:** Nao seja preguicoso com decisoes. Se voce decide usar float quando deveria ser BigDecimal, bugs financeiros aparecerão. Se voce nao trata null, NullPointerException aparecerá. Seja deliberado.
- **G27 -- Prefira estrutura a convencao:** Convencoes dependem de disciplina e sao facilmente violadas. Use mecanismos estruturais (interfaces, classes abstratas, compilador) que forcam o comportamento correto.
- **G28 -- Encapsule condicionais:**

```java
// Ruim
if (timer.hasExpired() && !timer.isRecurrent())

// Bom
if (shouldBeDeleted(timer))
```

- **G29 -- Evite condicionais negativas:**

```java
// Ruim
if (!buffer.shouldNotCompact())

// Bom
if (buffer.shouldCompact())
```

- **G30 -- Funcoes devem fazer uma coisa:** Reforcando o capitulo 3.
- **G31 -- Nao esconda acoplamentos temporais:** Se funcoes devem ser chamadas em ordem especifica, torne isso explicito na assinatura.

```java
// Ruim -- nada impede de chamar fora de ordem
inicializar();
processar();
finalizar();

// Bom -- cada funcao retorna o necessario para a proxima
Contexto ctx = inicializar();
Resultado res = processar(ctx);
finalizar(res);
```

- **G32 -- Nao seja arbitrario:** Tenha uma razao para a estrutura do seu codigo. Se a estrutura parece arbitraria, outros vao sentir liberdade para muda-la arbitrariamente.
- **G33 -- Encapsule condicoes de contorno:** Condicoes de contorno sao dificeis de acompanhar. Centralize-as.

```java
// Ruim
if (level + 1 < tags.length) {
    parts = new Parse(body, tags, level + 1, offset + endTag);
}

// Bom
int nextLevel = level + 1;
if (nextLevel < tags.length) {
    parts = new Parse(body, tags, nextLevel, offset + endTag);
}
```

- **G34 -- Funcoes devem descer apenas um nivel de abstracao.**
- **G35 -- Mantenha dados configuraveis em niveis altos:** Constantes de configuracao (como valores default, nomes de arquivos, timeouts) devem ficar em niveis altos de abstracao, faceis de encontrar e modificar.
- **G36 -- Evite navegacao transitiva (Lei de Demeter).**

### Nomes

- **N1 -- Escolha nomes descritivos.**
- **N2 -- Escolha nomes no nivel adequado de abstracao.**
- **N3 -- Use nomenclatura padrao quando possivel.** Padroes como Singleton, Factory, Decorator comunicam muito em uma unica palavra.
- **N4 -- Nomes inequivocos.** Escolha nomes que nao possam ser confundidos com outras coisas.
- **N5 -- Use nomes longos para escopos longos.** Variaveis com escopo curto (loop de 5 linhas) podem ter nomes curtos. Variaveis com escopo longo precisam de nomes longos e descritivos.
- **N6 -- Evite codificacoes.** Nao use prefixos, notacao hungara ou sufixos de tipo.
- **N7 -- Nomes devem descrever efeitos colaterais.**

```java
// Ruim -- nao menciona que cria se nao existir
public ObjectOutputStream getOos() throws IOException {
    if (oos == null) {
        oos = new ObjectOutputStream(socket.getOutputStream());
    }
    return oos;
}

// Bom
public ObjectOutputStream createOrReturnOos() throws IOException { ... }
```

### Testes

- **T1 -- Testes insuficientes:** Testes devem cobrir tudo que pode quebrar. Se ha condicoes nao testadas, os testes sao insuficientes.
- **T2 -- Use ferramentas de cobertura.** Ferramentas de cobertura facilitam encontrar lacunas nos testes.
- **T3 -- Nao pule testes triviais.** Sao faceis de escrever e seu valor documental e maior que o custo.
- **T4 -- Teste ignorado e uma pergunta sobre ambiguidade.** Se o requisito nao esta claro, um teste marcado como `@Ignore` documenta a ambiguidade.
- **T5 -- Teste condicoes de contorno.** A maioria dos bugs se esconde nos limites.
- **T6 -- Teste extensivamente ao redor de bugs.** Bugs tendem a se agrupar. Quando encontrar um bug, teste a funcao extensivamente.
- **T7 -- Padroes de falha sao reveladores.** Olhe para os padroes nos testes que falham -- eles revelam o diagnostico.
- **T8 -- Padroes de cobertura sao reveladores.** Olhar quais linhas nao estao sendo executadas pode indicar por que testes falham.
- **T9 -- Testes devem ser rapidos.** Testes lentos nao sao executados. Testes que nao sao executados nao tem valor.

### Licoes principais

- Este capitulo funciona como uma lista de verificacao (checklist) para revisao de codigo.
- Memorize os "cheiros" mais comuns e desenvolva sensibilidade para detecta-los.
- Codigo limpo e um processo de melhoria continua, nao um destino final.

---

## Principais Licoes

### Os 10 mandamentos do codigo limpo

1. **Nomes importam mais do que voce pensa.** Um bom nome elimina a necessidade de comentarios, torna o codigo pesquisavel e comunica intencao. Invista tempo em nomes e nao tenha medo de renomear.

2. **Funcoes devem ser pequenas e fazer uma unica coisa.** Se uma funcao tem mais de 20 linhas, provavelmente faz coisas demais. Se o nome precisa de "e" ou "ou", ela faz mais de uma coisa. Extraia sem medo.

3. **Comentarios sao um sinal de fracasso.** O melhor comentario e o que voce nao precisou escrever porque o codigo se explica. Nunca mantenha codigo comentado -- use controle de versao.

4. **Testes sao cidadaos de primeira classe.** Codigo de teste merece o mesmo cuidado que codigo de producao. Sem testes, toda refatoracao e uma roleta russa. Siga o FIRST: Fast, Independent, Repeatable, Self-Validating, Timely.

5. **O Principio da Responsabilidade Unica e o mais importante do SOLID.** Cada classe e cada funcao deve ter uma e apenas uma razao para mudar. Classes com muitas responsabilidades sao dificeis de entender, testar e manter.

6. **Elimine toda duplicacao que encontrar.** DRY (Don't Repeat Yourself) e talvez o principio mais universal da engenharia de software. Duplicacao multiplica o custo de mudancas e a chance de erros.

7. **Nao retorne null, nao passe null.** Null e a origem de incontaveis bugs. Use colecoes vazias, objetos especiais (Null Object pattern) ou excecoes em vez de null.

8. **Encapsule o que e externo.** APIs de terceiros, banco de dados, servicos externos -- encapsule tudo atras de interfaces que voce controla. Isso protege seu codigo de mudancas externas.

9. **A Regra do Escoteiro e inegociavel.** Toda vez que tocar em um arquivo, deixe-o melhor. Pequenas melhorias continuas evitam degradacao. "Depois eu limpo" e uma mentira.

10. **Codigo limpo nao nasce limpo.** Nenhum programador escreve codigo limpo na primeira tentativa. O processo e: faca funcionar, entao faca certo. Refatore continuamente, em passos pequenos, com testes como rede de seguranca.

### Resumo final

O livro pode ser condensado em uma unica frase: **trate codigo como literatura -- ele e escrito para ser lido por humanos, nao apenas por maquinas.** A capacidade de escrever codigo que outros (incluindo voce no futuro) possam ler, entender e modificar com confianca e o que separa um programador de um profissional de software.

A excelencia em codigo limpo nao vem de regras memorizadas, mas de pratica deliberada e constante. Leia codigo de outros, refatore seu proprio codigo, revise codigo de colegas e nunca pare de aprender.
