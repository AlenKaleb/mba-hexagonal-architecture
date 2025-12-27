# Serviços de aplicação (use cases)

Arquivos:
- `src/main/java/br/com/fullcycle/hexagonal/services/CustomerService.java`
- `src/main/java/br/com/fullcycle/hexagonal/services/EventService.java`
- `src/main/java/br/com/fullcycle/hexagonal/services/PartnerService.java`

## `CustomerService`

```java
package br.com.fullcycle.hexagonal.services;

import br.com.fullcycle.hexagonal.models.Customer;
import br.com.fullcycle.hexagonal.repositories.CustomerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

@Service
public class CustomerService {

    @Autowired
    private CustomerRepository repository;

    @Transactional
    public Customer save(Customer customer) {
        return repository.save(customer);
    }

    public Optional<Customer> findById(Long id) {
        return repository.findById(id);
    }

    public Optional<Customer> findByCpf(String cpf) {
        return repository.findByCpf(cpf);
    }

    public Optional<Customer> findByEmail(String email) {
        return repository.findByEmail(email);
    }

}
```

**Anotações técnicas**

- `@Service` indica que esta classe faz parte da **camada de aplicação**.
- `@Transactional` na operação de `save` garante commit/rollback automático.
- O uso de `Optional` evita `null` explícito e força tratamento de ausência.

## `EventService`

```java
package br.com.fullcycle.hexagonal.services;

import br.com.fullcycle.hexagonal.models.Event;
import br.com.fullcycle.hexagonal.models.Ticket;
import br.com.fullcycle.hexagonal.repositories.EventRepository;
import br.com.fullcycle.hexagonal.repositories.TicketRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

@Service
public class EventService {

    @Autowired
    private CustomerService customerService;

    @Autowired
    private EventRepository eventRepository;

    @Autowired
    private TicketRepository ticketRepository;

    @Transactional
    public Event save(Event event) {
        return eventRepository.save(event);
    }

    public Optional<Event> findById(Long id) {
        return eventRepository.findById(id);
    }
    
    public Optional<Ticket> findTicketByEventIdAndCustomerId(Long id, Long customerId) {
        return ticketRepository.findByEventIdAndCustomerId(id, customerId);
    }
}
```

**Anotações técnicas**

- `EventService` orquestra persistência de `Event` e consultas de `Ticket`.
- A presença de `CustomerService` sugere possibilidade de regras adicionais envolvendo clientes (mesmo que não usadas aqui).
- `findTicketByEventIdAndCustomerId` expressa uma **consulta específica de negócio**.

## `PartnerService`

```java
package br.com.fullcycle.hexagonal.services;

import br.com.fullcycle.hexagonal.models.Partner;
import br.com.fullcycle.hexagonal.repositories.PartnerRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Optional;

@Service
public class PartnerService {

    @Autowired
    private PartnerRepository repository;

    @Transactional
    public Partner save(Partner customer) {
        return repository.save(customer);
    }

    public Optional<Partner> findById(Long id) {
        return repository.findById(id);
    }

    public Optional<Partner> findByCnpj(String cnpj) {
        return repository.findByCnpj(cnpj);
    }

    public Optional<Partner> findByEmail(String email) {
        return repository.findByEmail(email);
    }

}
```

**Anotações técnicas**

- Camada de aplicação simples, delegando ao repositório.
- Regras de unicidade são verificadas no controller, mas poderiam ser movidas aqui em um cenário mais rico.
