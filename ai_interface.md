# Interfaz con el Modelo de IA (MCP - Fase de Verificación y Recomendaciones)

Este documento define el contrato de datos para la interacción entre la lógica principal de la herramienta de actualización de EKS y el modelo de Inteligencia Artificial encargado de la verificación pre-actualización y la generación de recomendaciones.

## 1. Entrada al Modelo de IA

El modelo de IA espera recibir el Contexto completo recopilado y estructurado según el esquema definido en `context_schema.json`.

* **Formato:** Objeto JSON.
* **Ejemplo:** Ver `examples/sample_context.yaml` (que, una vez parseado, se convierte en un objeto JSON).
* **Mecanismo de Entrega:** (Esto es una decisión de implementación, pero la interfaz define el formato). Podría ser:
    * El cuerpo de una solicitud HTTP POST a un endpoint de inferencia del modelo.
    * Un archivo JSON leído por el script del modelo.
    * Un objeto pasado directamente si el modelo se ejecuta como una librería o módulo dentro de la herramienta.

El modelo de IA debe ser capaz de parsear este objeto JSON y acceder a los datos relevantes (ej. versiones, estado, configuraciones, datos de compatibilidad) utilizando las rutas definidas en el esquema.

## 2. Salida del Modelo de IA (Resultados de Verificación y Recomendaciones)

El modelo de IA debe retornar un objeto JSON con los resultados de su análisis de verificación y las recomendaciones generadas.

* **Formato:** Objeto JSON.
* **Esquema de Salida:**
    ```json
    {
      "verification_status": {
        "type": "string",
        "enum": ["success", "warning", "error"],
        "description": "Overall status of the pre-update verification."
      },
      "summary": {
        "type": "string",
        "description": "A brief human-readable summary of the verification outcome."
      },
      "issues": {
        "type": "array",
        "description": "List of specific problems, potential risks, or incompatibilities found.",
        "items": {
          "type": "object",
          "properties": {
            "code": { "type": "string", "description": "A unique code for the type of issue (e.g., V_ADDON_001, V_NODE_HEALTH_002)." },
            "description": { "type": "string", "description": "Detailed description of the issue found." },
            "severity": { "type": "string", "enum": ["low", "medium", "high", "critical"], "description": "Severity level of the issue." },
            "related_context_path": { "type": "string", "description": "Optional: JSONPath or pointer indicating which part of the input context the issue relates to." },
             "details": {
                "type": "object",
                "description": "Optional: Additional structured details about the issue.",
                "additionalProperties": true // Allow arbitrary key-value pairs for specific issue details
             }
          },
          "required": ["code", "description", "severity"]
        }
      },
      "recommendations": {
        "type": "array",
        "description": "List of recommended actions based on the verification results.",
        "items": {
          "type": "object",
          "properties": {
            "code": { "type": "string", "description": "A unique code for the recommendation (e.g., R_ADDON_UPGRADE_001, R_INVESTIGATE_NODE_002)." },
            "description": { "type": "string", "description": "Detailed description of the recommended action." },
            "severity": { "type": "string", "enum": ["low", "medium", "high", "critical"], "description": "Severity level or priority of the recommendation." },
            "suggested_action": { "type": "string", "description": "Optional: A specific command or concrete step the user can take." },
            "mitigates_issues": {
              "type": "array",
              "items": {"type": "string"},
              "description": "Optional: List of issue codes that this recommendation helps mitigate."
            }
          },
          "required": ["code", "description", "severity"]
        }
      }
    }
    ```
* **Ejemplo de Salida:** Ver `examples/sample_recommendations.json`.
* **Mecanismo de Entrega:** (Decisión de implementación). Podría ser:
    * La respuesta HTTP del endpoint de inferencia.
    * Impreso en la salida estándar por un script.
    * Retornado como un objeto por una función.

La herramienta de actualización debe ser capaz de recibir y parsear esta salida JSON para presentar los resultados al usuario y potencialmente guiar los siguientes pasos del proceso de actualización.