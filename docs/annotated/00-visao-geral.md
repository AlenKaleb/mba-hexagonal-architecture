# Visão geral anotada da arquitetura

Esta exportação descreve o código da branch atual em arquivos Markdown com anotações técnicas. O sistema é um serviço Spring Boot organizado em camadas que refletem conceitos de **arquitetura hexagonal** (ports & adapters), embora esteja implementado com uma estrutura clássica de controllers/services/repositories.

## Conceitos-chave aplicados

- **Camada de entrada (adapters de entrada)**: controllers REST em `src/main/java/br/com/fullcycle/hexagonal/controllers`, responsáveis por traduzir HTTP em comandos do domínio.
- **Camada de aplicação (use cases)**: serviços em `src/main/java/br/com/fullcycle/hexagonal/services`, onde ficam as regras de orquestração e transações.
- **Camada de domínio**: entidades JPA em `src/main/java/br/com/fullcycle/hexagonal/models`, representando o estado e regras fundamentais do negócio.
- **Camada de saída (adapters de saída)**: repositórios Spring Data em `src/main/java/br/com/fullcycle/hexagonal/repositories`, implementando persistência como um detalhe de infraestrutura.
- **DTOs**: objetos de transporte em `src/main/java/br/com/fullcycle/hexagonal/dtos` para isolar a API pública do modelo interno.

## Padrões e boas práticas observadas

- **Dependency Injection** com `@Autowired` para acoplar implementações sem instância manual.
- **Repository Pattern** com Spring Data (`CrudRepository`) para abstrair acesso a dados.
- **DTO Pattern** para evitar exposição direta das entidades.
- **Transações** com `@Transactional` nos serviços, mantendo consistência ao salvar entidades relacionadas.
- **Validações de integridade** em controllers (ex.: evitar duplicidade de e-mail/CPF).

## Mapeamento rápido

- Bootstrap: `src/main/java/br/com/fullcycle/hexagonal/Main.java`
- Controllers: `src/main/java/br/com/fullcycle/hexagonal/controllers/*`
- Services: `src/main/java/br/com/fullcycle/hexagonal/services/*`
- Models: `src/main/java/br/com/fullcycle/hexagonal/models/*`
- DTOs: `src/main/java/br/com/fullcycle/hexagonal/dtos/*`
- Repositories: `src/main/java/br/com/fullcycle/hexagonal/repositories/*`
- Configs: `src/main/resources/application*.properties`
- Tests: `src/test/java/br/com/fullcycle/hexagonal/*`
