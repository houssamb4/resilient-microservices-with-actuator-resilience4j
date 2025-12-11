# TP 26 : Microservice observable & résilient avec MySQL + Actuator + Profiles + Wait Strategy + Resilience4j + Multi-instances

TP complet pour apprendre l’observabilité, la résilience, les health checks Docker, Resilience4j, Spring Profiles, multi-instances et MySQL avec Spring Boot. L’architecture inclut pricing-service, trois instances de book-service et une base MySQL avec volume.

Avec Spring Boot • MySQL • Actuator • Profiles • Wait Strategy • Resilience4j • Multi-instances

Ce lab met en place une architecture composée de deux microservices Spring Boot :

- **pricing-service** : renvoie un prix et peut simuler une panne  
- **book-service** : gère des livres, stock MySQL, et appelle pricing-service  
  - utilise fallback (Resilience4j) si pricing-service est indisponible  
  - déployé en **3 instances** pour simuler une architecture scalable  

Le tout est orchestré via **Docker Compose** avec healthchecks, wait strategies et un volume MySQL.

---

## 🎯 Objectifs pédagogiques

- Observer un service via **Actuator** (health, readiness, liveness)
- Gérer les configurations avec **Spring Profiles**
- Mettre en place un **retry + circuit breaker + fallback** (Resilience4j)
- Empêcher un démarrage trop tôt grâce à une **wait strategy**
- Déployer plusieurs instances d’un même microservice
- Comprendre pourquoi les verrous Java ne fonctionnent plus en multi-instance
- Persister les données MySQL grâce à un **volume Docker**

---

## 📋 Prérequis

- **Java 17+** (recommandé Java 21)
- **Maven 3.6+**
- **Docker** et **Docker Compose**
- Connaissances de base en Spring Boot, Docker et microservices

---

## 🚀 Démarrage rapide

### Avec Docker Compose (recommandé)

```bash
# Cloner le projet
git clone <repository-url>
cd resilient-microservices-with-actuator-resilience4j

# Démarrer tous les services
docker-compose up --build

# Vérifier que les services sont démarrés
docker-compose ps
```

### Développement local

```bash
# Démarrer MySQL avec Docker
docker run --name mysql-bookstore -e MYSQL_DATABASE=bookdb -e MYSQL_USER=bookuser -e MYSQL_PASSWORD=bookpass -e MYSQL_ROOT_PASSWORD=rootpass -p 3306:3306 -d mysql:8.4

# Démarrer pricing-service
cd pricing-service
mvn spring-boot:run

# Démarrer book-service (dans un nouveau terminal)
cd ../book-service
mvn spring-boot:run -Dspring-boot.run.profiles=docker
```

---

## 🔍 Points d'API

### Book Service (port 8081)

- `GET /api/books` - Liste des livres
- `GET /api/books/{id}` - Détails d'un livre
- `POST /api/books` - Créer un livre
- `GET /actuator/health` - État de santé du service
- `GET /actuator/info` - Informations du service

### Pricing Service (port 8082)

- `GET /api/prices/{bookId}` - Prix d'un livre
- `GET /actuator/health` - État de santé du service

---

## 🧪 Tests

### Tester la santé des services

```bash
# Book Service
curl http://localhost:8081/actuator/health

# Pricing Service
curl http://localhost:8082/actuator/health
```

### Tester la résilience

1. **Arrêter pricing-service** :
   ```bash
   docker-compose stop pricing-service
   ```

2. **Tester book-service** : Les appels devraient utiliser le fallback (prix = 0.0)

3. **Redémarrer pricing-service** :
   ```bash
   docker-compose start pricing-service
   ```

### Tester les profils Spring

- **Profil `dev`** : Utilise H2 en mémoire (pour développement)
- **Profil `docker`** : Utilise MySQL (pour production/Docker)

---

## ⚙️ Configuration

### Resilience4j (book-service)

```yaml
resilience4j:
  retry:
    instances:
      pricing:
        maxAttempts: 3
        waitDuration: 300ms
  circuitbreaker:
    instances:
      pricing:
        slidingWindowType: COUNT_BASED
        slidingWindowSize: 10
        minimumNumberOfCalls: 5
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
        permittedNumberOfCallsInHalfOpenState: 2
```

### Actuator

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info"
  endpoint:
    health:
      show-details: "always"
      probes:
        enabled: true
```

---

## 📊 Observabilité

- **Health Checks** : `/actuator/health` montre l'état des composants (DB, disque, etc.)
- **Readiness/Liveness Probes** : Activées pour Kubernetes/Docker
- **Métriques** : Exposition des métriques via Actuator

---

## 🛠️ Développement

### Structure du projet

```
.
├── book-service/
│   ├── src/main/java/com/example/bookservice/
│   │   ├── BookServiceApplication.java
│   │   ├── config/
│   │   ├── domain/
│   │   ├── repo/
│   │   ├── service/
│   │   └── web/
│   └── pom.xml
├── pricing-service/
│   ├── src/main/java/com/example/pricingservice/
│   └── pom.xml
├── docker-compose.yml
└── README.md
```

### Commandes utiles

```bash
# Nettoyer et reconstruire
docker-compose down -v
docker-compose up --build

# Logs
docker-compose logs -f book-service-1

# Accéder à MySQL
docker exec -it mysql-bookstore mysql -u bookuser -p bookdb
```

