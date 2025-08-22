[![Docker Hub - Airflow](https://img.shields.io/badge/Docker%20Hub-ashutro%2Fdataforge--airflow-blue)](https://hub.docker.com/r/ashutro/dataforge-airflow)
[![Docker Hub - Jupyter](https://img.shields.io/badge/Docker%20Hub-ashutro%2Fdataforge--jupyter-blue)](https://hub.docker.com/r/ashutro/dataforge-jupyter)
[![Docker Hub - Spark (Bitnami)](https://img.shields.io/badge/Docker%20Hub-bitnami%2Fspark-orange)](https://hub.docker.com/r/bitnami/spark)

DataForge

Docker-first data engineering stack: Spark, Hadoop (HDFS), Kafka + Zookeeper, Airflow, Postgres, MongoDB, and Jupyter — all wired together with Docker Compose.

⸻

✨ What you get
	•	Spark master/worker with web UIs
	•	HDFS NameNode/DataNode
	•	Kafka + Zookeeper
	•	Airflow (LocalExecutor) pre-wired to Postgres & Spark
	•	Postgres + MongoDB with persistent named volumes
	•	Jupyter (PySpark-ready)

Spark uses the official Bitnami image. Airflow and Jupyter are custom images hosted on Docker Hub under ashutro.

⸻

📦 Docker Hub Images

- `ashutro/dataforge-airflow:1.0` (custom Airflow image)
- `ashutro/dataforge-jupyter:1.0` (custom Jupyter image)
- `bitnami/spark` (official Spark image)

These images extend official bases:
- `ashutro/dataforge-airflow` → based on `apache/airflow:2.9.2` with extra JDK and Python dependencies.
- `ashutro/dataforge-jupyter` → based on `jupyter/base-notebook` (with PySpark support).

This ensures compatibility with official distributions while adding project-specific tooling.

⸻

🧰 Prerequisites
	•	Docker (Docker Desktop on macOS/Windows; Docker Engine on Linux)
	•	Docker Compose v2 (bundled with Docker Desktop; docker compose version should work)
	•	Git (for cloning the repo)
	•	(Optional) Docker Hub account if you plan to push your own images

⸻

🚀 Quick Start (local)

# 1) Clone
git clone https://github.com/ashutro/dataforge.git
cd dataforge

# 2) Create your env file
cp .env.example .env
# Edit .env with your credentials/secrets

# 3) (If not pulling from Docker Hub) Build custom images locally
# Only needed if you haven’t pushed your images yet
docker build -t ashutro/dataforge-airflow:1.0 ./airflow
docker build -t ashutro/dataforge-jupyter:1.0 ./jupyter

# 4) Start the stack
docker compose up -d

# 5) Check containers
docker compose ps


⸻

🌐 Access URLs (localhost)
	•	Airflow Web UI → http://localhost:8082
User/Pass: ${AIRFLOW_ADMIN_USER} / ${AIRFLOW_ADMIN_PASSWORD} (from .env)
	•	Spark Master UI → http://localhost:8080
	•	Spark Worker UI → http://localhost:8081
	•	HDFS NameNode UI → http://localhost:9870
	•	Kafka Broker → localhost:9092 (for host tools on macOS; see notes for Linux/Cloud)
	•	Postgres → localhost:5432  (user: ${POSTGRES_USER}, db: ${POSTGRES_DB})
	•	MongoDB → localhost:27017  (user: ${MONGO_INITDB_ROOT_USERNAME})
	•	Jupyter → http://localhost:8888
First login requires a token printed in logs: docker logs jupyter | grep -m1 token=

⸻

⚙️ Environment Variables

Copy .env.example to .env and edit values:

# Postgres
POSTGRES_USER=your_user
POSTGRES_PASSWORD=your_password
POSTGRES_DB=dataforge

# Mongo
MONGO_INITDB_ROOT_USERNAME=your_mongo_root
MONGO_INITDB_ROOT_PASSWORD=your_mongo_password

# Airflow admin
AIRFLOW_ADMIN_USER=admin
AIRFLOW_ADMIN_PASSWORD=admin
AIRFLOW_ADMIN_FIRSTNAME=Ashutosh
AIRFLOW_ADMIN_LASTNAME=Kumar
AIRFLOW_ADMIN_EMAIL=admin@example.com

docker-compose.yml uses these variables for container env and Airflow user creation.

⸻

🏗 Building & Publishing Images (optional)

You only need to build/push Airflow and Jupyter (Spark uses official Bitnami image):

# Build
docker build -t ashutro/dataforge-airflow:1.0 ./airflow
docker build -t ashutro/dataforge-jupyter:1.0 ./jupyter

# Login & push
docker login
docker push ashutro/dataforge-airflow:1.0
docker push ashutro/dataforge-jupyter:1.0

Once published, anyone can run the stack without building locally.

⸻

📦 Docker Hub Images

The Airflow and Jupyter images are custom builds hosted on Docker Hub under the username ashutro. The Spark image uses the official Bitnami Spark image as its base. You can extend these official bases or use the images directly in your own projects.

🔽 Pulling the images directly:
```bash
docker pull ashutro/dataforge-airflow:1.0
docker pull ashutro/dataforge-jupyter:1.0
docker pull bitnami/spark:3.5.0
```

⸻

🧩 Service-to-Service Connection Details
	•	Spark Master URL (inside containers): spark://spark-master:7077
	•	Kafka bootstrap (inside containers): kafka:9092
	•	Kafka bootstrap (host tools on macOS/Windows): host.docker.internal:9092 (as configured)
For Linux or Cloud, see Kafka notes below.
	•	Postgres DSN: postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
	•	Mongo URI: mongodb://${MONGO_INITDB_ROOT_USERNAME}:${MONGO_INITDB_ROOT_PASSWORD}@mongo:27017
	•	Airflow is configured with AIRFLOW_CONN_SPARK_DEFAULT=spark://spark-master:7077.

⸻

📈 Scaling Spark Workers

docker compose up -d --scale spark-worker=3

View workers in the Spark Master UI (http://localhost:8080).

⸻

☁️ Run on a Cloud VM (Ubuntu example)
	1.	Create a VM (Ubuntu 22.04). Open security group/firewall for the ports you need (or use SSH tunnels below).
	2.	SSH to the VM and install Docker & Compose:

curl -fsSL https://get.docker.com | sh
sudo usermod -aG docker $USER
newgrp docker


	3.	Clone and configure:

git clone https://github.com/ashutro/dataforge.git
cd dataforge
cp .env.example .env
# edit .env


	4.	(Optional) Login & pull your published images:

docker login
docker compose pull


	5.	Start:

docker compose up -d
docker compose ps


	6.	Access UIs securely using SSH tunnels (recommended):

# On your laptop
ssh -L 8082:localhost:8082 -L 8888:localhost:8888 -L 8080:localhost:8080 \
    -L 9870:localhost:9870 -L 9092:localhost:9092 user@your-vm-ip
# Then open http://localhost:8082, http://localhost:8888, etc.

# On Windows PowerShell (using OpenSSH client)
ssh.exe -L 8082:localhost:8082 -L 8888:localhost:8888 -L 8080:localhost:8080 `
    -L 9870:localhost:9870 -L 9092:localhost:9092 user@your-vm-ip
# Then open http://localhost:8082, http://localhost:8888, etc.

# On Windows with PuTTY
You can also use PuTTY if you prefer a GUI:
- Go to Connection > SSH > Tunnels
- Add the same local ports (8082, 8888, 8080, 9870, 9092) mapped to `localhost:PORT`
- Save and open the session to establish tunnels.


If you must expose ports publicly, ensure proper firewall rules and credentials.

⸻

🧠 Kafka on Linux/Cloud: advertised listeners

The compose file sets:

KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://host.docker.internal:9092

	•	macOS/Windows host tools → OK.
	•	Linux host tools → host.docker.internal may not resolve. Use either:
	•	inter-container only: PLAINTEXT://kafka:9092
	•	external clients: PLAINTEXT://<HOST_PUBLIC_IP>:9092
Update kafka service env in docker-compose.yml accordingly when running on Linux or Cloud.

⸻

🧪 Verifying the stack

# Check logs
docker compose logs -f airflow

# Jupyter token
docker logs jupyter | grep -m1 token=

# Postgres connectivity from host
psql postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@localhost:5432/${POSTGRES_DB}


⸻

🛑 Stop / Reset

# Stop but keep data
docker compose down

# Stop AND remove named volumes (data loss for Postgres/Mongo/HDFS!)
docker compose down -v


⸻

🧰 Troubleshooting
	•	Port already in use → Change the host port in docker-compose.yml or free the port.
	•	Apple Silicon  → Hadoop images run under linux/amd64 emulation. Expect overhead; allocate enough RAM/CPU in Docker Desktop.
	•	Airflow user creation repeats → The command tolerates existing user (|| true). Check airflow logs if webserver/scheduler don’t start.
	•	Kafka clients can’t connect → Fix KAFKA_CFG_ADVERTISED_LISTENERS for your environment (see Kafka section).
	•	Permission on Docker socket (Linux) → If Airflow needs Docker access via /var/run/docker.sock, ensure your user is in the docker group, or run with sudo.
	•	Windows WSL2 networking → On Windows with WSL2 backend, `localhost` works for exposed ports. If containers can’t connect to each other via `host.docker.internal`, update the compose file to use service names (e.g., `kafka:9092`) instead.

⸻

🔄 Upgrading & Version pinning
	•	Change image tags in docker-compose.yml (e.g., :1.1 or :3.5.0).
	•	Then:

docker compose pull
docker compose up -d


⸻

📚 Folder structure (suggested)

.
├── docker-compose.yml
├── README.md
├── .env.example  -> copy to .env
├── airflow/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── dags/
│   ├── logs/
│   └── plugins/
└── jupyter/
    ├── Dockerfile
    └── requirements.txt


⸻

📝 Credits
	•	Spark image by Bitnami
	•	Hadoop images by bde2020
	•	Kafka/Zookeeper images by Bitnami
	•	Airflow by Apache Airflow
	•	Jupyter images by Project Jupyter

⸻

📄 License

Choose a license (e.g., MIT) and add a LICENSE file if publishing publicly.
