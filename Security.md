# Seguridad en Kubernetes

## Autenticación

La autenticación responde a la pregunta: **¿Quién puede acceder al clúster?** Kubernetes no administra usuarios directamente, pero ofrece métodos para autenticar tanto a usuarios como a aplicaciones mediante cuentas de servicio (**Service Accounts**).

### Métodos de autenticación

#### Básica (**BASIC AUTH**)

- Requiere un archivo `.csv` que contiene tres columnas: nombre de usuario, contraseña y un ID único.
- Ejemplo de configuración:
    
    ```bash
    --basic-auth-file=user.csv
    ```
    
- Esta opción está **deprecada** desde la versión 1.19 y no se recomienda en entornos de producción.

#### Token

- Similar al método básico, pero en lugar de contraseñas, utiliza tokens.
- Configuración:
    
    ```bash
    --token-auth-file=user-token.csv
    ```
    
- No recomendado por razones de seguridad.

#### Cuentas de servicio (**Service Accounts**)

- Usadas para autenticar aplicaciones dentro del clúster.
- Se configuran automáticamente al crear pods, a menos que se especifique lo contrario.

#### KubeConfig

- Archivo predeterminado: `$HOME/.kube/config`.
    
- Contiene información sobre:
    
    - **Clusters:** Configuración del clúster.
    - **Contexts:** Contextos para cambiar entre múltiples configuraciones.
    - **Users:** Configuración del usuario, como certificados o tokens.
    
    Ejemplo de configuración de certificados:
    
    ```yaml
    certificate-authority: /etc/kubernetes/pki/ca.crt
    # O como base64:
    certificate-authority-data: <certificado_base64>
    ```
    

### Acceso a la API

- La API de Kubernetes organiza sus endpoints en varios grupos:
    
    - `/metrics`, `/healthz`, `/version`, `/api`, `/apis`, `/logs`.
- Ejemplo para verificar la versión de la API:
    
    ```bash
    curl https://kube-master:6443/version
    ```
    
- Para acceder sin credenciales explícitas, puede iniciarse un proxy:
    
    ```bash
    kubectl proxy
    ```
    
    Esto permite realizar solicitudes a través de `localhost:8001` utilizando el archivo `.kube/config`.
    

---

## Autorización

La autorización define **qué acciones puede realizar un usuario o aplicación dentro del clúster**.

### Modos de autorización

#### Node Authorizer

- Se aplica automáticamente a nodos del clúster.
- Permite acciones básicas a miembros del grupo `system:node`.

#### ABAC (**Attribute-Based Access Control**)

- Define políticas manualmente en un archivo JSON.
- Ejemplo:
    
    ```json
    {
      "kind": "Policy",
      "spec": {
        "user": "dev-user",
        "namespace": "*",
        "resource": "pods",
        "apiGroup": "x"
      }
    }
    ```
    

#### RBAC (**Role-Based Access Control**)

- Permite definir roles y asignarlos a usuarios o grupos.
    
- Ejemplo de definición de rol:
    
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: developer
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["list", "get", "create", "update", "delete"]
    ```
    
    Ejemplo de RoleBinding:
    
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: developer-binding
      namespace: default
    subjects:
      - kind: User
        name: usuario-ejemplo
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: developer
      apiGroup: rbac.authorization.k8s.io
    ```
    

#### ClusterRoles

- Roles aplicados a nivel de clúster.
    
- Ejemplo:
    
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: cluster-administrator
    rules:
      - apiGroups: [""]
        resources: ["nodes"]
        verbs: ["list", "get", "create"]
    ```
    
    Ejemplo de ClusterRoleBinding:
    
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: cluster-administrator-binding
    subjects:
      - kind: User
        name: usuario-ejemplo
        apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: cluster-administrator
      apiGroup: rbac.authorization.k8s.io
    ```
    

#### Webhook

- Permite delegar la autorización a servicios externos.

#### AlwaysAllow y AlwaysDeny

- Modos menos seguros que permiten (o niegan) todas las solicitudes.

### Verificación de permisos

- Para comprobar qué acciones puede realizar un usuario:
    
    ```bash
    kubectl auth can-i delete nodes
    kubectl auth can-i create pods --as dev-user -n test
    ```
    

---

## Admission Controllers

Los **Admission Controllers** permiten modificar o validar solicitudes antes de que lleguen al API server.

### Tipos de Admission Controllers

#### Validating Admission Controllers

- Validan las solicitudes y las rechazan si no cumplen con las políticas.
- Ejemplo: Comprobar que un namespace existe antes de crear un pod.

#### Mutating Admission Controllers

- Modifican las solicitudes antes de procesarlas.
- Ejemplo: Agregar un `storageClass` predeterminado si no está definido en un PersistentVolumeClaim.

### Webhooks de Admission Controllers

- Se utilizan para implementar controladores personalizados.
- Ejemplo de MutatingWebhookConfiguration:
    
    ```yaml
    apiVersion: admissionregistration.k8s.io/v1
    kind: MutatingWebhookConfiguration
    metadata:
      name: pod-label-mutation
    webhooks:
      - name: pod-labels-mutation.webhook.com
        clientConfig:
          service:
            name: webhook-server
            namespace: default
            path: /mutate
          caBundle: <Base64 del cert.pem>
        rules:
          - operations: ["CREATE"]
            apiGroups: [""]
            apiVersions: ["v1"]
            resources: ["pods"]
        admissionReviewVersions: ["v1"]
        sideEffects: None
    ```
    

---

## Custom Resource Definitions (CRD)

Los **CRD** permiten extender la API de Kubernetes con nuevos tipos de recursos personalizados.

### Ejemplo de definición de un CRD:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.mycompany.com
spec:
  group: mycompany.com
  names:
    kind: Database
    listKind: DatabaseList
    plural: databases
    singular: database
    shortNames:
      - db
  scope: Namespaced
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                engine:
                  type: string
                  enum:
                    - mysql
                    - postgres
                version:
                  type: string
                storage:
                  type: integer
                  minimum: 1
```

Ahora podría desplegar objetos del tipo Database y Kubernetes lo entendería:
```yaml
apiVersion: mycompany.com/v1
kind: Database
metadata:
  name: my-database
spec:
  engine: postgres
  version: "14"
  storage: 10
```
### Custom Controllers

- Complementan los CRD implementando la lógica para gestionar los nuevos recursos.
- Usualmente programados en Go. Ejemplo: [https://github.com/kubernetes/sample-controller](https://github.com/kubernetes/sample-controller)

### Operators

- Extensiones avanzadas que utilizan CRDs y controladores personalizados para automatizar la gestión de aplicaciones complejas.
- Repositorio de Operators: [OperatorHub](https://operatorhub.io/)
