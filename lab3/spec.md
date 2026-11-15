## 1. Намір (Intent)
Зобразити за допомогою діаграм процес придбання гри, у трьох сценаріях UC-02.

## 2. Актори та цілі
- **Gamer**: покупець і гравець.
- **Verified Publisher**: успадковує `Gamer` (`Verified Publisher IS-A Gamer`).
- **Payment Gateway** (`<<System>>`):платіжній сервіс.

## 3. Функціональні вимоги у синтаксисі EARS
- **[REQ-UB-01]**: Система **SHALL** зберігати наскрізний атрибут часу гри (`playtime_hours`) безпосередньо у сутності зв'язку `USER_LIBRARY` між користувачем та додатком.
- **[REQ-EV-02]**: **WHEN** користувач ініціює додавання товару до кошика або відкриває сторінку чекауту, система **SHALL** перевіряти наявність гри у `USER_LIBRARY`.
- **[REQ-EV-03]**: **WHEN** замовлення (`ORDER`) успішно створено у статусі `Pending`, система починає **SHALL** ініціювати  запит до `Payment Gateway`.
- **[REQ-ST-04]**: **WHILE** замовлення перебуває у статусі `Pending`, система **SHALL** буде блокувати повторну покупку тієї самої позиції за допомогою блокування рядка БД (`SELECT ... FOR UPDATE`) .
- **[REQ-CD-05]**: **IF** отримано підтвердження платежу, **THEN** система **SHALL** перевести замовлення в статус `Completed` та оновити `USER_LIBRARY` в межах локальної бази даних.
- **[REQ-CD-06]**: **IF** обраний товар має `app_type = 'DLC'`, **THEN** система **SHALL** перевіряти наявність базової гри (`parent_game_id`) у бібліотеці користувача.

## 4. Нефункціональні вимоги (Non-Functional Requirements)
- **[NFR-SEC-01] Безпека**: паролі зберігаються виключно у вигляді хешів (`USER.password_hash`).
- **[NFR-PERF-02] Продуктивність**: час відповіді системи  (`APP`, `CATEGORY`) **SHALL** не перевищувати 500 мс під навантаженням до 10 000 паралельних запитів на читання із застосуванням кешування в пам'яті.
- **[NFR-REL-03] Надійність та узгодженість**: оновлення локальних статусів замовлень (`ORDER`) та видача ігор у бібліотеку (`USER_LIBRARY`) виконуються в одній локальній транзакції БД.
- 
## 5. Прецедент використання (UC) та сценарії взаємодії

### `UC-02: Checkout Order` (Оформлення та оплата замовлення)
- **Актори:** `Gamer`, `PaymentGateway` (`<<System>>`)
- **Учасники (компоненти):** `OrderService`, `LibraryService`, `CatalogService`
- **Сутності БД :** `ORDER`, `ORDER_ITEM`, `USER_LIBRARY`, `APP`
- 
#### Сценарій 1 (Основний): Успішна покупка та видача ліцензії
1. `Gamer` відправляє запит `submitOrder(cartId)` до `OrderService`.
2. У циклі для кожного товару в кошику `OrderService` запитує у `LibraryService` перевірку наявності гри в бібліотеці (`validateOwnership(userId, appId)`).
3. `LibraryService` звертається до `USER_LIBRARY` і повертає `notOwned`.
4. `OrderService` створює запис у таблиці `ORDER` зі статусом `Pending` та фіксує позиції в `ORDER_ITEM` із поточною ціною (`price_at_purchase`).
5. `OrderService` передає дані на оплату через `PaymentGateway.processPayment(orderId, amount)` і отримує підтвердження `paymentSuccess(transactionId)`.
6. У межах однієї транзакції `OrderService` оновлює статус замовлення на `Completed` і через `LibraryService.addToLibrary(userId, appId)` додає гру в `USER_LIBRARY`.
7. `Gamer` отримує фінальне підтвердження `orderCompletedConfirmation(orderId)`.

#### Сценарій 2 (Альтернативний): Відхилення через дублювання контенту
1. `Gamer` викликає `submitOrder(cartId)`.
2. Для кожної позиції `OrderService` звертається до `CatalogService.getAppDetails(appId)`, який дістає з таблиці `APP` тип продукту (`app_type`) та посилання на базову гру (`parent_game_id`).
3. Якщо товар виявляється доповненням (`app_type = 'DLC'`), `OrderService` через `LibraryService.validateOwnership(userId, parentGameId)` перевіряє, чи є у користувача основна гра.
4. Далі `LibraryService.validateOwnership(userId, appId)` перевіряє, чи не куплена ця позиція раніше.
5. Якщо гра вже є в бібліотеці або бракує базової гри для DLC, цикл переривається через `break`, а `OrderService` повертає `checkoutRejectedError(appId, reason)`. Замовлення в БД не створюється, запит у банк не йде.

#### Сценарій 3: Банк відхилив платіж (`model_declined.md`)
1. Початкові перевірки бібліотеки та створення `ORDER` у статусі `Pending` проходять так само, як у першому сценарії.
2. На етапі оплати `PaymentGateway.processPayment(orderId, amount)` повертає відмову `paymentFailed(errorCode)`.
3. `OrderService` переводить замовлення у статус `Cancelled` (це знімає блокування на повторну покупку за вимогою `[R  EQ-ST-04]`).
4. `Gamer` отримує повідомлення `paymentDeclinedError(orderId)`, а таблиця `USER_LIBRARY` залишається без змін.

## 6. Матриця трасування (Traceability Matrix)

| Вимога ID | Тип | Прецеденти / Сценарії | Сутності БД (ЛР1) | Зв'язок | Статус |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[REQ-UB-01]** | Функціональна | `UC-01` (Browse Catalog) | `USER`, `USER_LIBRARY`, `APP` | **M:N** | ✅ Covered |
| **[REQ-EV-02]** | Функціональна | `UC-02` (Сценарій 1, 2) | `USER`, `APP`, `USER_LIBRARY` | **M:N** | ✅ Covered |
| **[REQ-EV-03]** | Функціональна | `UC-02` (Сценарій 1, 3) | `ORDER`, `ORDER_ITEM`, `PaymentGateway` | **1:N** | ✅ Covered |
| **[REQ-ST-04]** | Функціональна | `UC-02` (Сценарій 1, 3) | `ORDER`, `ORDER_ITEM` | **1:N** | ✅ Covered |
| **[REQ-CD-05]** | Функціональна | `UC-02` (Сценарій 1) | `ORDER`, `USER_LIBRARY`, `PaymentGateway` | **M:N** | ✅ Covered |
| **[REQ-CD-06]** | Функціональна | `UC-02` (Сценарій 2) | `APP` (`parent_game_id`), `USER_LIBRARY` | **Self-Ref** | ✅ Covered |
| **[NFR-SEC-01]** | Нефункціональна | Загальний контур безпеки | `USER` (`password_hash`) | **1:1** | ✅ Covered |
| **[NFR-PERF-02]** | Нефункціональна | `UC-01` (Browse Catalog) | `APP`, `CATEGORY` | **M:N** | ✅ Covered |
| **[NFR-REL-03]** | Нефункціональна | `UC-02` (Сценарій 1) | `ORDER`, `ORDER_ITEM`, `USER_LIBRARY` | **1:N** | ✅ Covered |

## 7. Критерії прийняття (Acceptance Criteria)
- [AC-1] Вимоги сформульовано за синтаксисом EARS.
- [AC-2] Забезпечено трасування між вимогами, сценаріями та сутностями ЛР1.
- [AC-3] Описано нефункціональні вимоги та їхній вплив на сутності й транзакції.
- [AC-4] Кожен із трьох сценаріїв `UC-02` має власну діаграму (`model_main.md`, `model_alt.md`, `model_declined.md`).
- [AC-5] Смуги активації (`activation bars`): головний координатор `OrderService` спрацьовує при отриманні `submitOrder` і деактивується лише після завершення всього сценарію.
- [AC-6] Усі види методів (`submitOrder`, `validateOwnership`, `getAppDetails`, `processPayment` тощо).
- [AC-7] Використано фрагменти (`loop`, `opt`, `break`, `rect`) відповідно до логіки сценарію.
- [AC-8] Взаємодія з базою даних показана через явні звернення до сутностей `ORDER`, `ORDER_ITEM`, `USER_LIBRARY` та `APP`, зі створенням статусу `Pending` до виклику платіжного шлюзу.
- [AC-9] Перевірка DLC через `CatalogService.getAppDetails(appId)` і `parent_game_id` присутня в усіх трьох сценаріях.
