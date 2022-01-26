Componentes básicos Kubernetes (k8s)
===

[TOC]

## Componentes básicos
1. **Pod**: Es un grupo de uno o más contenedores que compartes red, almacenamiento y las especificaciones de como desplegar los contenedores(docker, containerd, etc)
1. **Service**: Es utilizado para exponer una App dentro de un contenedor como servicio de red, asignadole una IP privado y/o pública.
1. **Volume**: Ya que los pods por defecto son efimeros, necesitarias un volumen si quieres tener data persistente o compartir archivos entre distintos pods.
1. **Namespace**: Es un espacio virtual que te permite separar lógicamente distintas App o proyecto dentro de un mismo clúster físico. Además te permite organizar y administrar permisos.


## Deployment manifiest
```yaml=
apiVersion: apps/v1
kind: Deployment
metadata:
  name: helloweb
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
      tier: web
  template:
    metadata:
      labels:
        app: hello
        tier: web
    spec:
      containers:
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 200m
```

##### Explicación:
- `apiVersion` Define la versión de la API de k8s que se usará, por ello es importante saber que versión tiene tu cluster de k8s.
- `kind` Define el tipo de servicio que va a crear este manifiesto.
- `metadata` se usa para indicar nombre y etiquetas a tus deployments y así poder referenciarlos en otros componentes.
- `spec.replicas` indica cuantas replicas del mismo Pod se levantaran.
- `spec.selector` Define cómo el Deployment identifica los Pods que debe gestionar.
- `spec.template` este campo contiene los siguientes sub-campos:
    - Los Pods se etiquetaran con `app: hello` y `tier: web` usando el campo labels.
    - `spec.containers.name` indica que se ejecutara un contenedor con el nombre `hello-app`
    - `spec.containers.image` indica que imagen se usara para levantar ese contenedor.
    - `spec.containers.ports.containerPort` Abre el puerto 8080 para recibir y enviar tráfico 

##### Desplegar deployment
```
kubectl create namespace web
kubectl apply -f deploy.yaml -n web
```

Para comprobar que se haya desplegado correctamente usamos:
```
kubectl get deploy -n web
```
*output*:
```
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE 
hello-app-deployment   3/3     3            3           1s  
```

Cuando inspeccionas los Deployments de tu clúster, se muestran los siguientes campos:

`NAME` enumera los nombre de los Deployments del clúster.
`READY` muestra cuántas réplicas de la aplicación están disponibles para sus usuarios. Sigue el patrón número de réplicas listas/deseadas.
`UP-TO-DATE` muestra el número de réplicas que se ha actualizado para alcanzar el estado deseado.
`AVAILABLE` muestra cuántas réplicas de la aplicación están disponibles para los usuarios.
`AGE` muestra la cantidad de tiempo que la aplicación lleva ejecutándose.

Usa este comando `kubectl describe deployments <name>` para ver la descripción completa del deployment creado.

##### Actualizar un deployment
```kubectl --record deploy/helloweb set image deployment.v1.apps/helloweb hello-app=nginx:1.9.1 -n web```

Este comando permite editar la imagen docker que utilizar el Pod en tiempo real. Esto cambiara la imagen, desplegara unos nuevos pods y por último eliminara los pods actuales.

Para verficar el cambio podemos ejecutar: `kubectl describe deployments helloweb -n web` o `kubectl describe pod helloweb-xxxx -n web`

##### Diagrama arquitectura

![](https://i.imgur.com/Cg2Egrc.png)
