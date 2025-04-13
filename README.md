# **Real-time Video Ranking Microservice**

## **Architecture**

![image](https://github.com/user-attachments/assets/336bc1e3-428f-4a08-8cff-8d7d11806891)

The system is structured as follows:

* **General Service:**  
  * Manages core data like user profiles, video metadata, and user history.  
  * Utilizes **RDS MySQL** for persistent data storage.  
  * Containerized and deployed on **AWS ECS**.  
* **Video Ranking Service:**  
  * Handles real-time video ranking based on interaction scores.  
  * Employs **Redis Sorted Sets (via ElastiCache)** for efficient storage and retrieval of ranked videos.  
  * Containerized and deployed on **AWS ECS**.  
* **Load Balancer (AWS Elastic Load Balancer):**  
  * Distributes incoming traffic from the internet to the Video Ranking Service.  
* **Communication:**  
  * Services communicate with each other using **gRPC** for efficient and fast data exchange.

## **Key Technologies**

* **Go:** Programming language for both microservices.  
* **AWS ECS:** Container orchestration service.  
* **AWS RDS (MySQL):** Relational database for persistent storage.  
* **AWS ElastiCache (Redis/Valkey):** In-memory data store for caching and real-time ranking.  
* **gRPC:** High-performance RPC framework for inter-service communication.  
* **Swagger (OpenAPI):** API documentation.

## **API Endpoints**

* Available end points and usagein the [Swagger document](http://vrs-lb-77799277.ap-southeast-1.elb.amazonaws.com:9000/swagger/index.html).

## **Testing**

* Unit tests core functionality, .  
* Run tests using `make test` in each service directory. Check coverage details using `make coverage`.

## **Deployment**

Detailed deployment instructions are available in the [Deployment Guide](Deployment.md).

## **Detail implementations**
* [General service](github.com/dailoi280702/vrs-general-service)
* [Video ranking service](github.com/dailoi280702/vrs-ranking-service)
