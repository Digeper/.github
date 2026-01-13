# Muzika - Music Platform

P2P web music listener running on Kubernetes.

## What is Done

Microservices-based P2P music platform with:
- **AuthorizationManager** - User authentication & JWT tokens
- **PlaylistManager** - Playlist CRUD operations
- **QueueManager** - Music queue & playback management
- **BandcampApi** - Bandcamp music search integration
- **SlskdDownload** - Song download service
- **MuzikaPlayer** - Vue.js frontend
- **Kafka** - Message broker for service communication
- **PostgreSQL** - Database

## Local Deployment

1. **Start dependencies:**
   ```bash
   docker-compose -f compose.yaml.yaml up -d  # PostgreSQL
   # Start Kafka locally on localhost:9092
   ```

2. **Run services:**
   ```bash
   # Backend services (from each module directory)
   mvn spring-boot:run
   
   # Frontend
   cd MuzikaPlayer
   npm install
   npm run dev
   ```

3. **Services run on:**
   - AuthorizationManager: `8091`
   - QueueManager: `8090`
   - PlaylistManager: `8092`
   - MuzikaPlayer: `5173` (Vite dev server)

## Kubernetes Deployment

**Prerequisites:** Azure AKS cluster, ACR registry, kubectl configured

1. **Deploy Kafka first:**
   ```bash
   kubectl apply -f kafka/namespace.yaml
   kubectl apply -f kafka/kafka-cluster.yaml
   kubectl apply -f kafka/kafka-topics.yaml
   ```

2. **Deploy services:**
   ```bash
   # Each service
   kubectl apply -k AuthorizationManager/k8s/
   kubectl apply -k PlaylistManager/k8s/
   kubectl apply -k QueueManager/k8s/
   kubectl apply -k BandcampApi/k8s/
   kubectl apply -k SlskdDownload/k8s/
   kubectl apply -k MuzikaPlayer/k8s/
   ```

3. **Or use CI/CD:** Push to `main` branch triggers GitHub Actions workflows for each service.

**Note:** Update `kustomization.yaml` with your ACR name and image tags before deploying.
