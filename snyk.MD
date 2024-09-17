# Despliegue de Jenkins con integración de Snyk usando Helm

## Paso 1: Desplegar Jenkins con Helm

### Agregar el repositorio de Jenkins

```bash
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

Instalar Jenkins
```bash
helm install mi-jenkins jenkins/jenkins
```

Acceder a Jenkins
Obtené la contraseña de admin:

```bash
printf $(kubectl get secret --namespace default mi-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode); echo
```

Reenviar el puerto:

```bash
kubectl port-forward svc/mi-jenkins 8080:8080
```

Paso 2: Configurar las credenciales en Jenkins
En Jenkins, ir a Administrar Jenkins > Administrar Credenciales.
Agregar una nueva credencial de tipo Secret text.
Usar el ID: snyk-token y pegar tu token de API de Snyk.
Paso 3: Actualizar el Pipeline de Jenkins
Agregar los siguientes stages al Jenkinsfile:

Stage: Escaneo de Snyk
```groovy
stage('Snyk Scan') {
    steps {
        withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            sh 'curl -Lo snyk https://static.snyk.io/cli/latest/snyk-linux'
            sh 'chmod +x snyk'
            sh './snyk auth $SNYK_TOKEN'
            sh './snyk test'
        }
    }
}
```

Stage: Análisis de Código con Snyk
```groovy
stage('Snyk Code Analysis') {
    steps {
        withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            sh './snyk code test --severity-threshold=medium'
        }
    }
}
```

Stage: Escaneo de Imágenes de Contenedor con Snyk
```groovy
stage('Snyk Container Scan') {
    steps {
        withCredentials([string(credentialsId: 'snyk-token', variable: 'SNYK_TOKEN')]) {
            sh './snyk container test mi-imagen:latest --file=Dockerfile --severity-threshold=medium'
        }
    }
}
```


Paso 4: Ejecutar y monitorear el Pipeline
Subir los cambios del Jenkinsfile al repositorio.
Ejecutar el pipeline desde Jenkins.
Revisar los resultados en la consola de Jenkins.