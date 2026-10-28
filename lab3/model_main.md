# Sequence Model: Основний сценарій UC-02 (Успішна оплата)

```mermaid
sequenceDiagram
    autonumber off
    actor Gamer as Гравець
    participant Order as OrderService
    participant Library as LibraryService
    participant Bank as PaymentGateway

    Gamer->>Order: Оформити замовлення (Кошик)
    Order->>Library: Проверить наявність гри
    Library-->>Order: Гри немає (можна купувати)
    
    Order->>Order: Створити замовлення (Pending)
    Order->>Bank: Провести платіж
    
    alt Платіж успішний
        Bank-->>Order: Платіж підтверджено
        Order->>Order: Змінити статус на Completed
        Order->>Library: Додати гру до бібліотеки
        Order-->>Gamer: Успішна покупка!
    else Платіж відхилено
        Bank-->>Order: Помилка оплати
        Order->>Order: Скасувати замовлення
        Order-->>Gamer: Помилка: платіж не пройшов
    end
