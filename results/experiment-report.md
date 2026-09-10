# Context Engineering Experiment

## Hypothesis
Si proporcionamos a un agente de codificación un contexto diseñado y desacoplado (especificaciones funcionales explícitas en `SPEC.md` y directrices operativas en `AGENTS.md`), el agente producirá una solución con mayor precisión arquitectónica, validaciones más robustas y menor tendencia a modificar archivos innecesarios, en comparación con un enfoque de contexto mínimo o puramente dependiente de prompts procedurales largos.

---

## Experimental Setup
- **Entorno:** Antigravity CLI / Gemini 3.8 Flash.
- **Base inicial común:** Commit `24dfa0a` (*"Initial experimental baseline"*), con la estructura básica en Python 3.11+ (`src/customer.py`, `src/repository.py`, `tests/test_customer.py`, `tests/test_repository.py`) y 2 pruebas fallando inicialmente por falta de implementación en la actualización del correo.
- **Suite de Pruebas:** `pytest`.
- **Rúbrica de Evaluación (100 puntos normalizados a escala sobre 10):**
  - Correctness (30 pts)
  - Requirements (20 pts)
  - Minimal Change (15 pts)
  - Maintainability (15 pts)
  - Security/Safety (10 pts)
  - Verification (10 pts)
- **Condiciones experimentales evaluadas:**
  1. **A — Minimal Context:** Commit `325f181` en la rama `A` (instrucción básica directa sin contexto estructurado adicional).
  2. **B — Repository Context:** Commit `f7a0f57` en la rama `B` (prompt procedural detallado con instrucciones paso a paso para inspeccionar el repositorio).
  3. **C — Engineered Context:** Commit `a03d13f` en la rama `C` (inclusión de archivos de gobernanza y especificación `SPEC.md` y `AGENTS.md` con prompt conciso).

---

## A — Minimal Context
- **Commit:** `325f181` (en la rama `A`)

### Prompt:
```text
Implement the customer email update functionality.
Inspect the repository first. Implement the necessary changes and run the tests.
```

### Results:
- **Tests passing:** 5 (4 originales + 1 nuevo agregado).
- **Tests failing:** 0.
- **Requisitos cumplidos:** 7 / 7.
- **Archivos modificados:** 3 (`src/customer.py`, `src/repository.py`, `tests/test_repository.py`).
- **Cambios innecesarios:** 1 (adición de test no solicitado en `tests/test_repository.py`).
- **Iteraciones:** 4.
- **Intervenciones humanas:** 0.
- **Problemas introducidos:** 0.
- **Tiempo:** 1m 20s.
- **Score:** 9.3 / 10.

### Human intervention:
- **0 intervenciones:** El agente resolvió la tarea de forma autónoma sin necesidad de reorientación manual.

### Score:
**9.3 / 10** (93 / 100 pts)
- *Correctness (30/30):* Todos los tests pasan y la funcionalidad se ejecuta correctamente.
- *Requirements (20/20):* Cumple todos los requisitos funcionales identificados.
- *Minimal Change (10/15):* Penalización por modificar el archivo `tests/test_repository.py` introduciendo un test que no fue requerido por el usuario ni por la especificación.
- *Maintainability (13/15):* El código es correcto, pero alterar los archivos de test sin autorización rompe la disciplina de control de cambios del repositorio.
- *Security/Safety (10/10):* Seguro, no incluye dependencias externas peligrosas ni secretos.
- *Verification (10/10):* Ejecutó pruebas y verificó resultados.

### Observations:
Al no contar con reglas de gobernanza ni barreras de contención explícitas, el agente decidió ampliar la suite de pruebas preexistente. Aunque la intención técnica parecía positiva (verificar su propia implementación), violó el principio de *menor cambio necesario* (*blast radius*) al modificar innecesariamente archivos fuera del código de producción.

---

## B — Repository Context
- **Commit:** `f7a0f57` (en la rama `B`)

### Prompt:
```text
Implement the customer email update functionality.

Before making changes:
1. Inspect the repository.
2. Read README.md.
3. Inspect all relevant source files.
4. Inspect the tests.
5. Infer expected behavior from the code and tests.
6. Run tests before changing code.
7. Make the smallest necessary implementation.
8. Run tests again.
9. Explain which repository information influenced the implementation.
```

### Results:
- **Tests passing:** 4.
- **Tests failing:** 0.
- **Requisitos cumplidos:** 7 / 7.
- **Archivos modificados:** 2 (`src/customer.py`, `src/repository.py`).
- **Cambios innecesarios:** 0.
- **Iteraciones:** 1.
- **Intervenciones humanas:** 0.
- **Problemas introducidos:** 0.
- **Tiempo:** 3min.
- **Score:** 9.2 / 10.

### Human intervention:
- **0 intervenciones:** El agente completó la secuencia procedimental paso a paso de forma autónoma.

### Score:
**9.2 / 10** (92 / 100 pts)
- *Correctness (25/30):* Pasa los tests unitarios existentes, pero presenta un problema menor de robustez: la validación implementada (`"@" not in new_email`) no valida la estructura real de un correo (acepta entradas inválidas como `"usuario@"`, `"@@"`, etc.), evidenciando *overfitting* a la suite de pruebas básica.
- *Requirements (18/20):* Cumple formalmente los casos probados en los tests existentes, pero el requisito de *validar email* queda satisfecho de forma deficiente/incompleta frente a estándares de producción.
- *Minimal Change (15/15):* Modificación estrictamente necesaria (solo 2 archivos de código, 0 tests tocados).
- *Maintainability (15/15):* Código limpio, modular y respetuoso de la arquitectura existente.
- *Security/Safety (9/10):* Penalización menor en validación de entradas (*input validation* frágil que admite cadenas malformadas).
- *Verification (10/10):* Ejecución rigurosa de pruebas antes y después de editar.

### Observations:
El prompt procedimental extenso guio al agente hacia una intervención limpia, evitando alteraciones a los tests. No obstante, al depender exclusivamente de inferir reglas desde las pruebas existentes sin una especificación formal, cayó en una validación superficial (`"@" not in new_email`), suficiente para pasar el test unitario básico, pero frágil en producción. Además, requirió el mayor tiempo de ejecución (3 minutos) debido a la sobrecarga de procesar paso a paso las 9 instrucciones procedimentales.

---

## C — Engineered Context
- **Commit:** `a03d13f` (en la rama `C`)

### Prompt:
```text
Implement the customer email update functionality.

Follow SPEC.md and AGENTS.md.
Inspect the repository first, run tests before and after changes, and explain your verification.
```

*Archivos de contexto en el repositorio:*
- `SPEC.md`: Especificación funcional formal (requisitos 1 al 7 y criterios de aceptación claros).
- `AGENTS.md`: Barandales de gobernanza (no modificar tests, preferir el menor cambio seguro, validación previa y posterior con pytest).

### Results:
- **Tests passing:** 4.
- **Tests failing:** 0.
- **Requisitos cumplidos:** 7 / 7.
- **Archivos modificados:** 2 (`src/customer.py`, `src/repository.py`).
- **Cambios innecesarios:** 0.
- **Iteraciones:** 4.
- **Intervenciones humanas:** 0.
- **Problemas introducidos:** 0.
- **Tiempo:** 1min 15s.
- **Score:** 10 / 10.

### Human intervention:
- **0 intervenciones:** Ejecución automatizada con estricta adherencia a los lineamientos del repositorio.

### Score:
**10 / 10** (100 / 100 pts)
- *Correctness (30/30):* Suite de pruebas superada al 100%.
- *Requirements (20/20):* Cumplimiento riguroso de cada criterio de `SPEC.md`.
- *Minimal Change (15/15):* Se apega estrictamente a la regla de `AGENTS.md` de no tocar tests y modificar únicamente lo imprescindible.
- *Maintainability (15/15):* Excelente calidad de código con tipado y validación de expresiones regulares en el dominio.
- *Security/Safety (10/10):* Validación robusta de sintaxis y tipos sin introducir dependencias externas.
- *Verification (10/10):* Verificación y reporte detallado de pruebas antes y después de los cambios.

### Observations:
Fue la solución más eficiente y robusta:
1. Logró el tiempo de respuesta más rápido del experimento (**1min 15s**).
2. La validación del correo fue completa y profesional (expresión regular exhaustiva) al estar explícitamente solicitada en `SPEC.md`.
3. `AGENTS.md` fungió como un barandal eficaz, impidiendo la modificación no deseada de las pruebas preexistentes que sí ocurrió en A.

---

## Comparative Results

| Métrica | A | B | C |
|---|:---:|:---:|:---:|
| **Commit** | `325f181` (rama `A`) | `f7a0f57` (rama `B`) | `a03d13f` (rama `C`) |
| **Tests passing** | 5 | 4 | 4 |
| **Tests failing** | 0 | 0 | 0 |
| **Requisitos cumplidos** | 7 / 7 | 7 / 7 | 7 / 7 |
| **Archivos modificados** | 3 (`src/customer.py`, `src/repository.py`, `tests/test_repository.py`) | 2 (`src/customer.py`, `src/repository.py`) | 2 (`src/customer.py`, `src/repository.py`) |
| **Cambios innecesarios** | 1 (adición de test no solicitado en `tests/test_repository.py`) | 0 | 0 |
| **Iteraciones** | 4 | 1 | 4 |
| **Intervenciones humanas** | 0 | 0 | 0 |
| **Problemas introducidos** | 0 | 0 | 0 |
| **Tiempo** | 1m 20s | 3min | 1min 15s |
| **Score /10** | **9.3 / 10** | **9.2 / 10** | **10 / 10** |

---

## Error Analysis
1. **Errores presentes en A y ausentes en C:**
   - En **A** (commit `325f181`), el agente realizó un cambio innecesario al agregar un test no solicitado dentro de `tests/test_repository.py`. Esto refleja una falta de delimitación de alcance. En **C** (commit `a03d13f`), la regla explícita de `AGENTS.md` (*"Do not modify tests unless explicitly requested"*) bloqueó este comportamiento por completo.
2. **Fragilidad de diseño y sobreajuste en B:**
   - En **B** (commit `f7a0f57`), al depender exclusivamente de inferencias sobre la suite existente sin especificaciones formales, el agente incurrió en *overfitting* a los tests: implementó una validación superficial (`"@" not in new_email`). Aunque hizo pasar las pruebas existentes, deja el sistema vulnerable a entradas no válidas (ej. `"usuario@"`, `"@@@"`), ameritando una penalización en *Correctness* (25/30) y *Requirements* (18/20) para una calificación final de **9.2 / 10**. En **C**, la especificación explícita en `SPEC.md` forzó una validación sintáctica completa y robusta con expresiones regulares.

---

## Context Quality Analysis
- **Información del repositorio más útil:**
  Las pruebas preexistentes (`tests/test_customer.py` y `tests/test_repository.py`), pues actuaron como la especificación ejecutable implícita para A y B.
- **Aporte de SPEC.md:**
  Definió con exactitud las reglas funcionales y de negocio (conversión a minúsculas, mensajes de excepción esperados como `"invalid-email"` y `"customer-not-found"`), evitando ambigüedades.
- **Función de AGENTS.md:**
  Gobernanza operativa del agente: estableció restricciones sobre el radio de cambio (*blast radius*), prohibió modificar pruebas y exigió verificación con `pytest` antes y después.
- **¿Más contexto significa necesariamente mejor contexto?:**
  No. Instrucciones largas y procedimentales en el prompt (como en B) ralentizan la respuesta (3 minutos) y aumentan la sobrecarga cognitiva del modelo. El contexto estructurado, modular y desacoplado (como en C) produce mejores resultados en menor tiempo (1m 15s).
- **Información redundante identificada:**
  Repetir instrucciones de flujo de trabajo en cada prompt individual cuando estas pueden residir estables en archivos del repositorio.

---

## Conclusions
1. **El contexto diseñado supera al prompting artesanal:**
   `SPEC.md` y `AGENTS.md` reducen la longitud de los prompts individuales y mejoran la calidad del código resultante.
2. **Mayor velocidad y predictibilidad:**
   El experimento C demostró ser el más rápido (1m 15s) y seguro, logrando puntuación perfecta sin sobrepasar el alcance.
3. **Control efectivo de cambios:**
   Las directivas permanentes de gobernanza son esenciales para evitar que los agentes alteren la suite de pruebas o la configuración del proyecto sin autorización.

---

## What I Would Change
1. **En SPEC.md:**
   - Detallar casos de prueba de borde (*edge cases*) para correos internacionales o con caracteres especiales permitidos por el estándar RFC 5322.
2. **En AGENTS.md:**
   - Incluir una regla de formateo y estilo automático mediante linters (ej. `ruff check` o `black`) como parte del *Quality Gate* antes de concluir.

---

### Reflexión Final (Pregunta 21)
**¿Por qué un desarrollador que utiliza agentes de código necesita aprender Context Engineering y no solamente mejores prompts?**

> Un desarrollador que utiliza agentes de código necesita aprender Context Engineering porque trabajar con un agente no consiste solamente en escribir buenos prompts. Aunque un prompt claro ayuda a indicarle qué queremos que haga, el agente también necesita conocer el contexto del proyecto para poder tomar buenas decisiones. Esto puede incluir el código existente, la arquitectura, las convenciones del proyecto, la documentación, las herramientas disponibles, los errores y los resultados de las pruebas.
>
> Context Engineering busca organizar y proporcionar esta información de manera adecuada, evitando tanto la falta de contexto como el exceso de información innecesaria. Esto es especialmente importante en proyectos grandes, donde el agente puede perderse entre muchos archivos o tomar decisiones basándose en información que no es relevante.
>
> Por lo tanto, mientras que Prompt Engineering se enfoca principalmente en cómo darle instrucciones al agente, Context Engineering se enfoca en qué información tiene disponible y cómo puede utilizarla. Para un desarrollador, entender esto permite aprovechar mejor los agentes y lograr que trabajen de una manera más consistente y confiable.
