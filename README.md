<div align="center">
    <img src="https://thesimpsonsapi.com/hero.webp" alt="The Simpsons Banner" width="300" />
</div>

# 🍩 The Simpsons API

RESTful API for The Simpsons data. Built with NestJS, Prisma, and PostgreSQL.

## 🚀 Quick Start

### Option 1: Docker Compose (Recommended)

```bash
# 1. Clone the repository
git clone https://github.com/liusc45/the-simpsons-api.git
cd the-simpsons-api/api

# 2. Create environment file
cp .env.example .env
# Edit .env and set your API_KEY (required for POST requests)

# 3. Start services with Docker Compose
docker-compose up --build

# 4. Run database migrations
docker exec simpsons-api npx prisma migrate deploy

# API will be available at http://localhost:3000/api
```

### Option 2: Local Development

```bash
# 1. Clone the repository
git clone https://github.com/liusc45/the-simpsons-api.git
cd the-simpsons-api

# 2. Start PostgreSQL (if not using Docker Compose)
cd api && docker-compose up postgresql -d

# 3. Install dependencies
npm install

# 4. Setup environment
cp api/.env.example api/.env
# Edit api/.env and configure DATABASE_URL and API_KEY

# 5. Run migrations and start development server
cd api
npx prisma migrate deploy
npm run dev
```

## 🐳 Docker Setup

### Environment Variables

Create a `.env` file in the `api/` directory with the following variables:

```env
# NestJS Configuration
LOGGER_LEVEL=log
PORT=3000
APP_URL=http://localhost:3000/api
API_KEY=your-secret-api-key-here

# Database Configuration
DATABASE_URL="postgresql://user:password@postgresql:5432/user"

# Docker (optional for private npm packages)
NPM_TOKEN=
NODE_ENV=production
```

### Docker Compose Services

The `docker-compose.yml` includes:

- **PostgreSQL**: Database with persistent volume (`postgres-data`)
- **API**: NestJS application with multistage build
- **Network**: `simpsons-network` for service communication
- **Health Checks**: Ensures PostgreSQL is ready before starting API

### Useful Docker Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f api
docker-compose logs -f postgresql

# Stop services
docker-compose down

# Rebuild and restart
docker-compose up --build -d

# Check service status
docker-compose ps

# Execute commands in containers
docker exec simpsons-api npm run test
docker exec simpsons-postgres psql -U user -d user
```

## 📊 Fill Database

### Method 1: Bruno Collection (Recommended)

Use the included `api-request-collection`:

1. **Open Bruno**: Open the `api-request-collection` folder
2. **Set API Key**: Configure your API_KEY in Bruno environment
3. **Execution order**:
   ```
   1. POST /api/episodes    → Create episodes
   2. POST /api/shorts      → Create shorts
   3. POST /api/locations   → Create locations
   4. POST /api/characters  → Create characters
   ```
4. **Run in order**: Each request has required data

### Method 2: Manual with curl

```bash
# Note: Use the API_KEY from your .env file

# Create episodes
curl -X POST http://localhost:3000/api/episodes \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d @api-request-collection/episode/create-many.bru

# Create characters (requires episodes to exist first)
curl -X POST http://localhost:3000/api/characters \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer your-api-key" \
  -d @api-request-collection/character/create-many.bru
```

## 🔧 Useful Commands

### Database Management

```bash
# Reset database (Docker)
docker exec simpsons-api npx prisma migrate reset --force

# Reset database (Local)
cd api && npx prisma migrate reset --force

# Check data counts
docker exec simpsons-postgres psql -U user -d user -c "SELECT COUNT(*) FROM \"Episode\";"
docker exec simpsons-postgres psql -U user -d user -c "SELECT COUNT(*) FROM \"Character\";"

# Backup database
docker exec simpsons-postgres pg_dump -U user user > backup.sql

# Restore database
docker exec -i simpsons-postgres psql -U user user < backup.sql
```

### Testing

```bash
# Run all tests (Docker)
docker exec simpsons-api npm run test:e2e

# Run all tests (Local)
npm run test:e2e

# Run specific test file
npm run test:e2e -- episodes.e2e-spec.ts
```

## 🐛 Troubleshooting

### Port Already in Use

If port 3000 or 5432 is already in use:

```bash
# Find and kill process using port 3000
lsof -ti:3000 | xargs kill -9

# Or change ports in docker-compose.yml:
ports:
  - "3001:3000"  # Map to different host port
```

### Database Connection Issues

```bash
# Check if PostgreSQL is healthy
docker-compose ps

# View PostgreSQL logs
docker-compose logs postgresql

# Ensure DATABASE_URL in .env matches docker-compose settings
DATABASE_URL="postgresql://user:password@postgresql:5432/user"
```

### Missing Prisma Schema

If you get "Prisma schema not found":

```bash
# Ensure prisma folder is copied in Dockerfile production stage
COPY --from=build $DIR/prisma prisma

# Rebuild the image
docker-compose up --build
```

### API Returns 401 Unauthorized

Ensure you're using the correct API_KEY from your `.env` file:

```bash
# Check API_KEY in container
docker exec simpsons-api printenv | grep API_KEY

# Use correct header format
curl -H "Authorization: Bearer your-api-key" http://localhost:3000/api/episodes
```

## 📚 Documentation

- **API Documentation**: https://thesimpsonsapi.com/
- **Bruno Collection**: `api-request-collection/`
- **Prisma Schema**: `api/prisma/schema.prisma`

## 🏗️ Architecture

```
the-simpsons-api/
├── api/
│   ├── src/               # NestJS source code
│   ├── prisma/            # Database schema and migrations
│   ├── Dockerfile         # Multistage Docker build
│   ├── docker-compose.yml # Service orchestration
│   └── .env.example       # Environment template
├── ui/                    # Frontend application
└── README.md
```

### Technologies

- **Backend**: NestJS, TypeScript
- **Database**: PostgreSQL 16, Prisma ORM
- **Containerization**: Docker, Docker Compose
- **API Testing**: Bruno
- **Testing**: Vitest, Supertest

## 📚 Data Attribution

This project uses data from [The Simpsons Wiki](https://simpsons.fandom.com/wiki/The_Simpsons_Wiki) under the [Creative Commons Attribution-ShareAlike License (CC BY-SA)](https://creativecommons.org/licenses/by-sa/3.0/).