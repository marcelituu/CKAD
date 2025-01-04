# Observability - CKAD

## Readiness Probe

### ¿Qué es?

- Las **Readiness Probes** determinan si un contenedor está listo para recibir tráfico.
- Un Pod no será considerado listo hasta que la Readiness Probe pase con éxito.
- Ideal para situaciones donde la aplicación necesita inicializarse completamente antes de aceptar solicitudes.

### Tipos de Readiness Probe

#### HTTP Test

```yaml
readinessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 8
```

- **httpGet**: Realiza una solicitud HTTP GET al contenedor.
- Parámetros:
    - `path`: Ruta en el servidor a verificar.
    - `port`: Puerto donde está disponible la aplicación.

#### TCP Test

```yaml
readinessProbe:
  tcpSocket:
    port: 808
```

- **tcpSocket**: Comprueba si el puerto especificado está abierto y acepta conexiones.

#### Exec Test

```yaml
readinessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

- **exec**: Ejecuta un comando dentro del contenedor. Si el comando devuelve `0`, la prueba es exitosa.

### Parámetros Clave

- **`initialDelaySeconds`**: Tiempo en segundos que Kubernetes espera antes de ejecutar la primera prueba.
- **`periodSeconds`**: Intervalo en segundos entre pruebas consecutivas.
- **`failureThreshold`**: Número de fallos consecutivos necesarios para marcar el Pod como no listo.

---

## Liveness Probe

### ¿Qué es?

- Las **Liveness Probes** determinan si un contenedor sigue funcionando.
- Si falla, Kubernetes reiniciará el contenedor.
- Útil para detectar bloqueos o estados irrecuperables de la aplicación.

### Tipos de Liveness Probe

#### HTTP Test

```yaml
livenessProbe:
  httpGet:
    path: /api/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
  failureThreshold: 8
```

- Similar al HTTP Test de Readiness Probe, verifica la disponibilidad de una ruta HTTP.

#### TCP Test

```yaml
livenessProbe:
  tcpSocket:
    port: 808
```

- Verifica que el puerto especificado esté activo.

#### Exec Test

```yaml
livenessProbe:
  exec:
    command:
      - cat
      - /tmp/healthy
```

- Ejecuta un comando en el contenedor para verificar su estado.

### Parámetros Clave

- **`initialDelaySeconds`**: Tiempo en segundos antes de realizar la primera prueba.
- **`periodSeconds`**: Intervalo en segundos entre pruebas consecutivas.
- **`failureThreshold`**: Número de fallos consecutivos antes de reiniciar el contenedor.

---

## Container Logging

### ¿Qué es?

- Los logs son esenciales para depurar y monitorear aplicaciones en Kubernetes.
- Kubernetes captura automáticamente los logs de los contenedores y los expone mediante el comando `kubectl logs`.

### Ejemplo de Uso

- Ver los logs de un contenedor específico:
    
    ```bash
    kubectl logs <pod-name> -c <container-name>
    ```
    
- Seguir los logs en tiempo real:
    
    ```bash
    kubectl logs <pod-name> -c <container-name> --follow
    ```
    

### Mejores Prácticas

1. **Escribir logs en stdout y stderr:** Kubernetes captura estas salidas automáticamente.
2. **Usar herramientas de agregación de logs:** Plataformas como Fluentd, Elasticsearch y Kibana (EFK) para centralizar y analizar logs.

---

## Monitor and Debug Applications

### Monitorización

- Kubernetes expone métricas a través de componentes como:
    - **Metrics Server:** Recolecta métricas básicas de recursos (CPU, memoria).
    - **Prometheus y Grafana:** Solución avanzada para métricas y visualización.

### Comandos Útiles

- Ver el uso de recursos:
    
    ```bash
    kubectl top pods
    kubectl top nodes
    ```
    
- Ver eventos relacionados con un Pod:
    
    ```bash
    kubectl describe pod <pod-name>
    ```
    

### Depuración

1. **Revisar los logs:**
    
    ```bash
    kubectl logs <pod-name>
    ```
    
2. **Ejecutar un shell en el contenedor:**
    
    ```bash
    kubectl exec -it <pod-name> -- /bin/bash
    ```
    
3. **Ver estado del Pod:**
    
    ```bash
    kubectl get pods -o wide
    ```
    
4. **Revisar eventos:**
    
    ```bash
    kubectl get events
    ```
    

---


## Ejemplo Completo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-example
spec:
  containers:
  - name: example-container
    image: nginx
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
      failureThreshold: 3
    livenessProbe:
      exec:
        command:
          - cat
          - /tmp/healthy
      initialDelaySeconds: 10
      periodSeconds: 10
      failureThreshold: 5
```