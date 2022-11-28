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

Obter os dois `ProductSpec` e iterar sobre as propriedades (e valores das categorias) de forma a saber quais diferem em valor.

Devemos definir se é possível comparar dois `ProductSpec` de categorias diferentes.

Convém definir se podemos comparar `Product` com `Product`, possibilitando comparar o mesmo `ProductSpec` de fornecedores diferentes, ou se tem de ser `ProductSpec` com `ProductSpec`

1. Com a primeira podemos comparar custos e impacto na comunidade
    - A especificação da API seria um `GET` a `/producers/{producerId}/products/{product1}/compare/{product2}`.
1. Com a segunda só queremos saber das diferenças da especificação do produto, que nem inclui o preço
    - A especificação da API seria um `GET` a `/products/{product1}/compare/{product2}`.

Podemos implementar ambos, mas acho que o primeiro é mais interessante.

```mermaid
sequenceDiagram
    actor A as Alice
    participant PC as ProductCatalog
    participant P as Product

    A ->> PC: getProduct(1)
    PC ->> P: get()
    activate P
    P -->> PC: product1
    PC -->> A: product1

    A ->> PC: getProduct(2)
    PC ->> P: get()
    activate P
    P -->> PC: product2
    PC -->> A: product2

    A ->> P: product1.compare(product2)
    P -->> A: differences

    deactivate P
    deactivate P
```

## RF-11: Encomenda dos produtos no cesto de compras

A especificação da API seria um `POST` a `/orders` com o *body* a conter o `userId` e o `cartId`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant C as Cart
    participant OC as OrderCatalog
    participant O as Order
    participant OI as OrderItem

    A ->> C: createOrder()
    C ->> OC: createOrder({userId, cartId})
    OC ->> O: new({userId, cartId})
    activate O

    O ->> C: getProducts()
    C -->> O: product[]

    loop for each product in products
        O ->> OI: new({productId, quantity})
        activate OI
        OI -->> O: orderItem
        deactivate OI

        O ->> O: addOrderItem(orderItem)
    end

    O -->> OC: order
    OC -->> A: order
    deactivate O
```

## RF-12: Pagamento de encomenda recorrendo a um sistema externo

Serviço externo: [**Stripe**](https://stripe.com), usando o [**Stripe Checkout**](https://stripe.com/docs/payments/checkout).

Começamos então por fazer um pedido `POST` ao *endpoint* `/orders/{orderId}/checkout` irá criar um *checkout session* no *Stripe* e redirecionar o utilizador para o preenchimento de dados relativos ao pagamento, do lado do *Stripe*.

Após a introdução dos dados de pagamento, o *Stripe* irá confirmar o sucesso da operação ao submeter um pedido `POST` a `/payments/webhook`, *aka* *webhook*.

```mermaid
sequenceDiagram
    actor A as Alice
    participant O as Order
    participant PG as PaymentGateway
    participant S as Stripe
    participant Su as Supplier

    A ->> O: checkout()
    O ->> PG: createCheckoutSession()
    PG ->> S: createCheckoutSession()
    S -->> PG: checkoutSession
    PG -->> O: checkoutSession
    O -->> A: checkoutSession

    Note right of A: Redirect to checkoutSession.url
    Note right of A: Fill payment details and submit to external service

    S -->> PG: webhook
    PG ->> O: setPaid()
    Note right of O: See RF-14 for notification details
    O -->> A: notifyPaid
    O -->> Su: notifyNewOrder
```

## RF-13: Cancelamento de encomenda, desde que dentro de um dado prazo

A especificação da API seria um `DELETE` a `/orders/{orderId}`.

Importante!!! Falta definir o prazo de cancelamento e outras regras.

Sugeria que o prazo de cancelamento fosse de 24h, mas que o utilizador pudesse cancelar a qualquer momento, desde que o produto ainda não tivesse sido enviado.

Também é possível definir um prazo de cancelamento para cada produto, definido pelo fornecedor, ou fornecedor-wide.

Este requisito será simplificado, i.e., o utilizador apenas pode cancelar encomendas na sua totalidade, não pode cancelar apenas um produto de uma encomenda.

```mermaid
sequenceDiagram
    actor A as Alice
    participant O as Order
    participant Su as Supplier
    participant OI as OrderItem
    participant P as Product
    participant PG as PaymentGateway
    participant S as Stripe

    A ->> O: getStatus()
    O -->> A: status
    Note left of O: If order shipped then we cannot cancel
    A ->> O: cancel()
    
    Note right of O: Implementation of different cancellation policies
    alt less than 24h since order
        O ->> O: cancel()
        O -->> A: orderCancelled
    else less than maximum supplier defined time since order
        Note right of O: Calculate max time to cancel
        O ->> OI: getProducts()
        OI -->> O: product[]

        loop for each product
            O ->> Su: getMaxCancelTime()
            Su -->> O: cancelTime
        end

        Note left of O: Whether we can cancel
        alt less than MAX(cancelTime) since order
            O ->> O: cancel()
            O -->> A: orderCancelled
        else more than MAX(cancelTime) since order
            O -->> A: orderCannotBeCancelled
        end
    else less than all of products return policy
        Note right of O: Calculate max time to cancel
        O ->> OI: getProducts()
        OI -->> O: product[]

        loop for each product
            O ->> P: getMaxCancelTime()
            P -->> O: cancelTime
        end

        Note left of O: Whether we can cancel
        alt less than MAX(cancelTime) since order
            O ->> O: cancel()
            O -->> A: orderCancelled
        else more than MAX(cancelTime) since order
            O -->> A: orderCannotBeCancelled
        end
    end
    
    Note right of O: If order paid then refund
    alt Order paid
        O ->> PG: refund(id)
        PG ->> S: refund(id)
        Note right of S: Stripe will notify us through webhook
        S -->> PG: refund
        PG -->> O: refund
        O -->> A: notifyUserRefund
    end
```

## RF-14: Notificação sobre a saída de produtos encomendados de um fornecedor

**Interpretação**: Notificação a cada `ShipmentEvent` em vez de apenas uma notificação aquando da saída dos produtos de uma `Order`.

Desta forma, não "existe" um *endpoint* para notificar o utilizador sobre a saída de produtos de uma encomenda, mas sim, um evento despoletado. Utilização do padrão ***Observer***?

Primeiramente, necessitamos de persistir as notificações, podendo assim ser vistas novamente posteriormente, e não perdidas (*ephemeral*).

Após isso, devemos notificar as partes interessadas, os *subscribers*. Após persistir a notificação, podemos notificar os *subscribers* de forma assíncrona, i.e., em *background*, através de *sockets* (caso o *subscriber* estiver à escuta), e através do *e-mail*.

Notas de implementação: Uso do padrão ***Strategy*** Criar uma interface `NotificationStrategy` e implementar `SocketNotificationStrategy` assim como `EmailNotificationStrategy` que, através dos `SocketGateway` e `EmailGateway`, respetivamente, notificam os *subscribers*.

```mermaid
sequenceDiagram
    actor S as System
    participant SE as ShipmentEvent
    participant N as Notification
    participant U as User
    participant NMF as NotificationMessageFactory
    # TODO these socket and email gateways may implement the same interface and be considered different Notification Strategies
    participant SG as SocketGateway
    participant EG as EmailGateway

    Note right of S: A System component makes a ShipmentEvent
    S ->> SE: new({orderId, status, address, etc})
    activate SE

    SE ->> SE: persist()

    SE ->> N: new(shippingEvent, subscriber)
    activate N
    N ->> N: persist()

    SE -->> S: shipmentEvent
    deactivate SE

    Note right of N: Who should receive the event notification

    loop for each notification.subscribers
        Note right of N: Push notification
        N ->> SG: findSocket(subscriber)
        alt Socket found
            SG -->> N: socket
            N ->> NMF: makeSocketNotificationMessage(shipmentEvent) # entity, type, emitter, etc
            NMF -->> N: message
            N ->> SG: send(socket, message)
            SG -->> N: notificationSent
        end

        # TODO we will allow disabling types of notifications sent through email, so we must check we can send this type of notification
        N ->> U: getNotificationPreferences()
        U -->> N: notificationPreferences

        Note right of N: Email notification
        alt canSendEmail(notificationPreferences, shipmentEvent)
            N ->> U: getEmail()
            U -->> N: email
            N ->> NMF: makeEmailNotificationMessage(shipmentEvent)
            NMF -->> N: message
            N ->> EG: send(email, message)
            EG -->> N: notificationSent
        end
    end

    deactivate N
```

## RF-15: Notificação sobre a chegada iminente de encomenda

Notar o diagrama de sequência acima. Visto que o `ShipmentEvent` é um evento que ocorre quando a encomenda tem uma atualização no envio, podemos utilizar o mesmo evento para notificar o utilizador sobre a chegada iminente da encomenda.

## RF-16: Visualização de relatório do impacto local das suas encomenda

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
