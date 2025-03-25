## 1. Minio Prometheus

#### 1. Install mc 

    curl https://dl.min.io/client/mc/release/linux-amd64/mc --create-dirs -o $HOME/minio-binaries/mc

    chmod +x $HOME/minio-binaries/mc

    export PATH=$PATH:$HOME/minio-binaries/

    mc --help

#### 2. Set minio alias

    bash +o history

    mc alias set myminio http://Minio-IP ACCESS_KEY SECRET KEY

    bash -o history

#### 3. Test connection

    mc admin info myminio

From <https://min.io/docs/minio/linux/reference/minio-mc.html#mc-install>

## 2. Setup prometheus

Prometheus Service
This procedure provides instruction for deploying Prometheus for rapid local evaluation and development. All other environments should have an existing Prometheus or Prometheus-compatible service with access to the MinIO cluster.

### Generate the Bearer Token

MinIO by default requires authentication for requests made to the metrics endpoints. While step is not required for MinIO deployments started with MINIO_PROMETHEUS_AUTH_TYPEset to "public", you can still use the command output for retrieving a Prometheus scrape_configsentry.
Use the mcadminprometheusgeneratecommand to generate a JWT bearer token for use by Prometheus in making authenticated scraping requests:

    mc admin prometheus generate myminio

Replace ALIAS with the alias of the MinIO deployment.

The command returns output similar to the following:

    scrape_configs:
    - job_name: minio-job
    bearer_token: eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJleHAiOjQ4MTg2NDkyODQsImlzcyI6InByb21ldGhldXMiLCJzdWIiOiJyb290dXNlciJ9.SwWPr_foxW0_3o8Sr6UFBdrxPF4zBagXeBpVJ-WibxqHEALtKHf_esxCdkJbz67qii331KSs70drL2xgptgP6A
    metrics_path: /minio/v2/metrics/cluster
    scheme: http
    static_configs:
    - targets: ['10.195.100.130']
    tls_config:
    insecure_skip_verify: true

Note: Change IP to Minio console endpoint
## 3. Add job name above to prometheus-value.yaml file

![Alt text](image.png)

And run

    helm upgrade prometheus-stack prometheus-community/kube-prometheus-stack -n prometheus -f prometheus-values.yaml 

Ref: <https://min.io/docs/minio/linux/operations/monitoring/collect-minio-metrics-using-prometheus.html> 

## 4. Minio Grafana dashboard

https://grafana.com/grafana/dashboards/13502-minio-dashboard/

![Alt text](image-1.png)
