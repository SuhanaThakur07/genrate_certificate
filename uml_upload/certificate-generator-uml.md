# Certificate Generator System: UML Diagrams

Project: Certificate Generator and College Club/Event Management System
Stack: Django, Django REST Framework, JWT, Cloudinary, SMTP (Gmail)

> Note: Class names and attributes below are modelled from the project description.
> Rename them to match the actual `models.py` before the viva.

---

## 1. Use Case Diagram

```mermaid
flowchart LR
    Admin([Admin])
    Org([Club Organizer])
    Part([Participant])

    subgraph SYS[Certificate Generator System]
        UC1((Login with JWT))
        UC2((Register Club))
        UC3((Register Event))
        UC4((Upload CSV, Template and Signature))
        UC5((Generate Certificates in Bulk))
        UC6((Send Certificate by Email))
        UC7((View Event History))
        UC8((Generate PDF Report))
        UC9((Manage Users and Data))
        UC10((Verify Certificate by ID))
    end

    Org --- UC1
    Org --- UC2
    Org --- UC3
    Org --- UC4
    Org --- UC5
    Org --- UC7
    Org --- UC8
    Admin --- UC1
    Admin --- UC9
    Admin --- UC7
    Part --- UC10
    UC5 -. include .-> UC6
    UC4 -. include .-> UC5
```

---

## 2. Class Diagram

```mermaid
classDiagram
    class User {
        +int id
        +string username
        +string email
        +string password
        +string role
        +login()
        +refreshToken()
    }
    class Club {
        +int id
        +string name
        +string description
        +registerClub()
        +getEvents()
    }
    class Event {
        +int id
        +string title
        +date eventDate
        +string details
        +registerEvent()
        +getDetails()
    }
    class CertificateTemplate {
        +int id
        +string imageUrl
        +int nameX
        +int nameY
        +string signatureUrl
    }
    class Participant {
        +int id
        +string name
        +string email
    }
    class Certificate {
        +string uid
        +string cloudinaryUrl
        +string qrCode
        +date issuedOn
        +generate()
        +verify()
    }
    class EmailService {
        +string smtpServer
        +int port
        +sendBulkEmails()
    }
    class CSVParser {
        +parse(file)
        +validate()
    }

    User "1" --> "*" Club : manages
    Club "1" --> "*" Event : hosts
    Event "1" --> "*" Participant : has
    Event "1" --> "1" CertificateTemplate : uses
    Participant "1" --> "1" Certificate : receives
    CertificateTemplate "1" --> "*" Certificate : generates
    CSVParser ..> Participant : creates
    Certificate ..> EmailService : sent by
```

---

## 3. Sequence Diagram (Bulk Certificate Generation)

```mermaid
sequenceDiagram
    actor Organizer
    participant API as Django REST API
    participant Auth as JWT Auth
    participant CSV as CSV Parser
    participant Gen as Certificate Engine
    participant Cloud as Cloudinary
    participant Mail as SMTP Email Service
    actor Participant

    Organizer->>API: POST /generate (CSV, template, signature) + JWT
    API->>Auth: Verify token
    Auth-->>API: Valid
    API->>CSV: Parse participant rows
    CSV-->>API: List of participants
    loop For each participant
        API->>Gen: Place name, QR and signature at coordinates
        Gen-->>API: Certificate image (in memory)
        API->>Cloud: Upload certificate
        Cloud-->>API: Secure URL
        API->>Mail: Send HTML email + certificate
        Mail-->>Participant: Certificate with ID and verify link
    end
    API-->>Organizer: Success response
```

---

## 4. Activity Diagram

```mermaid
flowchart TD
    A([Start]) --> B[Organizer logs in]
    B --> C{Valid JWT?}
    C -- No --> B
    C -- Yes --> D[Upload CSV, template and signature]
    D --> E{CSV valid?}
    E -- No --> F[Show error message] --> D
    E -- Yes --> G[Read next participant]
    G --> H[Generate certificate and QR code]
    H --> I[Upload to Cloudinary]
    I --> J[Save record in database]
    J --> K[Send email with certificate]
    K --> L{More participants?}
    L -- Yes --> G
    L -- No --> M[Show success summary]
    M --> N([End])
```

---

## 5. Component Diagram

```mermaid
flowchart LR
    subgraph Client
        UI[Web Frontend]
    end

    subgraph Backend[Django Backend]
        AUTH[Auth Module - JWT]
        EVT[Club and Event Module]
        CSVM[CSV Parsing Module]
        CERT[Certificate Engine]
        PDF[PDF Report Module]
        MAIL[Email Module - smtplib]
        ADM[Admin Panel]
    end

    DB[(Database)]
    CLD[(Cloudinary Storage)]
    SMTP[Gmail SMTP Server]

    UI -->|REST APIs| AUTH
    UI -->|REST APIs| EVT
    UI -->|REST APIs| CSVM
    CSVM --> CERT
    CERT --> CLD
    CERT --> MAIL
    MAIL -->|TLS port 587| SMTP
    EVT --> DB
    CERT --> DB
    AUTH --> DB
    EVT --> PDF
    ADM --> DB
```
