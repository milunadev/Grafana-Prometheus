# Grafana && Prometheus

```bash
minikube addons enable metrics-server

#Agrega el repositorio de Helm de Prometheus:
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

#Instalo Prometheus, tambien creando un namespace
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace

#Instalando Grafana en el namespace ya creado
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana --namespace monitoring

## VALIDANDO REPOS HELM
$ helm repo list
NAME                    URL
eks                     https://aws.github.io/eks-charts
prometheus-community    https://prometheus-community.github.io/helm-charts
grafana                 https://grafana.github.io/helm-charts

#Obtenemos la contrasena de Grafana.
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
TkQTayDFAatZHkcmo6btmeil40TZYhXmHDjfWOKt

#Correr los servicio en modo dettach
kubectl --namespace monitoring port-forward svc/prometheus-server 9090:80 &
kubectl --namespace monitoring port-forward svc/grafana 3000:80 &


## VALIDANDO PODS DEL NAMESPACE MONITORING
$ kubectl get pods -n monitoring
NAME                                                READY   STATUS    RESTARTS   AGE
grafana-56867d47cc-v74g4                            1/1     Running   0          5m31s
prometheus-alertmanager-0                           1/1     Running   0          7m34s
prometheus-kube-state-metrics-6b7d7b9bd9-jnmfr      1/1     Running   0          7m34s
prometheus-prometheus-node-exporter-xkxgk           1/1     Running   0          7m34s
prometheus-prometheus-pushgateway-568fbf799-qps5x   1/1     Running   0          7m34s
prometheus-server-579dc9cfdf-sjmhb                  2/2     Running   0          7m34s
```


```bash 
#CREANDO UN CONFIGMAP
kubectl create configmap grafana-datasources --from-file=datasources.yaml=grafana.yaml -n monitoring

#OBTENER EL YAML DEL DEPLOYMENT DE GRAFANA
kubectl get deployment <nombre-del-deployment-de-grafana> -n monitoring -o yaml > grafana-deployment.yaml

#AGREGAR ESTOS CAMPOS AL DEPLOYMENT DE 
#en volumes
volumes:
      - name: grafana-datasource-config
        configMap:
          name: grafana-datasources
#en volumemounts
volumeMounts:
        - name: grafana-datasource-config
          mountPath: "/etc/grafana/provisioning/datasources"
          readOnly: true

##REAPLICAR EL DEPLOYMENT ACTUALIZADO Y HACER NUEVAMENTE EL PORTFORWARDING
kubectl apply -f grafana-deployment.yaml
kubectl port-forward svc/grafana 3000:80 -n monitoring &
```