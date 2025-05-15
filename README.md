# Model Context Protocol (MCP) para Herramienta de Actualización de EKS con IA

## Tabla de Contenidos

1. [Introducción](#1-introducción)
2. [Propósito del MCP](#2-propósito-del-mcp-en-este-proyecto)
3. [Conceptos Clave](#3-conceptos-clave)
4. [Estructura del Contexto MCP](#4-estructura-del-contexto-mcp)
5. [Flujo del Protocolo MCP](#5-flujo-del-protocolo-mcp)
6. [Ejemplo de Contexto JSON](#6-ejemplo-de-contexto-json)
7. [Diagrama del Flujo MCP](#7-diagrama-del-flujo-mcp)
8. [Próximos Pasos](#8-próximos-pasos)
9. [Contribuciones](#9-contribuciones)

---

## 1. Introducción

El **Model Context Protocol (MCP)** es el componente central de nuestra herramienta para actualizar clústeres Amazon EKS. Define cómo se recopila, estructura y gestiona la información relevante ("Contexto") que se entrega a un modelo de Inteligencia Artificial, permitiendo automatizar la verificación previa y la generación de recomendaciones para el proceso de actualización.

## 2. Propósito del MCP

El MCP busca:

- Proveer al modelo de IA una vista unificada y estructurada del estado actual del clúster EKS.
- Asegurar que el modelo de IA tenga acceso a todo el contexto necesario para evaluar la preparación del clúster para una actualización.
- Facilitar la identificación proactiva de posibles problemas, incompatibilidades o riesgos antes de la actualización.
- Permitir la generación de recomendaciones precisas y accionables.

## 3. Conceptos Clave

- **Contexto:** Datos relevantes sobre el clúster EKS, su entorno y el proceso de actualización.
- **Protocolo:** Reglas y formatos para la recopilación, estructuración y entrega del Contexto al modelo de IA.
- **Modelo de IA:** Componente que consume el Contexto para análisis y generación de recomendaciones.

## 4. Estructura del Contexto MCP

El Contexto MCP incluye:

- **Información General del Clúster:** Nombre, ARN, ID, región, versiones actual y objetivo.
- **Node Groups:** Lista y detalles de node groups (tipo, escalado, AMI, Kubelet, estado, taints, métricas).
- **Add-ons Gestionados:** Lista y detalles de add-ons instalados (nombre, versión, estado, configuración).
- **Componentes Críticos y Cargas de Trabajo:** Estado de pods y réplicas en namespaces críticos, estado general de aplicaciones.
- **Red y Seguridad:** VPC, subredes, security groups, roles IAM.
- **Compatibilidad:** Información sobre compatibilidad de versiones y requisitos previos.
- **Historial:** Registros de actualizaciones anteriores y patrones históricos.

## 5. Flujo del Protocolo MCP

1. **Recopilación de Datos:** Interacción con APIs de AWS/Kubernetes y fuentes externas.
2. **Estructuración del Contexto:** Transformación y organización de datos en la estructura definida.
3. **Presentación al Modelo de IA:** El Contexto se pasa como entrada al modelo.
4. **Procesamiento por el Modelo de IA:** Análisis del Contexto, identificación de riesgos y generación de recomendaciones.
5. **Generación de Salida:** Resultado de la verificación y lista de recomendaciones.
6. **Consumo por la Herramienta:** Presentación de resultados al usuario o toma de decisiones automatizadas.

## 6. Ejemplo de Contexto JSON

```json
{
  "cluster_info": {
    "name": "mi-cluster",
    "arn": "arn:aws:eks:us-west-2:123456789012:cluster/mi-cluster",
    "id": "mi-cluster-id",
    "region": "us-west-2",
    "current_version": "1.18",
    "target_version": "1.21"
  },
  "node_groups": [
    {
      "name": "ng-1",
      "instance_type": "t3.medium",
      "scaling_details": {"min_size": 2, "max_size": 5, "desired_size": 3},
      "ami_version": "ami-0abcdef1234567890",
      "kubelet_version": "1.18",
      "status": "Ready",
      "taints": [{"key": "dedicated", "value": "gpu", "effect": "NoSchedule"}],
      "resource_metrics": {"cpu": "70%", "memory": "80%"}
    }
  ],
  "addons": [
    {
      "name": "vpc-cni",
      "current_version": "1.7.5",
      "status": "ACTIVE",
      "configuration": {"eni_config": "true", "max_enis": 5}
    }
  ],
  "system_components": {
    "kube_system": {
      "pods_status": {"Running": 50, "Pending": 2, "Failed": 1},
      "replicas": {"deployments": 10, "statefulsets": 2}
    }
  },
  "networking": {
    "vpc_id": "vpc-1234abcd",
    "subnet_cidrs": ["10.0.0.0/24", "10.0.1.0/24"],
    "security_groups": ["sg-1234abcd", "sg-5678efgh"]
  },
  "iam_roles": {
    "cluster_role": "arn:aws:iam::123456789012:role/EKS-Cluster-Role",
    "node_instance_role": "arn:aws:iam::123456789012:role/EKS-NodeInstance-Role"
  },
  "compatibility_data": {
    "kubernetes": {"current": "1.18", "target": "1.21", "compatible": true},
    "addons": [{"name": "vpc-cni", "version": "1.7.5", "compatible": true}]
  },
  "upgrade_history": [
    {
      "timestamp": "2021-06-01T12:00:00Z",
      "version": "1.18",
      "result": "Success",
      "duration_minutes": 30,
      "issues": []
    }
  ]
}
```

## 7. Diagrama del Flujo MCP

```mermaid
flowchart TD
    A[Fuentes de Datos<br>(AWS APIs, K8s API, Documentación, Historial)] --> B[Módulo de Recopilación de Datos]
    B --> C[MCP: Estructuración del Contexto]
    C --> D[Contexto Estructurado]
    D --> E[Modelo de IA<br>(Verificación y Recomendaciones)]
    E --> F[Resultados del Modelo<br>(Estado y Recomendaciones)]
    F --> G[Lógica Principal de la Herramienta<br>(Presentación/Toma de Acciones)]
```

## 8. Próximos Pasos

- Implementar el módulo de recopilación de datos.
- Definir y codificar la estructura del Contexto.
- Integrar con el modelo de IA.
- Desarrollar la lógica de verificación y recomendaciones.
- Realizar pruebas y validación.

## 9. Contribuciones

Las contribuciones son bienvenidas. Por favor, siga las pautas de contribución establecidas en el repositorio.

---