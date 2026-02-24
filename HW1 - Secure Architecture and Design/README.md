# Assignment 1 - Security Architecture and Design
## Internal Enterprise Application

# Task 1: System Definition and Architecture
## Internal Enterprise Application

### Application Components

| Component | Type | Description |
|-----------|------|-------------|
| Employee Portal | Frontend | Web interface employees use to access the system. Handles user sessions and serves UI. |
| Internal APIs | Backend Services | REST APIs that process all business logic, enforce rules, and coordinate data flow. |
| Application Database | Data Storage | Stores employee records, business data, and transactions. |
| Role Database | Data Storage | Stores permissions, roles, and access control matrices. |
| Corporate IDP | External Integration | Central identity provider (Active Directory/Okta) that handles authentication. |
| Audit Logs | Data Storage | Immutable storage for all user actions and system events. |
| Admin Portal | Admin Access | Privileged interface for administrators to manage users, roles, and system config. |

### Users and Roles

| User Type | Role | Access Level | Description |
|-----------|------|--------------|-------------|
| Employee | Regular staff | Basic | Can access portal, view own data, submit requests |
| Manager | Team supervisor | Elevated | Can view team data, approve requests |
| Administrator | IT/System admin | Full | Can manage users, roles, and system configuration |
| Auditor | Compliance | Read-only | Can view logs and access reports |
| Service Account | Automated processes | API-only | Used by batch jobs and system integrations |

### Data Types Handled

| Data Type | Examples | Classification |
|-----------|----------|----------------|
| PII | Name, Email, Employee ID, Address | Sensitive |
| Authentication Data | Password hashes, MFA tokens | Critical |
| Role Data | Permissions, group memberships | Internal |
| Business Records | Reports, documents, transactions | Internal |
| Audit Logs | User actions, timestamps, IPs | Sensitive |
| Financial Data | Salary, bonuses | Highly Sensitive |

### External Dependencies

| Dependency | Purpose | Criticality |
|------------|---------|-------------|
| Corporate Identity Provider | Employee authentication | HIGH |
| Email System | Notifications and alerts | MEDIUM |
| DNS Services | Name resolution | MEDIUM |
| Backup Infrastructure | Data protection | HIGH |

### Trust Boundaries

| Boundary | Name | Contains | Trust Level |
|----------|------|----------|-------------|
| B1 | Internet Zone | Employee only | Untrusted |
| B2 | DMZ/Entry Zone | Employee Portal | Semi-trusted |
| B3 | Internal Network | APIs, IDP, Admin Portal, Databases, Logs | Trusted |
| B4 | Privileged Zone | Admin Portal | Highly Restricted |
| B5 | Data Zone | Application DB, Role Database | Most Restricted |

### Architecture Diagram

![Architecture Diagram](enterprise%20architecture.png)

*Diagram shows all components, data flows (HTTPS, REST, SQL), and trust boundaries with clear zone separation.*

## Task 2: Asset Identification and Security Objectives

### Asset Inventory Table

| Asset ID | Asset Name | Description | Location | Classification |
|----------|------------|-------------|----------|----------------|
| A001 | Employee Credentials | Usernames, password hashes, MFA tokens | Corporate IDP | Critical |
| A002 | Session Tokens | Active user session identifiers | Employee Portal, APIs | Critical |
| A003 | Employee PII | Names, emails, employee IDs, addresses | Application Database | Sensitive |
| A004 | Role Permissions | RBAC matrices, access rights, group memberships | Role Database | Internal |
| A005 | Business Data | Internal documents, reports, transactions | Application Database | Internal |
| A006 | Audit Logs | User actions, timestamps, IP addresses | Audit Logs | Sensitive |
| A007 | API Keys | Service-to-service authentication tokens | Internal APIs, Config files | Critical |
| A008 | Financial Data | Salary information, bonuses, expenses | Application Database | Highly Sensitive |
| A009 | Admin Credentials | Privileged account credentials | Corporate IDP | Critical |
| A010 | Business Logic | Application code, API endpoints, workflows | Internal APIs | Internal |
| A011 | Database Backups | All data backups | Backup Storage | Critical |
| A012 | Encryption Keys | Keys used for data encryption | HSM/Key Store | Critical |

### Mapping Assets to Security Objectives

| Asset ID | Asset Name | Confidentiality | Integrity | Availability | Accountability |
|----------|------------|-----------------|-----------|--------------|----------------|
| A001 | Employee Credentials | HIGH | HIGH | MEDIUM | HIGH |
| A002 | Session Tokens | HIGH | HIGH | MEDIUM | MEDIUM |
| A003 | Employee PII | HIGH | MEDIUM | MEDIUM | HIGH |
| A004 | Role Permissions | HIGH | HIGH | HIGH | HIGH |
| A005 | Business Data | MEDIUM | HIGH | MEDIUM | MEDIUM |
| A006 | Audit Logs | HIGH | HIGH | MEDIUM | HIGH |
| A007 | API Keys | HIGH | HIGH | MEDIUM | HIGH |
| A008 | Financial Data | HIGH | HIGH | MEDIUM | HIGH |
| A009 | Admin Credentials | HIGH | HIGH | HIGH | HIGH |
| A010 | Business Logic | MEDIUM | HIGH | LOW | LOW |
| A011 | Database Backups | HIGH | HIGH | MEDIUM | MEDIUM |
| A012 | Encryption Keys | HIGH | HIGH | HIGH | HIGH |

### Security Objectives Definition

| Objective | Definition | Measurement |
|-----------|------------|-------------|
| Confidentiality | Data is accessible only to authorized users | Encryption, access controls |
| Integrity | Data is accurate and hasn't been tampered with | Checksums, validation, signing |
| Availability | Data is accessible when needed | Uptime, redundancy, backups |
| Accountability | Actions can be traced to specific users | Audit logs, non-repudiation |

### Critical Asset Risk Summary

| Risk Level | Asset Count | Examples |
|------------|-------------|----------|
| HIGH | 9 | Credentials, PII, Financial Data, Keys |
| MEDIUM | 2 | Business Data, Backups |
| LOW | 1 | Business Logic |
