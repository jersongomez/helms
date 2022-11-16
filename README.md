# Helm Chart para microservicios

Este repositorio tiene los helm charts para desplegar aplicaciones dentro de Openshift, configurando el archivo values.yaml puede instalar lo siguiente

- Configmap
- Deployment
- Horizontal Pod Autoscaler
- Ingress
- Route
- Service
- Service Account

## Archivo values

Archivo values.yaml para una aplicación, este ejemplo crea todos los recursos necesarios para desplegar una aplicación en Openshift.

> Recuerde cambiar los valores que requiera personalizar.
```yaml
# Values for app.

nameOverride: consulta-saldo-gateway
fullnameOverride: consulta-saldo-gateway

replicaCount: 1

image:
  repository: quay.io/lcoronad/consulta-saldo-gateway
  pullPolicy: IfNotPresent
  tag: "1.0.1"

imagePullSecrets: []

serviceAccount:
  create: true
  annotations: {}
  name: "consulta-saldo-gateway"

podAnnotations:
  sidecar.istio.io/inject: "true"
  sidecar.istio.io/rewriteAppHTTPProbe: 'true'

podSecurityContext: {}

securityContext: {}

service:
  type: ClusterIP
  port: 8080
  targetPort: http
  protocol: TCP
  name: http

routes:
  - enabled: true
    name: consulta-route
    ## Opcional, si no se especifica Openshift lo genera
    host: consulta-saldo-gateway.apps.cluster-8d6w2.8d6w2.sandbox1784.opentlc.com
    service:
      name: consulta-saldo-gateway
    targetPort: http
    tls:
      enabled: false
      termination: edge
      insecureEdgeTerminationPolicy: Redirect
      key:
      caCertificate:
      certificate:
      destinationCACertificate:

ingress:
  enabled: false
  className: ""
  annotations: {}

environments:
  - name: SPRING_APPLICATION_JSON
    value: '{"server":{"undertow":{"io-threads":10, "worker-threads":20 }}}'
  - name: JAVA_OPTIONS
    value: '-Xms640m -Xmx1024m -Dfile.encoding=ISO-8859-1'

environmentsFrom:
  - configMapRef:
      name: consulta-saldo-gateway-cm
  - secretRef:
      name: consulta-saldo-gateway-sc

resources:
  limits:
    cpu: 400m
    memory: 1280Mi
  requests:
    cpu: 200m
    memory: 896Mi

ports:
  name: http
  port: 8080
  protocol: TCP

probe:
  liveness:
    path: /api/consultas/health-check
    port: http
    failureThreshold: 3
    initialDelaySeconds: 40
    periodSeconds: 20
    successThreshold: 1
    timeoutSeconds: 120
  readiness:
    path: /api/consultas/health-check
    port: http
    failureThreshold: 3
    initialDelaySeconds: 40
    periodSeconds: 20
    successThreshold: 1
    timeoutSeconds: 120

autoscaling:
  enabled: true
  kind: Deployment
  minReplicas: 1
  maxReplicas: 2
  targetCPUUtilizationPercentage: 80
  #targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

configmap:
  name: consulta-saldo-gateway-cm
  data:
    spring.application.name: "Consulta Saldo Gateway"
    spring.mvc.static-path-pattern: /resources/**

secret:
  enabled: true
  name: consulta-saldo-gateway-sc
  data:
    amq.user: YWRtaW4=
    amq.password: YWRtaW4=
```

## Instalación

El siguiente comando despliegua una aplicación en Openshift

- [ ] Clonar este repositorio (solo la primera vez)
```
git clone https://gitlab.consulting.redhat.com/consulting-nola/middleware/xm/devops/helm-charts/applications.git
```

- [ ] Ubicarse en la carpeta del helm chart

```
cd applications/
```

- [ ] Hacer login en el cluster de Openshift donde se desea instalar

```
oc login .......
```

- [ ] Instalar
```
helm template -f values.yaml ./ | oc apply -f -
```

## Author

* **Lázaro Miguel Coronado Torres** - *Architect - lcoronad@redhat.com* 
