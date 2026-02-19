# LiterAlura — Plataforma de Gestión Bibliográfica

> Aplicación de consola construida con **Spring Boot** que conecta la biblioteca digital de Project Gutenberg (vía API Gutendex) con una base de datos relacional local en **PostgreSQL**.

**Desarrollado por:** Daniel Flores  
**Stack:** Java 17 · Spring Boot 3.2.4 · Spring Data JPA · PostgreSQL · Jackson

---

## 01 · Descripción General

LiterAlura va más allá de una simple descarga de información: implementa lógica de negocio para evitar duplicados, normalizar entidades de autores y libros, y ofrecer consultas estadísticas e históricas que transforman datos crudos en conocimiento útil directamente desde la consola.

La API consumida es **[Gutendex](https://gutendex.com)**, basada en el proyecto Gutenberg con más de 70.000 libros disponibles.

---

## 02 · Funcionalidades del Sistema

### 🔍 1. Búsqueda con Persistencia Inteligente
El sistema aplica una estrategia en dos etapas:
- Primero consulta **PostgreSQL** local — si el libro existe, lo devuelve sin llamadas de red
- Si no existe, consulta la **API Gutendex**, parsea el JSON y persiste el resultado
- Al guardar, verifica si el autor ya está registrado para evitar duplicados

### 📚 2. Inventario Bibliográfico
Lista todos los libros registrados con título, autor vinculado, idioma y número de descargas.

### ✍️ 3. Directorio de Autores
Vista dedicada a autores sin duplicados, con fechas de vida, respetando la integridad referencial de la relación Many-to-Many.

### 📅 4. Autores Vivos en un Año Específico
Ingresa cualquier año y el sistema ejecuta una consulta JPQL que cruza los rangos de vida de los autores para responder: *¿qué escritores estaban vivos en 1850?*

### 🌐 5. Filtro por Idioma
Segmenta la biblioteca por código internacional: `es`, `en`, `fr`, `pt`, `de`, `ru`, entre otros.

### 📊 6. Estadísticas con Java Streams
Procesa la colección completa y calcula en tiempo real:

| Métrica | Descripción |
|---|---|
| Promedio de descargas | Popularidad media de la biblioteca |
| Máximo de descargas | Libro más popular del catálogo |
| Mínimo de descargas | Libro con menor impacto registrado |
| Total de libros | Cantidad exacta de registros |

### 🏆 7. Top 10 — Ranking por Popularidad
Consulta ordenada que retorna los diez libros con mayor número de descargas.

### 🔎 8. Búsqueda Específica de Autores
Localiza autores por coincidencia parcial de nombre (`findByNombreContainsIgnoreCase`) o por año de nacimiento.

---

## 03 · Arquitectura del Proyecto

```
literalura/
├── src/main/java/com/alura/literalura/
│   ├── model/
│   │   ├── Autor.java              → Entidad JPA con fechas de vida
│   │   ├── Libro.java              → Entidad JPA con relación @ManyToMany
│   │   ├── DatosAutor.java         → Record para parseo JSON
│   │   └── DatosLibro.java         → Record para parseo JSON
│   ├── repository/
│   │   ├── AutorRepository.java
│   │   └── LibroRepository.java
│   ├── service/
│   │   ├── ConsumoAPI.java         → Cliente HTTP nativo de Java
│   │   ├── ConvierteDatos.java     → Deserializador con Jackson
│   │   └── IConvierteDatos.java    → Interfaz genérica <T>
│   ├── principal/
│   │   └── Principal.java          → Menú y flujo de control
│   └── LiteraluraApplication.java
└── src/main/resources/
    └── application.properties
```

---

## 04 · Decisiones Técnicas

| Decisión | Justificación |
|---|---|
| **Búsqueda híbrida BD → API** | Verifica localmente antes de hacer llamadas externas, reduciendo latencia y consumo de red |
| **Java Records (DTOs)** | Estructuras inmutables para mapear el JSON de Gutendex, desacoplando el modelo externo del dominio interno |
| **Interfaz genérica `IConvierteDatos<T>`** | Reutilización del motor de deserialización Jackson para cualquier modelo futuro |
| **`Optional<T>`** | Eliminación segura de `NullPointerException` en consultas que pueden no retornar resultados |
| **Java Streams API** | Procesamiento declarativo para filtros, ordenamientos y métricas estadísticas |
| **Relación `@ManyToMany`** | Un libro puede tener varios autores y un autor varios libros, con tabla intermedia `libro_autor` |
| **Variables de entorno** | Las credenciales nunca se hardcodean: se inyectan vía `${DB_HOST}`, `${DB_USER}`, etc. |

---

## 05 · Configuración

### Dependencias principales (`pom.xml`)

```xml
<!-- Spring Data JPA -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- PostgreSQL Driver -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

### `application.properties`

```properties
# Conexión segura mediante variables de entorno
spring.datasource.url=jdbc:postgresql://${DB_HOST:localhost}:5432/${DB_NAME}
spring.datasource.username=${DB_USER}
spring.datasource.password=${DB_PASSWORD}

# Hibernate / JPA
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
```

> **Seguridad:** El uso de `${VARIABLES}` garantiza que las credenciales no queden expuestas en el repositorio. Configúralas como variables de entorno del sistema o en IntelliJ en `Run > Edit Configurations > Environment Variables`.

---

## 06 · Cómo Ejecutar

1. Clona el repositorio y ábrelo en **IntelliJ IDEA** como proyecto Maven
2. Instala PostgreSQL y crea la base de datos:
   ```sql
   CREATE DATABASE literalura;
   ```
3. Define las variables de entorno `DB_HOST`, `DB_NAME`, `DB_USER` y `DB_PASSWORD`
4. Ejecuta `LiteraluraApplication.java` — Spring Boot levantará el contexto y creará las tablas automáticamente
5. Interactúa con el menú en la consola (opciones 1–8, ingresa `0` para salir)

---

## 07 · Hoja de Ruta

- [ ] Interfaz web con Spring MVC para una experiencia visual más rica
- [ ] Búsqueda directa por idioma en Gutendex antes de la persistencia local
- [ ] Sistema de recomendaciones basado en autores y géneros más consultados
- [ ] Exportación de datos en formato PDF o CSV
- [ ] Dashboard estadístico con gráficos de distribución por idioma y popularidad
- [ ] Autenticación de usuarios para múltiples catálogos personales

---

*Desarrollado por **Daniel Flores** — Desafío Alura LATAM · Spring Boot 3.2.4 · Java 17*



