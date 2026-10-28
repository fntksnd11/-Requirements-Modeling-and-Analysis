# Sequence Model: Основний сценарій UC-02 (Успішна оплата)

```mermaid
sequenceDiagram
    autonumber
    actor Gamer
    participant OrderService as OrderService <br>(ORDER, ORDER_ITEM)
    participant LibraryService as LibraryService <br>(USER_LIBRARY)
    participant PaymentGateway as PaymentGateway <br>(<<System>>)

    Gamer->>+OrderService: submitOrder(cartId, paymentDetails)
    OrderService->>+LibraryService: validateOwnership(userId, appId)
    LibraryService-->>-OrderService: ownershipConfirmed (Not Owned)
    
    OrderService->>OrderService: createOrder(status='Pending')
    OrderService->>+PaymentGateway: processPayment(orderId, amount)
    
    alt Платіж успішний (Payment Successful)
        PaymentGateway-->>OrderService: paymentSuccess(transactionId)
        OrderService->>OrderService: updateOrderStatus(orderId, 'Completed')
        OrderService->>+LibraryService: grantLicense(userId, appId)
        LibraryService-->>-OrderService: licenseGranted()
        OrderService-->>Gamer: orderCompletedConfirmation(orderId)
    else Платіж відхилено (Payment Declined)
        PaymentGateway-->>-OrderService: paymentFailed(errorCode)
        OrderService->>OrderService: updateOrderStatus(orderId, 'Cancelled')
        OrderService-->>Gamer: orderFailedError("Payment Declined")
    end
    deactivate OrderService
