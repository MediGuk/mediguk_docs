# 🏥 Mediguk - Documentación del Sistema

> Documentación completa, arquitectura y diagramas del sistema inteligente de triaje médico Mediguk

---

## 📚 Contenido de este Repositorio

- **[Arquitectura del Sistema](#-arquitectura-del-sistema)** - Diseño técnico y flujo de datos
- **[Repositorios del Proyecto](#-repositorios-del-proyecto)** - Organización del código
- **[Tech Stack](#-tech-stack)** - Tecnologías utilizadas
- **[Roadmap](#-roadmap)** - Plan de desarrollo

---

## 🎯 Visión General del Proyecto

### El Problema

Los sistemas de salud públicos enfrentan una crisis de sobrecarga:

- ⏰ **Tiempos de espera excesivos**: Pacientes esperan horas para ser atendidos
- 👨‍⚕️ **Personal sanitario sobrecargado**: Médicos y enfermeras trabajando al límite
- 🚑 **Triaje ineficiente**: Casos menores ocupan recursos de urgencias
- ❌ **Casos urgentes sin detectar**: Falta de priorización puede tener consecuencias graves

### La Solución: Mediguk

Sistema de triaje inteligente que utiliza IA para:

1. **Analizar síntomas del paciente** (texto + imagen + audio opcional)
2. **Clasificar urgencia automáticamente** (Baja/Media/Alta)
3. **Comparar con casos históricos** (pattern matching con datos reales de la clínica)
4. **Rutear al nivel apropiado** (Auxiliar → Enfermera → Médico General → Especialista)

### 🔑 Principio Fundamental

> **El profesional sanitario SIEMPRE toma la decisión final.**  
> Mediguk es una herramienta de apoyo, no un reemplazo del juicio médico.

---

## 🏗️ Arquitectura del Sistema

### Sistema de 3 Etapas Condicional
```
Paciente (foto + texto + audio opcional)
    ↓
    
STAGE 1: VLM Initial Analysis
- Analiza imagen (features visuales)
- Analiza texto (síntomas, duración)
- Extrae características clave
- Output: Condición preliminar, urgencia (0-10), confianza (%)
    ↓
    
STAGE 2: Data Analysis Engine
- Recibe output del VLM
- Busca en base de datos histórica de la clínica
- Pattern matching con casos reales
- Output: Condiciones coincidentes (%), urgencia (0-10), confianza (%)
    ↓
    
Decision Point: ¿Desacuerdo o incertidumbre?
- ¿Urgencia difiere >2 puntos?
- ¿Condiciones diferentes?
- ¿Confianza <70%?
    ↓
   SÍ (20-30% casos)          NO (70-80% casos)
    ↓                              ↓
STAGE 3: VLM Review           Skip Stage 3
- Revisa hallazgos data       (Acuerdo - procede)
- Re-examina foto                  ↓
- Reconcilia assessment
- Output: Urgencia final
    ↓                              ↓
    └──────────┬─────────────────┘
               ↓
               
Final System Recommendation (Safety-First Logic)
- Toma MAYOR urgencia de todas las etapas
- Muestra razonamiento completo (2 o 3 stages)
- Transparencia total para health worker
    ↓
    
Health Worker Queue
- Urgencia final (Baja/Media/Alta)
- Condiciones sugeridas
- Casos históricos similares
- Nivel recomendado (Auxiliar/Enfermera/Médico/Especialista)
- Razonamiento de todas las etapas visible
```

### ⚡ Rendimiento Esperado

- **Casos simples (~70-80%)**: 2 stages únicamente → 3-5 segundos
- **Casos complejos (~20-30%)**: 3 stages con review → 8-12 segundos
- **Ahorro de costes**: ~60% menos llamadas a VLM vs sistema de 3 stages siempre

### 🔒 Principio Safety-First

El sistema SIEMPRE prioriza la seguridad del paciente:
- En caso de duda → Escalada automática
- Urgencia = MAX(Stage1, Stage2, Stage3)
- Cualquier "red flag" → Routing a médico

---

## 📦 Repositorios del Proyecto

| Repositorio | Descripción | Tecnología | Estado |
|------------|-------------|------------|--------|
| [mediguk-docs](https://github.com/MediGuk/mediguk-docs) | Documentación y arquitectura | Markdown | ✅ Activo |
| [mediguk-backend](https://github.com/MediGuk/mediguk-backend) | API Gateway y orquestación | Spring Boot | 🚧 En desarrollo |
| [mediguk-frontend](https://github.com/MediGuk/mediguk-frontend) | App web pacientes | Next.js | 🚧 En desarrollo |

---

## 🛠 Tech Stack

### Backend
- **Java Spring Boot** - API Gateway
  - Coordinación de microservicios
  - Autenticación y autorización
  - Integración con sistemas hospitalarios
  
### Frontend
- **Next.js** - Aplicación web
  - UI/UX accesible (mobile-first)
  - PWA para funcionalidad offline
  - Responsive design

### Microservicios
- **Python** - ML/IA Service
  - Modelos de clasificación
  - Pattern matching
  - scikit-learn / TensorFlow
  
- **Go** - Image Processing Service
  - Alta concurrencia
  - Procesamiento eficiente
  - Optimización de rendimiento

### Datos
- **PostgreSQL** - Base de datos principal
- **S3-compatible storage** - Imágenes médicas
- **Redis** - Cache de respuestas IA

### Infraestructura
- **Docker** - Containerización
- **RabbitMQ/Kafka** - Message queue
- **Kubernetes** - Orquestación (futuro)

---

## ✨ Características Principales

### Para Pacientes
- 📸 Envío multimodal (foto + texto + audio)
- 💬 Preguntas de clarificación interactivas
- 📱 Accesible desde móvil
- 🔔 Notificaciones de estado

### Para Profesionales Sanitarios
- 📋 Cola priorizada automáticamente
- 📊 Información pre-análisis completa
- 🔍 Acceso a casos históricos similares
- 🤝 Sistema colaborativo (consulta entre colegas)
- 📈 Dashboard de analytics

### Sistema de IA
- 🧠 Vision-Language Model (VLM)
- 📚 Pattern matching con datos reales
- ✅ Sistema de consenso automático
- 🔒 Procesamiento local (GDPR compliant)
- 🎯 Safety-first logic

---

## 🗺️ Roadmap

### ✅ Fase 1: Diseño y Arquitectura (Completado)
- [x] Investigación del problema
- [x] Diseño de arquitectura del sistema
- [x] Definición de tech stack
- [x] Documentación inicial
- [x] Creación de organización GitHub

### 🚧 Fase 2: MVP Backend (En Progreso)
- [ ] Spring Boot API Gateway básico
- [ ] Sistema de autenticación
- [ ] Base de datos PostgreSQL
- [ ] Message queue (RabbitMQ)
- [ ] Endpoints básicos

### 📅 Fase 3: Servicios de IA (Planificado)
- [ ] Python ML service con modelo básico
- [ ] Integración con VLM (API o modelo local)
- [ ] Sistema de pattern matching
- [ ] Lógica de consenso (3-stage)

### 📅 Fase 4: Frontend Paciente (Planificado)
- [ ] Next.js app básica
- [ ] Upload de fotos
- [ ] Formulario de síntomas
- [ ] Sistema interactivo de preguntas

### 📅 Fase 5: Frontend Health Worker (Planificado)
- [ ] Dashboard de cola de pacientes
- [ ] Visualización de análisis IA
- [ ] Sistema de escalado
- [ ] Analytics básico

### 📅 Fase 6: Testing (Futuro)
- [ ] Unit tests
- [ ] Integration tests
- [ ] Validación con datos sintéticos
- [ ] Pruebas de carga

---

## 📚 Documentación Adicional

_(Se añadirá según avance el proyecto)_

- **Diagramas** - Diagramas técnicos y de flujo
- **API Specs** - Especificaciones OpenAPI
- **Guías de Desarrollo** - Setup y contribución
- **Deployment** - Instrucciones de despliegue

---

## 💡 Motivación Personal

Este proyecto nace de mi experiencia en el sistema de salud. Tras 3 años estudiando medicina, entendí los desafíos que enfrentan pacientes y profesionales. Al cambiar a desarrollo de aplicaciones web (DAW), vi la oportunidad de aplicar conocimientos técnicos a problemas reales en salud.

**Objetivos:**
1. **Educativo**: Aprender arquitectura de microservicios, ML, y sistemas críticos
2. **Portfolio**: Demostrar capacidad de diseñar sistemas complejos
3. **Impacto social**: Explorar cómo la tecnología puede mejorar la atención sanitaria
4. **Largo plazo**: Sentar bases para desarrollo real (con validación clínica)

---

## ⚖️ Consideraciones Legales y Éticas

### Estado Actual
🟡 **Proof of Concept** - Proyecto educativo en desarrollo

**NO está certificado para uso médico real** y requeriría:
- Validación clínica exhaustiva
- Aprobación regulatoria (AEMPS en España, CE marking en UE)
- Años de testing con supervisión médica
- Certificación como dispositivo médico de software

### Privacidad y Seguridad
- Cumplimiento GDPR/LOPD
- Procesamiento local de datos (no cloud)
- Encriptación end-to-end
- Derecho al olvido
- Logs de auditoría completos

### Responsabilidad
- Sistema **asistivo**, no **diagnóstico**
- Decisión final siempre con profesional sanitario
- Transparencia en razonamiento IA
- Monitoreo de sesgos y equidad

---

## 📄 Licencia

Este proyecto está bajo licencia MIT. Ver [LICENSE](LICENSE) para detalles.

**Nota**: Aunque el código es open source, cualquier uso en contexto médico real requiere validación clínica y cumplimiento regulatorio apropiado.

---

## 👤 Contacto

**[Aitor Mendiburu]**
- Profesional de DAW con background en medicina
- 💼 LinkedIn: [linkedin.com/in/aitor-mendi]
- 🐙 GitHub: [@mendibot](https://github.com/mendibot)
- 📧 Email: mendibot15@gmail.com

---

## 🙏 Agradecimientos

- Profesionales sanitarios de Osakidetza que inspiraron este proyecto
- Comunidad open-source de health-tech
- Profesores y mentores de DAW

---

<p align="center">
  <i>Construido con ❤️ para mejorar la atención sanitaria</i>
</p>

<p align="center">
  <i>⚠️ Proyecto educativo - No apto para uso médico real sin validación apropiada</i>
</p>