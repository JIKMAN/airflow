# airflow-materials
Materials for the course: The Hands-On Guide



디렉토리 생성

```bash
mkdir docker && cd docker
```

docker-compose.yaml 다운로드

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.1.2/docker-compose.yaml'
```

docker-compse 실행

```bash
docker-compose up airflow-init
docker-compose up -d
```

권한 변경

```bash
curl -LfO 'https://airflow.apache.org/docs/apache-airflow/2.1.2/airflow.sh'
chmod +x airflow.sh
```

