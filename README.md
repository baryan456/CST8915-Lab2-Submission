# CST8915 Lab 2 – Full-Stack Cloud-Native Development

## Student Information

- **Name:** Aryan Banoth
- **Student ID:** 041293006
- **Course:** CST8915 – Full-stack Cloud-native Development

---

## Lab Overview

The objective of this lab was to refactor the Algonquin Pet Store application according to the 12-Factor App methodology and deploy the application across four separate Azure Virtual Machines.

The application consists of four main components:

1. Store Front
2. Order Service
3. Product Service
4. RabbitMQ

Each component was deployed on its own Azure VM.

---

## Architecture

```text
                         User
                          |
                          v
                 +----------------+
                 |   Store Front  |
                 |     :8080      |
                 +-------+--------+
                         |
                  +------+------+
                  |             |
                  v             v
          +-------------+  +-------------+
          |   Product   |  |    Order    |
          |   Service   |  |   Service   |
          |    :3030    |  |    :3000    |
          +-------------+  +------+------+
                                  |
                                  v
                           +-------------+
                           |  RabbitMQ   |
                           |    :5672    |
                           +-------------+
```

### Application Flow

The Store Front loads product information from the Product Service.

When a customer places an order, the Store Front sends the order to the Order Service.

The Order Service then sends the order to RabbitMQ, which acts as the backing service for the application.

---

## Azure VM Deployment

The application was deployed across four separate Azure Virtual Machines:

| Component | Port | Purpose |
|---|---:|---|
| Store Front | 8080 | User interface |
| Order Service | 3000 | Processes customer orders |
| Product Service | 3030 | Provides product information |
| RabbitMQ | 5672 | Message queue / backing service |

---

# 12-Factor App Implementation

## 1. Codebase

For the Codebase factor, I created separate GitHub repositories for each microservice.

This allows each service to have its own source code, version control, and development workflow.

### Repositories

- [Order Service](https://github.com/baryan456/CST8915-lab2-orderservice)
- [Product Service](https://github.com/baryan456/CST8915-lab2-productservice)
- [Store Front](https://github.com/baryan456/CST8915-lab2-storefront)

Having separate repositories helps maintain independence between the microservices and allows each service to be developed and deployed independently.

---

## 2. Dependencies

For the Dependencies factor, each service explicitly defines its required dependencies.

### Node.js Services

The Order Service and Store Front use:

- `package.json`
- `package-lock.json`

Dependencies were installed using:

```bash
npm ci
```

### Product Service

The Product Service is implemented using Rust and uses:

- `Cargo.toml`
- `Cargo.lock`

The service was built using Cargo:

```bash
cargo build
```

This ensures that the dependencies required by each service are clearly defined and reproducible.

---

## 3. Configuration

For the Configuration factor, application configuration was separated from the source code by using environment variables.

### Order Service

The Order Service uses environment variables for:

- RabbitMQ connection information
- Application port

Example:

```text
PORT=3000
RABBITMQ_CONNECTION_STRING=<RabbitMQ connection string>
```

### Product Service

The Product Service uses:

```text
PORT=3030
```

### Store Front

The Store Front uses environment variables to identify the Order Service and Product Service:

```text
VUE_APP_ORDER_SERVICE_URL=http://9.160.106.20:3000
VUE_APP_PRODUCT_SERVICE_URL=http://9.160.106.50:3030
```

Sensitive configuration such as passwords was kept out of the GitHub repositories. The `.env` files were added to `.gitignore`.

This makes it possible to change configuration without modifying the application source code.

---

## 4. Backing Services

RabbitMQ was deployed on a dedicated Azure VM and treated as an external backing service.

The Order Service connects to RabbitMQ remotely using the RabbitMQ connection string stored in an environment variable.

RabbitMQ was configured with an application user and an `order_queue`.

The queue was verified using:

```bash
sudo rabbitmqctl list_queues name durable messages
```

The order placed through the Store Front was successfully added to the RabbitMQ queue.

---

# Application Verification

## Product Service

The Product Service was tested using:

```bash
curl http://9.160.106.50:3030/products
```

The service returned:

```text
[
  {"id":1,"name":"Dog Food","price":19.99},
  {"id":2,"name":"Cat Food","price":34.99},
  {"id":3,"name":"Bird Seeds","price":10.99}
]
```

This confirmed that the Product Service was running correctly.

---

## Order Service

The Order Service was deployed on port `3000`.

It was verified using:

```bash
curl -i http://9.160.106.20:3000
```

The service returned:

```text
Cannot GET /
```

This confirmed that the Express server was reachable. The `/` route is not an application endpoint.

The actual order endpoint was tested by sending an order to:

```text
POST /orders
```

and the service returned:

```text
Order received
```

---

## Store Front

The Store Front was deployed on port `8080`.

It was accessed through:

```text
http://158.23.163.234:8080
```

The Store Front successfully loaded the products from the Product Service.

An order was placed for two Dog Food products and the application displayed:

```text
Order for 2 x Dog Food placed successfully! Total: $39.98
```

---

## RabbitMQ Verification

After placing an order, the RabbitMQ queue was checked using:

```bash
sudo rabbitmqctl list_queues name durable messages
```

The `order_queue` contained the order message.

This confirmed the complete application flow:

```text
Store Front
     |
     v
Product Service

Store Front
     |
     v
Order Service
     |
     v
RabbitMQ
```

---

# Reflection Questions

## 1. What changes did you make to the order-service and product-service to comply with the Configurations and Backing Services factors of the 12-Factor App methodology?

I moved configuration values such as application ports and the RabbitMQ connection string into environment variables instead of hard-coding them in the application. The Order Service uses an environment variable to connect to RabbitMQ. RabbitMQ was deployed on a separate Azure VM and treated as an external backing service. The Product Service also uses an environment variable for its port.

## 2. Why is it important to use environment variables instead of hard-coding configurations in your application?

Environment variables separate configuration from application code. This makes the application easier to deploy in different environments because configuration values can be changed without modifying the source code. Environment variables also help protect sensitive information such as passwords and connection strings from being committed to source control.

## 3. Why is it important to have separate repositories for each microservice? How does this help maintain independence and scalability of each service?

Separate repositories allow each microservice to be developed, tested, maintained, and deployed independently. A change to one service does not require changes to the other services. This provides better independence between services and makes it easier to scale or update individual services when required.

---

# Demo Video

The Lab 2 demonstration video is available here:

[Watch the CST8915 Lab 2 Demo](https://youtu.be/6FVh9vbCRes)

The demonstration shows:

- Store Front running on its own Azure VM
- Four separate Azure VMs
- Product Service providing products
- Order Service processing an order
- Environment-based configuration
- RabbitMQ running as a backing service
- An order being placed through the Store Front
- The order being queued in RabbitMQ

---

# GitHub Repositories

### Order Service

https://github.com/baryan456/CST8915-lab2-orderservice

### Product Service

https://github.com/baryan456/CST8915-lab2-productservice

### Store Front

https://github.com/baryan456/CST8915-lab2-storefront


---

# Conclusion

This lab demonstrated how to refactor a full-stack application using principles from the 12-Factor App methodology and deploy its components independently across Azure Virtual Machines.

The final deployment successfully demonstrated communication between the Store Front, Product Service, Order Service, and RabbitMQ backing service.
