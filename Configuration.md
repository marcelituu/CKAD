# Configuration - CKAD

## Environment Variables

- Permiten configurar contenedores mediante variables de entorno.
- Se definen en el archivo de especificación del Pod, bajo `env`.
- Ejemplo:
    
    ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: example-pod
    spec:
      containers:
      - name: example-container
        image: nginx
        env:
        - name: EXAMPLE_VAR
          value: "Hello, World!"
    ```
    

---

## ConfigMaps

- Recurso para almacenar datos de configuración en formato clave-valor.
- Se utilizan para desacoplar datos de configuración del contenedor.
- Métodos de uso:
    1. Como variables de entorno.
    2. Montados como volúmenes.
- Crear un ConfigMap:
    
    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: example-configmap
    data:
      key1: value1
      key2: value2
    ```
    
- Usar ConfigMap como variable de entorno:
    
    ```yaml
    env:
    - name: CONFIG_VAR
      valueFrom:
        configMapKeyRef:
          name: example-configmap
          key: key1
    ```
    

---

## Secrets

- Recurso para almacenar datos sensibles (contraseñas, tokens, etc.) en formato codificado en Base64.
- Métodos de uso:
    1. Como variables de entorno.
    2. Montados como volúmenes.
- Crear un Secret:
    
    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: example-secret
    stringData:
      username: admin
      password: securepassword
    ```
    
- Usar Secret como variable de entorno:
    
    ```yaml
    env:
    - name: SECRET_USER
      valueFrom:
        secretKeyRef:
          name: example-secret
          key: username
    ```
    

---

## Service Accounts

- Proveen identidad a los Pods para interactuar con la API de Kubernetes.
- Por defecto, cada Pod usa el ServiceAccount `default`.
- Crear y asociar un ServiceAccount:
    
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: example-sa
    ```
    
    ```yaml
    spec:
      serviceAccountName: example-sa
    ```
    

---

## Resource Requirements

- Permiten definir límites y solicitudes de recursos para Pods y contenedores.
- Campos clave:
    - `requests`: Cantidad mínima garantizada.
    - `limits`: Cantidad máxima permitida.
- Ejemplo:
    
    ```yaml
    resources:
      requests:
        memory: "128Mi"
        cpu: "500m"
      limits:
        memory: "256Mi"
        cpu: "1000m"
    ```
    

---

## Taints and Tolerations

- **Taints**: Restringen qué Pods pueden ejecutarse en un nodo.
- **Tolerations**: Permiten que un Pod se ejecute en un nodo con taints específicos.
- Añadir un taint a un nodo:
    
    ```bash
    kubectl taint nodes node-name key=value:effect
    ```
    
- Definir tolerations en un Pod:
    
    ```yaml
    tolerations:
    - key: "key"
      operator: "Equal"
      value: "value"
      effect: "NoSchedule"
    ```
    

---

## Node Selectors

- Asignan Pods a nodos con base en etiquetas simples.
- Ejemplo:
    
    ```yaml
    spec:
      nodeSelector:
        disktype: ssd
    ```
    

---

## Node Affinity

- Ofrecen una forma más flexible de asignar Pods a nodos.
- Modos:
    - **RequiredDuringSchedulingIgnoredDuringExecution**: Obligatorio.
    - **PreferredDuringSchedulingIgnoredDuringExecution**: Opcional.
- Ejemplo:
    
    ```yaml
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: disktype
              operator: In
              values:
              - ssd
        preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 1
          preference:
            matchExpressions:
            - key: region
              operator: In
              values:
              - us-west
    ```
    