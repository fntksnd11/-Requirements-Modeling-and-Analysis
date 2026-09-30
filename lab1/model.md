# Концептуальна ER-діаграма: Steam Digital Store

```mermaid
erDiagram
    USER {
        bigint id PK
        string username
        string email
    }

    PUBLISHER {
        int id PK
        string publisher_name
    }

    APP {
        int id PK
        int publisher_id FK
        string title
        number price
    }

    CATEGORY {
        int id PK
        string category_name
    }

    PUBLISHER ||--o{ APP : "publishes"
    APP }o--o{ CATEGORY : "belongs_to"
    USER }o--o{ APP : "wishes"
```
