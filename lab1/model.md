```mermaid
erDiagram
    USER {
        string id PK
        string username
        string email
        string password_hash
        number wallet_balance
        string created_at
    }

    PUBLISHER {
        string id PK
        string publisher_name
        string website
        string support_email
    }

    APP {
        string id PK
        string publisher_id FK
        string parent_game_id FK
        string title
        string app_type
        number price
        string release_date
    }

    CATEGORY {
        string id PK
        string category_name
        string description
    }

    ORDER {
        string id PK
        string user_id FK
        string order_date
        number total_amount
        string status
    }

    ORDER_ITEM {
        string id PK
        string order_id FK
        string app_id FK
        number price_at_purchase
    }

    USER_LIBRARY {
        string id PK
        string user_id FK
        string app_id FK
        number playtime_hours
        string added_date
    }

    REVIEW {
        string id PK
        string user_id FK
        string app_id FK
        boolean is_recommended
        string content
    }

    PUBLISHER ||--o{ APP : "publishes"
    APP ||--o{ APP : "has_dlc"
    APP }o--o{ CATEGORY : "belongs_to"
    USER }o--o{ APP : "wishes"
    USER ||--o{ ORDER : "places"
    ORDER ||--o{ ORDER_ITEM : "contains"
    APP ||--o{ ORDER_ITEM : "included_in"
    USER ||--o{ USER_LIBRARY : "owns"
    APP ||--o{ USER_LIBRARY : "in_library"
    USER ||--o{ REVIEW : "writes"
    APP ||--o{ REVIEW : "reviewed_in"
```
