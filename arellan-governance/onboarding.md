# Engineering Onboarding Guide

## 1. Initial Local Setup
1. Clone the central operations management repo: `arellan-infra-devops`.
2. Execute `docker compose up -d` to mount the base PostgreSQL, Redis, and SonarQube instances locally.
3. Clone `arellan-backend-core` and run `npm run start:dev`.

## 2. Security Checks
Verify that your global Git configuration utilizes correct SSH keys matching your verified profile inside the closed GitHub organization tier.