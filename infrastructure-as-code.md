# Infraestructura como Código (IaC)

## Introducción

Este proyecto implementa la infraestructura como código utilizando **Terraform** para gestionar los recursos de Azure necesarios para el despliegue de la plataforma de microservicios. La infraestructura se define de forma declarativa, versionada y reproducible, permitiendo gestionar de manera consistente los entornos de desarrollo y producción.

## Arquitectura de la Infraestructura

La infraestructura desplegada incluye los siguientes componentes principales:

### Componentes Principales

1. **Azure Kubernetes Service (AKS)**: Clúster de Kubernetes gestionado para ejecutar los microservicios.
2. **Azure Container Registry (ACR)**: Registro privado de contenedores para almacenar las imágenes Docker de los microservicios.
3. **Virtual Network (VNet)**: Red virtual con subredes para aislar y organizar los recursos.
4. **Resource Group**: Grupo de recursos de Azure que contiene todos los componentes de la infraestructura.

### Integración entre Componentes

- El clúster AKS está configurado para autenticarse automáticamente con ACR mediante una asignación de roles (`AcrPull`), permitiendo que los nodos de Kubernetes extraigan imágenes sin necesidad de credenciales explícitas.
- La red virtual proporciona aislamiento y segmentación mediante subredes dedicadas para AKS y otros servicios.

## Estructura del Proyecto

```
infra/
├── main.tf                    # Configuración principal y orquestación de módulos
├── variables.tf               # Variables globales del proyecto
├── outputs.tf                 # Outputs globales
├── versions.tf                # Versiones de Terraform y proveedores
├── terraform.tfvars.example   # Ejemplo de configuración
├── environments/              # Configuraciones por entorno
│   ├── dev/
│   │   ├── terraform.tfvars.example
│   │   └── terraform.tfvars  # Configuración real (no versionada)
│   └── prod/
│       ├── terraform.tfvars.example
│       └── terraform.tfvars  # Configuración real (no versionada)
├── modules/                   # Módulos reutilizables de Terraform
│   ├── resource-group/       # Módulo para Resource Group
│   ├── networking/           # Módulo para Virtual Network y Subnets
│   ├── acr/                  # Módulo para Azure Container Registry
│   └── kubernetes-cluster/   # Módulo para AKS
└── scripts/                  # Scripts de automatización
    ├── setup-backend.sh      # Configuración del backend de Terraform
    ├── init-env.sh           # Inicialización de archivos de entorno
    ├── deploy.sh             # Script de despliegue
    └── destroy.sh            # Script de destrucción
```

## Módulos de Terraform

### 1. Resource Group (`modules/resource-group/`)

**Propósito**: Crea y gestiona el grupo de recursos de Azure que contiene todos los componentes.

**Recursos**:
- `azurerm_resource_group`: Grupo de recursos con nombre y ubicación configurables.

**Características**:
- Nomenclatura: `{name}-{environment}` (ej: `rg-ecommerce-microservices-dev`)
- Tags configurables para organización y gestión de costos
- Lifecycle policy para prevenir destrucción accidental

**Outputs**:
- `name`: Nombre del resource group
- `id`: ID del resource group
- `location`: Ubicación del resource group

### 2. Networking (`modules/networking/`)

**Propósito**: Crea la red virtual y las subredes necesarias para el clúster AKS.

**Recursos**:
- `azurerm_virtual_network`: Red virtual con espacio de direcciones configurable
- `azurerm_subnet`: Subredes para AKS y otros servicios (Application Gateway, etc.)

**Características**:
- Espacio de direcciones configurable (por defecto: `10.0.0.0/16`)
- Subredes configurables con prefijos de red específicos
- Nomenclatura: `{subnet_name}-{environment}` (ej: `aks-subnet-dev`)

**Outputs**:
- `vnet_id`: ID de la red virtual
- `vnet_name`: Nombre de la red virtual
- `aks_subnet_id`: ID de la subred de AKS (primera subred)
- `subnet_ids`: Lista de IDs de todas las subredes

### 3. Azure Container Registry (`modules/acr/`)

**Propósito**: Crea y configura el registro de contenedores para almacenar imágenes Docker.

**Recursos**:
- `azurerm_container_registry`: Registro de contenedores con SKU configurable

**Características**:
- Nomenclatura: `{name}{environment}` (ej: `acrvingesofttaller2dev`)
- SKU configurable (Basic, Standard, Premium)
- Admin user opcional (por defecto deshabilitado para mayor seguridad)
- Tags para organización

**Outputs**:
- `id`: ID del ACR
- `name`: Nombre del ACR
- `login_server`: URL del servidor de login (ej: `acrvingesofttaller2dev.azurecr.io`)
- `admin_username`: Usuario administrador (sensible)
- `admin_password`: Contraseña administrador (sensible)

### 4. Kubernetes Cluster (`modules/kubernetes-cluster/`)

**Propósito**: Crea y configura el clúster de Kubernetes (AKS) con sus nodos.

**Recursos**:
- `azurerm_kubernetes_cluster`: Clúster AKS principal
- `azurerm_kubernetes_cluster_node_pool`: Pools de nodos adicionales (opcionales)

**Características**:
- **Versión de Kubernetes**: Configurable (por defecto: `1.32`)
- **Node Pool por defecto**: Configurable (nombre, cantidad de nodos, tamaño de VM, tamaño de disco)
- **Node Pools adicionales**: Soporte para múltiples pools de nodos con diferentes configuraciones
- **RBAC**: Habilitado por defecto para seguridad
- **Network Plugin**: Azure CNI con políticas de red de Azure
- **Identity**: SystemAssigned para autenticación automática
- **Lifecycle**: Ignora cambios en `kubernetes_version` y `node_count` del pool por defecto (para permitir actualizaciones manuales)

**Configuración de Red**:
- Network plugin: `azure` (Azure CNI)
- Network policy: `azure`
- Service CIDR: `10.1.0.0/16`
- DNS Service IP: `10.1.0.10`

**Outputs**:
- `name`: Nombre del clúster
- `fqdn`: FQDN del clúster
- `identity`: Identidad del clúster
- `kube_config`: Configuración de Kubernetes (sensible)
- `kube_config_host`: Host del clúster (sensible)
- `cluster_id`: ID del clúster
- `kubelet_identity`: Identidad del kubelet (para asignación de roles ACR)

## Configuración por Entorno

El proyecto soporta múltiples entornos mediante archivos `terraform.tfvars` en la carpeta `environments/`. Cada entorno tiene su propia configuración que sobrescribe los valores por defecto.

### Variables Principales

#### Identificación y Ubicación
- `environment`: Entorno (dev, prod) - validado
- `location`: Región de Azure (por defecto: "Canada Central")
- `resource_group_name`: Nombre base del resource group
- `cluster_name`: Nombre base del clúster AKS
- `dns_prefix`: Prefijo DNS para el clúster

#### Kubernetes
- `kubernetes_version`: Versión de Kubernetes (por defecto: "1.32")
- `default_node_pool`: Configuración del pool de nodos por defecto
  - `name`: Nombre del pool
  - `node_count`: Número de nodos
  - `vm_size`: Tamaño de la VM (ej: "Standard_B2s", "Standard_D2s_v3")
  - `os_disk_size_gb`: Tamaño del disco OS
  - `type`: Tipo de pool (VirtualMachineScaleSets)
  - `max_pods`: Máximo de pods por nodo
- `additional_node_pools`: Pools de nodos adicionales (mapa de objetos)
- `rbac_enabled`: Habilitar RBAC (por defecto: true)

#### Networking
- `vnet_name`: Nombre de la red virtual
- `vnet_address_space`: Espacio de direcciones (por defecto: ["10.0.0.0/16"])
- `subnet_names`: Nombres de las subredes (por defecto: ["aks-subnet", "appgw-subnet"])
- `subnet_prefixes`: Prefijos de las subredes (por defecto: ["10.0.1.0/24", "10.0.2.0/24"])

#### Azure Container Registry
- `acr_name`: Nombre base del ACR (solo alfanumérico)
- `acr_sku`: SKU del ACR (Basic, Standard, Premium)
- `acr_admin_enabled`: Habilitar usuario administrador (por defecto: false)

#### Tags
- `tags`: Mapa de tags para aplicar a todos los recursos

### Ejemplo de Configuración (dev)

```hcl
environment = "dev"
location    = "Canada Central"

resource_group_name = "rg-ecommerce-microservices"
cluster_name        = "aks-ecommerce"
dns_prefix          = "ecommerce-k8s"

kubernetes_version = "1.32"

default_node_pool = {
  name            = "system"
  node_count      = 2
  vm_size         = "Standard_B2s"
  os_disk_size_gb = 30
  type            = "VirtualMachineScaleSets"
  max_pods        = 30
}

additional_node_pools = {}

rbac_enabled = true

tags = {
  Environment = "dev"
  ManagedBy   = "Terraform"
  Project     = "Ecommerce-Microservices"
}
```

## Backend de Terraform (State Remoto)

El proyecto utiliza un backend remoto de Azure Storage para almacenar el estado de Terraform de forma centralizada y segura.

### Configuración del Backend

El backend está configurado en `versions.tf`:

```hcl
backend "azurerm" {
  resource_group_name  = "rg-terraform-state"
  storage_account_name = "stterraformstatetaller2"
  container_name       = "terraform-state"
  key                  = "ecommerce-aks/terraform.tfstate"
}
```

### Setup del Backend

Antes de usar Terraform, es necesario configurar el backend ejecutando:

```bash
./scripts/setup-backend.sh
```

Este script crea:
- Resource Group: `rg-terraform-state`
- Storage Account: `stterraformstatetaller2`
- Container: `terraform-state`

**Nota**: El storage account debe tener un nombre único globalmente. Si el nombre está ocupado, debe modificarse en el script y en `versions.tf`.

## Scripts de Automatización

### 1. `setup-backend.sh`

**Propósito**: Configura el backend de Terraform (Storage Account y Container).

**Uso**:
```bash
./scripts/setup-backend.sh
```

**Funcionalidad**:
- Verifica autenticación de Azure CLI
- Crea Resource Group para el estado
- Crea Storage Account
- Crea Container para el estado

### 2. `init-env.sh`

**Propósito**: Inicializa los archivos de configuración de entorno copiando los ejemplos.

**Uso**:
```bash
./scripts/init-env.sh
```

**Funcionalidad**:
- Copia `terraform.tfvars.example` a `terraform.tfvars` para cada entorno (dev, prod)
- Solo crea archivos si no existen

### 3. `deploy.sh`

**Propósito**: Despliega la infraestructura completa para un entorno específico.

**Uso**:
```bash
./scripts/deploy.sh [dev|prod]
```

**Funcionalidad**:
- Verifica prerequisitos (Terraform, Azure CLI)
- Verifica autenticación de Azure
- Inicializa Terraform
- **Importa recursos existentes** automáticamente si ya existen en Azure:
  - Resource Group
  - Virtual Network
  - Subnets
  - ACR
  - AKS Cluster
- Valida la configuración
- Muestra el plan de cambios
- Aplica los cambios automáticamente
- Muestra los outputs (incluyendo kubeconfig)

**Características especiales**:
- Importación inteligente de recursos existentes para evitar conflictos
- Manejo de errores con mensajes claros
- Outputs con instrucciones para configurar kubectl

### 4. `destroy.sh`

**Propósito**: Destruye toda la infraestructura de un entorno.

**Uso**:
```bash
./scripts/destroy.sh [dev|prod]
```

**Funcionalidad**:
- Verifica prerequisitos
- Muestra el plan de destrucción
- Solicita confirmación explícita (debe escribir "yes")
- Destruye todos los recursos
- Mantiene el estado en el backend (debe eliminarse manualmente si es necesario)

**Advertencia**: Esta operación es **irreversible** y eliminará todos los recursos del entorno.

## Uso y Comandos

### Inicialización Inicial

1. **Configurar el backend** (solo la primera vez):
   ```bash
   cd infra
   ./scripts/setup-backend.sh
   ```

2. **Inicializar archivos de entorno**:
   ```bash
   ./scripts/init-env.sh
   ```

3. **Personalizar configuración**:
   Editar `environments/dev/terraform.tfvars` o `environments/prod/terraform.tfvars` según sea necesario.

### Despliegue

**Desplegar infraestructura para desarrollo**:
```bash
./scripts/deploy.sh dev
```

**Desplegar infraestructura para producción**:
```bash
./scripts/deploy.sh prod
```

### Comandos Terraform Manuales

Si prefieres usar Terraform directamente:

```bash
# Inicializar
terraform init

# Validar configuración
terraform validate

# Ver plan de cambios
terraform plan -var-file="environments/dev/terraform.tfvars"

# Aplicar cambios
terraform apply -var-file="environments/dev/terraform.tfvars"

# Ver outputs
terraform output

# Obtener kubeconfig
terraform output -raw kube_config > ~/.kube/config-aks-dev
export KUBECONFIG=~/.kube/config-aks-dev
```

### Configurar kubectl

Después del despliegue, configura kubectl para conectarte al clúster:

```bash
# Obtener kubeconfig
terraform output -raw kube_config > ~/.kube/config-aks-dev

# Configurar variable de entorno
export KUBECONFIG=~/.kube/config-aks-dev

# Verificar conexión
kubectl get nodes
```

## Outputs Principales

El proyecto expone los siguientes outputs globales:

- `cluster_name`: Nombre del clúster AKS
- `cluster_fqdn`: FQDN del clúster
- `cluster_identity`: Identidad del clúster
- `kube_config`: Configuración de Kubernetes (sensible)
- `resource_group_name`: Nombre del resource group
- `vnet_id`: ID de la red virtual
- `aks_subnet_id`: ID de la subred de AKS
- `acr_name`: Nombre del ACR
- `acr_login_server`: URL del servidor de login del ACR
- `acr_admin_username`: Usuario administrador del ACR (sensible)
- `acr_admin_password`: Contraseña administrador del ACR (sensible)

## Integración AKS-ACR

El proyecto configura automáticamente la integración entre AKS y ACR mediante una asignación de roles de Azure:

```hcl
resource "azurerm_role_assignment" "aks_acr_pull" {
  principal_id                     = module.kubernetes_cluster.kubelet_identity[0].object_id
  role_definition_name             = "AcrPull"
  scope                            = module.acr.id
  skip_service_principal_aad_check = true
}
```

Esto permite que los nodos de AKS extraigan imágenes del ACR sin necesidad de configurar credenciales manualmente.

## Mejores Prácticas Implementadas

### 1. Modularización
- Código organizado en módulos reutilizables
- Separación de responsabilidades por módulo
- Facilita mantenimiento y testing

### 2. Configuración por Entorno
- Archivos `terraform.tfvars` separados por entorno
- Ejemplos proporcionados (`terraform.tfvars.example`)
- Variables con validaciones

### 3. State Remoto
- Backend de Azure Storage para estado centralizado
- Permite trabajo colaborativo
- Protección contra pérdida de estado

### 4. Nomenclatura Consistente
- Patrón: `{nombre}-{environment}` o `{nombre}{environment}`
- Facilita identificación de recursos
- Cumple con convenciones de Azure

### 5. Tags
- Tags aplicados a todos los recursos
- Facilita gestión de costos y organización
- Configurables por entorno

### 6. Lifecycle Policies
- `prevent_destroy` configurado para recursos críticos
- `ignore_changes` para permitir actualizaciones manuales (ej: versión de Kubernetes, node count)
- Protección contra destrucción accidental

### 7. Validaciones
- Validación de variables (entorno, nombres, versiones)
- Mensajes de error claros
- Previene configuraciones inválidas

### 8. Seguridad
- RBAC habilitado en AKS
- Admin user de ACR deshabilitado por defecto
- Integración AKS-ACR mediante Managed Identity
- Outputs sensibles marcados como `sensitive`

### 9. Automatización
- Scripts de despliegue y destrucción
- Importación automática de recursos existentes
- Validación y verificación de prerequisitos

## Conclusión

La implementación de infraestructura como código mediante Terraform proporciona una base sólida y escalable para el despliegue y gestión de la plataforma de microservicios en Azure. La arquitectura modular, la configuración por entorno y los scripts de automatización facilitan el mantenimiento y la evolución de la infraestructura.

Los principales beneficios obtenidos incluyen:

- **Reproducibilidad**: La infraestructura puede ser desplegada de forma idéntica en múltiples entornos, eliminando discrepancias entre desarrollo y producción.
- **Versionado**: Todos los cambios en la infraestructura están versionados en Git, permitiendo rastrear modificaciones y realizar rollbacks cuando sea necesario.
- **Colaboración**: El estado remoto en Azure Storage permite que múltiples desarrolladores trabajen de forma segura sobre la misma infraestructura.
- **Automatización**: Los scripts de despliegue reducen el tiempo y los errores humanos en el proceso de provisionamiento.
- **Seguridad**: La integración automática AKS-ACR mediante Managed Identity elimina la necesidad de gestionar credenciales manualmente, mejorando la postura de seguridad.
- **Flexibilidad**: La configuración por entorno permite adaptar la infraestructura a diferentes necesidades (desarrollo, producción) sin duplicar código.
- **Mantenibilidad**: La modularización del código facilita la actualización y el mantenimiento de componentes individuales sin afectar el resto de la infraestructura.

Esta implementación sigue las mejores prácticas de la industria y proporciona una base confiable para el despliegue continuo de aplicaciones en un entorno de producción.