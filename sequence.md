# Diagramas de Sequência

## RF-01 - Navegação pelos produtos através da hierarquia de categorias

<!-- TODO isto em árvore? É muito custoso na BD gerar isto? Caching? -->
Na API temos o endpoint `/categories` para obter as categorias na sua forma hierarquica.

<!-- TODO ter a certeza desta forma, ou se faremos algo tipo /products?categoryId=1,2 -->
Podemos então selecionar as categorias que queremos visualizar e filtrar os produtos pelos IDs dessas categorias no endpoint `/products?categoryId=1&categoryId=2`

```mermaid
sequenceDiagram
    actor A as Alice
    participant CC as CategoryCatalog
    participant PC as ProductCatalog

    A ->> CC : listCategories()
    CC -->> A: category[]
    A ->> PC: listProducts(category1, category2)
    PC -->> A: products[]
```

## RF-02: Visualização de produtos e seus fornecedores

Através do endpoint `/products` podemos obter os produtos, que terão no seu interior o fornecedor.

<!-- TODO este endpoint -->
Contudo, para obter todos os fornecedores de um `ProductSpec` podemos usar o endpoint `/products/{id}/suppliers` que nos devolve todos os fornecedores que fornecem esse produto.

```mermaid
sequenceDiagram
    actor A as Alice
    participant PSC as ProductSpecCatalog
    participant SC as SupplierCatalog

    A ->> PSC: listProducts()
    PSC -->> A: productSpec[]
    A ->> SC: listSuppliers(productSpec1)
    SC -->> A: suppliers[]
```

## RF-03: Pesquisa de produtos através dos campos comuns a todos os produtos

<!-- TODO enumerar os campos comuns a todos os produtos -->
Podemos usar o endpoint `/products/search?q=` definindo o parâmetro `q` com o termo que queremos pesquisar. A pesquisa será relativa as campos comuns a todos os produtos.

```mermaid
sequenceDiagram
    actor A as Alice
    participant PC as ProductCatalog

    A ->> PC: searchProducts("term")
    PC -->> A: products[]
```

## RF-04: Colocação, consulta, e remoção de produtos num cesto de compra

<!-- TODO endpoints /cart -->
Para colocar um produto no carrinho de compras podemos usar o endpoint `/cart/@me/products/{productId}` com o método `PUT`, em que `productId` é id do produto que queremos colocar no carrinho.

```mermaid
sequenceDiagram
    actor A as Alice
    participant CC as CartCatalog
    participant C as Cart: cart

    A ->> CC: getCart(@me)
    CC ->> C: get()
    activate C
    C -->> CC: cart
    CC -->> A: cart
    A ->> C: addProduct(productId)
    C -->> A: productAdded
    deactivate C
```

Para consultar os produtos no carrinho podemos usar o endpoint `/cart/@me/products` com o método `GET`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant CC as CartCatalog
    participant C as Cart: cart

    A ->> CC: getCart(@me)
    CC ->> C: get()
    activate C
    C -->> CC: cart
    CC -->> A: cart
    A ->> C: listProducts()
    C -->> A: products[]
    deactivate C
```

Remover um produto do carrinho é feito através do endpoint `/cart/@me/products/{productId}` com o método `DELETE`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant CC as CartCatalog
    participant C as Cart: cart

    A ->> CC: getCart(@me)
    CC ->> C: get()
    activate C
    C -->> CC: cart
    CC -->> A: cart
    A ->> C: removeProduct(productId)
    C -->> A: productRemoved
    deactivate C
```

## RF-05: Criação de uma conta no sistema

### Opção 1

<!-- TODO desta forma temos os campos todos a null???? 
deveriamos antes fazer uma tabela de credenciais? mas aí teriamos de permitir que a FK utilizador seja nulável -->
1. O utilizador tem de se registar. Como utilizamos um sistema externo de autenticação, temos de usar o *endpoint* `/auth/login` que irá criar uma credencial no sistema e *autenticar* o utilizador.
<!-- TODO escrever sobre isto na API.
Tem de permitir que um Admin crie contas para outros utilizadores. Passar o token no body? -->
2. O utilizador tem de criar a sua conta. Para isto temos de usar o endpoint `/users` com o método `POST`. Este *endpoint* é protegido, por isso temos de usar o *access token* obtido anteriormente.

```mermaid
sequenceDiagram
    actor A as Alice
    participant F as Firebase
    participant Au as LocalAuthService
    participant UC as UserCatalog
    participant U as User

    A ->> F: authenticate()
    F -->> A: fbToken
    A ->> Au: login(fbToken)
    Note right of F: We may be able to verify locally, skipping this step
    alt verify token
        Au ->> F: verifyToken(fbToken)
        F -->> Au: fbToken
    end
    Au -->> A: accessToken
    A ->> UC: createUser(accessToken, {username, email, ...})
    UC ->> U: new({username, email, ...})
    activate U
    U -->> UC: user
    UC -->> A: user
    deactivate U
```

### Opção 2

1. Criar uma conta sem credenciais relacionadas. Feito através dum `POST` a `/users` com os dados necessários, sendo este *endpoint* **não** protegido.
2. Após a criação do utilizador com sucesso, solicitar a adição de uma credencial (dos vários tipos `authType` possíveis: `password`, `google`, `facebook`, `twitter`)
   1. Autenticar o utilizador no serviço externo
   2. Obtenção de um token do serviço externo
   3. Adição da credencial ao utilizador através de `PUT` a `/users/@me/credentials/{authType}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant F as Firebase
    participant Au as LocalAuthService
    participant UC as UserCatalog
    participant U as User

    A ->> UC: createUser(accessToken, {username, email, ...})
    UC ->> U: new({username, email, ...})
    activate U
    U -->> UC: user
    UC -->> A: user
    A ->> F: authenticate()
    F -->> A: fbToken
    Note left of Au: This is weird because login would require a user to already exist
    A ->> Au: login(fbToken)
    Note left of Au: We may be able to verify locally, skipping this step
    alt verify token
        Au ->> F: verifyToken(fbToken)
        F -->> Au: fbToken
    end
    Au -->> A: accessToken
    A ->> U: addCredential(accessToken)
    U -->> A: credentialAdded
    deactivate U
```

## RF-06: Fazer login no sistema

Autenticar o utilizador no serviço externo através do *frontend* e obter um *token* do serviço externo.

Usar o *endpoint* `/auth/login` com o método `POST`, forneçendo o *token* no *body* de forma a trocá-lo por um *access token* do nosso sistema.

```mermaid
sequenceDiagram
    actor A as Alice
    participant F as Firebase
    participant Au as LocalAuthService

    A ->> F: authenticate()
    F -->> A: fbToken
    A ->> Au: login(fbToken)
    Note right of F: We may be able to verify locally, skipping this step
    alt verify token
        Au ->> F: verifyToken(fbToken)
        F -->> Au: fbToken
    end
    Au -->> A: accessToken    
```

## RF-07: Edição dos dados e remoção da sua conta no sistema

Para editar os dados do utilizador podemos usar o *endpoint* `/users/@me` com o método `PUT`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant UC as UserCatalog
    participant U as User

    A ->> UC: getUser(@me)
    UC ->> U: get()
    activate U
    U -->> UC: user
    UC -->> A: user
    A ->> U: edit({username, email, ...})
    U -->> A: userEdited
    deactivate U
```

Para remover a conta do utilizador podemos usar o *endpoint* `/users/@me` com o método `DELETE`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant UC as UserCatalog
    participant U as User

    A ->> UC: getUser(@me)
    UC ->> U: get()
    activate U
    U -->> UC: user
    UC -->> A: user
    A ->> U: delete()
    U -->> A: userDeleted
    deactivate U
```

## RF-08: Pesquisa de produtos através dos campos específicos das categorias

<!-- TODO -->
esta tá chata de definir na API, mas é importante

## RF-09: Visualização do histórico de encomendas e seus detalhes

## RF-10: Comparação de dois produtos, com as diferenças em destaque

## RF-11: Encomenda dos produtos no cesto de compras

## RF-12: Pagamento de encomenda recorrendo a um sistema externo

## RF-13: Cancelamento de encomenda, desde que dentro de um dado prazo

## RF-14: Notificação sobre a saída de produtos encomendados de um fornecedor

## RF-15: Notificação sobre a chegada iminente de encomenda

## RF-16: Visualização de relatório do impacto local3 das suas encomenda

## RF-17: Exportação dos dados das encomendas para ficheiros JSON

## RF-18: Criação, gestão, e remoção de unidade de produção

## RF-19: Criação, gestão, e remoção de produto, e ligação a unidade de produção

## RF-20: Visualização de unidade de produção e dos seus produtos

## RF-21: Criação, edição, e remoção de veículo de transporte de produtos

## RF-22: Notificação sobre encomenda de consumidor

## RF-23: Visualização de encomenda de consumidor

## RF-24: Colocação de produto encomendado em veículo de transporte disponível

## RF-25: Registo de saída de veículo de transporte com produtos encomendados

## RF-26: Registo de chegada iminente de encomenda a casa do consumidor

## RF-27: Visualização de relatório do impacto local das vendas dos seus produtos

## RF-28: Desativação e reativação de qualquer conta de utilizador no sistema

## RF-29: Visualização de relatório do impacto local das vendas de produtos
