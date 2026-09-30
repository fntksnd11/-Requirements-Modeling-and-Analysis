Специфікація концептуальної ER-моделі: Steam 

1. Намір:
Розробити ER-модель для платформи Steam.

2. Сутності та атрибути:
- **USER**: `id` (BIGINT, PK), `username` (String), `email` (String).
- **PUBLISHER**: `id` (INT, PK), `publisher_name` (String).
- **APP**: `id` (INT, PK), `publisher_id` (INT, FK), `title` (String), `price` (Decimal).
- **CATEGORY**: `id` (INT, PK), `category_name` (String).

3. Зв'язки:
- `PUBLISHER` ── `APP`: **1:N** (*publishes*)
- `APP` ── `CATEGORY`: **M:N** (*belongs_to*, чистий M:N)
- `USER` ── `APP`: **M:N** (*wishes*, Wishlist, чистий M:N)

4. Критерії прийняття:
**Чисті M:N**:ШІшка додала окремі атрибути (`App`↔`Category`, `Wishlist`),видаляємо і сформульовуємо як чисті M:N зв'язки.
