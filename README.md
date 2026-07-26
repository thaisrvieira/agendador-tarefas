# Microsserviço de Agendamento de Tarefas

Microsserviço responsável pelo CRUD de tarefas agendadas pelos usuários, incluindo controle de status de notificação, com persistência em MongoDB e autenticação JWT validada de forma distribuída junto ao microsserviço de Usuário.

## 📌 Sobre o projeto

Este serviço faz parte de uma aplicação distribuída em arquitetura de microsserviços, composta por:

- [**BFF**](#) — orquestra as chamadas para os demais serviços
- [**Usuário**](#) — gerenciamento de usuários, autenticação e endereços
- **Agendador de Tarefas** (este repositório) — CRUD de tarefas agendadas
- [**Notificação**](#) — envio de e-mails de notificação

## 🚀 Tecnologias utilizadas

- **Java 17**
- **Spring Boot 4** / Spring Framework 7
- **Spring Data MongoDB** — persistência NoSQL das tarefas
- **Spring Security** + **JWT** (jjwt) — autenticação e autorização
- **Spring Cloud OpenFeign** — comunicação com o microsserviço de Usuário para validação de dados do usuário autenticado
- **MapStruct** — mapeamento entre entidades e DTOs (incluindo atualização parcial com `@MappingTarget`)
- **Gradle** — gerenciamento de dependências e build
- **Docker** / Docker Compose — containerização
- **Lombok**

## 🏗️ Arquitetura

Estrutura em camadas:

```
controller/
  TarefasController              → Endpoints REST de tarefas

business/
  TarefasService                  → Regras de negócio: criação, busca, atualização, exclusão e status
  dto/                             → TarefasDTO, UsuarioDTO
  mapper/
    TarefasConverter                → Conversão entre TarefasDTO e TarefasEntity (MapStruct)
    TarefaUpdateConverter            → Atualização parcial de entidade, ignorando campos nulos do DTO (MapStruct)

infrastructure/
  entity/TarefasEntity             → Documento MongoDB da tarefa
  enums/StatusNotificacaoEnum        → Status da tarefa (PENDENTE, NOTIFICADO, CANCELADO)
  repository/TarefasRepository       → Spring Data MongoDB, com buscas por período e por e-mail
  client/UsuarioClient               → Feign Client para o microsserviço de Usuário
  security/
    SecurityConfig                    → Configuração do Spring Security (stateless, JWT)
    JwtUtil                            → Geração/validação de token JWT
    JwtRequestFilter                   → Filtro de autenticação via Bearer Token
    UserDetailsServiceImpl              → Carrega os dados do usuário autenticado consultando o microsserviço de Usuário via Feign
  exceptions/                        → ResourceNotFoundException, UnauthorizedException
```

## 🔗 Autenticação distribuída entre microsserviços

Um destaque arquitetural deste serviço: ele não possui uma base de usuários própria. Ao validar um token JWT, o `UserDetailsServiceImpl` consulta o microsserviço de **Usuário** via Feign (`UsuarioClient`) para obter os dados da conta autenticada, mantendo a responsabilidade de gerenciamento de usuários isolada em um único serviço.

## 📋 Funcionalidades

- Criação de tarefas, com data de criação e status inicial (`PENDENTE`) definidos automaticamente, associadas ao e-mail extraído do token JWT
- Busca de tarefas por período (data inicial e final), filtrando apenas tarefas com status `PENDENTE`
- Busca de tarefas por e-mail do usuário autenticado
- Atualização parcial de tarefas (apenas os campos informados no corpo da requisição são alterados)
- Alteração do status de notificação da tarefa (`PENDENTE`, `NOTIFICADO`, `CANCELADO`)
- Exclusão de tarefas por ID
- Persistência otimizada para dados não relacionais com MongoDB

## ⚙️ Como executar

### Pré-requisitos
- Java 17
- Gradle
- MongoDB (ou via Docker Compose)
- Microsserviço de Usuário em execução (necessário para a validação de autenticação)

### Rodando localmente

```bash
./gradlew clean build
./gradlew bootRun
```

### Rodando com Docker Compose

```bash
docker compose up --build
```

## 🔒 Segurança

Todos os endpoints exigem um token JWT válido (Bearer Token), emitido pelo microsserviço de Usuário no login. O token é validado localmente, mas os dados do usuário são obtidos em tempo real do microsserviço de Usuário.

## 👩‍💻 Autora

**Thaís Rodrigues Vieira**
[LinkedIn](https://www.linkedin.com/in/thais-vieira-8471523a2/) | [GitHub](https://github.com/thaisrvieira)
