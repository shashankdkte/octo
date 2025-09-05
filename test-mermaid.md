# Mermaid Test Page

This page tests if Mermaid diagrams render correctly on GitHub Pages.

## Simple Flowchart Test

```mermaid
flowchart TD
    A[Start] --> B{Is it working?}
    B -->|Yes| C[Great!]
    B -->|No| D[Check config]
    D --> A
    C --> E[End]
```

## Sequence Diagram Test

```mermaid
sequenceDiagram
    participant U as User
    participant S as System
    participant D as Database
    
    U->>S: Request data
    S->>D: Query database
    D-->>S: Return results
    S-->>U: Display data
```

## ER Diagram Test

```mermaid
erDiagram
    CUSTOMER ||--o{ ORDER : places
    ORDER ||--|{ LINE-ITEM : contains
    CUSTOMER {
        string name
        string email
    }
    ORDER {
        int orderNumber
        date orderDate
    }
```

If you can see the diagrams above rendered as interactive graphics (not just code blocks), then Mermaid is working correctly!
