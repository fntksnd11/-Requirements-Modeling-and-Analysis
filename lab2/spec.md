## 1. Намір (Intent)
Зафіксувати формалізовані структуровані вимоги до домену Steam у синтаксисі EARS,побудувати Use Case діаграму та забезпечити двостороннє трасування (M:N).

## 2. Актори та цілі
- **Gamer**: покупець і гравець.
- **Verified Publisher**: успадковує `Gamer` (`Verified Publisher IS-A Gamer`).
- **Payment Gateway** (`<<System>>`): зовнішній платіжний сервіс.

## 3. Функціональні вимоги у синтаксисі EARS
- **[REQ-UB-01]**: Система **SHALL** зберігати час гри (`playtime_hours`) у сутності `USER_LIBRARY`.
- **[REQ-EV-02]**: **WHEN** користувач натискає «Додати в кошик», система **SHALL** перевіряти наявність гри у `USER_LIBRARY`.
- **[REQ-EV-03]**: **WHEN** користувач оформлює замовлення (`ORDER`), система **SHALL** ініціювати платіж до `Payment Gateway`.
- **[REQ-ST-04]**: **WHILE** статус замовлення є `Pending`, система **SHALL** блокувати повторну покупку.
- **[REQ-CD-05]**: **IF** платіж підтверджено, **THEN** система **SHALL** змінити статус замовлення на `Completed` та оновити `USER_LIBRARY`.
- **[REQ-CD-06]**: **IF** гра має `app_type = 'DLC'`, **THEN** система **SHALL** перевіряти наявність `parent_game_id`. *(Непокрита вимога)*.

## 4. Нефункціональні вимоги (Non-Functional Requirements)
- **[NFR-SEC-01] Безпека (Security)**: Система **SHALL** зберігати паролі користувачів виключно у вигляді криптографічних хешів у сутності `USER` (`password_hash`), а весь трафік під час транзакцій передавати через HTTPS (TLS 1.3).
- **[NFR-PERF-02] Продуктивність (Performance)**: Час відповіді системи при пошуку або фільтрації в каталозі (`Browse Catalog` / `Filter by Category`) **SHALL** не перевищувати 500 мс при паралельному навантаженні до 10 000 користувачів.
- **[NFR-REL-03] Надійність та цілісність даних (Reliability)**: Зміна статусів замовлень та оновлення бібліотеки (`ORDER` + `USER_LIBRARY`) **SHALL** виконуватися атомарно в межах однієї транзакції ACID для запобігання втраті даних при збоях мережі*.

| Вимога ID | Тип вимоги | Прецеденти / User Story | Пов'язані сутності бази даних (ЛР1) | Тип зв'язку | Статус покриття |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **[REQ-UB-01]** | Функціональна | `UC-01` (Browse Catalog), <br> `US-01` (Play & Library) | `USER`, `USER_LIBRARY`, `APP` | **M:N** | ✅ **Covered** |
| **[REQ-EV-02]** | Функціональна | `UC-02` (Checkout Order), <br> `US-01` (Purchase Flow) | `USER`, `APP`, `USER_LIBRARY` | **M:N** | ✅ **Covered** |
| **[REQ-EV-03]** | Функціональна | `UC-02` (Checkout Order), <br> `UC-Payment` (Process Payment) | `ORDER`, `ORDER_ITEM`, `Payment Gateway` | **1:N / API** | ✅ **Covered** |
| **[REQ-ST-04]** | Функціональна | `UC-02` (Checkout Order) | `ORDER`, `ORDER_ITEM` | **1:N** | ✅ **Covered** |
| **[REQ-CD-05]** | Функціональна | `UC-02` (Checkout Order), <br> `US-01` (Purchase Flow) | `ORDER`, `USER_LIBRARY`, `Payment Gateway` | **M:N** | ✅ **Covered** |
| **[REQ-CD-06]** | Функціональна | *Відсутній (Planned for Sprint 2)* | `APP` (`parent_game_id`) | **Self-Ref** | ⚠️ **UNCOVERED** *(Технічний борг)* |
| **[NFR-SEC-01]** | Нефункціональна | Загальний контур безпеки (Auth & Payment) | `USER` (`password_hash`) | **1:1** | ✅ **Covered** |
| **[NFR-PERF-02]** | Нефункціональна | `UC-01` (Browse Catalog), <br> `UC-Filter` (Filter by Category) | `APP`, `CATEGORY` | **M:N** | ✅ **Covered** |
| **[NFR-REL-03]** | Нефункціональна | `UC-02` (Checkout Order), <br> `US-01` (Purchase Flow) | `ORDER`, `ORDER_ITEM`, `USER_LIBRARY` | **1:N** | ✅ **Covered** |


## 4. User Stories та Gherkin
**US-01: Купівля гри**
```gherkin
Scenario: Успішна купівля гри
  Given користувач авторизований та обрав гру
  When користувач підтверджує замовлення
  Then система змінює статус замовлення на Completed та оновлює USER_LIBRARY






