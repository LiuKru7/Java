# ⚙️ Spring Boot Properties – PostgreSQL & H2 Config

This guide outlines how to configure a Spring Boot project to use PostgreSQL for production and H2 for in-memory testing. It includes examples for both `application.properties` and `application-test.properties`.

---

## 🐘 PostgreSQL Configuration – `application.properties`

Use this configuration for your **default production or development** environment with PostgreSQL.

```properties
# PostgreSQL database connection
spring.datasource.url=jdbc:postgresql://localhost:5432/test
spring.datasource.username=postgres
spring.datasource.password=admin
spring.datasource.driver-class-name=org.postgresql.Driver

# Hibernate configuration
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect

# Application name
spring.application.name=TenantAndProperties

# Logging
spring.output.ansi.enabled=always
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.mapstruct=DEBUG
```

---

## 🧪 H2 In-Memory Database – `application-test.properties`

Use this configuration for **unit/integration testing** with an in-memory H2 database.

```properties
# H2 in-memory database
spring.datasource.url=jdbc:h2:mem:tenant
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=

# Hibernate for H2
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true

# Enable H2 web console
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console

# Logging
spring.output.ansi.enabled=always
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.mapstruct=DEBUG
```

---

## ✅ Best Practices

- Use **H2** with `@SpringBootTest` for faster, isolated integration testing.
- Keep production and test configs in separate files using `@ActiveProfiles("test")`.
- Set `ddl-auto=create-drop` for test environments, and use `validate` or `update` in production (if schema is stable).
- Enable `h2-console` for easier debugging during test runs.

---

## 🧠 Profile Switching

Use different profiles to load the correct configuration automatically:

```bash
# For default (PostgreSQL)
mvn spring-boot:run

# For test (H2)
mvn test -Dspring.profiles.active=test
```
