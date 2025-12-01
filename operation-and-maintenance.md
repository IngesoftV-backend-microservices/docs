# Guías de Operación y Mantenimiento

## Introducción

Este documento proporciona guías prácticas para las operaciones diarias y el mantenimiento de la plataforma de microservicios desplegada en Azure Kubernetes Service (AKS). Incluye procedimientos para monitoreo, escalado, actualizaciones, troubleshooting y mantenimiento preventivo.

## Operaciones Diarias

### Verificación del Estado del Clúster

**Verificar nodos del clúster:**
```bash
kubectl get nodes
kubectl describe node <nombre-nodo>
```

**Verificar estado general de pods:**
```bash
kubectl get pods -n ecommerce-dev
kubectl get pods -n ecommerce-dev -o wide
```

**Verificar servicios:**
```bash
kubectl get services -n ecommerce-dev
kubectl get endpoints -n ecommerce-dev
```

**Verificar deployments:**
```bash
kubectl get deployments -n ecommerce-dev
kubectl rollout status deployment/<nombre-deployment> -n ecommerce-dev
```

### Verificación de Salud de Servicios

**Verificar health checks de un servicio:**
```bash
# Obtener el nombre del pod
kubectl get pods -n ecommerce-dev -l app=user-service

# Verificar logs del health check
kubectl logs <pod-name> -n ecommerce-dev | grep health

# Ejecutar health check manualmente
kubectl exec <pod-name> -n ecommerce-dev -- curl http://localhost:8700/user-service/actuator/health
```

**Verificar endpoints de Actuator:**
```bash
# Health general
curl http://<node-ip>:<nodeport>/user-service/actuator/health

# Liveness
curl http://<node-ip>:<nodeport>/user-service/actuator/health/liveness

# Readiness
curl http://<node-ip>:<nodeport>/user-service/actuator/health/readiness

# Métricas Prometheus
curl http://<node-ip>:<nodeport>/user-service/actuator/prometheus
```

### Monitoreo de Recursos

**Verificar uso de recursos:**
```bash
# Uso de recursos por pod
kubectl top pods -n ecommerce-dev

# Uso de recursos por nodo
kubectl top nodes

# Describir recursos de un pod
kubectl describe pod <pod-name> -n ecommerce-dev
```

**Verificar límites y requests:**
```bash
kubectl get pod <pod-name> -n ecommerce-dev -o jsonpath='{.spec.containers[*].resources}'
```

## Gestión de Pods y Servicios

### Reiniciar un Servicio

**Reiniciar un deployment:**
```bash
kubectl rollout restart deployment/<nombre-deployment> -n ecommerce-dev
kubectl rollout status deployment/<nombre-deployment> -n ecommerce-dev
```

**Eliminar un pod específico (se recreará automáticamente):**
```bash
kubectl delete pod <pod-name> -n ecommerce-dev
```

**Escalar un servicio:**
```bash
# Escalar a un número específico de réplicas
kubectl scale deployment/<nombre-deployment> --replicas=3 -n ecommerce-dev

# Verificar el escalado
kubectl get deployment <nombre-deployment> -n ecommerce-dev
```

### Verificación de Logs

**Ver logs de un pod:**
```bash
# Logs en tiempo real
kubectl logs -f <pod-name> -n ecommerce-dev

# Logs de los últimos 100 líneas
kubectl logs --tail=100 <pod-name> -n ecommerce-dev

# Logs de todos los pods de un deployment
kubectl logs -f deployment/<nombre-deployment> -n ecommerce-dev

# Logs de un contenedor específico (si hay múltiples contenedores)
kubectl logs <pod-name> -c <container-name> -n ecommerce-dev
```

**Buscar errores en logs:**
```bash
kubectl logs <pod-name> -n ecommerce-dev | grep -i error
kubectl logs <pod-name> -n ecommerce-dev | grep -i exception
```

### Troubleshooting de Pods

**Verificar eventos:**
```bash
# Eventos del namespace
kubectl get events -n ecommerce-dev --sort-by='.lastTimestamp'

# Eventos de un pod específico
kubectl describe pod <pod-name> -n ecommerce-dev
```

**Verificar estado de un pod:**
```bash
kubectl get pod <pod-name> -n ecommerce-dev -o yaml
kubectl describe pod <pod-name> -n ecommerce-dev
```

**Problemas comunes y soluciones:**

1. **Pod en estado `Pending`**:
   - Verificar recursos disponibles: `kubectl describe pod <pod-name> -n ecommerce-dev`
   - Verificar si hay nodos disponibles: `kubectl get nodes`
   - Verificar límites de recursos

2. **Pod en estado `CrashLoopBackOff`**:
   - Ver logs: `kubectl logs <pod-name> -n ecommerce-dev --previous`
   - Verificar configuración: `kubectl describe pod <pod-name> -n ecommerce-dev`
   - Verificar health checks

3. **Pod en estado `ImagePullBackOff`**:
   - Verificar que la imagen existe en ACR
   - Verificar autenticación AKS-ACR: `az aks check-acr --name <cluster-name> --resource-group <rg-name> --acr <acr-name>`
   - Verificar permisos del ServiceAccount

4. **Pod no responde a health checks**:
   - Verificar que el endpoint de health esté disponible
   - Verificar configuración de probes en el deployment
   - Verificar conectividad de red

## Escalado

### Escalado Manual

**Escalar un servicio específico:**
```bash
kubectl scale deployment/user-service --replicas=3 -n ecommerce-dev
kubectl scale deployment/order-service --replicas=2 -n ecommerce-dev
```

**Verificar el escalado:**
```bash
kubectl get deployment -n ecommerce-dev
kubectl get pods -n ecommerce-dev -l app=user-service
```

### Escalado del Clúster

**Escalar nodos del clúster (requiere Terraform o Azure CLI):**

Con Azure CLI:
```bash
# Escalar el node pool por defecto
az aks scale \
  --resource-group <resource-group-name> \
  --name <cluster-name> \
  --node-count 3 \
  --nodepool-name system
```

Con Terraform:
```hcl
# En environments/dev/terraform.tfvars
default_node_pool = {
  name            = "system"
  node_count      = 3  # Cambiar de 2 a 3
  vm_size         = "Standard_B2s"
  os_disk_size_gb = 30
  type            = "VirtualMachineScaleSets"
  max_pods        = 30
}
```

Luego aplicar:
```bash
cd infra
terraform apply -var-file="environments/dev/terraform.tfvars"
```

### Consideraciones de Escalado

- **Recursos disponibles**: Verificar que haya suficientes recursos en el clúster antes de escalar
- **Distribución de carga**: Asegurar que el load balancer distribuya correctamente el tráfico
- **Health checks**: Verificar que los servicios puedan manejar el aumento de carga
- **Costos**: Considerar el impacto en costos de Azure al escalar nodos

## Actualizaciones y Despliegues

### Actualizar una Imagen

**Actualizar la versión de una imagen:**
```bash
# Editar el deployment
kubectl set image deployment/<nombre-deployment> \
  <container-name>=<nueva-imagen>:<nueva-version> \
  -n ecommerce-dev

# Verificar el rollout
kubectl rollout status deployment/<nombre-deployment> -n ecommerce-dev
```

**Actualizar usando Kustomize:**
```bash
# Editar el archivo de deployment en manifests-k8s/base/services/
# Cambiar la versión de la imagen

# Aplicar cambios
kubectl apply -k manifests-k8s/overlays/dev/
```

### Rollback de un Deployment

**Ver historial de rollouts:**
```bash
kubectl rollout history deployment/<nombre-deployment> -n ecommerce-dev
```

**Hacer rollback a una versión anterior:**
```bash
# Rollback a la versión anterior
kubectl rollout undo deployment/<nombre-deployment> -n ecommerce-dev

# Rollback a una versión específica
kubectl rollout undo deployment/<nombre-deployment> --to-revision=2 -n ecommerce-dev

# Verificar el rollback
kubectl rollout status deployment/<nombre-deployment> -n ecommerce-dev
```

### Estrategias de Despliegue

El proyecto utiliza diferentes estrategias según el servicio:

- **Rolling Update** (por defecto): Actualización gradual sin downtime
- **Recreate**: Para servicios que no soportan múltiples versiones (ej: cloud-config)

**Verificar estrategia:**
```bash
kubectl get deployment <nombre-deployment> -n ecommerce-dev -o jsonpath='{.spec.strategy.type}'
```

## Monitoreo con Prometheus y Grafana

### Acceso a Grafana

**Obtener acceso a Grafana:**
```bash
# Obtener el NodePort de Grafana
kubectl get service grafana -n ecommerce-dev

# Acceder a Grafana
# http://<node-ip>:<nodeport>
# Usuario: admin
# Contraseña: admin (configurado en grafana-secret.yaml)
```

### Dashboards Disponibles

1. **Métricas Técnicas**: CPU, memoria, latencia, tasa de errores
2. **Métricas de Negocio - Order Service**: Total de órdenes, órdenes por estado, valor total
3. **Métricas de Negocio - Payment Service**: Total de pagos, tasa de éxito, pagos por estado

### Verificar Métricas en Prometheus

**Acceder a Prometheus:**
```bash
# Obtener el NodePort de Prometheus
kubectl get service prometheus -n ecommerce-dev

# Acceder a Prometheus
# http://<node-ip>:<nodeport>
```

**Consultas útiles:**
```promql
# Tasa de errores 5xx
sum(rate(http_server_requests_seconds_count{namespace="ecommerce-dev",status=~"5.."}[5m])) / sum(rate(http_server_requests_seconds_count{namespace="ecommerce-dev"}[5m]))

# CPU usage por pod
sum(rate(container_cpu_usage_seconds_total{namespace="ecommerce-dev"}[5m])) by (pod)

# Memoria usage por pod
sum(container_memory_working_set_bytes{namespace="ecommerce-dev"}) by (pod)

# Total de órdenes creadas
sum(ecommerce_orders_created_total{namespace="ecommerce-dev"})

# Tasa de éxito de pagos
(sum(ecommerce_payments_successful_total{namespace="ecommerce-dev"}) / sum(ecommerce_payments_total{namespace="ecommerce-dev"})) * 100
```

### Alertas

**Verificar alertas activas:**
```bash
# Acceder a Alertmanager
# http://<node-ip>:32093

# Ver reglas de alerta en Prometheus
# Navegar a: Alerts
```

**Alertas configuradas:**
- Alta tasa de errores 5xx (> 5%)
- Alta latencia (> 500ms)
- API Gateway caído
- Servicios sin pods activos

## Gestión de Logs con Kibana

### Acceso a Kibana

**Obtener acceso a Kibana:**
```bash
# Obtener el NodePort de Kibana
kubectl get service kibana -n ecommerce-dev

# Acceder a Kibana
# http://<node-ip>:<nodeport>
```

### Visualizar Logs

1. **Crear Index Pattern**:
   - Ir a Stack Management > Index Patterns
   - Crear patrón: `ecommerce-logs-*`
   - Seleccionar campo de tiempo: `@timestamp`

2. **Explorar Logs**:
   - Ir a Discover
   - Seleccionar el index pattern creado
   - Filtrar por campos: `kubernetes.namespace`, `kubernetes.pod.name`, `log.level`

### Búsquedas Útiles

**Buscar errores:**
```
log.level: ERROR
```

**Buscar logs de un servicio específico:**
```
kubernetes.labels.app: user-service
```

**Buscar logs de un pod específico:**
```
kubernetes.pod.name: user-service-xxxxx
```

**Buscar por mensaje:**
```
message: "Exception"
```

## Mantenimiento Preventivo

### Tareas Regulares

**Diarias:**
- Verificar estado de pods y servicios
- Revisar alertas en Prometheus/Alertmanager
- Verificar logs de errores en Kibana
- Monitorear uso de recursos

**Semanales:**
- Revisar métricas de negocio en Grafana
- Analizar tendencias de uso de recursos
- Verificar capacidad del clúster
- Revisar y limpiar logs antiguos

**Mensuales:**
- Revisar y actualizar versiones de imágenes
- Actualizar dependencias de seguridad
- Revisar y optimizar configuración de recursos
- Análisis de costos de Azure

### Limpieza de Recursos

**Limpiar pods terminados:**
```bash
kubectl delete pods --field-selector=status.phase==Succeeded -n ecommerce-dev
kubectl delete pods --field-selector=status.phase==Failed -n ecommerce-dev
```

**Limpiar recursos no utilizados:**
```bash
# Ver recursos no utilizados
kubectl get all -n ecommerce-dev

# Eliminar deployments, services, etc. no utilizados
kubectl delete deployment <nombre> -n ecommerce-dev
```

### Actualización de Kubernetes

**Verificar versión actual:**
```bash
kubectl version
az aks show --name <cluster-name> --resource-group <rg-name> --query kubernetesVersion
```

**Actualizar versión de Kubernetes (requiere Terraform):**
```hcl
# En environments/dev/terraform.tfvars
kubernetes_version = "1.33"  # Actualizar a nueva versión
```

**Nota**: El lifecycle policy en Terraform ignora cambios en `kubernetes_version` para permitir actualizaciones manuales controladas.

## Gestión de Configuración

### Actualizar ConfigMaps

**Ver ConfigMaps:**
```bash
kubectl get configmaps -n ecommerce-dev
kubectl describe configmap <nombre> -n ecommerce-dev
```

**Editar ConfigMap:**
```bash
kubectl edit configmap <nombre> -n ecommerce-dev
```

**Aplicar cambios:**
```bash
# Reiniciar pods para aplicar cambios
kubectl rollout restart deployment/<nombre-deployment> -n ecommerce-dev
```

### Actualizar Secrets

**Ver Secrets (sin mostrar valores):**
```bash
kubectl get secrets -n ecommerce-dev
kubectl describe secret <nombre> -n ecommerce-dev
```

**Actualizar Secret:**
```bash
# Crear nuevo secret
kubectl create secret generic <nombre> \
  --from-literal=username=<valor> \
  --from-literal=password=<valor> \
  -n ecommerce-dev \
  --dry-run=client -o yaml | kubectl apply -f -

# Reiniciar pods
kubectl rollout restart deployment/<nombre-deployment> -n ecommerce-dev
```

## Procedimientos de Emergencia

### Servicio Caído

1. **Verificar estado:**
   ```bash
   kubectl get pods -n ecommerce-dev -l app=<nombre-servicio>
   kubectl describe pod <pod-name> -n ecommerce-dev
   ```

2. **Ver logs:**
   ```bash
   kubectl logs <pod-name> -n ecommerce-dev --previous
   ```

3. **Reiniciar servicio:**
   ```bash
   kubectl rollout restart deployment/<nombre-deployment> -n ecommerce-dev
   ```

4. **Si no se recupera, hacer rollback:**
   ```bash
   kubectl rollout undo deployment/<nombre-deployment> -n ecommerce-dev
   ```

### Clúster No Responde

1. **Verificar nodos:**
   ```bash
   kubectl get nodes
   az aks show --name <cluster-name> --resource-group <rg-name>
   ```

2. **Verificar estado del clúster en Azure:**
   ```bash
   az aks show --name <cluster-name> --resource-group <rg-name> --query powerState
   ```

3. **Reiniciar nodos si es necesario:**
   ```bash
   az vm restart --ids $(az vm list -g <node-resource-group> --query "[].id" -o tsv)
   ```

### Pérdida de Datos

1. **Verificar backups** (si están configurados)
2. **Verificar estado de PersistentVolumes:**
   ```bash
   kubectl get pv
   kubectl get pvc -n ecommerce-dev
   ```
3. **Contactar al equipo de infraestructura** para recuperación desde backups

### Alto Uso de Recursos

1. **Identificar pods consumiendo más recursos:**
   ```bash
   kubectl top pods -n ecommerce-dev --sort-by=memory
   kubectl top pods -n ecommerce-dev --sort-by=cpu
   ```

2. **Verificar límites de recursos:**
   ```bash
   kubectl describe pod <pod-name> -n ecommerce-dev | grep -A 5 Resources
   ```

3. **Escalar horizontalmente:**
   ```bash
   kubectl scale deployment/<nombre-deployment> --replicas=<nuevo-numero> -n ecommerce-dev
   ```

4. **Escalar verticalmente (editar deployment):**
   ```bash
   kubectl edit deployment/<nombre-deployment> -n ecommerce-dev
   # Aumentar resources.requests y resources.limits
   ```

## Comandos Útiles de Referencia Rápida

### Verificación de Estado
```bash
# Estado general
kubectl get all -n ecommerce-dev

# Pods por servicio
kubectl get pods -n ecommerce-dev -l app=<nombre-servicio>

# Eventos recientes
kubectl get events -n ecommerce-dev --sort-by='.lastTimestamp' | tail -20
```

### Gestión de Deployments
```bash
# Ver deployments
kubectl get deployments -n ecommerce-dev

# Ver historial
kubectl rollout history deployment/<nombre> -n ecommerce-dev

# Reiniciar
kubectl rollout restart deployment/<nombre> -n ecommerce-dev

# Escalar
kubectl scale deployment/<nombre> --replicas=<n> -n ecommerce-dev
```

### Logs y Debugging
```bash
# Logs en tiempo real
kubectl logs -f deployment/<nombre> -n ecommerce-dev

# Logs anteriores (si el pod se reinició)
kubectl logs <pod-name> -n ecommerce-dev --previous

# Ejecutar comando en pod
kubectl exec -it <pod-name> -n ecommerce-dev -- /bin/sh
```

### Recursos y Configuración
```bash
# Uso de recursos
kubectl top pods -n ecommerce-dev
kubectl top nodes

# Ver configuración
kubectl get configmap -n ecommerce-dev
kubectl get secret -n ecommerce-dev

# Describir recurso
kubectl describe <tipo>/<nombre> -n ecommerce-dev
```

## Checklist de Operación Diaria

- [ ] Verificar estado de todos los pods: `kubectl get pods -n ecommerce-dev`
- [ ] Verificar que no haya pods en estado `CrashLoopBackOff` o `Error`
- [ ] Revisar alertas en Prometheus/Alertmanager
- [ ] Verificar métricas clave en Grafana (CPU, memoria, tasa de errores)
- [ ] Revisar logs de errores en Kibana
- [ ] Verificar uso de recursos del clúster: `kubectl top nodes`
- [ ] Verificar que los servicios respondan a health checks
- [ ] Revisar eventos recientes: `kubectl get events -n ecommerce-dev`

## Conclusión

Estas guías proporcionan los procedimientos necesarios para operar y mantener la plataforma de microservicios de forma efectiva y confiable. El seguimiento adecuado de las operaciones diarias, el monitoreo proactivo y el mantenimiento preventivo son fundamentales para garantizar la disponibilidad, rendimiento y estabilidad del sistema.

Los aspectos clave cubiertos en esta documentación incluyen:

- **Operaciones diarias**: Verificación continua del estado de servicios, health checks y monitoreo de recursos, permitiendo detectar problemas de forma temprana.
- **Gestión de servicios**: Procedimientos estandarizados para escalado, actualizaciones y rollbacks, minimizando el impacto en la disponibilidad del sistema.
- **Monitoreo y observabilidad**: Uso de Prometheus, Grafana y Kibana para tener visibilidad completa del estado del sistema, métricas de negocio y logs centralizados.
- **Troubleshooting**: Procedimientos estructurados para identificar y resolver problemas comunes, reduciendo el tiempo de resolución de incidentes.
- **Mantenimiento preventivo**: Tareas regulares que previenen problemas antes de que afecten a los usuarios finales.
- **Procedimientos de emergencia**: Respuestas rápidas y efectivas ante situaciones críticas que garantizan la continuidad del servicio.

La implementación de estas prácticas operacionales contribuye significativamente a la estabilidad y confiabilidad de la plataforma, permitiendo que el equipo pueda gestionar el sistema de manera eficiente y responder rápidamente ante cualquier situación que pueda afectar el servicio.

