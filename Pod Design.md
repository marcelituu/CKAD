# Pod Design - CKAD

## Labels, Selectors, and Annotations

### Labels

- Son pares clave-valor que se asignan a los objetos de Kubernetes (Pods, Services, Deployments, etc.).
- Facilitan la organización y selección de recursos.

#### Ejemplo:

```yaml
metadata:
  labels:
    app: frontend
    environment: production
```

### Selectors

- Permiten seleccionar objetos basados en etiquetas.
- Usados principalmente en Services, ReplicaSets y Deployments.

#### Ejemplo:

```yaml
spec:
  selector:
    matchLabels:
      app: frontend
```

### Annotations

- Almacenan metadatos adicionales sobre un objeto en forma de pares clave-valor.
- No se usan para selección, sino para proporcionar información adicional.

#### Ejemplo:

```yaml
metadata:
  annotations:
    description: "Este es un frontend de producción."
```

---

## Rolling Updates & Rollbacks in Deployments

### Rolling Updates

- Permiten actualizar las aplicaciones sin tiempo de inactividad.
- Gradualmente reemplazan Pods antiguos con nuevos.

#### Ejemplo:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

- **`maxUnavailable`**: Número máximo de Pods que pueden no estar disponibles durante la actualización.
- **`maxSurge`**: Número máximo de Pods adicionales permitidos durante la actualización.

1. Recreate -> Elimina todos los pods actuales, y levanta unos nuevos
2. Rolling Update -> Se levanta la nueva versión junto a la antigua para evitar que haya corte de servicio (Defualt)

### Rollbacks

- Kubernetes permite volver a una versión anterior si la actualización falla.

#### Comando:

```bash
kubectl rollout undo deployment <deployment-name>
```

---

## Deployment Strategies

### Blue-Green Deployment

- Mantiene dos entornos separados: "blue" (producción actual) y "green" (nueva versión).
- El tráfico se redirige al entorno "green" una vez validado.

#### Pasos:

1. Despliegue la nueva versión en el entorno "green".
2. Verifique que funciona correctamente.
3. Actualice el Service para apuntar al entorno "green".

### Canary Deployment

- Despliega la nueva versión a un subconjunto pequeño de usuarios antes de implementarla completamente.
- Permite validar la nueva versión con menor impacto.

#### Ejemplo:

```yaml
spec:
  replicas: 10
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:canary
```

- Aumente gradualmente el número de réplicas si no hay problemas detectados.

---

## Jobs

### ¿Qué son?

- Ejecutan tareas únicas (one-time tasks) y aseguran su finalización exitosa.

#### Ejemplo:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: example
        image: busybox
        command: ["/bin/sh", "-c", "echo Hello, Kubernetes"]
      restartPolicy: Never
  backoffLimit: 4
```

- **`restartPolicy`**: Debe ser `Never` o `OnFailure`.
- **`backoffLimit`**: Número de intentos antes de que el Job falle.

---

## CronJobs

### ¿Qué son?

- Programan tareas recurrentes utilizando una sintaxis cron.

#### Ejemplo:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: example-cronjob
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: example
            image: busybox
            command: ["/bin/sh", "-c", "echo Hello, Kubernetes"]
          restartPolicy: OnFailure
```

- **`schedule`**: Sintaxis cron para definir la frecuencia de ejecución.
    - Ejemplo: `*/5 * * * *` ejecuta cada 5 minutos.
- **`jobTemplate`**: Define la plantilla del Job que se ejecutará.

---

## Mejores Prácticas

1. **Uso consistente de etiquetas:** Ayuda a organizar y rastrear los recursos fácilmente.
2. **Revisar estrategias de despliegue:** Use Rolling Updates por defecto y estrategias avanzadas como Blue-Green o Canary para despliegues críticos.
3. **Monitorear Jobs y CronJobs:** Verifique el estado regularmente para asegurarse de que se completen correctamente.
4. **Automatizar tareas repetitivas:** Utilice CronJobs para minimizar la intervención manual en tareas programadas.