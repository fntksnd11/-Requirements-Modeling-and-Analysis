# Концептуальна ER-діаграма: Steam Digital Store

```mermaid
erDiagram
    USER {
        string id PK
        string username
        string email
    }

    PUBLISHER {
        string id PK
        string publisher_name
    }

    APP {
        string id PK
        string publisher_id FK
        string title
        number price
    }

    CATEGORY {
        string id PK
        string category_name
    }

    PUBLISHER ||--o{ APP : "publishes"
    APP }o--o{ CATEGORY : "belongs_to"
    USER }o--o{ APP : "wishes"
```
