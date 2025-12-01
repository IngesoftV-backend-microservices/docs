# Patrones de Diseño y Arquitectura - E-Commerce Microservices

Este documento identifica y documenta los patrones de diseño y arquitectónicos utilizados en el proyecto de microservicios E-Commerce, describiendo qué patrones se aplicaron, dónde se implementaron y qué beneficios aportaron al sistema.

## Tabla de Contenidos

1. [Patrones Arquitectónicos](#patrones-arquitectónicos)
2. [Patrones de Diseño](#patrones-de-diseño)
3. [Patrones de Comunicación](#patrones-de-comunicación)
4. [Patrones de Resiliencia](#patrones-de-resiliencia)
5. [Patrones de Observabilidad](#patrones-de-observabilidad)
6. [Patrones de Manejo de Errores](#patrones-de-manejo-de-errores)
7. [Patrones de Configuración](#patrones-de-configuración)

---

## Patrones Arquitectónicos

### 1. Arquitectura de Microservicios

**Descripción:** El sistema está dividido en múltiples servicios independientes, cada uno con su propia base de datos y responsabilidades específicas del dominio de negocio.

**Servicios del dominio implementados:**
- **User Service**: Gestiona la información de usuarios, credenciales de autenticación y direcciones de envío.
- **Product Service**: Administra el catálogo de productos y las categorías asociadas.
- **Order Service**: Maneja la creación y gestión de órdenes de compra y carritos de compra.
- **Payment Service**: Procesa las transacciones de pago asociadas a las órdenes.
- **Shipping Service**: Gestiona los items de orden relacionados con el envío de productos.
- **Favourite Service**: Permite a los usuarios marcar productos como favoritos.

**Servicios de infraestructura implementados:**
- **API Gateway**: Actúa como punto de entrada único para todas las peticiones del sistema, centralizando el enrutamiento y funcionalidades transversales.
- **Service Discovery**: Implementa un servidor de registro y descubrimiento de servicios utilizando Netflix Eureka, permitiendo que los servicios se encuentren dinámicamente.
- **Cloud Config**: Proporciona un servidor de configuración centralizada que permite gestionar la configuración de todos los microservicios desde un punto único.
- **Proxy Client**: Cliente web que consume los microservicios y presenta la interfaz de usuario final.

**Ubicación:** Cada servicio está organizado en su propio directorio dentro de la estructura del proyecto, manteniendo una separación clara de responsabilidades.

**Beneficios obtenidos:**
- **Escalabilidad independiente**: Cada servicio puede escalarse según sus necesidades específicas sin afectar a otros servicios.
- **Despliegue independiente**: Los servicios pueden desplegarse y actualizarse de forma independiente, reduciendo el riesgo y el tiempo de despliegue.
- **Tecnología heterogénea**: Permite utilizar diferentes tecnologías y frameworks según las necesidades de cada servicio.
- **Aislamiento de fallos**: Un fallo en un servicio no afecta directamente a otros servicios, mejorando la resiliencia del sistema.

---

### 2. API Gateway Pattern

**Descripción:** Se implementó un punto de entrada único que centraliza el enrutamiento de peticiones hacia los microservicios apropiados y proporciona funcionalidades transversales como autenticación, balanceo de carga y circuit breaking.

**Tecnología utilizada:** Spring Cloud Gateway, que proporciona un gateway reactivo y no bloqueante para el enrutamiento de peticiones.

**Funcionalidades implementadas:**
- **Enrutamiento basado en rutas**: Las peticiones se enrutan a los microservicios correspondientes según el path de la URL.
- **Balanceo de carga**: Utiliza load balancing automático para distribuir la carga entre múltiples instancias de un mismo servicio.
- **Configuración CORS**: Centraliza la configuración de Cross-Origin Resource Sharing para permitir peticiones desde el cliente web.
- **Integración con Circuit Breaker**: Se integra con Resilience4j para implementar circuit breaking y proteger el sistema ante fallos.

**Ubicación:** El API Gateway se encuentra implementado en el directorio `api-gateway/` del proyecto.

**Beneficios obtenidos:**
- **Punto de entrada único**: Simplifica la comunicación de los clientes, que solo necesitan conocer una única URL.
- **Desacoplamiento de clientes**: Los clientes no necesitan conocer la ubicación específica de cada microservicio.
- **Funcionalidades transversales centralizadas**: Funcionalidades como autenticación, logging y monitoreo se implementan una sola vez en el gateway.
- **Simplificación de la comunicación**: Reduce la complejidad de la comunicación entre clientes y servicios.

---

### 3. Service Discovery Pattern

**Descripción:** Los servicios se registran automáticamente en un servidor de descubrimiento y se localizan dinámicamente sin necesidad de configuración estática de URLs, permitiendo que el sistema se adapte automáticamente a cambios en la topología de servicios.

**Tecnología utilizada:** Netflix Eureka, que proporciona un servidor de registro y descubrimiento de servicios.

**Implementación:**
- **Servidor de descubrimiento**: Se implementó un servicio dedicado (`service-discovery`) que actúa como servidor Eureka y mantiene el registro de todos los servicios disponibles.
- **Clientes Eureka**: Todos los microservicios se configuran como clientes Eureka, registrándose automáticamente al iniciar y enviando heartbeats periódicos para indicar que están activos.

**Ubicación:** El servidor de descubrimiento se encuentra en el directorio `service-discovery/`, mientras que la configuración de clientes está presente en los archivos de configuración de cada microservicio.

**Beneficios obtenidos:**
- **Descubrimiento dinámico de servicios**: Los servicios se encuentran automáticamente sin necesidad de configuración manual.
- **Eliminación de configuración estática**: No es necesario hardcodear URLs de servicios en la configuración.
- **Alta disponibilidad**: El sistema puede adaptarse automáticamente cuando servicios se agregan o eliminan.
- **Balanceo de carga automático**: El service discovery integra balanceo de carga entre múltiples instancias del mismo servicio.

---

### 4. Configuration Server Pattern

**Descripción:** Se implementó un servidor de configuración centralizada que permite gestionar la configuración de todos los microservicios desde un punto único, facilitando cambios de configuración sin necesidad de recompilar o redesplegar servicios.

**Tecnología utilizada:** Spring Cloud Config Server, que proporciona un servidor de configuración centralizada con soporte para múltiples repositorios y perfiles.

**Implementación:**
- **Servidor de configuración**: Se implementó un servicio dedicado (`cloud-config`) que actúa como servidor de configuración.
- **Perfiles de entorno**: Se configuraron diferentes perfiles (dev, stage, prod) para gestionar configuraciones específicas por entorno.
- **Integración con servicios**: Todos los microservicios se configuran para obtener su configuración desde el servidor de configuración de forma opcional, manteniendo compatibilidad con configuración local.

**Ubicación:** El servidor de configuración se encuentra en el directorio `cloud-config/` del proyecto.

**Beneficios obtenidos:**
- **Configuración centralizada**: Todas las configuraciones se gestionan desde un punto único, facilitando su mantenimiento.
- **Cambios sin recompilación**: Los cambios de configuración pueden aplicarse sin necesidad de recompilar los servicios.
- **Gestión de múltiples entornos**: Permite mantener configuraciones diferentes para desarrollo, staging y producción de forma organizada.
- **Versionado de configuración**: La configuración puede versionarse junto con el código, permitiendo rastrear cambios históricos.

---

## Patrones de Diseño

### 1. Repository Pattern

**Descripción:** Se abstrajo la lógica de acceso a datos mediante interfaces que proporcionan una capa orientada a objetos para interactuar con la base de datos, separando la lógica de negocio de los detalles de implementación de persistencia.

**Tecnología utilizada:** Spring Data JPA, que proporciona una implementación automática del patrón Repository basada en interfaces.

**Implementación:**
- **Interfaces Repository**: Se crearon interfaces que extienden `JpaRepository` para cada entidad del dominio, proporcionando operaciones CRUD básicas automáticamente.
- **Queries personalizadas**: Se implementaron queries personalizadas utilizando anotaciones `@Query` y JPQL cuando fue necesario realizar operaciones más complejas.
- **Separación de capas**: Las clases de servicio utilizan los repositorios sin conocer los detalles de implementación de persistencia.

**Ubicación:** Las interfaces de repositorio se encuentran en el directorio `repository/` de cada microservicio.

**Beneficios obtenidos:**
- **Abstracción de la capa de persistencia**: La lógica de negocio no depende directamente de la implementación de base de datos.
- **Queries reutilizables**: Las queries definidas en los repositorios pueden reutilizarse en múltiples lugares.
- **Testing simplificado**: Los repositorios pueden mockearse fácilmente en pruebas unitarias.
- **Separación de responsabilidades**: Se mantiene una clara separación entre la lógica de negocio y el acceso a datos.

---

### 2. Service Layer Pattern

**Descripción:** Se implementó una capa de servicios que encapsula la lógica de negocio, separándola de los controladores (capa de presentación) y los repositorios (capa de persistencia).

**Implementación:**
- **Interfaces de servicio**: Se definieron interfaces que establecen los contratos de los servicios, permitiendo diferentes implementaciones.
- **Clases de servicio**: Se implementaron clases anotadas con `@Service` que contienen la lógica de negocio y utilizan inyección de dependencias para acceder a repositorios.
- **Gestión transaccional**: Se utilizó la anotación `@Transactional` para gestionar transacciones de base de datos de forma declarativa.
- **Inyección de dependencias**: Se utilizó inyección por constructor con `@RequiredArgsConstructor` de Lombok para mantener las dependencias inmutables.

**Ubicación:** Las clases de servicio se encuentran en el directorio `service/` de cada microservicio.

**Beneficios obtenidos:**
- **Separación de responsabilidades**: La lógica de negocio está claramente separada de la presentación y persistencia.
- **Reutilización de lógica**: La lógica de negocio puede reutilizarse desde diferentes controladores o servicios.
- **Testing facilitado**: La lógica de negocio puede probarse de forma aislada sin necesidad de levantar toda la aplicación.
- **Transacciones gestionadas**: Las transacciones de base de datos se gestionan automáticamente por el framework.

---

### 3. DTO (Data Transfer Object) Pattern

**Descripción:** Se implementaron objetos de transferencia de datos que transportan información entre capas sin exponer la estructura interna de las entidades de dominio, proporcionando control sobre qué datos se exponen en las APIs.

**Implementación:**
- **Clases DTO**: Se crearon clases DTO inmutables o con builders para cada entidad que necesita ser transferida entre capas.
- **Serialización**: Se utilizaron anotaciones de Jackson para controlar la serialización JSON de los DTOs.
- **Mapeo entre capas**: Se implementaron clases helper o métodos de mapeo para convertir entre entidades de dominio y DTOs, manteniendo la separación de capas.

**Ubicación:** Las clases DTO se encuentran en el directorio `dto/` de cada microservicio.

**Beneficios obtenidos:**
- **Desacoplamiento de entidades**: Las entidades de dominio no se exponen directamente en las APIs, permitiendo cambios internos sin afectar a los clientes.
- **Control de datos expuestos**: Se tiene control granular sobre qué información se expone en cada endpoint.
- **Versionado de APIs**: Los DTOs facilitan la evolución de las APIs sin romper compatibilidad.
- **Optimización de transferencia**: Solo se transfieren los datos necesarios, reduciendo el tamaño de las respuestas.

---

### 4. Builder Pattern

**Descripción:** Se utilizó el patrón Builder para construir objetos complejos de manera fluida y legible, especialmente útil para crear DTOs y respuestas de error con múltiples campos.

**Tecnología utilizada:** Lombok `@Builder`, que genera automáticamente la implementación del patrón Builder.

**Implementación:**
- **Uso en DTOs**: Los DTOs utilizan el builder para facilitar su construcción con múltiples campos opcionales.
- **Respuestas de error**: Las respuestas de error estandarizadas se construyen utilizando builders para mantener consistencia.
- **Configuración**: Se utilizó en clases de configuración donde se necesitaba construir objetos con múltiples parámetros.

**Ubicación:** El patrón Builder se utiliza en múltiples clases a través de la anotación `@Builder` de Lombok.

**Beneficios obtenidos:**
- **Construcción flexible**: Permite construir objetos con diferentes combinaciones de campos de forma clara.
- **Código más legible**: La construcción de objetos es más legible que usar constructores con múltiples parámetros.
- **Inmutabilidad opcional**: Los objetos construidos pueden ser inmutables si se requiere.
- **Validación durante construcción**: Permite validar que los objetos se construyan correctamente.

---

### 5. Dependency Injection Pattern

**Descripción:** Se implementó inversión de control mediante inyección de dependencias, donde las dependencias se proporcionan a las clases en lugar de ser creadas directamente por ellas.

**Framework utilizado:** Spring Framework, que proporciona un contenedor de inversión de control completo.

**Implementación:**
- **Método de inyección**: Se utilizó inyección por constructor como método preferido, ya que permite dependencias inmutables y facilita el testing.
- **Anotaciones**: Se utilizó `@RequiredArgsConstructor` de Lombok para generar automáticamente constructores con las dependencias marcadas como `final`.
- **Gestión del ciclo de vida**: Spring gestiona automáticamente el ciclo de vida de los objetos y sus dependencias.

**Ubicación:** El patrón se aplica en todas las clases de servicio y controladores del proyecto.

**Beneficios obtenidos:**
- **Bajo acoplamiento**: Las clases no dependen de implementaciones concretas, sino de abstracciones.
- **Testabilidad mejorada**: Las dependencias pueden inyectarse fácilmente en pruebas unitarias mediante mocks.
- **Flexibilidad**: Permite cambiar implementaciones sin modificar el código que las utiliza.
- **Gestión de ciclo de vida**: El framework gestiona automáticamente la creación y destrucción de objetos.

---

## Patrones de Comunicación

### 1. REST API Pattern

**Descripción:** Se implementó comunicación entre servicios utilizando HTTP y principios REST, proporcionando una interfaz estándar y ampliamente adoptada para la comunicación entre componentes del sistema.

**Implementación:**
- **Controladores REST**: Se implementaron controladores anotados con `@RestController` que exponen endpoints HTTP.
- **Métodos HTTP**: Se utilizaron los métodos estándar HTTP (GET, POST, PUT, DELETE, PATCH) según la semántica de cada operación.
- **Formato de datos**: Se utilizó JSON como formato estándar para el intercambio de datos entre servicios.

**Ubicación:** Los controladores REST se encuentran en los directorios `controller/` o `resource/` de cada microservicio.

**Beneficios obtenidos:**
- **Estándar ampliamente adoptado**: REST es un estándar conocido que facilita la integración con otros sistemas.
- **Stateless**: Las peticiones son independientes, lo que facilita el escalado horizontal.
- **Cacheable**: Las respuestas pueden cachearse según los headers HTTP estándar.
- **Independiente de plataforma**: Puede utilizarse desde cualquier plataforma que soporte HTTP.

---

### 2. Feign Client Pattern

**Descripción:** Se implementaron clientes HTTP declarativos que simplifican las llamadas entre microservicios, reduciendo el código boilerplate necesario para realizar peticiones HTTP.

**Tecnología utilizada:** Spring Cloud OpenFeign, que proporciona clientes HTTP declarativos basados en interfaces.

**Implementación:**
- **Interfaces Feign**: Se crearon interfaces anotadas con `@FeignClient` que definen los contratos de comunicación con otros servicios.
- **Integración con Service Discovery**: Los clientes Feign se integran automáticamente con Eureka para descubrir servicios dinámicamente.
- **Configuración centralizada**: La configuración de los clientes (timeouts, retries, etc.) se centraliza en archivos de configuración.

**Ubicación:** Los clientes Feign se encuentran principalmente en el `proxy-client`, donde se realizan llamadas a múltiples microservicios.

**Beneficios obtenidos:**
- **Código declarativo**: Las interfaces definen claramente qué servicios se consumen y cómo.
- **Integración con service discovery**: Los servicios se descubren automáticamente sin configuración estática.
- **Manejo automático de errores**: Feign proporciona manejo de errores integrado que puede personalizarse.
- **Configuración centralizada**: La configuración de los clientes se gestiona de forma centralizada.

---

### 3. Load Balanced RestTemplate

**Descripción:** Se implementó un cliente HTTP con balanceo de carga integrado para comunicación entre servicios, permitiendo distribuir la carga entre múltiples instancias del mismo servicio.

**Tecnología utilizada:** Spring Cloud LoadBalancer, que proporciona balanceo de carga integrado con service discovery.

**Implementación:**
- **RestTemplate balanceado**: Se configuró un `RestTemplate` con la anotación `@LoadBalanced` que integra balanceo de carga automático.
- **Integración con Eureka**: El LoadBalancer utiliza Eureka para descubrir instancias disponibles de cada servicio.
- **Distribución de carga**: Las peticiones se distribuyen automáticamente entre las instancias disponibles.

**Ubicación:** La configuración del RestTemplate balanceado se encuentra en clases de configuración de los servicios que requieren comunicación HTTP.

**Beneficios obtenidos:**
- **Balanceo de carga automático**: La carga se distribuye automáticamente sin configuración manual.
- **Integración con service discovery**: Utiliza el service discovery para encontrar instancias disponibles.
- **Resiliencia mejorada**: Si una instancia falla, las peticiones se redirigen automáticamente a otras instancias.
- **Configuración simple**: La configuración es mínima gracias a la integración con Spring Cloud.

---

## Patrones de Resiliencia

### 1. Circuit Breaker Pattern

**Descripción:** Se implementó un circuit breaker que previene fallos en cascada al abrir el circuito cuando un servicio falla repetidamente, protegiendo el sistema y permitiendo su recuperación automática.

**Tecnología utilizada:** Resilience4j, una biblioteca de resiliencia para Java que proporciona implementación de circuit breakers.

**Implementación:**
- **Configuración en API Gateway**: El circuit breaker se configuró en el API Gateway para proteger las llamadas a microservicios downstream.
- **Estados del circuito**: El circuit breaker opera en tres estados: CLOSED (funcionamiento normal), OPEN (circuito abierto, rechaza peticiones) y HALF_OPEN (permite algunas peticiones para probar recuperación).
- **Umbrales configurables**: Se configuraron umbrales de tasa de fallos, número mínimo de llamadas y duración de espera antes de intentar recuperación.

**Ubicación:** La configuración del circuit breaker se encuentra en los archivos de configuración del API Gateway.

**Beneficios obtenidos:**
- **Prevención de fallos en cascada**: Evita que un servicio fallido afecte a todo el sistema.
- **Recuperación automática**: El circuito se cierra automáticamente cuando el servicio se recupera.
- **Protección de recursos**: Evita que se consuman recursos en peticiones que probablemente fallarán.
- **Mejor experiencia de usuario**: Proporciona respuestas rápidas (fallback) en lugar de timeouts largos.

---

### 2. Retry Pattern

**Descripción:** Se implementó un mecanismo de reintentos automáticos para operaciones que fallan, utilizando estrategias de backoff exponencial para mejorar la resiliencia ante fallos temporales.

**Tecnología utilizada:** Spring Cloud Gateway Retry Filter (nativo) combinado con Resilience4j.

**Implementación:**
- **Configuración en API Gateway**: El patrón de reintentos se configuró en el API Gateway para las rutas hacia microservicios.
- **Estrategia de backoff**: Se implementó backoff exponencial que aumenta el tiempo de espera entre reintentos (50ms, 100ms, 200ms, etc.).
- **Selectividad**: Solo se reintentan excepciones específicas (errores 5xx, timeouts) y no errores de negocio (4xx).
- **Número configurable de intentos**: Se configuró un número máximo de reintentos (típicamente 3) antes de considerar la operación como fallida.

**Ubicación:** La configuración de reintentos se encuentra en los archivos de configuración del API Gateway.

**Beneficios obtenidos:**
- **Recuperación automática de fallos temporales**: Los fallos transitorios (red, timeouts) se resuelven automáticamente sin intervención manual.
- **Backoff exponencial**: Evita sobrecargar servicios que se están recuperando al aumentar el tiempo entre reintentos.
- **Configuración específica por tipo de excepción**: Solo reintenta errores que tienen probabilidad de resolverse con reintentos.
- **Mejora de disponibilidad**: Reduce significativamente la tasa de errores percibidos por usuarios ante fallos temporales.

---

### 3. Timeout Pattern

**Descripción:** Se implementaron límites de tiempo máximo para la ejecución de operaciones, previniendo bloqueos indefinidos y mejorando la responsividad del sistema.

**Tecnología utilizada:** Resilience4j TimeLimiter, que proporciona control de timeouts para operaciones asíncronas y síncronas.

**Implementación:**
- **Configuración en API Gateway**: Se configuró un timeout de 5 segundos para las operaciones del API Gateway, asegurando respuestas rápidas.
- **Cancelación automática**: Las operaciones que exceden el timeout se cancelan automáticamente, liberando recursos.
- **Integración con circuit breaker**: Los timeouts frecuentes pueden abrir el circuit breaker para proteger el sistema.

**Ubicación:** La configuración de timeouts se encuentra en los archivos de configuración del API Gateway.

**Beneficios obtenidos:**
- **Prevención de bloqueos indefinidos**: Evita que operaciones lentas o colgadas consuman recursos indefinidamente.
- **Mejora de responsividad**: Garantiza que los servicios respondan dentro de tiempos razonables.
- **Cancelación automática**: Libera threads, conexiones y memoria atrapados en operaciones bloqueadas.
- **Mejor experiencia de usuario**: Los usuarios reciben respuestas rápidas o errores claros, no esperas indefinidas.

---

### 4. Health Check Pattern

**Descripción:** Se implementaron endpoints de health check que permiten verificar el estado de salud de los servicios, facilitando la integración con orquestadores como Kubernetes y sistemas de monitoreo.

**Tecnología utilizada:** Spring Boot Actuator, que proporciona endpoints de salud y métricas de forma automática.

**Implementación:**
- **Endpoint de salud**: Se expone el endpoint `/actuator/health` en todos los microservicios.
- **Integración con Kubernetes**: Se configuraron liveness y readiness probes en los deployments de Kubernetes que utilizan estos endpoints.
- **Health checks específicos**: Se configuraron health checks para verificar el estado de dependencias críticas (base de datos, servicios externos).

**Ubicación:** La configuración de health checks se encuentra en los archivos de configuración de Kubernetes y en la configuración de Actuator de cada servicio.

**Beneficios obtenidos:**
- **Monitoreo de salud**: Permite verificar el estado de los servicios de forma programática.
- **Orquestación mejorada**: Kubernetes puede reiniciar pods no saludables automáticamente.
- **Detección temprana de problemas**: Los problemas se detectan antes de que afecten a los usuarios.
- **Integración con plataformas**: Facilita la integración con sistemas de monitoreo y alertas.

---

## Patrones de Observabilidad

### 1. Distributed Tracing Pattern

**Descripción:** Se implementó rastreo distribuido de peticiones a través de múltiples servicios, permitiendo seguir el flujo completo de una petición desde el cliente hasta todos los servicios involucrados.

**Tecnología utilizada:** Spring Cloud Sleuth para la generación de trazas y Zipkin para su visualización.

**Implementación:**
- **Generación automática de trazas**: Spring Cloud Sleuth genera automáticamente trace IDs y span IDs para cada petición.
- **Propagación de trazas**: Los trace IDs se propagan automáticamente entre servicios mediante headers HTTP.
- **Visualización**: Se implementó un servidor Zipkin que recopila y visualiza las trazas de todos los servicios.
- **Sampling configurable**: Se configuró la probabilidad de muestreo para controlar el volumen de trazas generadas.

**Ubicación:** La configuración de tracing se encuentra en los archivos de configuración de cada microservicio, mientras que el servidor Zipkin se despliega como un servicio independiente.

**Beneficios obtenidos:**
- **Visibilidad completa de peticiones**: Permite ver el flujo completo de una petición a través de todos los servicios.
- **Debugging facilitado**: Facilita identificar dónde ocurren problemas en peticiones distribuidas.
- **Análisis de rendimiento**: Permite identificar cuellos de botella y servicios lentos en el flujo de peticiones.
- **Identificación de dependencias**: Ayuda a entender las dependencias entre servicios y su impacto en el rendimiento.

---

### 2. Logging Pattern

**Descripción:** Se implementó un sistema de logging estructurado que registra eventos y errores en toda la aplicación, facilitando el debugging y la auditoría.

**Tecnología utilizada:** SLF4J como fachada de logging y Logback como implementación.

**Implementación:**
- **Logging estructurado**: Se utilizó logging estructurado con anotación `@Slf4j` de Lombok en todas las clases de servicio.
- **Niveles de log**: Se configuraron diferentes niveles de log (INFO, WARN, ERROR) según el entorno.
- **Trace IDs en logs**: Los trace IDs generados por Sleuth se incluyen automáticamente en los logs para correlación.
- **Centralización de logs**: Los logs se centralizan mediante Filebeat y Elasticsearch para búsqueda y análisis.

**Ubicación:** El logging se utiliza en todas las clases de servicio del proyecto, mientras que la configuración de Logback se encuentra en los archivos de recursos de cada servicio.

**Beneficios obtenidos:**
- **Debugging facilitado**: Los logs estructurados facilitan la identificación y resolución de problemas.
- **Auditoría**: Permite rastrear acciones y cambios en el sistema para propósitos de auditoría.
- **Monitoreo**: Los logs pueden analizarse para detectar patrones y problemas.
- **Correlación de eventos**: Los trace IDs permiten correlacionar logs de múltiples servicios relacionados con la misma petición.

---

## Patrones de Manejo de Errores

### 1. Global Exception Handler Pattern

**Descripción:** Se implementó un manejador global de excepciones que centraliza el manejo de errores en toda la aplicación, proporcionando respuestas de error consistentes y estandarizadas.

**Implementación:**
- **Clase manejadora**: Se creó una clase anotada con `@RestControllerAdvice` que captura todas las excepciones no manejadas.
- **Manejo específico por tipo**: Se implementaron métodos `@ExceptionHandler` para diferentes tipos de excepciones (ResourceNotFoundException, DuplicateResourceException, etc.).
- **Respuestas estandarizadas**: Todas las excepciones se convierten en respuestas `ErrorResponse` con formato consistente que incluye timestamp, código de error, mensaje y trace ID.
- **Logging centralizado**: Todas las excepciones se registran en los logs con información contextual.

**Ubicación:** El manejador global de excepciones se encuentra en el directorio `exception/` de cada microservicio.

**Beneficios obtenidos:**
- **Manejo consistente de errores**: Todas las excepciones se manejan de la misma forma en toda la aplicación.
- **Respuestas estandarizadas**: Los clientes reciben respuestas de error con formato consistente, facilitando su manejo.
- **Logging centralizado**: Todas las excepciones se registran de forma centralizada, facilitando el debugging.
- **Mejor experiencia de usuario**: Los usuarios reciben mensajes de error claros y útiles en lugar de errores genéricos.

---

### 2. Custom Exception Pattern

**Descripción:** Se implementaron excepciones personalizadas específicas del dominio que proporcionan contexto semántico y códigos de error estandarizados, mejorando la claridad del manejo de errores.

**Implementación:**
- **Excepciones personalizadas**: Se crearon excepciones específicas como `ResourceNotFoundException`, `DuplicateResourceException`, `InvalidInputException` y `ExternalServiceException`.
- **Códigos de error**: Cada excepción incluye un código de error estandarizado que identifica el tipo de problema.
- **Mensajes contextuales**: Las excepciones incluyen mensajes descriptivos que explican el problema de forma clara.
- **Manejo específico**: El manejador global de excepciones puede manejar cada tipo de excepción de forma específica.

**Ubicación:** Las excepciones personalizadas se encuentran en el directorio `exception/custom/` de cada microservicio.

**Beneficios obtenidos:**
- **Semántica clara**: Las excepciones tienen nombres que describen claramente el tipo de problema.
- **Códigos de error estandarizados**: Los códigos de error permiten a los clientes manejar errores de forma programática.
- **Manejo específico por tipo**: Cada tipo de excepción puede manejarse de forma diferente según las necesidades.
- **Mejor debugging**: Los nombres descriptivos de las excepciones facilitan la identificación de problemas en los logs.

---

## Patrones de Configuración

### 1. Externalized Configuration Pattern

**Descripción:** Se implementó configuración externa a la aplicación, permitiendo cambios de configuración sin necesidad de recompilar o redesplegar los servicios.

**Implementación:**
- **Archivos de configuración**: Se utilizaron archivos YAML (`application.yml`, `application-{profile}.yml`) para definir la configuración.
- **Variables de entorno**: Se implementó soporte completo para variables de entorno que pueden sobrescribir valores de configuración.
- **Integración con Config Server**: Los servicios pueden obtener configuración desde el servidor de configuración centralizada de forma opcional.
- **Perfiles múltiples**: Se configuraron diferentes perfiles (dev, stage, prod) para gestionar configuraciones específicas por entorno.

**Ubicación:** Los archivos de configuración se encuentran en el directorio `src/main/resources/` de cada microservicio.

**Beneficios obtenidos:**
- **Configuración flexible**: Los cambios de configuración pueden aplicarse sin modificar código.
- **Múltiples entornos**: Permite mantener configuraciones diferentes para desarrollo, staging y producción.
- **Seguridad mejorada**: Las configuraciones sensibles pueden gestionarse mediante variables de entorno o secretos.
- **Sin recompilación**: Los cambios de configuración no requieren recompilar la aplicación.

---

### 2. Environment-based Configuration

**Descripción:** Se implementó configuración específica por entorno utilizando perfiles de Spring, permitiendo que la misma aplicación se comporte de forma diferente según el entorno en el que se ejecute.

**Implementación:**
- **Perfiles de Spring**: Se configuraron perfiles específicos (dev, stage, prod) que activan diferentes configuraciones.
- **Archivos por perfil**: Se crearon archivos de configuración específicos (`application-dev.yml`, `application-prod.yml`) para cada entorno.
- **Activación de perfiles**: Los perfiles se activan mediante la variable de entorno `SPRING_PROFILES_ACTIVE` o configuración de Kubernetes.

**Ubicación:** Los archivos de configuración por perfil se encuentran en el directorio `src/main/resources/` de cada microservicio.

**Beneficios obtenidos:**
- **Separación de entornos**: Las configuraciones de diferentes entornos están claramente separadas.
- **Configuración específica**: Cada entorno puede tener configuraciones optimizadas para sus necesidades.
- **Seguridad por entorno**: Las configuraciones sensibles pueden variar según el entorno.
- **Despliegue simplificado**: El mismo artefacto puede desplegarse en diferentes entornos cambiando solo el perfil activo.

---

### 3. Feature Toggle Pattern

**Descripción:** Se implementó un sistema de feature toggles (feature flags) que permite habilitar o deshabilitar funcionalidades en tiempo de ejecución mediante configuración, sin necesidad de recompilar o redesplegar la aplicación.

**Tecnología utilizada:** Spring Boot `@ConfigurationProperties` para leer configuración de features desde archivos YAML o variables de entorno.

**Implementación:**
- **Servicio de feature toggles**: Se implementó un `FeatureToggleService` que verifica el estado de las features antes de ejecutar funcionalidades.
- **Configuración externa**: Los estados de las features se configuran en archivos YAML o variables de entorno.
- **Verificación en runtime**: El código verifica el estado del toggle antes de ejecutar funcionalidades nuevas o experimentales.
- **Ubicación estratégica**: Se implementó principalmente en el `proxy-client`, donde se toman decisiones sobre qué funcionalidades exponer.

**Ubicación:** El servicio de feature toggles se encuentra en el directorio `config/feature/` del `proxy-client`, mientras que la configuración está en los archivos de configuración.

**Beneficios obtenidos:**
- **Despliegue continuo sin riesgo**: Permite desplegar código nuevo en producción sin activarlo inmediatamente.
- **Testing A/B y Canary Releases**: Facilita probar nuevas funcionalidades con un subconjunto de usuarios antes de activarlas completamente.
- **Rollback rápido**: Permite desactivar funcionalidades problemáticas cambiando solo la configuración, sin necesidad de redesplegar.
- **Control granular**: Cada feature puede activarse o desactivarse independientemente de otras.
- **Configuración por entorno**: Diferentes features pueden estar activas en diferentes entornos según las necesidades.

---

## Patrones Implementados y Mejorados Durante el Proyecto

Esta sección documenta los patrones de resiliencia y configuración que se implementaron o mejoraron específicamente durante el desarrollo del proyecto.

### Patrones de Resiliencia Mejorados

#### 1. Circuit Breaker Pattern - Fallback Centralizado

**Estado inicial:** El proyecto ya tenía implementado circuit breaker básico con Resilience4j en el API Gateway.

**Mejoras implementadas:**
- **Fallback Controller centralizado**: Se creó un controlador dedicado con endpoints de fallback específicos para cada servicio (order, product, payment, shipping, user, favourite, proxy).
- **Respuestas consistentes**: Cuando el circuit breaker se abre, el sistema responde con HTTP 503 y un payload estructurado que incluye información clara sobre el servicio no disponible.
- **Mejor observabilidad**: Los logs muestran explícitamente qué fallback se invoca, facilitando el diagnóstico cuando múltiples servicios están degradados.


**Beneficios de las mejoras:**
- **Experiencia de usuario mejorada**: Los clientes reciben respuestas informativas en lugar de errores genéricos o nulos.
- **Diagnóstico facilitado**: Los logs explícitos aceleran la identificación de problemas cuando múltiples servicios están degradados.
- **Reducción de carga**: El fallback evita sobrecargar servicios que ya están inestables.

---

#### 2. Retry Pattern

**Estado inicial:** El proyecto no tenía implementado un mecanismo de retry configurado para manejar fallos temporales.

**Implementación:**
- **Filtro Retry de Spring Cloud Gateway**: Se configuró el filtro Retry nativo de Spring Cloud Gateway en todas las rutas del API Gateway para reintentar automáticamente peticiones fallidas.
- **Estrategia de backoff exponencial**: Se configuró un backoff exponencial con un primer retry después de 50ms, máximo de 500ms y factor de multiplicación de 2.
- **Criterios de retry**: Se configuró para reintentar en caso de códigos de estado HTTP específicos: `BAD_GATEWAY`, `GATEWAY_TIMEOUT`, `INTERNAL_SERVER_ERROR`, `SERVICE_UNAVAILABLE`.
- **Métodos HTTP**: Se aplica a los métodos GET, POST, PUT y DELETE.
- **Número de reintentos**: Se configuró para realizar hasta 3 intentos antes de considerar la operación como fallida.

**Ubicación:** Configurado en `api-gateway/src/main/resources/application.yml` en las rutas del API Gateway mediante el filtro `Retry` de Spring Cloud Gateway.

**Beneficios obtenidos:**
- **Resiliencia ante fallos temporales**: Los fallos transitorios de red o servicios se manejan automáticamente sin intervención manual.
- **Mejor experiencia de usuario**: Los usuarios no experimentan errores por fallos temporales que se resuelven en segundos.
- **Reducción de carga operacional**: Menos necesidad de intervención manual para manejar fallos temporales.
- **Backoff inteligente**: El backoff exponencial evita sobrecargar servicios que se están recuperando.

---

#### 3. Timeout Pattern

**Estado inicial:** El proyecto no tenía configurados timeouts explícitos para limitar el tiempo de ejecución de operaciones.

**Implementación:**
- **Resilience4j TimeLimiter**: Se implementó el TimeLimiter de Resilience4j que se integra automáticamente con el Circuit Breaker para establecer un límite máximo de tiempo de ejecución de operaciones.
- **Timeout configurado**: Se estableció un timeout de 5 segundos para todas las operaciones del API Gateway que utilizan el Circuit Breaker.
- **Cancelación automática**: Se configuró para cancelar automáticamente las operaciones que excedan el timeout, liberando recursos del sistema.
- **Integración con Circuit Breaker**: El TimeLimiter trabaja en conjunto con el Circuit Breaker de Resilience4j, de manera que las operaciones que excedan el timeout son registradas como fallos por el Circuit Breaker.

**Ubicación:** Configurado en `api-gateway/src/main/resources/application.yml` en la sección `resilience4j.timelimiter`. Las dependencias de Resilience4j se encuentran en `api-gateway/pom.xml`.

**Beneficios obtenidos:**
- **Prevención de bloqueos indefinidos**: Las operaciones no pueden quedarse bloqueadas indefinidamente esperando respuestas.
- **Mejor responsividad**: El sistema responde rápidamente cuando un servicio está degradado o no responde.
- **Liberación de recursos**: Las operaciones que exceden el timeout se cancelan automáticamente, liberando recursos del sistema.
- **Integración con circuit breaker**: Los timeouts trabajan en conjunto con el circuit breaker para mejorar la resiliencia general del sistema.

---

### Patrones de Configuración Implementados

#### 1. Feature Toggle Pattern

**Descripción:** Se implementó un sistema de feature toggles (feature flags) que permite habilitar o deshabilitar funcionalidades en tiempo de ejecución mediante configuración, sin necesidad de recompilar o redesplegar la aplicación.

**Implementación:**
- **FeatureToggleService**: Se implementó un servicio que verifica el estado de las features antes de ejecutar funcionalidades, proporcionando métodos como `isFeatureEnabled()`, `isFeatureDisabled()` y `requireFeature()`.
- **FeatureToggleProperties**: Se utilizó `@ConfigurationProperties` de Spring Boot para mapear la configuración de features desde archivos YAML o variables de entorno.
- **Configuración externa**: Los estados de las features se configuran mediante la propiedad `features.toggles` en `application.yml` o variables de entorno (ej. `FEATURE_ENHANCED_SEARCH`).
- **Uso en controladores**: Se implementó en el `ProductController` del `proxy-client` para controlar la funcionalidad de búsqueda mejorada (`enhanced-search`), permitiendo activar o desactivar el filtrado avanzado de productos.


**Beneficios obtenidos:**
- **Despliegue continuo sin riesgo**: Permite desplegar código nuevo en producción sin activarlo inmediatamente.
- **Testing A/B y Canary Releases**: Facilita probar nuevas funcionalidades con un subconjunto de usuarios antes de activarlas completamente.
- **Rollback rápido**: Permite desactivar funcionalidades problemáticas cambiando solo la configuración, sin necesidad de redesplegar.
- **Control granular**: Cada feature puede activarse o desactivarse independientemente de otras.
- **Configuración por entorno**: Diferentes features pueden estar activas en diferentes entornos según las necesidades.

---

## Resumen de Patrones Implementados

El proyecto implementó una combinación de patrones arquitectónicos, de diseño, comunicación, resiliencia, observabilidad y configuración que trabajan juntos para crear un sistema robusto, escalable y mantenible. 

**Patrones ya presentes en la arquitectura inicial:**
- Arquitectura de Microservicios
- API Gateway Pattern
- Service Discovery Pattern
- Configuration Server Pattern
- Repository Pattern
- Service Layer Pattern
- DTO Pattern
- Builder Pattern
- Dependency Injection Pattern
- REST API Pattern
- Feign Client Pattern
- Load Balanced RestTemplate
- Circuit Breaker Pattern (básico)
- Health Check Pattern (básico)
- Distributed Tracing Pattern
- Logging Pattern
- Global Exception Handler Pattern
- Custom Exception Pattern
- Externalized Configuration Pattern
- Environment-based Configuration
- Feature Toggle Pattern

**Patrones implementados o mejorados durante el proyecto:**
- Circuit Breaker Pattern - Fallback centralizado (mejora)
- Retry Pattern - Reintentos automáticos con backoff exponencial (implementación)
- Timeout Pattern - Límites de tiempo para operaciones (implementación)
- Feature Toggle Pattern - Sistema de feature flags (implementación)

La aplicación de estos patrones, tanto los existentes como los mejorados, permitió:

- **Separación clara de responsabilidades** mediante arquitectura de microservicios y capas bien definidas.
- **Resiliencia ante fallos** mediante circuit breakers mejorados con fallback centralizado, retries automáticos con backoff exponencial y timeouts que previenen bloqueos indefinidos.
- **Observabilidad completa** mediante tracing distribuido y logging estructurado.
- **Mantenibilidad mejorada** mediante patrones de diseño que facilitan el testing y la evolución del código.
- **Flexibilidad operacional** mediante configuración externa y feature toggles que permiten control granular de funcionalidades.

La combinación de estos patrones proporciona una base sólida para un sistema de microservicios de producción que puede escalar, evolucionar y mantenerse de forma efectiva.
