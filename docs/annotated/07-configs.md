# Configurações da aplicação

Arquivos:
- `src/main/resources/application.properties`
- `src/main/resources/application-test.properties`

## `application.properties`

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/hexagonal
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
```

**Anotações técnicas**

- Configuração de **DataSource** para MySQL local.
- `ddl-auto=create` recria o schema a cada startup — útil em desenvolvimento, arriscado em produção.
- `show-sql=true` é bom para debugging, mas deve ser evitado em produção por desempenho/logs.

## `application-test.properties`

```properties
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.username=sa
spring.datasource.password=

spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=create
```

**Anotações técnicas**

- Usa H2 em memória para testes, acelerando execução e isolando dados.
- `ddl-auto=create` garante schema limpo por execução de testes.
