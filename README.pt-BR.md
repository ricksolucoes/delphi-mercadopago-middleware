# Delphi Mercado Pago Middleware

> Middleware REST de pagamentos para o Mercado Pago desenvolvido com Delphi, Horse e RESTRequest4Delphi, projetado com arquitetura em camadas, dependências explícitas e fronteiras de aplicação testáveis.

🌐 **Documentação:** [English](README.md) | **Português (Brasil)**

> Esta é a versão em Português do Brasil. O [README em inglês](README.md) é a versão canônica da documentação.

![Delphi](https://img.shields.io/badge/Delphi-10%2B-C71A36)
![Horse](https://img.shields.io/badge/Horse-REST%20Server-2F4F4F)
![RESTRequest4Delphi](https://img.shields.io/badge/HTTP%20Client-RESTRequest4Delphi-4B6EAF)
![dataset--serialize](https://img.shields.io/badge/Serialization-dataset--serialize-6A5ACD)
![Mercado Pago](https://img.shields.io/badge/Integration-Mercado%20Pago-009EE3)
![Architecture](https://img.shields.io/badge/Architecture-Layered-555555)

---

## Sobre o projeto

**Delphi Mercado Pago Middleware** é um projeto de middleware backend projetado para isolar aplicações Delphi dos contratos externos e das preocupações de infraestrutura do Mercado Pago.

O projeto concentra-se na construção de uma integração de pagamentos sustentável, com fronteiras arquiteturais claras, dependências explícitas e acesso controlado a serviços externos.

Em vez de acoplar diretamente a aplicação cliente às APIs do Mercado Pago, o middleware introduz um contrato REST interno entre a aplicação e a infraestrutura de pagamentos.

```text
Aplicação Delphi
        |
        | HTTP / JSON
        v
Middleware Delphi
        |
        v
Horse
        |
        v
Routes
        |
        v
Controllers
        |
        v
Services
        |
        +-------------------+
        |                   |
        v                   v
    Gateways           Repositories
        |
        v
RESTRequest4Delphi
        |
        | HTTPS / JSON
        v
   Mercado Pago
```

O contrato externo do Mercado Pago permanece isolado do contrato interno da aplicação sempre que for tecnicamente viável.

---

## Objetivos

Este projeto está sendo desenvolvido como uma implementação prática de princípios de engenharia de software aplicados a uma integração financeira real.

Os principais objetivos são:

* fornecer uma fronteira REST estável entre aplicações Delphi e o Mercado Pago;
* isolar contratos externos de pagamento atrás de gateways;
* manter regras de negócio independentes de frameworks HTTP;
* aplicar inversão explícita de dependência por meio de interfaces;
* melhorar a testabilidade substituindo dependências externas em testes unitários;
* centralizar a comunicação HTTP externa;
* proteger credenciais privadas dentro do middleware;
* tratar operações de pagamento com os cuidados adequados de segurança e idempotência;
* evoluir a integração sem acoplar desnecessariamente as aplicações cliente às mudanças das APIs externas.

---

## Stack tecnológica

| Tecnologia             | Responsabilidade                                   |
| ---------------------- | -------------------------------------------------- |
| **Delphi 10+**         | Implementação do middleware                        |
| **Horse**              | Servidor REST e infraestrutura HTTP                |
| **RESTRequest4Delphi** | Comunicação HTTP de saída                          |
| **dataset-serialize**  | Serialização de `TDataSet` quando apropriado       |
| **Mercado Pago**       | Plataforma externa de pagamentos                   |

O projeto tem como alvo **Delphi 10 ou superior**.

As APIs específicas das bibliotecas são validadas de acordo com a versão efetivamente utilizada pelo projeto antes da implementação.

---

## Arquitetura

O projeto segue uma arquitetura em camadas baseada em **MVC + Service Layer**, mantendo as preocupações de infraestrutura fora das regras da aplicação.

```text
Route
  |
  v
Controller
  |
  v
Service
  |
  +---------------------+
  |                     |
  v                     v
Gateway             Repository
  |                     |
  v                     v
API Externa          Persistência
```

### Routes

As Routes pertencem à infraestrutura HTTP do Horse.

Responsabilidades:

* definir rotas HTTP;
* associar métodos HTTP aos Controllers;
* anexar os middlewares HTTP necessários.

Routes não devem conter regras de negócio, lógica de persistência ou código de integração com o Mercado Pago.

### Controllers

Controllers adaptam requisições HTTP para a camada de aplicação.

Responsabilidades:

* receber a entrada HTTP;
* realizar o tratamento estrutural da entrada;
* criar DTOs de requisição;
* invocar Services;
* traduzir resultados da aplicação em respostas HTTP.

Controllers não se comunicam diretamente com o Mercado Pago.

### Services

Services implementam casos de uso e regras de negócio da aplicação.

Responsabilidades:

* validação de negócio;
* orquestração;
* tomada de decisão;
* coordenação entre Gateways e Repositories.

Services permanecem independentes de `THorseRequest` e `THorseResponse`.

### Gateways

Gateways isolam integrações externas.

```text
Service
   |
   v
IPaymentGateway
   |
   v
Mercado Pago Gateway
   |
   v
RESTRequest4Delphi
   |
   v
Mercado Pago
```

Detalhes HTTP específicos do Mercado Pago pertencem a essa fronteira.

### Repositories

Quando persistência é necessária, o acesso é exposto por meio de contratos de Repository.

```text
Service
   |
   v
Repository Interface
   |
   v
Repository Implementation
   |
   v
Banco de Dados
```

Nenhuma tecnologia de persistência é assumida até que seja explicitamente definida pelo projeto.

---

## Contratos internos vs. externos

Um princípio arquitetural central deste projeto é manter a API interna do middleware independente dos contratos do Mercado Pago sempre que for tecnicamente viável.

```text
Aplicação Cliente
       |
       v
Internal Request DTO
       |
       v
Service
       |
       v
Mapper
       |
       v
Mercado Pago DTO
       |
       v
Gateway
       |
       v
Mercado Pago
```

O middleware é responsável por traduzir entre os contratos internos da aplicação e os contratos externos de pagamento.

Isso reduz o impacto de futuras mudanças nas APIs externas sobre as aplicações cliente.

---

## Princípios de engenharia

O projeto é orientado pelas seguintes práticas de engenharia:

### SOLID

Os princípios SOLID são aplicados de forma pragmática, com ênfase em:

* Single Responsibility Principle;
* Dependency Inversion Principle;
* interfaces pequenas e coesas;
* dependências explícitas;
* baixo acoplamento;
* alta coesão.

O objetivo é manutenção e testabilidade, e não abstração desnecessária.

### Design orientado a interfaces

As fronteiras da aplicação são expostas por interfaces quando isso oferece valor arquitetural concreto.

Direção típica de dependências:

```text
Controller -> IService
Service    -> IGateway
Service    -> IRepository
```

Integrações externas concretas permanecem atrás de abstrações.

### Dependency Injection

As dependências são preferencialmente fornecidas de forma explícita por construtores.

O projeto evita dependências ocultas por estado global mutável ou padrões de Service Locator.

### Clean Code

A base de código prioriza:

* nomenclatura clara;
* classes coesas;
* responsabilidades pequenas;
* dependências explícitas;
* duplicação mínima;
* ausência de abstrações desnecessárias;
* refatoração incremental;
* alterações mínimas tecnicamente corretas.

### TDD e testabilidade

Novos comportamentos e refatorações relevantes são projetados considerando testabilidade.

Fluxo de desenvolvimento preferido:

```text
Requisito
    |
    v
Cenários
    |
    v
Testes
    |
    v
Implementação
    |
    v
Refatoração
```

Serviços externos reais do Mercado Pago não devem ser necessários para testes unitários.

Dependências externas devem poder ser substituídas por fakes ou implementações de teste.

---

## Princípios de segurança

A integração de pagamentos é tratada como uma responsabilidade backend sensível à segurança.

O projeto estabelece os seguintes princípios:

* credenciais privadas do Mercado Pago permanecem no middleware;
* secrets não devem ser versionados no código-fonte;
* headers `Authorization` não devem ser expostos em logs;
* Access Tokens e Client Secrets nunca devem ser registrados em logs;
* CVV e dados sensíveis de pagamento nunca devem ser registrados em logs;
* a validação TLS não deve ser desabilitada como solução alternativa;
* logs devem ser sanitizados;
* ambientes devem ser separados;
* retries de operações financeiras não devem ser executados às cegas;
* idempotência deve ser tratada de acordo com o contrato oficial da API;
* autenticidade de Webhooks deve ser validada de acordo com a documentação atual do Mercado Pago.

---

## Idempotência

Operações financeiras exigem tratamento especial porque uma falha de rede não significa necessariamente que a operação remota não foi processada.

O projeto diferencia:

```text
Nova operação lógica
```

de:

```text
Nova tentativa da mesma operação lógica
```

Quando a idempotência for exigida pela API do Mercado Pago selecionada, a implementação deve seguir o contrato oficial vigente.

Uma nova tentativa não deve criar automaticamente uma nova operação financeira sem avaliar o estado da requisição anterior.

---

## Correlação e observabilidade

Quando aplicável, as requisições devem utilizar um `CorrelationId` ao longo do fluxo do middleware:

```text
Cliente
  |
  v
Horse
  |
  v
Controller
  |
  v
Service
  |
  v
Gateway
  |
  v
Serviço Externo
```

`CorrelationId` e `IdempotencyKey` representam conceitos diferentes e não devem ser tratados como identificadores intercambiáveis.

Os logs devem fornecer informações suficientes para diagnóstico sem expor dados sensíveis.

---

## Tratamento de erros

O middleware deve preservar a semântica HTTP adequada.

Falhas externas do Mercado Pago não devem ser automaticamente expostas diretamente aos consumidores.

A fronteira da aplicação é responsável por traduzir falhas técnicas para um contrato interno de erro apropriado, preservando informações de diagnóstico sanitizadas.

Grandes blocos `try/except` duplicados entre Controllers devem ser evitados quando o tratamento centralizado de exceções HTTP for apropriado.

---

## Serialização de dados

`dataset-serialize` é utilizado somente quando a conversão `TDataSet` ↔ JSON for uma preocupação de infraestrutura apropriada.

Ele não é destinado a substituir:

* DTOs;
* Entities;
* modelos de domínio;
* Mappers.

Quando regras de negócio operarem sobre dados persistidos, a direção preferida é:

```text
TDataSet
   |
   v
Mapper
   |
   v
DTO / Entity
   |
   v
Service
```

---

## Integração com Mercado Pago

As APIs do Mercado Pago são tratadas como contratos externos que podem evoluir independentemente deste middleware.

Antes de implementar ou alterar uma integração, o projeto exige a validação da documentação oficial vigente em relação, quando aplicável, a:

* produto/API utilizado;
* endpoint;
* método HTTP;
* autenticação;
* headers obrigatórios;
* contrato de request;
* contrato de response;
* idempotência;
* regras de Webhook;
* statuses;
* respostas de erro;
* requisitos de ambiente.

Nenhum produto do Mercado Pago é assumido automaticamente.

O fluxo real de pagamento deve ser selecionado de acordo com o requisito de negócio e a documentação oficial vigente.

Documentação oficial:

https://www.mercadopago.com.br/developers/

---

## Estrutura do projeto

A organização conceitual segue responsabilidades, e não conveniência de framework:

```text
MercadoPago
|
+-- Contracts
+-- Routes
+-- Controllers
+-- Services
+-- Gateways
+-- DTO
+-- Mappers
+-- Middlewares
+-- Serialization
+-- Config
+-- Exceptions
+-- Tests
```

Isso representa a organização arquitetural pretendida.

Diretórios e units devem ser introduzidos apenas quando possuírem uma responsabilidade real na implementação.

---

## Decisões de design

### Por que um middleware?

Acoplar diretamente uma aplicação desktop ou de negócio a um provedor externo de pagamentos faz com que mudanças na API externa se propaguem pela base de código cliente.

O middleware cria uma fronteira controlada:

```text
Contrato da Aplicação
        !=
Contrato do Mercado Pago
```

Isso permite que preocupações específicas de infraestrutura permaneçam isoladas dos consumidores voltados ao negócio.

### Por que Gateways?

Gateways evitam que contratos HTTP externos se espalhem por Controllers e Services.

Eles fornecem uma única fronteira arquitetural para a comunicação externa de pagamentos.

### Por que interfaces?

Interfaces tornam as dependências explícitas e permitem que implementações externas sejam substituídas durante testes automatizados.

### Por que Service Layer?

A Service Layer mantém casos de uso e decisões de negócio independentes do Horse e de objetos específicos de HTTP.

---

## Regras de desenvolvimento

Algumas regras fundamentais do projeto são:

```text
Route      != Regra de Negócio
Controller != Cliente Mercado Pago
Service    != Framework HTTP
DTO        != Infraestrutura
Repository != Controller
Gateway    == Fronteira de Integração Externa
```

E, especificamente:

```text
Horse                -> infraestrutura HTTP
RESTRequest4Delphi   -> infraestrutura HTTP de saída
dataset-serialize     -> infraestrutura de serialização de TDataSet
Mercado Pago         -> dependência externa
```

As regras de negócio devem permanecer tão independentes quanto possível das quatro dependências.

---

## Roadmap

O projeto evolui de forma incremental.

As preocupações de engenharia planejadas incluem:

* [ ] evolução do contrato da API interna;
* [ ] autenticação e autorização da aplicação;
* [ ] implementações de Gateways do Mercado Pago conforme casos de uso validados;
* [ ] persistência quando exigida pelos requisitos de negócio;
* [ ] contrato padronizado de erros da aplicação;
* [ ] correlação e logging estruturado;
* [ ] tratamento de idempotência quando exigido;
* [ ] processamento de Webhooks e validação de autenticidade quando aplicável;
* [ ] testes unitários para Services e regras de negócio;
* [ ] testes de integração para as fronteiras HTTP;
* [ ] estratégia de configuração de ambientes;
* [ ] validação automatizada de qualidade e build.

Os itens deste roadmap representam trabalho de engenharia planejado e não devem ser interpretados como funcionalidades já implementadas.

---

## Status atual

Este repositório está em desenvolvimento ativo.

Funcionalidades, endpoints e produtos do Mercado Pago são documentados como implementados somente depois que seus contratos e comportamentos são validados em relação ao código-fonte real e à documentação oficial vigente.

O projeto evita deliberadamente apresentar capacidades planejadas como funcionalidades concluídas.

---

## Referências

### Mercado Pago

Documentação oficial para desenvolvedores:

https://www.mercadopago.com.br/developers/

### Horse

https://github.com/HashLoad/horse

### RESTRequest4Delphi

https://github.com/viniciussanchez/RESTRequest4Delphi

### dataset-serialize

https://github.com/viniciussanchez/dataset-serialize

---

## Aviso legal

Este é um projeto de software independente e não é um SDK oficial do Mercado Pago.

O Mercado Pago é uma plataforma externa integrada pelo middleware de acordo com sua documentação oficial para desenvolvedores.

---

## Contribuindo

Atualmente, o projeto prioriza consistência arquitetural, segurança, testabilidade e evolução incremental.

Contribuições devem preservar as fronteiras arquiteturais existentes e evitar a introdução de dependências de infraestrutura nas regras de negócio.

---

## Licença

Este projeto é licenciado sob a **MIT License**.

* [MIT License](LICENSE)
