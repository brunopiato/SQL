# Artigo sobre SQL

*Autor*: Bruno Garcia Piato <br>
*Data*: 19 de outubro de 2023


# Objetivos do projeto
Meu principal objetivo neste projeto foi adquirir experiência e familiaridade com a linguagem SQL para realizar consultas e acessos a bancos de dados relacionais, consolidando meu aprendizado e demonstrando minhas habilidades e capacidades analíticas com esta ferramenta. Minha estratégia para tanto foi a de obter um bando de dados relacional com que eu pudesse trabalhar respondendo perguntas de negócio e desenvolvendo análises dos dados nele depositados. 


# O conjunto de dados
Neste projeto eu utilizei um conjunto de dados disponível na plataforma de desafios de dados Kaggle. Ele se refere ao sistema de vendas da e-commerce barasileira Olist. O modelo de negócios de e-commerce tem suas idiossincrasias em termos das entidades que o compõem e as relações estabelecidas entre elas para que os negócios e transações possam ocorrer de maneira flúida. Por este motivo, estudar e conhecer o modelo de negócios é fundamental para que a análise de dados se dê de forma satisfatória. Em contrapartida a análise de dados e investigação das relações entre entidades serve como material de estudos para compreender o funcionamento e fluxo de processos do modelo de negócios. 

Os arquivos obtidos da plataforma foram transformados em um banco de dados relacional em SQLite para que os acessos e consultas pudessem ser realizados usando a linguagem SQL (*Structured Query Language*). Uma vez convertidos em um banco de dados, utilei o VSCode como IDE para a investigação e análise dos dados em conjunção com as extensões SQLite e diversas outras que permitissem o trabalho centralizado em um único ambiente de desenvolvimento. 


## O Modelo Entidade Relacionamento (MER)
Os Modelos Entidade Relacionamento, conhecidos como MER, são estruturas ideais que relacionam as entidades de um negócio. Isto é, é um esquema que representa como as entidades que compõem um negócio se relacionam entre si. Para representar graficamente estas relações utilizamos o Diagrama Entidade Relacional (DER). Abaixo podemos ver como está disposto o DER do modelo de negócios da Olist de acordo com o que está disponível no desafio do Kaggle.

![Diagrama Entidade Relacional da Olist](https://i.imgur.com/HRhd2Y0.png)

Nele podemos notar a existência de oito diferentes entidades, Orders, Order Items, Order Reviews, Order Payments, Customers, Products, Sellers e Geolocation. Cada uma delas se relaciona de uma forma diferente. A entidade Orders é central para as relações que se estabelecem entre as entidades e a dimensão *order_id* é a chave que liga a maior parte das demais entidades a ela. Além disso as chaves *customer_id*, *product_id* e *zip_code_prefix* podem ser utilizadas para conectar as demais entidades entre si.

## Cardinalidade entre entidades e chaves de conexão
A cardinalidade entre entidades é um atributo fundamental dos bancos de dados, uma vez que nos permite compreender como ocorrem as relações entre elas. As entidades podem se relacionar de um para um (1:1), de um para muitos (1:n) ou de muitos para muitos (n:m), de acordo com o número de entradas de uma tabela que se relacionam com a outra. Em outras palavras, há entidades em que uma entrada se relaciona a apenas uma entrada de outra tabela, como um vendedor e um endereço. Entretanto podemos ter entidades em que uma entrada se relaciona a diversas entradas de outra entidade, como um pedido que pode conter diversos itens. Por fim, pode haver entidades em que diversas entradas se relacionam a múltiplas entradas de outra tabela, como produtos e pedidos. Compreender estas relações é fundamental para entender o modelo de negócios e sermos capazes de entregar as melhores análises possíveis.

As relações entre entidades ocorrem através das chaves. As chaves, isto é, variáveis, que são únicas em uma tabela e servem como características a serem acessadas por outras tabelas são as chaves primárias. Por exemplo, o ID de usuário na entidade de Clientes é uma chave primária, já que não se repete e é usada para acessar as informações daquele cliente.

Já variáveis cujas entradas podem aparecer diversas vezes em uma entidade e são utilizadas para referenciar chaves primárias de outras tabelas são conhecidas como chaves estrangeiras. A exemplo deste tipo de chaves podemos citar o ID de usuário na entidade Pedidos. Uma vez que um mesmo cliente pode ser responsável por mais de um pedido na maior parte dos modelos de negócios, esta variável é tida como chave estrangeira e servirá para acessar as informações de cada usuário na entidade de Clientes. 

No contexto dos dados utilizados neste projeto, por exemplo, a variável *order_id* é uma chave primária na entidade Orders e é uma chave estrangeira nas entidade Order Items e Order Payments, uma vez que um mesmo podido pode conter diversos itens e ter sido pago de mais de uma maneira diferente, tendo uma cardinalidade de um para muitos com ambas as entidades. 

Isto pode ser verificado utilizando as consultas a seguir.
```sql
-- Verificando se há entradas repetidas na 
-- variável order_id na entidade Orders
SELECT 
    o.order_id,
    COUNT(o.order_id) 
FROM orders o
GROUP BY o.order_id
HAVING  
    COUNT(o.order_id) > 1

-- Verificando se há entradas repetidas na 
-- variável order_id na entidade Order Items
SELECT 
    oi.order_id,
    COUNT(oi.order_id) 
FROM order_items oi
GROUP BY oi.order_id
HAVING  
    COUNT(oi.order_id) > 1

-- Verificando se há entradas repetidas na 
-- variável order_id na entidade Payments
SELECT 
    p.order_id,
    COUNT(p.order_id) 
FROM order_payments p
GROUP BY p.order_id
HAVING  
    COUNT(p.order_id) > 1
```

# Respondendo algumas perguntas de negócios

#### Qual foi o faturamenteo diário médio, agregado e mínimo ordenado cronologicamente?

```sql
SELECT 
    DATE(o.order_purchase_timestamp) AS data, 
    AVG(oi.price) AS media_vendas,
    SUM(oi.price) AS total_vendas,
    MIN(oi.price) as min_vendas
FROM 
    orders o LEFT JOIN order_items oi 
        ON (o.order_id = oi.order_id)
GROUP BY 
    DATE(o.order_purchase_timestamp)
ORDER BY 
    DATE(o.order_purchase_timestamp) ASC
```

#### Qual foi o número de clientes e o faturamenteo diário médio, agregado e mínimo ordenado cronologicamente no ano de 2017?

```sql
SELECT 
    DATE(o.order_purchase_timestamp) AS data, 
    COUNT(DISTINCT o.customer_id) AS clientes,
    COUNT(DISTINCT o.order_id) AS pedidos,
    AVG(oi.price) AS media_vendas,
    SUM(oi.price) AS total_vendas,
    MIN(oi.price) as min_vendas,
    AVG(ore.review_score) AS media_review
FROM 
    orders o LEFT JOIN order_items oi 
        ON (o.order_id = oi.order_id) LEFT JOIN order_reviews ore
        ON (o.order_id = ore.order_id)
WHERE
    DATE(o.order_purchase_timestamp) BETWEEN ('2017-01-01' AND '2017-12-31')
GROUP BY 
    DATE(o.order_purchase_timestamp)
ORDER BY 
    DATE(o.order_purchase_timestamp) ASC
```
