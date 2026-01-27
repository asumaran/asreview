# Comprehensive PR Review

Ejecuta un review completo de pull request usando multiples agentes especializados en paralelo.

**Arguments:** "$ARGUMENTS"

---

## ⛔ REGLA CRITICA - DRAFT REVIEW OBLIGATORIO ⛔

**NUNCA, JAMAS, BAJO NINGUNA CIRCUNSTANCIA publicar comentarios directamente en un PR.**

Los reviews SIEMPRE deben crearse como **DRAFT (PENDING)** para que el usuario los revise antes de publicar.

**PROHIBIDO:**
- ❌ Usar `event: "COMMENT"` o `event: "APPROVE"` o `event: "REQUEST_CHANGES"` al crear el review
- ❌ Usar la API `/pulls/{pr}/comments` que crea comentarios sueltos (fuera del review)
- ❌ Cualquier accion que publique comentarios visibles al autor del PR sin aprobacion del usuario

**OBLIGATORIO:**
- ✅ Crear review SIN el parametro `event` (se crea como PENDING automaticamente)
- ✅ Usar la API `/pulls/{pr}/reviews` con array de `comments` incluido
- ✅ Esperar a que el usuario revise y publique manualmente

**Si tienes duda, NO envies el review. Pregunta al usuario primero.**

---

## Instrucciones de Idioma

**IMPORTANTE:**
- El reporte completo debe estar en **ESPAÑOL**
- Para cada hallazgo, incluir un **"PR Comment"** en **INGLES**, casual y breve
- Los PR Comments deben ser directos, como si hablaras con un colega
- **NO usar tablas** - usar listas para presentar los hallazgos

**FLUJO DE COMENTARIOS:**
- Los hallazgos **criticos** e **importantes** se envian automaticamente como **draft review** a GitHub
- Las **sugerencias** solo se muestran en el reporte (no se envian a GitHub)
- El draft queda pendiente para que el usuario lo revise y publique manualmente
- Despues de enviar, se inicia un flujo interactivo para ajustar o eliminar comentarios

Ejemplo de formato para cada hallazgo:
```
1. **[agente]** `archivo.ts:42`
   - **Problema:** Posible inyeccion SQL por concatenacion de strings
   - **PR Comment:** `hey, this query looks vulnerable to SQL injection - mind using parameterized queries instead?`
```

**FORMATO DE PR COMMENTS:**
- Usar backticks para referencias a codigo: variables, funciones, clases, tipos, etc.
- Ejemplos:
  - ❌ "the hasCapability method is duplicated"
  - ✅ "the `hasCapability` method is duplicated"
  - ❌ "validateExecutableNodes has N+1 queries"
  - ✅ "`validateExecutableNodes` has N+1 queries"
  - ❌ "consider using Promise.all"
  - ✅ "consider using `Promise.all`"

## Flujo del Review

### Paso 1: Identificar el PR

1. Si se proporciona numero o URL en argumentos, usar ese
2. Si no, verificar si el branch actual tiene PR abierto: `gh pr view --json number,title,body,url`
3. Si no hay PR, revisar cambios del branch actual contra main

### Paso 2: Checkout y Obtener Informacion del PR

```bash
# Checkout del branch del PR para acceso local a archivos
gh pr checkout <number>

# Detalles del PR
gh pr view <number> --json title,body,files,additions,deletions,baseRefName,headRefName,author,headRefOid

# Listar archivos cambiados (GUARDAR ESTA LISTA)
gh pr diff <number> --name-only
```

**IMPORTANTE:** Guardar la lista de archivos cambiados. Los agentes SOLO deben analizar estos archivos.

### Paso 3: Lanzar Agentes en Paralelo

Lanzar TODOS los siguientes agentes simultaneamente usando la herramienta Task. Cada agente debe recibir:
- La **lista exacta de archivos cambiados** (rutas completas)
- Instruccion de usar la herramienta **Read** para leer los archivos (NO el diff)
- Instruccion de reportar **numeros de linea exactos** del archivo real
- **Instruccion de idioma**: Reporte en español, PR comments en ingles casual
- **Instruccion de formato**: Usar listas, NO tablas
- **SOLO analizar los archivos de la lista** - ignorar cualquier otro archivo

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

Compilar los hallazgos en un reporte unico y conciso. **NO repetir informacion** - cada hallazgo debe aparecer solo una vez.

```markdown
# Resumen del Review

**PR:** #<numero> - <titulo>
**Autor:** <autor>
**Branch:** <head> -> <base>
**Archivos Cambiados:** <cantidad> (+<adiciones>/-<eliminaciones>)

---

## Que hace este PR

<Escribir 2-4 oraciones narrando de forma clara y concisa que implementa este PR basandose en los cambios reales del codigo, no en la descripcion del PR. Redactar como una historia facil de digerir que permita al revisor entender rapidamente el proposito y alcance de los cambios. Evitar lenguaje tecnico excesivo.>

Ejemplo:
> Este PR agrega la funcionalidad de exportar reportes a PDF. Introduce un nuevo servicio que convierte los datos del reporte a formato PDF usando la libreria jsPDF, y conecta un boton "Exportar" en la interfaz de usuario que dispara la descarga. Tambien incluye validaciones para manejar reportes vacios.

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

## Sin Hallazgos

Agentes que no encontraron problemas: <lista de agentes>
```

**IMPORTANTE:** NO agregar secciones adicionales despues de los hallazgos. El reporte termina aqui.

### Paso 5: Verificar y Enviar Draft Review

Despues de compilar el reporte, verificar cada hallazgo critico e importante:

1. **Leer el codigo real** de cada archivo/linea mencionado usando `gh pr diff <number>` o la herramienta Read
2. **Descartar falsos positivos** - si el hallazgo no aplica al codigo real, eliminarlo del reporte
3. **Obtener el commit SHA** del PR: `gh pr view <number> --json headRefOid -q .headRefOid`
4. **Enviar como draft review** a GitHub usando UNA SOLA llamada a la API:

```bash
# ⛔ CRITICO: NO incluir el parametro "event" - esto mantiene el review como PENDING (draft)
# ⛔ NUNCA usar: -f event="COMMENT" ni -f event="APPROVE" ni -f event="REQUEST_CHANGES"
# ⛔ NUNCA usar la API /pulls/{pr}/comments - eso crea comentarios SUELTOS visibles inmediatamente

# ✅ CORRECTO: Crear review SIN event, con todos los comentarios en el body JSON
cat << 'EOF' | gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews -X POST --input -
{
  "commit_id": "<commit_sha>",
  "body": "<mensaje casual y breve>",
  "comments": [
    {
      "path": "ruta/al/archivo.ts",
      "line": 42,
      "body": "comentario en ingles casual"
    }
  ]
}
EOF
```

**MENSAJE DEL REVIEW (body):**
- Debe ser casual, breve y amigable (1 linea)
- Generarlo dinamicamente segun el contexto del PR
- Ejemplos:
  - "Looking good! Just leaving a few comments."
  - "Nice work on this! A few things caught my eye."
  - "Solid PR, just some minor suggestions."
  - "Great progress! Left some thoughts below."
- NO incluir conteos, titulos formales, ni resumenes extensos

---

**⛔ PROHIBIDO - NUNCA HACER:**
- ❌ `gh api .../reviews -f event="COMMENT"` → PUBLICA EL REVIEW INMEDIATAMENTE
- ❌ `gh api .../reviews -f event="APPROVE"` → APRUEBA Y PUBLICA INMEDIATAMENTE
- ❌ `gh api .../reviews -f event="REQUEST_CHANGES"` → PUBLICA INMEDIATAMENTE
- ❌ `gh api .../pulls/{pr}/comments` → CREA COMENTARIOS SUELTOS VISIBLES INMEDIATAMENTE
- ❌ Crear review primero y agregar comentarios despues por separado

**✅ OBLIGATORIO:**
- ✅ Usar `gh api .../reviews` SIN el parametro `event`
- ✅ Incluir array `comments` en el JSON body
- ✅ Verificar que la respuesta tenga `"state": "PENDING"`
- ✅ Si la respuesta NO tiene `"state": "PENDING"`, ALERTAR AL USUARIO INMEDIATAMENTE

---

**IMPORTANTE:**
- Solo enviar hallazgos **criticos** e **importantes** (NO sugerencias)
- El review queda como PENDING (draft) hasta que el usuario lo publique manualmente
- Guardar el **review_id** retornado para poder modificar/eliminar comentarios despues
- Informar al usuario cuantos comentarios se enviaron y el review_id

**VERIFICACION POST-CREACION (OBLIGATORIA):**
Despues de crear el review, SIEMPRE verificar que el estado sea PENDING:
```bash
# La respuesta de la API debe incluir "state": "PENDING"
# Si ves "state": "COMMENTED" o cualquier otro estado, ALGO SALIO MAL
# En ese caso, ALERTAR AL USUARIO INMEDIATAMENTE y ofrecer eliminar los comentarios
```

Si el review NO quedo como PENDING:
1. Informar al usuario del error
2. Ofrecer eliminar los comentarios publicados
3. Investigar que salio mal antes de reintentar

### Paso 6: Iteracion Interactiva

Despues de enviar el draft, iniciar flujo interactivo con el usuario:

```
Se creo draft review (ID: <review_id>) con X comentarios. Vamos a revisarlos uno por uno.

---

**Comentario 1/X** - `archivo.ts:42`

Codigo:
[Mostrar las lineas relevantes del codigo usando el diff del PR]

Comentario enviado:
> hey, this query looks vulnerable to SQL injection...

¿Que hacemos?
- "ok" - mantener como esta
- "ajusta: <nuevo texto>" - modificar el comentario
- "elimina" - borrar este comentario
```

Para obtener los comentarios del review:
```bash
gh api repos/{owner}/{repo}/pulls/{pr_number}/reviews/{review_id}/comments
```

Para cada respuesta del usuario:
- **"ok"** → pasar al siguiente comentario
- **"ajusta: X"** → actualizar el comentario:
  ```bash
  gh api repos/{owner}/{repo}/pulls/comments/{comment_id} -X PATCH -f body="<nuevo texto>"
  ```
- **"elimina"** → eliminar el comentario:
  ```bash
  gh api repos/{owner}/{repo}/pulls/comments/{comment_id} -X DELETE
  ```

Al terminar todos los comentarios:
```
Revision completada. El draft review (ID: <review_id>) tiene X comentarios listos para publicar.

Puedes ir a GitHub para revisar y publicar el review manualmente.
```

**IMPORTANTE:** NUNCA ofrecer publicar el review. La publicacion es SIEMPRE manual por el usuario.

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
- Hallazgos criticos e importantes se verifican leyendo el codigo real antes de enviar
- Los comentarios se envian como **draft review** (pendiente) para revision del usuario
- Flujo interactivo permite ajustar o eliminar comentarios antes de publicar
