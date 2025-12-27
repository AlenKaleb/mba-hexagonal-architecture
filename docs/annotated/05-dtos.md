# DTOs (Data Transfer Objects)

Arquivos:
- `src/main/java/br/com/fullcycle/hexagonal/dtos/CustomerDTO.java`
- `src/main/java/br/com/fullcycle/hexagonal/dtos/EventDTO.java`
- `src/main/java/br/com/fullcycle/hexagonal/dtos/PartnerDTO.java`
- `src/main/java/br/com/fullcycle/hexagonal/dtos/SubscribeDTO.java`
- `src/main/java/br/com/fullcycle/hexagonal/dtos/TicketDTO.java`

## `CustomerDTO`

```java
package br.com.fullcycle.hexagonal.dtos;

import br.com.fullcycle.hexagonal.models.Customer;

public class CustomerDTO {
    private Long id;
    private String name;
    private String cpf;
    private String email;

    public CustomerDTO() {
    }

    public CustomerDTO(Customer customer) {
        this.id = customer.getId();
        this.name = customer.getName();
        this.cpf = customer.getCpf();
        this.email = customer.getEmail();
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCpf() {
        return cpf;
    }

    public void setCpf(String cpf) {
        this.cpf = cpf;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

**Anotações técnicas**

- DTO desacopla a API REST da entidade de domínio.
- Construtor com `Customer` facilita montagem em respostas.

## `EventDTO`

```java
package br.com.fullcycle.hexagonal.dtos;

import br.com.fullcycle.hexagonal.models.Event;

import java.time.format.DateTimeFormatter;

public class EventDTO {

    private Long id;
    private String name;
    private String date;
    private int totalSpots;
    private PartnerDTO partner;

    public EventDTO() {
    }

    public EventDTO(Event event) {
        this.id = event.getId();
        this.name = event.getName();
        this.date = event.getDate().format(DateTimeFormatter.ISO_DATE);
        this.totalSpots = event.getTotalSpots();
        this.partner = new PartnerDTO(event.getPartner());
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public int getTotalSpots() {
        return totalSpots;
    }

    public void setTotalSpots(int totalSpots) {
        this.totalSpots = totalSpots;
    }

    public PartnerDTO getPartner() {
        return partner;
    }

    public void setPartner(PartnerDTO partner) {
        this.partner = partner;
    }

}
```

**Anotações técnicas**

- Formatação de data padronizada com `ISO_DATE` evita divergências entre front/back.
- `PartnerDTO` embutido representa composição no payload.

## `PartnerDTO`

```java
package br.com.fullcycle.hexagonal.dtos;

import br.com.fullcycle.hexagonal.models.Partner;

public class PartnerDTO {
    private Long id;
    private String name;
    private String cnpj;
    private String email;

    public PartnerDTO() {
    }

    public PartnerDTO(Long id) {
        this.id = id;
    }

    public PartnerDTO(Partner partner) {
        this.id = partner.getId();
        this.name = partner.getName();
        this.cnpj = partner.getCnpj();
        this.email = partner.getEmail();
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getCnpj() {
        return cnpj;
    }

    public void setCnpj(String cnpj) {
        this.cnpj = cnpj;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

**Anotações técnicas**

- Construtor com `id` é útil quando apenas a referência é necessária.

## `SubscribeDTO`

```java
package br.com.fullcycle.hexagonal.dtos;

public class SubscribeDTO {

    private Long customerId;

    public Long getCustomerId() {
        return customerId;
    }

    public void setCustomerId(Long customerId) {
        this.customerId = customerId;
    }
}
```

**Anotações técnicas**

- DTO enxuto para operação específica de inscrição, reduzindo payload.

## `TicketDTO`

```java
package br.com.fullcycle.hexagonal.dtos;

import br.com.fullcycle.hexagonal.models.Ticket;
import br.com.fullcycle.hexagonal.models.TicketStatus;

import java.time.Instant;

public class TicketDTO {
    private Long id;
    private int spot;
    private CustomerDTO customer;
    private EventDTO event;
    private TicketStatus status;
    private Instant paidAt;
    private Instant reservedAt;

    public TicketDTO() {
    }

    public TicketDTO(Ticket ticket) {
        this.id = ticket.getId();
        this.customer = new CustomerDTO(ticket.getCustomer());
        this.event = new EventDTO(ticket.getEvent());
        this.status = ticket.getStatus();
        this.paidAt = ticket.getPaidAt();
        this.reservedAt = ticket.getReservedAt();
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public int getSpot() {
        return spot;
    }

    public void setSpot(int spot) {
        this.spot = spot;
    }

    public CustomerDTO getCustomer() {
        return customer;
    }

    public void setCustomer(CustomerDTO customer) {
        this.customer = customer;
    }

    public EventDTO getEvent() {
        return event;
    }

    public void setEvent(EventDTO event) {
        this.event = event;
    }

    public TicketStatus getStatus() {
        return status;
    }

    public void setStatus(TicketStatus status) {
        this.status = status;
    }

    public Instant getPaidAt() {
        return paidAt;
    }

    public void setPaidAt(Instant paidAt) {
        this.paidAt = paidAt;
    }

    public Instant getReservedAt() {
        return reservedAt;
    }

    public void setReservedAt(Instant reservedAt) {
        this.reservedAt = reservedAt;
    }
}
```

**Anotações técnicas**

- DTO composto, agregando `CustomerDTO` e `EventDTO`.
- Campos `Instant` preservam precisão de data/hora no transporte.
