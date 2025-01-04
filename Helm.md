# Helm

## Introduction to Helm

Helm es un gestor de paquetes para Kubernetes que simplifica la instalación, actualización y gestión de aplicaciones en un clúster. Es similar a herramientas como apt en Ubuntu o yum en CentOS, pero diseñado específicamente para Kubernetes.

### Ventajas de usar Helm:

- **Gestión simplificada de aplicaciones**: Permite instalar y configurar aplicaciones complejas con un solo comando.
- **Reusabilidad**: Los "charts" de Helm permiten reutilizar configuraciones para despliegues similares.
- **Versionamiento**: Gestiona diferentes versiones de aplicaciones y configuraciones fácilmente.
- **Rollback**: Facilita el rollback a versiones anteriores en caso de problemas.

---

## Installing Helm

### Prerrequisitos:

- Un clúster de Kubernetes funcionando.
- Acceso al comando `kubectl` configurado para comunicarse con el clúster.

### Pasos para instalar Helm:

1. **Descargar Helm**
    
    - Desde la [página oficial de Helm](https://helm.sh/docs/intro/install/), selecciona el paquete adecuado para tu sistema operativo.
2. **Validar la instalación** Ejecuta el siguiente comando para verificar que Helm está correctamente instalado:
    
    ```bash
    helm version
    ```
    
    Deberías ver una salida similar a:
    
    ```
    version.BuildInfo{Version:"v3.x.x", GitCommit:"xxxxx", ...}
    ```
    
3. **Configurar Helm (opcional)**
    
    - Helm no requiere instalar un componente adicional en el clúster (a diferencia de versiones anteriores que requerían Tiller).
    - Si necesitas configurar repositorios de charts adicionales, puedes usar:
        
        ```bash
        helm repo add <nombre-repo> <URL>
        ```
        
    - Por ejemplo, para agregar el repositorio oficial de Helm:
        
        ```bash
        helm repo add stable https://charts.helm.sh/stable
        ```
        
4. **Actualizar Repositorios** Asegúrate de tener los charts más recientes:
    
    ```bash
    helm repo update
    ```

Con el comando `helm env` nos saca la información de la configuración del usuario

---

## Helm Concepts

### Chart

Un chart es un paquete de Helm que contiene todos los recursos necesarios para desplegar una aplicación en Kubernetes.

- **Estructura de un chart**:
    
    ```
    mychart/
    │   Chart.yaml         # Metadatos del chart (nombre, versión, descripción, etc.)
    │   values.yaml        # Valores por defecto para las configuraciones
    │   templates/         # Plantillas de Kubernetes que Helm renderiza
    │   charts/            # Charts dependientes
    │   README.md          # Documentación opcional
    ```
    

### Release

Una release es una instancia instalada de un chart en un clúster de Kubernetes.

- Puedes tener múltiples releases de un mismo chart con diferentes configuraciones.
- Comandos clave:
    - **Instalar**: `helm install <release-name> <chart>`
    - **Listar**: `helm list`
    - **Eliminar**: `helm uninstall <release-name>`

### Repository

Un repositorio de Helm es una colección de charts empaquetados y versionados.


- **Comandos útiles**:
    - Agregar un repositorio: `helm repo add <nombre> <URL>`
    - Listar repositorios: `helm repo list`
    - Buscar charts: `helm search repo <chart-name>`

### Values

Los "values" en Helm son configuraciones que se pueden personalizar al instalar un chart.

- **Archivo de valores (`values.yaml`)**:
    - Define configuraciones por defecto que se pueden sobrescribir.
- **Sobrescribir valores al instalar**:
    
    ```bash
    helm install <release-name> <chart> --set <key>=<value>
    ```
    
    - O utilizando un archivo:
    
    ```bash
    helm install <release-name> <chart> -f custom-values.yaml
    ```

Ejemplo
```shell
helm install release-1 bitnami/wordpress -f values-pre.yaml
```


### Template

Los templates son archivos YAML en el directorio `templates/` de un chart. Helm utiliza estos templates para generar manifiestos de Kubernetes basados en los "values" proporcionados.

- **Sintaxis**: Usa Go templates para generar recursos dinámicos.
- **Ejemplo básico de un template**:
    
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: {{ .Release.Name }}-config
    data:
      key: {{ .Values.configKey }}
    ```
    

---

Con estos conceptos y pasos básicos, estás listo para empezar a usar Helm en tu clúster de Kubernetes. Puedes continuar explorando temas como **subcharts**, **hooks**, y cómo crear tus propios charts personalizados para aplicaciones más avanzadas.

## Helm commands
```bash
helm list # lista las instalaciones actuales con helm
```

Este comando nos permite descarganos toda la definición del chart sin instalarlo, útil por si queremos ver como funciona por dentro o incluso tocar algun archivo
```bash
helm pull --untar bitnami/wordpress
```
