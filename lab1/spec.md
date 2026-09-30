Специфікація концептуальної ER-моделі: Steam 

1. Намір:
Розробити ER-модель для платформи Steam.

2. Сутності та атрибути:
- **USER**: `id` (UUID, PK), `username` (String), `email` (String).
- **PUBLISHER**: `id` (UUID, PK), `publisher_name` (String).
- **APP**: `id` (UUID, PK), `publisher_id` (UUID, FK), `title` (String), `price` (Decimal).
- **CATEGORY**: `id` (UUID, PK), `category_name` (String).
- **ORDER**: `id` (UUID, PK), `user_id` (UUID, FK), `order_date` (DateTime), `total_amount` (Decimal), `status` (String).
- **ORDER_ITEM** *(Асоціативна)*: `id` (UUID, PK), `order_id` (UUID, FK), `app_id` (UUID, FK), `price_at_purchase` (Decimal).
- **USER_LIBRARY** *(Асоціативна)*: `id` (UUID, PK), `user_id` (UUID, FK), `app_id` (UUID, FK), `playtime_hours` (Integer), `added_date` (DateTime).
- **REVIEW** *(Асоціативна)*: `id` (UUID, PK), `user_id` (UUID, FK), `app_id` (UUID, FK), `is_recommended` (Boolean), `content` (String).
- 
3. Зв'язки:
- `PUBLISHER` ── `APP`: **1:N** (*publishes*)
- `APP` ── `APP`: **1:N** (*has_dlc*, self-reference)
- `APP` ── `CATEGORY`: **M:N** (*belongs_to*, чистий M:N)
- `USER` ── `APP`: **M:N** (*wishes*, Wishlist, чистий M:N)
- `USER` ── `ORDER`: **1:N** (*places*)
- `ORDER` ── `ORDER_ITEM` ── `APP`: **1:N** та **N:1** 
- `USER` ── `USER_LIBRARY` ── `APP`: **1:N** та **N:1** 
- `USER` ── `REVIEW` ── `APP`: **1:N** та **N:1** 

4. Критерії прийняття:
**Чисті M:N**:ШІшка додала окремі атрибути (`App`↔`Category`, `Wishlist`),видаляємо і сформульовуємо як чисті M:N зв'язки.
**Типізація ID**: Усі Primary Keys мають єдиний тип `UUID`.
**Обґрунтованість сутностей**: Проміжні сутності (`Order_Item`, `User_Library`, `Review`) створюються тільки за наявності власних атрибутів,щоб ункнути помилок під час дій у магазині.
**100% Узгодженість**: Підправляємо назви всіх атрибутів і сутностей у `spec.md` та `model.md` ідентичні.
