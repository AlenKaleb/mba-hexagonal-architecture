# Testes automatizados

Arquivos:
- `src/test/java/br/com/fullcycle/hexagonal/MainTests.java`
- `src/test/java/br/com/fullcycle/hexagonal/controllers/CustomerControllerTest.java`
- `src/test/java/br/com/fullcycle/hexagonal/controllers/PartnerControllerTest.java`
- `src/test/java/br/com/fullcycle/hexagonal/controllers/EventControllerTest.java`

## `MainTests`

```java
package br.com.fullcycle.hexagonal;

import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.ActiveProfiles;

@ActiveProfiles("test")
@SpringBootTest
class MainTests {

    @Test
    void contextLoads() {
    }

}
```

**Anotações técnicas**

- Teste “smoke” para garantir que o contexto Spring inicializa.
- `@ActiveProfiles("test")` carrega `application-test.properties`.

## `CustomerControllerTest`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.CustomerDTO;
import br.com.fullcycle.hexagonal.repositories.CustomerRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest
public class CustomerControllerTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private ObjectMapper mapper;

    @Autowired
    private CustomerRepository customerRepository;

    @AfterEach
    void tearDown() {
        customerRepository.deleteAll();
    }

    @Test
    @DisplayName("Deve criar um cliente")
    public void testCreate() throws Exception {

        var customer = new CustomerDTO();
        customer.setCpf("12345678901");
        customer.setEmail("john.doe@gmail.com");
        customer.setName("John Doe");

        final var result = this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        var actualResponse = mapper.readValue(result, CustomerDTO.class);
        Assertions.assertEquals(customer.getName(), actualResponse.getName());
        Assertions.assertEquals(customer.getCpf(), actualResponse.getCpf());
        Assertions.assertEquals(customer.getEmail(), actualResponse.getEmail());
    }

    @Test
    @DisplayName("Não deve cadastrar um cliente com CPF duplicado")
    public void testCreateWithDuplicatedCPFShouldFail() throws Exception {

        var customer = new CustomerDTO();
        customer.setCpf("12345678901");
        customer.setEmail("john.doe@gmail.com");
        customer.setName("John Doe");

        // Cria o primeiro cliente
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        customer.setEmail("john2@gmail.com");

        // Tenta criar o segundo cliente com o mesmo CPF
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andExpect(MockMvcResultMatchers.status().isUnprocessableEntity())
                .andExpect(MockMvcResultMatchers.content().string("Customer already exists"));
    }

    @Test
    @DisplayName("Não deve cadastrar um cliente com e-mail duplicado")
    public void testCreateWithDuplicatedEmailShouldFail() throws Exception {

        var customer = new CustomerDTO();
        customer.setCpf("12345618901");
        customer.setEmail("john.doe@gmail.com");
        customer.setName("John Doe");

        // Cria o primeiro cliente
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        customer.setCpf("99999918901");

        // Tenta criar o segundo cliente com o mesmo CPF
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andExpect(MockMvcResultMatchers.status().isUnprocessableEntity())
                .andExpect(MockMvcResultMatchers.content().string("Customer already exists"));
    }

    @Test
    @DisplayName("Deve obter um cliente por id")
    public void testGet() throws Exception {

        var customer = new CustomerDTO();
        customer.setCpf("12345678901");
        customer.setEmail("john.doe@gmail.com");
        customer.setName("John Doe");

        final var createResult = this.mvc.perform(
                        MockMvcRequestBuilders.post("/customers")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(customer))
                )
                .andReturn().getResponse().getContentAsByteArray();

        var customerId = mapper.readValue(createResult, CustomerDTO.class).getId();

        final var result = this.mvc.perform(
                        MockMvcRequestBuilders.get("/customers/{id}", customerId)
                )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn().getResponse().getContentAsByteArray();

        var actualResponse = mapper.readValue(result, CustomerDTO.class);
        Assertions.assertEquals(customerId, actualResponse.getId());
        Assertions.assertEquals(customer.getName(), actualResponse.getName());
        Assertions.assertEquals(customer.getCpf(), actualResponse.getCpf());
        Assertions.assertEquals(customer.getEmail(), actualResponse.getEmail());
    }
}
```

**Anotações técnicas**

- Usa `MockMvc` para testar controllers sem subir servidor real.
- `tearDown` limpa o repositório, garantindo isolamento entre testes.
- Testes cobrem: criação válida, duplicidade e consulta por ID.

## `PartnerControllerTest`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.PartnerDTO;
import br.com.fullcycle.hexagonal.repositories.PartnerRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;

@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest
public class PartnerControllerTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private ObjectMapper mapper;

    @Autowired
    private PartnerRepository partnerRepository;

    @AfterEach
    void tearDown() {
        partnerRepository.deleteAll();
    }

    @Test
    @DisplayName("Deve criar um parceiro")
    public void testCreate() throws Exception {

        var partner = new PartnerDTO();
        partner.setCnpj("41536538000100");
        partner.setEmail("john.doe@gmail.com");
        partner.setName("John Doe");

        final var result = this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        var actualResponse = mapper.readValue(result, PartnerDTO.class);
        Assertions.assertEquals(partner.getName(), actualResponse.getName());
        Assertions.assertEquals(partner.getCnpj(), actualResponse.getCnpj());
        Assertions.assertEquals(partner.getEmail(), actualResponse.getEmail());
    }

    @Test
    @DisplayName("Não deve cadastrar um parceiro com CNPJ duplicado")
    public void testCreateWithDuplicatedCPFShouldFail() throws Exception {

        var partner = new PartnerDTO();
        partner.setCnpj("41536538000100");
        partner.setEmail("john.doe@gmail.com");
        partner.setName("John Doe");

        // Cria o primeiro parceiro
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        partner.setEmail("john2@gmail.com");

        // Tenta criar o segundo parceiro com o mesmo CPF
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andExpect(MockMvcResultMatchers.status().isUnprocessableEntity())
                .andExpect(MockMvcResultMatchers.content().string("Partner already exists"));
    }

    @Test
    @DisplayName("Não deve cadastrar um parceiro com e-mail duplicado")
    public void testCreateWithDuplicatedEmailShouldFail() throws Exception {

        var partner = new PartnerDTO();
        partner.setCnpj("41536538000100");
        partner.setEmail("john.doe@gmail.com");
        partner.setName("John Doe");

        // Cria o primeiro parceiro
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.header().exists("Location"))
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        partner.setCnpj("66666538000100");

        // Tenta criar o segundo parceiro com o mesmo CNPJ
        this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andExpect(MockMvcResultMatchers.status().isUnprocessableEntity())
                .andExpect(MockMvcResultMatchers.content().string("Partner already exists"));
    }

    @Test
    @DisplayName("Deve obter um parceiro por id")
    public void testGet() throws Exception {

        var partner = new PartnerDTO();
        partner.setCnpj("41536538000100");
        partner.setEmail("john.doe@gmail.com");
        partner.setName("John Doe");

        final var createResult = this.mvc.perform(
                        MockMvcRequestBuilders.post("/partners")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(partner))
                )
                .andReturn().getResponse().getContentAsByteArray();

        var partnerId = mapper.readValue(createResult, PartnerDTO.class).getId();

        final var result = this.mvc.perform(
                        MockMvcRequestBuilders.get("/partners/{id}", partnerId)
                )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn().getResponse().getContentAsByteArray();

        var actualResponse = mapper.readValue(result, PartnerDTO.class);
        Assertions.assertEquals(partnerId, actualResponse.getId());
        Assertions.assertEquals(partner.getName(), actualResponse.getName());
        Assertions.assertEquals(partner.getCnpj(), actualResponse.getCnpj());
        Assertions.assertEquals(partner.getEmail(), actualResponse.getEmail());
    }
}
```

**Anotações técnicas**

- Testes espelham os cenários do controller e validam respostas HTTP e payloads.
- Limpeza pós-teste mantém a base H2 consistente.

## `EventControllerTest`

```java
package br.com.fullcycle.hexagonal.controllers;

import br.com.fullcycle.hexagonal.dtos.EventDTO;
import br.com.fullcycle.hexagonal.dtos.PartnerDTO;
import br.com.fullcycle.hexagonal.dtos.SubscribeDTO;
import br.com.fullcycle.hexagonal.models.Customer;
import br.com.fullcycle.hexagonal.models.Partner;
import br.com.fullcycle.hexagonal.repositories.CustomerRepository;
import br.com.fullcycle.hexagonal.repositories.EventRepository;
import br.com.fullcycle.hexagonal.repositories.PartnerRepository;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.ActiveProfiles;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.result.MockMvcResultMatchers;
import org.springframework.transaction.annotation.Transactional;

@ActiveProfiles("test")
@AutoConfigureMockMvc
@SpringBootTest
class EventControllerTest {

    @Autowired
    private MockMvc mvc;

    @Autowired
    private ObjectMapper mapper;

    @Autowired
    private CustomerRepository customerRepository;

    @Autowired
    private PartnerRepository partnerRepository;

    @Autowired
    private EventRepository eventRepository;

    private Customer johnDoe;
    private Partner disney;

    @BeforeEach
    void setUp() {
        johnDoe = customerRepository.save(new Customer(null, "John Doe", "123", "john@gmail.com"));
        disney = partnerRepository.save(new Partner(null, "Disney", "456", "disney@gmail.com"));
    }

    @AfterEach
    void tearDown() {
        eventRepository.deleteAll();
        customerRepository.deleteAll();
        partnerRepository.deleteAll();
    }

    @Test
    @DisplayName("Deve criar um evento")
    public void testCreate() throws Exception {

        var event = new EventDTO();
        event.setDate("2021-01-01");
        event.setName("Disney on Ice");
        event.setTotalSpots(100);
        event.setPartner(new PartnerDTO(disney.getId()));

        final var result = this.mvc.perform(
                        MockMvcRequestBuilders.post("/events")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(event))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        var actualResponse = mapper.readValue(result, EventDTO.class);
        Assertions.assertEquals(event.getDate(), actualResponse.getDate());
        Assertions.assertEquals(event.getTotalSpots(), actualResponse.getTotalSpots());
        Assertions.assertEquals(event.getName(), actualResponse.getName());
    }

    @Test
    @Transactional
    @DisplayName("Deve comprar um ticket de um evento")
    public void testReserveTicket() throws Exception {

        var event = new EventDTO();
        event.setDate("2021-01-01");
        event.setName("Disney on Ice");
        event.setTotalSpots(100);
        event.setPartner(new PartnerDTO(disney.getId()));

        final var createResult = this.mvc.perform(
                        MockMvcRequestBuilders.post("/events")
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(event))
                )
                .andExpect(MockMvcResultMatchers.status().isCreated())
                .andExpect(MockMvcResultMatchers.jsonPath("$.id").isNumber())
                .andReturn().getResponse().getContentAsByteArray();

        var eventId = mapper.readValue(createResult, EventDTO.class).getId();

        var sub = new SubscribeDTO();
        sub.setCustomerId(johnDoe.getId());

        this.mvc.perform(
                        MockMvcRequestBuilders.post("/events/{id}/subscribe", eventId)
                                .contentType(MediaType.APPLICATION_JSON)
                                .content(mapper.writeValueAsString(sub))
                )
                .andExpect(MockMvcResultMatchers.status().isOk())
                .andReturn().getResponse().getContentAsByteArray();

        var actualEvent = eventRepository.findById(eventId).get();
        Assertions.assertEquals(1, actualEvent.getTickets().size());
    }
}
```

**Anotações técnicas**

- `@BeforeEach` prepara entidades dependentes para os testes.
- `@Transactional` permite carregar as relações sem problemas de sessão ao final do teste.
- Verificação final confirma que o `Ticket` foi persistido via agregação do `Event`.
