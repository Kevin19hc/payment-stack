#  Payment Stack (Runner)

Este repositorio **no contiene código de negocio**.  
Su función es orquestar los microservicios del ecosistema de pagos con **Docker Compose**, levantando toda la infraestructura con un solo comando.

Incluye los siguientes microservicios:

| Servicio | Puerto | Descripción |
|-----------|---------|-------------|
| `client-service` | 8081 | Gestión de clientes (registro, consulta, actualización). |
| `recipient-service` | 8082 | Administración de destinatarios o proveedores asociados. |
| `product-service` | 8083 | Catálogo de productos disponibles para pago. |
| `payment-service` | 8084 | Procesamiento de pagos y notificaciones vía RabbitMQ. |

---

##  Infraestructura

| Servicio | Puerto | Descripción |
|-----------|---------|-------------|
| `MongoDB` | 27017 | Base de datos NoSQL compartida (cada micro usa su propia colección). |
| `RabbitMQ` | 5672 / 15672 | Mensajería entre servicios (solo usado por `payment-service`). UI en [http://localhost:15672](http://localhost:15672). |

---

##  **RabbitMQ Configuration**

### **Exchange**
| Property | Value |
|-----------|--------|
| **Name** | `payment.exchange` |
| **Type** | `topic` |
| **Durable** | `true` |
| **Auto Delete** | `false` |
| **Description** | Main exchange where payment-related events are published. |


### **Queues**
| Queue | Durable | Auto Delete | Routing Key | Description |
|--------|----------|--------------|------------|--------------|
| `payment.status.queue` | `true` | `false` |  | Queue that receives notifications when a payment status changes. |

We can consult the definition in the following document:
```
/docs/rabbit_definitions.json
```

---

##  Requisitos previos

- Docker instalado.
- Puertos libres: `27017`, `5672`, `15672`, `8081–8084`.
---

##  Configuración mínima por microservicio

| Servicio | Puerto | Base de datos |
|-----------|---------|---------------|
| client-service | 8081 | `clients_db` |
| recipient-service | 8082 | `recipients_db` |
| product-service | 8083 | `products_db` |
| payment-service | 8084 | `payments_db` |

---

##  Cómo ejecutar el stack

Esta sección explica paso a paso cómo levantar todo el ecosistema de microservicios usando **Docker Compose**.  
El stack incluye los servicios de *clientes, destinatarios, productos y pagos*, junto con **MongoDB** y **RabbitMQ**.

---

###  1.-Clonar los proyectos

Antes de poder levantar el entorno, necesitas tener **todos los microservicios y el stack orquestador** en tu computadora.  
Cada uno está en su propio repositorio, pero **todos deben estar en la misma carpeta de trabajo** para que Docker Compose pueda encontrarlos correctamente.

---

####  Paso 1: Crear la carpeta principal

Crea una carpeta donde estarán los repositorios del proyecto.  
En este ejemplo, usaremos `/workspace`, pero puedes usar cualquier nombre o ruta:

```bash
# Crear la carpeta base del proyecto
mkdir -p /workspace
```
```bash
# Entrar a la carpeta creada
cd /workspace
```
---

####  Paso 2: Clonar los microservicios

Cada microservicio tiene su propio repositorio.  
Ejecuta los siguientes comandos:

```bash
# Clonar los microservicios principales
git clone https://github.com/Kevin19hc/client-service.git client-service
git clone https://github.com/Kevin19hc/recipient-service.git recipient-service
git clone https://github.com/Kevin19hc/product-service.git product-service
git clone https://github.com/Kevin19hc/payment-service.git payment-service
```

 **Estos comandos crearán carpetas llamadas** `client-service`, `recipient-service`, `product-service` y `payment-service` **dentro de la carpeta** `/workspace`.

---

####  Paso 3: Estructura de carpetas esperada

Una vez que hayas clonado todos los proyectos, deberías tener una estructura de carpetas como esta:

```
/workspace/
├─ client-service/
├─ recipient-service/
├─ product-service/
├─ payment-service/
└─ payment-stack/
    └─ docker-compose.yml
```
---

###  2️.- Levantar todos los servicios

Una vez que tengas todos los repositorios clonados y la estructura de carpetas correcta, puedes **levantar todo el entorno con Docker Compose**.

Desde la carpeta del stack (`payment-stack`), ejecuta:

```bash
docker-compose up --build
```
---

###  3.- Verificación de APIs y paneles

Una vez que el stack esté en ejecución, podrás acceder a las interfaces **Swagger UI** de cada microservicio y al panel de administración de **RabbitMQ** desde tu navegador.

---

####  Swagger UI

| Servicio | URL |
|----------|-----|
| ️ Clients | [http://localhost:8081/swagger-ui/index.html](http://localhost:8081/swagger-ui/index.html) |
| Recipients | [http://localhost:8082/swagger-ui/index.html](http://localhost:8082/swagger-ui/index.html) |
| Products | [http://localhost:8083/swagger-ui/index.html](http://localhost:8083/swagger-ui/index.html) |
| Payments | [http://localhost:8084/swagger-ui/index.html](http://localhost:8084/swagger-ui/index.html) |

---

####  RabbitMQ Management

-  **URL:** [http://localhost:15672](http://localhost:15672)
-  **Usuario:** `user`
-  **Contraseña:** `pass`

 **Tip:**  
Desde este panel puedes visualizar las **colas**, **exchanges** y **mensajes** si el `payment-service` está utilizando RabbitMQ para la mensajería interna.
