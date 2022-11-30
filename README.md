# Oficina Neo4j - Saber 2022

Este documento apresenta a parte mão na massa da oficina sobre bancos em dados em grafos em Neo4j realizada na [SABER 2022](https://saber.inf.ufpr.br/).

## Criação da conta no Neo4j Aura

Para iniciar a oficina, crie sua conta no [Neo4j Aura](https://console.neo4j.io/). Você pode fazer usar uma conta de e-mail de preferência, incluindo SSO com uma conta google.

Será enviado um link de verificação para seu e-mail. Após verificar o link, acesse sua conta e abra o dashboard.

## Geração da instância de exemplo

Para esta oficina, vamos criar uma instância de exemplo com um subconjunto pequeno de dados do [Stack Overflow](https://stackoverflow.com/), um lugar que todo dev conhece :)

Depois de ter criado sua conta, a seção de instâncias deverá estar vazia. 

Clique em "New Instance" e na sequência crie uma instância gratuita do tipo "Stackoverflow Data".

Ao clicar em "Create" abrirá uma janela com a senha do banco de dados gerada. Salve essa senha pois ela será utilizada posteriormente. <instrução de finalização>

Aguarde uns instantes enquanto sua instância é criada. 

## Abertura do Neo4j Browser

Quando finalizar a criação da sua instância, clique em "Query".

Em uma nova aba, será aberto o Neo4j Browser. O Neo4j Browser é uma interface para testar e visualizar o resultado de queries Cyphers. 

Primeiramente, será necessário conectar ao banco de dados. Para isso, utilize a senha salva nas etapas anteriores. 

Com o usuário conectado, já é possível executar as queries em cypher.

## Entendendo os dados

Geralmente nosso banco começaria vazio e precisariamos popular - ou criando os nós e relacionamentos usando Cypher ou importando os dados em formato CSV ou GraphQL.

Mas nosso foco nesta oficina é entender principalmente como funciona a leitura e interpretação dos dados em Cypher. Caso tenha interesse de entender como criar/importar dados [esta documentação](https://neo4j.com/docs/cypher-manual/current/) e [esta documentação](https://neo4j.com/developer/data-import/) vão te ajudar.


### Visualizar o esquema do grafo

O comando:

```cypher
CALL db.schema.visualization()
```

retornará o esquema do grafo.

### Retornar um subconjunto de nós

O comando

```cypher
MATCH (v) RETURN v LIMIT 100
```

retornará os 100 primeiros nós que se adequam ao padrão do comando `MATCH`.

### Retornando todos os nós de um tipo específico de labels

O comando

```cypher
MATCH (u:User) RETURN u
```

retornará todos os nós com a label `User`.


### Descobrindo os labels disponíveis

O comando

```cypher
MATCH (v) RETURN labels(v)
```

retornará todos os labels de **cada nó** no grafo. 

Compare agora com o comando:

```cypher
MATCH (v) RETURN DISTINCT labels(v)
```

Que retorna todos os labels únicos existentes. 

### Descobrindo as propriedades de um nó de usuário
O comando

```cypher
MATCH (u:User) RETURN DISTINCT keys(u)
```

retorna as chaves de propriedades dos nós com label usuário. Note como a definição de propriedades em ordem diferentes afeta a disposição/diferenciação das chaves.

### Retornando a lista de nomes dos usuários

O comando

```cypher
MATCH (u:User) RETURN u.display_name as username
```

retorna a lista dos nomes dos usuários já transformando o output para ter um valor desejado. 

### Descobrindo o ID de um nó de label "Question"

O comando

```cypher
MATCH (q:Question) RETURN ID(q) as id, q.title LIMIT 100
```

retorna a lista dos ids associados aos títulos dos nós das 100 primeiras questões.

### Contando o total de usuários

O comando

```cypher
MATCH (u:User) RETURN COUNT(u) as total
```
retorna o total de nós com label User já transformando o output para ter o nome de total.


### Retornando um subgrafo 

O comando

```cypher
MATCH (u:User)-[r:ASKED]->(q:Question) RETURN u, r, q
```

retorna o subgrafo de todos os usuários que perguntaram uma questão. Note como *a relação também é um resultado possível*.


### Retornando um subconjunto de relações

O comando

```cypher
MATCH (u)-[r]->(v) RETURN r LIMIT 100
```

retorna os dados das 100 primeiras relações que satisfazem o padrão em `MATCH`.


### Filtrando resultados

Existem dois modos de filtrar resultados:

O comando

```cypher
MATCH (u:User { display_name: "schernichkin" }) RETURN u
```

retorna todos os nós de User cujo valor de `display_name` é "schernichkin".

O mesmo pode ser obtido por

```cypher
MATCH (u:User) WHERE u.display_name = "schernichkin" RETURN u
```

### O where pode mais!

O comando

```cypher
MATCH (u:User) WHERE u.display_name STARTS WITH "s" RETURN u
```

retorna todos os usuários na qual o valor de `display_name` inicia com "s".

Se quisermos filtrar apenas o `diplay_name`:

```cypher
MATCH (u:User) WHERE u.display_name STARTS WITH "s" RETURN u.display_name
```

Que tal dar uma olhadinha no que a [clausula `WHERE` pode mais](https://neo4j.com/docs/cypher-manual/5/clauses/where/)? :)

## Perguntas mais díficeis...

### Qual os 3 usuários que mais fizeram perguntas?

O comando

```cypher
MATCH (u:User)-[a:ASKED]->(q:Question) RETURN u.display_name, COUNT(a) as totalAsks ORDER BY totalAsks DESC LIMIT 3
```

lista os três usuários que mais fizeram perguntas.

### Quais usuários que fizeram perguntas com a tag "neo4j"?

O comando

```cypher
MATCH (u:User)-[:ASKED]->(:Question)-[:TAGGED]->(:Tag {name : "neo4j"}) RETURN u.display_name
```
retorna o `display_name` de todos os usuários que fizeram uma pergunta com a tag "neo4j". Note como o importante é que os nós se adequem obrigatóriamente ao padrão. Por isso é garantido

### Quantos usuários que fizeram perguntas com a tag "graph"?

Para isto basta modificar o comando anterior:

```cypher
MATCH (u:User)-[:ASKED]->(:Question)-[:TAGGED]->(:Tag {name : "neo4j"}) RETURN COUNT(u)
```

Contando os nós de usuário.

### Quais usuários que já proveram respostas E comentários?

O comando

```cypher
MATCH (:Comment)<-[:COMMENTED]-(u:User)-[:PROVIDED]->(:Answer) RETURN DISTINCT u.display_name
```

identifica os usuários que tanto responderam quanto comentaram algo. 

Porém, uma outra forma de fazer é a seguinte

```cypher
MATCH (u:User)-[:PROVIDED]->(:Answer)
WITH u
MATCH (u2:User)-[:COMMENTED]->(:Comment)
WHERE u2.uuid = u.uuid
RETURN DISTINCT u2.display_name
```

## Desafios!

1) Escreva uma query que exiba as 5 tags mais utilizadas nas questões neste subconjunto de dados.
2) Escreva uma query em cypher que retorne todos as perguntas com o corpo (`body_markdown`) com tamanho menor que 120 caracteres
3) Escreva uma query que retorne todos os usuários que responderam a própria questão. 
