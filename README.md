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

