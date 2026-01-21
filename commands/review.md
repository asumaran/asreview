# Comprehensive PR Review

Ejecuta un review completo de pull request usando multiples agentes especializados en paralelo.

**Arguments:** "$ARGUMENTS"

## Instrucciones de Idioma

**IMPORTANTE:**
- El reporte completo debe estar en **ESPAÑOL**
- Para cada hallazgo, incluir un **"PR Comment"** en **INGLES**, casual y breve, listo para copiar al PR
- Los PR Comments deben ser directos, como si hablaras con un colega

Ejemplo de formato para cada hallazgo:
```
**Ubicacion:** archivo.ts:42
**Problema:** Posible inyeccion SQL por concatenacion de strings
**PR Comment:** `hey, this query looks vulnerable to SQL injection - mind using parameterized queries instead?`
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

| # | Agente | Problema | Ubicacion | PR Comment |
|---|--------|----------|-----------|------------|
| 1 | security-reviewer | Vulnerabilidad de inyeccion SQL | file.ts:42 | `hey, this query looks vulnerable to SQL injection - consider using parameterized queries` |
| 2 | code-reviewer | Posible null pointer | api.ts:15 | `heads up - this could throw if user is null, maybe add a check?` |

---

## Problemas Importantes (X encontrados)

Deberian arreglarse antes de mergear.

| # | Agente | Problema | Ubicacion | PR Comment |
|---|--------|----------|-----------|------------|
| 1 | error-handler-reviewer | Error silenciado en catch | service.ts:88 | `this catch block swallows the error - might want to log or rethrow` |

---

## Sugerencias (X encontradas)

Mejoras opcionales.

| # | Agente | Sugerencia | Ubicacion | PR Comment |
|---|--------|------------|-----------|------------|
| 1 | code-simplifier | Se puede simplificar condicionales | utils.ts:23 | `nit: could simplify this with early return` |

---

## Precision de la Descripcion del PR

<Resultado de pr-description-checker>

- [ ] Todos los cambios declarados estan implementados
- [ ] No hay cambios sin documentar
- Puntaje de Precision: X/10

---

## Cobertura de Tests

<Resumen de test-reviewer>

- Cobertura estimada: X%
- Tests criticos faltantes: X
- Calidad de tests: BUENA/ADECUADA/POBRE

---

## Evaluacion de Seguridad

<Resumen de security-reviewer>

- Nivel de Riesgo: CRITICO/ALTO/MEDIO/BAJO/NINGUNO
- Vulnerabilidades: X criticas, X altas, X medias, X bajas

---

## Observaciones Positivas

Lo que esta bien hecho en este PR:

- <De varios agentes>
- <Buenos patrones encontrados>
- <Areas bien testeadas>

---

## Plan de Accion

### Antes de Mergear (Requerido)

1. [ ] Arreglar: <Problema critico 1> en file:line
2. [ ] Arreglar: <Problema critico 2> en file:line

### Recomendado

3. [ ] Atender: <Problema importante 1> en file:line

### Opcional

4. [ ] Considerar: <Sugerencia 1> en file:line

---

## Metadata del Review

| Agente | Estado | Hallazgos |
|--------|--------|-----------|
| pr-description-checker | Listo | X problemas |
| architecture-reviewer | Listo | X problemas |
| security-reviewer | Listo | X problemas |
| comment-analyzer | Listo | X problemas |
| test-reviewer | Listo | X problemas |
| error-handler-reviewer | Listo | X problemas |
| type-reviewer | Listo | X problemas |
| code-reviewer | Listo | X problemas |
| code-simplifier | Listo | X sugerencias |
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
- Los PR Comments estan listos para copiar y pegar en GitHub
