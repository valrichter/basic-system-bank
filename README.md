# <img id="go" src="https://devicon-website.vercel.app/api/go/plain.svg?color=%2300ACD7" width="30" /> Proyecto extenso de Backend con Go
Se desarolla el servicio web backend basico de un banco. Se aprendio a como diseñar, desarrollar e implementar un servicio web backend desde cero, proporcionando APIs para que el frontend realice las siguientes acciones:
   1. Crear y gestionar cuentas bancarias.
   2. Registrar todos los cambios de saldo en cada una de las cuentas.
   3. Realizar una transferencia de dinero entre 2 cuentas.

## ⚡ Acciones realizadas durante el proyecto:
- [🗃️ 1. Trabajando con la DB [PostgreSQL + sqlc]](#seccion-1)  
Se profundizo en el diseño de bases de datos, permitiendo modelar y gestionar datos de manera eficiente. Se interactuo con la base de datos utilizando transacciones y se comprendieron los niveles de aislamiento de la base de datos. Tambien se aprendió a utilizar Docker para crear entornos locales de desarrollo y GitHub Actions para automatizar las pruebas unitarias.
- [🧩 2. Construccion de una RESTful HTTP JSON API [Gin + JWT + PASETO]](#seccion-2)  
Se desarrollo una RESTful APIs utilizando el framework Gin en Golang. Se apredio a cargar configuraciones de la aplicación, simular mocks de bases de datos para pruebas sólidas y se aplico autenticación de usuarios junto con la seguridad de las APIs con tokens JWT y PASETO.
- [☁️ 3. DevOps: Deployar la aplicacion a produccion [Docker + Kubernetes + AWS]](#seccion-3)  
Se amplio el conocimiento al de como construir y desplegar una aplicación en un cluster de Kubernetes en AWS. A través de guias detalladas, se comprendio como crear imágenes Docker eficientes, configurar bases de datos de produccion, gestionar secretos de manera segura, implementar Kubernetes con EKS.

## 🔨 Tecnologias usadas:
- **PostgreSQL**: docker image postgres:15.4
- **Docker**: docker v24.0.6
- **Kubectl**: Kubectl Client Version: v1.28.2
- **CI**: GitHub Actions
- **SQLC**: sqlc-dev/sqlc v1.21.0
- **Migrate**: golang-migrate v4.16.2
- **Make**: GNU Make v4.3
- **jq**: jq v1.6
- **AWS CLI**: aws-cli v2.13.24
- **AWS**: ECR, RDS, Secrets Manager, EKS

## 📦 Herramietas:
- **Gin**: gin-gonic/gin v1.9.1
- **Testify**: stretchr/testify v1.8.4
- **Viper**: spf13/viper v1.16.0
- **GoMock**: golang/mock v1.6.0
- **JWT**: golang-jwt/jwt v3.2.2+incompatible
- **Paseto**: o1egl/paseto v1.0.0
- **GoFakeit**: brianvoe/gofakeit/v6 v6.23.2

## Seccion 1
- [🗃️ 1. Trabajando con la DB [PostgreSQL + sqlc]](#seccion-1)
- [🧩 2. Construccion de una RESTful HTTP JSON API [Gin + JWT + PASETO]](#seccion-2)
- [☁️ 3. DevOps: Deployar la aplicacion a produccion [Docker + Kubernetes + AWS]](#seccion-3)

### 🗃️ Trabajando con la DB [PostgreSQL + sqlc]
Se profundizo en el diseño de bases de datos, permitiendo modelar y gestionar datos de manera eficiente. Se interactuo con la base de datos utilizando transacciones y se comprendieron los niveles de aislamiento de la base de datos. Tambien se aprendió a utilizar Docker para crear entornos locales de desarrollo y GitHub Actions para automatizar las pruebas unitarias.

**1.** Esquema de la DB y relacion entre tablas
   - Crear una Account (Owner, Balance, Currency)
   - Registrar todos los cambios de balance en la cuenta (Entry)
   - Hacer transferencias de dinero entre 2 Accounts (Transfer)

   <img src="https://github.com/valrichter/basic-system-bank/assets/67121197/f0087f1e-ab3b-4532-a7bc-1a578c7c1e2c"/><br>

**2.** Configuracion de imagen de PostgreSQL en Docker y creacion de la DB mediante un archivo *.sql*
   - Se agregaron indices (indexes) de los atributos mas importantes de cada tabla para mayor eficiencia a la hora de la busqueda

**3.** Creacion de versiones de la DB. Configuracion golang-migrate para hacer migraciones
s de la DB de una version a otra:
   - Se agrego un Makefile para mayor comodidad a la hora de ejecutar comandos necesarios

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/f45876db-0fe9-4b3a-9e38-7256e346bb16"/><br>

**4.** Generacion de CRUD basico para las tablas Account, Entry & Transfer con sqlc. Configuracion de sqlc para hacer consultas SQL con codigo Go
   - Como funciona:
      - **Input**: Se escribe la consulta en SQL ---> **Blackbox**: [sqlc] ---> **Output**: Funciones en Golang con interfaces para poder utilizarlas y hacer consultas

**5.** Generacion de datos falsos y creacion de Unit Tests para CRUD de la tabla Account:
   - Utlizacion del archivo ```random.go```

**6.** Creacion de una transaccion ```StoreTx.go``` con las propiedades ACID para la transferencia de dinero entre 2 Accounts y su respectivo Unit Test
   - Funcionalidad de negocio a implementar ---> Transferir de la cuenta bancaria "account1" a la cuenta bancaria "account2" 10 USD
   - Pasos de la implementacion:
     1. Crear un registro de la transferecnia de 10 USD
     2. Crear un ```entry``` de dinero para la account1 con el un amount = -10
     3. Crear un ```entry``` de dinero para la account2 con el un amount = +10
     4. Restar 10 USD del balance total que posee la account1
     5. Sumar 10 al balance de la account2

**7.** Creacion de Unit Tests (TDD) con go routines para simular tracciones concurrentes. Aplicando ```transaction locks``` con la clausula ```FOR UPDATE``` para evitar que se leean o escribar valores erroneos de una misma variable.
   - Como utilizamos ```transaction locks```:
     - Si la transaccion1 quiere acceder a una variable la cual en ese momento esta siendo utilizada por la transaccion2, la transaccion1 debera esperar a que la transaccion2 termine en COMMIT o ROLLBACK antes de poder acceder a dicha varibale  
   
**8.** Modificacion del codigo para evitar situaciones deadlock y Unit Test para transactions deadlocks. Aclarar que esto solo se puede evitar/mitigar y corregir dentro del codigo y la logica de negocio
   - La Transaccion A para finalizar necesita de la Data 2, la cual esta siendo usada (y por ende bloqueanda) por la Transaccion B
   - A su vez la Transaccion B para finalizar necesita de la Data 1 la cual esta siendo usada (y por ende bloqueanda) por la Transaccion A
   - Esto provoca el problema de Deadlock

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/c4f841bd-2d33-4a91-b829-f4b39397b098"/><br>

**9.** Estudio de los distintos Insolation Levels en PostgreSQL:
| Read Phenomena / Isonlation Levels ANSI | Read Uncommited | Read Commited | Repeatable Read | Serializable |
| :-------------------------------------: | :-------------: | :-----------: | :-------------: | :----------: |
|               Dirty Read                |       NO        |      NO       |       NO        |      NO      |
|           Non-Repeatable Read           |       SI        |      SI       |       NO        |      NO      |
|              Phantom Read               |       SI        |      SI       |       NO        |      NO      |
|          Serialization Anomaly          |       SI        |      SI       |       SI        |      NO      |

**10.** Implementacion de Continouous Integration (CI) con GitHub Actions para garatizar la calidad del codigo y reducir posibles errores
   - El ```Workflow``` consta de varios Jobs
   - Cada ```Job``` es un proceso automatizado
   - Los Jobs pueden ser ejecutados o bien por un ```event``` que ocurre dentro del repositorio de github o estableciendo un ```scheduled``` o ```manually``` (manualmente)
   - Para poder ejecutar un Job necesitamos especificar un Runner para cada uno de ellos
   - Un ```Runner``` es un servidor que escucha los Jobs diponibles y solo ejecuta un Job a la vez. Es parecido a un caontainer de docker
   - Luego cada Runner informa su progreso, logs y resultados a github
   - Un Job es un conjunto de ```Steps``` que se ejecutaran en un mismo Runner
   - Todos los Jobs se ejecutan en paralelo exepto cuando hay algunos Jobs que dependen entre si, entonces esos se ejecutan en serie
   - Los ```Step``` son tareas individuales que se ejecutan en serie dentro de un Job
   - Un Step puede contener una o varias Actions que se ejecutan en serie
   - Una ```Action``` es un comando independiente y estas se pueden reutilizar. Por ej: ```actions/checkout@v4``` la cual verifica si nuestro codigo corre localmente

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/c79b7e51-e376-4a0e-9831-4bd1a711ffc1"/><br>

**Seccion 1:** Arquitectura de la aplicacion en la primer seccion
   - Resumen:
      - Modelado de los datos
      - EWjecucion en entorno local
      - Base de datos local (PostgreSQL docker)
      - Implementacion basica de CI

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/94e19962-d5f6-48c0-bddd-d74701b1b4dc"/><br>

***

## Seccion 2
- [🗃️ 1. Trabajando con la DB [PostgreSQL + sqlc]](#seccion-1)
- [🧩 2. Construccion de una RESTful HTTP JSON API [Gin + JWT + PASETO]](#seccion-2)
- [☁️ 3. DevOps: Deployar la aplicacion a produccion [Docker + Kubernetes + AWS]](#seccion-3)

### 🧩 Construccion de una RESTful HTTP JSON API [Gin + JWT + PASETO]
Se desarrollo una RESTful APIs utilizando el framework Gin en Golang. Se apredio a cargar configuraciones de la aplicación, simular mocks de bases de datos para pruebas sólidas y se aplico autenticación de usuarios junto con la seguridad de las APIs con tokens JWT y PASETO.

**1.** Implementacion de una RESTful HTTP API basico con el framework Gin, configurancion del server y agregado de las funciones createAccount, getAccount by id y listAccount para listar cuentas mediante paginacion con la respectiva validacion de los datos recibidos a traves de JSON

**2.** Creacion y configuracion de variables de entorno ```.env``` con la herramienta Viper

**3.** Implementacion de database mock (DB temporal) con GoMock para testear los metodos en ```account.go``` de la API HTTP y logrando una covertura del 100% del metodo GetAccounts
   - Porque implementar Mock Database:
     - Escribir test independientes es mas facil porque cada test utilizara su propia db
     - Test mas rapidos ya que no se espera la coneccion a la db y tampoco hay espera en la ejecucion de las querys. Todas la acciones son realizadas en memoria
     - Permite escribir test con 100% de covertura ya que podemos configurar casos extremos

**4.** Implementacion de ```transfer.go``` de la API HTTP para enviar dinero entre dos cuentas y se agrego un ```validator.go``` para validar la Currency de las cuentas relacionadas con la transferencia de dinero

**5.** Cambio en la base de datos. 
   - Se agrego la tabla User para que cada usuario pueda tener distintas Accounts con diferentes Currency como ARS, UDS o EUR

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/54005e6b-ebad-4689-af1d-d1b602b25c9a"/><br>

**6.** Implentacion y test de CRUD para la tabla Users, manejo de errores de PostgreSQL y fix de la API ```account.go``` para que funcione con la nueva tabla Users

**7.** Implentacion del la API ```user.go``` y encriptacion de la password de los Users utilizando bcrypt
   - Como funciona:
      - **Input**: password123 ---> **Blackbox**: bcrypt ---> **Output**: $2a$10$BqwmET/4eq5.uIibth/rrOqSC2eqo5cy80Yj2RKuLicpRCIm1RlX. 

**8.** Creacion de unit test mas solidos con un comparador gomock personalizado para la funcion createUser de la API ```user.go```.

**9.** Compresion del funcionamento de la autenticacion por tokens, diferencias entre JWT y PASETO, problemas de seguridad de JWT y como funciona PASETO
   - Como funciona la autenticacion por token:
     - El cliente proporciona username y password
     - Si son correctos el servidor creara y firmara un token con la secret key
     - Luego mandara un access token
     - Luego si el cliente desea acceder a algun recurso lo hace utilizando el token en la solicitud
     - El servidor verifica el toke, si es valido autoriza la solicitud

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/07e1ed84-4838-4a4f-899f-2671aca1becd"/><br>

**10.** Creacion y verificacion de tokens de JWT y PASETO con sus respectivos tests

**11.** Implementacionde de la API de login para que devuelve el token de acceso ya sea en PASETO o JWT
   - La duracion del token de login se establecio en 15 minutos

**12.** Implementacion de middleware de autenticación y reglas de autorización usando Gin. Permitiendo manejar errores de manera mas eficiente

**Seccion 2.** Arquitectura de la aplicacion en la segunda seccion
   - Resumen:
      - Creacion de la API
      - Autenticaion de Users
      - Encriptacion de passwords
      - Database Mock Test

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/f0991003-1fdc-4a26-bd85-87d0dc3ee534"/><br>

***

## Seccion 3
- [🗃️ 1. Trabajando con la DB [PostgreSQL + sqlc]](#seccion-1)
- [🧩 2. Construccion de una RESTful HTTP JSON API [Gin + JWT + PASETO]](#seccion-2)
- [☁️ 3. DevOps: Deployar la aplicacion a produccion [Docker + Kubernetes + AWS]](#seccion-3)

### ☁️ DevOps: Deployar la aplicacion a produccion [Docker + Kubernetes + AWS]
Se amplio el conocimiento al de como construir y desplegar una aplicación en un cluster de Kubernetes en AWS. A través de guias detalladas, se comprendio como crear imágenes Docker eficientes, configurar bases de datos de produccion, gestionar secretos de manera segura, implementar Kubernetes con EKS.

**1.** Se creo un archivo ```Dockerfile``` multietapa para crear una imagen mínima de Golang Docker que contenga solo el binario ejecutable de la app
   - Esto es util a la hora de correr una aplicacion de produccion en cualquier parte

**2.** El container de la DB y el container de la APP, ambos, fueron conectados a una misma network para que puedan comunicarse entre containers

**3.** Configuracion de ```Docker-compose``` para inicializar los dos servicios (APP y DB), coordinarlos y controlar las órdenes de inicio del servicio
   - Esta parte requirio mucha investigacion sobre como funciona docker y docker-compose

**4.** Investigacion de como usar AWS para conrrer servicios en la nube y creacion de una cuenta de AWS

**5.** Se automatizo la creacion y envío de la imagen de Docker a ```AWS ECR``` con Github Actions. Esto se configuro en el archivo ```github/workflow/deploy.yml```
   - Cada vez que se hace un nuevo pull a la rama main se crea y envia una nueva imagen de docker al repositorio de AWS ECR

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/c38dd623-46cd-4c87-a247-6e9449e7abac"/><br>

**6.** Configuracion de ```AWS RDS``` para levantar una PostgreSQL DB de produccion en la nube

**7.** Almacenamiento y recuperacion secretos de producción con el administrador de secretos ```AWS Secrets Manager``` 
  - Se uso la aplicacion de AWS CLI para conectar darle a GithubActions las credenciales de AWS
  - Se uso jq para poder extraer las variables de AWS Secrets Manager:
    - ```aws secretsmanager get-secret-value --secret-id go-basic-bank```
    - ```aws secretsmanager get-secret-value --secret-id go-basic-bank --query SecretString --output text | jq -r 'to_entries|map("(.key)=(.value)")|.[]'```

<img src="https://github.com/valrichter/go-basic-bank/assets/67121197/ff64d37b-760d-4f7d-bd00-17bf155884e7"/><br>

**8.** Comprecion de la arquitectura de Kubernetes y como crear un cluster EKS en AWS y agregarle nodos trabajadores

**9.** Utilizacion kubectl y k9s para conectarse a un cluster de Kubernetes en ```AWS EKS```
   - Se le dio acceso a GitHub Actions al cluster de kubernetes AWS EKS mediante Kubectl para mas adelante poder crear un Continous Deployment (CD)

**10.** Deployment de una aplicación web en un cluster de Kubernetes en AWS EKS
   - Se utilizo Kubernetes para exponer la API bancaria al publico junto con el servicio cloud AWS EKS
   - La API esta expuesta a traves un servicio de LoadBalance de Kubernetes
   - El ```deployment.yml``` se uso para configurar los pods
   - El ```service.yml``` para exponer el container de la app al publico. Es decir, el servicio de API esta expuesto y corriendo en AWS para que pueda consumirlo cualquiera que lo desee

**11.** Se investigo como registrar un dominio y configurar A-record usando AWS Route53
   - No se puedo implementar por falta de presupuesto (no tengo 12 USD para comprar "go-basic-bank.com")

**12.** Se investigo como usar el servicio Ingress para enrutar el trafico a diferentes servicios en Kubernetes con nginx ingress
   - No se puedo implementar por falta de dominio (presuspuesto de 12 USD)
   - Solo que expueso la URL que proporciona el Load Blanacer (no es lo mejor, deberia exponerlo con un Ingress pero es lo que hay)

**13.** Se investigo la emision automatica de certificados TLS en Kubernetes con Let's Encrypt
   - No se pueod implementar, es necesario un servicio Ingress
