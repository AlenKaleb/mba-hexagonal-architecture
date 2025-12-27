# Modelos de domínio (entidades JPA)

Arquivos:
- `src/main/java/br/com/fullcycle/hexagonal/models/Customer.java`
- `src/main/java/br/com/fullcycle/hexagonal/models/Event.java`
- `src/main/java/br/com/fullcycle/hexagonal/models/Partner.java`
- `src/main/java/br/com/fullcycle/hexagonal/models/Ticket.java`
- `src/main/java/br/com/fullcycle/hexagonal/models/TicketStatus.java`

## `Customer`

```java
package br.com.fullcycle.hexagonal.models;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

import java.util.Objects;

import static jakarta.persistence.GenerationType.*;

@Entity
@Table(name = "customers")
public class Customer {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    private String name;

    private String cpf;

    private String email;

    public Customer() {
    }

    public Customer(Long id, String name, String cpf, String email) {
        this.id = id;
        this.name = name;
        this.cpf = cpf;
        this.email = email;
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

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Customer customer = (Customer) o;
        return Objects.equals(id, customer.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

**Anotações técnicas**

- `@Entity` + `@Table` mapeiam a classe para tabela `customers`.
- `@GeneratedValue(strategy = IDENTITY)` delega a geração do ID ao banco.
- `equals/hashCode` baseados apenas no `id` são comuns em entidades persistentes.

## `Event`

```java
package br.com.fullcycle.hexagonal.models;

import jakarta.persistence.*;

import java.time.LocalDate;
import java.util.HashSet;
import java.util.Objects;
import java.util.Set;

import static jakarta.persistence.GenerationType.IDENTITY;

@Entity
@Table(name = "events")
public class Event {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    private String name;

    private LocalDate date;

    private int totalSpots;

    @ManyToOne(fetch = FetchType.LAZY)
    private Partner partner;

    @OneToMany(cascade = CascadeType.ALL, mappedBy = "event")
    private Set<Ticket> tickets;

    public Event() {
        this.tickets = new HashSet<>();
    }

    public Event(Long id, String name, LocalDate date, int totalSpots, Set<Ticket> tickets) {
        this.id = id;
        this.name = name;
        this.date = date;
        this.totalSpots = totalSpots;
        this.tickets = tickets != null ? tickets : new HashSet<>();
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

    public LocalDate getDate() {
        return date;
    }

    public void setDate(LocalDate date) {
        this.date = date;
    }

    public int getTotalSpots() {
        return totalSpots;
    }

    public void setTotalSpots(int totalSpots) {
        this.totalSpots = totalSpots;
    }

    public Partner getPartner() {
        return partner;
    }

    public void setPartner(Partner partner) {
        this.partner = partner;
    }

    public Set<Ticket> getTickets() {
        return tickets;
    }

    public void setTickets(Set<Ticket> tickets) {
        this.tickets = tickets;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Event event = (Event) o;
        return Objects.equals(id, event.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

**Anotações técnicas**

- Relação `@ManyToOne` com `Partner`: um parceiro pode ter muitos eventos.
- `@OneToMany(cascade = CascadeType.ALL)` garante que tickets sejam persistidos junto ao evento.
- `tickets` é inicializado no construtor para evitar `NullPointerException`.

## `Partner`

```java
package br.com.fullcycle.hexagonal.models;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

import static jakarta.persistence.GenerationType.IDENTITY;

@Entity
@Table(name = "partners")
public class Partner {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    private String name;

    private String cnpj;

    private String email;

    public Partner() {
    }

    public Partner(Long id, String name, String cnpj, String email) {
        this.id = id;
        this.name = name;
        this.cnpj = cnpj;
        this.email = email;
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

- Entidade simples, usada por `Event` como relação `@ManyToOne`.

## `Ticket`

```java
package br.com.fullcycle.hexagonal.models;

import jakarta.persistence.*;

import java.time.Instant;
import java.util.Objects;

import static jakarta.persistence.GenerationType.IDENTITY;

@Entity
@Table(name = "tickets")
public class Ticket {

    @Id
    @GeneratedValue(strategy = IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    private Customer customer;

    @ManyToOne(fetch = FetchType.LAZY)
    private Event event;

    @Enumerated(EnumType.STRING)
    private TicketStatus status;

    private Instant paidAt;

    private Instant reservedAt;

    public Ticket() {
    }

    public Ticket(Long id, Customer customer, Event event, TicketStatus status, Instant paidAt, Instant reservedAt) {
        this.id = id;
        this.customer = customer;
        this.event = event;
        this.status = status;
        this.paidAt = paidAt;
        this.reservedAt = reservedAt;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Customer getCustomer() {
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    public Event getEvent() {
        return event;
    }

    public void setEvent(Event event) {
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

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Ticket ticket = (Ticket) o;
        return Objects.equals(customer, ticket.customer) && Objects.equals(event, ticket.event);
    }

    @Override
    public int hashCode() {
        return Objects.hash(customer, event);
    }
}
```

**Anotações técnicas**

- `@Enumerated(EnumType.STRING)` grava o enum como texto, evitando números mágicos.
- `equals/hashCode` baseados no par `(customer, event)` ajudam a evitar duplicidade de inscrição.

## `TicketStatus`

```java
package br.com.fullcycle.hexagonal.models;

public enum TicketStatus {
    PENDING, PROCESSING, PAID;
}
```

**Anotações técnicas**

- Enum simples para estados do ticket, garantindo **type safety**.
