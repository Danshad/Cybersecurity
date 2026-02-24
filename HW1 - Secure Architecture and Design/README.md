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
