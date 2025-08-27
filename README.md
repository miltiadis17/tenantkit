# TenantKit — Spring Boot 3 / Java 17 (Tenant-aware RBAC Starter)

A minimal, production-minded starter for **tenant-aware RBAC** on **Spring Boot 3**/**Java 17** with **JWT (access + refresh)**, **MySQL**, **Liquibase**, **Testcontainers**, and a tiny React/TypeScript admin (login, tenants, users).

## Why
- Convert multi-tenant CRM learnings into a clean **Java/Spring** starter.
- Ship secure **tenant-scoped** APIs with **RBAC** and CI-friendly tests.
- Be explicit about **JWT rotation**, **clock skew**, and **auditing**.

## Features
- **Auth:** Spring Security, JWT (access/refresh), clock-skew tolerance, rotation on password change.
- **Tenant Scope:** tenant resolver (from JWT `tenant_id` claim or `X-Tenant-Id` header) + servlet filter.
- **RBAC:** role hierarchy (e.g., `ROLE_ADMIN`, `ROLE_MANAGER`, `ROLE_USER`) with method/endpoint security.
- **Persistence:** MySQL + Liquibase migrations; Flyway variant possible.
- **Docs:** OpenAPI/Swagger UI.
- **Tests:** Integration tests with Testcontainers (MySQL), `@SpringBootTest` slices.
- **DX:** Docker Compose for DB; Makefile/Maven wrapper optional.
- **Tiny Admin (React/TS):** Login, list tenants/users, create/update basics (RTK Query).

## Stack
**Backend:** Java 17, Spring Boot 3, Spring Security, JPA/Hibernate, Liquibase, MySQL  
**Frontend (optional):** React, TypeScript, Redux Toolkit, RTK Query, Vite/Tailwind  
**Tooling:** Docker, Testcontainers, OpenAPI, Maven

## Quickstart
### Prerequisites
- JDK 17+
- Maven 3.9+
- Docker (for MySQL/Testcontainers)

### Run DB
```bash
docker compose up -d
# MySQL on 3306, user/pass via .env (see below)
```

### Configure `application.yml`
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/tenantkit?useSSL=false&allowPublicKeyRetrieval=true
    username: tenantkit
    password: change-me
  jpa:
    hibernate:
      ddl-auto: validate
    properties:
      hibernate:
        format_sql: true
        jdbc:
          time_zone: UTC
  liquibase:
    change-log: classpath:db/changelog/db.changelog-master.yaml

security:
  jwt:
    issuer: "tenantkit"
    audience: "tenantkit-api"
    accessTtlSeconds: 900    # 15m
    refreshTtlSeconds: 2592000 # 30d
    clockSkewSeconds: 60     # tolerate skew
    secret: "replace-with-strong-secret-or-use-RSA-keys"
```

### Dependencies (snippet)
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-j</artifactId>
  <scope>runtime</scope>
</dependency>
<dependency>
  <groupId>org.liquibase</groupId>
  <artifactId>liquibase-core</artifactId>
</dependency>
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.5.0</version>
</dependency>
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>mysql</artifactId>
  <scope>test</scope>
</dependency>
```

### Security config (sketch)
```java
@Bean
SecurityFilterChain security(HttpSecurity http) throws Exception {
    http.csrf(csrf -> csrf.disable())
        .sessionManagement(sm -> sm.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/auth/**","/actuator/**","/swagger-ui/**","/v3/api-docs/**").permitAll()
            .anyRequest().authenticated()
        )
        .addFilterBefore(tenantFilter(), UsernamePasswordAuthenticationFilter.class)
        .oauth2ResourceServer(oauth -> oauth.jwt(jwt -> jwt.decoder(jwtDecoder())));
    return http.build();
}

@Bean
JwtDecoder jwtDecoder() {
    NimbusJwtDecoder decoder = NimbusJwtDecoder.withSecretKey(secretKey()).build();
    decoder.setJwtValidator(new DelegatingOAuth2TokenValidator<>(
        new JwtTimestampValidator(Duration.ofSeconds(clockSkewSeconds)),
        new JwtIssuerValidator(issuer),
        new AudienceValidator(audience)
    ));
    return decoder;
}
```

### Tenant resolver/filter (sketch)
```java
public class TenantFilter extends OncePerRequestFilter {
  @Override
  protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain)
      throws ServletException, IOException {
    String tenantId = resolveFromJwtOrHeader(req); // claim "tenant_id" or header "X-Tenant-Id"
    TenantContext.setTenantId(tenantId);
    try { chain.doFilter(req, res); } finally { TenantContext.clear(); }
  }
}
```

### Example endpoints
- `POST /auth/login` → returns access/refresh tokens  
- `POST /auth/refresh` → rotates tokens  
- `GET /me` → current user (tenant-scoped)  
- `GET /tenants` (ADMIN)  
- `GET /users` (ADMIN/MANAGER)  
- `GET /projects` (scoped); `POST /projects` (role-limited)

### Testing
```java
@Testcontainers
@SpringBootTest
class ProjectApiIT {
  @Container static MySQLContainer<?> db = new MySQLContainer<>("mysql:8.4");
  // ... seed data per-tenant
}
```

### Tiny Admin (React/TS) — optional
- Vite + React + TS + RTK Query
- Pages: Login, Tenants, Users
- `.env` → `VITE_API_URL=http://localhost:8080`

## Roadmap
- MFA (TOTP), audit logging, RSA keypair support
- Schema-per-tenant example with multi-datasource
- More tests + example GitHub Actions

## License
MIT (c) Miltiadis Kiourntzidis
