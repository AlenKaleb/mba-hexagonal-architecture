# Controllers REST (adapters de entrada)

Arquivos:
- `src/main/java/br/com/fullcycle/hexagonal/controllers/CustomerController.java`
- `src/main/java/br/com/fullcycle/hexagonal/controllers/EventController.java`
- `src/main/java/br/com/fullcycle/hexagonal/controllers/PartnerController.java`

## `CustomerController`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.CustomerDTO;
import br.com.fullcycle.hexagonal.models.Customer;
import br.com.fullcycle.hexagonal.services.CustomerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;

@RestController
@RequestMapping(value = "customers")
public class CustomerController {

    @Autowired
    private CustomerService customerService;

    @PostMapping
    public ResponseEntity<?> create(@RequestBody CustomerDTO dto) {
        if (customerService.findByCpf(dto.getCpf()).isPresent()) {
            return ResponseEntity.unprocessableEntity().body("Customer already exists");
        }
        if (customerService.findByEmail(dto.getEmail()).isPresent()) {
            return ResponseEntity.unprocessableEntity().body("Customer already exists");
        }

        var customer = new Customer();
        customer.setName(dto.getName());
        customer.setCpf(dto.getCpf());
        customer.setEmail(dto.getEmail());

        customer = customerService.save(customer);

        return ResponseEntity.created(URI.create("/customers/" + customer.getId())).body(customer);
    }

    @GetMapping("/{id}")
    public ResponseEntity<?> get(@PathVariable Long id) {
        var customer = customerService.findById(id);
        if (customer.isEmpty()) {
            return ResponseEntity.notFound().build();
        }

        return ResponseEntity.ok(customer.get());
    }
}
```

**Anotações técnicas**

- `@RestController` + `@RequestMapping`: expõem endpoints HTTP como adapter de entrada.
- O controller **não** acessa o repositório diretamente; delega para `CustomerService`, mantendo separação de responsabilidades.
- As validações de duplicidade (CPF/e-mail) são feitas antes de persistir, evitando violação de integridade na camada de dados.
- `ResponseEntity` permite controlar status HTTP e payloads, boa prática para API REST.
- **Padrão DTO**: o request usa `CustomerDTO`, evitando acoplamento direto à entidade.

## `EventController`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.EventDTO;
import br.com.fullcycle.hexagonal.dtos.SubscribeDTO;
import br.com.fullcycle.hexagonal.models.Event;
import br.com.fullcycle.hexagonal.models.Ticket;
import br.com.fullcycle.hexagonal.models.TicketStatus;
import br.com.fullcycle.hexagonal.services.CustomerService;
import br.com.fullcycle.hexagonal.services.EventService;
import br.com.fullcycle.hexagonal.services.PartnerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.web.bind.annotation.*;

import java.time.Instant;
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

import static org.springframework.http.HttpStatus.CREATED;

@RestController
@RequestMapping(value = "events")
public class EventController {

    @Autowired
    private CustomerService customerService;

    @Autowired
    private EventService eventService;

    @Autowired
    private PartnerService partnerService;

    @PostMapping
    @ResponseStatus(CREATED)
    public Event create(@RequestBody EventDTO dto) {
        var event = new Event();
        event.setDate(LocalDate.parse(dto.getDate(), DateTimeFormatter.ISO_DATE));
        event.setName(dto.getName());
        event.setTotalSpots(dto.getTotalSpots());

        var partner = partnerService.findById(dto.getPartner().getId());
        if (partner.isEmpty()) {
            throw new RuntimeException("Partner not found");
        }
        event.setPartner(partner.get());

        return eventService.save(event);
    }

    @Transactional
    @PostMapping(value = "/{id}/subscribe")
    public ResponseEntity<?> subscribe(@PathVariable Long id, @RequestBody SubscribeDTO dto) {

        var maybeCustomer = customerService.findById(dto.getCustomerId());
        if (maybeCustomer.isEmpty()) {
            return ResponseEntity.unprocessableEntity().body("Customer not found");
        }

        var maybeEvent = eventService.findById(id);
        if (maybeEvent.isEmpty()) {
            return ResponseEntity.notFound().build();
        }

        var maybeTicket = eventService.findTicketByEventIdAndCustomerId(id, dto.getCustomerId());
        if (maybeTicket.isPresent()) {
            return ResponseEntity.unprocessableEntity().body("Email already registered");
        }

        var customer = maybeCustomer.get();
        var event = maybeEvent.get();

        if (event.getTotalSpots() < event.getTickets().size() + 1) {
            throw new RuntimeException("Event sold out");
        }

        var ticket = new Ticket();
        ticket.setEvent(event);
        ticket.setCustomer(customer);
        ticket.setReservedAt(Instant.now());
        ticket.setStatus(TicketStatus.PENDING);

        event.getTickets().add(ticket);

        eventService.save(event);

        return ResponseEntity.ok(new EventDTO(event));
    }
}
```

**Anotações técnicas**

- `@ResponseStatus(CREATED)` retorna 201 no `create`, uma prática RESTful para criação.
- `@Transactional` garante que a inscrição e o vínculo `Event -> Ticket` sejam persistidos atomicamente.
- Conversão de data via `DateTimeFormatter.ISO_DATE` evita ambiguidade de formato.
- O controller checa:
  - existência de cliente
  - existência do evento
  - duplicidade de inscrição
  - capacidade total (`totalSpots`)
- **Padrão de agregados**: o `Event` agrega `Ticket`; o ticket é adicionado ao conjunto do evento e o evento é salvo.

## `PartnerController`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.PartnerDTO;
import br.com.fullcycle.hexagonal.models.Partner;
import br.com.fullcycle.hexagonal.services.PartnerService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.net.URI;

@RestController
@RequestMapping(value = "partners")
public class PartnerController {

    @Autowired
    private PartnerService partnerService;

    @PostMapping
    public ResponseEntity<?> create(@RequestBody PartnerDTO dto) {
        if (partnerService.findByCnpj(dto.getCnpj()).isPresent()) {
            return ResponseEntity.unprocessableEntity().body("Partner already exists");
        }
        if (partnerService.findByEmail(dto.getEmail()).isPresent()) {
            return ResponseEntity.unprocessableEntity().body("Partner already exists");
        }

        var partner = new Partner();
        partner.setName(dto.getName());
        partner.setCnpj(dto.getCnpj());
        partner.setEmail(dto.getEmail());

        partner = partnerService.save(partner);

        return ResponseEntity.created(URI.create("/partners/" + partner.getId())).body(partner);
    }

    @GetMapping("/{id}")
    public ResponseEntity<?> get(@PathVariable Long id) {
        var partner = partnerService.findById(id);
        if (partner.isEmpty()) {
            return ResponseEntity.notFound().build();
        }

        return ResponseEntity.ok(partner.get());
    }

}
```

**Anotações técnicas**

- Similar ao `CustomerController`, com validações de duplicidade (CNPJ/e-mail).
- **Boa prática**: retorno `201` com `Location` (`ResponseEntity.created(...)`).
- Mantém o controller “magro”, com regras delegadas ao service.
