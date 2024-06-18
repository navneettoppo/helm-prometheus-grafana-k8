# helm-prometheus-grafana-Otelk8

#Prometheus Installation using helm-charts
```
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm repo update
  helm install prometheus prometheus-community/prometheus -n monitoring
  
  kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=prometheus-server-ext -n monitoring
  kubectl get service
```

#Grafana Installation using helm-charts
```
  helm repo add grafana https://grafana.github.io/helm-charts 
  helm repo update
  helm install grafana grafana/grafana -n monitoring


	Get your 'admin' user password by running:
	kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

  kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext -n monitoring
  kubectl get service
  
  kubectl get secret --namespace default grafana -o jsonpath="{.data.admin-password}" | ForEach-Object { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) }
```

#Opentelementry Installation with helm-chart
  ```
  helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
  helm repo update
  helm install my-opentelemetry-collector open-telemetry/opentelemetry-collector --set image.repository="otel/opentelemetry-collector-k8s" -n monitoring
  helm install my-otel-demo open-telemetry/opentelemetry-demo
  kubectl port-forward svc/my-otel-demo-frontendproxy 8080:8080

  ```

# Setting Up Prometheus and Grafana with Persistent Storage on Kubernetes

Create Persistent Volume for Prometheus
Create a file `pv-prom.yaml`:

  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: prometheus-pv
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: local-storage
    local:
      path: /prometheus-data
    nodeAffinity:
      required:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - <NODE_PROMETHEUS_RUNS>
  ```
  ```
``kubectl apply -f pv-prom.yaml
  ```

Create Persistent Volume Claim for Prometheus
Create a file `pvc-prom.yaml`:
  ```
  apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: prometheus-pvc
spec:
  storageClassName: local-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

  ```
  ```
  kubectl apply -f pvc-prom.yaml
  ```

Create Persistent Volume for Grafana
Create a file `pv-grafana.yaml`:
  ```
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: grafana-pv
  spec:
    capacity:
      storage: 5Gi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: local-storage
    local:
      path: /grafana-data
    nodeAffinity:
      required:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                  - <NODE_GRAFANA_RUNS>

  ```
  ```
  kubectl apply -f pv-grafana.yaml
  ```

Create Persistent Volume Claim for Grafana
Create a file pvc-grafana.yaml:
  ```
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: grafana-pvc
  spec:
    storageClassName: local-storage
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 5Gi
  ```
  ```
  kubectl apply -f pvc-grafana.yaml
  ```

Update Prometheus Deployment
Update Prometheus to use persistent storage:

  ```
  helm upgrade prometheus prometheus-community/prometheus \
    --set server.persistentVolume.enabled=true \
    --set server.persistentVolume.storageClass=local-storage \
    --set server.persistentVolume.existingClaim=prometheus-pvc
  ```
Update Grafana Deployment
Update Grafana to use persistent storage:
  ```
  helm upgrade grafana grafana/grafana \
    --set persistence.enabled=true \
    --set persistence.storageClassName="local-storage" \
    --set persistence.existingClaim="grafana-pvc"
  ```

Make sure to replace <NODE_PROMETHEUS_RUNS> and <NODE_GRAFANA_RUNS> with the appropriate node names where Prometheus and Grafana will be running.


