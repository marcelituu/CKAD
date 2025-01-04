
Nos permite hacer un replace rápido, sin tener que hacer delete y apply
```
kubectl replace -f webapp.yaml --force
```

# Comandos útiles para Kubernetes (kubectl)

Este documento contiene una recopilación de comandos interesantes y útiles para trabajar con Kubernetes, organizados por categorías.

---

## Configuración y Contextos

### Ver los contextos disponibles y el contexto actual:

```bash
kubectl config get-contexts
kubectl config current-context
```

### Cambiar de contexto:

```bash
kubectl config use-context <nombre-del-contexto>
```

### Configurar un contexto específico:

```bash
kubectl config set-context <nombre-del-contexto> --namespace=<namespace> --cluster=<cluster> --user=<user>
```

### Exportar configuración a otro archivo kubeconfig:

```bash
kubectl config view --flatten > nuevo-kubeconfig.yaml
```

### Poner un namespace por default
```
kubectl config set-context --current --namespace=mi-namespace
```


---

## Exploración del Cluster

### Obtener nodos:

```bash
kubectl get nodes
```

### Ver detalles de un nodo:

```bash
kubectl describe node <nombre-del-nodo>
```

### Obtener todos los recursos en un namespace:

```bash
kubectl get all -n <namespace>
```

### Obtener todos los namespaces:

```bash
kubectl get namespaces
```

### Ver eventos recientes del cluster:

```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
```

---

## Gestión de Pods

### Listar pods en un namespace:

```bash
kubectl get pods -n <namespace>
```

### Ver detalles de un pod:

```bash
kubectl describe pod <nombre-del-pod> -n <namespace>
```

### Ver logs de un pod:

```bash
kubectl logs <nombre-del-pod> -n <namespace>
```

### Seguir logs de un pod en tiempo real:

```bash
kubectl logs <nombre-del-pod> -n <namespace> -f
```

### Ejecutar un comando en un pod:

```bash
kubectl exec -it <nombre-del-pod> -n <namespace> -- <comando>
```

---

## Gestión de Deployments

### Ver los deployments:

```bash
kubectl get deployments -n <namespace>
```

### Escalar un deployment:

```bash
kubectl scale deployment <nombre-del-deployment> --replicas=<numero> -n <namespace>
```

### Aplicar cambios a un deployment:

```bash
kubectl apply -f <archivo-deployment.yaml>
```

### Eliminar un deployment:

```bash
kubectl delete deployment <nombre-del-deployment> -n <namespace>
```

### Reemplazar un deployment:
```
kubectl replace -f webapp.yaml --force
```

---

## Servicios y Endpoints

### Listar servicios en un namespace:

```bash
kubectl get services -n <namespace>
```

### Describir un servicio:

```bash
kubectl describe service <nombre-del-servicio> -n <namespace>
```

### Ver endpoints asociados a un servicio:

```bash
kubectl get endpoints <nombre-del-servicio> -n <namespace>
```

---

## ConfigMaps y Secretos

### Crear un ConfigMap desde un archivo:

```bash
kubectl create configmap <nombre-del-configmap> --from-file=<ruta-al-archivo>
```

### Ver ConfigMaps en un namespace:

```bash
kubectl get configmaps -n <namespace>
```

### Describir un ConfigMap:

```bash
kubectl describe configmap <nombre-del-configmap> -n <namespace>
```

### Crear un secreto desde un archivo:

```bash
kubectl create secret generic <nombre-del-secreto> --from-file=<ruta-al-archivo>
```

### Ver secretos en un namespace:

```bash
kubectl get secrets -n <namespace>
```

### Decodificar un secreto (por ejemplo, un token):

```bash
kubectl get secret <nombre-del-secreto> -n <namespace> -o jsonpath="{.data.<clave>}" | base64 --decode
```

---

## Debug y Troubleshooting

### Obtener métricas de recursos:

```bash
kubectl top nodes
kubectl top pods -n <namespace>
```

### Hacer debug de un pod con una imagen temporal:

```bash
kubectl run debug-pod --image=<imagen-debug> --restart=Never -it --rm
```

### Ver eventos relacionados con un recurso específico:

```bash
kubectl describe <tipo-de-recurso> <nombre> -n <namespace>
```

### Probar conectividad desde un pod:

```bash
kubectl exec -it <nombre-del-pod> -n <namespace> -- curl <url>
```

---

## YAML y Configuración

### Exportar la definición YAML de un recurso existente:

```bash
kubectl get <tipo-recurso> <nombre> -n <namespace> -o yaml > recurso.yaml
```

### Aplicar un archivo YAML:

```bash
kubectl apply -f <archivo.yaml>
```

### Eliminar un recurso desde YAML:

```bash
kubectl delete -f <archivo.yaml>
```

---

## Alias Útiles

### Crear alias para comandos frecuentes:

En el archivo `~/.bashrc` o `~/.zshrc`, agrega:

```bash
alias k="kubectl"
alias kgp="kubectl get pods"
alias kga="kubectl get all"
alias kgn="kubectl get nodes"
alias kaf="kubectl apply -f"
```

---

## Recursos Adicionales

- [Documentación oficial de Kubernetes](https://kubernetes.io/docs/)
- [Kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
