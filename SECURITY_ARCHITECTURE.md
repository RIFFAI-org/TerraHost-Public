# Security Architecture & Flow Documentation

## สถาปัตยกรรมระบบรักษาความปลอดภัย TerraHost

เอกสารนี้อธิบายสถาปัตยกรรมระบบรักษาความปลอดภัยและ flow การทำงานของระบบ TerraHost

---

## System Architecture Overview

### High-Level Architecture
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend       │    │   Database       │
│   (Next.js)     │◄──►│   (Node.js)     │◄──►│   (PostgreSQL)   │
│                 │    │                 │    │                 │
│ - React         │    │ - Express       │    │ - Users         │
│ - TypeScript    │    │ - JWT Auth      │    │ - Sessions      │
│ - 2FA UI        │    │ - 2FA Service   │    │ - Files         │
│ - File Upload   │    │ - File Service  │    │ - Teams         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Nextcloud     │    │   Azure         │    │   Monitoring    │
│   (File Storage)│    │   (Production)  │    │   & Logging     │
│                 │    │                 │    │                 │
│ - GeoTIFF Files │    │ - Azure DB      │    │ - Audit Logs    │
│ - File Sharing  │    │ - Azure Monitor │    │ - Error Tracking│
│ - User Storage  │    │ - Azure CDN     │    │ - Performance   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Security Layers

### Layer 1: Network Security
- **HTTPS/TLS**: All communications encrypted
- **CORS**: Cross-origin resource sharing protection
- **Firewall**: Network-level protection
- **Azure VNet**: Virtual network isolation (Production)

### Layer 2: Application Security
- **JWT Authentication**: Token-based authentication
- **Session Management**: Database-backed sessions
- **2FA Protection**: TOTP + Backup codes
- **API Key Management**: Secure API access

### Layer 3: Data Security
- **Password Hashing**: bcrypt (saltRounds = 12)
- **Data Encryption**: SHA-256 for sensitive data
- **File Encryption**: Nextcloud encryption
- **Database Encryption**: Azure encryption at rest

### Layer 4: Access Control
- **Role-Based Access**: admin, user, viewer roles
- **File Permissions**: Granular file access control
- **Team Management**: Team-based access control
- **API Permissions**: Permission-based API access

---

## Authentication Flow

### User Registration Flow
```
1. User submits registration form
2. Frontend validates input
3. Backend validates data
4. Password hashed with bcrypt (saltRounds = 12)
5. User created with is_active = false
6. Email verification sent (if implemented)
7. Admin approval required
```

### User Login Flow
```
1. User submits credentials
2. Backend validates email/username
3. Password verification with bcrypt
4. Check user status (is_active)
5. Create session in database
6. Generate JWT token with session ID
7. Return token to frontend
8. Frontend stores token securely
```

### 2FA Authentication Flow
```
1. User enables 2FA
2. Generate TOTP secret
3. Generate QR code
4. User scans QR code with authenticator app
5. User enters 6-digit code
6. Verify TOTP code
7. Generate backup codes (8 codes)
8. Store hashed backup codes
9. Enable 2FA for user
```

---

## File Security Flow

### File Upload Security
```
1. User selects GeoTIFF file
2. Frontend validates file type (.tif/.tiff)
3. File sent to backend with JWT token
4. Backend verifies JWT and session
5. Validate file format and size
6. Calculate file checksum
7. Upload to Nextcloud with encryption
8. Save file metadata to database
9. Set file permissions
10. Return success response
```

### File Access Control
```
1. User requests file access
2. Backend verifies JWT token
3. Check user permissions for file
4. Check team access if applicable
5. Generate secure download URL
6. Log access attempt
7. Return file or access denied
```

---

## Security Middleware Stack

### Request Processing Pipeline
```
1. CORS Middleware
2. Rate Limiting (implemented for API keys)
3. Request Validation
4. JWT Token Verification
5. Session Validation
6. 2FA Verification (if required)
7. Permission Check
8. API Key Validation (for API endpoints)
9. Request Processing
10. Response Logging
```

### Error Handling
```
1. Catch all errors
2. Log security-relevant errors
3. Sanitize error messages
4. Return appropriate HTTP status
5. Audit log entry
```

---

## Monitoring & Logging

### Security Events Logged
- **Authentication Events**: Login, logout, failed attempts
- **2FA Events**: Setup, verification, backup code usage
- **File Operations**: Upload, download, delete, share
- **API Usage**: API key usage, permission checks
- **System Events**: Errors, security violations
- **Session Events**: Creation, expiration, invalidation

### Log Storage
- **Development**: Console logs + Coolify monitoring
- **Production**: Azure Monitor + Application Insights
- **Database**: Audit tables for critical events
- **File System**: Log files for detailed debugging

---

## Security Configuration

### Environment Variables
```bash
# JWT Configuration
JWT_SECRET=your-super-secret-jwt-key
JWT_EXPIRES_IN=7d

# Database Security
DB_HOST=your-db-host
DB_PASSWORD=your-db-password

# File Storage
NEXTCLOUD_URL=your-nextcloud-url
NEXTCLOUD_USERNAME=your-username
NEXTCLOUD_PASSWORD=your-password

# Session Security
SESSION_SECRET=your-session-secret
```

### Security Headers (to be implemented)
```javascript
// Security headers to add (helmet package installed but not implemented)
// Current: Basic CORS protection
// Future: Add helmet for comprehensive security headers
// Note: helmet package is in package.json but not used in app.js
```

### Current Security Implementation
- **CORS**: Basic cross-origin protection
- **Session Security**: Secure session configuration
- **JWT Security**: Token-based authentication
- **Rate Limiting**: API key rate limiting (100 req/15min)

---

## Incident Response

### Security Incident Detection
1. **Automated Monitoring**: System health checks (basic implementation)
2. **Log Analysis**: Manual log analysis (no automated pattern detection)
3. **User Reports**: Security issue reporting
4. **External Alerts**: Third-party security services (planned)

### Response Procedures
1. **Detection**: Identify security incident
2. **Assessment**: Evaluate severity and impact
3. **Containment**: Isolate affected systems
4. **Investigation**: Analyze root cause
5. **Recovery**: Restore normal operations
6. **Documentation**: Record incident details
7. **Prevention**: Implement preventive measures

---

## Performance & Scalability

### Security Performance
- **JWT Verification**: ~1ms per request
- **Password Hashing**: ~100ms (bcrypt saltRounds=12)
- **File Encryption**: ~10ms per MB
- **Database Queries**: ~5ms average

### Scalability Considerations
- **Session Storage**: Database-backed for scalability
- **File Storage**: Nextcloud clustering support
- **Database**: Connection pooling (2-10 connections)
- **Caching**: Redis for session caching (planned, not implemented)

---

## Continuous Security

### Security Updates
- **Dependencies**: Regular security updates (manual)
- **Code Reviews**: Security-focused code reviews (manual)
- **Penetration Testing**: Regular security assessments (planned)
- **Vulnerability Scanning**: Automated security scans (planned)

### Compliance Monitoring
- **Audit Logs**: Comprehensive audit trail
- **Access Reviews**: Regular access permission reviews
- **Security Training**: Team security awareness
- **Incident Drills**: Regular security incident simulations

---

*เอกสารนี้ได้รับการอัปเดตล่าสุด: ${new Date().toLocaleDateString('th-TH')}*
*เวอร์ชัน: 1.0*
*สถานะ: ใช้งานจริง*
