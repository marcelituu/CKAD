# Multi-Container Pods - CKAD

## Multi-Container Pods

- Los Pods en Kubernetes pueden contener múltiples contenedores que trabajan juntos.
- Motivos comunes para usar Multi-Container Pods:
    - **Sidecar**: Contenedor que proporciona soporte o funcionalidades complementarias.
    - **Adapter**: Convierte la salida de un contenedor a un formato usable por otro.
    - **Ambassador**: Proporciona acceso a servicios externos.

### Comunicación entre contenedores

- Los contenedores dentro de un Pod comparten:
    - **Red**: Se comunican usando `localhost` y puertos internos.
    - **Almacenamiento**: Pueden montar los mismos volúmenes.

### Ejemplo de Multi-Container Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  containers:
  - name: main-container
    image: nginx
    ports:
    - containerPort: 80
  - name: sidecar-container
    image: busybox
    command: ["/bin/sh", "-c", "while true; do echo Sidecar running; sleep 30; done;"]
```

---

## Init Containers

- Contenedores especiales que se ejecutan **antes** de que inicien los contenedores principales.
- Usos comunes:
    - Preparar datos o dependencias.
    - Verificar condiciones previas al inicio del Pod.

### Características

- Se ejecutan de forma secuencial.
- Deben completarse exitosamente antes de iniciar los contenedores principales.

### Ejemplo de Pod con Init Container

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-with-init-container
spec:
  initContainers:
  - name: init-container
    image: busybox
    command: ["/bin/sh", "-c", "echo Preparing environment; sleep 10;"]
  containers:
  - name: main-container
    image: nginx
    ports:
    - containerPort: 80
```

### Notas sobre Init Containers

- No comparten el ciclo de vida con los contenedores principales.
- Pueden compartir volúmenes con los contenedores principales para intercambiar datos.
- Si un Init Container falla, el Pod intenta reiniciarlo hasta que tenga éxito o exceda el límite de intentos.

---