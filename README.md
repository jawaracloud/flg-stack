# FLG Monitoring Stack
Log Monitoring with Fluent-bit, Loki, Grafana

![The Flow of FLG Stack !](/assets/flow.gif "The Flow of FLG Stack ")

## Create Docker Network and Volume
```bash
docker network create flg-network --driver bridge
docker volume create loki_data --driver local
```

## Run NGINX with Docker Run
```bash
docker run \
    -d \
    --name nginx \
    --network flg-network \
    -v ./nginx_logs:/var/log/nginx \
    -p 8080:80 \
    nginx:1.26.0-alpine
```

## Run Loki with Docker Run
```bash
docker run -d \
  --name loki \
  --restart always \
  -v $(pwd)/loki/loki-config.yml:/etc/loki/loki-config.yaml \
  -v loki_data:/tmp/loki \
  --network flg-stack \
  grafana/loki:k204-4901a5c \
  -config.file=/etc/loki/loki-config.yaml
```

## Run Grafana with Docker Run
```bash
docker run -d \
    --name grafana \
    --restart always \
    -p 3000:3000 \
    -v grafana_data:/var/lib/grafana \
    -v grafana_logs:/var/log/grafana \
    -v $(pwd)/grafana/grafana.ini:/etc/grafana/grafana.ini \
    -v $(pwd)/grafana/datasource.yml:/etc/grafana/provisioning/datasources/datasource.yml \
    --network flg-stack \
    grafana/grafana:11.0.0
```

## Run Fluent-bit with Docker Run
```bash
docker run -d \
    --name fluent_bit \
    --network flg-network \
    --restart always \
    -v $(pwd)/fluent-bit.conf:/fluent-bit/etc/fluent-bit.conf \
    -v $(pwd)/parsers.conf:/fluent-bit/parsers.conf \
    -v $(pwd)/nginx_logs:/var/log/nginx \
    fluent/fluent-bit:3.0.6 \
    fluent-bit \
    -c /fluent-bit/etc/fluent-bit.conf \
    -R /fluent-bit/parsers.conf
```

## Access Grafana
- Open http://localhost:3000 in your browser, login use admin/admin as username/password
- Go to Explore -> label filter change to log and value nginx
- Add operations and select JSON
- You should see logs from NGINX

![The Dashrboad from FLG Stack !](/assets/dashboard.png "The Dashrboad from FLG Stack")
