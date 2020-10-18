# Catálogo de Code Smells ORM


A tecnologia *ORM* apresenta diversos benefícios, com abstração das comunicações pertinentes a camada de dados e conversão entre objetos e o banco de dados relacional, permitindo ao desenvolvedor um foco maior no
desenvolvimento das regras de negócio. Todavia essa abstração pode trazer problemas de desempenho e manutenibilidade se as configurações, mapeamentos e instruções ORM forem realizadas de forma inadequada. Nesta seção, apresentamos o catálogo de *code smells* ORM em Java. A definição do que representa um *code smell* é decorrente de uma análise subjetiva baseada na
experiência e na intuição humana. Para que uma prática seja considerada
um *code smell* considerando o domínio específico de ORM em Java, ela deve indicar uma má escolha de implementação e sugerir sintomas que podem ser indicativos de algo errado no código, indicando a necessidade de refatoração. Portanto, com base nos resultados das revisões (RR e GLR), extraímos más práticas relacionadas a ORM em Java. O catálogo possui o foco em Java devido à linguagem possuir uma especificação padrão para ORM através da API JPA que permite categorizar de uma mesma maneira *code smells* ORM em diferentes *frameworks*. Foram selecionados *code smells* cujo conteúdo possui pelo menos três citações em fontes diferentes nas revisões realizadas, resultando em oito *code smells* ORM.

Os *code smells* selecionados foram agrupados por
tipos, definidos como os possíveis problemas causados
pelo conjunto de *code smells* relacionados. Para uniformização, cada
*code smell* ORM está estruturado em: *descrição do smell*, com uma
descrição resumida do *code smell* ORM bem como exemplo prático;
sugestão de refatoração, apresentando as melhores práticas encontradas
nas revisões realizadas e um exemplo prático corrigindo o apresentado na
descrição do *smell*; por fim, uma discussão e detalhamento referente ao
*code smell* em questão. 
A Tabela abaixo apresenta os oito *code smells* ORM selecionados para o catálogo, contendo um breve resumo e categorizados por tipo de problema, seguidos pelas referências das evidências do RR e GLR a partir dos quais os extraímos.

<!-- <img src="figures/catalogo_tabela.jpg" width="800"> -->

|Tipo |Code Smell ORM | Referências da Revisão | Resumo|
|-----|---------------|------------------------|-------|
|[Dados em Excesso](#Dados-em-Excesso)|[1. Eager como estratégia de busca nos relacionamentos a nível de classe](#eager-como-estratégia-de-busca-nos-relacionamentos-a-nível-de-classe-estático) | [RR01], [RR03], [RR04], [GLR02], [GLR03], [GLR04], [GLR05], [GLR07], [GLR09], [GLR10], [GLR11], [GLR12], [GLR13], [GLR14], [GLR15], [GLR16], [GLR17], [GLR18]. | Atributos que representam relacionamento mapeados com estratégia de busca Eager a nível de classe (estático) faz o objeto relacionado sempre ser carregado pelo ORM, mesmo quando não utilizado, sem poder alterar o comportamento a nível de consulta (dinâmico)|
|[Dados em Excesso](#Dados-em-Excesso)| [2. Recuperação de dados sem projeção para somente leitura](#recuperação-de-dados-sem-projeção-para-somente-leitura) | [RR01], [GLR02], [GLR03], [GLR05], [GLR17]. | Não utilização de projeção ou DTOs para recuperar somente os atributos desejados do banco de dados quando o resultado for utilizado apenas para leitura, faz com que sejam recuperados dados em excesso. |
|[Dados em Excesso](#Dados-em-Excesso)| [3. Atualização desnecessária de toda a entidade](#atualização-desnecessária-de-toda-a-entidade) | [RR01], [RR02], [RR03]. | Qualquer alteração de atributos na classe faz com que todas as colunas da entidade mapeada sejam atualizadas no banco de dados, quando seria necessário atualizar apenas as colunas dos atributos alterados. |
|[Dados em Excesso](#Dados-em-Excesso)| [4. Falta de paginação quando não necessário todo o resultado](#falta-de-paginação-quando-não-necessário-todo-o-resultado) | [GLR01], [GLR02], [GLR05], [GLR13], [GLR17]. | Consultas que retornam coleção de dados sem a utilização dos parâmetros de paginação para limitar os resultados quando não utilizado todos os registros. |
|[N + 1](#problema-do-n1)| [5. Falta de Join Fetch: estratégia de busca Eager](#falta-de-join-fetch-nas-consultas-de-atributos-do-tipo-eager) | [RR01], [RR04], [GLR02], [GLR04], [GLR05], [GLR06], [GLR07], [GLR09], [GLR10], [GLR11], [GLR12], [GLR13], [GLR14], [GLR15], [GLR16], [GLR17], [GLR18]. | Consultas ORM sem utilização de JOIN FETCH para os atributos que estão mapeados como Eager a nível de classe, gerando N consultas adicionais para carregar os objetos relacionados. |
|[N + 1](#problema-do-n1)| [6. Acesso um por um: estrutura de repetição e estratégia de busca Lazy](#acesso-um-por-um-estrutura-de-repetição-e-estratégia-de-busca-do-tipo-lazy) | [RR03], [RR04], [GLR01], [GLR05], [GLR08], [GLR10], [GLR11], [GLR12], [GLR13], [GLR14], [GLR16], [GLR19]. | Ao recuperar um atributo de um objeto referente a uma entidade dentro de uma estrutura de repetição, se a estratégia de busca for Lazy, consultas adicionais serão executadas para cada iteração. |
|[N + 1](#problema-do-n1)| [7. @OneToMany unilateral com uso inadequado de coleções](#onetomany-unilateral-com-uso-inadequado-de-coleções) | [RR05], [GLR07], [GLR15]. | @OneToMany unilateral com uso inadequando de coleções Java faz com que a cada inserção/remoção de um elemento seja modificado todos os registros da coleção no banco de dados. |
|[Outros](#outros-code-smells-orm)| [8. Não uso de consultas somente leitura](#não-uso-de-consultas-somente-leitura) | [GLR01], [GLR02], [GLR15]. | Objetos recuperados da base de dados para fins únicos de consultas sem que possuam configurações para somente leitura, faz com que seja gerenciado pelo contexto de persistência desnecessariamente. |



## Dados em excesso 
----------------

Segundo (Mihalcea et al. 2018), trazer dados em excesso do banco de
dados para a aplicação pode ser considerado o problema de desempenho
número um na maioria das aplicações que utilizam JPA. (Chen, Shang,
Jiang, et al. 2016) relata que o fato do ORM operar em um nível inferior
(acesso a dados), não permite identificar quais dados serão utilizados
no retorno e assim não fornece uma abordagem ideal de recuperação de
dados para todos os casos, podendo gerar problemas de desempenho ao
recuperar dados em excesso do banco de dados. A Figura
abaixo apresenta um exemplo ao requerer apenas a
matricula do objeto `discente`, a consulta gerada pelo *framework* ORM
busca mais informações que a desejada no banco de dados. A seguir são
apresentados os *code smells* relacionados a este problema.

<p align="center"> 
<img src="figures/dadosExcesso.png">
</p>

### Eager como estratégia de busca nos relacionamentos a nível de classe (estático) 

#### Descrição do *Smell*

Utilização da estratégia de busca <span
style="font-variant:small-caps;">Eager</span> (anotação
`FetchType.EAGER`) nos atributos que representam relacionamentos a nível
de classe, ou a falta de anotação `FetchType.LAZY` para relacionamentos
com anotação `@ManyToOne` e `@OneToOne` que por padrão utilizam a
estratégia de busca <span style="font-variant:small-caps;">Eager</span>,
faz com que o objeto relacionado sempre seja recuperado do banco de
dados mesmo quando não utilizado, podendo causar desperdício de
processamento computacional, problemas de desempenho e manutenção pois
não permite a alteração da estratégia de busca em nível de consulta
(dinâmico). O Exemplo abaixo demonstra a ocorrência do
*smell* utilizando a anotação `@ManyToOne` com a estratégia de busca do
tipo <span style="font-variant:small-caps;">Eager</span>. O objetivo do
código é recuperar apenas a matrícula de um discente, porém recupera
também a entidade `Pessoa` sem necessidade através de <span
style="font-variant:small-caps;">JOIN</span>.
  
  **Código Java:**
```java
    @Entity
    class Discente{
      @ManyToOne(fetch = FetchType.EAGER)
      private Pessoa pessoa;	
      ...
    }

    public static void main(String[] args) {
      ...
      Discente d = findDiscenteById(1);
      d.getMatricula();
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT * FROM Discente d LEFT OUTER JOIN Pessoa p
    ON     p.id_pessoa = d.id_pessoa WHERE   d.id=1;
```
---------------------------
#### Sugestão de Refatoração

A estratégia de busca definida para relacionamentos a nível de classe
deve ser do tipo <span style="font-variant:small-caps;">Lazy</span>,
conforme recomendações encontradas nas revisões realizadas, evitando
assim carregar o objeto relacionado para todas as regras de negócio que
utilizam o objeto da classe em questão. Com a utilização de <span
style="font-variant:small-caps;">Lazy</span>, o ORM irá recuperar o
objeto relacionado somente quando alguma informação do mesmo for
solicitada. Se for necessário que o objeto do relacionamento seja
recuperado com o comportamento <span
style="font-variant:small-caps;">Eager</span> para alguma regra de
negócio de forma a evitar consultas adicionais, deve ser realizado o
tratamento em nível de consulta, realizando <span
style="font-variant:small-caps;">JOIN FETCH</span> na consulta para a
regra de negócio ou através da anotação `@NamedEntityGraph` introduzida
no JPA 2.1[^1] que permite definir um grafo de entidades a serem
recuperadas no banco de dados. O Exemplo abaixo apresenta a refatoração do Exemplo anterior, alterando a estratégia de busca para <span
style="font-variant:small-caps;">Lazy</span>, evitando assim o <span
style="font-variant:small-caps;">JOIN</span> desnecessário com a
entidade `Pessoa`.
  
  **Código Java:**
```java
    @Entity
    class Discente{
     @ManyToOne(fetch = FetchType.LAZY)
     private Pessoa pessoa;	
     ...
    }
    public static void main(String[] args) {
     ...
     Discente d = findDiscenteById(1);
     d.getMatricula();
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT * FROM Discente d WHERE d.id=1;
```
  ---------------------------

#### Detalhamento e Discussão

Uma das causas de excesso de dados em *frameworks* ORM é a utilização
indevida da estratégia de busca do tipo <span
style="font-variant:small-caps;">Eager</span>. Essa estratégia é
utilizada para carregar previamente as informações do banco de dados de
um objeto relacionado. O mapeamento na forma estática utiliza uma
anotação a nível de classe no atributo que representa outro objeto, de
forma a sempre carregar o objeto relacionado. Por exemplo: objeto A
possui um atributo relacionado ao objeto B. Utilizando <span
style="font-variant:small-caps;">Eager</span>, sempre que qualquer
atributo do objeto A for carregado, o objeto B também será, independente
se alguma informação do objeto B será utilizada. (Bauer and King 2005)

Segundo a documentação oficial do Hibernate, a utilização da estratégia
de busca do tipo <span style="font-variant:small-caps;">Eager</span> de
forma estática (definida a nível de classe) é quase sempre uma má
escolha (Mihalcea et al. 2018). Esta estratégia é considerando um *code
smell* ORM devido aos seguintes riscos:

-   Os dados do objeto relacionado sempre serão carregados por completo,
    ainda que não sejam utilizados. Assim, poderá acarretar em um
    processamento desnecessário ao banco de dados, carregando sempre o
    objeto relacionado. O problema pode se agravar caso o objeto
    relacionado também esteja com outros objetos associados utilizando
    <span style="font-variant:small-caps;">Eager</span>, gerando mais
    consultas desnecessárias. (Chen et al. 2014);

-   Se durante uma consulta ORM não for realizado o <span
    style="font-variant:small-caps;">JOIN FETCH</span> para todas os
    relacionamentos anotados como <span
    style="font-variant:small-caps;">Eager</span>, será realizada uma
    sub-consulta para cada relacionamento, podendo gerar o problema do
    `N+1` (Mihalcea et al. 2018).

-   Ao definir um relacionamento na classe para <span
    style="font-variant:small-caps;">Eager</span> não poderá
    sobrescrever o comportamento para <span
    style="font-variant:small-caps;">Lazy</span> em nível dinâmico
    (nível de consulta). Já o contrário (de <span
    style="font-variant:small-caps;">Lazy</span> para <span
    style="font-variant:small-caps;">Eager</span>) é possível utilizando
    <span style="font-variant:small-caps;">JOIN FETCH</span> na consulta
    (Mihalcea 2017).

(Chen et al. 2014) relata que após realizada a refatoração alterando a
estratégia de busca de <span
style="font-variant:small-caps;">Eager</span> para <span
style="font-variant:small-caps;">Lazy</span> em uma classe com 300
registros associada a outra com 10 registros, houve um aumento de 71% de
desempenho. Para um dos autores responsáveis pela documentação do
Hibernate, (Vlad Mihalcea 2019a) a estratégia de busca <span
style="font-variant:small-caps;">Eager</span> definida de forma estática
é um *code smell*. Na maioria das vezes é utilizado pelo desenvolvedor
para simplificar o desenvolvimento, porém não são considerados os
problemas de desempenho e manutenibilidade a longo prazo.

Apesar da documentação oficial da versão 5 do Hibernate (Mihalcea et al.
2018) e da documentação oficial do EclipseLink (Oracle 2015) recomendar
a não utilização, atualmente <span
style="font-variant:small-caps;">Eager</span> é a estratégia de busca
padrão para mapeamentos do tipo `ManyToOne` e `OneToOne` nos
*frameworks* ORM em Java. O motivo, segundo (Mihalcea et al. 2018), é a
implementação do JPA. Antes da especificação JPA, o Hibernate por
exemplo definia todos os seus mapeamentos com a estratégia de busca
<span style="font-variant:small-caps;">Lazy</span> por padrão. Com o
surgimento da especificação JPA 1.0, pensava-se que nem todos os
provedores que iriam implementar JPA usariam *proxies*, podendo causar
erros ao buscar uma informação de forma <span
style="font-variant:small-caps;">Lazy</span> em uma conexão fechada,
lançando a exceção `LazyInitializationException`. Como são *frameworks*
ORM que implementam o JPA, seguem a especificação, porém recomendam em
suas documentações oficiais que os mapeamentos do tipo `ManyToOne` e
`OneToOne` devem ser explicitamente definidos como <span
style="font-variant:small-caps;">Lazy</span>. As demais estratégias de
busca são <span style="font-variant:small-caps;">Lazy</span> por padrão.
(Chen, Shang, Jiang, et al. 2016) relata que relacionamentos `ManyToOne`
e `OneToOne` apesar de representar um relacionamento de associação
única, podem gerar grandes problemas de desempenho utilizando <span
style="font-variant:small-caps;">Eager</span> quando o objeto recuperado
contém grandes dados (por exemplo, binários ou imagens) ou o objeto
recuperado também recupera de forma <span
style="font-variant:small-caps;">Eager</span> muitos outros objetos
relacionados). O problema se agrava quando associações do tipo
`OneToMany` e `ManyToMany`, que representam coleções e por padrão são do
tipo <span style="font-variant:small-caps;">Lazy</span>, são definidas
explicitamente com a estratégia de busca do tipo <span
style="font-variant:small-caps;">Eager</span>. Com essa estratégia, a
consulta pode exigir um produto cartesiano limitando a velocidade da
consulta a quantidade de registros associados, ou gerando o problema de
`N+1` (o qual será visto na próxima seção) com $N$ consultas adicionais
(V. Mihalcea 2016).

Com base no exposto, consideramos um *code smell* a utilização da
estratégia de busca <span style="font-variant:small-caps;">Eager</span>
a nível de classe (forma estática). Seu uso pode não gerar problemas
caso a classe sempre necessite do objeto relacionado e este possua dados
pequenos na base de dados. Mesmo nestes casos, é considerado um *code
smell* pois é complexo ter certeza no momento do desenvolvimento que
todos os casos de usos futuros precisarão do objeto do relacionamento e
que a base de dados se manterá da mesma forma, sendo recomendado
utilizar <span style="font-variant:small-caps;">Lazy</span> a nível de
classe e utilizar o comportamento <span
style="font-variant:small-caps;">Eager</span> a nível de consulta
através de <span style="font-variant:small-caps;">Join Fetch</span> para
os que necessitarem. Fazendo uma relação com o apresentado na Seção
\[sec:bad\_smells\] é possível concluir que é uma indicação de
refatoração, uma má escolha de implementação e indica possíveis
problemas futuros relativos a desempenho e manutenibilidade. Portanto,
qualquer atributo em uma classe que representa uma associação entre
entidades contendo qualquer anotação de relacionamento (`@ManyToMany`,
`@OneToMany`, `@ManytoOne`, `@OneToOne`) com o `FetchType.EAGER`
definido, ou, em específico para relacionamentos <span
style="font-variant:small-caps;">ManyToOne</span> e <span
style="font-variant:small-caps;">OneToOne</span> que não estejam
explicitamente com `FetchType.LAZY` anotados, são considerados *code
smells* para o contexto deste trabalho. Os casos dos relacionamentos
<span style="font-variant:small-caps;">ManyToMany</span> e <span
style="font-variant:small-caps;">OneToMany</span> são considerados como
mais graves, devido ao *Many* (Muitos) ao fim do relacionamento, o que
faz com que a consulta de um objeto apenas, trará sempre $N$ outros
objetos do relacionamento.

### Recuperação de dados sem projeção para somente leitura 

#### Descrição do *Smell*

Utilizar consultas ORM sem a utilização de projeção quando a finalidade
for somente leitura (quando nenhum gerenciamento do contexto de
persistência é necessário) pode ocasionar problemas de desempenho,
especialmente ao recuperar informações grandes como colunas do tipo
<span style="font-variant:small-caps;">BLOB</span> ou junções com outras
entidades de forma desnecessária, gerando incompatibilidade entre dados
necessários e dados recuperados. O Exemplo abaixo
demonstra a ocorrência do *smell*, onde o HQL utilizado recupera todas
as colunas da entidade `Discente`, incluindo dados grandes (coluna
`arquivo`), sendo apenas a informação da matrícula necessária para o
caso de uso.
  
  **Código Java:**
```java
    public Discente findDiscenteById(Integer id){
     String hql = "FROM Discente d WHERE id = :id";
     Query q = entityManager.createQuery(hql.toString());
     q.setInteger("id", id);
     return (Discente) q.uniqueResult();
    }

    public static void main(String[] args) {
     ...
     Discente d = findDiscenteById(1);
     d.getMatricula();
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT d.id, d.matricula, d.arquivo, d.data_cadastro, d... 
    FROM   Discente d WHERE d.id=1;
```
  ---------------------------

#### Sugestão de Refatoração

Para evitar trazer todos os dados da entidade, a recomendação é utilizar
projeção nas consultas. Uma das formas de realizar projeção em consultas
ORM é utilizar o operador `NEW`. Esse operador permite declarar apenas
as colunas ou atributos que são necessários carregar do banco de dados
através do construtor da classe consultada ou de um . O Exemplo abaixo demonstra um ajuste no Exemplo anterior utilizando o operador `NEW` para trazer o
próprio objeto apenas com a informação necessária para o caso de uso.
  
  **Código Java:**
```java
    public Discente findDiscenteById(Integer id){
     ...
     hql.append("SELECT new Discente(matricula) ")
     hpl.append("FROM Discente WHERE id = :id");
     Query q = entityManager.createQuery(hql.toString());
     q.setInteger("id", id);
     return (Discente) q.uniqueResult();
    }

    public static void main(String[] args) {
     ...
     Discente d = findDiscenteById(1);
     d.getMatricula();
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT d.matricula FROM Discente d WHERE d.id=1;
```
  ---------------------------

#### Detalhamento e Discussão

Realizar consultas sem projeção pode ocasionar problemas de desempenho e
manutenção, trazendo mais informação do que necessário. Quando realizada
uma consulta sem projeção, o *framework* ORM não sabe quais dados serão
necessários posteriormente, trazendo todas as colunas da entidade e seus
relacionamentos (dependendo da estratégia de busca). Essa prática pode
recuperar colunas que contenham informações grandes do banco de dados
(como uma imagem), ou muitos relacionamentos gerando um processamento
desnecessário pelo banco e pela aplicação (Chen, Shang, Jiang, et al.
2016). Se for necessário atualizar os dados posterior a recuperação, é
importante buscar a entidade completa para ser gerenciada pelo
*framework* ORM, isso devido ao chamado *mecanismo de verificação suja*,
conforme discutido na Seção \[jpa-secao\] (Mihalcea et al. 2018).

No entendimento de (Mihalcea et al. 2018), se o objetivo for recuperar
informações somente leitura, é recomendável utilizar manipulando somente
os atributos desejados para o caso de uso. Essa estratégia traz alguns
benefícios, como reduzir a carga no contexto de persistência em execução
devido a projeções realizadas em s não precisarem ser gerenciadas pelo
*framework* ORM, além de evitar que atributos que estejam mapeados com a
estratégia de busca <span style="font-variant:small-caps;">Eager</span>
na entidade sejam carregados.

Analisando o exposto até então, poderia-se sugerir como recomendação de
refatoração o uso de s. Todavia, o uso do padrão para este contexto não
é um consenso. Apesar das vantagens dessa abordagem, para (Bauer and
King 2005) utilizar s com objetivo de recuperar informação do banco de
dados ao invés da própria entidade poderia causar outras situações
indesejadas. Segundo o autor, o uso do padrão , neste contexto, poderia
adicionar dois *code smells* elencados por (Fowler 2018):

-   *Shotgun change smell*: Uma pequena alteração em alguma parte do
    código requer alterações em várias classes. No caso do , se a classe
    da entidade for alterada, os s que utilizam atributos daquela
    entidade também deverão ser alterados.

-   *Parallel class hierarchies*: Duas hierarquias de classes distintas
    contêm classes semelhantes em uma correspondência de um para um.
    Nesse caso, fica evidente a hierarquia de classes paralelas.
    Utilizando , teríamos por exemplo: `Usuario` e `UsuarioDTO`,
    `Discente` e `DiscenteDTO`.

Entretanto, ao analisar apenas o aspecto de desempenho, o pode
apresentar uma superioridade por não ser gerenciado pelo contexto do
JPA. Segundo (Janssen 2019a), o fato das entidades serem gerenciadas
pelo JPA faz com que sejam armazenadas no cache de primeiro nível. Isso
evita instruções repetidas e otimiza gravações no banco de dados. Porém,
o gerenciamento de cache de primeiro nível leva tempo e pode ser um
problema se executadas grandes quantidades de consultas em entidades.

A Figura abaixo apresenta o comparativo de tempo na
execução de instruções utilizando entidade *versus* .

Perante o exposto, podemos catalogar como *code smells* consultas ORM
que não utilizam projeção e são utilizadas somente para consulta, pois
podem gerar problemas futuros de desempenho e manutenibilidade pelo
excesso de colunas desnecessárias. Consideramos como *code smell* apenas
consultas ORM declaradas, pois consultas usando o `find` do
`EntityManager` ou geradas automaticamente a partir do nome do método
são muito comuns e geralmente menos problemáticas por utilizar apenas
uma entidade. No caso de um objeto de retorno ser atualizável, também
não deve ser considerado como *code smell*. Como recomendação, podemos
citar a utilização de projeções utilizando a própria classe da entidade
ou s, deixando o desenvolvedor escolher a melhor estratégia para cada
contexto.

### Atualização desnecessária de toda a entidade

#### Descrição do *Smell*

Atualização de todas as colunas de uma entidade, quando somente um
atributo do objeto é atualizado, acarreta em atualização de dados em
excesso e pode gerar problemas de desempenho, especialmente quando a
entidade possui índices não clusterizados, dados grandes binários ou um
grande número de colunas. O Exemplo abaixo demonstra
o *smell* no qual é desejado atualizar apenas o campo `observação` da
entidade `Discente`, mas a instrução `Update` gerada pelo ORM atualiza
todas as colunas da entidade, incluindo índices e tipo <span
style="font-variant:small-caps;">BLOB</span>.
  
  **Código Java:**
```java
    @Entity
    class Discente{
    ... 
     @Column
     @Index
     private Long matricula;
    	
     @Column
     private UploadedFile arquivo;
     ...
    }

    public static void main(String[] args) {
     ...
     Discente d = findDiscenteById(1);
     d.setObservacao("Discente com pendencias");
     entityManager.persist(d);
    }
 ```       

  **Instrução SQL gerada:**
```sql
    UPDATE Discente
    SET    id = 1, matricula = 111, observacao = "Discente com pendencias", 
           arquivo = "arquivo BLOB" 
    WHERE  id = 1;

```
  ---------------------------

#### Sugestão de Refatoração

Sugere-se utilizar parametrizações nos *frameworks* ORM para que
realizem atualização apenas das alterações realizadas. No *Hibernate*
por exemplo é possível utilizar a anotação `@DynamicUpdate` ou realizar
instruções ORM (HQL/JPQL ou Criteria) para atualizar somente as colunas
desejadas. Alguns *frameworks* como o EclipseLink apresentam esta
parametrização por padrão. O Exemplo abaixo apresenta a utilização da anotação
`@DynamicUpdate` fazendo com que seja atualizado apenas o atributo
`observacao` na instrução `Update` gerada pelo ORM.
  
  **Código Java:**
```java
    @Entity
    @DynamicUpdate
    class Discente{
    ... 
     @Column
     @Index
     private Long matricula;

     @Column
     private UploadedFile arquivo;
     ...
    }

    public static void main(String[] args) {
     ...
     Discente d = findDiscenteById(1);
     d.setObservacao("Discente com pendencias");
     entityManager.persist(d);
    }
```
  **Instrução SQL gerada:**
```sql
    UPDATE discente SET observacao = "Discente com pendências" WHERE id = 1;
```
  ---------------------------

#### Detalhamento e Discussão

Com relação a atualização de todos os campos da entidade, existe uma
distinção dependendo do *framework* ORM utilizado. Utilizando o
Hibernate, é atualizado o objeto inteiro por padrão, mesmo que a
atualização seja realizada apenas em um atributo. Isso ocasiona
processamento desnecessário ao banco de dados, principalmente em
entidades fortemente indexadas e com grande número de colunas (Chen,
Shang, Jiang, et al. 2016). O mesmo não se aplica ao padrão do
EclipseLink, que realiza a atualização somente dos campos modificados.
No EclipseLink a atualização de todos os campos somente será realizada
de forma explicita pelo programador através de SQL ou JPQL (EclipseLink
2017). Porém, para (Mihalcea et al. 2018), o fato do Hibernate realizar
a atualização com todos os atributos possui algumas vantagens:

-   Permite que sejam utilizados recursos de cache;

-   Permite a atualização em lotes, mesmo se várias entidades
    modificarem propriedades diferentes.

Mesmo com estes fatores, quando a entidade possui índices não
clusterizados, os quais o banco de dados poderá atualizá-los de forma
redundante, recomenda-se não atualizar todos os campos (Chen, Shang,
Yang, et al. 2016). Para isso, o Hibernate disponibiliza uma anotação a
nível de classe chamada `@DynamicUpdate`. Esta anotação faz com que
sempre seja atualizado apenas o campo alterado na entidade representada
pela classe (Mihalcea et al. 2018).

Com base no exposto, podemos considerar *code smell* quando possuir as
seguintes premissas: o *framework* ORM em uso utiliza a atualização de
toda entidade por padrão; a classe da entidade não possuir configuração
para atualização apenas dos atributos alterados, como `@DynamicUpdate`
do Hibernate; possuir índices não clusterizados relacionados a entidade.
Nesse caso, conforme exposto na introdução deste capítulo referente a
definição de um *code smell* ORM, indica uma necessidade de refatoração
sendo uma prática não recomendada.

### Falta de paginação quando não necessário todo o resultado

#### Descrição do *Smell*

Recuperar entidades completas do banco de dados sem especificar
parâmetros de paginação quando os registros não são totalmente
utilizados, implica em recuperação de dados em excesso causando uma
sobrecarga desnecessária, além de utilizar um alto consumo de memória
para armazenar a informação recuperada gerando problemas de desempenho.
O Exemplo abaixo apresenta a ocorrência do *Smell*
recuperando todos os registros da entidade `Discente` que entraram no
ano de 2020, mas utilizando efetivamente apenas 10 registros que foram
solicitados pela camada de visão, supondo que o método
`discentesPeriodo` foi chamado com este propósito.
  
  **Código Java:**
```java
    public List<Discente> findByYear(int year){
     String hql = "FROM Discente d WHERE ano = :ano";
     Query q = entityManager.createQuery(hql.toString());
     ...
    }

     public List<Discente> discentesPeriodo(int year,int page,int limit) {
     int fromIndex = (page - 1) * limit;
     List<Discente> discentes = findDiscenteByYear(year);
     return discentes.subList(fromIndex, Math.min(fromIndex + limit, discentes.size()));    
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT d.* FROM Discente d WHERE ano = 2020;
```
  ---------------------------

#### Sugestão de Refatoração

A especificação JPA disponibiliza os seguintes métodos:
`setFirstResult(int)` e `setMaxResults(int)` fornecidos pelas interfaces
`Query` ou `TypedQuery` provendo recursos de paginação que podem ser
utilizados ao realizar uma consulta ORM. Desta forma é possível realizar
uma limitação nos resultados que solicitam uma coleção de dados. O
Exemplo abaixo apresenta refatoração do
exemplo anterior utilizando uma consulta paginada
para gerar a instrução SQL com limites para o resultado, recuperando
apenas os 10 registros da entidade `Discente` solicitados pela camada de
visão, evitando problemas de desempenho com o crescimento dos dados.
  
  **Código Java:**
```java
    public List<Discente> findByYear(int year, int fromIndex, int limit){
     String hql = "FROM Discente d WHERE ano = :ano";
     Query q = entityManager.createQuery(hql.toString());
     q.setFirstResult(fromIndex);
     q.setMaxResults(limit);
     ....
    }

    public List<Discente> discentePeriodo(int year,int page,int limit) {
     int fromIndex = (page - 1) * limit;
     return List<Discente> discentes = findDiscenteByYear(year,fromIndex,limit);
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT d.* FROM Discente d WHERE ano = 2020 LIMIT 10 OFFSET 10;
```
  ---------------------------

#### Detalhamento e Discussão

Segundo (Vlad Mihalcea 2019a) para obter o melhor resultado nas
instruções SQL geradas pelo ORM, é importante não apenas considerar o
número de colunas, consultas e junções realizados, mas também o número
de registros que esta sendo recuperado do banco de dados. Para recuperar
apenas um conjunto específico de dados, é possível configurar elementos
como a paginação especificando um limite para quantidade de registros na
execução de uma consulta (Mihalcea 2017) através dos seguintes métodos
disponibilizados pela especificação JPA (Gomes 2016):

-   `setFirstResult(int)`: Parâmetro utilizado para especificar o
    deslocamento;

-   `setMaxResults(int)`: Parâmetro utilizado para especificar o limite.

Na camada de visualização, as informações geralmente são apresentadas ao
usuário em um formulário paginado, apresentando uma relação natural com
a parametrização referente a paginação de dados ORM (Medeiros 2015).
Exemplo: Uma entidade possui 1.000.000 registros. Em uma pagina web é
apresentado um subconjunto de informações exibindo 100 registros.
Portanto os parâmetros `setFirstResult(1)` e `setMaxResults(100)` devem
ser utilizados para recuperar apenas os registros utilizados. Outro
agravante em não utilizar a paginação, é que os dados tendem a crescer
com o tempo (Vlad Mihalcea 2019a). Em um primeiro momento pode não
apresentar problemas de desempenho mas com o aumento de dados pode se
tornar um grande problema. Também dificulta a manutenção do código, pois
o ajuste implica em alterações na estrutura dos métodos conforme
exemplo na seção de sugestão de refatoração.

Com base no exposto, podemos considerar um *code smell* ORM a não
utilização de paginação em coleções de dados quando os registros
recuperados não serão integralmente utilizados, sendo uma má escolha de
implementação, cabível de refatoração por indicar possíveis problemas
futuros relativos a desempenho com o crescimento da quantidade de
registros.

## Problema do N+1
---------------

Os *frameworks* ORM abstraem o SQL e o mapeamento manual dos objetos,
simplificando o desenvolvimento especialmente em operações de criação,
consulta, atualização e destruição (CRUD). Mas isso tem um preço, e
conforme a estratégia de busca adotada em cada situação, o *framework*
ORM pode precisar realizar mais consultas a partir de uma consulta para
completar a informação de seus relacionamentos e montar o objeto
desejado e seus relacionamentos. Essa ação pode causar um problema de
desempenho dependendo da quantidade de consultas que são realizadas para
completar a operação (Vlad Mihalcea 2019d). Esse problema é conhecido
como `N+1`, quando a partir de uma instrução ORM são geradas <span
style="font-variant:small-caps;">N</span> outras, sendo apresentado no
Capítulo \[chap:intro\] como exemplo motivacional desta pequisa. A
Figura abaixo apresenta um exemplo no qual ao
solicitar uma lista do objeto `Pessoa`, a partir da consulta principal
(`select * from pessoa`) são geradas <span
style="font-variant:small-caps;">N</span> outras consultas para obter o
relacionamento com a entidade `discente`.


<p align="center"> 
<img src="figures/n1-representacao.png">
</p>


Através da análise do código, podemos detectar *code smells* que podem
gerar o problema de `N+1`, os quais são descritos a seguir.

### Falta de <span style="font-variant:small-caps;">JOIN FETCH</span> nas consultas de atributos do tipo <span style="font-variant:small-caps;">Eager</span>

#### Descrição do *Smell*

Consultas ORM utilizando objetos que contenham atributos de
relacionamento com a estratégia de busca do tipo <span
style="font-variant:small-caps;">Eager</span> e sem realizar o <span
style="font-variant:small-caps;">JOIN FETCH</span> para o objeto
relacionado, faz com que o *framework* ORM realize $N$ consultas até que
todos os objetos associados aos atributos do tipo <span
style="font-variant:small-caps;">Eager</span> fiquem com os dados
completos gerando o problema do `N+1`. O exemplo abaixo apresenta um código no qual uma consulta
para retornar uma lista da entidade `Discente` apresenta $N$ consultas
adicionais para entidade `Pessoa` realizadas pelo ORM para preencher as
informações do objeto `Pessoa` que esta com relacionamento <span
style="font-variant:small-caps;">Eager</span>.
  
  **Código Java:**
```java
    @Entity
    class Discente{
     ...
     @ManyToOne(fetch = FetchType.EAGER)
     private Pessoa pessoa;	
     ...
    }
    public List<Discente> findDiscentesByYear(int year){
     String hql = "FROM Discente d WHERE d.ano = :ano";
     Query q = entityManager.createQuery(hql.toString());
     q.setInteger("ano", year);
     return (List<Discente>) q.getResultList();
    }

    public static void main(String[] args) {
     List<Discente> d = findDiscenteByYear(2020);
     ...
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT * FROM Discente d WHERE   d.ano=2020;
    -- Consultas adicionais geradas: 
    SELECT * FROM Pessoa p where p.id_pessoa = 1;
    SELECT * FROM Pessoa p where p.id_pessoa = 2;
    SELECT * FROM Pessoa p where p.id_pessoa = 3;
    ....
```
  ---------------------------

#### Sugestão de Refatoração 

Para corrigir, uma das formas possíveis seria alterar o mapeamento para
<span style="font-variant:small-caps;">Lazy</span>. Porém, se não for
possível alterar o tipo de estratégia na classe devido a dependências,
uma possibilidade de refatoração para evitar o `N+1` seria realizar
<span style="font-variant:small-caps;">JOIN FETCH</span> para todos os
atributos <span style="font-variant:small-caps;">Eager</span> na
consulta ORM. Desta forma o *framework* ORM fará junções para trazer as
informações dos relacionamentos em uma única consulta em vez de gerar
$N$ consultas extras. O exemplo abaixo apresenta a correção para o
problema de N+1 gerado pelo exemplo anterior
adicionando <span style="font-variant:small-caps;">JOIN FETCH</span>
para recuperar em forma de junção a entidade `Pessoa`.
  
  **Código Java:**
```java
    @Entity
    class Discente{
     ...
     @ManyToOne(fetch = FetchType.EAGER)
     private Pessoa pessoa;	
     ...
    }
    public List<Discente> findDiscentesByYear(int year){
     ...
     hql.append("FROM Discente d ");
     hql.append("JOIN FETCH d.pessoa p WHERE d.ano = :ano");
     Query q = entityManager.createQuery(hql.toString());
     q.setInteger("ano", year);
     return (List<Discente>)  q.getResultList();
    }

    public static void main(String[] args) {
     List<Discente> d = findDiscenteByYear(2020);
     ...
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT * FROM Discente d INNER JOIN Pessoa p
    ON     p.id_pessoa = d.id_pessoa WHERE d.ano=2020;
```
  ---------------------------

#### Detalhamento e Discussão

Utilizar EAGER como estratégia de busca para relacionamentos é um *code
smell* como visto na Seção \[subsection:EAGER\_ESTATICO\], portanto a
melhor abordagem é evitar o uso de <span
style="font-variant:small-caps;">Eager</span> como estratégia de busca a
nível de classe. Porém para casos onde se deseja realizar consultas ORM
e seja complexo a refatoração no código ao alterar a estratégia de busca
(de <span style="font-variant:small-caps;">Eager</span> para <span
style="font-variant:small-caps;">Lazy</span>) na classe devido a
dependências, é necessário utilizar a cláusula <span
style="font-variant:small-caps;">JOIN FETCH</span> para todos os
atributos do EAGER para evitar o problema N + 1 (Vlad Mihalcea 2019a). O
problema de dependência mencionado ocorre quando o código existente
espera que o objeto relacionado já esteja carregado. A alteração para
<span style="font-variant:small-caps;">Lazy</span> pode causar exceções,
como `LAZYInitializationException` no caso do Hibernate (Mihalcea et al.
2018). Existem opções boas e ruins de tratar essa exceção. Utilizar o
padrão *Open Session in View* ou ativar a opção
`hibernate.enable_lazy_load_no_trans` são consideradas anti-padrões e
portanto opções ruins para evitar a exceção (Vlad Mihalcea 2020b). Esses
são considerados anti-padrões pois tratam apenas os sintomas e não
resolvem a causa real do problema  (Vlad Mihalcea 2019c). A melhor
opção, nesse caso, é buscar todos os relacionamentos necessários antes
de fechar o contexto de persistência por meio da cláusula <span
style="font-variant:small-caps;">JOIN FETCH</span> (Mihalcea et al.
2018). Por esse motivo, para evitar o `N+1`, a solução recomendada
quando não possível realizar a refatoração de <span
style="font-variant:small-caps;">Eager</span> para <span
style="font-variant:small-caps;">Lazy</span> é utilizar <span
style="font-variant:small-caps;">JOIN FETCH</span> nas consultas ORM
conforme recomendado na Seção \[subsection:refat\].

Com relação a refatoração apresentada utilizando <span
style="font-variant:small-caps;">JOIN FETCH</span> é necessário um
cuidado e testes após alteração, pois dependendo do contexto o número de
junções também pode afetar o desempenho (Bauer, King, and Gregory 2016),
cabendo uma análise do desenvolvedor para avaliar qual a melhor opção.
Existem diferenças entre realizar um <span
style="font-variant:small-caps;">JOIN</span> comum e um <span
style="font-variant:small-caps;">JOIN FETCH</span>. Segundo (Janssen
2020) a palavra <span style="font-variant:small-caps;">FETCH</span> da
instrução <span style="font-variant:small-caps;">JOIN FETCH</span> é
específica da JPA. Ela informa ao provedor de persistência para
inicializar também a associação no objeto recuperado e não somente
realizar a junção entre as duas entidades. 

<!-- Para exemplificar de forma
prática o exposto até o momento, no capítulo \[chap:intro\] utilizamos
como motivação desta pesquisa um problema ocasionado pelo HQL do
[Exemplo \[alg:java\]]{}, e pelo mapeamento realizado no [Exemplo
\[alg:sig\_eager\]]{}, que gerava o problema `N+1` no sistema do . Foram
realizadas duas refatorações que são detalhadas nesta seção utilizando
as formas descritas no parágrafo anterior:

-   Refatoração 1 - Ajuste das $N$ consultas na entidade `Pessoa`: Uma
    das soluções possíveis seria alterar as estratégias de busca do tipo
    <span style="font-variant:small-caps;">Eager</span> para <span
    style="font-variant:small-caps;">Lazy</span> em ambos os casos do
    [Exemplo \[alg:sig\_eager\]]{}, de forma que o ORM buscasse as
    informações somente quando solicitadas. Seria a melhor estratégia na
    fase de concepção do sistema, pois atualmente existem centenas de
    casos de uso que utilizam a entidade `DiscenteGraduacao`, dos quais
    muitos deles esperam que a informação de *Discente* e *Pessoa* já
    estejam carregadas. Para modificar a estratégia de busca seria
    necessária uma grande refatoração em todos os pontos do código que
    as utilizam, tendo em vista que uma chamada ao atributo
    (`discenteGraduacao.getDiscente().getPessoa()`) em um momento que a
    sessão que carregou o objeto estiver fechada, iria gerar a exceção
    `LazyInitializationException`. Isso também evidencia o problema de
    manutenção causado pelo *smell*. Com base nesses fatores, optou-se
    por adicionar *JOIN FECTH* na consulta , para continuar recuperando
    a informação de `Pessoa` mas evitando $N$ consultas extras,
    utilizando *JOIN* para trazer as informações da entidade `Pessoa` já
    na consulta principal. Após esta refatoração, a quantidade de
    consultas adicionais diminuiu de 6767 para 116.

    ![[]{data-label="fig:fetch"}](figures/fetch.png)

-   Refatoração 2 - Ajuste da estratégia de busca: Observou-se que as
    116 consultas adicionais que restaram eram referentes a atributos
    <span style="font-variant:small-caps;">Eager</span> de
    `DiscenteGraduacao` referentes as entidades relacionadas `Polo`,
    `TipoCotaDiscente` e `TipoCancelamento`. Observado no código que não
    possuíam muitos casos de usos que aguardavam essas informações de
    forma <span style="font-variant:small-caps;">Eager</span>, sendo
    possível realizar a alteração para <span
    style="font-variant:small-caps;">Lazy</span> ajustando alguns poucos
    casos de usos. Após essa nova refatoração (Figura \[fig:eager\]), o
    tempo de execução da consulta diminuiu de 14 segundos para 11
    segundos em média e a quantidade de consultas adiciona diminuiu de
    116 para 0.

    ![[]{data-label="fig:eager"}](figures/eager.png) -->

Conforme as situações expostas, consultas ORM utilizando entidades que
contenham atributos com a estratégia de busca do tipo <span
style="font-variant:small-caps;">Eager</span> e sem realizar o <span
style="font-variant:small-caps;">JOIN FETCH</span> para a entidade
relacionada, poderão causar `N+1`, sendo uma prática não recomendada que
indica necessidade de refatoração com potencial para gerar problemas de
desempenho e manutenibilidade, podendo ser catalogado como um *code
smell ORM*.

### Acesso um por um: estrutura de repetição e estratégia de busca do tipo <span style="font-variant:small-caps;">Lazy</span> 

#### Descrição do *Smell*

Quando necessário capturar determinado atributo em uma estrutura de
coleção de objetos utilizando alguma estrutura de repetição, cujo
atributo está com a estratégia de busca do tipo <span
style="font-variant:small-caps;">Lazy</span>, em cada passagem pela
estrutura de repetição serão geradas consultas adicionais. Essa questão
é melhor visualizada a partir do exemplo abaixo que
demonstra uma consulta para recuperação de uma lista do objeto `Pessoa`
e ao acessar o atributo `discentes` em uma estrutura de repetição, são
geradas consultas extras para carregar o relacionamento <span
style="font-variant:small-caps;">Lazy</span>.
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
     ...
     @OneToMany(fetch = FetchType.LAZY)
     private List <Discente> discentes;	
     ...
    }

    public List<Pessoa> findAll(){
     String hql = "FROM Pessoa pes";
     ...
    }

    public static void main(String[] args) {
     List<Pessoa> pessoas = findAll();
     for (Pessoa pessoa : pessoas){
      assertNotNull(pessoa.getDiscentes());
     }
    }
```
  **Instrução SQL gerada:**
```sql
    SELECT * FROM Pessoa p;
    -- Consultas adicionais geradas:
    SELECT * FROM Discente d where d.id_pessoa = 1
    SELECT * FROM Discente d where d.id_pessoa = 2
    SELECT * FROM Discente d where d.id_Pessoa = 3
    ....
```
  ---------------------------

#### Sugestão de Refatoração 

Abaixo listamos duas soluções possíveis para evitar `N+1` neste caso:

-   `@BatchSize(N)`: utilizar a anotação `@BatchSize` sobre o atributo
    do tipo <span style="font-variant:small-caps;">Lazy</span>. Dessa
    forma em vez de gerar $N$ consultas, o *framework* ORM utilizará o
    operador `IN` do SQL. Poderá ser definido o número de elementos como
    parâmetro, definindo o tamanho para o carregamento em lote de
    coleções ou entidades. O exemplo abaixo apresenta a refatoração do
    exemplo anterior utilizando a anotação
    `@BatchSize`
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
      ...
      @OneToMany(fetch = FetchType.LAZY)
      @BatchSize(size = 5)
      private List <Discente> discentes;	
      ...
    }

    public static void main(String[] args) {
      List<Pessoa> pessoas = findAll();
      for (Pessoa pessoa : pessoas){
      assertNotNull(pessoa.getDiscentes());
      }
    }
```
 **Instrução SQL gerada:**
```sql
    SELECT * FROM Pessoa p;
    -- Consultas adicionais geradas:
    SELECT * FROM Discente d where d.id_pessoa in (1,2,3,4,5)
    SELECT * FROM Discente d where d.id_pessoa in (6,7,8,9,10)
    ....
```
  ---------------------------

-   <span style="font-variant:small-caps;">JOIN FETCH</span> ou
    projeções DTO: embora utilizar a anotação `@BatchSize` seja melhor
    do que carregar os registros um a um, utilizar <span
    style="font-variant:small-caps;">JOIN FETCH</span> para carregar
    antecipadamente os objetos ou utilizar s pode ser uma estratégia
    melhor, sendo possível obter todos os dados em uma única consulta. O
    exemplo abaixo demonstra a
    utilização de <span style="font-variant:small-caps;">JOIN
    FETCH</span> como refatoração para o exemplo anterior apresentado na Seção Descrição do Smell.
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
      ...
      @OneToMany(fetch = FetchType.LAZY)
      private List <Discente> discentes;	
      ...
    }

    public List<Pessoa> findAll(){
      String hql = "FROM Pessoa pes JOIN FETCH pes.discentes";
      ...
    }

    public static void main(String[] args) {
      List<Pessoa> pessoas = findAll();
      for (Pessoa pessoa : pessoas){
        assertNotNull(pessoa.getDiscentes());
      }
    }
```
**Instrução SQL gerada:**
```sql
    SELECT * FROM Pessoa p INNER JOIN Discente d
    ON     p.id_pessoa = d.id_pessoa;
```
 ---------------------------

#### Detalhamento e Discussão

Por motivos de desempenho, conforme exposto na Seção
\[fetchtypePadrao\], a especificação JPA utiliza as coleções
`@OneToMany` e `@ManyToMany` com estratégia de busca <span
style="font-variant:small-caps;">Lazy</span> por padrão. Isso significa,
segundo (Hut 2015), que o contexto de persistência não carregará a
entidade de relacionamento até que uma chamada seja feita pelo código.
Entretanto, quando o código solicita os dados, a transação já foi
encerrada. Sem a possibilidade de junção, o *framework* ORM realiza
consultas adicionais para recuperar a informação (Mihalcea et al. 2018),
o que em uma estrutura de repetição gera um número de consultas em
excesso prejudicando o desempenho do sistema. Conforme (Marciniec 2019),
uma das estratégias de correção muito utilizada e facilmente encontrada
no StackOverFlow é a alteração para a estratégia de busca <span
style="font-variant:small-caps;">Eager</span> a nível de classe. Porém,
esta resolução substitui um problema por outro, mostrando a dificuldade
de manutenção causada pelo *smell*. Conforme comentado na Seção
\[subsection:EAGER\_ESTATICO\], utilizar <span
style="font-variant:small-caps;">Eager</span> a nível de classe poderá
gerar o problema de dados em excesso, carregando mais dados do que o
necessário, sendo uma melhor estratégia seguir as refatorações sugeridas
na Seção \[refatoracao\_lazy\].

Podemos concluir que o acesso um por um utilizando uma estrutura de
repetição com a estratégia de busca do tipo <span
style="font-variant:small-caps;">Lazy</span> é um *code smell* a ser
catalogado pelos motivos expostos. As seguintes condições se aplicam
para ser considerado um *code smell* ORM desta categoria: estar em uma
estrutura de repetição com acesso a um atributo com estratégia <span
style="font-variant:small-caps;">Lazy</span>; não ser carregado de forma
antecipada na consulta (*JOIN FECTH*); não conter a anotação
`@BatchSize` no atributo do relacionamento.

### `@OneToMany` unilateral com uso inadequado de coleções 

#### Descrição do *Smell*

Utilizar a anotação `@OneToMany` de forma unilateral (sem o
correspondente `@ManyToOne` no outro lado da relação) com o uso
inadequado de coleções, como `Collection` ou `List`, poderá gerar o
problema de N+1 quando uma alteração em um elemento de uma coleção faz
com que todos os registros relacionados sejam removidos e adicionados
novamente, dependendo da semântica utilizada. O exemplo abaixo demonstra a ocorrência deste *smell*. Neste
exemplo, ao adicionar um novo `Discente` na coleção `discentes` do
objeto `Pessoa` (considera-se que existem registros pré-existentes de
discentes na entidade representativa da relação `Pessoa_Discente` na
base de dados), são geradas instruções adicionais, removendo todos os
registros existentes da entidade `Pessoa_Discente` e adicionando-os
novamente juntamente com o novo registro.
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
     ...
     @OneToMany(cascade = CascadeType.ALL)
     private List<Discente> discentes = new ArrayList<Discente>();	
    }
    @Entity
    class Discente{
     @Id
     private Integer id;
     private Integer matricula;
    }

    public static void main(String[] args) {
     Pessoa pessoa = findById(1);
     Discente discente = new Discente(5,12345);
     pessoa1.add(discente);
     entityManager.persist(pessoa);
    }
```
  **Instrução SQL gerada:**
```sql
    INSERT INTO Discente (id, matricula) VALUES (5,12345);
    DELETE FROM Pessoa_Discente WHERE pessoa_id=1;
    INSERT INTO Pessoa_Discente (pessoa_id, discente_id) VALUES (1, 1);
    INSERT INTO Pessoa_Discente (pessoa_id, discente_id) VALUES (1, 2);
    ...
    INSERT INTO Pessoa_Discente (pessoa_id, discente_id) VALUES (1, 5);
```
  ---------------------------

#### Sugestão de Refatoração

Existem duas formas possíveis de ajustar este *code smell*:

-   Transformar em bidirecional: A melhor solução para este *smell* é
    transformar a relação em bidirecional, adicionando um atributo
    relacionado ao objeto no outro lado da relação com a anotação
    `@ManyToOne` informando o atributo da relação com o parâmetro
    `mappedBy` no `@OneToMany` (Vlad Mihalcea 2016a). Assim, segundo
    (Mihalcea et al. 2018) o *framework* ORM irá utilizar o lado com a
    anotação `@ManyToOne` da relação bidirecional sempre que um dos
    lados forem manipulados. O exemplo abaixo demonstra o ajuste realizado
    no exemplo anterior, adicionando a anotação
    `@ManyToOne`, o atributo responsável pelo relacionamento na classe
    `Discente` e informando através da anotação `mappedBy` na classe
    `Pessoa`. Além das instruções geradas pelo ORM serem otimizadas, a
    estrutura das entidades é modificada ficando a entidade `Discente`
    com a chave estrangeira relacionado a Pessoa, deixando de ter uma
    tabela adicional para representar a relação.
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
      ...
      @OneToMany(cascade = CascadeType.ALL, mappedBy = "pessoa")
      private List<Discente> discentes = new ArrayList<Discente>();	
    }
    @Entity
    class Discente{
      @Id
      private Integer id;
      private Integer matricula;
      @ManyToOne(fetch = FetchType.LAZY)
      private Pessoa pessoa;
    }

    public static void main(String[] args) {
      Pessoa pessoa = findById(1);
      Discente discente = new Discente(5,12345,pessoa);
      pessoa1.add(discente);
      entityManager.persist(pessoa);
    }
```
  **Instrução SQL gerada:**
```sql
    INSERT INTO Discente (id, matricula, pessoa) VALUES (5,12345,1);
```
 ---------------------------

-   Utilizar a coleção `Set`: Caso não seja possível ou desejável ter
    uma relação bidirecional, sugere-se trocar o tipo da coleção para a
    coleção `SET`. Esta coleção tem como característica não armazenar
    valores duplicados, fazendo com que *framework* ORM altere no banco
    de dados apenas o registro modificado na coleção. O exemplo abaixo demonstra a refatoração
    realizada no exemplo anterior apresentado na Seção Descrição do Smell trocando a coleção
    `List` por `Set`. Desta forma o relacionamento continua sendo
    representado pela entidade `Pessoa_Discente`, mas a instrução gerada
    pelo ORM é igualmente proporcional a alteração realizada nos
    objetos, inserindo apenas o novo registro.
  
  **Código Java:**
```java
    @Entity
    class Pessoa{
      ...
      @OneToMany(cascade = CascadeType.ALL)
      private Set<Discente> discentes = new HashSet<Discente>();	
    }
    @Entity
    class Discente{
      @Id
      private Integer id;
      private Integer matricula;
    }

    public static void main(String[] args) {
      Pessoa pessoa = findById(1);
      Discente discente = new Discente(5,12345);
      pessoa1.add(discente);
      entityManager.persist(pessoa);
    }
```
  **Instrução SQL gerada:**
```sql
    INSERT INTO Discente (id, matricula) VALUES (5,12345);
    INSERT INTO Pessoa_Discente (pessoa_id, discente_id) VALUES (1, 5);
```
  ---------------------------

#### Detalhamento e Discussão

De acordo com (Węgrzynowicz 2013), a anotação `@OneToMany` em JPA
utilizada no lado proprietário (ou seja, unidirecional de forma a
gerenciar a persistência) é uma das formas mais comuns de representar
associações um-para-muitos entre classes que representam entidades
persistentes. A Tabela \[tab:onetomany1\] apresenta três semânticas de
coleções utilizadas pelo ORM com a combinação do tipo de coleção Java e
anotações do JPA.

|Semânticas |Tipo Java |Anotações |
|-----------|----------|-----------|
|Semântica `Bag` | `java.util.Collection` `java.util.List` |`@OneToMany`|
|Semântica `List` | `java.util.List` | `@OneToMany` | (`@IndexColumn` v `@OrderColumn`)|
Semântica `Set` | `java.util.Set` | `@OneToMany`|
</br>

O número e o tipo de instrução executada pelo ORM muda de acordo com a
semântica utilizada. A Tabela \[tab:onetomany2\] demonstra o número de
instruções geradas ao persistir uma coleção após uma inserção ou remoção
dada a semântica utilizada. (Węgrzynowicz 2013)

|Semânticas |Adicionando um elemento | Removendo um elemento|
|-----------|----------|-----------|
Semântica `Bag` | 1 remoção, N inserções | 1 remoção, N inserções|
Semântica `List` | 1 inserção, N atualizações | 1 remoção, N atualizações|
Semântica `Set` | 1 inserção | 1 remoção|
</br>

Segundo (Vlad Mihalcea 2016a), a utilização do tipo `Set` para
representar uma coleção unilateral é a forma mais eficiente, seguido
pelo tipo `List` com anotações de ordenação (`@IndexColumn` ou
`@OrderColumn`), enquanto tipos da semântica `Bag` representadas por
coleções não ordenadas são menos eficiente. Além disso, o fato de
existir suporte para coleções em ORM não significa que todo
relacionamento de um para muitos em banco de dados deve ser transformado
para coleção, algumas vezes uma associação muitos para um (`@ManyToOne`)
é suficiente e a coleção pode ser substituída por uma consulta simples
na entidade. A utilização de `Set` de acordo com a tabela
\[tab:onetomany2\] demonstra ser mais eficiente, pois uma operação na
coleção requer apenas uma única instrução no banco de dados evitando o
<span style="font-variant:small-caps;">N+1</span>. Entretanto, para
(Węgrzynowicz 2013) ela não é uma exclusividade e existem alguns casos
em que utilizar outros tipos pode ser mais eficaz. Como o uso de `Set`
em Java requer uma verificação de exclusividade nos elementos, no caso
de qualquer inserção de novos elementos, todos os elementos da coleção
devem ser carregados na memória principal. Sendo assim, casos em que a
coleção na maioria das vezes é fortemente modificada com muitas
alterações de uma vez, a utilização da semântica `Bag` ou `List` pode se
tornar mais eficaz. Porém, para a maioria dos casos `Set` é mais
eficiente.

A utilização de coleções em relacionamentos `@OneToMany` unilaterais que
utilizam a semântica `Bag` ou `List` (conforme Tabela
\[tab:onetomany1\]) é considerado *Code Smell* por indicar problemas de
desempenho e manutenção de código quando necessária refatoração devido
ao aumento de registros na entidade que é representada pela coleção. A
troca para utilização de relacionamento bidirecional ou a utilização de
são estratégias mais eficientes por executar apenas a instrução da
modificação da coleção evitando <span
style="font-variant:small-caps;">N+1</span>.

Outros *code smells* ORM
------------------------

### Não uso de consultas somente leitura

#### Descrição do *Smell*

Por padrão todas as consultas a entidades em JPA são do tipo <span
style="font-variant:small-caps;">READ-WRITE</span> (leitura-gravação) e
são gerenciadas pelo contexto de persistência. Recuperar objetos da base
de dados utilizando o próprio objeto da entidade para fins únicos de
consulta e sem informar ao *framework* ORM que o objeto recuperado não
necessita ser gerenciado pelo contexto de persistência, faz com que a
busca seja menos eficiente, ocupando espaço desnecessário na memória ao
guardar o estado do objeto e deixando a fase de liberação do contexto de
persistência mais lenta. O exemplo abaixo
apresenta a ocorrência do *smell* onde se deseja obter para fins únicos
de consulta o objeto `Discente`, mas é recuperado no modo padrão <span
style="font-variant:small-caps;">READ-WRITE</span> do JPA, mantendo o
objeto de retorno `Discente` no contexto de persistência
desnecessariamente.
  
  **Código Java:**
```java
    public Discente findDiscenteById(Integer id){
     String hql = "FROM Discente d WHERE id = :id";
     Query q = entityManager.createQuery(hql.toString(), Discente.class);
     q.setInteger("id", id);
     return q.uniqueResult();
    }
```
#### Sugestão de Refatoração 

Existem diferentes configurações possíveis que podem ser utilizadas para
informar que o objeto recuperado será utilizado somente para consulta,
diferenciando para cada *framework* ORM. Estas configurações podem ser a
nível de classe, sessão ou consulta. Outra opção é a utilização de DTOs
em vez de recuperar a entidade mapeada. O exemplo abaixo demonstra a utilização da
configuração a nível de consulta para o *framework* ORM Hibernate como
refatoração para o exemplo anterior, fazendo
com que o objeto `Discente` não seja gerenciado pelo contexto de
persistência. Na Seção \[sub:detalhamento\_somente\_leitura\] são
detalhadas outras formas possíveis de configuração.
```java
    public Discente findDiscenteById(Integer id){
     String hql = "FROM Discente d WHERE id = :id";
     Query q = entityManager.createQuery(hql.toString(), Discente.class);
     q.setInteger("id", id);
     q.setHint("org.hibernate.readOnly", true)
     return q.uniqueResult();
    }
```
#### Detalhamento e Discussão 

Recuperar entidades no modo somente leitura é mais eficiente do que
buscar entidades em modo de leitura e gravação (Mihalcea et al. 2018). É
possível informar ao *framework* ORM que determinada consulta é do tipo
somente leitura para consultas que retornam objetos representativos de
entidades no banco de dados e que não serão modificados posteriormente.
Como todas as consultas a entidades em JPA são do tipo <span
style="font-variant:small-caps;">READ-WRITE</span> por padrão,
modificações no estado da entidade são traduzidas em comandos de
atualização (<span style="font-variant:small-caps;">SQL UPDATE</span>).
Ao definir que uma consulta é somente leitura, o estado do objeto não
será gerenciado pelo *framework* ORM e, dessa forma, não necessita
detectar modificações na entidade representada pelo objeto. Além disso,
as entidades somente leituras são ignoradas durante o `flush` (Vlad
Mihalcea 2019a). As configurações podem ser realizadas a nível de
entidade, nível de sessão ou a nível de consulta:

-   A nível de entidade: permite que a entidade ou a coleção da entidade
    seja do tipo somente leitura. Qualquer tentativa de salvar nova
    informação na base será lançada uma exceção (Janssen 2017a). Pode
    ser através da anotação `@Immutable` para o Hibernate (Mihalcea et
    al. 2018), conforme exemplo abaixo, ou
    anotação `@ReadyOnly` para o EclipseLink (EclipseLink 2017).
```java
    @Entity(name = "parametros") @Immutable
    public static class Parametros {
      @Id
      private Long id;
      private String nome;}   
```
-   A nível de sessão: É possível transformar a sessão do
    `entityManager` em uma sessão que fará consultas somente leitura
    (Bauer, King, and Gregory 2016). O exemplo abaixo demonstra a utilização no
    código fonte.
```java
    Session session = entityManager.unwrap(Session.class);
    session.setDefaultReadOnly(true);
```
-   A nível de consulta: é possível definir na própria consulta ORM,
    também com distinções entre os *frameworks* ORM. Para o Hibernate, é
    possível realizar a consulta utilizando
    `setHint(org.hibernate.readOnly,true)`.

Portanto, se um objeto representativo de uma entidade for recuperado da
base para fins únicos de consulta a dados e não esteja definido como
somente leitura a nível de entidade, sessão ou consulta será considerado
um *code smell* ORM. Manter o objeto gerenciado pelo *framework* ORM
desnecessariamente pode ser considerado um *code smell* ORM por indicar
uma possibilidade de refatoração por questões de desperdício de recurso
computacional.

<div id="refs" class="references">

<div id="ref-mlr16_jpa_tips">

Ahmed, Sayem. 2018. “JPA Tips: Avoiding the N + 1 Select Problem.”
Disponível em: [
		https://www.javacodegeeks.com/2018/04/jpa-tips-avoiding-the-n-1-select-problem.html](
		https://www.javacodegeeks.com/2018/04/jpa-tips-avoiding-the-n-1-select-problem.html).

</div>

<div id="ref-aniche2018code">

Aniche, Maurício, Gabriele Bavota, Christoph Treude, Marco Aurélio
Gerosa, and Arie van Deursen. 2018a. “Code Smells for
Model-View-Controller Architectures.” *Empirical Software Engineering*.
Springer.

</div>

<div id="ref-Aniche2018">

———. 2018b. “Code Smells for Model-View-Controller Architectures.”
*Empirical Software Engineering* 23 (4): 2121–57.
doi:[10.1007/s10664-017-9540-2](https://doi.org/10.1007/s10664-017-9540-2).

</div>

<div id="ref-4602670">

Ayewah, N., W. Pugh, D. Hovemeyer, J. D. Morgenthaler, and J. Penix.
2008. “Using Static Analysis to Find Bugs.” *IEEE Software* 25 (5):
22–29. doi:[10.1109/MS.2008.130](https://doi.org/10.1109/MS.2008.130).

</div>

<div id="ref-1998">

Barry, D., and T. Stanienda. 1998. “Solving the Java Object Storage
Problem.” *Computer* 31 (11): 33–40.
doi:[10.1109/2.730734](https://doi.org/10.1109/2.730734).

</div>

<div id="ref-bauer2005hibernate">

Bauer, C., and G. King. 2005. *Hibernate in Action*. In Action Series.
Manning. <https://books.google.com.br/books?id=WCmSQgAACAAJ>.

</div>

<div id="ref-bauer2016java">

Bauer, Christian, Gavin King, and Gary Gregory. 2016. *Java Persistence
with Hibernate*. Manning Publications Co.

</div>

<div id="ref-unity">

Borrelli, Antonio, Vittoria Nardone, Giuseppe A. Di Lucca, Gerardo
Canfora, and Massimiliano Di Penta. 2020. “Detecting Video Game-Specific
Bad Smells in Unity Projects.” In *17th International Conference on
Mining Software Repositories (Msr ’20)*. Seoul, Republic of Korea:
Association for Computing Machinery.

</div>

<div id="ref-badsmellpadroesprojeto">

Bouhours, Cédric, Hervé Leblanc, and Christian Percebois. 2009. “Bad
smells in design and design patterns.” *The Journal of Object
Technology* 8 (3). Chair of Software Engineering: 43–63.
<https://hal.archives-ouvertes.fr/hal-00522587>.

</div>

<div id="ref-rapidBruno">

Cartaxo, Bruno, Gustavo Pinto, and Sergio Soares. 2018. “The Role of
Rapid Reviews in Supporting Decision-Making in Software Engineering
Practice.” In *Proceedings of the 22nd International Conference on
Evaluation and Assessment in Software Engineering 2018*, 24–34. EASE’18.
New York, NY, USA: Association for Computing Machinery.
doi:[10.1145/3210459.3210462](https://doi.org/10.1145/3210459.3210462).

</div>

<div id="ref-Cartaxo2020rapidreviews">

———. 2020. “Contemporary Empirical Methods in Software Engineering.” In,
first. Springer.

</div>

<div id="ref-evidence_briefing">

Cartaxo, Bruno, Gustavo Pinto, Elton Vieira, and Sergio Soares. 2016.
“Evidence Briefings: Towards a Medium to Transfer Knowledge from
Systematic Reviews to Practitioners.” In *Proceedings of the 10th
Acm/Ieee International Symposium on Empirical Software Engineering and
Measurement*, 57. Association for Computing Machinery.
doi:[10.1145/2961111.2962603](https://doi.org/10.1145/2961111.2962603).

</div>

<div id="ref-chen_2015_improve_quality">

Chen, T. 2015. “Improving the Quality of Large-Scale Database-Centric
Software Systems by Analyzing Database Access Code.” In *2015 31st Ieee
International Conference on Data Engineering Workshops*, 245–49.
doi:[10.1109/ICDEW.2015.7129584](https://doi.org/10.1109/ICDEW.2015.7129584).

</div>

<div id="ref-Chen:2016:Redundant:Data">

Chen, T., W. Shang, Z. M. Jiang, A. E. Hassan, M. Nasser, and P. Flora.
2016. “Finding and Evaluating the Performance Impact of Redundant Data
Access for Applications That Are Developed Using Object-Relational
Mapping Frameworks.” *IEEE Transactions on Software Engineering* 42
(12): 1148–61.
doi:[10.1109/TSE.2016.2553039](https://doi.org/10.1109/TSE.2016.2553039).

</div>

<div id="ref-Chen_Emparical_Study">

Chen, T., W. Shang, J. Yang, A. E. Hassan, M. W. Godfrey, M. Nasser, and
P. Flora. 2016. “An Empirical Study on the Practice of Maintaining
Object-Relational Mapping Code in Java Systems.” In *2016 Ieee/Acm 13th
Working Conference on Mining Software Repositories (Msr)*, 165–76.
doi:[10.1109/MSR.2016.026](https://doi.org/10.1109/MSR.2016.026).

</div>

<div id="ref-Chen:2014:Performance:Anti-patterns">

Chen, Tse-Hsun, Weiyi Shang, Zhen Ming Jiang, Ahmed E. Hassan, Mohamed
Nasser, and Parminder Flora. 2014. “Detecting Performance Anti-Patterns
for Applications Developed Using Object-Relational Mapping.” In
*Proceedings of the 36th International Conference on Software
Engineering*, 1001–12. ICSE 2014. New York, NY, USA: Association for
Computing Machinery.
doi:[10.1145/2568225.2568259](https://doi.org/10.1145/2568225.2568259).

</div>

<div id="ref-eclipseLinkBestPractices">

Clarke, Doug. 2007. “What Is Object-Relational Mapping.” Disponível em:
<https://wiki.eclipse.org/EclipseLink/FAQ/JPA/BestPractices>.

</div>

<div id="ref-badsmellplanilhas">

Cunha, J., J. P. Fernandes, P. Martins, J. Mendes, and J. Saraiva. 2012.
“SmellSheet Detective: A Tool for Detecting Bad Smells in Spreadsheets.”
In *2012 Ieee Symposium on Visual Languages and Human-Centric Computing
(Vl/Hcc)*, 243–44.
doi:[10.1109/VLHCC.2012.6344535](https://doi.org/10.1109/VLHCC.2012.6344535).

</div>

<div id="ref-badsmellplanilhas2">

Cunha, Jácome, João P. Fernandes, Hugo Ribeiro, and João Saraiva. 2012.
“Towards a Catalog of Spreadsheet Smells.” In *Computational Science and
Its Applications – Iccsa 2012*, edited by Beniamino Murgante, Osvaldo
Gervasi, Sanjay Misra, Nadia Nedjah, Ana Maria A. C. Rocha, David
Taniar, and Bernady O. Apduhan. Berlin, Heidelberg: Springer Berlin
Heidelberg.

</div>

<div id="ref-jpa2006javaDeMichiel">

DeMichiel, Linda, and Michael Keith. 2006. “Java Persistence Api.” *JSR*
220.

</div>

<div id="ref-JpaComparativoFramework">

Dhingra, Neha, Emad Abdelmoghith, and Hussien Mouftah. 2017. “Review on
Jpa Based Orm Data Persistence Framework.” *International Journal of
Computer Theory and Engineering* 9 (January): 318–28.
doi:[10.7763/IJCTE.2017.V9.1160](https://doi.org/10.7763/IJCTE.2017.V9.1160).

</div>

<div id="ref-Spring-Petclinic">

Dubois, Julien. 2013. “Improving the Performance of the Spring-Petclinic
Sample Application (Part 1 of 5).” Disponível em:
[encurtador.com.br/bxIUV](encurtador.com.br/bxIUV).

</div>

<div id="ref-mlr12_bost_performance">

Durix, Hippolyte. 2020. “Boost the Performance of Your Spring Data Jpa
Application.” Disponível em: [
		https://blog.ippon.tech/boost-the-performance-of-your-spring-data-jpa-application/](
		https://blog.ippon.tech/boost-the-performance-of-your-spring-data-jpa-application/).

</div>

<div id="ref-eclipseLinkManual">

EclipseLink. 2013. “Java Persistence Api (Jpa) Extensions Reference for
Eclipselink.” Disponível em:
<https://www.eclipse.org/eclipselink/documentation/2.4/eclipselink_jpa_extensions.pdf>.

</div>

<div id="ref-eclipselinkmigration">

———. 2017. “FAQ - Migrate Hibernate to Eclipselink.” Disponível em:
<https://www.eclipse.org/eclipselink/documentation/2.5/solutions/migrhib002.htm>.

</div>

<div id="ref-eclipseLink">

———. 2019. “What Is Object-Relational Mapping.” Disponível em:
<https://wiki.eclipse.org/EclipseLink/FAQ/JPA#What_is_Object-Relational_Mapping>.

</div>

<div id="ref-elmari2010sistemas">

Elmari, R, and SB Navathe. 2010. *Sistemas de Banco de Dados. 6 Edição*.
Editora Pearson.

</div>

<div id="ref-badsmellxp">

Elssamadisy, Amr, and Gregory Schalliol. 2002. “Recognizing and
Responding to ‘Bad Smells’ in Extreme Programming.” In *Proceedings of
the 24th International Conference on Software Engineering*, 617–22. ICSE
’02. New York, NY, USA: ACM.
doi:[10.1145/581339.581418](https://doi.org/10.1145/581339.581418).

</div>

<div id="ref-engines2019db">

Engines, DB. 2019. “DB Engines Ranking.” Disponível em:
<https://db-engines.com/en/ranking_categories/>.

</div>

<div id="ref-mlrfora1">

Fabrício, U. 2015. “I Discovered an Undocumented Way to Improve Jpa
Performance.” Disponível em:
<https://bewire.be/blog/i-discovered-an-undocumented-way-to-improve-jpa-performance/>.

</div>

<div id="ref-badsmelljavascript">

Fard, A. M., and A. Mesbah. 2013. “JSNOSE: Detecting Javascript Code
Smells.” In *2013 Ieee 13th International Working Conference on Source
Code Analysis and Manipulation (Scam)*, 116–25.
doi:[10.1109/SCAM.2013.6648192](https://doi.org/10.1109/SCAM.2013.6648192).

</div>

<div id="ref-codemetric3">

Fontana, Francesca Arcelli, Vincenzo Ferme, Marco Zanoni, and Aiko
Yamashita. 2015. “Automatic Metric Thresholds Derivation for Code Smell
Detection.” In *Proceedings of the Sixth International Workshop on
Emerging Trends in Software Metrics*, 44–53. WETSoM ’15. Piscataway, NJ,
USA: IEEE Press. <http://dl.acm.org/citation.cfm?id=2821491.2821501>.

</div>

<div id="ref-topLink">

Foundation, Eclipse. 2019. “What Is Object-Relational Mapping.”
Disponível em:
<https://wiki.eclipse.org/EclipseLink/FAQ/JPA#What_is_Object-Relational_Mapping>.

</div>

<div id="ref-Martinfowler2018refactoring">

Fowler, M. 2018. *Refactoring: Improving the Design of Existing Code*.
Addison-Wesley Signature Series (Fowler). Pearson Education.
[https://books.google.com.br/books?id=2H1\\\_DwAAQBAJ](https://books.google.com.br/books?id=2H1\_DwAAQBAJ).

</div>

<div id="ref-MartinFowler:1999">

Fowler, Martin. 1999. *Refactoring: improving the design of existing
code*. Addison-Wesley Object Technology Series. Reading, MA:
Addison-Wesley. <http://cds.cern.ch/record/424198>.

</div>

<div id="ref-frakes1992information">

Frakes, William B, and Ricardo Baeza-Yates. 1992. *Information
Retrieval: Data Structures and Algorithms*. Prentice-Hall, Inc.

</div>

<div id="ref-arquitetura-smell-1">

Garcia, J., D. Popescu, G. Edwards, and N. Medvidovic. 2009.
“Identifying Architectural Bad Smells.” In *CSMR’09*.

</div>

<div id="ref-arquitetura-smell-2">

Garcia, Joshua, Daniel Popescu, George Edwards, and Nenad Medvidovic.
2009. “Toward a Catalogue of Architectural Bad Smells.” In
*Architectures for Adaptive Software Systems*, edited by Raffaela
Mirandola, Ian Gorton, and Christine Hofmeister. Berlin, Heidelberg:
Springer Berlin Heidelberg.

</div>

<div id="ref-MLR2019">

Garousi, Vahid, Michael Felderer, and Mika V. Mäntylä. 2019. “Guidelines
for Including Grey Literature and Conducting Multivocal Literature
Reviews in Software Engineering.” *Information and Software Technology*
106: 101–21.
doi:[https://doi.org/10.1016/j.infsof.2018.09.006](https://doi.org/https://doi.org/10.1016/j.infsof.2018.09.006).

</div>

<div id="ref-GitHub">

GitHub. 2020. “Pesquisar Repositórios.” Disponível em:
<https://docs.github.com/pt/github/searching-for-information-on-github/searching-for-repositories>.

</div>

<div id="ref-mlr5_perform_improvment">

Gomes, Gustavo. 2016. “Performance Improvement in Java Applications: ORM
/ Jpa.” Disponível em: [
	https://dzone.com/articles/performance-improvement-in-java-applications-orm-j](
	https://dzone.com/articles/performance-improvement-in-java-applications-orm-j).

</div>

<div id="ref-Goodman">

Goodman, Leo. 1961. “Snowball Sampling.” *Ann Math Stat* 32 (March).
doi:[10.1214/aoms/1177705148](https://doi.org/10.1214/aoms/1177705148).

</div>

<div id="ref-badsmellusuability">

Grigera, Julián, Alejandra Garrido, and José Matías Rivero. 2014. “A
Tool for Detecting Bad Usability Smells in an Automatic Way.” In *Web
Engineering*, edited by Sven Casteleyn, Gustavo Rossi, and Marco
Winckler. Cham: Springer International Publishing.

</div>

<div id="ref-mlr11_avoid_jpa_performance">

Hut, Chris. 2015. “Avoiding Jpa Performance Pitfalls.” Disponível em: [
		https://www.veracode.com/blog/secure-development/avoiding-jpa-performance-pitfalls](
		https://www.veracode.com/blog/secure-development/avoiding-jpa-performance-pitfalls).

</div>

<div id="ref-mlr19_improve_jpa_perfomance">

Janssen, Thorben. 2015. “How to Improve Jpa Performance | Rebel.”
Disponível em:
<https://www.jrebel.com/blog/how-to-improve-jpa-performance/>.

</div>

<div id="ref-thobernJanssenHibernateTips">

———. 2017a. *Hibernate Tips*. Nermina Miller.

</div>

<div id="ref-mlr6_the_best_way_vlad">

———. 2017b. “Solve Hibernate Performance Issues in Development.”
Disponível em: [
	https://stackify.com/find-hibernate-performance-issues/](
	https://stackify.com/find-hibernate-performance-issues/).

</div>

<div id="ref-mlr8_comon_hibernate_thorben">

———. 2018. “3 Common Hibernate Performance Issues in Your Log.”
Disponível em: [
		https://www.baeldung.com/hibernate-common-performance-problems-in-logs](
		https://www.baeldung.com/hibernate-common-performance-problems-in-logs).

</div>

<div id="ref-thorbenJanssen">

———. 2019a. “Entities or Dtos – When Should You Use Which Projection.”
Disponível em:
<https://thoughts-on-java.org/entities-dtos-use-projection>.

</div>

<div id="ref-throbenXMLAnotacao">

———. 2019b. “Mapping Definitions in Jpa and Hibernate – Annotations, Xml
or Both.” Disponível em:
<https://thoughts-on-java.org/mapping-definitions-jpa-hibernate-annotations-xml/>.

</div>

<div id="ref-mlr3_tips_to_boost">

———. 2020. “7 Tips to Boost Your Hibernate Performance - Thoughts on
Java.” Disponível em: [
	https://thoughts-on-java.org/tips-to-boost-your-hibernate-performance/](
	https://thoughts-on-java.org/tips-to-boost-your-hibernate-performance/).

</div>

<div id="ref-keith2018pro">

Keith, Mike, Merrick Schincariol, and Massimo Nardone. 2018. *Pro Jpa 2
in Java Ee 8: An in-Depth Guide to Java Persistence Apis*. Apress.

</div>

<div id="ref-badsmell_change1">

Khomh, Foutse, Massimiliano Di Penta, and Yann-Gael Gueheneuc. 2009. “An
Exploratory Study of the Impact of Code Smells on Software
Change-Proneness.” In *2009 16th Working Conference on Reverse
Engineering*, 75–84. IEEE.

</div>

<div id="ref-som">

Kohonen, Teuvo. 1990. “The Self-Organizing Map.” *Proceedings of the
IEEE* 78 (9). IEEE: 1464–80.

</div>

<div id="ref-catalog_of_code_smell_orm">

Loli, Samuel, Leopoldo Teixeira, and Bruno Cartaxo. 2020. “A Catalog of
Object-Relational Mapping Code Smells for Java.” In *SBES 2020
Research*. Natal, Rio Grande do Norte, Brasil.

</div>

<div id="ref-jvmreport">

Maple, S., and A. Binstoc. 2018. “JVM Ecosystem Report 2018.” In *JVM
Ecosystem Report 2018*.

</div>

<div id="ref-mlr10_n1_spring">

Marciniec, Michal. 2019. “JPA: N+1 Select Problem \[Spring & Jpa
Pitfalls Series.” Disponível em: [
		https://codete.com/blog/jpa-n-plus-1-select-problem/](
		https://codete.com/blog/jpa-n-plus-1-select-problem/).

</div>

<div id="ref-Martin:2008:CCH:1388398">

Martin, Robert C. 2008. *Clean Code: A Handbook of Agile Software
Craftsmanship*. 1st ed. Upper Saddle River, NJ, USA: Prentice Hall PTR.

</div>

<div id="ref-mlrfora2">

Mat, B. 2011. “Hibernate Performance - Stack Overflow.” Disponível em:
<https://stackoverflow.com/questions/5155718/hibernate-performance>.

</div>

<div id="ref-mlr13_JPA_otimizando">

Medeiros, Igor. 2015. “Java Persistence Api: Otimizando a Performance
Das Aplicações.” Disponível em: [
		https://www.devmedia.com.br/java-persistence-api-otimizando-a-performance-das-aplicacoes/32091](
		https://www.devmedia.com.br/java-persistence-api-otimizando-a-performance-das-aplicacoes/32091).

</div>

<div id="ref-high_perform_vlad">

Mihalcea, V. 2016. *High-Performance Java Persistence*. Vlad Mihalcea.
[https://books.google.com.br/books?id=g\\\_gaMQAACAAJ](https://books.google.com.br/books?id=g\_gaMQAACAAJ).

</div>

<div id="ref-mlr7_hibernate_performance">

Mihalcea, Vlad. 2016a. “Hibernate Performance Tuning and Best
Practices.” Disponível em: [
	https://in.relation.to/2016/09/28/performance-tuning-and-best-practices/](
	https://in.relation.to/2016/09/28/performance-tuning-and-best-practices/).

</div>

<div id="ref-mlr17_High-Performance">

———. 2016b. “High-Performance Hibernate Devoxx France.” Disponível em:
<https://www.slideshare.net/VladMihalcea/high-performance-hibernate-devoxx-france>.

</div>

<div id="ref-mlr1_eclipslink">

———. 2017. “Performance Features.” Disponível em:
<https://www.eclipse.org/eclipselink/documentation/2.7/solutions/performance001.htm/>;
Eclipse Foundation.

</div>

<div id="ref-mlr2_vlad_mihalcea_2019">

———. 2019a. “Hibernate Performance Tuning Tips.” Disponível em:
<https://vladmihalcea.com/hibernate-performance-tuning-tips/>; Vlad
Mihalcea.

</div>

<div id="ref-mlr18_eager_fetching_code_smell">

———. 2019b. “EAGER Fetching Is a Code Smell When Using Jpa and
Hibernate.” Disponível em:
<https://vladmihalcea.com/eager-fetching-is-a-code-smell/>.

</div>

<div id="ref-vlad_mihalcea_enable_lazy_load">

———. 2019c. “The Hibernate.enable\_lazy\_load\_no\_trans Anti-Pattern.”
Disponível em:
<https://vladmihalcea.com/the-hibernate-enable_lazy_load_no_trans-anti-pattern>.

</div>

<div id="ref-vlad_mihalcea_n1">

———. 2019d. “How to Detect the Hibernate N+1 Query Problem During
Testing.” Disponível em:
<https://vladmihalcea.com/how-to-detect-the-n-plus-one-query-problem-during-testing/>.

</div>

<div id="ref-mlr4_the_best_way_vlad">

———. 2020a. “The Best Way to Prevent Jpa and Hibernate Performance
Issues.” Disponível em: [
	https://vladmihalcea.com/jpa-hibernate-performance-issues/](
	https://vladmihalcea.com/jpa-hibernate-performance-issues/).

</div>

<div id="ref-vlad_mihalcea_lazyInitialization">

———. 2020b. “The Best Way to Handle the Lazyinitializationexception.”
<https://vladmihalcea.com/the-best-way-to-handle-the-lazyinitializationexception/>.

</div>

<div id="ref-ManyToOneCommon">

———. 2020c. “ManyToOne Jpa and Hibernate Association Best Practices.”
Disponível em: <https://vladmihalcea.com/manytoone-jpa-hibernate/>.

</div>

<div id="ref-mlr15_hibernate_543">

Mihalcea, Vlad, Steve Ebersole, Gunnar Morling Andrea Boriero, Gail
Badner, Chris Cranford, Emmanuel Bernard, Sanne Grinovero, et al. 2018.
“Hibernate Orm 5.4.3.Final User Guide.” Disponível em:
<https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html>.

</div>

<div id="ref-hibernate_hql">

———. 2019. “HQL and Jpql.” Disponível em:
<https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#hql>.

</div>

<div id="ref-10.1007/11687061_7">

Monteiro, Miguel P., and João M. Fernandes. 2006. “Towards a Catalogue
of Refactorings and Code Smells for Aspectj.” In *Transactions on
Aspect-Oriented Software Development I*, edited by Awais Rashid and
Mehmet Aksit. Berlin, Heidelberg: Springer Berlin Heidelberg.

</div>

<div id="ref-mlr14_hibernate_n1">

Munhoz, João. 2019. “Hibernate and the N+1 Selections Problem -
Quintoandar Tech.” Disponível em: [
		https://medium.com/quintoandar-tech-blog/hibernate-and-the-n-1-selections-problem-c497710fa3fe](
		https://medium.com/quintoandar-tech-blog/hibernate-and-the-n-1-selections-problem-c497710fa3fe).

</div>

<div id="ref-codemetric1">

Munro, M. J. 2005. “Product Metrics for Automatic Identification of ‘Bad
Smell’ Design Problems in Java Source-Code.” In *11th Ieee International
Software Metrics Symposium (Metrics’05)*, 15–15.
doi:[10.1109/METRICS.2005.38](https://doi.org/10.1109/METRICS.2005.38).

</div>

<div id="ref-gustavoPinto">

Nazário, M. F. C., E. Guerra, R. Bonifácio, and G. Pinto. 2019.
“Detecting and Reporting Object-Relational Mapping Problems: An
Industrial Report.” In *2019 Acm/Ieee International Symposium on
Empirical Software Engineering and Measurement (Esem)*, 1–6.

</div>

<div id="ref-openJpaManual">

OpenJpa, Apache. 2019. “Apache Openjpa 3.1 User’s Guide.” Disponível em:
<http://openjpa.apache.org/builds/3.1.0/apache-openjpa/docs/manual.pdf>.

</div>

<div id="ref-oppenheim2000questionnaire">

Oppenheim, A.N. 2000. *Questionnaire Design, Interviewing and Attitude
Measurement*. Bloomsbury Academic.
<https://books.google.com.br/books?id=6V4GnZS7TO4C>.

</div>

<div id="ref-mlr9_toplink_perfomance">

Oracle. 2015. “9 Oracle Toplink (Eclipselink) Jpa Performance Tuning.”
Disponível em: [
		https://docs.oracle.com/middleware/1212/core/ASPER/toplink.htm](
		https://docs.oracle.com/middleware/1212/core/ASPER/toplink.htm).

</div>

<div id="ref-Palomba:2013:DBS:3107656.3107692">

Palomba, Fabio, Gabriele Bavota, Massimiliano Di Penta, Rocco Oliveto,
Andrea De Lucia, and Denys Poshyvanyk. 2013. “Detecting Bad Smells in
Source Code Using Change History Information.” In *Proceedings of the
28th Ieee/Acm International Conference on Automated Software
Engineering*, 268–78. ASE’13. Piscataway, NJ, USA: IEEE Press.
doi:[10.1109/ASE.2013.6693086](https://doi.org/10.1109/ASE.2013.6693086).

</div>

<div id="ref-database_smell">

Sharma, Tushar, Marios Fragkoulis, Stamatia Rizou, Magiel Bruntink, and
Diomidis Spinellis. 2018. “Smelly Relations: Measuring and Understanding
Database Schema Quality.” In *Proceedings of the 40th International
Conference on Software Engineering: Software Engineering in Practice*,
55–64. ICSE-Seip ’18. New York, NY, USA: Association for Computing
Machinery.
doi:[10.1145/3183519.3183529](https://doi.org/10.1145/3183519.3183529).

</div>

<div id="ref-silberschatzsistema">

Silberschatz, Abraham, Henry F Korth, and S Sudarshan. 2006. *Sistema de
Banco de Dados*. Elsevier Editora Ltda.

</div>

<div id="ref-codemetric2">

Simon, F., F. Steinbruckner, and C. Lewerentz. 2001. “Metrics Based
Refactoring.” In *Proceedings Fifth European Conference on Software
Maintenance and Reengineering*, 30–38.
doi:[10.1109/CSMR.2001.914965](https://doi.org/10.1109/CSMR.2001.914965).

</div>

<div id="ref-SpotBugsPlugin">

SpotBugs. 2018a. “Implement Spotbugs Plugin.” Disponível em:
<https://spotbugs.readthedocs.io/en/stable/implement-plugin.html>.

</div>

<div id="ref-spotbugsOficial">

———. 2018b. “SpotBugs Manual.” Disponível em:
<https://spotbugs.readthedocs.io/en/stable/>.

</div>

<div id="ref-terra2008ferramentas">

Terra, Ricardo, and Roberto S Bigonha. 2008. “Ferramentas Para análise
Estática de códigos Java.” *Monografia, Universidade Federal de Minas
Gerais (UFMG), Belo Horizonte*.

</div>

<div id="ref-TOM20131498">

Tom, Edith, Aybüke Aurum, and Richard Vidgen. 2013. “An Exploration of
Technical Debt.” *Journal of Systems and Software* 86 (6): 1498–1516.
doi:[https://doi.org/10.1016/j.jss.2012.12.052](https://doi.org/https://doi.org/10.1016/j.jss.2012.12.052).

</div>

<div id="ref-Tomassi:2018:BWE:3236024.3275439">

Tomassi, David A. 2018. “Bugs in the Wild: Examining the Effectiveness
of Static Analyzers at Finding Real-World Bugs.” In *Proceedings of the
2018 26th Acm Joint Meeting on European Software Engineering Conference
and Symposium on the Foundations of Software Engineering*, 980–82.
ESEC/Fse 2018. New York, NY, USA: ACM.
doi:[10.1145/3236024.3275439](https://doi.org/10.1145/3236024.3275439).

</div>

<div id="ref-experienced_and_inexperienced">

Tsunoda, T., H. Washizaki, Y. Fukazawa, S. Inoue, Y. Hanai, and M.
Kanazawa. 2017. “Evaluating the Work of Experienced and Inexperienced
Developers Considering Work Difficulty in Sotware Development.” In *2017
18th Ieee/Acis International Conference on Software Engineering,
Artificial Intelligence, Networking and Parallel/Distributed Computing
(Snpd)*, 161–66.

</div>

<div id="ref-vial2018lessons">

Vial, Gregory. 2018. “Lessons in Persisting Object Data Using
Object-Relational Mapping.” *IEEE Software*. IEEE.

</div>

<div id="ref-antipatterns_of_one_to_many">

Węgrzynowicz, P. 2013. “Performance Antipatterns of One to Many
Association in Hibernate.” In *2013 Federated Conference on Computer
Science and Information Systems*, 1475–81.

</div>

<div id="ref-white2014java">

White, Oliver. 2014. “Java Tools and Technologies Landscape for 2014.”
*Retrieved Octubre* 30: 2014.

</div>

<div id="ref-badsmell_manutebilidade1">

Yamashita, Aiko, and Leon Moonen. 2013. “Exploring the Impact of
Inter-Smell Relations on Software Maintainability: An Empirical Study.”
In *Proceedings of the 2013 International Conference on Software
Engineering*, 682–91. IEEE Press.

</div>

</div>

[^1]: <https://docs.jboss.org/hibernate/jpa/2.1/api/>


