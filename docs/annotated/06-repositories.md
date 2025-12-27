# Repositórios (adapters de saída)

Arquivos:
- `src/main/java/br/com/fullcycle/hexagonal/repositories/CustomerRepository.java`
- `src/main/java/br/com/fullcycle/hexagonal/repositories/EventRepository.java`
- `src/main/java/br/com/fullcycle/hexagonal/repositories/PartnerRepository.java`
- `src/main/java/br/com/fullcycle/hexagonal/repositories/TicketRepository.java`

## `CustomerRepository`

```java
package br.com.fullcycle.hexagonal.repositories;

import br.com.fullcycle.hexagonal.models.Customer;
import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface CustomerRepository extends CrudRepository<Customer, Long> {

    Optional<Customer> findByCpf(String cpf);

    Optional<Customer> findByEmail(String email);
}
```

**Anotações técnicas**

- `CrudRepository` fornece CRUD básico sem implementação manual.
- Métodos `findByCpf` e `findByEmail` são **queries derivadas** do Spring Data.

## `EventRepository`

```java
package br.com.fullcycle.hexagonal.repositories;

import br.com.fullcycle.hexagonal.models.Event;
import org.springframework.data.repository.CrudRepository;

public interface EventRepository extends CrudRepository<Event, Long> {

}
```

**Anotações técnicas**

- Interface vazia, usando apenas os métodos CRUD padrão.

## `PartnerRepository`

```java
package br.com.fullcycle.hexagonal.repositories;

import br.com.fullcycle.hexagonal.models.Partner;
import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface PartnerRepository extends CrudRepository<Partner, Long> {

    Optional<Partner> findByCnpj(String cnpj);

    Optional<Partner> findByEmail(String email);
}
```

**Anotações técnicas**

- Reuso do padrão de unicidade com queries derivadas.

## `TicketRepository`

```java
package br.com.fullcycle.hexagonal.repositories;

import br.com.fullcycle.hexagonal.models.Ticket;
import org.springframework.data.repository.CrudRepository;

import java.util.Optional;

public interface TicketRepository extends CrudRepository<Ticket, Long> {

    Optional<Ticket> findByEventIdAndCustomerId(Long id, Long customerId);
}
```

**Anotações técnicas**

- Consulta derivada por relacionamento, usada para impedir múltiplas inscrições de um mesmo cliente em um evento.
