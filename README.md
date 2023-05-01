# A Tutorial On Nest.js Metrics - Prometheus & Grafana Tutorial

We look at how to expose metrics for our Nest.js server and get them scraped by Prometheus. We also set up a Grafana server using Helm & Kubernetes to view the metrics.

```bash
nest new nestjs-metrics-tutorial
cd nestjs-metrics-tutorial
npm i @willsoto/nestjs-prometheus prom-client
npm i uuid
npm run start:dev
```

### Dockerization
```dockerfile
FROM node:alpine AS development
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY . . 
RUN npm run build

FROM node:alpine as production
ARG NODE_ENV=production
ENV NODE_ENV=${NODE_ENV}
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install --omit=dev
COPY . .
COPY --from=development /usr/src/app/dist ./dist
CMD ["node", "dist/main"]

```

```bash
docker build -t <remote-tag>
docker push <remote-tag>
```

### Setting up the k8s
```bash
choco install kubernetes-helm
mkdir k8s && cd k8s
helm create nestjs-metrics
cd nestjs-metrics
cd templates
rm -rf tests
kubectl create deployment nestjs-metrics --image={remote-tag} --port 3000 --dry-run=client -o yaml > deployment.yaml

```

Add promethus and grafana dependencies in 'k8s/nestjs-metrics/Chart.yaml'.

```yaml
dependencies:
  - name: prometheus
    version: '15.18.0'
    repository: 'https://prometheus-community.github.io/helm-charts'
  - name: grafana
    version: '6.43.5'
    repository: 'https://grafana.github.io/helm-charts'
```

Update the 'k8s/nestjs-metrics/values.yaml'.

```yaml
prometheus:
  alertmanager:
    enabled: false
  
  pushgateway:
    enabled: false

  nodeExporter:
    enabled: false

grafana:
  service:
    type: NodePort
```

### Run the kubectl

```bash
cd ..
helm dependency update
helm install nestjs-metrics .

kubectl get po
kubectl logs nestjs-metrics-b8896bc77-9dgrw --follow
```

### Run grafana and 
Open the grafana dashboard 'http://localhost:{NodePort}/' and set prometheus as data source. 

```bash
kubectl get service
kubectl get secret nestjs-metrics-grafana -o jsonpath="{.data.admin-password}" | base64 --decode; echo
```

Tutorial: https://www.youtube.com/watch?v=2ESOGJTXv1s
