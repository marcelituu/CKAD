# Volumes and State Persistence in Kubernetes

## Volumes

### 1. **emptyDir**

- **Descripción**: Un volumen temporal que se crea cuando el pod inicia y se borra cuando el pod termina.
- **Uso**: Ideal para almacenar datos temporales que solo necesitan existir mientras el pod está en ejecución.
- **Persistencia**: Se borra junto con el pod.

**Ejemplo de YAML con `emptyDir`:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: example-volume
  volumes:
    - name: example-volume
      emptyDir: {}
```

---

### 2. **hostPath**

- **Descripción**: Monta un directorio o archivo del nodo de Kubernetes en el pod.
- **Uso**: Utilizado en situaciones especiales, como acceso a logs o configuraciones específicas del nodo.
- **Persistencia**: Permanece en el nodo, pero si el pod se mueve a otro nodo, los datos no se transfieren.

**Ejemplo de YAML con `hostPath`:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-example
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: hostpath-volume
  volumes:
    - name: hostpath-volume
      hostPath:
        path: /data          # Directorio en el nodo
        type: Directory       # Tipo de recurso del nodo
```

---

### 3. **Persistent Volume (PV) y Persistent Volume Claim (PVC)**

- **Descripción**: Una solución para almacenamiento persistente gestionado. Los **Persistent Volumes (PV)** son recursos de almacenamiento creados por un administrador, mientras que los **Persistent Volume Claims (PVC)** son solicitudes de los pods para acceder a un PV.
- **Uso**: Ideal para aplicaciones que necesitan almacenamiento persistente y gestionado, como bases de datos.
- **Persistencia**: Persisten aunque el pod se elimine. Son independientes del ciclo de vida del pod.

**Ejemplo de YAML con `Persistent Volume Claim (PVC)`:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
    - name: example-container
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: example-pvc
  volumes:
    - name: example-pvc
      persistentVolumeClaim:
        claimName: example-claim
```

---

## Persistent Volumes (PV)

### Campos principales:

1. **capacity**: Define la capacidad de almacenamiento deseada. Ejemplos: `10Gi`, `20Gi`, `100Mi`.
2. **accessModes**: Indica cómo puede ser accedido el volumen.
    - **ReadWriteOnce (RWO)**: Acceso de lectura/escritura por un solo nodo.
    - **ReadOnlyMany (ROX)**: Acceso de solo lectura desde múltiples nodos.
    - **ReadWriteMany (RWX)**: Acceso de lectura/escritura desde múltiples nodos.
3. **persistentVolumeReclaimPolicy**: Define qué sucede con los datos cuando se elimina el PV.
    - **Retain**: Los datos se retienen.
    - **Recycle**: Los datos se eliminan (para `hostPath` y `nfs`).
    - **Delete**: El PV y los datos se eliminan (aplica a volúmenes de nube como AWS EBS).
4. **storageClassName**: Especifica la clase de almacenamiento para asociarse con el PVC.
5. **volumeMode**:
    - **Filesystem**: Acceso en formato de sistema de archivos.
    - **Block**: Acceso en modo bloque, sin sistema de archivos intermedio.
6. **mountOptions**: Configuración adicional para ciertos tipos de almacenamiento.
7. **nodeAffinity**: Define en qué nodos se puede montar el volumen.

**Ejemplo de YAML con PV y PVC:**

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /data
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: manual
```

---

## Storage Class

### ¿Qué es?

- Las Storage Classes permiten la creación dinámica de recursos de almacenamiento sin necesidad de crear manualmente los PV.
- Se configuran con parámetros específicos del proveedor de almacenamiento (e.g., AWS, Google Cloud, etc.).

**Ejemplo de YAML con Storage Class:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**Ejemplo de YAML con PVC usando Storage Class:**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: example-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard
```

---

## Tipos de volúmenes comunes

### **hostPath**

- **Uso**: Ideal para desarrollo o pruebas, pero no recomendado para producción.

### **gcePersistentDisk**

- **Uso**: Almacenamiento persistente en Google Cloud.

### **awsElasticBlockStore**

- **Uso**: Disco en bloque de Amazon Web Services.

### **nfs**

- **Uso**: Monta un recurso compartido en red.

---

## Mejores prácticas

1. **Seleccionar el tipo de volumen adecuado:**
    - Use `emptyDir` para almacenamiento temporal.
    - Use `hostPath` solo para pruebas locales.
    - Use `PV` y `PVC` para almacenamiento persistente.
2. **Implementar Storage Classes:** Aproveche las capacidades de aprovisionamiento dinámico.
3. **Asegurar el respaldo:** Configure mecanismos de respaldo para volúmenes críticos.
4. **Monitorear el uso del almacenamiento:** Supervise continuamente la capacidad y el estado de los volúmenes.
5. **Planificar la expansión:** Use volúmenes expansibles si es probable que los requisitos de almacenamiento crezcan.