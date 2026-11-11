# feversani-labs ☁️⚙️
### Laboratorio Modular de Arquitectura Cloud, Infrastructure as Code (IaC) & DevOps Engineering

![AWS](https://img.shields.io/badge/AWS-232F3E?style=for-the-badge&logo=amazonwebservices&logoColor=white)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=for-the-badge&logo=githubactions&logoColor=white)

Bienvenido a **`feversani-labs`**, un ecosistema técnico de arquitecturas de referencia, módulos reutilizables y laboratorios prácticos en la nube mantenidos por [**Diego Feversani**](https://github.com/fvrsn) (*Cloud & DevOps Engineer | AWS Solutions Architect Professional*).

Cada repositorio de esta organización está diseñado como un bloque de construcción independiente y reproducible mediante **Infrastructure as Code (Terraform)**, aplicando estándares reales de producción en **SecOps**, **High Availability** y **FinOps**.

---

## Catálogo de Arquitecturas por Patrón

Los repositorios se organizan según el patrón de ingeniería que resuelven. Esta estructura modular permite desplegar componentes aislados para pruebas puntuales o encadenarlos como base para plataformas más complejas.

### 1. Base Infrastructure & Governance
Cimientos para la gestión de estado remoto, parámetros centralizados y gobierno de sesiones efímeras.

| Repositorio | Stack & Servicios | Objetivo y Diseño Técnico |
| :--- | :--- | :--- |
| [**`ephemeral-aws-sandbox`**](https://github.com/feversani-labs/ephemeral-aws-sandbox) | `Terraform` `AWS S3` `SSM Parameter Store` | **Fundación del entorno de laboratorio:** Aprovisiona el *remote backend* nativo en S3 y parámetros en *SSM Parameter Store* optimizados para sesiones de laboratorio efímeras (**2h TTL**), garantizando trazabilidad y limpieza determinística. |

### 2. Compute Hardening & SecOps
Patrones de cómputo seguro, eliminación de superficie de ataque en red y automatización de arranque.

| Repositorio | Stack & Servicios | Objetivo y Diseño Técnico |
| :--- | :--- | :--- |
| [**`my-ec2-aws`**](https://github.com/feversani-labs/my-ec2-aws) | `Terraform` `AWS EC2` `SSM Session Manager` `Docker` `IAM` | **Compute Hardening (SSH-less):** Despliegue de instancia EC2 endurecida sin llaves SSH ni puerto `22` expuesto. Gestión auditada vía *AWS SSM Session Manager*, aprovisionamiento de **Docker** mediante `user_data` y políticas de **IAM Least Privilege**. |

### 3. High Availability & Networking
Diseños resilientes Multi-AZ, balanceo de carga, autoescalado y aislamiento en subredes privadas.

| Repositorio | Stack & Servicios | Objetivo y Diseño Técnico |
| :--- | :--- | :--- |
| [**`aws-high-availability-asg`**](https://github.com/feversani-labs/aws-high-availability-asg) | `Terraform` `AWS VPC` `ALB` `Auto Scaling` `Security Groups` | **Arquitectura Enterprise Multi-AZ:** Topología de alta disponibilidad con *Application Load Balancer (ALB)* en subredes públicas y *Auto Scaling Group (ASG)* distribuido en subredes privadas a través de múltiples Zonas de Disponibilidad. |

<!-- =========================================================================
GUÍA RÁPIDA PARA AGREGAR NUEVOS REPOSITORIOS AL CATÁLOGO:
1. Si pertenece a una categoría existente (1, 2 o 3), agrega una fila a su tabla:
| [**`nuevo-repo`**](https://github.com/feversani-labs/nuevo-repo) | `Terraform` `AWS ...` | **Título corto:** Descripción técnica del patrón implementado. |

2. Si inaugura un nuevo dominio (ej. Containers/EKS, CI/CD & GitOps, Platform Engineering),
   descomenta y utiliza el siguiente bloque:

### 4. Containers, Kubernetes (EKS) & GitOps
Orquestación de contenedores, clústeres de grado productivo y entrega continua declarativa.

| Repositorio | Stack & Servicios | Objetivo y Diseño Técnico |
| :--- | :--- | :--- |
| [**`nombre-del-repo`**](https://github.com/feversani-labs/nombre-del-repo) | `Terraform` `Amazon EKS` `ArgoCD` | **Patrón:** Descripción técnica de la arquitectura y decisiones de diseño. |
========================================================================= -->

---

## Principios de Diseño y Decisiones de Arquitectura (ADRs)

Todas las implementaciones dentro de **`feversani-labs`** siguen cuatro principios innegociables de ingeniería Cloud:

### 1. Entornos Efímeros y FinOps (Objetivo Costo `~$0 USD` en Labs)
La mejor arquitectura de laboratorio es aquella que valida escenarios complejos de producción sin generar costos ociosos (*idle costs*).
- **Ciclo de vida efímero (2h TTL):** Los laboratorios están pensados para levantarse en minutos (`terraform apply`), ejecutar pruebas de validación/carga y destruirse limpiamente (`terraform destroy`) dentro de ventanas de tiempo acotadas.
- **Right-Sizing & Free-Tier Friendly:** Selección consciente de tipos de instancia, políticas de retención de logs y desacoplamiento de recursos con cobro por hora para mantener una factura cercana a **`$0 USD`** fuera de las ventanas activas de prueba.

### 2. Security by Default (Zero-Trust & SSH-less)
- **Cero exposición innecesaria:** Ninguna instancia expone el puerto `22` (SSH) a Internet. Todo acceso operativo se realiza a través de **AWS Systems Manager (SSM) Session Manager** utilizando túneles cifrados y autenticados por IAM.
- **IAM Least Privilege:** Cada carga de trabajo posee un *Instance Profile* o *IAM Role* dedicado únicamente con los permisos estrictamente necesarios para operar.
- **Segmentación de Red:** Separación estrecha entre capa de ingreso pública (*ALB*) y capa de cómputo en subredes privadas, encadenando *Security Groups* por referencia en lugar de rangos CIDR abiertos.

### 3. Reproducibilidad e Idempotencia con IaC
- **100% Declarativo:** Ningún cambio se realiza manualmente por consola (*ClickOps Free*). Toda configuración de red, cómputo, permisos y *bootstrapping* (`user_data`) vive en código versionado de **Terraform**.
- **Desacoplamiento mediante SSM Parameter Store & S3:** El uso de `ephemeral-aws-sandbox` permite compartir metadatos y mantener el estado de Terraform seguro, bloqueado y centralizado sin *hardcodear* identificadores entre capas.

### 4. Resiliencia y Alta Disponibilidad (Multi-AZ)
- **Tolerancia a fallos por diseño:** Las topologías de red y cómputo distribuyen las cargas en múltiples Zonas de Disponibilidad (*Multi-AZ*), evitando puntos únicos de fallo (*SPOF*).
- **Autorecuperación (*Self-Healing*):** Uso de *Health Checks* en balanceadores de carga (*ALB*) integrados con *Auto Scaling Groups* para reemplazar instancias degradadas de forma automática y transparente.

---

## Flujo de Trabajo Recomendado (Quickstart)

Para desplegar cualquiera de los laboratorios siguiendo el orden de dependencias de la organización:

```bash
# 1. Inicializar la capa base (Backend S3 + Parámetros SSM para la sesión efímera)
git clone https://github.com/feversani-labs/ephemeral-aws-sandbox.git
cd ephemeral-aws-sandbox && terraform init && terraform apply

# 2. Desplegar el laboratorio objetivo (ej. Compute Hardening o High Availability)
git clone https://github.com/feversani-labs/aws-high-availability-asg.git
cd aws-high-availability-asg && terraform init && terraform apply

# 3. Destrucción limpia al finalizar la ventana de laboratorio (FinOps discipline)
terraform destroy -auto-approve
```

---

## Autor y Mantenimiento

Diseñado y mantenido por **Diego Gabriel Feversani**  
*Cloud & DevOps Engineer — AWS Certified Solutions Architect Professional*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Conectar-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/diegofeversani)
[![Email](https://img.shields.io/badge/Email-diego.feversani%40gmail.com-EA4335?style=flat-square&logo=gmail&logoColor=white)](mailto:diego.feversani@gmail.com)
