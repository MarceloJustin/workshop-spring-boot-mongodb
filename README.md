# Workshop Spring Boot + MongoDB

![Java](https://img.shields.io/badge/Java-21-007396?style=flat-square&logo=openjdk&logoColor=white)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-3.5.7-6DB33F?style=flat-square&logo=springboot&logoColor=white)
![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white)
![Maven](https://img.shields.io/badge/Maven%20Wrapper-3.9.11-C71A36?style=flat-square&logo=apachemaven&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=flat-square)

## 📌 Sobre o Projeto

API REST desenvolvida como workshop prático de integração entre **Spring Boot** e **MongoDB**. O projeto modela uma rede social simplificada com usuários, posts e comentários, explorando os principais conceitos do Spring Data MongoDB: mapeamento de documentos, referências entre coleções (`@DBRef`), subdocumentos embutidos e consultas customizadas com `@Query`.

## 🚀 Tecnologias Utilizadas

- **Java 21**
- **Spring Boot 3.5.7**
- **Spring Data MongoDB**
- **Spring Web (REST)**
- **Maven Wrapper 3.9.11** (não é necessário instalar o Maven — o wrapper cuida disso)

## 🏗️ Arquitetura

O projeto segue o padrão em camadas **Resource → Service → Repository**:

```
src/main/java/com/nelioalves/workshopmongo/
├── config/          # Seed de dados na inicialização (CommandLineRunner)
├── domain/          # Entidades MongoDB (User, Post)
├── dto/             # Objetos de transferência (UserDTO, AuthorDTO, CommentDTO)
├── repository/      # Interfaces MongoRepository
├── resources/       # Controllers REST + tratamento de exceções
│   ├── exception/   # StandardError, ResourseExceptionHandler
│   └── util/        # Utilitário de decodificação de parâmetros de URL
└── services/        # Regras de negócio + exceções de domínio
    └── exception/   # ObjectNotFoundException
```

**Modelo de dados:**
- `User` referencia seus posts via `@DBRef(lazy = true)` — referência entre coleções
- `Post` embute `AuthorDTO` e `List<CommentDTO>` diretamente no documento — subdocumentos

## 📋 Funcionalidades

**Usuários**
- CRUD completo (criar, listar, buscar por ID, atualizar, deletar)
- Listar todos os posts de um usuário

**Posts**
- Buscar post por ID
- Buscar posts por título (regex, case-insensitive, via `@Query`)
- Busca completa com filtro de intervalo de datas — pesquisa em título, corpo e texto dos comentários simultaneamente

**Inicialização**
- Ao subir a aplicação, a classe `Instantiation` limpa e recria o banco com dados de exemplo (3 usuários, 2 posts com comentários)

## 🔗 Principais Endpoints

### Usuários — `/users`

| Método | Endpoint          | Descrição                        | Status de sucesso |
|--------|-------------------|----------------------------------|-------------------|
| GET    | `/users`          | Lista todos os usuários          | 200 OK            |
| GET    | `/users/{id}`     | Busca usuário por ID             | 200 OK            |
| POST   | `/users`          | Cria novo usuário                | 201 Created       |
| PUT    | `/users/{id}`     | Atualiza usuário                 | 204 No Content    |
| DELETE | `/users/{id}`     | Remove usuário                   | 204 No Content    |
| GET    | `/users/{id}/posts` | Lista posts do usuário         | 200 OK            |

### Posts — `/posts`

| Método | Endpoint                  | Descrição                                          | Status de sucesso |
|--------|---------------------------|-----------------------------------------------------|-------------------|
| GET    | `/posts/{id}`             | Busca post por ID                                   | 200 OK            |
| GET    | `/posts/titlesearch`      | Busca posts por título (`?text=`)                   | 200 OK            |
| GET    | `/posts/fullsearch`       | Busca completa (`?text=&minDate=&MaxDate=`)         | 200 OK            |

## 📨 Exemplos de Requisição

**Criar usuário**
```http
POST /users
Content-Type: application/json

{
  "name": "João Silva",
  "email": "joao@email.com"
}
```

**Busca completa com filtro de datas**
```http
GET /posts/fullsearch?text=viagem&minDate=2018-03-01&MaxDate=2018-03-31
```

Retorna posts onde o texto aparece no título, corpo ou em qualquer comentário, dentro do intervalo de datas informado. A data máxima é estendida em 1 dia internamente para incluir o dia inteiro.

**Resposta de um post**
```json
{
  "id": "64a1b2c3...",
  "date": "2018-03-21T00:00:00.000+00:00",
  "title": "Partiu viagem",
  "body": "Vou viajar para São Paulo. Abraços!",
  "author": {
    "id": "64a1b2c1...",
    "name": "Maria Brown"
  },
  "comments": [
    {
      "text": "Boa Viagem mano",
      "date": "2018-03-21T00:00:00.000+00:00",
      "author": { "id": "64a1b2c2...", "name": "Alex Green" }
    }
  ]
}
```

## ⚠️ Tratamento de Erros

Exceções são interceptadas centralmente por `ResourseExceptionHandler` e retornam um corpo padronizado (`StandardError`):

```json
{
  "timestamp": 1718000000000,
  "status": 404,
  "error": "Não encontrado",
  "message": "Objeto não encontrado",
  "path": "/users/id-inexistente"
}
```

| Situação               | Status HTTP | Mensagem          |
|------------------------|-------------|-------------------|
| ID não encontrado      | 404         | Objeto não encontrado |

## 🧪 Testes

O projeto conta apenas com o teste de contexto padrão gerado pelo Spring Initializr (`contextLoads()`), sem testes de negócio escritos.

## ▶️ Como Executar

**Pré-requisitos**
- Java 21
- MongoDB rodando em `localhost:27017`

**Clonar o repositório**
```bash
git clone https://github.com/MarceloJustin/workshopmongo.git
cd workshopmongo
```

**Configuração**

A conexão com o banco está em `src/main/resources/application.properties`:
```properties
spring.data.mongodb.uri=mongodb://localhost:27017/workshop_mongo
```

Ajuste a URI caso seu MongoDB use porta, host ou credenciais diferentes.

**Executar**

Linux / macOS:
```bash
./mvnw spring-boot:run
```

Windows:
```bash
mvnw.cmd spring-boot:run
```

A API estará disponível em `http://localhost:8080`. Ao iniciar, o banco é automaticamente populado com dados de exemplo.

## 👨‍💻 Autor

**Marcelo Justin**

[![GitHub](https://img.shields.io/badge/GitHub-MarceloJustin-181717?style=flat-square&logo=github)](https://github.com/MarceloJustin)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-marcelojustin-0A66C2?style=flat-square&logo=linkedin)](https://www.linkedin.com/in/marcelojustin)

## 📄 Licença

Este projeto está sob a licença MIT. Consulte o arquivo [LICENSE](LICENSE) para mais detalhes.
