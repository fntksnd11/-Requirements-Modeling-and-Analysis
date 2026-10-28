# Sequence Model: Альтернативний сценарій UC-02 (Перевірка кошика, дублікат або DLC)

```mermaid
sequenceDiagram
    autonumber
    actor Gamer
    participant OrderService as OrderService <br>(ORDER, ORDER_ITEM)
    participant LibraryService as LibraryService <br>(USER_LIBRARY)
    participant CatalogService as CatalogService <br>(APP)

    Gamer->>+OrderService: submitOrder(cartId, paymentDetails)
    
    loop Для кожного товару в кошику (Validate Cart Items)
        OrderService->>+CatalogService: getAppDetails(appId)
        CatalogService-->>-OrderService: appDetails(appType, parentGameId)
    end

    opt Перевірка на DLC (REQ-CD-06)
        OrderService->>+LibraryService: checkParentGameOwnership(userId, parentGameId)
        LibraryService-->>-OrderService: parentOwnershipResult
    end

    OrderService->>+LibraryService: validateOwnership(userId, appId)
    LibraryService-->>-OrderService: isAlreadyOwned (True)

    alt Гра вже придбана або відсутня базова гра DLC
        OrderService-->>Gamer: checkoutRejectedError("App already in library or base game missing")
    end
    deactivate OrderService
