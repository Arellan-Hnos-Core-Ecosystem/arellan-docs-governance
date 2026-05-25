# Checklist de Pull Request

Lista de verificación obligatoria antes de aprobar cualquier PR hacia `develop` o `main`. El reviewer debe confirmar cada ítem antes de dar `APPROVED`.

## Checklist del Autor (Antes de Abrir el PR)

```
CÓDIGO
- [ ] TypeScript strict: sin errores de compilación (tsc --noEmit)
- [ ] ESLint: sin errores (npm run lint)
- [ ] Prettier: código formateado (npm run format:check)
- [ ] Sin console.log ni código de debug
- [ ] Sin credenciales, tokens ni API keys hardcodeadas
- [ ] Sin `any` en TypeScript (sin excepción)

TESTS
- [ ] Tests unitarios escritos para la nueva lógica
- [ ] Tests de integración si hay nuevos endpoints
- [ ] Cobertura total >= 80% (npm run test:coverage)
- [ ] Cobertura en módulos finance y auth >= 90%
- [ ] Todos los tests pasan (npm test)

SEGURIDAD
- [ ] Todos los nuevos endpoints tienen guards (@UseGuards)
- [ ] Roles correctos en @Roles() para cada endpoint
- [ ] Si el endpoint es público: anotado con @Public() y justificado
- [ ] Validación con class-validator en todos los DTOs nuevos
- [ ] Sin SQL raw con concatenación de strings (usar tagged templates)

BASE DE DATOS
- [ ] Migraciones son backwards-compatible (no rompen el schema existente)
- [ ] Nuevas tablas tienen índices en campos de búsqueda frecuente
- [ ] audit_logs no tiene endpoints de DELETE ni UPDATE
- [ ] Campos sensibles cifrados con AES-256 si aplica

DOCUMENTACIÓN
- [ ] Descripción del PR explica qué cambia y por qué
- [ ] Cambios de API documentados (si aplica)
- [ ] Breaking changes señalados explícitamente
```

## Checklist del Reviewer

```
COMPRENSIÓN
- [ ] Entiendo qué hace este cambio y por qué
- [ ] El cambio está alineado con la arquitectura del proyecto
- [ ] No hay efectos secundarios no documentados

CÓDIGO
- [ ] La lógica es correcta y no tiene bugs obvios
- [ ] No hay duplicación de lógica que debería estar en un service
- [ ] El manejo de errores es apropiado (no catch vacíos)
- [ ] Los nombres de variables/funciones son descriptivos

SEGURIDAD
- [ ] No hay nuevas superficies de ataque introducidas
- [ ] No hay datos sensibles en logs
- [ ] Los guards de autorización están correctamente configurados
- [ ] Si hay cambios en el flujo de pago: revisión extra cuidadosa

TESTS
- [ ] Los tests cubren el caso principal (happy path)
- [ ] Los tests cubren casos edge importantes
- [ ] Los tests son legibles y tienen nombres descriptivos
- [ ] La cobertura no bajó del mínimo

BASE DE DATOS
- [ ] Las migraciones se pueden aplicar y revertir limpiamente
- [ ] No hay queries N+1 (usar include de Prisma apropiadamente)
- [ ] Índices adecuados para las nuevas queries
```

## Proceso de Aprobación

```
1. Reviewer lee el PR completo (diff + descripción)
2. Reviewer corre los tests localmente si hay dudas
3. Reviewer deja comentarios en líneas específicas si algo no queda claro
4. Si todo OK: comentario "APPROVED" + botón de aprobación en GitHub
5. Autor hace merge con --no-ff (no squash, preservar historia)
```

## Tamaño Ideal de PR

- **Ideal:** < 400 líneas de código changed (fácil de revisar en < 1 hora)
- **Máximo:** < 800 líneas (si es mayor, considerar dividir)
- **Excepción:** Migraciones de schema pueden ser grandes pero deben revisarse con cuidado extra

## Etiquetas de PR

| Label | Uso |
|-------|-----|
| `[HOTFIX]` | Parche urgente de producción |
| `[SECURITY]` | Cambio de seguridad / criptografía |
| `[BREAKING]` | Cambio que rompe compatibilidad |
| `[FINANCE]` | Afecta el módulo financiero |
| `[DB]` | Incluye migraciones de base de datos |
| `[WIP]` | Work in progress — no mergear aún |
