# Fuentes de Datos del Contexto (MCP - Fase de Verificación)

Este documento detalla las fuentes y métodos de recopilación para construir el objeto de Contexto definido en `context_schema.json`. La herramienta de actualización es responsable de orquestar estas llamadas y transformar los resultados en la estructura del esquema.

## 1. cluster_info

* `name`, `arn`, `region`, `current_k8s_version`:
    * **Fuente:** API de AWS EKS.
    * **Método:** `aws eks describe-cluster --name <cluster-name>`. Extraer los campos `cluster.name`, `cluster.arn`, `cluster.region`, `cluster.version`.
    * **Permisos AWS Requeridos:** `eks:DescribeCluster`.
* `target_k8s_version`:
    * **Fuente:** Entrada del usuario a la herramienta o archivo de configuración de la herramienta.
    * **Método:** Lectura directa.

## 2. node_groups

* Lista de Node Groups:
    * **Fuente:** API de AWS EKS.
    * **Método:** `aws eks list-node-groups --cluster-name <cluster-name>`.
    * **Permisos AWS Requeridos:** `eks:ListNodeGroups`.
* Detalles de cada Node Group (`name`, `type`, `instance_type`, `ami_id`, `scaling_config`, `status`, `node_count`, `health`, `tags`):
    * **Fuente:** API de AWS EKS y potencialmente Kubernetes API para estado de nodos más detallado.
    * **Método:** Por cada Node Group listado: `aws eks describe-node-group --cluster-name <cluster-name> --node-group-name <ng-name>`. Para estado detallado de nodos en Self-Managed o visibilidad extra: Kubectl `get nodes`.
    * **Permisos AWS Requeridos:** `eks:DescribeNodeGroup`.
    * **Permisos K8s Requeridos:** `nodes:list` (para obtener estado de nodos si aplica).
    * **Nota:** Obtener `kubelet_version` a menudo implica describir la AMI o conectarse a los nodos; puede requerir pasos adicionales o inferirse de la versión de Kubernetes.

## 3. addons

* Lista de Add-ons y sus detalles (`name`, `current_version`, `resolve_conflicts`, `status`, `configuration_values`):
    * **Fuente:** API de AWS EKS.
    * **Método:** `aws eks list-addons --cluster-name <cluster-name>`, seguido de `aws eks describe-addon --cluster-name <cluster-name> --addon-name <addon-name>` para cada uno.
    * **Permisos AWS Requeridos:** `eks:ListAddons`, `eks:DescribeAddon`.

## 4. system_component_status

* Estado de pods críticos del sistema (`name`, `namespace`, `resource_type`, `current_replicas`, `desired_replicas`, `status`, `warnings`):
    * **Fuente:** Kubernetes API.
    * **Método:** Kubectl `get deployments -n kube-system`, `get daemonsets -n kube-system`, `get pods -n kube-system`. Analizar el output para obtener recuentos de réplicas y estado general. Kubectl `describe pod <pod-name> -n <namespace>` para eventos/advertencias.
    * **Permisos K8s Requeridos:** `deployments:list`, `daemonsets:list`, `pods:list`, `pods:get`.

## 5. network_security_config

* `vpc_id`, `subnet_ids`, `security_group_ids` (asociados al clúster/nodos):
    * **Fuente:** API de AWS EKS y EC2.
    * **Método:** `aws eks describe-cluster --name <cluster-name>` (para VPC y subredes asociadas al control plane y configuraciones de red). `aws ec2 describe-security-groups --filters "Name=tag:kubernetes.io/cluster/<cluster-name>,Values=owned"` o `aws ec2 describe-network-interfaces --filters "Name=vpc-id,Values=<vpc-id>" ...` (para encontrar SGs asociados a ENIs de CNI o nodos).
    * **Permisos AWS Requeridos:** `eks:DescribeCluster`, `ec2:DescribeSecurityGroups`, `ec2:DescribeNetworkInterfaces`.
* `iam_roles` (`cluster_role_arn`, `node_instance_role_arn`):
    * **Fuente:** API de AWS EKS y EC2/IAM.
    * **Método:** `aws eks describe-cluster --name <cluster-name>` (para el rol del clúster). `aws eks describe-node-group --cluster-name <cluster-name> --node-group-name <ng-name> --query 'nodeGroup.nodeRole'` (para el rol de instancia de nodo). Puede requerir `iam:ListRoles` o `iam:GetRole` si se necesita verificar detalles del rol.
    * **Permisos AWS Requeridos:** `eks:DescribeCluster`, `eks:DescribeNodeGroup`, `iam:ListRoles`, `iam:GetRole` (si es necesario).

## 6. compatibility_data

* `k8s_addon_matrix`, `version_requirements`:
    * **Fuente:** Documentación oficial de AWS EKS, bases de conocimiento internas, información scrapeada de lanzamientos de AWS.
    * **Método:** Obtención y parseo de datos de fuentes externas. Esto es típicamente información estática o semi-estática mantenida por la herramienta o un servicio externo.
* `known_issues`:
    * **Fuente:** AWS EKS Release Notes, GitHub issues de proyectos relevantes (Kubernetes, CNI, CoreDNS), bases de conocimiento internas.
    * **Método:** Similar a los datos de compatibilidad; puede implicar web scraping, APIs de rastreo de issues o mantenimiento manual de una base de datos interna.

## 7. history

* `last_update_status`, `last_update_timestamp`, `previous_issues`:
    * **Fuente:** Registros internos de la herramienta de actualización, logs históricos (CloudWatch Logs, sistema de logging centralizado), o un sistema de CMDB/historial de operaciones.
    * **Método:** Consulta a la base de datos o sistema de archivos donde la herramienta registra sus operaciones pasadas. Requiere que la herramienta persista estos datos.

---

**Nota Importante:** La implementación concreta de la "Recopilación de Datos" debe manejar errores, reintentos, paginación de APIs y transformar los resultados de las APIs en el formato exacto definido en `context_schema.json`.