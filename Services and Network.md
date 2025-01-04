# Services and Networking - CKAD

## Services

### Service Types

#### 1. **ClusterIP**

- **Descripción**: Es el tipo de servicio predeterminado en Kubernetes.
- **Función**: Expone el servicio internamente en el clúster, asignando una IP accesible solo dentro del clúster.
- **Casos de uso**: Ideal para la comunicación interna entre aplicaciones y componentes dentro del clúster.

#### Ejemplo de creación:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: clusterip-example
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: ClusterIP
```

#### 2. **NodePort**

- **Descripción**: Asigna un puerto en cada nodo del clúster.
- **Función**: Permite acceder al servicio externamente a través de la IP del nodo y el puerto específico asignado (en el rango de 30000-32767).
- **Casos de uso**: Útil para pruebas y acceso externo básico, aunque menos flexible y escalable que otros tipos de servicios externos.

#### Ejemplo de creación:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-example
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      nodePort: 30007
  type: NodePort
```

#### 3. **LoadBalancer**

- **Descripción**: Proporciona un balanceador de carga externo que distribuye el tráfico a los _pods_ del servicio.
- **Función**: Asigna automáticamente una IP externa y utiliza el balanceo de carga del proveedor de nube (si está disponible) para dirigir el tráfico a los nodos.
- **Casos de uso**: Ideal para exponer servicios al tráfico de internet en entornos de producción, especialmente en nubes públicas.

#### Ejemplo de creación:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-example
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer
```

---

## Ingress

- Kubernetes no incluye un controlador de Ingress de forma predeterminada.
- Puede elegir entre opciones como **nginx**, **traefik**, **istio**, siendo **nginx** el más común.

#### Ejemplo de creación:

```bash
kubectl create ingress <ingress-name> --rule="host/path=service:port"

kubectl create ingress ingress-test --rule="wear.my-online-store.com/wear*=wear-service:80"
```

---

## Network Policies

### ¿Qué son?

- **Network Policies** permiten controlar el tráfico de red hacia y desde los Pods.
- Se basan en etiquetas para definir reglas que determinan qué tráfico está permitido.

### Componentes principales:

1. **Pod Selector**: Selecciona los Pods a los que se aplican las reglas.
2. **Ingress Rules**: Controlan el tráfico entrante hacia los Pods seleccionados.
3. **Egress Rules**: Controlan el tráfico saliente desde los Pods seleccionados.

#### Ejemplo de una Network Policy:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-nginx
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```

- **Descripción**: Permite que los Pods con la etiqueta `role: frontend` accedan a los Pods con la etiqueta `app: nginx` en el puerto TCP 80.

---

## Developing Network Policies

### Mejores prácticas:

1. **Entienda el tráfico:** Analice los patrones de comunicación de los Pods antes de crear reglas restrictivas.
2. **Empiece con permisos amplios:** Cree reglas abiertas inicialmente y luego restrinja gradualmente.
3. **Etiquetas consistentes:** Asegúrese de que las etiquetas sean claras y bien definidas.
4. **Validación y pruebas:** Pruebe las políticas en un entorno de desarrollo antes de implementarlas en producción.

#### Herramientas útiles:

- **kubectl**: Para inspeccionar y depurar Network Policies.
    
    ```bash
    kubectl describe networkpolicy <policy-name>
    ```
    
- **Calico**: Ofrece soporte avanzado para Network Policies.

---

## Ejemplo completo de Network Policy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: restrict-traffic
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: backend
    ports:
    - protocol: TCP
      port: 443
  egress:
  - to:
    - podSelector:
        matchLabels:
          role: database
    ports:
    - protocol: TCP
      port: 5432
```

- **Descripción**:
    - Permite que los Pods con la etiqueta `role: backend` accedan a `myapp` en el puerto 443.
    - Permite que `myapp` se comunique con Pods etiquetados como `role: database` en el puerto 5432.

---

## Resumen

- **Servicios:** Defina cómo los Pods se exponen internamente y externamente.
- **Ingress:** Gestione el tráfico HTTP/HTTPS hacia los Servicios.
- **Network Policies:** Controle el tráfico entre Pods de forma granular para mejorar la seguridad.
- **Desarrollo:** Adopte un enfoque iterativo para crear y probar políticas de red.