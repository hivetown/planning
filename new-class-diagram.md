# Diagrama de classes

```mermaid
classDiagram

    ProducerGateway --|> Producer
    
    Producer "1" -- "*" ProductionUnit
    Producer "*" -- "1" UserType
    ProductionUnitGateway --|> ProductionUnit

    ConsumerGateway --|> Consumer
    CartGateway --|> Cart
    Consumer "*" -- "1" UserType
    Consumer "1" -- "*" Order
    Consumer "1" -- "1" Cart
    
    ProductionUnit "1" -- "*" ProducerProduct
    ProductionUnit "1" -- "*" Carrier
    CarrierGateway --|> Carrier
    Carrier "*" -- "1" CarrierStatus
    
    
    Cart "1" -- "*" CartItem
    CartItemGateway --|> CartItem

    User <|-- Consumer
    User <|-- Producer
    User "*" -- "*" Address
    AddressGateway --|> Address

    OrderItemGateway --|> OrderItem
    Order "1" -- "*" OrderItem
    OrderGateway --|> Order

    OrderItem "1" -- "1" ProducerProduct
    ProducerProductGateway --|> ProducerProduct
    ProductSpecGateway --|> ProductSpec
    ProducerProduct "*" -- "1" ProductSpec

    ShipmentProducts "1" -- "*" ProducerProduct
    ShipmentEvent "1" -- "1" ShipmentProducts
    
    FieldGateway --|> Field
    ProductSpec "*" -- "*" Field
    ProductSpec "*" -- "*" Category
    CategoryGateway --|> Category

    ProducerProduct "*" -- "1" ProductStatus

    Field "*" -- "1" FieldType
    Category "*" -- "1" Category
    Category "*" -- "*" Field
    Field "1" -- "*" FieldPossibleValue

    Order "1" -- "*" ShipmentEvent
    ShipmentEvent "*" -- "1" ShipmentStatus
    ShipmentEventGateway --|> ShipmentEvent
    ShipmentStatusGateway --|> ShipmentStatus

    class User {
        <<abstract>>
        -number id
        -string name
        -string email
        -number phone
        -number vat
        -UserType type
        
        +canLogin() boolean
        %% TODO separate table?
        +getNotificationPreferences() unknown
    }

    class Consumer {
        -UserType type = CONSUMER
        -Cart cart
        -List~Order~ orders
        -List~Address~ addresses
    }

    class ConsumerGateway {
        +get(number id) Consumer
        +insert(Consumer consumer) Consumer
        +update(Consumer consumer) Consumer
        +delete(Consumer consumer) void
    }

    class Producer {
        -UserType type = PRODUCER
        List~ProductionUnit~ productionUnits
    }

    class ProducerGateway {
        +get(number id) Producer
        +insert(Producer producer) Producer
        +update(Producer producer) Producer
        +delete(Producer producer) void
    }

    class UserType {
        <<enum>>
        +CONSUMER
        +PRODUCER
    }

    class Address {
        -number id
        -number number
        -string door
        -number floor
        -string zipCode
        -string street
        -string parish
        -string county
        -string city
        -string district
        -number latitude
        -number longitude
    }

    class AddressGateway {
        +get(number id) Address
        +insert(Address address) Address
        +update(Address address) Address
        +delete(Address address) void
    }

    class ProductionUnit {
        -number id
        -string name
        -Address address
        -Producer producer
        -List~Carrier~ carriers
        -List~ProducerProduct~ products
    }

    class ProductionUnitGateway {
        +get(number id) ProductionUnit
        +insert(ProductionUnit productionUnit) ProductionUnit
        +update(ProductionUnit productionUnit) ProductionUnit
        +delete(ProductionUnit productionUnit) void
    }

    class Carrier {
        -number id
        -string licensePlate
        -ProductionUnit productionUnit
        -CarrierStatus status
    }

    class CarrierGateway {
        +get(number id) Carrier
        +insert(Carrier carrier) Carrier
        +update(Carrier carrier) Carrier
        +delete(Carrier carrier) void
    }

    class CarrierStatus {
        <<enum>>
        +AVAILABLE
        +UNAVAILABLE
    }

    class OrderItem {
        -number id
        -number quantity
        -number price
        -ProducerProduct product
        -Order order
    }

    class OrderItemGateway {
        +get(number id) OrderItem
        +insert(OrderItem orderItem) OrderItem
        +update(OrderItem orderItem) OrderItem
        +delete(OrderItem orderItem) void
    }

    class Order {
        -number id
        -Consumer consumer
        -Address shippingAddress
        -List~OrderItem~ items
        +addItem(OrderItem item) void
        +canCancel() void
        +cancel() void
    }

    class OrderGateway {
        +findFromConsumer(Consumer consumer) List~Order~
        +get(number id) Order
        +insert(Order order) Order
        +update(Order order) Order
        +delete(Order order) void
    }

    class ShipmentEvent {
        -number id
        -Date date
        -Shipment shipment
        -ShipmentStatus status
        -Address address
    }

    class ShipmentEventGateway {
        +get(number id) ShipmentEvent
        +insert(ShipmentEvent shipmentEvent) ShipmentEvent
        +update(ShipmentEvent shipmentEvent) ShipmentEvent
        +delete(ShipmentEvent shipmentEvent) void
    }

    class Shipment {
        -number id
        -Order order
        -Carrier carrier
        -List~ShipmentEvent~ events
        -List~ProducerProduct~ products
    }

    class ShipmentGateway {
        +get(number id) Shipment
        +insert(Shipment shipment) Shipment
        +update(Shipment shipment) Shipment
        +delete(Shipment shipment) void
    }

    %% TODO representa os tipos de shipment status: "em preparação", "em transporte", "entregue", etc
    class ShipmentStatus {
        -number id
        -string name
        -string description
    }

    %% TODO: necessário? só se for para o admin
    %% Só se for para a BD  
    class ShipmentStatusGateway {
        +get(number id) ShipmentStatus
        +insert(ShipmentStatus shipmentStatus) ShipmentStatus
        +update(ShipmentStatus shipmentStatus) ShipmentStatus
        +delete(ShipmentStatus shipmentStatus) void
    }

    class ProducerProduct {
        -number id
        -number currentPrice
        -Date productionDate
        -Producer producer
        -ProductionUnit productionUnit
        -ProductSpec specification
        -ProductStatus status
    }

    class ProducerProductGateway {
        +findFromSpec(ProductSpec spec) List~ProducerProduct~
        +findFromProductionUnit(ProductionUnit productionUnit) List~ProducerProduct~
        +get(number id) ProducerProduct
        +insert(ProducerProduct producerProduct) ProducerProduct
        +update(ProducerProduct producerProduct) ProducerProduct
        +delete(ProducerProduct producerProduct) void
    }

    class ProductSpec {
        -number id
        -string name
        -string description
        -List~string~ images
        -List~Category~ categories
        %% TODO fieldvalues
        -List~Field~ fields
        +compare(ProductSpec other)
    }

    class ProductSpecGateway {
        +findFromCategory(List~Category~ category) List~ProductSpec~
        +find(string query, ProductSpecSearchFilter filter) List~ProductSpec~
        +get(number id) ProductSpec
        +insert(ProductSpec productSpec) ProductSpec
        +update(ProductSpec productSpec) ProductSpec
        +delete(ProductSpec productSpec) void
    }

    class ProductStatus {
        <<enum>>
        +AVAILABLE
        +SOLD_OUT
    }

    class Category {
        -number id
        -string name
        -Category parent
        -List~Field~ fields
    }
    
    class CategoryGateway {
        +get(number id) Category
        +insert(Category category) Category
        +update(Category category) Category
        +delete(Category category) void
    }

    class Field {
        -number id
        -string name
        -string unit
        -FieldType type
        -List~FieldPossibleValue~ possibleValues
    }

    class FieldGateway {
        +get(number id) Field
        +insert(Field field) Field
        +update(Field field) Field
        +delete(Field field) void
    }

    class FieldPossibleValue {
        -Field field
        -String value
        +getField() Field
        +getValue() String
    }

    class FieldType{
        <<enum>>
        +TEXT
        +NUMBER
        +DATE
        +BOOLEAN
        +ENUM
    }

    class Cart {
        %% o id é o mesmo que o do consumidor
        -Consumer consumer
        -List~CartItem~ items
        +addProduct(ProducerProduct product, number quantity) void
    }

    class CartGateway {
        +get(number consumerId) Cart
        +insert(Cart cart) Cart
        +update(Cart cart) Cart
        +delete(Cart cart) void
    }

    class CartItem {
        %% tecnicamente é uma combinação dos ids do producerProduct e do cart
        -number id
        -number quantity
        -ProducerProduct product
        -Cart cart
    }

    class CartItemGateway {
        +findFromCart(Cart cart) List~CartItem~
        +get(number cartId, number producerProductId) CartItem
        +insert(CartItem cartItem) CartItem
        +update(CartItem cartItem) CartItem
        +delete(CartItem cartItem) void
    }

    class OrderFactory {
        +createFromCart(Cart cart) Order
    }

    class OrderItemFactory {
        +create(Order order, ProducerProduct product, number quantity) OrderItem
    }
```
