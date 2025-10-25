# Security Flow Diagrams

## ไดอะแกรม Flow ระบบรักษาความปลอดภัย TerraHost

---

## Authentication Flow Diagram

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend
    participant D as Database
    participant N as Nextcloud

    Note over U,N: User Login Flow
    U->>F: Enter credentials
    F->>B: POST /auth/login
    B->>D: Validate user
    D-->>B: User data
    B->>B: Verify password (bcrypt)
    B->>D: Create session
    D-->>B: Session ID
    B->>B: Generate JWT token
    B-->>F: JWT token + user info
    F->>F: Store token securely
    F-->>U: Login successful

    Note over U,N: 2FA Verification (if enabled)
    U->>F: Enter 6-digit code
    F->>B: POST /2fa/verify
    B->>B: Verify TOTP code
    B->>D: Update 2FA status
    B-->>F: 2FA verified
    F-->>U: Access granted
```

---

## File Upload Security Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend
    participant D as Database
    participant N as Nextcloud

    Note over U,N: Secure File Upload
    U->>F: Select GeoTIFF file
    F->>F: Validate file type (.tif/.tiff)
    F->>B: POST /files/upload (with JWT)
    B->>B: Verify JWT token
    B->>D: Validate session
    B->>B: Validate file format
    B->>B: Calculate checksum
    B->>N: Upload encrypted file
    N-->>B: Upload success
    B->>D: Save file metadata
    B->>D: Set file permissions
    B-->>F: Upload successful
    F-->>U: File uploaded
```

---

## API Key Authentication Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant B as Backend
    participant D as Database

    Note over C,D: API Key Authentication
    C->>B: API request with API key
    B->>B: Extract API key from header
    B->>D: Validate API key (SHA-256 hash)
    D-->>B: API key data + permissions
    B->>B: Check permissions
    alt Valid API key
        B->>B: Process request
        B-->>C: API response
    else Invalid API key
        B-->>C: 401 Unauthorized
    end
```

---

## Security Middleware Stack

```mermaid
graph TD
    A[Incoming Request] --> B[CORS Middleware]
    B --> C[Rate Limiting]
    C --> D[Request Validation]
    D --> E[JWT Token Verification]
    E --> F[Session Validation]
    F --> G[2FA Check]
    G --> H[Permission Check]
    H --> I[API Key Validation]
    I --> J[Process Request]
    J --> K[Response Logging]
    K --> L[Return Response]
    
    E -->|Invalid Token| M[401 Unauthorized]
    F -->|Invalid Session| N[401 Unauthorized]
    G -->|2FA Required| O[2FA Challenge]
    H -->|No Permission| P[403 Forbidden]
    I -->|Invalid API Key| Q[401 Unauthorized]
```

---

## 2FA Setup Flow

```mermaid
flowchart TD
    A[User enables 2FA] --> B[Generate TOTP Secret]
    B --> C[Generate QR Code]
    C --> D[Display QR Code to User]
    D --> E[User scans with Authenticator App]
    E --> F[User enters 6-digit code]
    F --> G[Verify TOTP code]
    G -->|Valid| H[Generate 8 backup codes]
    G -->|Invalid| I[Show error message]
    I --> F
    H --> J[Hash backup codes with SHA-256]
    J --> K[Store in database]
    K --> L[Display backup codes to user]
    L --> M[2FA enabled successfully]
```

---

## System Architecture Overview

```mermaid
graph TB
    subgraph "Frontend Layer"
        F1[Next.js Frontend]
        F2[React Components]
        F3[2FA UI]
        F4[File Upload UI]
    end
    
    subgraph "Backend Layer"
        B1[Express.js Server]
        B2[JWT Authentication]
        B3[2FA Service]
        B4[File Service]
        B5[API Key Management]
    end
    
    subgraph "Database Layer"
        D1[PostgreSQL]
        D2[Users Table]
        D3[Sessions Table]
        D4[Files Table]
        D5[Teams Table]
    end
    
    subgraph "Storage Layer"
        S1[Nextcloud]
        S2[GeoTIFF Files]
        S3[File Encryption]
    end
    
    subgraph "Infrastructure"
        I1[Development: Coolify]
        I2[Production: Azure]
        I3[Azure Database]
        I4[Azure Monitor]
    end
    
    F1 --> B1
    B1 --> D1
    B1 --> S1
    B1 --> I1
    B1 --> I2
```

---

## Password Security Flow

```mermaid
sequenceDiagram
    participant U as User
    participant F as Frontend
    participant B as Backend
    participant D as Database

    Note over U,D: Password Registration
    U->>F: Enter password
    F->>B: POST /auth/register
    B->>B: Validate password strength
    B->>B: Hash password (bcrypt saltRounds=12)
    B->>D: Store hashed password
    D-->>B: User created
    B-->>F: Registration successful

    Note over U,D: Password Login
    U->>F: Enter password
    F->>B: POST /auth/login
    B->>D: Get user data
    D-->>B: Hashed password
    B->>B: Compare password (bcrypt)
    B-->>F: Login result
```

---

## File Access Control Flow

```mermaid
flowchart TD
    A[User requests file] --> B[Verify JWT token]
    B --> C[Check user permissions]
    C --> D{Has file access?}
    D -->|Yes| E[Check team access]
    D -->|No| F[Access denied]
    E --> G{Team member?}
    G -->|Yes| H[Generate download URL]
    G -->|No| I[Check individual permissions]
    I --> J{Has individual access?}
    J -->|Yes| H
    J -->|No| F
    H --> K[Log access attempt]
    K --> L[Return file]
```

---

## Security Incident Response

```mermaid
flowchart TD
    A[Security Incident Detected] --> B[Assess Severity]
    B --> C{High Severity?}
    C -->|Yes| D[Immediate Response]
    C -->|No| E[Standard Response]
    D --> F[Isolate Affected Systems]
    E --> G[Log Incident]
    F --> H[Notify Security Team]
    G --> I[Investigate Root Cause]
    H --> I
    I --> J[Implement Fix]
    J --> K[Test Solution]
    K --> L[Restore Services]
    L --> M[Document Incident]
    M --> N[Update Security Measures]
```

---

## Performance Monitoring

```mermaid
graph LR
    A[Application] --> B[Health Checks]
    A --> C[Error Tracking]
    A --> D[Performance Metrics]
    
    B --> E[Azure Monitor]
    C --> E
    D --> E
    
    E --> F[Alerts]
    E --> G[Dashboards]
    E --> H[Logs]
    
    F --> I[Security Team]
    G --> J[Operations Team]
    H --> K[Audit Trail]
```

---

*เอกสารนี้ได้รับการอัปเดตล่าสุด: ${new Date().toLocaleDateString('th-TH')}*
*เวอร์ชัน: 1.0*
*สถานะ: ใช้งานจริง*
