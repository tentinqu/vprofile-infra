# vProfile Deployment Guide

A complete AWS deployment of a Java-based Spring MVC web app using MySQL, RabbitMQ, Memcached, ElasticSearch, and Tomcat with autoscaling, load balancing, and HTTPS.

---

## Prerequisites

- JDK 17 or 21  
- Maven 3.9  
- MySQL 8  

---

## Technologies Used

- Spring MVC  
- Spring Security  
- Spring Data JPA  
- Maven  
- JSP  
- Tomcat  
- MySQL  
- Memcached  
- RabbitMQ  
- ElasticSearch  

---

## Database Setup

Import the SQL dump to your MySQL DB server:

```bash
mysql -u <user_name> -p accounts < src/main/resources/db_backup.sql

## Architecture

