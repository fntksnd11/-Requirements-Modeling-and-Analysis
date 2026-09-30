Специфікація концептуальної ER-моделі: Steam 

1. Намір:
Розробити ER-модель для платформи Steam.

2. Сутності та атрибути:
- **USER**: `id` (BIGINT, PK), `username` (String), `email` (String).
- **PUBLISHER**: `id` (INT, PK), `publisher_name` (String).
- **APP**: `id` (INT, PK), `publisher_id` (INT, FK), `title` (String), `price` (Decimal).
- **CATEGORY**: `id` (INT, PK), `category_name` (String).
- **APP_CATEGORY**: `app_id` (INT, FK), `category_id` (INT, FK).
- **WISHLIST**: `user_id` (BIGINT, FK), `app_id` (INT, FK), `added_at` (DateTime).

3. Зв'язки:
- `PUBLISHER` ── `APP`: **1:N**
- `APP` ── `APP_CATEGORY` ── `CATEGORY`: **1:N** та **N:1**
- `USER` ── `WISHLIST` ── `APP`: **1:N** та **N:1**

4. Критерії прийняття:
Модель повинна містити основні сутності користувачів, ігор та категорій.
