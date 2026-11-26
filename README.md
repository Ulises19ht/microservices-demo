# Online Boutique: Implementación de Arquitectura DevOps y Pipeline CI/CD

## ¿Por qué este esfuerzo?

La principal razón de la implementación de esta solución fue resolver los problemas que limitaban directamente el crecimiento de Online Boutique. En su fase inicial, la plataforma contaba con una estructura y forma de trabajo que presentaban riesgos operativos importantes, los cuales retrasaban la entrega de nuevas funcionalidades. Asimismo, ante un eventual aumento en el número de usuarios, los riesgos de seguridad se volverían inevitables.

De no atenderse, estos problemas escalarían hacia mayores costos operativos, tiempos de inactividad y una experiencia de compra poco competitiva. La solución consistió en adoptar una arquitectura DevOps que automatiza los procesos de entrega, fortalece la seguridad mediante el manejo controlado de credenciales y permite que la plataforma evolucione de manera continua.

---

## Estructura de los Microservicios y del Repositorio

La estructura del proyecto está diseñada para organizar todos los componentes necesarios para desarrollar, automatizar y desplegar la aplicación. Cada directorio dentro del repositorio cumple un papel fundamental para el funcionamiento correcto de la plataforma:

- **`.github/workflows/`**: Contiene los archivos en formato YAML de los pipelines de GitHub Actions. Estos definen la automatización necesaria para compilar, validar y liberar nuevas versiones de forma rápida, segura y repetible mediante un *runner* auto-hospedado.
- **`src/`**: Representa el núcleo de la aplicación, ya que almacena el código fuente independiente de los 11 microservicios que conforman Online Boutique (`frontend`, `cartservice`, `checkoutservice`, `paymentservice`, entre otros). Esta separación facilita el mantenimiento: cuando un servicio requiere actualizarse, no es necesario modificar ni recompilar toda la aplicación.
- **`helm-chart/`**: Permite administrar toda la plataforma como una sola solución unificada, aunque internamente esté compuesta por múltiples servicios independientes. Contiene las plantillas y configuraciones de Helm encargadas de definir cómo desplegar y actualizar la aplicación en el clúster de Kubernetes.

---

## Funcionamiento del Pipeline CI/CD

El pipeline es el flujo automatizado que toma el código desarrollado y lo convierte en una nueva versión funcional dentro del entorno productivo. Este flujo se dispara automáticamente ante cada cambio integrado en las ramas principales (`main` o `master`), permitiendo que cualquier mejora comience a procesarse de inmediato sin necesidad de intervención operativa manual.

### Estrategia Matrix (Matriz de Ejecución)
Dado que la aplicación está compuesta por 11 microservicios, crear y mantener un pipeline individual para cada uno sería ineficiente. Para optimizar este proceso, se implementó una estrategia Matrix en GitHub Actions, permitiendo que un único flujo de trabajo administre, compile y despliegue de manera dinámica todos los microservicios de la plataforma.

### Etapas del Flujo Automatizado
1. **Obtención del código:** El *runner* auto-hospedado en WSL descarga la última versión del repositorio.
2. **Inyección segura de secretos:** Mediante HashiCorp Vault, el pipeline recupera de forma dinámica las credenciales de acceso (`username` y `token`), evitando exponer contraseñas o llaves en el código fuente.
3. **Construcción y publicación de imágenes (CI):** Se compila la imagen Docker de cada microservicio definido en la matriz y se publica en Docker Hub bajo las etiquetas `latest` y el identificador único del *commit* (`github.sha`).
4. **Despliegue continuo en Kubernetes (CD):** Se construyen las dependencias del paquete Helm y se ejecuta `helm upgrade --install --force` para actualizar los servicios en el clúster sin interrumpir la operación.
