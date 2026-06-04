# Explora-map: deploy

Configuración de despregamento de Explora en produción mediante Docker Compose, Nginx e Let's Encrypt.

## Infraestrutura

- Servidor: Hetzner CX22 (Falkenstein, Alemaña)
- Sistema operativo: Debian 12
- Dominio: explora-mapa.eu (rexistrado en Gandi)
- HTTPS: Let's eNCRYPT / Certbot
- Postal (autohospedado) : Brevo

---

## Contido do repositorio

```
deploy/
├── docker-compose.yml            # Stack principal de produción
├── postal-docker-compose.yml     # Stack do servidor de correo Postal
├── .env.example                  # Plantilla de variables de entorno
└── README.md
```

> O ficheiro `.env` con credenciais reais non está no repositorio. Vive exclusivamente no servidor.

---

## Stack de produción (`docker-compose.yml`)

O stack principal levanta tres servizos:

- postgres: PostgreSQL 16 con extensión PostGIS (postgis/postgis:16-3.4-alpine). Inclúe healthcheck; o backend non arrinca ata que a BD estea lista.
- backend:API REST Spring Boot compilada desde ./backend, con perfil prod activo. Expón o porto 8080.
- frontend: Contedor que integra Nginx, serve o build estático da SPA e xestiona a terminación TLS directamente. Expón os portos 80 e 443, monta os certificados de Let's Encrypt en modo só lectura e o directorio webroot de Certbot para renovacións.

Os contedores backend e frontend están conectados á rede externa postal_postal-net para comunicarse co servidor de correo Postal, que corre nun stack independente.

---

## Variables de entorno

Copia `.env.example` a `.env` e cubre os valores antes de arrincar:

```bash
cp .env.example .env
```
- POSTGRES_USER: Usuario de PostgreSQL
- POSTGRES_PASSWORD: Contrasinal de PostgreSQL
- CORS_ALLOWED_ORIGINS: Orixes permitidas para CORS (ex. https://explora-mapa.eu)
- JWT_SECRET:  Segredo para firma de tokens JWT (mínimo 32 caracteres)
- MAIL_HOST: Host do servidor SMTP
- MAIL_PORT: Porto SMTP
- MAIL_USERNAME: Usuario SMTP
- MAIL_PASSWORD: Contrasinal SMTP
- APP_BASE_URL: URL pública do frontend (ex. https://explora-mapa.eu)

---

## Despregamento

### Requisitos previos

- Docker e Docker Compose instalados no servidor
- Certificado TLS obtido con Certbot e montado en Nginx
- Ficheiro `.env` cuberto con todas as variables

### Arrincar o stack

```bash
docker compose up -d
```

### Ver logs

```bash
docker compose logs -f backend
docker compose logs -f frontend
```

### Actualizar a aplicación

```bash
docker compose pull
docker compose up -d --no-deps backend
```

---

## Servidor de correo (Postal)

O ficheiro postal-docker-compose.yml configura unha instancia de Postal como servidor de correo transaccional autoaloxado para o envío de correos de verificación de conta.

Arrincar de forma independente:

```bash
docker compose -f postal-docker-compose.yml up -d
```

---

## Seguridade

- As credenciais nunca se commiten ao repositorio.
- O JWT secret debe ser único e xerado de forma aleatoria para cada instalación.
- O acceso SSH ao servidor restrinxese mediante claves públicas.
- Os portos da base de datos non están expostos publicamente; só son accesibles entre os contedores da rede interna de Docker.

---

## Repositorios relacionados

- Frontend: SPA React + Leaflet
- Bckend: API REST Spring Boot
- Docs: Memoria técnica, diagramas e guía de estilos

---

## Licenza

Este proxecto está publicado baixo licenza GNU GENERAL PUBLIC LICENSE. Consulta o ficheiro `LICENSE` para máis información.