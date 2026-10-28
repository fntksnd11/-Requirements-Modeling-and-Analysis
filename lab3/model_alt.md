# Sequence Model: Альтернативний сценарій UC-02 (Перевірка кошика, дублікат або DLC)

```mermaid
sequenceDiagram
    autonumber off
    actor Gamer as Гравець
    participant Order as OrderService
    participant Library as LibraryService
    participant Catalog as CatalogService

    Gamer->>Order: Оформити замовлення (Кошик)
    
    loop Для кожного товару в кошику
        Order->>Catalog: Отримати деталі продукту (appId)
        Catalog-->>Order: Повернути appType та parentGameId
    end

    opt Перевірка на DLC
        Order->>Library: Перевірити наявність базової гри
        Library-->>Order: Результат перевірки parent game
    end

    Order->>Library: Перевірити наявність гри в бібліотеці
    Library-->>Order: Гра вже придбана (True)

    alt Гра вже є або базова гра відсутня
        Order-->>Gamer: Помилка: товар вже у бібліотеці або немає бази для DLC
    end
