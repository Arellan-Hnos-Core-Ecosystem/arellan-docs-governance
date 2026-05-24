# Global Software Coding Standards

## 1. Backend Standards (NestJS)
- Strict TypeScript compilation modes are mandatory.
- Use explicit data transfer objects (DTOs) for payload mutation validations.
- Controllers must focus on data routing; all core business rules must exist in decoupled services.

## 2. Frontend Standards (Next.js / React)
- Functional hooks and component segmentation are required.
- Maintain layout isolation utilizing shared design tokens via `arellan-design-system`.