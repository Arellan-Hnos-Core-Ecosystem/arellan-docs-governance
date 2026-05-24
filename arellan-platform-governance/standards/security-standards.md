# Enterprise Security Architecture and Data Protection Standards

## 1. Cryptographic Identity & Session State
- **Token Signing Architecture:** All authentication tokens issued by `arellan-auth-service` must utilize asymmetric JSON Web Tokens (JWT) signed with the RS256 algorithm. The private key remains securely injected via environment variables inside the production container, rotated automatically every 90 days via AWS Secrets Manager scripts.
- **Sliding Window Session Policy:**
  - `arellan-frontend-web` (Finance & Administrative): 60-minute token lifespan with an automatic absolute expiration at 180 minutes to prevent session hijacking on office computers.
  - `arellan-mechanic-ui` (Workshop Tablets): 12-hour token persistence, cryptographically pinned to the specific MAC Address and hardware fingerprint of the authorized physical tablet in Surquillo.
- **Multi-Factor Authentication (MFA):** Mandatory for roles: `Owner` and `Finance`. Implementation utilizes Time-based One-Time Password (TOTP) algorithms (RFC 6238).

## 2. API Authorization & Data Masking (Insider Threat Defenses)
- **Role-Based Access Control (RBAC):** Every endpoint exposed through the `arellan-api-gateway` must evaluate the claim array payload. The system strictly isolates database multi-tenant lines:
  - `Role::Mechanic` or `Role::Apprentice` queries targeting work orders must execute through a selective database projection. 
  - **Strict Rule:** Phone numbers, home addresses, customer national identity numbers (DNI/RUC), and vehicle cost margins MUST be completely stripped out (masked) from the JSON payload response at the Gateway layer before hitting the workshop UI.
- **Rate Limiting & Perimetric Firewalls:**
  - External endpoints (`arellan-client-portal`) are restricted to a maximum of 30 requests per minute per IP address utilizing a Redis-backed Token Bucket algorithm to stop brute-force or Denial of Service (DDoS) attempts.