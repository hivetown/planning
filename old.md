    %% TB is also readable
    direction LR
    User "*" -- "*" Address 
    User <|-- Consumer
    User <|-- Producer

    AddressGateway --|> Address
    ProducerGateway --|> Producer
    ConsumerGateway --|> Consumer
    OrderGateway --|> Order
    ProductionUnitGateway --|> ProductionUnit
    CartGateway --|> Cart
    
    CarrierGateway --|> Carrier
    ProducerProductGateway --|> ProducerProduct
    ProductSpecGateway --|> ProductSpec
    FieldGateway --|> Field
    CategoryGateway --|> Category
    OrderItemGateway --|> OrderItem
    CartItemGateway --|> CartItem
    ShipmentEventGateway --|> ShipmentEvent
    ShipmentStatusGateway --|> ShipmentStatus


    Consumer "*" -- "1" UserType
    Consumer "1" -- "*" Order
    Consumer "1" -- "1" Cart

    Producer "*" -- "1" UserType
    Producer "1" -- "*" ProductionUnit

    ProductionUnit "1" -- "*" Carrier
    ProductionUnit "1" -- "*" ProducerProduct

    Carrier "*" -- "1" CarrierStatus

    OrderItem "1" -- "1" ProducerProduct

    Order "1" -- "*" OrderItem
    Order "1" -- "*" ShipmentEvent

    ShipmentEvent "*" -- "1" ShipmentStatus

    ProducerProduct "*" -- "1" ProductStatus
    ProducerProduct "*" -- "1" ProductSpec

    ProductSpec "*" -- "*" Field
    ProductSpec "*" -- "*" Category

    Category "*" -- "1" Category
    Category "*" -- "*" Field

    Field "*" -- "1" FieldType

    Cart "1" -- "*" CartItem