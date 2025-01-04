# Core Concepts - CKAD

## Docker vs Containerd

### Docker

- Docker es una plataforma para desarrollar, enviar y ejecutar aplicaciones en contenedores.
- Proporciona herramientas para construir imágenes, gestionar contenedores y crear redes y volúmenes.
- Funciona como una capa superior que interactúa con runtimes como `containerd` para gestionar contenedores.
- Incluye:
    - Docker CLI (interfaz de línea de comandos).
    - Docker Daemon (gestiona contenedores).
    - Docker Compose (define aplicaciones multi-contenedor).

### Containerd

- Containerd es un runtime de contenedores de nivel bajo.
- Se encarga de ejecutar y gestionar el ciclo de vida de los contenedores.
- Más ligero y optimizado en comparación con Docker.
- Actúa como base para otros sistemas (Docker lo usa internamente).
- Compatible con Kubernetes, ya que se integra fácilmente como runtime de contenedores.

### Resumen

|Característica|Docker|Containerd|
|---|---|---|
|Complejidad|Alto|Bajo|
|Función principal|Gestión integral|Runtime de contenedores|
|Uso común|Desarrolladores|Kubernetes y sistemas base|

---

## Conceptos Clave de Kubernetes

### Pods

- Son la unidad básica de Kubernetes.
- Representan uno o más contenedores que comparten:
    - Red.
    - Almacenamiento (volúmenes).
- Usados para ejecutar aplicaciones individuales o servicios.
- Características:
    - Siempre se despliegan y gestionan como una unidad.
    - Habitualmente contienen un solo contenedor principal (pueden incluir sidecars).

### ReplicaSets

- Garantizan un número deseado de réplicas de un Pod en ejecución.
- Supervisan y reemplazan Pods fallidos para mantener el estado deseado.
- Declarados en archivos YAML bajo la clave `spec.replicas`.
- Ejemplo básico:
    
    ```yaml
    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: example-replicaset
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: example
      template:
        metadata:
          labels:
            app: example
        spec:
          containers:
          - name: example-container
            image: nginx
    ```
    

### Deployments

- Se utilizan para gestionar ReplicaSets y permitir actualizaciones declarativas.
- Ofrecen funcionalidades avanzadas como:
    - Rollouts controlados.
    - Rollbacks automáticos en caso de error.
- Incluyen configuraciones de escalado, estrategia de actualización y más.
- Ejemplo básico:
    
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: example-deployment
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: example
      template:
        metadata:
          labels:
            app: example
        spec:
          containers:
          - name: example-container
            image: nginx
    ```
    

### Namespaces

- Proporcionan aislamiento lógico dentro de un clúster de Kubernetes.
- Útiles para:
    - Organizar recursos en equipos o proyectos.
    - Aplicar políticas de acceso o cuotas de recursos específicas.
- Kubernetes incluye namespaces predefinidos:
    - `default`: Espacio por defecto para recursos sin namespace especificado.
    - `kube-system`: Contiene recursos internos del clúster.
    - `kube-public`: Accesible públicamente.
    - `kube-node-lease`: Para información de nodos.
- Crear un namespace:
    
    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: example-namespace
    ```