# ODR – Backend

Java 21 · Spring Boot 4 · Spring Data JPA · Spring Security · Flyway · PostgreSQL

## Estrutura

```
Backend/
├── Dockerfile
├── pom.xml
└── src/
    ├── main/
    │   ├── java/odr_con/
    │   │   ├── Application.java
    │   │   ├── config/        # configurações (segurança, CORS, beans)
    │   │   ├── controller/    # endpoints REST
    │   │   ├── dto/           # objetos de entrada/saída da API
    │   │   ├── entity/        # entidades JPA
    │   │   ├── exception/     # exceções e handler global
    │   │   ├── repository/    # interfaces Spring Data
    │   │   ├── security/      # filtros/JWT (quando forem criados)
    │   │   └── service/       # regras de negócio
    │   └── resources/
    │       ├── application.yaml          # base
    │       ├── application-dev.yaml      # local
    │       ├── application-prod.yaml     # hospedagem (variáveis de ambiente)
    │       └── db/migration/             # migrações Flyway (V1__, V2__ ...)
    └── test/                              # testes (H2 em memória, perfil "test")
```

## Rodar localmente

Com Docker (API + PostgreSQL):

```bash
docker compose up --build        # na raiz do repositório
```

Ou só o banco no Docker e a API pela IDE/Maven:

```bash
docker compose up -d db
cd Backend
./mvnw spring-boot:run           # perfil dev, usa localhost:5432/odr
```

Health check: `GET http://localhost:8080/actuator/health`

## Banco de dados

- O schema é criado e versionado pelo **Flyway**. Para alterar o banco, crie uma nova migração em
  `src/main/resources/db/migration` (ex.: `V2__adicionar_coluna_x.sql`). **Nunca edite uma migração já aplicada.**
- O Hibernate está em `ddl-auto: validate`: ele só confere se as entidades batem com o schema.
- `schema.sql` e `odr.obs` na raiz são os arquivos SQLite originais, mantidos só como referência.

## Deploy

O `Backend/Dockerfile` funciona em Render, Railway, Fly.io, etc. (diretório de build: `Backend`).
Variáveis de ambiente necessárias em produção:

| Variável | Exemplo |
|---|---|
| `SPRING_PROFILES_ACTIVE` | `prod` (já é o padrão da imagem) |
| `DB_URL` | `jdbc:postgresql://HOST:5432/NOME` |
| `DB_USER` | usuário do banco |
| `DB_PASSWORD` | senha do banco |
| `CORS_ALLOWED_ORIGINS` | `https://seu-front.com` |
| `PORT` | normalmente injetada pela plataforma |

Health check da plataforma: `/actuator/health`.
