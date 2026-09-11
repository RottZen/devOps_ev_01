# Evaluación Parcial 1: Tu primer pipeline de despliegue
**Asignatura:** Ingeniería DevOps (DOY0101)  
**Institución:** Duoc UC  

### Integrantes del grupo:
* **Integrante 1:** Ignacio Olmos
* **Integrante 2:** Diego Gajardo
* **Profesor:** Guillermo Villacura

---

## 1. De qué trata el proyecto

Para esta evaluación usamos como base un proyecto de biblioteca que teníamos en Spring Boot con Java 21. El proyecto está dividido en varios microservicios:

* **api-gateway:** Es la puerta de entrada para todas las peticiones.
* **eureka:** Es el servidor para registrar los microservicios y que se puedan encontrar entre ellos.
* **ms-usuarios:** Microservicio para crear y manejar los usuarios del sistema.
* **ms-catalogo:** Microservicio que maneja los libros y los préstamos.
* **ms-recursos:** Microservicio para administrar recursos digitales de la biblioteca.
* **common:** Es una carpeta con clases y DTOs que comparten los microservicios para no repetir código.

---

## 2. Modelos de Ramas (IE1)

Investigamos los diferentes modelos de trabajo con ramas en Git para ver cuál nos servía más para trabajar en equipo:

### Comparación de los modelos:

1. **GitFlow:**  
   * **Cómo funciona:** Usa dos ramas fijas: `main` (solo para código listo y probado) y `develop` (donde se junta todo lo que vamos programando). Cuando alguien hace algo nuevo, saca una rama `feature/` desde develop. Si hay una emergencia en producción, se saca un `hotfix/` directo desde main.
   * **Ventajas:** Muy ordenado, es difícil romper lo que ya funciona porque nadie toca main directo.
   * **Desventajas:** Hay que hacer hartos merges y coordinarse bien para no dejar ramas botadas, pero vale completamente la pena.
   * **Uso en la nube:** Sirve mucho cuando somos solo estudiantes trabajando, por que podemos revisar bien antes de subir a un servidor.

2. **GitHub Flow:**  
   * **Cómo funciona:** Es bastante más simple. Solo hay una rama principal (`main`). Uno saca una rama para su cambio, la prueba, abre un Pull Request y al tiro se mezcla a main y se va a producción.
   * **Ventajas:** Es súper rápido, no hay que estar pasando cambios de una rama a otra.
   * **Desventajas:** Si metes un bug a main, se va directo a producción porque no hay una rama develop para probar tranquilos.
   * **Uso en la nube:** Muy usado en páginas web o apps que se actualizan todos los días.

3. **Trunk-Based Development:**  
   * **Cómo funciona:** Todos los programadores trabajan directamente sobre la rama principal o en ramas muy cortas que duran menos de un día.
   * **Ventajas:** Nadie se queda desactualizado y se evitan los conflictos gigantes de código.
   * **Desventajas:** Requiere que el equipo tenga mucha experiencia y que existan miles de pruebas automáticas para no romper todo.
   * **Uso en la nube:** Es el modelo que usan profecinales como Google o Netflix.

### ¿Por qué elegimos GitFlow para este trabajo?
Elegimos **GitFlow** porque somos 2 integrantes y recién estamos aprendiendo DevOps. Nos pareció la forma más segura de trabajar, ya que:
* Mantenemos `main` limpia con la versión que funciona.
* Usamos `develop` para ir juntando las partes que cada uno hacía sin miedo a romper la entrega final.
* Nos obligó a crear ramas separadas (`feature/` y `hotfix/`), lo que nos sirvió para simular cómo se arreglan errores urgentes en una empresa real sin frenar el desarrollo.

---

## 3. Buenas prácticas que definimos para el equipo (IE5)

Para trabajar ordenados y que el profe entienda qué hicimos en cada paso, acordamos estas reglas:

### Nombres de las ramas:
* `main`: Código final que funciona. Prohibido hacer push directo acá.
* `develop`: Rama común donde juntamos el trabajo.
* `feature/auditoria-catalogo`: Para cosas nuevas (o tambien `feature/notificaciones-usuarios`).
* `hotfix/corregir-puerto-gateway`: Para arreglar fallas graves urgentes.

### Nombres de los commits (Conventional Commits):
Escribimos los commits en minúsculas indicando qué se hizo:
* `feat:` si agregamos algo nuevo al código.
* `fix:` si corregimos un error o bug.
* `ci:` si tocamos cosas del pipeline o GitHub Actions.


### Limpieza de carpetas:
Configuramos el archivo `.gitignore` para no subir cosas pesadas ni basura al repositorio:
* No subimos la carpeta `target/` (que son los ejecutables que compila Maven).
* No subimos las carpetas de nuestros editores (`.idea/` de IntelliJ ni `.vscode/`).
* No subimos archivos de base de datos local ni logs.

### Reglas para revisar el código:
* Ninguno puede subir código directo a `main` ni a `develop`.
* Todo se sube mediante un **Pull Request (PR)**.
* El otro compañero tiene que entrar a GitHub, revisar los cambios y aprobar el PR con un comentario antes de poder hacer el merge.
* Si el pipeline de GitHub Actions falla (cruz roja), no se puede hacer el merge hasta arreglar el error.

---

## 4. Pipeline y Automatización con GitHub Actions (IE3 y IE4)

Para cumplir con la parte de CI (Integración Continua) y el "entorno cloud simulado", creamos el archivo `.github/workflows/ci.yml`.

### ¿Qué hace el pipeline?
Cada vez que alguien hace un `push` a la rama `develop` o abre un `pull request` hacia `main`, GitHub levanta de forma automática una máquina virtual en su nube (con Ubuntu) y hace lo siguiente:
1. **Descarga el código:** Clona nuestro repositorio en la máquina de GitHub.
2. **Instala Java 21:** Configura el JDK 21 de Eclipse Temurin para que coincida con la versión que usamos en el proyecto.
3. **Compila con Maven:** Ejecuta el comando `mvn clean compile -DskipTests` para verificar que todo el código esté bien escrito y que ningún microservicio tenga errores de compilación.

### ¿Por qué esto es importante en DevOps?
Porque en vez de decir *"en mi computador sí funciona"*, el pipeline prueba el código en un servidor neutro en la nube. Si alguien sube código malo o borra algo sin querer, el pipeline se pone rojo de inmediato y nos avisa antes de que el error llegue a producción.

---

## 5. Registro de cambios y trabajo colaborativo (IE2)

Para la simulación colaborativa hicimos 2 ramas de feature y 1 de hotfix usando Pull Requests:

| Tipo | Rama que creamos | Rama de destino | Quién lo programó | Quién lo revisó y aprobó | Resultado | Qué se hizo |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Feature 1** | `feature/auditoria-catalogo` | `develop` | Diego Gajardo | Ignacio Olmos | Aprobado y Mergeado | Agregamos parámetros de auditoría en la configuración de `ms-catalogo`. |
| **Feature 2** | `feature/notificaciones-usuarios` | `develop` | Ignacio Olmos | Diego Gajardo | Aprobado y Mergeado | Habilitamos el módulo de notificaciones en `ms-usuarios`. |
| **Hotfix 1** | `hotfix/corregir-puerto-gateway` | `main` | Ignacio Olmos | Diego Gajardo | Aprobado y Mergeado | Arreglamos los tiempos de timeout en `api-gateway` para evitar caídas en producción. |

---

## 6. Declaración de uso de Inteligencia Artificial

Usamos herramientas de Inteligencia Artificial para:
* Consultar dudas sobre sintaxis de comandos Git y cómo armar el archivo YAML de GitHub Actions.
* Pedir ideas para ordenar las secciones del informe.



---

## 7. Conclusiones y reflexiones personales

### Reflexión de Ignacio Olmos:
> Yo creo que todo este sistema o workflow fue bastante interesante y ágil de usar, si bien hay cosas que aún no logro comprender del todo, siento que si practico lo suficiente lograré sacarle provecho a como funciona todo esto de Git.

### Reflexión de Diego Gajardo:
> Para mi lo mas valioso fue aprender lo que es un PR y como abordarlo.
Esta evaluación me ayudó a entender para qué sirve realmente Git en un trabajo en equipo. Antes solo hacía commit y push directo a main, pero con GitFlow y los Pull Requests entendí lo importante que es revisar el código de mi compañero antes de mezclarlo. 