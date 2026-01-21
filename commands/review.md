# Comprehensive PR Review

Ejecuta un review completo de pull request usando multiples agentes especializados en paralelo.

**Arguments:** "$ARGUMENTS"

## Instrucciones de Idioma

**IMPORTANTE:**
- El reporte completo debe estar en **ESPAÑOL**
- Para cada hallazgo, incluir un **"PR Comment"** en **INGLES**, casual y breve
- Los PR Comments deben ser directos, como si hablaras con un colega
- **NO usar tablas** - usar listas para presentar los hallazgos

**CRITICO - SOLO REPORTE:**
- Este review es SOLO un reporte para el revisor
- **NUNCA** comentar automaticamente en el PR de GitHub
- **NUNCA** usar `gh pr comment` o `gh pr review`
- **NUNCA** postear comentarios en ninguna plataforma
- Los "PR Comments" son sugerencias de texto que el revisor puede copiar MANUALMENTE si lo desea

Ejemplo de formato para cada hallazgo:
```
1. **[agente]** `archivo.ts:42`
   - **Problema:** Posible inyeccion SQL por concatenacion de strings
   - **PR Comment:** `hey, this query looks vulnerable to SQL injection - mind using parameterized queries instead?`
```

## Flujo del Review

### Paso 1: Identificar el PR

1. Si se proporciona numero o URL en argumentos, usar ese
2. Si no, verificar si el branch actual tiene PR abierto: `gh pr view --json number,title,body,url`
3. Si no hay PR, revisar cambios del branch actual contra main

### Paso 2: Obtener Informacion del PR

```bash
# Detalles del PR
gh pr view <number> --json title,body,files,additions,deletions,baseRefName,headRefName,author

# Obtener el diff
gh pr diff <number>

# Listar archivos cambiados
gh pr diff <number> --name-only
```

### Paso 3: Lanzar Agentes en Paralelo

Lanzar TODOS los siguientes agentes simultaneamente usando la herramienta Task. Cada agente debe recibir:
- El numero de PR o nombre del branch
- La lista de archivos cambiados
- Instrucciones de enfocarse solo en los cambios del PR
- **Instruccion de idioma**: Reporte en español, PR comments en ingles casual
- **Instruccion de formato**: Usar listas, NO tablas

**Agentes a lanzar (TODOS en paralelo):**

1. **pr-description-checker** - Verificar descripcion vs cambios reales
2. **architecture-reviewer** - Evaluar diseño y sugerir alternativas
3. **security-reviewer** - Encontrar vulnerabilidades de seguridad
4. **comment-analyzer** - Verificar precision de comentarios
5. **test-reviewer** - Revisar cobertura de tests
6. **error-handler-reviewer** - Encontrar silent failures
7. **type-reviewer** - Analizar diseño de tipos (si hay archivos TypeScript)
8. **code-reviewer** - Calidad general del codigo
9. **code-simplifier** - Sugerir simplificaciones

### Paso 4: Agregar Resultados

Compilar los hallazgos en un reporte unico:

```markdown
# Resumen del Review

**PR:** #<numero> - <titulo>
**Autor:** <autor>
**Branch:** <head> -> <base>
**Archivos Cambiados:** <cantidad> (+<adiciones>/-<eliminaciones>)

---

## Problemas Criticos (X encontrados)

Deben arreglarse antes de mergear.

1. **[security-reviewer]** `file.ts:42`
   - **Problema:** Vulnerabilidad de inyeccion SQL por concatenacion de strings en query
   - **PR Comment:** `hey, this query looks vulnerable to SQL injection - consider using parameterized queries`

2. **[code-reviewer]** `api.ts:15`
   - **Problema:** Posible null pointer exception si user es undefined
   - **PR Comment:** `heads up - this could throw if user is null, maybe add a check?`

---

## Problemas Importantes (X encontrados)

Deberian arreglarse antes de mergear.

1. **[error-handler-reviewer]** `service.ts:88`
   - **Problema:** Catch block vacio que traga el error sin loggearlo
   - **PR Comment:** `this catch block swallows the error - might want to log or rethrow`

2. **[test-reviewer]** `utils.ts:23`
   - **Problema:** Funcion nueva sin cobertura de tests
   - **PR Comment:** `this new function doesn't have tests - would be good to add some coverage`

---

## Sugerencias (X encontradas)

Mejoras opcionales.

1. **[code-simplifier]** `utils.ts:45`
   - **Sugerencia:** Se puede simplificar usando early return en vez de nesting
   - **PR Comment:** `nit: could simplify this with early return`

2. **[type-reviewer]** `types.ts:12`
   - **Sugerencia:** Considerar usar branded type para el ID
   - **PR Comment:** `optional: a branded type here would catch invalid IDs at compile time`

---

## Precision de la Descripcion del PR

<Resultado de pr-description-checker>

**Puntaje de Precision:** X/10

### Cambios Verificados
- [x] Feature A implementada correctamente
- [x] Bug B arreglado como se describe

### Discrepancias
1. `file.ts:30` - La descripcion dice X pero el codigo hace Y
   - **PR Comment:** `the description mentions X but I see Y in the code - which is correct?`

---

## Cobertura de Tests

<Resumen de test-reviewer>

- **Cobertura estimada:** X%
- **Calidad de tests:** BUENA/ADECUADA/POBRE

### Tests Faltantes Criticos
1. `auth.ts:validateToken()` - Sin tests para caso de token expirado
2. `api.ts:fetchUser()` - Sin tests para error de red

---

## Evaluacion de Seguridad

<Resumen de security-reviewer>

- **Nivel de Riesgo:** CRITICO/ALTO/MEDIO/BAJO/NINGUNO

### Vulnerabilidades por Severidad
- Criticas: X
- Altas: X
- Medias: X
- Bajas: X

---

## Observaciones Positivas

Lo que esta bien hecho en este PR:

1. Buena separacion de concerns en `services/auth.ts`
2. Tests exhaustivos para casos edge en `utils.spec.ts`
3. Manejo de errores apropiado en `api/client.ts`
4. Documentacion clara en las funciones publicas

---

## Plan de Accion

### Antes de Mergear (Requerido)

1. [ ] Arreglar vulnerabilidad SQL en `db.ts:42`
2. [ ] Agregar null check en `api.ts:15`

### Recomendado

3. [ ] Agregar tests para `auth.ts:validateToken()`
4. [ ] Loggear errores en catch block de `service.ts:88`

### Opcional

5. [ ] Simplificar condicionales en `utils.ts:45`
6. [ ] Actualizar descripcion del PR para reflejar cambio en `config.ts`

---

## Metadata del Review

- **pr-description-checker:** X hallazgos
- **architecture-reviewer:** X hallazgos
- **security-reviewer:** X hallazgos
- **comment-analyzer:** X hallazgos
- **test-reviewer:** X hallazgos
- **error-handler-reviewer:** X hallazgos
- **type-reviewer:** X hallazgos
- **code-reviewer:** X hallazgos
- **code-simplifier:** X sugerencias
```

## Ejemplos de Uso

**Revisar PR del branch actual:**
```
/asreview:review
```

**Revisar PR especifico:**
```
/asreview:review 458
/asreview:review https://github.com/owner/repo/pull/458
```

## Notas

- Todos los agentes corren en paralelo para reviews rapidos
- Cada agente se enfoca en su especialidad para analisis profundo
- Los resultados incluyen referencias file:line para navegacion facil
- Los PR Comments son texto sugerido para copiar manualmente
- **NUNCA se postean comentarios automaticamente** - este es solo un reporte
