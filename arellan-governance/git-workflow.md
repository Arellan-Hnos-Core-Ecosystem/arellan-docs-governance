# Multi-Repo Git Flow & Branching Governance Protocol (Plan B)

## 1. Context and Constraints
Due to the constraints of the decentralized GitHub Free organization tier, automated branch protection rule enforcement is inactive for private repositories. Therefore, this protocol acts as a contract of engineering honor between the Core Developers. Any violation breaks code integrity and systemic auditability.

## 2. Branch Topology
Every repository within the 13-tier architecture must maintain the following branch configuration:

- `main`: Absolute production state. Contains fully audited, compiled, and signed release code running at the physical facility.
- `staging`: Quality Assurance and Pre-Production mirroring cloud infrastructure. Used for end-user acceptance testing with the Owners.
- `develop`: Central trunk for feature integration. All developers sync their working branches here.
- `feature/[domain]/[task-codename]`: Short-lived isolated branches for specific tasks. Examples:
  - `feature/finance/dynamic-qr-webhook`
  - `feature/iot/zkteco-log-parser`
- `hotfix/[incident-ticket-id]`: Immediate production patches bypassing staging but back-merged retroactively.

## 3. The Commit and Peer-Review Lifecycle

[feature Branch] ──> Create Pull Request ──> Peer Review (1 Approval) ──> Merge into develop ──> Deploy to Staging


1. **Local Development Boundaries:** No developer is permitted to work locally or commit directly on `main`, `staging`, or `develop`.
2. **Semantic Commit Standards:** Every commit message must parse correctly through standard changelog generators:
    - `feat([domain]):` A new business feature (e.g., `feat(finance): add double-entry ledger validation`).
    - `fix([domain]):` A bug resolution (e.g., `fix(inventory): resolve race condition on stock decrement`).
    - `security([domain]):` Cryptographic or authorization updates (e.g., `security(auth): rotate RS256 token keys`).
    - `chore([domain]):` Devops, config, or package upgrades.
3. **The Mandatory Peer Review Flow:** - Upon feature completion, a Pull Request targeting `develop` must be opened.
    - The author must notify the second engineer with the cryptographic hash of the branch.
    - The reviewer must perform a visual check, verify testing coverage (minimum 80% via Jest/Qodo), and leave an explicit `APPROVED` comment.
    - Merges must be executed utilizing `--no-ff` (No Fast Forward) to preserve the historical merge bubble for architectural forensic analysis.