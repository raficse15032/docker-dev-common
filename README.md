# Docker Development Common

This repository contains Docker Compose configurations for common development services with organized environment configurations.

## 🏗️ Project Structure

```
docker-dev-common/
├── docker-compose.yml              # Base configuration with network
├── docker-compose.mysql.yml        # MySQL database service
├── docker-compose.adminer-mysql.yml # Adminer database management
├── docker-compose.kafka.yml        # Apache Kafka with KRaft mode
├── docker-compose.override.yml     # Development overrides (optional)
├── .env                            # Main environment variables
├── .gitignore                      # Git ignore rules
├── kafka/
│   ├── .env                        # Kafka-specific environment variables
│   └── .env.example               # Kafka environment template
├── mysql/
│   ├── .env                        # MySQL-specific environment variables
│   └── .env.example               # MySQL environment template
└── README.md                       # This file
```

## 🚀 Services Available

### MySQL Database
- **File**: `docker-compose.mysql.yml`
- **Image**: `mysql:8.0.43-debian`
- **Port**: 3306 (configurable via `MYSQL_PUBLISH_PORT`)
- **Environment**: `mysql/.env`
- **Features**:
  - Persistent data volume
  - Custom database and user configuration
  - UTF8MB4 character set support

### Adminer (Database Management)
- **File**: `docker-compose.adminer-mysql.yml`
- **Image**: `adminer:5.3.0`
- **Port**: 8081 (configurable via `ADMINER_PUBLISH_PORT`)
- **Environment**: `mysql/.env`
- **Features**:
  - Web-based database management
  - Auto-connects to MySQL
  - Custom theme support

### Apache Kafka with KRaft
- **File**: `docker-compose.kafka.yml`
- **Image**: `apache/kafka:4.0.1-rc0`
- **Environment**: `kafka/.env`
- **Ports**:
  - Kafka: 9092 (configurable via `KAFKA_PUBLISH_PORT`)
  - Controller: 9093 (configurable via `KAFKA_CONTROLLER_PORT`)
  - Kafka UI: 8080 (configurable via `KAFKA_UI_PORT`)
- **Features**:
  - Official Apache Kafka image
  - KRaft mode (no ZooKeeper required)
  - Kafka UI for topic management and monitoring
  - Auto topic creation enabled
  - Health checks configured
  - Performance-optimized JVM settings

## ⚙️ Setup

### 1. Create the external network
```bash
docker network create docker-dev-common
```

### 2. Set up environment files
```bash
# Copy main environment file
cp .env.example .env

# Copy service-specific environment files
cp kafka/.env.example kafka/.env
cp mysql/.env.example mysql/.env
```

### 3. Configure environment variables
Edit the environment files according to your needs:
- `.env` - Main configuration and port mappings
- `kafka/.env` - Kafka-specific settings
- `mysql/.env` - MySQL and Adminer settings

## 🎯 Usage

### Quick Start (All Services)
```bash
# Using the COMPOSE_FILE variable from .env
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs
```

### Individual Services

**MySQL only**:
```bash
docker compose up -d mysql
```

**MySQL + Adminer**:
```bash
docker compose up -d mysql adminer
```

**Kafka only**:
```bash
docker compose up -d kafka
```

### Stop Services
```bash
# Stop all services
docker compose down

# Stop and remove volumes
docker compose down -v
```

## 🌐 Service Access

| Service | URL | Default Port |
|---------|-----|--------------|
| MySQL | `localhost:3306` | 3306 |
| Adminer | http://localhost:8081 | 8081 |
| Kafka | `localhost:9092` | 9092 |
| Kafka UI | http://localhost:8080 | 8080 |

## 📊 Kafka Usage

### Access Kafka UI
Open http://localhost:8080 to access the Kafka UI for:
- Topic management and creation
- Message browsing and publishing
- Consumer group monitoring
- Broker information and metrics
- Cluster overview

### Connect to Kafka
- **External**: `localhost:9092`
- **Internal (containers)**: `kafka:29092`
- **Controller**: `kafka:9093`

### Kafka CLI Examples

**Create a topic**:
```bash
docker exec kafka /opt/kafka/bin/kafka-topics.sh \
  --create --topic test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 --replication-factor 1
```

**List topics**:
```bash
docker exec kafka /opt/kafka/bin/kafka-topics.sh \
  --list --bootstrap-server localhost:9092
```

**Produce messages**:
```bash
docker exec -it kafka /opt/kafka/bin/kafka-console-producer.sh \
  --topic test-topic --bootstrap-server localhost:9092
```

**Consume messages**:
```bash
docker exec -it kafka /opt/kafka/bin/kafka-console-consumer.sh \
  --topic test-topic --from-beginning \
  --bootstrap-server localhost:9092
```

## 🗄️ MySQL Usage

### Access via Adminer
1. Open http://localhost:8081
2. Login with credentials from `mysql/.env`:
   - **Server**: `mysql`
   - **Username**: `appuser` (or as configured)
   - **Password**: As set in `mysql/.env`
   - **Database**: As set in `mysql/.env`

### Direct MySQL Connection
```bash
# Connect via Docker
docker exec -it mysql mysql -u appuser -p

# Connect from host (requires MySQL client)
mysql -h localhost -P 3306 -u appuser -p
```

## 🔧 Environment Configuration

### Main `.env` file
Contains service orchestration and port mappings:
```properties
COMPOSE_FILE=docker-compose.yml:docker-compose.mysql.yml:docker-compose.adminer-mysql.yml:docker-compose.kafka.yml

# Port configurations
MYSQL_PUBLISH_PORT=3306
ADMINER_PUBLISH_PORT=8081
KAFKA_PUBLISH_PORT=9092
KAFKA_UI_PORT=8080
```

### Service-specific environment files
- `kafka/.env` - Kafka broker, KRaft, and UI settings
- `mysql/.env` - Database credentials and Adminer configuration

## 🐛 Troubleshooting

### General Issues
```bash
# Check container status
docker compose ps

# View logs for all services
docker compose logs

# View logs for specific service
docker compose logs kafka
docker compose logs mysql
```

### Kafka Issues
```bash
# Check Kafka health
docker exec kafka /opt/kafka/bin/kafka-broker-api-versions.sh \
  --bootstrap-server localhost:9092

# Check Kafka logs
docker compose logs kafka

# Restart Kafka
docker compose restart kafka
```

### MySQL Issues
```bash
# Check MySQL status
docker exec mysql mysqladmin -u root -p status

# Access MySQL directly
docker exec -it mysql bash

# Reset MySQL data (⚠️ destroys data)
docker compose down -v
docker volume rm docker-dev-common_mysql_data
```

### Network Issues
```bash
# Recreate network
docker network rm docker-dev-common
docker network create docker-dev-common

# Check network connectivity
docker exec kafka ping mysql
```

## 🔒 Security Notes

- **Development Only**: This configuration is optimized for development
- **Default Credentials**: Change default passwords in production
- **Network**: Services communicate via internal Docker network
- **Ports**: Only necessary ports are exposed to host

## 📝 Development Notes

- **Data Persistence**: All data is stored in Docker volumes
- **Auto-restart**: Services restart automatically unless stopped
- **Health Checks**: Kafka includes health checks for reliable startup
- **Override Support**: Use `docker-compose.override.yml` for local customizations
- **Git Ignore**: Environment files are ignored by git (use `.example` files for templates)

