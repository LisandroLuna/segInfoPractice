# Instalación de Falco con Falcosidekick Web UI

Esta guía proporciona instrucciones paso a paso para instalar Falco con la interfaz web de Falcosidekick utilizando Helm. También incluye una prueba simple para verificar que Falco esté funcionando correctamente.

## Requisitos Previos

- Un clúster de Kubernetes en funcionamiento
- `kubectl` configurado para interactuar con tu clúster
- Helm instalado y configurado

## Pasos de Instalación

### 1. Agregar el repositorio de Helm de Falco

Agregar el repositorio oficial de Helm de Falco a tu configuración de Helm:

```bash
helm repo add falcosecurity https://falcosecurity.github.io/charts
```

2. Actualizar los repositorios de Helm
Actualizar la caché local de los repositorios de Helm:

```bash
helm repo update
```

3. Instalar Falco con Falcosidekick y la Web UI habilitada
Instalar Falco en un nuevo namespace llamado falco, habilitando el TTY, Falcosidekick y la Web UI de Falcosidekick:

```bash
helm install falco falcosecurity/falco --create-namespace --namespace falco --set tty=true --set falcosidekick.enabled=true --set falcosidekick.webui.enabled=true
```

4. Verificar la instalación
Verificar que los pods de Falco y Falcosidekick estén en ejecución:

```bash
kubectl get pods -n falco
```

El resultado esperado es algo similar a lo siguiente:

```sql
NAME                                      READY   STATUS    RESTARTS   AGE
falco-xxxxx                               1/1     Running   0          1m
falco-falcosidekick-xxxxxx                1/1     Running   0          1m
falco-falcosidekick-ui-xxxxxx             1/1     Running   0          1m
```

5. Acceder a la interfaz web de Falcosidekick
Por defecto, la interfaz web de Falcosidekick no está expuesta externamente. Podés acceder utilizando el port forwarding.

Configurar el port forwarding:

```bash
kubectl port-forward service/falco-falcosidekick-ui 2802:2802 -n falco
```

Acceder a la interfaz web:

Abrí tu navegador web y dirigite a:

```arduino
http://localhost:2802
```

Deberías ver el panel de la interfaz web de Falcosidekick.

Pruebas de Falco
Para asegurarse de que Falco está detectando eventos de seguridad correctamente, podés realizar la siguiente prueba.

1. Desplegar un pod de prueba
Crear un pod de prueba utilizando la siguiente definición en YAML:

```yaml
# test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: alpine
      command: ["sleep", "infinity"]
```

Aplicá el YAML para crear el pod:

```bash
kubectl apply -f test-pod.yaml
```

2. Ejecutar un shell dentro del pod de prueba
Ejecutar un shell dentro del contenedor es considerado sospechoso y debería generar una alerta de Falco.

```bash
kubectl exec -it test-pod -- /bin/sh
```

Dentro del shell del contenedor, podés ejecutar cualquier comando o simplemente salir:

```bash
/ # ls
/ # exit
```

3. Verificar las alertas en la interfaz web de Falcosidekick
Volvé a la interfaz web de Falcosidekick y buscá una nueva alerta. Deberías ver un evento similar a:

```less
Se ha iniciado un shell en un contenedor con un terminal adjunto (user=root container=pod://test-pod)
```
Esto confirma que Falco está monitoreando tu clúster y reportando eventos de seguridad.

Limpieza
Eliminar el pod de prueba
```bash
kubectl delete -f test-pod.yaml
```
Desinstalar Falco y Falcosidekick
```bash
helm uninstall falco -n falco
```
Eliminar el namespace de Falco
```bash
kubectl delete namespace falco
```