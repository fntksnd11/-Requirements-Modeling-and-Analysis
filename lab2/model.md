```markdown
# Use Case Діаграма: Steam Digital Store

```plantuml
@startuml
left to right direction
actor "Gamer" as gamer
actor "Verified Publisher" as publisher
actor "Payment Gateway" as payment << System >>

gamer <|-- publisher : generalization

rectangle "Steam Digital Store System" {
    usecase "Browse Catalog" as UC_Browse
    usecase "Filter by Category" as UC_Filter
    usecase "Checkout Order" as UC_Checkout
    usecase "Process Payment" as UC_Payment
    usecase "Write Review" as UC_Review
    usecase "Publish App / DLC" as UC_Publish
    usecase "Validate App License" as UC_Validate

    gamer --> UC_Browse
    gamer --> UC_Checkout
    gamer --> UC_Review

    publisher --> UC_Publish

    UC_Browse <.. UC_Filter : <<extend>>
    UC_Checkout ..> UC_Payment : <<include>>
    UC_Review ..> UC_Validate : <<include>>

    UC_Payment --> payment
}
@enduml
