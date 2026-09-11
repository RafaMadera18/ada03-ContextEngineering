# ADA-03 — Context Engineering Experiment
> **Ingeniería de Software asistida por IA · UADY**

Este repositorio contiene el desarrollo y análisis del experimento de **Context Engineering**, cuyo objetivo es responder a la pregunta de investigación:
> *¿Cómo cambia el resultado de un coding agent cuando mejora la calidad y estructura del contexto disponible?*

---

## 📋 Descripción del Proyecto

El proyecto implementa un módulo en Python para la gestión de clientes (`Customer API`) con persistencia en memoria y reglas de validación/auditoría. 

La tarea asignada al agente de código consistió en implementar la funcionalidad de actualización de correo electrónico (`update_email` / `update_customer_email`), evaluando su desempeño bajo tres condiciones controladas de contexto.

---

## 🔬 Condiciones Experimentales y Ramas

El experimento se ejecutó de forma aislada a partir de una línea base común (`commit 24dfa0a`), organizándose en ramas independientes:

| Condición | Rama | Commit | Descripción del Contexto |
|---|:---:|:---:|---|
| **A — Minimal Context** | `A` | `325f181` | Prompt básico y directo sin contexto adicional ni barandales. |
| **B — Repository Context** | `B` | `f7a0f57` | Prompt procedimental detallado (9 pasos) indicando cómo inspeccionar el código y deducir reglas. |
| **C — Engineered Context** | `C` | `a03d13f` | Contexto estructurado en el repositorio con especificación formal (`SPEC.md`) y reglas operativas (`AGENTS.md`), con prompt conciso. |

---

## 📁 Estructura del Repositorio

```text
ada-03-context-engineering/
├── README.md               # Documentación general del proyecto y experimento
├── SPEC.md                 # Especificación formal de requisitos y criterios de aceptación (Condición C)
├── AGENTS.md               # Directrices de gobernanza, barandales y calidad para agentes (Condición C)
├── pyproject.toml          # Configuración de dependencias y empaquetado
├── src/
│   ├── customer.py         # Entidad Customer y lógica de dominio
│   └── repository.py       # Repositorio CustomerRepository en memoria
├── tests/
│   ├── test_customer.py    # Pruebas unitarias de dominio
│   └── test_repository.py  # Pruebas unitarias del repositorio
└── results/
    └── experiment-report.md # Reporte final comparativo y análisis del experimento
```

---

## 🚀 Requisitos e Instalación

- **Python 3.11+**
- **pytest**

Para configurar el entorno y ejecutar las pruebas:

```bash
# Crear entorno virtual
python -m venv .venv

# Activar entorno virtual
# En Windows:
.venv\Scripts\activate
# En Linux/macOS:
source .venv/bin/activate

# Instalar dependencias
pip install -e .
```

### Ejecutar Pruebas Automatizadas

```bash
pytest -v
```

---

## 📊 Resumen de Resultados

Los resultados cuantitativos y cualitativos recopilados en las tres ejecuciones se resumen en la siguiente tabla:

| Métrica | A (Minimal) | B (Repository) | C (Engineered) |
|---|:---:|:---:|:---:|
| **Commit** | `325f181` | `f7a0f57` | `a03d13f` |
| **Tests passing** | 5 | 4 | 4 |
| **Tests failing** | 0 | 0 | 0 |
| **Requisitos cumplidos** | 7 / 7 | 7 / 7 | 7 / 7 |
| **Archivos modificados** | 3 (`src/`, `tests/`) | 2 (`src/`) | 2 (`src/`) |
| **Cambios innecesarios** | 1 (test no solicitado) | 0 | 0 |
| **Iteraciones** | 4 | 1 | 4 |
| **Intervenciones humanas** | 0 | 0 | 0 |
| **Tiempo de respuesta** | 1m 20s | 3min | **1min 15s** |
| **Score /10** | **9.3 / 10** | **9.2 / 10** | **10 / 10** |

Para el análisis detallado de errores, calidad del contexto, preguntas de reflexión y conclusiones completas, consulta el reporte en:
👉 [`results/experiment-report.md`](results/experiment-report.md)

---

## 💡 Hallazgos Principales

1. **Contexto Estructurado vs. Prompts Largos:** Desacoplar las reglas en archivos versionados (`SPEC.md` y `AGENTS.md`) redujo el tiempo de ejecución a casi la mitad comparado con prompts procedimentales largos (1m 15s frente a 3m), obteniendo una implementación más limpia.
2. **Control del *Blast Radius*:** Sin directivas de gobernanza (en A), el agente modificó archivos de test no solicitados. Las políticas claras en `AGENTS.md` evitaron alteraciones accidentales en el proyecto.
3. **Robustez ante el *Overfitting*:** En B, el agente dedujo las reglas únicamente de las pruebas existentes e implementó una validación débil (`"@" not in email`). En C, contar con requisitos formales promovió una validación robusta con expresiones regulares y chequeo de tipos.