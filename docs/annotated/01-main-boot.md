# Bootstrap da aplicação (`Main.java`)

Arquivo: `src/main/java/br/com/fullcycle/hexagonal/Main.java`

```java
package br.com.fullcycle.hexagonal;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Main {

    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }

}
```

## Anotações técnicas

- `@SpringBootApplication` agrega três anotações: `@Configuration`, `@EnableAutoConfiguration` e `@ComponentScan`.
  - **Boa prática**: centralizar o bootstrap num único ponto, permitindo o auto-scan das camadas (controllers, services, repositories).
- `SpringApplication.run(...)` inicializa o contexto Spring, cria o **IoC Container** e publica os beans.
- Esse arquivo representa o **ponto de entrada** da aplicação, equivalente ao _adapter_ que liga a infraestrutura (runtime JVM + Spring) ao domínio.
