# Diagramas de Sequência dos Requisitos Funcionais

## TOC
- [Diagramas de Sequência dos Requisitos Funcionais](#diagramas-de-sequência-dos-requisitos-funcionais)
  - [TOC](#toc)
  - [Notas sobre API](#notas-sobre-api)
    - [Filtros na QueryString](#filtros-na-querystring)
  - [RF-01 - Navegação pelos produtos através da hierarquia de categorias](#rf-01---navegação-pelos-produtos-através-da-hierarquia-de-categorias)
  - [RF-02: Visualização de produtos e seus fornecedores](#rf-02-visualização-de-produtos-e-seus-fornecedores)
  - [RF-03: Pesquisa de produtos através dos campos comuns a todos os produtos](#rf-03-pesquisa-de-produtos-através-dos-campos-comuns-a-todos-os-produtos)
  - [RF-04: Colocação, consulta, e remoção de produtos num cesto de compra](#rf-04-colocação-consulta-e-remoção-de-produtos-num-cesto-de-compra)
    - [Colocação](#colocação)
    - [Consulta](#consulta)
    - [Remoção](#remoção)
  - [RF-05: Criação de uma conta no sistema](#rf-05-criação-de-uma-conta-no-sistema)
  - [RF-06: Fazer login no sistema](#rf-06-fazer-login-no-sistema)
  - [RF-07: Edição dos dados e remoção da sua conta no sistema](#rf-07-edição-dos-dados-e-remoção-da-sua-conta-no-sistema)
    - [Edição](#edição)
    - [Remoção](#remoção-1)
  - [RF-08: Pesquisa de produtos através dos campos específicos das categorias](#rf-08-pesquisa-de-produtos-através-dos-campos-específicos-das-categorias)
  - [RF-09: Visualização do histórico de encomendas e seus detalhes](#rf-09-visualização-do-histórico-de-encomendas-e-seus-detalhes)
  - [RF-10: Comparação de dois produtos, com as diferenças em destaque](#rf-10-comparação-de-dois-produtos-com-as-diferenças-em-destaque)
  - [RF-11: Encomenda dos produtos no cesto de compras](#rf-11-encomenda-dos-produtos-no-cesto-de-compras)
  - [RF-12: Pagamento de encomenda recorrendo a um sistema externo](#rf-12-pagamento-de-encomenda-recorrendo-a-um-sistema-externo)
  - [RF-13: Cancelamento de encomenda, desde que dentro de um dado prazo](#rf-13-cancelamento-de-encomenda-desde-que-dentro-de-um-dado-prazo)
  - [RF-14: Notificação sobre a saída de produtos encomendados de um fornecedor](#rf-14-notificação-sobre-a-saída-de-produtos-encomendados-de-um-fornecedor)
  - [RF-15: Notificação sobre a chegada iminente de encomenda](#rf-15-notificação-sobre-a-chegada-iminente-de-encomenda)
  - [RF-16: Visualização de relatório do impacto local das suas encomenda](#rf-16-visualização-de-relatório-do-impacto-local-das-suas-encomenda)
  - [RF-17: Exportação dos dados das encomendas para ficheiros JSON](#rf-17-exportação-dos-dados-das-encomendas-para-ficheiros-json)
  - [RF-18: Criação, gestão, e remoção de unidade de produção](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção)
    - [Criação](#criação)
    - [Gestão](#gestão)
    - [Remoção](#remoção-2)
  - [RF-19: Criação, gestão, e remoção de produto, e ligação a unidade de produção](#rf-19-criação-gestão-e-remoção-de-produto-e-ligação-a-unidade-de-produção)
    - [Criação](#criação-1)
    - [Gestão](#gestão-1)
    - [Remoção](#remoção-3)
    - [Ligação a unidade de produção](#ligação-a-unidade-de-produção)
  - [RF-20: Visualização de unidade de produção e dos seus produtos](#rf-20-visualização-de-unidade-de-produção-e-dos-seus-produtos)
    - [Visualização de unidade de produção](#visualização-de-unidade-de-produção)
    - [Visualização dos produtos da unidade de produção](#visualização-dos-produtos-da-unidade-de-produção)
  - [RF-21: Criação, edição, e remoção de veículo de transporte de produtos](#rf-21-criação-edição-e-remoção-de-veículo-de-transporte-de-produtos)
    - [Criação](#criação-2)
    - [Edição](#edição-1)
    - [Remoção](#remoção-4)
  - [RF-22: Notificação sobre encomenda de consumidor](#rf-22-notificação-sobre-encomenda-de-consumidor)
  - [RF-23: Visualização de encomenda de consumidor](#rf-23-visualização-de-encomenda-de-consumidor)
  - [RF-24: Colocação de produto encomendado em veículo de transporte disponível](#rf-24-colocação-de-produto-encomendado-em-veículo-de-transporte-disponível)
  - [RF-25: Registo de saída de veículo de transporte com produtos encomendados](#rf-25-registo-de-saída-de-veículo-de-transporte-com-produtos-encomendados)
  - [RF-26: Registo de chegada iminente de encomenda a casa do consumidor](#rf-26-registo-de-chegada-iminente-de-encomenda-a-casa-do-consumidor)
  - [RF-27: Visualização de relatório do impacto local das vendas dos seus produtos](#rf-27-visualização-de-relatório-do-impacto-local-das-vendas-dos-seus-produtos)
  - [RF-28: Desativação e reativação de qualquer conta de utilizador no sistema](#rf-28-desativação-e-reativação-de-qualquer-conta-de-utilizador-no-sistema)
  - [RF-29: Visualização de relatório do impacto local das vendas de produtos](#rf-29-visualização-de-relatório-do-impacto-local-das-vendas-de-produtos)

## Notas sobre API

### Filtros na QueryString

Exemplo:

```md
filter[address][city]=London&
filter[address][street]=12+High+Street&
filter[location][]=10&
filter[location][]=20&
filter[name]=David&
filter[nationality]=Danish
```

irá construir o seguinte filtro:

```json
filter: {
  "name": "David",
  "nationality": "Danish",
  "address": {
    "street": "12 High Street",
    "city": "London",
  },
  "location": [10, 20],
}
```

## RF-01 - Navegação pelos produtos através da hierarquia de categorias

<!-- TODO isto em árvore? É muito custoso na BD gerar isto? Caching? -->
Na API temos o *endpoint* GET `/categories` para obter as categorias na sua forma hierárquica.

<!-- TODO ter a certeza desta forma, ou se faremos algo tipo /products?categoryId=1,2 -->
A partir desse temos conhecimento dos *IDs* aos quais nos iremos referir no filtro.

O *endpoint* a usar para listar os `ProductSpec` é `/products`

Exemplo: `/products?filter[categoryId][]=1&filter[categoryId][]=2`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as CategoryGateway
    participant PSG as ProductSpecGateway

    A ->> API: GET /categories
    API ->> CG : getTree()
    CG -->> API: category[]
    API --> A: category[]

    Note right of A: Alice wants the products from the categories with ID 1 and 2
    A ->> API: GET /products?filter[categoryId][]=1&filter[categoryId][]=2
    API ->> PSG: findFromCategory([1, 2])
    PSG -->> API: products[]
    API -->> A: products[]
```

## RF-02: Visualização de produtos e seus fornecedores

**Interpretação**: Visualização da especificação dos produtos [e noutra página] dos fornecedores que vendem essa especificação.
No máximo teríamos um ou dois fornecedores que vendem cada produto na lista de produtos.

A interface irá ser semelhante a:

- Uma lista principal, paginada, de `ProductSpec`, com preço mínimo, máximo, e médio dos fornecedores.
- Após clicar num `ProductSpec`, irá abrir uma página com mais detalhes sobre o `ProductSpec` tais como uma descrição mais detalhada e os valores dos *fields* das categorias. Terá de ter também um botão para comparar com outros produtos. Poderá, eventualmente, ter o histórico dos preços dos diversos fornecedores.
  - Irá ter também a lista de `ProducerProduct`, mostrando o fornecedor e o preço. Ao clicar somos redirecionados para a página do `ProducerProduct`.
- A página do `ProducerProduct` terá o histórico de preços desse produto e outras informações pertinentes, se bem que já deve estar tudo na página do `ProductSpec`, tornando a página atual mais direcionada à compra.

Através do *endpoint* GET `/products` podemos obter os `ProductSpec`.

<!-- TODO bom nome para o endpoint? confuso? ideias? -->
Contudo, para obter todos os `ProducerProduct` (e por consequência os fornecedores) de um `ProductSpec` podemos usar o endpoint `/products/{id}/products` que nos devolve todos os fornecedores que fornecem esse produto.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant PSG as ProductSpecGateway
    participant PPG as ProducerProductGateway

    A ->> API: GET /products
    API ->> PSG: find()
    PSG -->> API: productSpec[]
    API -->> A: productSpec[]

    Note right of A: Alice wants to see the Producers of the ProductSpec with ID 1
    A ->> API: GET /products/1/products
    API ->> PPG: findFromSpec(1)
    PSG -->> API: producerProduct[]
    API -->> A: producerProduct[]
    Note right of A: producerProduct[] contains the Producer object
```

## RF-03: Pesquisa de produtos através dos campos comuns a todos os produtos

<!-- TODO enumerar os campos comuns a todos os produtos -->
Podemos usar o *endpoint* GET `/products/search?q=`, definindo o parâmetro `q` com o termo que queremos pesquisar. A pesquisa será relativa as campos comuns a todos os produtos, sendo estes o nome, TODO completar.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant PSC as ProductSpecGateway

    A ->> API: GET /products/search?q=iphone
    API ->> PSC: find("iphone")
    PSC -->> API: productSpec[]
    API -->> A: productSpec[]
```

## RF-04: Colocação, consulta, e remoção de produtos num cesto de compra

### Colocação
<!-- TODO endpoints /cart -->
Para colocar um `ProducerProduct` no carrinho de compras podemos usar o endpoint `/consumers/{consumerId}/cart/products/{productId}` com o método `PUT`, em que `productId` é o id do `ProducerProduct` que queremos colocar no carrinho.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CaG as CartGateway
    participant PPG as ProducerProductGateway
    participant CaIF as CartItemFactory
    participant CaI as CartItem
    participant CaIG as CartItemGateway

    A ->> API: PUT /consumers/{consumerId}/cart/products/1
    API ->> CaG: get(consumerId)
    CaG -->> API: cart
    API ->> PPG: get(1)
    PPG -->> API: producerProduct

    Note right of A: Alice wants to add the ProducerProduct with ID 1 to her cart
    API ->> CaIF: create(cart, producerProduct)

    CaIF ->> CaI: new(cart, producerProduct, quantity)
    CaI -->> CaIF: cartItem

    CaIF ->> CaIG: insert(cartItem)
    CaIG -->> CaIF: savedCartItem

    CaIF -->> API: savedCartItem
    
    API -->> A: savedCartItem
```

### Consulta

Para consultar os produtos no carrinho podemos usar o endpoint `/consumers/{consumerId}/cart/products` com o método `GET`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CaG as CartGateway
    participant CaIG as CartItemGateway

    A ->> API: GET /consumers/{consumerId}/cart/products
    API ->> CaG: get(consumerId)
    CaG -->> API: cart
    API ->> CaIG: findFromCart(cart)
    CaIG -->> API: cartItem[]
    API -->> A: cartItem[]
```

### Remoção
<!--  -->
Remover um produto do carrinho é feito através do endpoint `/consumers/{consumerId}/cart/products/{productId}` com o método `DELETE`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CaIG as CartItemGateway

    A ->> API: DELETE /consumers/{consumerId}/cart/products/{producerProductId}
    API ->> CaIG: get(consumerId, producerProductId)
    CaIG -->> API: cartItem
    API ->> CaIG: delete(cartItem)
    CaIG -->> API: true
    API -->> A: true
```

## RF-05: Criação de uma conta no sistema

1. Criar uma conta sem credenciais relacionadas. Feito através dum `POST` a `/consumers` ou `/producers` com os dados necessários, sendo este *endpoint* **não** protegido.
2. Após a criação do utilizador com sucesso, solicitar a adição de uma credencial (dos vários tipos `authType` possíveis: `password`, `google`, `facebook`, `twitter`)
   1. Autenticar o utilizador no serviço externo
   2. Obtenção de um token do serviço externo
   3. Criação da credencial ao utilizador através de `POST` a `/consumers/{consumerId}/credentials` ou `/producers/{producerId}/credentials`, dependendo do tipo de utilizador.

   - Até à adição da credencial, o utilizador ficará num estado em que apenas existe partialmente. Se não adicionarmos uma credencial, o utilizador não se poderá autenticar, ocupando recursos de forma desnecessária.

Iremos apenas contemplar a criação de um `Consumer`, mas o processo é idêntico para um `Producer`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CF as ConsumerFactory
    participant C as Consumer
    participant CG as ConsumerGateway
    participant Au as AuthService
    participant F as Firebase
    participant CredF as CredentialFactory
    participant Cred as Credential
    participant CredG as CredentialGateway
    participant EAu as ExternalAuthGateway

    Note right of A: First, we show the user creation form
    A ->> API: POST /consumers

    API ->> CF: create({name, email, ...})

    CF ->> C: new({name, email, ...})
    C -->> CF: consumer

    CF ->> CG: insert(consumer)
    CG -->> CF: savedConsumer

    CF -->> API: savedConsumer

    API ->> Au: createAccessToken(consumer)
    Au -->> API: temporaryAccessToken

    API -->> A: temporaryAccessToken

    A ->> F: authenticate(email, password)
    F -->> A: token

    Note right of A: Then, user is asked to add a credential
    Note right of A: token is sent in headers
    A ->> API: POST /consumers/{consumerId}/credentials
    API ->> Au: authenticate(temporaryAccessToken)
    Au ->> CredG: get(temporaryAccessToken)
    CredG -->> Au: consumer
    Au -->> API: consumer

    API ->> CredG: checkUnique(token)
    CredG -->> API: true

    API ->> Au: addCredential(consumer, {authType: "password", token})

    Note right of Au: We may be able to verify locally, skipping this step
    alt verify token
        Au ->> EAu: verifyToken(extToken)
        EAu -->> Au: extToken
    end

    Au ->> Cred: new({authType: "password", token})
    Cred -->> Au: credential
    Au ->> CredG: create(credential)
    CredG -->> Au: credential
    Au -->> API: credential
    API -->> A: credential
```

## RF-06: Fazer login no sistema

Autenticar o utilizador no serviço externo através do *frontend* e obter um *token* do serviço externo.

Usar o *endpoint* `/auth/login` com o método `POST`, forneçendo o *token* no *body* de forma a trocá-lo por um *access token* do nosso sistema.

```mermaid
sequenceDiagram
    actor A as Alice
    participant F as Firebase
    participant API
    participant Au as AuthService
    participant EAu as ExternalAuthGateway
    participant CredG as CredentialGateway
    participant CG as ConsumerGateway
    participant C as Consumer

    A ->> F: authenticate()
    F -->> A: extToken

    A ->> API: POST /auth/login
    API ->> Au: authenticate(extToken)

    Note right of Au: We may be able to verify locally, skipping this step
    alt verify token
        Au ->> EAu: verifyToken(extToken)
        EAu -->> Au: extToken
    end

    Au ->> CredG: get(extToken)
    CredG -->> Au: credential

    Note right of Au: We can apply this to Producer as well
    Au ->> CG: get(credential.userId)
    CG -->> Au: consumer

    Au ->> C: canLogin()
    C -->> Au: true

    Au ->> Au: generateAccessToken(credential)
    Au -->> API: accessToken
    API -->> A: accessToken    
```

## RF-07: Edição dos dados e remoção da sua conta no sistema

<!-- TODO rever por causa de usar um User (modelo Concrete) -->

O aplicado em `/consumers` é aplicado da mesma forma em `/producers`.

### Edição

Para editar os dados do utilizador podemos usar o *endpoint* `/consumers/{consumerId}` com o método `PUT`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway
    participant C as Consumer

    A ->> API: PUT /consumers/{consumerId}
    API ->> CG: get(consumerId)
    activate C
    CG -->> API: consumer
    Note right of API: we update the consumer
    API ->> CG: update(consumer)
    deactivate C
    CG -->> API: updatedConsumer
    API -->> A: updatedConsumer
```

### Remoção

<!-- TODO rever por causa de usar um User (modelo Concrete) -->

Para remover a conta do utilizador podemos usar o *endpoint* `/consumers/{consumerId}` com o método `DELETE`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway

    A ->> API: DELETE /consumers/{consumerId}
    API ->> CG: delete(consumerId)
    CG -->> API: deletedConsumer
    API -->> A: deletedConsumer
```

## RF-08: Pesquisa de produtos através dos campos específicos das categorias

Usando o mesmo *endpoint* `/products/search` do [rf-03](#rf-03-pesquisa-de-produtos-através-dos-campos-comuns-a-todos-os-produtos) mas adicionando os campos específicos das categorias.

Ver [notas da api](#notas-sobre-api) para detalhes de utilização de filtros.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant PG as ProductGateway

    A ->> API: GET /products/search?filter[category][name]=fruit
    API ->> PG: find({category: {name: "fruit"}})
    PG -->> API: productSpec[]
    API -->> A: productSpec[]
```

## RF-09: Visualização do histórico de encomendas e seus detalhes

Para obter o histórico de encomendas do utilizador podemos usar o *endpoint* `/consumers/{consumerId}/orders` com o método `GET`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway
    participant OG as OrderGateway

    A ->> API: GET /consumers/{consumerId}/orders
    API ->> CG: get(consumerId)
    CG -->> API: consumer

    API ->> OG: findFromConsumer(consumer)
    OG -->> API: order[]
    API -->> A: order[]
```

## RF-10: Comparação de dois produtos, com as diferenças em destaque

Obter os dois `ProductSpec` e iterar sobre as propriedades (e valores das categorias) de forma a saber quais diferem em valor.

Devemos definir se é possível comparar dois `ProductSpec` de categorias diferentes.

Convém definir se podemos comparar `ProducerProduct` com `ProducerProduct`, possibilitando comparar o mesmo `ProductSpec` de fornecedores diferentes, `ProductSpec` com `ProductSpec`, ou ambos.

1. Com a primeira podemos comparar custos e impacto na comunidade
    - A especificação da API seria um `GET` a `/producers/{producerId}/products/{product1}/compare/{product2}`.
      - Isto parece estranho porque semânticamente estariamos a comparar produtos de um fornecedor com outro produto do mesmo fornecedor (e não outro produto qualquer)
2. Com a segunda só queremos saber das diferenças da especificação do produto, que nem inclui o preço
    - A especificação da API seria um `GET` a `/products/{product1}/compare/{product2}`.

Podemos implementar ambos, mas acho que o primeiro é mais interessante.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant PSG as ProductSpecGateway
    participant PS as ProductSpec

    A ->> API: GET /products/{product1}/compare/{product2}

    API ->> PSG: get(1)
    PSG -->> API: product1

    API ->> PSG: get(2)
    PSG -->> API: product2

    API ->> PS: product1.compare(product2)
    PS -->> API: differences

    API -->> A: differences
```

## RF-11: Encomenda dos produtos no cesto de compras

A especificação da API seria um `POST` a `/consumers/{consumerId}/orders` com o *body* a conter o `cartId`.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CaG as CartGateway
    participant OF as OrderFactory
    participant O as Order
    participant C as Cart
    participant OG as OrderGateway
    participant CIG as CartItemGateway
    participant OIF as OrderItemFactory
    participant OI as OrderItem
    participant OIG as OrderItemGateway

    A ->> API: POST /consumers/{consumerId}/orders

    API ->> CaG: get(consumerId)
    activate C
    CaG -->> API: cart

    API ->> OF: createFromCart(cart)
    OF ->> O: new()

    OF ->> CIG: findFromCart(cart)
    CIG -->> OF: cartItem[]
    deactivate C

    loop for each cartItem.product
        OF ->> OIF: createFromCartItem(order, cartItem)
        OIF ->> OI: new(order, cartItem)
        OI -->> OIF: orderItem
        OIF ->> OIG: insert(orderItem)
        OIG -->> OIF: insertedOrderItem

        OF ->> O: addItem(orderItem)
        O -->> OF: addedOrderItem
    end

    OF ->> OG: insert(order)
    OG -->> OF: insertedOrder
    OF -->> API: order
    API -->> A: order
```

## RF-12: Pagamento de encomenda recorrendo a um sistema externo

Serviço externo: [**Stripe**](https://stripe.com), usando o [**Stripe Checkout**](https://stripe.com/docs/payments/checkout).

Começamos então por fazer um pedido `POST` ao *endpoint* `/consumers/{consumerId}/orders/{orderId}/checkout` irá criar um *checkout session* no *Stripe* e redirecionar o utilizador para o preenchimento de dados relativos ao pagamento, do lado do *Stripe*.

Após a introdução dos dados de pagamento, o *Stripe* irá confirmar o sucesso da operação ao submeter um pedido `POST` a `/payments/webhook`, *aka* *webhook*.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway
    participant OG as OrderGateway
    participant O as Order
    participant PG as PaymentGateway
    participant S as Stripe
    participant Su as Supplier

    A ->> API: POST /consumers/{consumerId}/orders/{orderId}/checkout

    API ->> CG: get(consumerId)
    CG -->> API: consumer

    API ->> OG: get(orderId)
    OG -->> API: order

    API ->> PG: createCheckoutSession(order)
    PG ->> S: createCheckoutSession(order)
    S -->> PG: checkoutSession
    PG -->> API: checkoutSession

    API -->> A: checkoutSession

    Note right of A: Redirect to checkoutSession.url
    Note right of A: Fill payment details and submit to external service

    S ->> API: POST /payments/webhook
    API ->> PG: handleWebhookEvent()
    PG ->> OG: updateStatus(order, paid)
    OG -->> PG: order
    PG -->> API: order

    Note right of O: Notify subscribers. See RF-14

    API -->> S: 200 OK
```

## RF-13: Cancelamento de encomenda, desde que dentro de um dado prazo

A especificação da API seria um `DELETE` a `/consumers/{consumerId}/orders/{orderId}`.

<!-- TODO -->
Importante!!! Falta definir o prazo de cancelamento e outras regras.

Sugeria que o prazo de cancelamento fosse de 24h, mas que o utilizador pudesse cancelar a qualquer momento, desde que o produto ainda não tivesse sido enviado.

Também é possível definir um prazo de cancelamento para cada produto, definido pelo fornecedor, ou fornecedor-wide.

Este requisito será simplificado, i.e., o utilizador apenas pode cancelar encomendas na sua totalidade, não pode cancelar apenas um produto de uma encomenda.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway
    participant OG as OrderGateway
    participant O as Order
    participant PG as PaymentGateway
    participant S as Stripe

    A ->> API: DELETE /consumers/{consumerId}/orders/{orderId}

    API ->> CG: get(consumerId)
    CG -->> API: consumer

    API ->> OG: get(orderId)
    OG -->> API: order

    Note right of API: Check if order is cancelable
    API ->> O: canCancel()
    O -->> API: true

    Note right of API: Affect state, making order cancelled
    API ->> O: cancel()
    O -->> API: order

    API ->> OG: update(order)
    OG -->> API: updatedOrder

    alt Order paid
        API ->> PG: refund(updatedOrder.payment)
        PG ->> S: refund(id)
        Note right of S: Stripe will notify us through webhook
        S ->> API: POST /payments/webhook
        API ->> PG: handleWebhookEvent()
        Note right of O: Notify subscribers. See RF-14
        PG ->> O: setStatus(refunded)
        O -->> PG: order
        PG -->> API: order
        API -->> S: 200 OK
    end

    API -->> A: updatedOrder        
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
    participant SEF as ShipmentEventFactory
    participant SE as ShipmentEvent
    participant SEG as ShipmentEventGateway
    participant NF as NotificationFactory
    participant N as Notification
    participant NFG as NotificationGateway
    %% TODO we need user
    participant U as User
    participant NMF as NotificationMessageFactory
    # SocketGateway is NOT a data table gateway. It is a facade to the socket server
    participant SG as SocketGateway
    participant EG as EmailGateway

    Note right of S: A System component makes a ShipmentEvent
    S ->> SEF: create()
    SEF ->> SE: new({orderId, status, address, etc})
    activate SE
    SE -->> SEF: shipmentEvent

    SEF ->> SEG: insert(shipmentEvent)
    SEG -->> SEF: savedShipmentEvent
    SEF -->> S: savedShipmentEvent
    deactivate SE

    # TODO confirmar que notificações são criadas doutras formas
    SEF ->> NF: createShipmentNotification(shippingEvent, subscriber)

    NF ->> N: new({shipmentEvent, subscriber})

    activate N
    N -->> NF: notification
    
    NF ->> NFG: insert(notification)
    NFG -->> NF: savedNotification
    
    Note right of N: Deliver notifications

    loop for each subscriber in notification.subscribers
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

Notar o diagrama de sequência [acima](#rf-14-notificação-sobre-a-saída-de-produtos-encomendados-de-um-fornecedor). Visto que o `ShipmentEvent` é um evento que ocorre quando a encomenda tem uma atualização no envio, podemos utilizar o mesmo evento para notificar o utilizador sobre a chegada iminente da encomenda.

## RF-16: Visualização de relatório do impacto local das suas encomenda

Não sei
<!-- TODO não faço ideia -->

## RF-17: Exportação dos dados das encomendas para ficheiros JSON

Um *endpoint* próprio para obter os dados tratados seria o mais eficiente. Depois, apenas seria necessário gravar a resposta num ficheiro (a *API* retorna sempre em *JSON*).

`/consumers/{consumerId}/orders/export`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant OG as OrderGateway

    A ->> API: GET /consumers/{consumerId}/orders/export

    Note right of API: We fetch the orders, loading the order items and the products, the Producer, and payment details
    API ->> OG: findFromConsumer(consumerId)
    OG -->> API: order[]

    API -->> A: order[]
```

## RF-18: Criação, gestão, e remoção de unidade de produção

É mais simples separar em três partes: criação, gestão, e remoção.

### Criação

A criação de uma unidade de produção é feita através de um *endpoint* próprio, que recebe os dados necessários para a criação da unidade de produção. Para tal, efetua-se `POST` a `/producers/{producerId}/units`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PUF as ProductionUnitFactory
    participant PU as ProductionUnit
    participant PUG as ProductionUnitGateway

    A ->> API: POST /producers/{producerId}/units

    API ->> PUF: create(producer, {name, description, etc})

    PUF ->> PU: new(producer, {name, description, etc})
    activate PU
    PU -->> PUF: unit

    PUF ->> PUG: insert(unit)
    PUG -->> PUF: savedUnit
    deactivate PU

    PUF -->> API: savedUnit
    API -->> A: savedUnit
```

### Gestão

A gestão de uma unidade de produção é feita através de um *endpoint* próprio, que recebe os dados necessários para a gestão da unidade de produção. Para tal, efetua-se `PUT` a `/producers/{producerId}/units/{unitId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant UG as UnitGateway
    participant PU as ProductionUnit

    A ->> API: PUT /producers/{producerId}/units/{unitId}

    API ->> UG: get(unitId)
    activate PU
    UG -->> API: unit

    Note right of API: We update the unit with the new data

    API ->> UG: update(updatedUnit)
    UG -->> API: savedUnit
    deactivate PU

    API -->> A: savedUnit
```

### Remoção

A remoção de uma unidade de produção é feita através de um *endpoint* próprio, que recebe os dados necessários para a remoção da unidade de produção. Para tal, efetua-se `DELETE` a `/producers/{producerId}/units/{unitId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PUG as ProductionUnitGateway

    A ->> API: DELETE /producers/{producerId}/units/{unitId}

    API ->> PUG: delete(unitId)
    PUG -->> API: deletedUnit

    API -->> A: deletedUnit
```

## RF-19: Criação, gestão, e remoção de produto, e ligação a unidade de produção

É mais simples separar em quatro partes: criação, gestão, remoção, e ligação a unidade de produção.

### Criação

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a criação de um produto é feita através de um *endpoint* próprio, que recebe os dados necessários para a criação do produto. Para tal, efetua-se `POST` a `/producers/{producerId}/products`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PF as ProductFactory
    participant P as Product
    participant PG as ProductGateway

    A ->> API: POST /producers/{producerId}/products

    API ->> PF: create(producer, {name, description, etc})

    PF ->> P: new(producer, {name, description, etc})
    activate P
    P -->> PF: product

    PF ->> PG: insert(product)
    PG -->> PF: savedProduct
    deactivate P

    PF -->> API: savedProduct
    API -->> A: savedProduct
```

### Gestão

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a gestão de um produto é feita através de um *endpoint* próprio, que recebe os dados necessários para a gestão do produto. Para tal, efetua-se `PUT` a `/producers/{producerId}/products/{productId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PG as ProductGateway
    participant P as Product
    participant PPF as ProductPriceFactory
    participant PP as ProductPrice
    participant PPG as ProductPriceGateway

    A ->> API: PUT /producers/{producerId}/products/{productId}

    API ->> PG: get(productId)
    activate P
    PG -->> API: product

    Note right of API: We update the product with the new data
    API ->> P: update({name, description, etc})
    
    alt price is changed
        P ->> PPF: create(product, {price})
        PPF ->> PP: new({product, currentPrice})
        activate PP
        PP -->> PPF: productPrice
        PPF ->> PPG: create(productPrice)
        PPG -->> PPF: savedProductPrice
        deactivate PP
        PPF -->> P: savedProductPrice

        Note right of P: price = currentPrice
    end


    P -->> API: updatedProduct

    API ->> PG: update(updatedProduct)
    PG -->> API: savedProduct
    deactivate P

    API -->> A: savedProduct
```

### Remoção

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a remoção de um produto é feita através de um *endpoint* próprio, que recebe os dados necessários para a remoção do produto. Para tal, efetua-se `DELETE` a `/producers/{producerId}/products/{productId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PG as ProductGateway

    A ->> API: DELETE /producers/{producerId}/products/{productId}

    API ->> PG: delete(productId)
    PG -->> API: deletedProduct

    API -->> A: deletedProduct
```

### Ligação a unidade de produção

Para ligar um produto a uma unidade de produção é necessário que ambos tenham sido criados previamente. Depois, resta apenas adicionar o produto à coleção de produtos da unidade, através de `PUT` a `/producers/{producerId}/units/{unitId}/products/{productId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant PUG as ProductionUnitGateway
    participant PU as ProductionUnit
    participant PG as ProductGateway
    
    A ->> API: PUT /producers/{producerId}/units/{unitId}/products/{productId}

    API ->> PUG: get(unitId)
    activate PU
    PUG -->> API: unit

    API ->> PG: get(productId)
    PG -->> API: product

    Note right of API: We add the product to the unit
    API ->> PU: addProduct(product)
    PU -->> API: unit
    
    API ->> PUG: update(updatedUnit)
    PUG -->> API: updatedUnit

    deactivate PU

    API -->> A: updatedUnit
```

## RF-20: Visualização de unidade de produção e dos seus produtos

Iremos novamente separar em duas partes: visualização de unidade de produção, e visualização dos produtos da unidade de produção.

### Visualização de unidade de produção

Apenas obtém os dados base da unidade de produção, através de `GET` a `/producers/{producerId}/units/{unitId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant UG as UnitGateway

    A ->> API: GET /producers/{producerId}/units/{unitId}

    API ->> UG: get(unitId)
    UG -->> API: unit

    API -->> A: unit
```

### Visualização dos produtos da unidade de produção

<!-- TODO este endpoint -->
De forma a obter a coleção de produtos da unidade de produção, efetua-se `GET` a `/producers/{producerId}/units/{unitId}/products`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant UG as ProductionUnitGateway
    participant PrPG as ProducerProductGateway
    
    A ->> API: GET /producers/{producerId}/units/{unitId}/products

    API ->> UG: get(unitId)
    UG -->> API: unit

    Note right of API: We get the products from the unit
    API ->> PrPG: findFromProductionUnit()
    PrPG -->> API: products

    API -->> A: products
```

## RF-21: Criação, edição, e remoção de veículo de transporte de produtos

Bastante semelhante a [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção).

Separando em três partes: criação, gestão, e remoção.

### Criação

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a criação de um veículo de transporte de produtos é feita através de um `POST` a `/producers/{producerId}/units/{unitId}/carriers`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant CF as CarrierFactory
    participant CG as CarrierGateway
    participant C as Carrier

    A ->> API: POST /producers/{producerId}/units/{unitId}/carriers

    Note right of API: We create the carrier
    API ->> CF: create(producerId, unitId, {name, description, etc})
    CF ->> C: new(producerId, unitId, {name, description, etc})
    activate C
    C -->> CF: carrier

    CF ->> CG: insert(carrier)
    CG -->> CF: savedCarrier
    deactivate C
    CF -->> API: savedCarrier

    API -->> A: savedCarrier
```

### Edição

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a edição de um *carrier* faz-se através de um `PUT` a `/producers/{producerId}/units/{unitId}/carriers/{carrierId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as CarrierGateway

    A ->> API: PUT /producers/{producerId}/units/{unitIt}/carriers/{carrierId}

    API ->> CG: get(carrierId)
    CG -->> API: carrier

    Note right of API: We edit the carrier

    API ->> CG: update(carrier)
    CG -->> API: savedCarrier

    API -->> A: savedCarrier
```

### Remoção

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a remoção de um *carrier* faz-se através de um `DELETE` a `/producers/{producerId}/units/{unitId}/carriers/{carrierId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant CG as CarrierGateway

    A ->> API: DELETE /producers/{producerId}/units/{unitId}/carriers/{carrierId}

    API ->> CG: delete(carrierId)
    CG -->> API: deletedCarrier

    API -->> A: deletedCarrier
```

## RF-22: Notificação sobre encomenda de consumidor

Podemos usar o mesmo evento `ShipmentEvent` para notificar o utilizador sobre a criação de uma encomenda.

Assim, iriamos ter um `ShipmentEvent` criado, cujo `ShipmentStatus` seria encomenda `criada`, porém `pending` seria mais rigoroso.

> Notar o diagrama de sequência [acima](#rf-14-notificação-sobre-a-saída-de-produtos-encomendados-de-um-fornecedor). Visto que o `ShipmentEvent` é um evento que ocorre quando a encomenda tem uma atualização no envio, podemos utilizar o mesmo evento para notificar o utilizador sobre a chegada iminente da encomenda.

## RF-23: Visualização de encomenda de consumidor

A visualização de uma encomenda de consumidor é feita através de um `GET` a `/consumers/{consumerId}/orders/{orderId}`

```mermaid
sequenceDiagram
    actor A as Alice
    participant API as API
    participant CG as ConsumerGateway
    participant OG as OrderGateway

    A ->> API: GET /consumers/{consumerId}/orders/{orderId}

    API ->> CG: get(consumerId)
    CG -->> API: consumer

    API ->> OG: get(orderId)
    OG -->> API: order
    API -->> A: order
```

## RF-24: Colocação de produto encomendado em veículo de transporte disponível

TODO

## RF-25: Registo de saída de veículo de transporte com produtos encomendados

Basicamente o mesmo que o [RF-14](#rf-14-notificação-sobre-a-saída-de-produtos-encomendados-de-um-fornecedor), mas com um `ShipmentStatus` diferente.

## RF-26: Registo de chegada iminente de encomenda a casa do consumidor

Basicamente o mesmo que o [RF-14](#rf-14-notificação-sobre-a-saída-de-produtos-encomendados-de-um-fornecedor), mas com um `ShipmentStatus` diferente.

## RF-27: Visualização de relatório do impacto local das vendas dos seus produtos

Não sei

## RF-28: Desativação e reativação de qualquer conta de utilizador no sistema

De forma semelhante ao [RF-18](#rf-18-criação-gestão-e-remoção-de-unidade-de-produção), a desativação de uma conta de utilizador faz-se através de um `PUT` a `/consumers/{consumerId}`, especificando o campo `disabledOn` com a data a partir da qual a conta fica desativada ou `null`, de forma a reativar a conta.

O mesmo se aplica aos produtores.

Poderíamos incluir um extra em que permite guardar o histórico de desativações e reativações, assim como quem despoletou a alteração e a razão.

```mermaid
sequenceDiagram
    actor A as Alice
    participant API
    participant CG as ConsumerGateway
    participant U as User


    A ->> API: PUT /consumers/{consumerId}

    API ->> CG: get(userId)
    CG -->> API: consumer

    Note right of API: We edit the disabledOn field

    API ->> CG: update(consumer)
    CG -->> API: savedConsumer

    API -->> A: savedConsumer
```

## RF-29: Visualização de relatório do impacto local das vendas de produtos

Não sei
