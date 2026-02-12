# 🛠️ Development Guide - Mediguk

Guía completa para desarrolladores que trabajen en el proyecto Mediguk.

---

## 📋 Índice

- [Tech Stack Detallado](#-tech-stack-detallado)
- [Configuración del Entorno](#-configuración-del-entorno)
- [Code Standards](#-code-standards)
- [Git Workflow](#-git-workflow)
- [Testing](#-testing)
- [Deployment](#-deployment)

---

## 🛠 Tech Stack Detallado

### Backend

#### Spring Boot (API Gateway & Orchestration)
```
Java: 17 LTS
Spring Boot: 3.2.x
Spring Security: OAuth2 + JWT
Spring Data JPA
```

**Dependencias clave:**
- `spring-boot-starter-web` - REST APIs
- `spring-boot-starter-security` - Autenticación
- `spring-boot-starter-data-jpa` - ORM
- `spring-cloud-starter-gateway` - API Gateway
- `springdoc-openapi` - API Documentation

**Build Tool:** Maven

#### Python ML Service
```
Python: 3.10+
Framework: FastAPI
ML: PyTorch / TensorFlow
```

**Dependencias clave:**
```
fastapi==0.104.1
uvicorn==0.24.0
torch==2.1.0
transformers==4.35.0
pillow==10.1.0
numpy==1.26.0
scikit-learn==1.3.2
```

**Virtual Environment:** `venv` o `poetry`

#### Go Image Service
```
Go: 1.21+
Framework: Gin
Image Processing: imaging library
```

**Dependencias clave:**
```go
github.com/gin-gonic/gin
github.com/disintegration/imaging
github.com/aws/aws-sdk-go-v2/service/s3
```

### Frontend

#### Next.js (Patient & Health Worker Apps)
```
Node.js: 18 LTS
Next.js: 14.x (App Router)
React: 18.x
TypeScript: 5.x
```

**Dependencias clave:**
```json
{
  "next": "^14.0.0",
  "react": "^18.2.0",
  "typescript": "^5.3.0",
  "tailwindcss": "^3.3.0",
  "@tanstack/react-query": "^5.0.0",
  "axios": "^1.6.0",
  "zod": "^3.22.0",
  "react-hook-form": "^7.48.0"
}
```

**UI Library:** Tailwind CSS + shadcn/ui

### Databases & Storage

#### PostgreSQL
```
Version: 14+
Extensions: 
  - pg_trgm (fuzzy search)
  - uuid-ossp (UUIDs)
```

**Schema Management:** Flyway migrations

#### Redis
```
Version: 7.x
Use cases:
  - API response cache
  - Session storage
  - Rate limiting
```

#### S3-Compatible Storage
```
Provider: MinIO (local dev) / AWS S3 (production)
Use: Medical images, audit logs
```

### Infrastructure

#### Message Queue
```
RabbitMQ 3.12+
or
Apache Kafka 3.5+
```

**Queues:**
- `image.processing`
- `ml.stage1`
- `ml.stage2`
- `ml.stage3`
- `notifications`

#### Containerization
```
Docker: 24.x
Docker Compose: 2.x (for local dev)
Kubernetes: 1.28+ (for production)
```

---

## 🚀 Configuración del Entorno

### Prerrequisitos
```bash
# Instalar herramientas base
- Java 17 (SDKMAN recomendado)
- Node.js 18 LTS (nvm recomendado)
- Python 3.10+ (pyenv recomendado)
- Go 1.21+
- Docker & Docker Compose
- PostgreSQL 14+
- Redis 7+
```

### Setup Backend (Spring Boot)
```bash
# Clonar repo
git clone https://github.com/MediGuk/mediguk-backend.git
cd mediguk-backend

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tus credenciales

# Instalar dependencias y compilar
./mvnw clean install

# Ejecutar
./mvnw spring-boot:run

# API disponible en: http://localhost:8080
```

**application.yml:**
```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mediguk
    username: ${DB_USER}
    password: ${DB_PASSWORD}
  
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false

  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: ${JWT_ISSUER_URI}
```

### Setup ML Service (Python)
```bash
# Clonar repo
git clone https://github.com/MediGuk/mediguk-ml-service.git
cd mediguk-ml-service

# Crear virtual environment
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt

# Configurar variables de entorno
cp .env.example .env

# Ejecutar
uvicorn app.main:app --reload --port 8001

# API disponible en: http://localhost:8001
```

### Setup Image Service (Go)
```bash
# Clonar repo
git clone https://github.com/MediGuk/mediguk-image-service.git
cd mediguk-image-service

# Instalar dependencias
go mod download

# Configurar .env
cp .env.example .env

# Ejecutar
go run main.go

# Service disponible en: http://localhost:8002
```

### Setup Frontend (Next.js)
```bash
# Clonar repo
git clone https://github.com/MediGuk/mediguk-frontend.git
cd mediguk-frontend

# Instalar dependencias
npm install

# Configurar variables de entorno
cp .env.local.example .env.local

# Ejecutar en desarrollo
npm run dev

# App disponible en: http://localhost:3000
```

### Docker Compose (Todo junto)
```bash
# En la raíz del proyecto
docker-compose up -d

# Servicios disponibles:
# - Backend: http://localhost:8080
# - ML Service: http://localhost:8001
# - Image Service: http://localhost:8002
# - Frontend: http://localhost:3000
# - PostgreSQL: localhost:5432
# - Redis: localhost:6379
```

---

## 📏 Code Standards

### Linting & Formatting

#### Backend (Java)

**Checkstyle** + **Google Java Format**

`checkstyle.xml`:
```xml
<?xml version="1.0"?>
<!DOCTYPE module PUBLIC
  "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
  "https://checkstyle.org/dtds/configuration_1_3.dtd">
<module name="Checker">
  <module name="TreeWalker">
    <module name="Indentation">
      <property name="basicOffset" value="2"/>
    </module>
    <module name="LineLength">
      <property name="max" value="120"/>
    </module>
  </module>
</module>
```

**Maven plugin:**
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-checkstyle-plugin</artifactId>
  <version>3.3.0</version>
  <configuration>
    <configLocation>checkstyle.xml</configLocation>
  </configuration>
</plugin>
```

**Ejecutar:**
```bash
./mvnw checkstyle:check
```

#### Python

**Black** + **Flake8** + **isort**

`pyproject.toml`:
```toml
[tool.black]
line-length = 100
target-version = ['py310']

[tool.isort]
profile = "black"
line_length = 100

[tool.flake8]
max-line-length = 100
extend-ignore = E203, W503
```

**requirements-dev.txt:**
```
black==23.11.0
flake8==6.1.0
isort==5.12.0
mypy==1.7.0
```

**Ejecutar:**
```bash
black .
isort .
flake8 .
mypy .
```

#### Frontend (TypeScript/Next.js)

**ESLint** + **Prettier**

`.eslintrc.json`:
```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "plugins": ["@typescript-eslint"],
  "rules": {
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
    "react-hooks/exhaustive-deps": "warn"
  }
}
```

`.prettierrc`:
```json
{
  "semi": true,
  "trailingComma": "es5",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2
}
```

**package.json scripts:**
```json
{
  "scripts": {
    "lint": "next lint",
    "format": "prettier --write .",
    "type-check": "tsc --noEmit"
  }
}
```

---

### Pre-commit Hooks (Husky)

**Frontend setup:**
```bash
npm install --save-dev husky lint-staged

npx husky install

npx husky add .husky/pre-commit "npx lint-staged"
```

**package.json:**
```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write"
    ],
    "*.{json,md}": [
      "prettier --write"
    ]
  }
}
```

**Backend (Maven):**

Usar `git-code-format-maven-plugin`:
```xml
<plugin>
  <groupId>com.cosium.code</groupId>
  <artifactId>git-code-format-maven-plugin</artifactId>
  <version>4.3</version>
  <executions>
    <execution>
      <goals>
        <goal>install-hooks</goal>
      </goals>
    </execution>
  </executions>
</plugin>
```

---

## 🔀 Git Workflow

### Branch Strategy
```
main                  ← Producción (protegida)
  ↑
develop              ← Desarrollo activo
  ↑
feature/ISSUE-123    ← Features nuevas
bugfix/ISSUE-456     ← Bug fixes
hotfix/CRITICAL      ← Fixes urgentes en producción
```

### Naming Conventions

**Branches:**
```
feature/triaje-system
feature/ml-stage3
bugfix/image-upload-error
hotfix/security-patch
```

**Commits:** Conventional Commits
```
feat: add VLM stage 3 reconciliation logic
fix: resolve image upload timeout issue
docs: update architecture documentation
test: add unit tests for data analysis
chore: update dependencies
refactor: simplify orchestration service
```

**Ejemplos:**
```bash
git commit -m "feat(ml): implement stage 3 VLM review"
git commit -m "fix(api): handle null pointer in patient service"
git commit -m "docs: add development setup guide"
```

### Pull Request Process

1. **Crear branch** desde `develop`
2. **Desarrollar** feature/fix
3. **Commits** siguiendo Conventional Commits
4. **Push** y abrir PR a `develop`
5. **Code review** (al menos 1 aprobación)
6. **CI/CD** pasa (tests, linting)
7. **Merge** con squash commit

**PR Template:**
```markdown
## Descripción
[Descripción breve de los cambios]

## Tipo de cambio
- [ ] Feature nueva
- [ ] Bug fix
- [ ] Breaking change
- [ ] Documentación

## Testing
- [ ] Unit tests añadidos/actualizados
- [ ] Integration tests pasan
- [ ] Probado manualmente

## Checklist
- [ ] Código sigue code standards
- [ ] Documentación actualizada
- [ ] No hay warnings en linter
- [ ] Tests pasan
```

---

## 🧪 Testing

### Backend (Java)

**JUnit 5** + **Mockito** + **Spring Boot Test**
```java
@SpringBootTest
class PatientServiceTest {
  
  @Autowired
  private PatientService patientService;
  
  @MockBean
  private PatientRepository patientRepository;
  
  @Test
  void shouldCreatePatient() {
    // Given
    Patient patient = new Patient("John Doe");
    when(patientRepository.save(any())).thenReturn(patient);
    
    // When
    Patient result = patientService.createPatient(patient);
    
    // Then
    assertNotNull(result);
    assertEquals("John Doe", result.getName());
  }
}
```

**Ejecutar:**
```bash
./mvnw test
./mvnw verify  # Include integration tests
```

### Python

**pytest** + **pytest-cov**
```python
import pytest
from app.services.ml_service import MLService

@pytest.fixture
def ml_service():
    return MLService()

def test_vlm_analysis(ml_service):
    # Given
    image_path = "tests/fixtures/rash.jpg"
    symptoms = "Red itchy rash on arm"
    
    # When
    result = ml_service.analyze_stage1(image_path, symptoms)
    
    # Then
    assert result.urgency >= 0
    assert result.urgency <= 10
    assert result.confidence > 0.5
```

**Ejecutar:**
```bash
pytest
pytest --cov=app --cov-report=html
```

### Frontend (Next.js)

**Jest** + **React Testing Library**
```typescript
import { render, screen } from '@testing-library/react';
import { UploadForm } from '@/components/UploadForm';

describe('UploadForm', () => {
  it('should render upload button', () => {
    render(<UploadForm />);
    
    const button = screen.getByRole('button', { name: /upload/i });
    expect(button).toBeInTheDocument();
  });
  
  it('should validate file type', async () => {
    render(<UploadForm />);
    
    const file = new File(['content'], 'test.txt', { type: 'text/plain' });
    const input = screen.getByLabelText(/upload photo/i);
    
    await userEvent.upload(input, file);
    
    expect(screen.getByText(/invalid file type/i)).toBeInTheDocument();
  });
});
```

**package.json:**
```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

---

## 🚢 Deployment

### Environments
```
Development  → localhost
Staging      → staging.mediguk.dev
Production   → api.mediguk.com
```

### CI/CD (GitHub Actions)

`.github/workflows/backend.yml`:
```yaml
name: Backend CI/CD

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
          
      - name: Run tests
        run: ./mvnw verify
        
      - name: Run Checkstyle
        run: ./mvnw checkstyle:check
```

### Docker Build

**Backend Dockerfile:**
```dockerfile
FROM eclipse-temurin:17-jdk-alpine AS build
WORKDIR /app
COPY . .
RUN ./mvnw clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

**Build & Run:**
```bash
docker build -t mediguk-backend .
docker run -p 8080:8080 mediguk-backend
```

---

## 📝 Convenciones Adicionales

### Naming

**Variables/Functions:**
- `camelCase` para JavaScript/TypeScript/Java
- `snake_case` para Python
- `PascalCase` para clases/componentes

**Archivos:**
- `kebab-case.tsx` para componentes React
- `PascalCase.java` para clases Java
- `snake_case.py` para módulos Python

### Comentarios

**Documenta:**
- ✅ APIs públicas
- ✅ Lógica compleja no obvia
- ✅ Decisiones arquitectónicas importantes

**No documentes:**
- ❌ Código auto-explicativo
- ❌ Comentarios redundantes

**Ejemplo bueno:**
```typescript
// Calculate urgency using safety-first logic:
// Take the maximum urgency from all stages to ensure
// we never under-triage a potentially serious condition
const finalUrgency = Math.max(stage1.urgency, stage2.urgency, stage3?.urgency ?? 0);
```

**Ejemplo malo:**
```typescript
// Set urgency to max
const urgency = Math.max(a, b, c);
```

---

## 🔗 Enlaces Útiles

- [Spring Boot Docs](https://spring.io/projects/spring-boot)
- [Next.js Docs](https://nextjs.org/docs)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Go Gin Docs](https://gin-gonic.com/docs/)

---

<p align="center">
  <i>Última actualización: Febrero 2025</i>
</p>
```
