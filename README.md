# Laboratorio Técnico: Pipeline CI/CD con GitHub Actions y Jenkins

## Descripción del Proyecto

Este repositorio contiene la configuración completa de dos pipelines CI/CD para una aplicación web Python (Flask), alineados con los principios DevOps de automatización, colaboración y entrega continua.

- **CI (Integración Continua)**: Implementado con **GitHub Actions**, se ejecuta automáticamente ante cada push o pull request.
- **CD (Entrega Continua)**: Implementado con **Jenkins**, define los stages para construir, publicar y desplegar la aplicación en un cluster Kubernetes.

---

## Arquitectura del Pipeline CI/CD

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        FLUJO CI/CD COMPLETO                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  DESARROLLADOR                                                               │
│       │                                                                      │
│       ▼                                                                      │
│  ┌─────────┐    push/PR     ┌──────────────────────────────────────┐        │
│  │  GitHub  │──────────────▶│      GITHUB ACTIONS (CI)              │        │
│  │   Repo   │               │  ┌──────────────────────────────┐    │        │
│  └─────────┘               │  │ 1. Checkout código            │    │        │
│       │                     │  │ 2. Instalar dependencias      │    │        │
│       │                     │  │ 3. Análisis estático (Flake8) │    │        │
│       │                     │  │ 4. Ejecutar tests (Pytest)    │    │        │
│       │                     │  └──────────────────────────────┘    │        │
│       │                     └──────────────────────────────────────┘        │
│       │                                                                      │
│       ▼                                                                      │
│  ┌──────────────────────────────────────────┐                               │
│  │           JENKINS (CD)                    │                               │
│  │  ┌────────────────────────────────────┐  │                               │
│  │  │ 1. Checkout repositorio            │  │                               │
│  │  │ 2. Ejecutar tests pre-deploy       │  │                               │
│  │  │ 3. Build imagen Docker             │  │                               │
│  │  │ 4. Push a DockerHub                │  │                               │
│  │  │ 5. Deploy en Kubernetes            │  │                               │
│  │  └────────────────────────────────────┘  │                               │
│  └──────────────────────────────────────────┘                               │
│       │                                                                      │
│       ▼                                                                      │
│  ┌──────────────────┐                                                        │
│  │  Kubernetes (K8s) │ ◀── Deployment + Service (LoadBalancer)               │
│  │  - 2 réplicas     │                                                       │
│  │  - Health checks  │                                                       │
│  └──────────────────┘                                                        │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Estructura del Repositorio

```
labTec/
├── .github/
│   └── workflows/
│       └── ci.yml              # Pipeline CI con GitHub Actions
├── src/
│   └── app.py                  # Aplicación web Flask
├── tests/
│   └── test_app.py             # Pruebas unitarias
├── k8s/
│   ├── deployment.yaml         # Manifiesto de despliegue K8s
│   └── service.yaml            # Manifiesto de servicio K8s
├── docs/
│   └── documentacion_tecnica.md # Documentación técnica detallada
├── Dockerfile                  # Definición de imagen Docker
├── Jenkinsfile                 # Pipeline CD con Jenkins
├── requirements.txt            # Dependencias Python
└── README.md                   # Este archivo
```

---

## Tecnologías y Herramientas Seleccionadas

| Herramienta | Fase | Justificación |
|---|---|---|
| **GitHub Actions** | CI | Integración nativa con GitHub, ejecución automática en eventos, sin infraestructura adicional, YAML declarativo |
| **Jenkins** | CD | Orquestador robusto, soporte para pipelines complejos, extensible con plugins, estándar de la industria |
| **Python/Flask** | Aplicación | Framework ligero, rápido de prototipar, ideal para microservicios |
| **Pytest** | Testing | Framework de testing robusto, fixtures avanzadas, reportes detallados |
| **Flake8** | Calidad | Análisis estático que asegura adherencia a PEP8 y detecta errores potenciales |
| **Docker** | Containerización | Portabilidad, consistencia entre entornos, estándar de la industria para contenedores |
| **Kubernetes** | Orquestación | Escalabilidad, auto-healing, rolling updates, entorno agnóstico de infraestructura |
| **DockerHub** | Registro | Registro público/privado accesible, integración directa con Docker y K8s |

---

## Capturas de Pantalla

### Pipeline CI ejecutado exitosamente en GitHub Actions
![CI Pipeline](https://drive.google.com/uc?export=view&id=1wb0fv-iMr7xaIXgDzX2NdTIDu0H4J_cA)

### Jenkinsfile en el repositorio con stages definidos
![Jenkinsfile](https://drive.google.com/uc?export=view&id=1sM2_KQxAw_-6wGxJPs5e8POFuCjiixqu)

### Aplicación web corriendo localmente
![App Running](https://drive.google.com/uc?export=view&id=1L6DtsEZWanUWA74ftS5gzNoUJS37e6Om)

---

## Pipeline CI - GitHub Actions (Detalle)

**Archivo:** `.github/workflows/ci.yml`

**Trigger:** Se ejecuta automáticamente en cada `push` o `pull_request` a la rama `main`.

**Stages:**
1. **Checkout** - Clona el código fuente del repositorio
2. **Setup Python** - Configura el entorno con Python 3.11 y caché de pip
3. **Instalar dependencias** - Instala las dependencias del proyecto
4. **Análisis estático** - Ejecuta Flake8 para validar calidad de código
5. **Pruebas unitarias** - Ejecuta Pytest con reporte detallado

---

## Pipeline CD - Jenkins (Detalle)

**Archivo:** `Jenkinsfile`

**Stages:**
1. **Checkout** - Clona el repositorio desde GitHub
2. **Test** - Ejecuta pruebas como validación pre-despliegue
3. **Build Docker Image** - Construye la imagen con tag del build number
4. **Push to Registry** - Publica la imagen en DockerHub
5. **Deploy to Kubernetes** - Aplica manifiestos y verifica el rollout

---

## Cómo Ejecutar Localmente

```bash
# Instalar dependencias
pip install -r requirements.txt

# Ejecutar la aplicación
python src/app.py

# Ejecutar tests
pytest tests/ -v

# Ejecutar análisis estático
flake8 src/ --max-line-length=120

# Construir imagen Docker
docker build -t labtech-app .

# Ejecutar contenedor
docker run -p 5000:5000 labtech-app
```

---

## Relación con Principios DevOps

- **Automatización**: Ambos pipelines eliminan intervención manual en build, test y deploy.
- **Feedback rápido**: CI ejecuta en cada commit, detectando errores de forma temprana.
- **Entrega continua**: CD permite desplegar de forma confiable y repetible.
- **Infraestructura como código**: Kubernetes manifiestos y Dockerfiles versionados.
- **Colaboración**: GitHub como punto central, PRs validados automáticamente.

---

## Autor

Jagler David Velásquez Velásquez  
Universidad de La Sabana
