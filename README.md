# 🚲 Workshop: GitHub Copilot para Contoso Biker

## Desarrollo asistido por IA con C# y ASP.NET Core

![GitHub Copilot](https://img.shields.io/badge/GitHub%20Copilot-Habilitado-brightgreen)
![.NET](https://img.shields.io/badge/.NET-10%20LTS-512BD4)
![C#](https://img.shields.io/badge/C%23-14-239120)
![OpenAPI](https://img.shields.io/badge/OpenAPI-3.1-6BA539)
![Pruebas](https://img.shields.io/badge/Pruebas-xUnit-orange)
![Duración](https://img.shields.io/badge/Duración-3%20horas-red)
![Idioma](https://img.shields.io/badge/Idioma-Español-yellow)

---

## 📋 Tabla de contenidos

- [🎯 Introducción](#-introducción)
- [🧠 Conceptos clave de GitHub Copilot](#-conceptos-clave-de-github-copilot)
- [🛠️ Pre-requisitos](#️-pre-requisitos)
- [📅 Agenda del workshop](#-agenda-del-workshop)
- [🔬 Ejercicio 1: API REST con Copilot](#-ejercicio-1-api-rest-con-copilot-45-min)
- [🔬 Ejercicio 2: Frontend e integración](#-ejercicio-2-frontend-e-integración-30-min)
- [🔬 Ejercicio 3: Pruebas y refactoring](#-ejercicio-3-pruebas-y-refactoring-25-min)
- [🤖 Ejercicio 4: Creación de agentes](#-ejercicio-4-creación-de-agentes-35-min)
- [📖 Referencia rápida](#-referencia-rápida)
- [🆘 ¿Te quedaste atrás?](#-te-quedaste-atrás)
- [✅ Checklist final](#-checklist-final)
- [🙋 Preguntas frecuentes](#-preguntas-frecuentes)
- [📚 Recursos adicionales](#-recursos-adicionales)
- [👥 Créditos](#-créditos)

---

## 🎯 Introducción

Este workshop práctico de **3 horas** te lleva de cero a una aplicación completa para **Contoso Biker**, una red ficticia de renta de bicicletas. No vas a escribir el código "a mano": vas a **dirigir a GitHub Copilot** para que lo escriba por ti, y vas a aprender a juzgar cuándo acertó y cuándo no.

Al terminar sabrás:

- ✅ Aprovechar el autocompletado y las sugerencias en línea de Copilot
- ✅ Escribir prompts que comunican **intención**, no instrucciones mecánicas
- ✅ Construir una API REST en ASP.NET Core con documentación OpenAPI automática
- ✅ Generar un frontend que consume esa API
- ✅ Producir pruebas unitarias y de integración asistidas por IA
- ✅ Usar Copilot Chat para explicar, corregir y refactorizar código existente
- ✅ **Configurar y crear tus propios agentes personalizados** para que Copilot trabaje como tu equipo necesita

> 💡 La complejidad del negocio es **deliberadamente baja**. El objetivo no es entregar un sistema de producción, sino ver con claridad **cómo Copilot acelera cada fase del desarrollo**.

### Estándares del proyecto

| Aspecto | Estándar |
|---------|----------|
| Tipo de aplicación | API REST + página web de administración |
| Lenguaje | C# 14 |
| Plataforma | .NET 10 (LTS, soporte hasta noviembre de 2028) |
| Framework | ASP.NET Core **Minimal APIs** |
| Documentación de la API | OpenAPI 3.1 con `Microsoft.AspNetCore.OpenApi` + interfaz **Scalar** |
| Idioma | Español en código, comentarios y documentación |
| Persistencia | En memoria (`Dictionary<int, T>`), sin base de datos |
| Frontend | HTML + JavaScript vanilla + Bootstrap 5 por CDN, servido desde `wwwroot` |
| Pruebas | xUnit + `WebApplicationFactory<Program>` |

### Escenario: Contoso Biker 🚲

**Contoso Biker** renta bicicletas en varias ciudades y hoy administra todo en hojas de cálculo. Necesita un sistema digital que gestione:

- **Clientes** — los ciclistas registrados: nombre, email, teléfono, ciudad
- **Bicicletas** — el inventario: código, modelo, tipo, tarifa por día, estado
- **Rentas** *(bonus)* — los movimientos: inicio, devolución y extensión de una renta

Las tres entidades se relacionan así:

```
Cliente  1 ──────< N  Bicicleta        (una bicicleta rentada pertenece a un cliente)
Cliente  1 ──────< N  Renta
Bicicleta 1 ─────< N  Renta            (historial de movimientos de cada bicicleta)
```

> 📝 **Sobre las rentas:** los ejercicios guiados cubren **Clientes** y **Bicicletas**. Las **Rentas** son un desafío opcional al final del Ejercicio 1, pensado para quien avance rápido. Si no llegas, no pasa nada: el valor del workshop está en el proceso, no en completar todo el código.

### Arquitectura de la solución

```
┌──────────────────────────────────────────────────────────────────┐
│                          NAVEGADOR WEB                           │
│                                                                  │
│  ┌──────────────────────┐      ┌──────────────────────────────┐  │
│  │  Panel de Contoso    │      │   Referencia de la API       │  │
│  │  Biker               │      │   (Scalar, generada sola)    │  │
│  │                      │      │                              │  │
│  │  Bootstrap 5 (CDN)   │      │   GET  /scalar               │  │
│  │  JavaScript vanilla  │      │   GET  /openapi/v1.json      │  │
│  │  fetch() → /api/...  │      │                              │  │
│  │                      │      │                              │  │
│  │  GET /               │      │                              │  │
│  └──────────┬───────────┘      └──────────────┬───────────────┘  │
│             │                                 │                  │
└─────────────┼─────────────────────────────────┼──────────────────┘
              │      HTTP (mismo origen)        │
              ▼                                 ▼
┌──────────────────────────────────────────────────────────────────┐
│                  ContosoBiker.Api  (ASP.NET Core)                │
│                                                                  │
│  ┌────────────────────────┐  ┌────────────────────────────────┐  │
│  │  Archivos estáticos    │  │      Minimal APIs              │  │
│  │                        │  │                                │  │
│  │  UseDefaultFiles()     │  │  /api/clientes    → CRUD       │  │
│  │  UseStaticFiles()      │  │  /api/bicicletas  → CRUD       │  │
│  │                        │  │  /api/rentas      → CRUD  (*)  │  │
│  │  wwwroot/index.html    │  │  /api/estadisticas → resumen   │  │
│  └────────────────────────┘  └───────────────┬────────────────┘  │
│                                              │                   │
│                                              ▼                   │
│                        ┌──────────────────────────────────────┐  │
│                        │  Servicios (Singleton, en memoria)   │  │
│                        │                                      │  │
│                        │   Servicios/ClienteServicio.cs       │  │
│                        │   Servicios/BicicletaServicio.cs     │  │
│                        │   Servicios/RentaServicio.cs    (*)  │  │
│                        │                                      │  │
│                        │  Modelos (con DataAnnotations)       │  │
│                        │   Modelos/Cliente.cs                 │  │
│                        │   Modelos/Bicicleta.cs               │  │
│                        │   Modelos/Renta.cs              (*)  │  │
│                        └──────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                   ▲
                                   │ WebApplicationFactory<Program>
┌──────────────────────────────────┴───────────────────────────────┐
│                      ContosoBiker.Tests  (xUnit)                 │
│   ClienteServicioTests.cs   → pruebas unitarias                  │
│   BicicletaServicioTests.cs → pruebas unitarias                  │
│   ApiIntegracionTests.cs    → pruebas de integración HTTP        │
└──────────────────────────────────────────────────────────────────┘

(*) Rentas es el desafío opcional del Paso 1.8
```

**Cómo fluye una petición:**

1. El usuario abre `http://localhost:5080/`. `UseDefaultFiles()` reescribe `/` a `/index.html` y `UseStaticFiles()` entrega el archivo desde `wwwroot`.
2. El JavaScript de esa página llama a `fetch('/api/...')` — mismo origen, así que no hace falta configurar CORS.
3. ASP.NET Core enruta la petición al endpoint correspondiente.
4. Antes de ejecutar el endpoint, la **validación integrada de .NET 10** revisa el modelo recibido y responde `400` si algo no cumple las reglas.
5. El servicio correspondiente lee o modifica el diccionario en memoria y devuelve el resultado.
6. En paralelo, `/openapi/v1.json` describe toda la API y `/scalar` la muestra como documentación navegable donde puedes lanzar peticiones reales.

---

## 🧠 Conceptos clave de GitHub Copilot

### ¿Qué es GitHub Copilot?

GitHub Copilot es un **asistente de programación con IA** integrado en tu editor. A diferencia de un buscador, Copilot ve tu contexto: los archivos abiertos, la estructura del proyecto, los nombres que usas y los comentarios que escribes. Con eso:

- Completa líneas y bloques completos mientras escribes
- Responde preguntas sobre tu propio código
- Crea y modifica archivos cuando se lo pides
- Ejecuta comandos de terminal y corrige lo que falla

Piénsalo como un colega con muchísima lectura y cero contexto de tu negocio: **tú pones el contexto, él pone la velocidad.**

### El arte del prompting

La diferencia entre un resultado mediocre y uno excelente casi nunca está en el modelo: está en **cómo describes lo que quieres**.

**❌ Prompt débil:**

```
Hazme la app de bicicletas
```

> ¿Qué falta? Todo: el dominio, la tecnología, el alcance, el resultado esperado. Copilot tiene que **adivinar**, y cuando adivina, acierta poco.

**✅ Prompt efectivo:**

```csharp
// Endpoint para consultar la disponibilidad de una bicicleta de Contoso Biker.
// Recibe el id de la bicicleta en la ruta.
// Devuelve el estado actual (disponible/rentada/mantenimiento), la tarifa por día
// y, si está rentada, el nombre del cliente que la tiene.
// Responde 404 si la bicicleta no existe.
// Usa Minimal APIs con WithSummary() para que aparezca documentado en OpenAPI.
```

> Fíjate en las cuatro capas: **contexto de negocio** (Contoso Biker, bicicleta), **intención** (consultar disponibilidad), **contrato** (qué entra, qué sale, qué pasa si falla) y **tecnología** (Minimal APIs, OpenAPI).

**Una regla práctica:** si tu prompt no le diría a un desarrollador junior lo suficiente para empezar, tampoco se lo dice a Copilot.

| Ingrediente | Pregunta que responde | Ejemplo |
|-------------|----------------------|---------|
| Contexto | ¿De qué negocio hablamos? | "para el inventario de Contoso Biker" |
| Intención | ¿Qué problema resuelve? | "quiero filtrar las bicicletas libres" |
| Contrato | ¿Qué entra y qué sale? | "recibe `?estado=`, devuelve una lista JSON" |
| Restricciones | ¿Qué NO debe hacer? | "sin base de datos, todo en memoria" |
| Tecnología | ¿Con qué se implementa? | "Minimal APIs de .NET 10" |

> 📚 ¿Quieres más ejemplos de la comunidad? Revisa [github/awesome-copilot](https://github.com/github/awesome-copilot): instrucciones, agentes y configuraciones reutilizables.

### Cómo darle contexto: las referencias `#`

Copilot no lee tu proyecto entero en cada mensaje. Las referencias `#` le dicen **exactamente dónde mirar**.

| Referencia | Qué aporta | Ejemplo |
|------------|-----------|---------|
| `#codebase` | Busca en todo el espacio de trabajo | `#codebase ¿dónde se valida el código de la bicicleta?` |
| `#file:ruta` | Un archivo concreto | `#file:Modelos/Cliente.cs genera un modelo equivalente para Bicicleta` |
| `#selection` | Lo que tienes seleccionado en el editor | `#selection explica esta expresión LINQ` |
| `#fetch` | Contenido de una URL | `#fetch https://learn.microsoft.com/... resume esta página` |
| `#githubRepo` | Código de un repositorio público | `#githubRepo dotnet/aspnetcore busca ejemplos de MapGroup` |
| `#terminalLastCommand` | La última salida de la terminal | `#terminalLastCommand ¿por qué falló?` |

> ⚠️ **Si vienes de tutoriales viejos:** el participante `@workspace` ya no forma parte de la documentación actual de VS Code; su equivalente hoy es **`#codebase`**. Si ves `@workspace` en un material antiguo, tradúcelo mentalmente.

> 💡 En modo agente no necesitas nombrar cada herramienta: el agente decide sola cuáles usar. Las referencias `#` siguen siendo útiles cuando quieres **forzar** una fuente concreta.

### Los modos de Copilot Chat

> ⚠️ **Nota honesta:** la interfaz de Copilot cambia con frecuencia (nombres, íconos, ubicación del selector). Si lo que ves no coincide exactamente con lo descrito aquí, el **concepto** sigue siendo válido: consulta con la persona que imparte el taller o revisa la [documentación oficial](https://code.visualstudio.com/docs/copilot/overview).

#### 1️⃣ Modo Ask (preguntar) 💬

| Aspecto | Detalle |
|---------|---------|
| Qué hace | Responde preguntas. **No toca tus archivos.** |
| Cuándo usarlo | Explorar, entender, comparar opciones, aprender |
| Riesgo | Ninguno |

```
[Modo Ask]
"¿Cuál es la diferencia entre Minimal APIs y controladores en ASP.NET Core?"

→ Copilot EXPLICA. No crea nada.
```

#### 2️⃣ Modo Agent (agente) 🤖

| Aspecto | Detalle |
|---------|---------|
| Qué hace | Crea y modifica archivos, ejecuta comandos, itera si algo falla |
| Cuándo usarlo | Implementar, generar, refactorizar, correr pruebas |
| Riesgo | Medio — revisa siempre el diff antes de aceptar |

```
[Modo Agent]
"Crea el modelo Bicicleta para Contoso Biker con validaciones de DataAnnotations"

→ Copilot CREA el archivo con el código completo.
```

#### 3️⃣ Modo Plan (planificar) 📋

| Aspecto | Detalle |
|---------|---------|
| Qué hace | Investiga y propone un plan **antes** de tocar nada |
| Cuándo usarlo | Tareas grandes que cruzan varios archivos |
| Riesgo | Bajo — tú apruebas el plan |

```
[Modo Plan]
"Implementa la funcionalidad completa de rentas: modelo, servicio,
endpoints y pruebas"

→ Copilot PROPONE:
  1. Modelos/Renta.cs con validaciones
  2. Servicios/RentaServicio.cs con datos de ejemplo
  3. Endpoints/RentasEndpoints.cs registrado en Program.cs
  4. Pruebas de integración para el flujo inicio → devolución

→ Tú revisas y apruebas antes de que se ejecute.
```

#### Comparativa

| Característica | Ask 💬 | Agent 🤖 | Plan 📋 |
|----------------|--------|----------|---------|
| Modifica archivos | ❌ No | ✅ Sí | ✅ Sí, tras aprobación |
| Ejecuta comandos | ❌ No | ✅ Sí | ✅ Sí, tras aprobación |
| Velocidad | Alta | Alta | Menor |
| Control que tienes | N/A | Medio | Alto |
| Ideal para | Aprender | Implementar | Tareas complejas |

### Comandos especiales (`/`)

Escribe `/` en el chat para ver los disponibles. Los que usaremos:

| Comando | Para qué sirve | Cómo se usa |
|---------|----------------|-------------|
| `/explain` | Explica el código seleccionado | Selecciona código → `/explain` |
| `/fix` | Propone y aplica una corrección | Selecciona código con problema → `/fix` |
| `/tests` | Genera pruebas del código seleccionado | Selecciona código → `/tests` |
| `/doc` | Genera comentarios de documentación | Desde el chat en línea del editor |
| `/plan` | Investiga y propone un plan | `/plan implementar rentas` |
| `/new` | Crea la estructura de un proyecto | `/new API en .NET 10` |
| `/agents` | Abre la configuración de agentes | `/agents` |
| `/create-agent` | Genera un agente personalizado | `/create-agent` |

> 💡 La disponibilidad de cada comando depende de la superficie de chat y del modo activo. Si uno no aparece, no está roto: no aplica a ese contexto.

### Qué hace bien y qué hace mal

Saber esto te ahorra la mitad de la frustración:

| Copilot es excelente en… | Copilot falla en… |
|---------------------------|-------------------|
| Código repetitivo y CRUD | Reglas de negocio que no le contaste |
| Seguir un patrón que ya existe en tu repo | Decidir la arquitectura por ti |
| Generar pruebas a partir de código | Saber qué es "correcto" en tu dominio |
| Traducir entre lenguajes y frameworks | APIs muy nuevas o muy de nicho |
| Explicar código ajeno | Garantizar que no inventó una función |

> 🧭 **Regla de oro del taller:** Copilot propone, **tú dispones**. Nunca aceptes código que no puedas explicar.

---

## 🛠️ Pre-requisitos

### Software necesario

Verifica que tienes todo antes de empezar:

```powershell
dotnet --version   # debe mostrar 10.0.x
code --version     # Visual Studio Code
git --version      # Git
```

Si `dotnet --version` no muestra una versión 10.x, descarga el SDK desde [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download/dotnet/10.0).

> 📝 **Por qué .NET 10:** es la versión **LTS** vigente, con soporte hasta noviembre de 2028. Todo el material de este taller está probado contra el SDK `10.0.4xx`.

> 📝 **Sin base de datos:** el taller guarda todo en diccionarios en memoria. Los datos se pierden al reiniciar la aplicación, pero se recargan datos de ejemplo automáticamente. Así nadie pierde media hora instalando SQL Server.

### Extensiones de VS Code

| Extensión | Para qué |
|-----------|----------|
| **GitHub Copilot** | Sugerencias en línea |
| **GitHub Copilot Chat** | Chat, modos Ask/Agent/Plan y agentes |
| **C# Dev Kit** (Microsoft) | IntelliSense, depuración y explorador de pruebas de C# |

### Cuenta de GitHub

Necesitas una cuenta con **GitHub Copilot activo** (individual, de organización o educativa). Comprueba que funciona: abre VS Code, pulsa `Ctrl+Alt+I` y escribe cualquier pregunta. Si responde, estás listo.

### Conectividad

El taller usa CDN para Bootstrap y NuGet para los paquetes. Necesitas internet.

---

## 📅 Agenda del workshop

| Hora | Bloque | Actividad | Modo de Copilot |
|------|--------|-----------|-----------------|
| 0:00 – 0:15 | Bienvenida | Setup, verificación del entorno e introducción | — |
| 0:15 – 1:00 | **Ejercicio 1** | API REST con Minimal APIs + OpenAPI | Ask → Agent |
| 1:00 – 1:10 | ☕ Descanso | Pausa y preguntas rápidas | — |
| 1:10 – 1:40 | **Ejercicio 2** | Frontend en `wwwroot` e integración | Agent |
| 1:40 – 2:05 | **Ejercicio 3** | Pruebas unitarias y de integración, refactoring | Agent + `/tests` |
| 2:05 – 2:15 | ☕ Descanso | Pausa | — |
| 2:15 – 2:50 | **Ejercicio 4** | Personalización y creación de agentes | Agent + `/create-agent` |
| 2:50 – 3:00 | Cierre | Recapitulación, tips avanzados y recursos | — |

> ⏱️ Los tiempos son orientativos. Ajusta al ritmo del grupo, pero **no excedas las 3 horas**: la atención cae en picado.

> 🎓 **Para quien imparte:** si el setup inicial se alarga por problemas de instalación, recorta el Ejercicio 3 a los Pasos 3.1–3.4 (generar y ejecutar pruebas) y omite el refactoring. Lo imprescindible es que todo el mundo complete los Ejercicios 1, 2 y 4. **El Ejercicio 4 es el diferenciador del taller: no lo sacrifiques.**

---

## 🔬 Ejercicio 1: API REST con Copilot (45 min)

### Objetivos

- ✅ Configurar las instrucciones de Copilot para todo el proyecto
- ✅ Crear la estructura de la solución con el CLI de .NET
- ✅ Implementar una API REST con Minimal APIs
- ✅ Obtener documentación OpenAPI navegable sin escribirla
- ✅ Experimentar con autocompletado y con el modo agente

---

### Paso 1.1 · Explorar con el modo Ask 🔍

> 💡 **Importante:** asegúrate de estar en **modo Ask** 💬. Este modo no toca tus archivos: solo responde.

📍 **Cómo activarlo:**

1. Abre Copilot Chat con `Ctrl+Alt+I`
2. Busca el selector de modo en la parte superior del panel
3. Elige **Ask**

🤖 **PROMPT — copia y pega en Copilot Chat:**

```
Trabajo en Contoso Biker, una red de renta de bicicletas, y voy a diseñar su
sistema de gestión desde cero.

Ayúdame a entender, sin escribir código todavía:

1. Qué entidades necesito para gestionar:
   - Clientes (nombre, email, teléfono, ciudad)
   - Bicicletas (código de inventario, modelo, tipo, tarifa por día, estado)
   - Rentas (inicio, devolución y extensión de una renta)

2. Qué endpoints REST hacen falta para un CRUD básico de cada una, y qué
   código de estado HTTP debería devolver cada uno.

3. Cómo organizo esto con Minimal APIs de ASP.NET Core en .NET 10:
   qué carpetas, qué archivos y qué responsabilidad tiene cada uno.

4. Cómo se genera la documentación OpenAPI en .NET 10 y por qué ya no se usa
   Swashbuckle en las plantillas nuevas.
```

📝 **Observa:** Copilot responde con detalle pero **no crea ningún archivo**. Este es el modo para pensar antes de teclear.

> 🌟 **Momento wow:** fíjate en que entiende el dominio de renta de bicicletas y propone una arquitectura coherente sin que le hayas dado detalles técnicos. El punto 4 además revela un cambio real de la plataforma que mucha documentación vieja todavía no refleja.

---

### Paso 1.2 · Configurar las instrucciones del proyecto

> 💡 **¿Por qué ahora y no después?** El archivo `.github/copilot-instructions.md` define las reglas que Copilot aplicará a **todo** lo que genere de aquí en adelante. Si lo creas antes de escribir la primera línea, los modelos, la API, el frontend y las pruebas saldrán con las mismas convenciones. Si lo creas al final, ya no sirve de nada.

> 💡 **Cambia a modo Agent** 🤖. Este modo sí puede crear archivos.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/copilot-instructions.md con exactamente este contenido:

# Instrucciones de Copilot — Contoso Biker

## Contexto del proyecto
Contoso Biker es una red de renta de bicicletas. Este repositorio contiene una
API REST que gestiona clientes, bicicletas y rentas, más una página web de
administración. Los datos viven en memoria: no hay base de datos.

## Idioma
- Todo el código, los comentarios y la documentación se escriben en **español**.
- Los mensajes de error que ve el usuario van en español.
- Los nombres de clases, métodos y variables van en español, salvo los términos
  técnicos estándar del framework (Get, Post, Id, Api, Http, Endpoint).

## Plataforma
| Aspecto | Estándar |
|---------|----------|
| Lenguaje | C# 14 |
| Plataforma | .NET 10 (LTS) |
| Framework | ASP.NET Core Minimal APIs |
| Documentación | OpenAPI con Microsoft.AspNetCore.OpenApi + Scalar |
| Pruebas | xUnit |

## Nomenclatura
- Clases y métodos públicos: PascalCase en español (`ClienteServicio`,
  `ObtenerPorId`).
- Variables locales y parámetros: camelCase en español (`clienteId`,
  `bicicletaNueva`).
- Constantes: PascalCase (`TiposValidos`).
- Archivos de endpoints: `<Entidad>Endpoints.cs` con un método de extensión
  `Map<Entidad>Endpoints`.

## Reglas de implementación
- Usa Minimal APIs agrupadas con `MapGroup`, nunca controladores.
- Cada endpoint declara `WithName`, `WithSummary` y los `Produces` que
  correspondan, para que la documentación OpenAPI sea útil.
- Valida los modelos con DataAnnotations y deja que la validación integrada de
  .NET 10 devuelva el 400.
- Los servicios de datos se registran como Singleton y precargan datos de ejemplo.
- Nunca devuelvas 200 con un cuerpo vacío: usa 404 si no existe, 409 si hay
  conflicto y 204 al eliminar.
- Documenta cada clase y cada método público con comentarios XML (`///`) en español.
````

✅ **Verifica:** el archivo debe existir en `.github/copilot-instructions.md`. Ábrelo y léelo — es el "contrato" del resto del taller.

> 🌟 **Momento wow:** en los pasos siguientes, cuando Copilot genere código en español, con `MapGroup` y con `WithSummary` **sin que se lo pidas explícitamente**, estarás viendo estas instrucciones en acción.

---

### Paso 1.3 · Crear la estructura de la solución

🤖 **PROMPT en modo Agent:**

```
Crea la estructura de la solución de Contoso Biker con el CLI de .NET.

Necesito:
- Una solución llamada ContosoBiker
- Un proyecto de Web API llamado ContosoBiker.Api (Minimal APIs, la plantilla
  por defecto de .NET 10)
- Un proyecto de pruebas xUnit llamado ContosoBiker.Tests
- Ambos proyectos añadidos a la solución
- Una referencia del proyecto de pruebas al de la API
- El paquete Scalar.AspNetCore en el proyecto de API, para tener una interfaz
  interactiva sobre el documento OpenAPI
- El paquete Microsoft.AspNetCore.Mvc.Testing en el proyecto de pruebas

Ejecuta los comandos y muéstrame el resultado.
```

📝 **Si prefieres hacerlo a mano** (o el agente no ejecuta comandos en tu entorno):

```powershell
mkdir contoso-biker
cd contoso-biker

dotnet new sln -n ContosoBiker
dotnet new webapi  -n ContosoBiker.Api   -o ContosoBiker.Api
dotnet new xunit   -n ContosoBiker.Tests -o ContosoBiker.Tests

dotnet sln add ContosoBiker.Api/ContosoBiker.Api.csproj ContosoBiker.Tests/ContosoBiker.Tests.csproj
dotnet add ContosoBiker.Tests/ContosoBiker.Tests.csproj reference ContosoBiker.Api/ContosoBiker.Api.csproj

dotnet add ContosoBiker.Api/ContosoBiker.Api.csproj   package Scalar.AspNetCore
dotnet add ContosoBiker.Tests/ContosoBiker.Tests.csproj package Microsoft.AspNetCore.Mvc.Testing
```

✅ **Verifica** que la estructura quedó así:

```
contoso-biker/
├── ContosoBiker.slnx
├── ContosoBiker.Api/
│   ├── ContosoBiker.Api.csproj
│   ├── Program.cs
│   ├── appsettings.json
│   └── Properties/launchSettings.json
└── ContosoBiker.Tests/
    ├── ContosoBiker.Tests.csproj
    └── UnitTest1.cs
```

> 📝 **Dos detalles que sorprenden:**
> 1. **El archivo de solución es `.slnx`, no `.sln`.** Desde .NET 10, `dotnet new sln` genera el formato XML nuevo. Es normal y funciona igual.
> 2. **`dotnet new webapi` genera Minimal APIs**, no controladores. Si alguna vez necesitas controladores, el flag es `--use-controllers`.

🔧 **Un ajuste obligatorio que Copilot no suele hacer solo.** Abre `ContosoBiker.Tests/ContosoBiker.Tests.csproj` y cambia la primera línea:

```xml
<!-- Así viene de la plantilla -->
<Project Sdk="Microsoft.NET.Sdk">

<!-- Así debe quedar -->
<Project Sdk="Microsoft.NET.Sdk.Web">
```

> ⚠️ **Por qué importa:** `Microsoft.AspNetCore.Mvc.Testing` exige que el proyecto de pruebas use el SDK **Web**. Si no haces este cambio, las pruebas de integración del Ejercicio 3 fallarán con errores de dependencias que cuesta mucho diagnosticar. Pídeselo a Copilot así si prefieres:
>
> ```
> En ContosoBiker.Tests.csproj, cambia el SDK de Microsoft.NET.Sdk a
> Microsoft.NET.Sdk.Web, que es lo que requiere Microsoft.AspNetCore.Mvc.Testing.
> ```

También puedes borrar `ContosoBiker.Tests/UnitTest1.cs`: lo reemplazaremos en el Ejercicio 3.

---

### Paso 1.4 · El modelo de Cliente

Aquí empieza lo interesante. Vamos a pedir el código describiendo **intención y reglas de negocio**, no la sintaxis.

🤖 **PROMPT en modo Agent:**

```
Crea ContosoBiker.Api/Modelos/Cliente.cs con el modelo de los ciclistas
registrados en Contoso Biker.

Campos: Id, Nombre, Email, Telefono, Ciudad.

Reglas de negocio que quiero que queden expresadas como DataAnnotations, con
mensajes de error en español:
- El Id lo asigna la API, nunca el cliente HTTP.
- El nombre es obligatorio y tiene entre 3 y 80 caracteres.
- El email es obligatorio y debe tener formato de correo válido.
- El teléfono es obligatorio y son exactamente 10 dígitos.
- La ciudad es obligatoria y no pasa de 60 caracteres.

Documenta la clase y cada propiedad con comentarios XML en español.
```

📝 **Revisa la sugerencia antes de aceptar.** Pregúntate:

- ¿Los nombres están en español, como pediste en las instrucciones?
- ¿Usó `[Required]`, `[EmailAddress]`, `[StringLength]` y `[RegularExpression]`?
- ¿Los mensajes de error están en español y son entendibles para un usuario final?
- ¿El `namespace` es `ContosoBiker.Api.Modelos`?

> 💡 **Si generó el código en inglés**, añade al prompt: *"Sigue las instrucciones de `.github/copilot-instructions.md`."* Es el recordatorio más útil del taller.

> 🎓 **Concepto — por qué DataAnnotations y no `if`:** en .NET 10 la validación de Minimal APIs es automática. Basta con anotar el modelo y activar el servicio (lo haremos en el Paso 1.6) para que cualquier petición inválida reciba un `400` con el detalle de los errores **sin que tú escribas una sola línea de validación** en el endpoint.

---

### Paso 1.5 · El modelo de Bicicleta y los servicios en memoria

Ahora le pediremos que **repita un patrón que ya existe**. Esta es una de las cosas que Copilot hace mejor.

🤖 **PROMPT en modo Agent:**

```
#file:ContosoBiker.Api/Modelos/Cliente.cs

Siguiendo exactamente el mismo estilo de ese archivo, crea
ContosoBiker.Api/Modelos/Bicicleta.cs para el inventario de Contoso Biker.

Campos: Id, Codigo, Modelo, Tipo, PrecioPorDia, Estado, ClienteId, FechaAlta.

Reglas de negocio:
- El código sigue el formato BIKE-000 (tres dígitos).
- El modelo es obligatorio, entre 3 y 60 caracteres.
- El tipo solo puede ser: montaña, ruta, urbana o eléctrica.
- El precio por día va de 0 a 10000 y nunca es negativo.
- El estado solo puede ser: disponible, rentada o mantenimiento.
- ClienteId es opcional (int?) y es null cuando la bicicleta está disponible.
- FechaAlta es un DateOnly con valor por defecto la fecha de hoy.

Expón también los tipos y estados válidos como arreglos estáticos públicos,
para poder reutilizarlos desde las pruebas.
```

Después, los dos almacenes de datos:

🤖 **PROMPT en modo Agent:**

```
Crea los servicios de datos en memoria de Contoso Biker:

ContosoBiker.Api/Servicios/ClienteServicio.cs
ContosoBiker.Api/Servicios/BicicletaServicio.cs

Cada uno debe:
- Guardar los datos en un Dictionary<int, T> privado y llevar un contador de ids.
- Precargar datos de ejemplo realistas en español en el constructor: 3 clientes
  de distintas ciudades de México y 4 bicicletas (una de cada tipo) con estados
  variados: una rentada al cliente 1, dos disponibles y una en mantenimiento.
- Exponer: ObtenerTodos/ObtenerTodas, ObtenerPorId, Crear, Actualizar, Eliminar.
- Exponer una propiedad Total con el número de elementos.

Además, ClienteServicio necesita ExisteEmail(email, idAIgnorar) para detectar
correos duplicados ignorando mayúsculas.

Y BicicletaServicio necesita:
- ExisteCodigo(codigo, idAIgnorar), también ignorando mayúsculas.
- ObtenerPorCliente(clienteId).
- TotalDisponibles, que cuenta las bicicletas en estado "disponible".
- LiberarDeCliente(clienteId), que pone en "disponible" y sin cliente todas las
  bicicletas de un cliente que se dio de baja.

Documenta todo con comentarios XML en español.
```

📝 **Observa:**

- ¿Detectó el patrón del primer servicio y lo replicó en el segundo?
- ¿Los datos de ejemplo son coherentes? (¿la bicicleta rentada apunta a un cliente que existe?)
- ¿`ExisteEmail` compara con `StringComparison.OrdinalIgnoreCase`?

> 🌟 **Momento wow:** al pasar `#file:` y decir "el mismo estilo", Copilot lee el archivo real y replica su estructura, su nivel de documentación y hasta su forma de nombrar. **Cada participante obtendrá un resultado ligeramente distinto** — compáralos con quien tengas al lado, es el mejor ejercicio de la sesión.

> 🎓 **Concepto — `LiberarDeCliente` no es un capricho.** Es una regla de integridad: si borras un cliente, sus bicicletas no pueden quedar apuntando a un id que ya no existe. Copilot nunca habría inventado esta regla solo; la sabes tú, porque conoces el negocio. **Ese es tu trabajo en este taller.**

---

### Paso 1.6 · Los endpoints y la aplicación

🤖 **PROMPT en modo Agent:**

```
Crea los endpoints REST de Contoso Biker como métodos de extensión:

ContosoBiker.Api/Endpoints/ClientesEndpoints.cs   → MapClientesEndpoints()
ContosoBiker.Api/Endpoints/BicicletasEndpoints.cs → MapBicicletasEndpoints()

Cada archivo es una clase estática con un método de extensión sobre
IEndpointRouteBuilder que agrupa sus rutas con MapGroup y les pone WithTags.

Endpoints de /api/clientes:
- GET    /            → lista todos
- GET    /{id:int}    → uno, o 404 con { mensaje } si no existe
- POST   /            → crea; 409 si el email ya existe; 201 con la cabecera
                        Location apuntando al recurso nuevo
- PUT    /{id:int}    → actualiza; 404 si no existe; 409 si el email pertenece
                        a otro cliente
- DELETE /{id:int}    → elimina y libera sus bicicletas; 204 si todo bien,
                        404 si no existía

Endpoints de /api/bicicletas:
- Los mismos cinco, con estas diferencias:
  - GET / acepta el filtro opcional ?estado=disponible|rentada|mantenimiento
  - POST y PUT devuelven 409 si el código ya existe y 400 si el ClienteId
    indicado no corresponde a ningún cliente

Todos los endpoints deben declarar WithName, WithSummary y los Produces que
correspondan, para que la documentación OpenAPI sea completa.
Los servicios se reciben por inyección de dependencias en cada handler.
```

Y ahora el punto de entrada:

🤖 **PROMPT en modo Agent:**

```
Reescribe ContosoBiker.Api/Program.cs como la aplicación de Contoso Biker.

Servicios a registrar:
- ClienteServicio y BicicletaServicio como Singleton.
- AddValidation() para la validación automática de modelos de .NET 10.
- AddProblemDetails() para que los errores salgan en formato ProblemDetails.
- AddOpenApi() con un document transformer que ponga:
  Title "API de Contoso Biker", Version "v1" y una descripción en español.

Pipeline, en este orden exacto:
1. UseExceptionHandler() y UseStatusCodePages()
2. Solo en entorno de desarrollo: MapOpenApi() y MapScalarApiReference()
3. UseDefaultFiles() y después UseStaticFiles(), para poder servir más adelante
   wwwroot/index.html en la raíz "/"
4. MapClientesEndpoints() y MapBicicletasEndpoints()

Añade también un endpoint GET /api/estadisticas que devuelva un objeto con
totalClientes, totalBicicletas y bicicletasDisponibles, etiquetado con
WithTags("Estadísticas").

No agregues UseHttpsRedirection: en el taller trabajamos sobre HTTP en un
puerto fijo para evitar problemas de certificados.
Comenta cada bloque en español explicando qué hace.
```

⚠️ **El detalle que más gente rompe.** Revisa que el orden de estas tres líneas sea exactamente este:

```csharp
app.UseDefaultFiles();   // solo reescribe "/" → "/index.html"
app.UseStaticFiles();    // este es el que realmente entrega el archivo
```

> **Por qué:** `UseDefaultFiles` **no sirve archivos**, solo reescribe la URL. Si lo pones sin `UseStaticFiles` detrás, o si los pones en orden inverso, la raíz `/` devolverá **404** y perderás veinte minutos buscando el error en el HTML. Lo verás en acción en el Ejercicio 2.

Finalmente, fija el puerto para que todas las personas del taller usen la misma URL. Abre `ContosoBiker.Api/Properties/launchSettings.json` y deja el perfil `http` así:

```json
"http": {
  "commandName": "Project",
  "dotnetRunMessages": true,
  "launchBrowser": true,
  "launchUrl": "",
  "applicationUrl": "http://localhost:5080",
  "environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  }
}
```

> 💡 **Si la sugerencia de Copilot se quedó corta** —por ejemplo, olvidó `WithSummary` o no registró un servicio— no rehagas el prompt desde cero. Itera: *"Añade `WithSummary` y `Produces` a todos los endpoints de bicicletas"*. Iterar es la forma natural de trabajar con un agente, no una señal de que lo hiciste mal.

---

### Paso 1.7 · Ejecutar y explorar la documentación

🤖 **PROMPT en modo Agent:**

```
Compila la solución y ejecuta la API de Contoso Biker con el perfil http.
Si hay errores de compilación, corrígelos y vuelve a intentar.
```

📝 **Alternativa manual:**

```powershell
cd ContosoBiker.Api
dotnet run --launch-profile http
```

**Abre en el navegador:**

| URL | Qué deberías ver |
|-----|------------------|
| `http://localhost:5080/scalar` | Documentación interactiva de la API |
| `http://localhost:5080/openapi/v1.json` | El documento OpenAPI en crudo |
| `http://localhost:5080/api/clientes` | Los 3 clientes de ejemplo en JSON |
| `http://localhost:5080/api/bicicletas?estado=disponible` | Solo las bicicletas libres |

✅ **Verifica:**

- El título que aparece es **"API de Contoso Biker"**, no el nombre del ensamblado
- Los endpoints están agrupados por etiquetas: Clientes, Bicicletas, Estadísticas
- Cada endpoint muestra el resumen que escribiste en `WithSummary`
- Puedes lanzar peticiones reales desde Scalar

🧪 **Prueba la validación automática.** Desde Scalar, haz un `POST /api/clientes` con este cuerpo:

```json
{
  "nombre": "A",
  "email": "esto-no-es-un-email",
  "telefono": "123",
  "ciudad": ""
}
```

Deberías recibir un **400** con los cuatro errores descritos en español.

> 🌟 **Momento wow:** no escribiste ni una línea de validación en los endpoints, ni una línea de documentación OpenAPI a mano. Las anotaciones del modelo y `WithSummary` hicieron todo el trabajo. **Esto es lo que significa "la plataforma trabaja para ti".**

> 🎓 **Concepto — por qué ya no es Swagger:** hasta .NET 8, las plantillas incluían Swashbuckle. En .NET 9 se retiró (su mantenimiento comunitario se detuvo) y se sustituyó por `Microsoft.AspNetCore.OpenApi`, que **genera el documento pero no trae interfaz**. Por eso añadimos Scalar: es la capa visual sobre ese documento. Si ves tutoriales que hablan de `AddSwaggerGen()`, son anteriores a .NET 9.

---

### Paso 1.8 · Desafío bonus: las rentas ⭐

> 📝 **Este paso es OPCIONAL.** Es para quien terminó antes de tiempo. Si el grupo va justo, quien imparte puede indicar que se salte y pasar al Ejercicio 2. No afecta a nada de lo que viene después.

Ahora te toca a ti. Usa lo aprendido para que Copilot construya la funcionalidad completa de rentas.

🤖 **PROMPT sugerido (adáptalo a tu estilo):**

```
#codebase

Siguiendo los patrones que ya existen en el proyecto, implementa la
funcionalidad de rentas de Contoso Biker:

1. Modelos/Renta.cs
   Campos: Id, BicicletaId, ClienteId, Tipo, Dias, Monto, Fecha, Descripcion.
   Reglas: el tipo solo puede ser inicio, devolucion o extension; los días van
   de 1 a 30; la descripción no pasa de 150 caracteres. El Monto lo calcula la
   API, no el cliente.

2. Servicios/RentaServicio.cs
   Mismo patrón que los otros servicios, con datos de ejemplo coherentes con
   los clientes y bicicletas existentes. Añade una propiedad IngresoTotal que
   sume los montos de los movimientos que no son devoluciones.

3. Endpoints/RentasEndpoints.cs en /api/rentas
   Reglas de negocio del POST:
   - 400 si la bicicleta o el cliente no existen.
   - 409 si el tipo es "inicio" y la bicicleta no está disponible.
   - El monto se calcula como PrecioPorDia × Dias (0 para devoluciones).
   - Un movimiento "inicio" pone la bicicleta en "rentada" y le asigna el
     cliente; una "devolucion" la deja "disponible" y sin cliente.

4. Regístralo en Program.cs y añade totalRentas e ingresoTotal al endpoint
   /api/estadisticas.
```

> 💡 **Observa el poder de `#codebase`:** Copilot analiza los archivos que ya escribiste y genera código **consistente** con ellos — mismo estilo de servicio, misma forma de devolver errores, misma documentación. Compara el resultado con lo que habrías escrito tú.

> 🧠 **Reflexión:** el prompt anterior es largo. ¿Eso es malo? No. Lo largo no es el problema; lo **vago** sí. Cada línea de ese prompt es una decisión de negocio que solo tú podías tomar.

---

### 🛠️ Solución de problemas del Ejercicio 1

| Problema | Solución |
|----------|----------|
| `dotnet --version` muestra 8.x o 9.x | Instala el SDK de .NET 10 desde [aquí](https://dotnet.microsoft.com/download/dotnet/10.0) y reinicia la terminal |
| `The type or namespace 'Scalar' could not be found` | Falta el paquete: `dotnet add ContosoBiker.Api/ContosoBiker.Api.csproj package Scalar.AspNetCore` |
| `AddValidation()` no existe | Estás en .NET 9 o anterior. Es una novedad de .NET 10; comprueba que el `.csproj` tenga `<TargetFramework>net10.0</TargetFramework>` |
| Scalar responde 404 | Solo se mapea en desarrollo. Verifica que `ASPNETCORE_ENVIRONMENT` sea `Development` en `launchSettings.json` |
| El título de OpenAPI dice "ContosoBiker.Api" | Falta el document transformer en `AddOpenApi()`. Pídeselo a Copilot: *"añade un document transformer que ponga el título 'API de Contoso Biker'"* |
| `Failed to determine the https port for redirect` | Quita `app.UseHttpsRedirection()`: en este taller trabajamos sobre HTTP |
| El puerto 5080 está ocupado | Cambia `applicationUrl` en `launchSettings.json` a otro puerto y usa esa URL en todo el taller |
| La validación no rechaza datos inválidos | Falta `builder.Services.AddValidation()` en `Program.cs` |
| Copilot genera el código en inglés | Añade al prompt: *"Sigue las instrucciones de `.github/copilot-instructions.md`"* |

---

## 🔬 Ejercicio 2: Frontend e integración (30 min)

> ⚠️ **Requisito previo:** la API del Ejercicio 1 debe estar funcionando.

> 📝 **Enfoque deliberadamente simple:** el frontend es **un solo archivo HTML** en `wwwroot/index.html`, con Bootstrap 5 por CDN y JavaScript vanilla usando `fetch()`. **No hay React, ni Vue, ni npm, ni build de frontend.** Todo vive en un archivo y ASP.NET Core lo sirve directamente.

### Objetivos

- ✅ Crear una página que consuma tu propia API
- ✅ Generar JavaScript vanilla con `fetch()` mediante Copilot
- ✅ Entender el pipeline de archivos estáticos de ASP.NET Core
- ✅ Practicar `/explain` sobre código que no escribiste tú

---

### Paso 2.1 · La página principal

> 💡 **Nota:** todo el frontend vive en `ContosoBiker.Api/wwwroot/index.html`. ASP.NET Core lo sirve en `/` gracias a `UseDefaultFiles()` + `UseStaticFiles()`, que configuraste en el Paso 1.6. Como la página y la API comparten origen, `fetch('/api/...')` funciona sin configurar CORS.

🤖 **PROMPT en modo Agent:**

```
Crea ContosoBiker.Api/wwwroot/index.html: el panel de administración de
Contoso Biker.

Es UNA sola página con JavaScript vanilla embebido. Nada de React, Vue ni npm.

Requisitos:

1. Bootstrap 5 y Bootstrap Icons por CDN. Nada instalado localmente.

2. Barra de navegación superior con el nombre "Contoso Biker", un ícono de
   bicicleta, y tres enlaces que alternan secciones sin recargar la página:
   Inicio, Clientes, Bicicletas.

3. Sección Inicio: cuatro tarjetas de indicadores (Clientes, Bicicletas,
   Disponibles, Ingreso) que se llenan llamando a GET /api/estadisticas.

4. Sección Clientes:
   - Tabla con id, nombre, email, teléfono y ciudad, cargada desde
     GET /api/clientes.
   - Botón "Nuevo cliente" que abre un modal de Bootstrap con el formulario.
   - Botones de editar y eliminar en cada fila. El de eliminar pide confirmación.
   - El formulario hace POST para crear y PUT para editar.

5. Pie de página con "© Contoso Biker · Sistema de Gestión de Bicicletas".

6. Paleta visual de ciclismo: verde oscuro #1b4332 como color principal,
   verde medio #40916c para acentos, verde claro #b7e4c7 para texto sobre
   fondo oscuro. Fondo general gris muy claro.

7. Todo el JavaScript va en una sola etiqueta <script> al final del body.
   Escribe una función auxiliar que envuelva a fetch() y centralice el manejo
   de errores, de forma que:
   - Un 204 no intente parsear JSON.
   - Los errores de validación (que llegan como ProblemDetails con un objeto
     "errors") se muestren como un mensaje legible al usuario.
   - Los errores de negocio (que llegan como { mensaje }) también se muestren.
   - Los mensajes aparezcan en una alerta de Bootstrap arriba del contenido.

Los nombres de funciones y variables van en español.
```

> 🎓 **Concepto — el punto 7 es el que separa un demo de una app.** Le estás pidiendo a Copilot que maneje **tres formas distintas de respuesta** de tu API: sin cuerpo (204), errores de validación (`ProblemDetails`) y errores de negocio (`{ mensaje }`). Copilot no lo habría deducido: tú conoces el contrato porque lo diseñaste en el Ejercicio 1.

---

### Paso 2.2 · La sección de inventario

🤖 **PROMPT en modo Agent:**

```
#file:ContosoBiker.Api/wwwroot/index.html

Añade a esa misma página la sección de Bicicletas, reutilizando exactamente el
mismo patrón de JavaScript que ya usa la sección de Clientes.

1. Tabla con: código (en formato monoespaciado), modelo, tipo, precio por día,
   estado y nombre del cliente que la tiene rentada.

2. El precio se muestra como moneda mexicana con separadores de miles,
   usando Intl.NumberFormat('es-MX').

3. El estado se muestra como una insignia de color:
   disponible = verde, rentada = amarillo, mantenimiento = rojo.

4. Botón "Nueva bicicleta" con un modal que incluya:
   - Código, con validación del formato BIKE-000 en el propio input
   - Modelo
   - Tipo (desplegable: montaña, ruta, urbana, eléctrica)
   - Precio por día (numérico)
   - Estado (desplegable)
   - Cliente asignado: un desplegable que se llena desde GET /api/clientes,
     con una opción "Sin asignar" que envía null

5. Editar y eliminar por fila, igual que en Clientes.

6. Para pintar el nombre del cliente en cada fila, carga clientes y bicicletas
   en paralelo con Promise.all y construye un índice id → nombre.
```

> 💡 **Observa:** al pasar `#file:` y decir "el mismo patrón que ya usa la sección de Clientes", Copilot mantiene la coherencia del código. No mezcla estilos ni reinventa el manejo de errores que ya escribió.

---

### Paso 2.3 · Verificar el pipeline de archivos estáticos

> 📝 Este paso puede que ya esté resuelto si el Paso 1.6 salió bien. Compruébalo en treinta segundos.

🤖 **PROMPT en modo Ask:**

```
#file:ContosoBiker.Api/Program.cs

¿Este archivo puede servir wwwroot/index.html en la raíz "/"?
Revisa específicamente si llama a UseDefaultFiles() y a UseStaticFiles(),
y si están en el orden correcto. Explica qué hace cada uno.
```

Si falta algo:

🤖 **PROMPT en modo Agent:**

```
Añade a Program.cs, antes de mapear los endpoints de la API, las llamadas
UseDefaultFiles() y UseStaticFiles() en ese orden, para servir
wwwroot/index.html en la raíz. No modifiques los endpoints existentes.
```

> 🎓 **Concepto — por qué hacen falta los dos.** Es el error más común al servir archivos estáticos en ASP.NET Core moderno:
>
> | Método | Qué hace realmente |
> |--------|--------------------|
> | `UseDefaultFiles()` | **No sirve nada.** Es un reescritor de URL: convierte `/` en `/index.html` y pasa la petición al siguiente componente |
> | `UseStaticFiles()` | Es quien **lee el archivo de `wwwroot` y lo devuelve** |
> | `MapStaticAssets()` | Optimización de .NET 9+ (compresión, huella digital, caché). **Tampoco sirve documentos por defecto por sí sola** |
>
> Si configuras `UseDefaultFiles()` sin `UseStaticFiles()`, la raíz `/` devuelve **404**. Si los inviertes, también. Es un fallo silencioso: no hay excepción ni error en el log, simplemente no aparece la página.

---

### Paso 2.4 · Ejecutar y probar la integración

🤖 **PROMPT en modo Agent:**

```
Ejecuta la aplicación de Contoso Biker con el perfil http.
```

📝 **Alternativa manual:**

```powershell
cd ContosoBiker.Api
dotnet run --launch-profile http
```

**Abre `http://localhost:5080/`** y recorre esta lista:

| ✅ | Comprobación |
|----|--------------|
| ☐ | El panel carga y muestra la barra verde de Contoso Biker |
| ☐ | Las cuatro tarjetas de Inicio muestran números reales, no guiones |
| ☐ | La pestaña Clientes lista los 3 clientes de ejemplo |
| ☐ | Puedes crear un cliente nuevo desde el modal y aparece en la tabla |
| ☐ | Al crear un cliente con email inválido, aparece un mensaje de error **en español** |
| ☐ | Al intentar repetir un email existente, aparece el mensaje de conflicto |
| ☐ | La pestaña Bicicletas muestra las insignias de colores por estado |
| ☐ | Los precios se ven como `$320.00`, con formato de moneda |
| ☐ | La bicicleta rentada muestra el nombre de su cliente |
| ☐ | Al eliminar un cliente, sus bicicletas quedan "disponible" y sin cliente |
| ☐ | `http://localhost:5080/scalar` sigue funcionando |

> 🌟 **Momento wow:** el punto de "al eliminar un cliente, sus bicicletas quedan disponibles" atraviesa **las tres capas** que construiste: el JavaScript llama al endpoint, el endpoint invoca `LiberarDeCliente`, y el servicio actualiza el inventario. Y lo escribiste todo con prompts.

---

### Paso 2.5 · Entender el código con `/explain`

> 💡 **Concepto:** Copilot genera código rápido. `/explain` es lo que evita que ese código se convierta en una caja negra en tu repositorio.

📍 **Instrucciones:**

1. Abre `wwwroot/index.html`
2. Selecciona la función auxiliar que envuelve a `fetch()`
3. En Copilot Chat escribe:

🤖 **PROMPT:**

```
/explain Explícame esta función paso a paso:
1. ¿Qué hace en cada rama?
2. ¿Por qué trata el código 204 de forma especial?
3. ¿Qué pasa si la respuesta no es JSON válido?
4. ¿Qué problemas potenciales le ves? ¿Falta algún caso?
```

📝 **Observa:** Copilot explicará el flujo y probablemente señale huecos reales — por ejemplo, que no hay tiempo de espera máximo, o que un fallo de red lanza una excepción distinta a un error HTTP.

🧠 **Pregunta para el grupo:** ¿alguna de las mejoras que sugirió vale la pena implementarla? ¿Cuáles son ruido para un taller y cuáles serían imprescindibles en producción?

---

### 🛠️ Solución de problemas del Ejercicio 2

| Problema | Solución |
|----------|----------|
| `/` devuelve 404 | Falta `UseStaticFiles()` o está antes de `UseDefaultFiles()`. Ver Paso 2.3 |
| `/` devuelve la lista de endpoints en vez del HTML | El archivo no está en `ContosoBiker.Api/wwwroot/index.html`. Verifica la ruta exacta |
| La página carga pero las tablas están vacías | Abre las herramientas de desarrollo (`F12`) → pestaña Consola. Casi siempre es una ruta de `fetch` mal escrita |
| Bootstrap no aplica estilos | Revisa tu conexión: se carga desde CDN |
| Los modales no abren | Falta `bootstrap.bundle.min.js` al final del body, o el `id` del modal no coincide |
| Los acentos se ven como `Ã±` | Falta `<meta charset="UTF-8">` en el `<head>` |
| Los cambios en el HTML no se reflejan | Recarga forzada con `Ctrl+F5`, o reinicia `dotnet run` |
| El desplegable de clientes sale vacío | Se llena al entrar en la pestaña Bicicletas; comprueba que `GET /api/clientes` responde |

---

## 🔬 Ejercicio 3: Pruebas y refactoring (25 min)

> ⚠️ **Requisito previo:** el Ejercicio 1, con la API funcionando. El frontend no hace falta.

> 🎓 **Para quien imparte:** si vas justo de tiempo, prioriza los Pasos 3.1 a 3.4 (generar y ejecutar pruebas). Los Pasos 3.5 y 3.6 son valiosos pero prescindibles.

### Objetivos

- ✅ Generar pruebas unitarias con `/tests`
- ✅ Escribir pruebas de integración que llaman a la API de verdad
- ✅ Entender por qué unas pruebas son independientes y otras no
- ✅ Practicar refactoring asistido con `/explain` y `/fix`

---

### Paso 3.1 · Pruebas unitarias con `/tests`

> 💡 **Comando especial:** `/tests` genera pruebas para el código que tengas seleccionado.

📍 **Cómo usarlo:**

1. Abre `ContosoBiker.Api/Servicios/ClienteServicio.cs`
2. Selecciona todo el archivo (`Ctrl+A`)
3. Abre Copilot Chat y escribe:

🤖 **PROMPT:**

```
/tests Genera pruebas unitarias con xUnit para este servicio, en el archivo
ContosoBiker.Tests/ClienteServicioTests.cs.

Quiero cubrir estos escenarios:
1. Obtener todos devuelve los 3 clientes de ejemplo
2. Obtener por id con un id que existe devuelve el cliente correcto
3. Obtener por id con un id inexistente devuelve null
4. Crear asigna un id nuevo y aumenta el total
5. ExisteEmail detecta duplicados ignorando mayúsculas y minúsculas
6. ExisteEmail no considera duplicado el propio email al pasar idAIgnorar
7. Actualizar con un id existente modifica los datos
8. Actualizar con un id inexistente devuelve null
9. Eliminar con un id existente devuelve true y reduce el total
10. Eliminar con un id inexistente devuelve false

Requisitos:
- Nombres de prueba descriptivos en español, con el formato
  Metodo_Escenario_ResultadoEsperado
- Patrón Arrange-Act-Assert, separado por líneas en blanco
- Cada prueba crea su PROPIA instancia de ClienteServicio, de forma que las
  pruebas sean independientes y puedan ejecutarse en cualquier orden
```

> 🎓 **Concepto — la última línea es la importante.** El servicio guarda los datos en memoria. Si todas las pruebas compartieran una sola instancia, la que elimina un cliente rompería a la que cuenta cuántos hay, y el resultado dependería del orden de ejecución. Crear una instancia por prueba elimina el problema de raíz. **Esto es exactamente el tipo de detalle que Copilot no adivina y tú sí sabes.**

---

### Paso 3.2 · Pruebas del inventario

🤖 **PROMPT en modo Agent:**

```
#file:ContosoBiker.Tests/ClienteServicioTests.cs

Con el mismo estilo, crea ContosoBiker.Tests/BicicletaServicioTests.cs con
pruebas unitarias de BicicletaServicio:

- ObtenerTodas devuelve las 4 bicicletas de ejemplo
- Un [Theory] con [InlineData] que compruebe cuántas bicicletas hay en cada
  estado (disponible, rentada, mantenimiento)
- ObtenerPorCliente devuelve solo las bicicletas de ese cliente
- ExisteCodigo detecta duplicados ignorando mayúsculas
- Crear asigna un id nuevo
- LiberarDeCliente deja las bicicletas de ese cliente en estado "disponible"
  y con ClienteId en null
- Eliminar con un id inexistente devuelve false

Cada prueba crea su propia instancia del servicio.
```

> 💡 **Observa el `[Theory]`:** es la forma de xUnit de ejecutar la misma prueba con distintos datos. Si Copilot generó tres pruebas casi idénticas en vez de un `[Theory]`, pídeselo: *"unifica esas tres pruebas en un [Theory] con [InlineData]"*. Es un buen momento para hablar de duplicación en el código de pruebas.

---

### Paso 3.3 · Pruebas de integración de la API

Estas pruebas no llaman a los servicios: **levantan la aplicación completa en memoria y le hacen peticiones HTTP reales**.

🤖 **PROMPT en modo Agent:**

```
#codebase

Crea ContosoBiker.Tests/ApiIntegracionTests.cs con pruebas de integración de
la API de Contoso Biker, usando WebApplicationFactory<Program> y xUnit.

Cada prueba debe crear su propia instancia de WebApplicationFactory dentro de
un bloque using, para que los datos en memoria partan siempre del mismo estado.

Escenarios a cubrir:

Frontend y estadísticas
- GET / devuelve 200 con contenido text/html que contiene "Contoso Biker"
- GET /api/estadisticas devuelve los totales correctos

Clientes
- GET /api/clientes devuelve los 3 clientes
- GET /api/clientes/999 devuelve 404
- POST con datos válidos devuelve 201 y la cabecera Location correcta
- POST con un email mal formado devuelve 400
- POST sin nombre devuelve 400
- POST con un email que ya existe devuelve 409
- PUT sobre un cliente existente actualiza los datos
- DELETE de un cliente existente devuelve 204 y deja sus bicicletas
  en estado "disponible" y sin cliente
- DELETE de un cliente inexistente devuelve 404

Bicicletas
- GET /api/bicicletas?estado=disponible devuelve solo las disponibles
- POST con un tipo que no está permitido devuelve 400

Usa System.Net.Http.Json (GetFromJsonAsync, PostAsJsonAsync, PutAsJsonAsync)
y nombres de prueba descriptivos en español.
```

> ⚠️ **Si la compilación falla diciendo que no encuentra `Program`:** revisa que `ContosoBiker.Tests.csproj` use `<Project Sdk="Microsoft.NET.Sdk.Web">` (Paso 1.3) y que tenga referencia al proyecto de la API.

> 🎓 **Concepto — algo que cambió y confunde a mucha gente.** En .NET 9 y anteriores había que añadir esta línea al final de `Program.cs` para que las pruebas pudieran referenciar la clase:
>
> ```csharp
> public partial class Program { }   // ❌ ya NO hace falta en .NET 10
> ```
>
> **En .NET 10 un generador de código la emite automáticamente.** Es más: si la escribes a mano, un analizador te avisará de que la quites. Si Copilot te la sugiere —porque aprendió de miles de ejemplos escritos para versiones anteriores— **es un caso perfecto para practicar el juicio crítico: rechaza la sugerencia.**

> 🧠 **Reflexión para el grupo:** esto ilustra el límite real de Copilot. No "sabe" qué versión usas; predice lo más probable según lo que ha visto. Tu trabajo es saber cuándo lo más probable ya no es lo correcto.

---

### Paso 3.4 · Ejecutar las pruebas

🤖 **PROMPT en modo Agent:**

```
Ejecuta todas las pruebas de la solución de Contoso Biker y muéstrame el
resultado. Si alguna falla, analiza la causa y corrígela.
```

📝 **Alternativa manual:**

```powershell
# desde la carpeta raíz de la solución
dotnet test
```

✅ **Verifica:**

- Todas las pruebas pasan
- Las unitarias y las de integración se ejecutan juntas
- No hay errores de referencias ni de dependencias

Ejecuta `dotnet test` **dos veces seguidas**. Si el número de pruebas que pasan cambia entre ejecuciones, tienes pruebas que dependen del orden — vuelve al Paso 3.1 y revisa que cada una cree su propia instancia.

> 💡 **Tip:** para ver el nombre de cada prueba, usa `dotnet test -v normal`. Para ejecutar solo un grupo: `dotnet test --filter "FullyQualifiedName~ClienteServicio"`.

---

### Paso 3.5 · Refactoring con `/explain` y `/fix` *(si hay tiempo)*

> 💡 **Concepto:** hasta ahora usamos Copilot para **crear**. Ahora lo usamos para **mejorar**.

📍 **Ejercicio:**

1. Abre `ContosoBiker.Api/Endpoints/ClientesEndpoints.cs`
2. Selecciona el bloque completo de endpoints
3. En Copilot Chat:

🤖 **PROMPT:**

```
/explain Analiza este código con ojo crítico y dime:
1. ¿Hay lógica duplicada entre endpoints que se pueda extraer?
2. ¿Falta alguna validación de negocio importante?
3. ¿Hay algún riesgo de seguridad o de fuga de información en los mensajes
   de error?
4. ¿El manejo de errores es coherente entre todos los endpoints?
5. ¿Qué pasaría si dos peticiones simultáneas crearan un cliente a la vez?
```

Después, si alguna sugerencia te convence:

```
/fix Aplica la mejora número 1 que sugeriste, sin cambiar el contrato público
de los endpoints ni romper las pruebas existentes.
```

Y comprueba que no rompiste nada:

```
Ejecuta dotnet test y confírmame que todas las pruebas siguen pasando.
```

> 🌟 **Momento wow:** la pregunta 5 tiene una respuesta real e incómoda. `Dictionary<TKey, TValue>` **no es seguro para acceso concurrente**, y el contador `_siguienteId++` tampoco es atómico. En un taller no importa; en producción sería un error serio. Copilot suele detectarlo y proponer `ConcurrentDictionary` o `Interlocked.Increment`. **Esa es la clase de revisión que cuesta cara si la haces tarde.**

> 🧭 **Disciplina:** nunca apliques un `/fix` sin ejecutar las pruebas después. El ciclo correcto es **explicar → corregir → probar**, siempre en ese orden.

---

### Paso 3.6 · Documentación con `/doc` *(si hay tiempo)*

📍 **Instrucciones:**

1. Abre `ContosoBiker.Api/Servicios/BicicletaServicio.cs`
2. Coloca el cursor sobre un método que no tenga comentarios XML
3. Abre el chat en línea del editor con `Ctrl+I` y escribe:

🤖 **PROMPT:**

```
/doc Genera comentarios XML completos en español para este método:
- Resumen claro de qué hace y cuándo usarlo
- <param> para cada parámetro
- <returns> describiendo el valor devuelto, incluyendo qué significa null
- <example> con una línea de uso
```

> 💡 **Por qué importa más de lo que parece:** los comentarios XML no solo documentan para las personas. Alimentan IntelliSense, pueden incorporarse al documento OpenAPI y **se convierten en contexto para el propio Copilot** en futuras sesiones. Documentar bien hoy hace que Copilot acierte más mañana.

---

### 🛠️ Solución de problemas del Ejercicio 3

| Problema | Solución |
|----------|----------|
| No compila: no encuentra el tipo `Program` | `ContosoBiker.Tests.csproj` debe usar `<Project Sdk="Microsoft.NET.Sdk.Web">` |
| `WebApplicationFactory` no existe | Falta el paquete: `dotnet add ContosoBiker.Tests/ContosoBiker.Tests.csproj package Microsoft.AspNetCore.Mvc.Testing` |
| No encuentra los modelos desde las pruebas | Falta la referencia de proyecto (Paso 1.3) |
| Las pruebas pasan solas pero fallan juntas | Comparten estado. Cada prueba debe crear su propia instancia del servicio o de la fábrica |
| El resultado cambia entre ejecuciones | Mismo problema de estado compartido |
| `dotnet test` no encuentra pruebas | Ejecútalo desde la carpeta de la solución y confirma que existe `Microsoft.NET.Test.Sdk` |
| Un aviso pide quitar `public partial class Program` | Hazle caso: en .NET 10 sobra |
| `/tests` genera pruebas incompletas | Selecciona menos código o enumera los escenarios explícitamente en el prompt |
| `/fix` no modifica nada | Cambia a modo Agent: en modo Ask no puede tocar archivos |

---

## 🤖 Ejercicio 4: Creación de agentes (35 min)

> ⚠️ **Requisito previo:** tener el proyecto de los Ejercicios 1 a 3. Si te quedaste atrás, no importa: este ejercicio funciona igual con lo que tengas.

Hasta aquí has usado a Copilot **tal como viene**. En este ejercicio vas a **moldearlo**: definir cómo piensa, qué reglas sigue, qué herramientas puede tocar y qué papel juega en tu equipo.

Esta es la parte que convierte a Copilot de "un autocompletado muy bueno" en **infraestructura de tu equipo de desarrollo**.

### Objetivos

- ✅ Entender los niveles de personalización y cuándo usar cada uno
- ✅ Escribir instrucciones que se aplican solo a ciertos archivos
- ✅ Crear un **agente personalizado** desde cero
- ✅ Restringir las herramientas de un agente y entender por qué eso importa
- ✅ Orquestar varios agentes (subagentes)
- ✅ Conectar herramientas externas con **MCP**
- ✅ Delegar trabajo completo al **agente en la nube** de GitHub

---

### Paso 4.0 · El mapa: cinco niveles de personalización

Antes de crear nada, ubica las piezas. Todas conviven y **se acumulan**.

```
                    ┌──────────────────────────────────┐
     Más amplio     │  1. copilot-instructions.md      │  Reglas para TODO el repo
          ▲         │     .github/copilot-instructions.md│
          │         └──────────────────────────────────┘
          │         ┌──────────────────────────────────┐
          │         │  2. *.instructions.md            │  Reglas por tipo de archivo
          │         │     .github/instructions/        │  (applyTo: '**/*.cs')
          │         └──────────────────────────────────┘
          │         ┌──────────────────────────────────┐
          │         │  3. AGENTS.md                    │  Reglas compartidas entre
          │         │     raíz del repositorio         │  distintas herramientas de IA
          │         └──────────────────────────────────┘
          │         ┌──────────────────────────────────┐
          │         │  4. *.agent.md                   │  Un "personaje" con rol,
          │         │     .github/agents/              │  herramientas y modelo propios
          │         └──────────────────────────────────┘
          ▼         ┌──────────────────────────────────┐
     Más específico │  5. MCP servers                  │  Herramientas NUEVAS que
                    │     .vscode/mcp.json             │  Copilot no tenía
                    └──────────────────────────────────┘
```

| Nivel | Archivo | Cuándo usarlo |
|-------|---------|---------------|
| **1. Instrucciones del repo** | `.github/copilot-instructions.md` | Reglas que aplican siempre: idioma, stack, convenciones |
| **2. Instrucciones por ruta** | `.github/instructions/*.instructions.md` | Reglas que solo aplican a ciertos archivos (pruebas, frontend, SQL…) |
| **3. Instrucciones entre herramientas** | `AGENTS.md` | Reglas compartidas con otras herramientas de IA además de Copilot |
| **4. Agentes personalizados** | `.github/agents/*.agent.md` | Un rol concreto: revisor, documentador, arquitecto… |
| **5. Servidores MCP** | `.vscode/mcp.json` | Conectar Copilot a sistemas externos: bases de datos, APIs, navegador |

> ⚠️ **Si vienes de material anterior:** los **"custom chat modes"** (`.chatmode.md`) pasaron a llamarse **"custom agents"** (`.agent.md`). Es el mismo concepto con nombre nuevo. Si tienes archivos `.chatmode.md`, basta con renombrarlos a `.agent.md` y moverlos a `.github/agents/`.

---

### Paso 4.1 · Instrucciones que aplican solo a ciertos archivos

El `copilot-instructions.md` del Paso 1.2 aplica a todo. Pero hay reglas que solo tienen sentido en un contexto: las convenciones de pruebas no aplican al HTML, y las del HTML no aplican a los endpoints.

Para eso existen los archivos `.instructions.md` con la propiedad **`applyTo`**: un patrón glob que decide a qué archivos se adjuntan **automáticamente**.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/instructions/pruebas.instructions.md con este contenido
exacto:

---
name: 'Pruebas de Contoso Biker'
description: 'Convenciones al crear o modificar pruebas del proyecto.'
applyTo: '**/ContosoBiker.Tests/**/*.cs'
---

# Convenciones de pruebas

- Usa xUnit. Nunca MSTest ni NUnit.
- Nombra cada prueba como `Metodo_Escenario_ResultadoEsperado`, en español.
  Ejemplo: `ObtenerPorId_ConIdInexistente_DevuelveNull`.
- Estructura cada prueba en Arrange-Act-Assert, separando los tres bloques con
  una línea en blanco. No escribas los comentarios // Arrange, // Act, // Assert.
- Cada prueba crea su propia instancia del servicio o su propia
  `WebApplicationFactory`. Nunca compartas estado entre pruebas: los datos
  viven en memoria y el orden de ejecución no está garantizado.
- Cuando varias pruebas solo cambian en los datos, únelas en un `[Theory]`
  con `[InlineData]`.
- Para las pruebas de integración usa `System.Net.Http.Json`
  (`GetFromJsonAsync`, `PostAsJsonAsync`, `PutAsJsonAsync`).
- No añadas `public partial class Program { }`: en .NET 10 se genera solo.
- Afirma también los códigos de estado HTTP, no solo el cuerpo de la respuesta.
````

Crea ahora una segunda, para el frontend:

````
Crea el archivo .github/instructions/frontend.instructions.md:

---
name: 'Frontend de Contoso Biker'
description: 'Convenciones para la página de administración.'
applyTo: '**/wwwroot/**/*.html'
---

# Convenciones del frontend

- JavaScript vanilla únicamente. Nada de React, Vue, jQuery ni npm.
- Bootstrap 5 y Bootstrap Icons se cargan por CDN.
- Todo el JavaScript va en una sola etiqueta `<script>` al final del `<body>`.
- Los nombres de funciones y variables van en español.
- Todas las llamadas a la API pasan por la función auxiliar que envuelve a
  `fetch()`. No llames a `fetch()` directamente desde la lógica de pantalla.
- Los errores siempre se muestran al usuario en una alerta de Bootstrap,
  nunca solo en la consola.
- Los importes se formatean con `Intl.NumberFormat('es-MX')`.
- Usa rutas relativas (`/api/...`): la página y la API comparten origen.
````

🧪 **Comprueba que funciona.** Abre cualquier archivo de `ContosoBiker.Tests` y pide algo en modo Agent:

```
Añade una prueba que verifique que POST /api/bicicletas con un precio negativo
devuelve 400.
```

Fíjate en el resultado: debería nombrar la prueba en español con el formato indicado, no incluir los comentarios `// Arrange`, y crear su propia fábrica. **No se lo pediste en este prompt** — vino del archivo `pruebas.instructions.md`.

> 💡 **Los tres campos del encabezado:**
>
> | Campo | Para qué sirve |
> |-------|----------------|
> | `name` | Nombre visible en la interfaz |
> | `description` | Permite que Copilot lo descubra y lo adjunte por su cuenta cuando es relevante |
> | `applyTo` | Patrón glob. Se adjunta **automáticamente** al trabajar con archivos que coinciden. Usa `'**'` para todos |
>
> Si omites `description` y `applyTo`, el archivo solo se usa cuando lo adjuntas a mano.

> ⚠️ **Límite importante:** las instrucciones personalizadas afectan al **chat**, no al autocompletado en línea mientras escribes en el editor.

---

### Paso 4.2 · `AGENTS.md`: instrucciones que sobreviven a la herramienta

`AGENTS.md` es un formato **abierto y compartido** entre distintos asistentes de IA. No es específico de Copilot. La idea: escribes las reglas del proyecto una vez, en la raíz del repositorio, y cualquier herramienta compatible las respeta.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo AGENTS.md en la raíz del repositorio:

# Contoso Biker — Guía para agentes de IA

## Qué es este proyecto
API REST y panel de administración para una red de renta de bicicletas.
Los datos viven en memoria: no hay base de datos y no debe añadirse ninguna.

## Estructura
- `ContosoBiker.Api/Modelos/` — entidades con validaciones DataAnnotations
- `ContosoBiker.Api/Servicios/` — almacenes en memoria, registrados como Singleton
- `ContosoBiker.Api/Endpoints/` — un archivo por entidad, con un método de
  extensión `Map<Entidad>Endpoints`
- `ContosoBiker.Api/wwwroot/` — la página de administración
- `ContosoBiker.Tests/` — pruebas unitarias y de integración

## Cómo compilar y probar
```powershell
dotnet build
dotnet test
dotnet run --project ContosoBiker.Api --launch-profile http
```
La aplicación queda en http://localhost:5080 y la documentación en /scalar.

## Reglas que no se negocian
- Todo el código y la documentación en español.
- Minimal APIs agrupadas con `MapGroup`. Nunca controladores.
- La validación se hace con DataAnnotations en el modelo, no con `if` en el
  endpoint.
- Los códigos de estado son significativos: 404 si no existe, 409 si hay
  conflicto, 204 al eliminar, 201 con cabecera `Location` al crear.
- No introduzcas dependencias nuevas sin justificarlo.
- Después de cualquier cambio de código, ejecuta `dotnet test`.
````

> 💡 **`copilot-instructions.md` o `AGENTS.md`, ¿cuál uso?** Puedes tener los dos y se complementan. En la práctica: `copilot-instructions.md` para reglas de estilo específicas de Copilot; `AGENTS.md` para el conocimiento del proyecto que cualquier agente necesita — cómo se compila, cómo se prueba, qué no se debe tocar. `AGENTS.md` además lo lee el agente en la nube de GitHub (Paso 4.7).

---

### Paso 4.3 · Qué es exactamente un agente personalizado

Un **agente personalizado** es un archivo Markdown que define un **rol** para Copilot. Contiene:

- Un **encabezado** con metadatos: nombre, descripción, herramientas permitidas, modelo preferido
- Un **cuerpo** con las instrucciones de ese rol, que se antepone a lo que tú escribas en el chat

Se guardan en `.github/agents/` con extensión `.agent.md`.

| Dónde lo pones | Alcance |
|----------------|---------|
| `.github/agents/` | Solo este repositorio. Se comparte con el equipo por Git |
| `~/.copilot/agents/` | Todos tus proyectos, solo para ti |

**Campos del encabezado que más vas a usar:**

| Campo | Para qué |
|-------|----------|
| `description` | Qué hace el agente. Aparece como texto guía en el chat |
| `name` | Su nombre. Si lo omites, se usa el del archivo |
| `tools` | Lista de herramientas permitidas. **Si lo omites, tiene acceso a todas** |
| `model` | Un modelo, o una lista en orden de preferencia |
| `agents` | Otros agentes que puede invocar como subagentes |
| `argument-hint` | Pista de uso que se muestra en el campo de chat |
| `handoffs` | Botones de acción sugerida al terminar una respuesta |
| `user-invocable` | `false` para agentes que solo se usan como subagentes |

> 🎓 **Por qué `tools` es el campo más importante.** Un agente sin `tools` puede hacer **cualquier cosa**: editar archivos, ejecutar comandos, borrar. Un agente revisor que solo puede **leer** no puede romper nada aunque se equivoque. Restringir herramientas no es burocracia: es **diseño de seguridad**.

---

### Paso 4.4 · Tu primer agente: el revisor de APIs

Vamos a crear un agente que revise código de la API contra las reglas de Contoso Biker **sin poder modificar nada**.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/agents/revisor-api.agent.md con este contenido exacto:

---
name: Revisor de API
description: Revisa endpoints y modelos de Contoso Biker contra las convenciones del proyecto. Solo lee y reporta, nunca modifica código.
argument-hint: Indica el archivo o la carpeta que quieres revisar
tools: ['search/codebase', 'search/usages']
---

# Revisor de API de Contoso Biker

Eres una persona revisora de código especializada en APIs de ASP.NET Core.
Tu trabajo es **encontrar problemas y explicarlos**, nunca corregirlos.
No edites archivos ni ejecutes comandos bajo ninguna circunstancia.

## Qué revisas, en este orden

1. **Contrato HTTP**
   - ¿Cada endpoint devuelve el código de estado correcto?
     404 si no existe, 409 si hay conflicto, 204 al eliminar,
     201 con cabecera `Location` al crear.
   - ¿Hay algún endpoint que devuelva 200 con cuerpo vacío?

2. **Validación**
   - ¿Las reglas están como DataAnnotations en el modelo, o se colaron
     como `if` dentro del endpoint?
   - ¿Los mensajes de error están en español y los entendería un usuario final?

3. **Documentación OpenAPI**
   - ¿Cada endpoint declara `WithName`, `WithSummary` y los `Produces`
     que corresponden?
   - ¿Los resúmenes describen la intención o solo repiten el nombre del método?

4. **Convenciones del proyecto**
   - ¿Nombres en español, salvo términos técnicos del framework?
   - ¿Se usa `MapGroup` en lugar de controladores?
   - ¿Hay comentarios XML en las clases y métodos públicos?

5. **Riesgos reales**
   - ¿Los mensajes de error filtran información interna?
   - ¿Hay problemas de concurrencia en los servicios en memoria?
   - ¿Hay lógica duplicada entre endpoints?

## Cómo entregas el resultado

Devuelve una tabla y nada más antes de ella:

| Severidad | Archivo | Línea | Hallazgo | Corrección sugerida |
|-----------|---------|-------|----------|---------------------|

Severidades: 🔴 Alta, 🟡 Media, ⚪ Baja.

Después de la tabla, añade un párrafo corto llamado **Veredicto** con tu
valoración general y las dos cosas más importantes a arreglar primero.

Si no encuentras ningún problema, dilo explícitamente en lugar de inventar
hallazgos menores para rellenar.
````

> 💡 **Atajo:** también puedes crear agentes sin escribir el archivo a mano. Pulsa `Ctrl+Shift+P` → **Chat: New Custom Agent**, o escribe `/create-agent` en el chat y describe lo que quieres. Prueba los dos caminos y quédate con el que prefieras.

---

### Paso 4.5 · Usar tu agente

📍 **Cómo invocarlo:**

1. Abre Copilot Chat (`Ctrl+Alt+I`)
2. Abre el desplegable de **agente** en el panel de chat
3. Selecciona **Revisor de API**
4. Escribe tu petición

🤖 **PROMPT (con el agente Revisor de API seleccionado):**

```
Revisa ContosoBiker.Api/Endpoints/ClientesEndpoints.cs y
ContosoBiker.Api/Endpoints/BicicletasEndpoints.cs
```

📝 **Observa tres cosas:**

1. **El formato de respuesta es el que definiste**, con su tabla y su veredicto. No tuviste que pedirlo en el prompt.
2. **No intentó corregir nada**, aunque encuentre problemas evidentes.
3. Si le pides explícitamente *"arregla el problema 1"*, **no podrá**: no tiene herramientas de edición.

🧪 **Pruébalo.** Con el agente Revisor seleccionado, escribe:

```
Corrige el primer hallazgo de tu tabla.
```

Te dirá que no puede modificar archivos. **Ese es exactamente el comportamiento que diseñaste.**

> 🌟 **Momento wow:** acabas de crear un rol reutilizable. Cualquier persona que clone este repositorio tiene ese revisor disponible al instante, con los mismos criterios. **Las convenciones de tu equipo dejaron de vivir en un wiki que nadie lee y pasaron a ser ejecutables.**

---

### Paso 4.6 · Un segundo agente: el documentador

Un agente por rol. Vamos por el segundo, este sí con permiso de edición.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/agents/documentador.agent.md:

---
name: Documentador
description: Añade y mejora comentarios XML en español en el código de Contoso Biker, sin cambiar la lógica.
argument-hint: Indica el archivo que quieres documentar
tools: ['search/codebase', 'edit']
---

# Documentador de Contoso Biker

Tu única tarea es mejorar la documentación del código. **Nunca cambias la
lógica**: ni renombras, ni reordenas, ni "aprovechas para" refactorizar.
Si ves un error de lógica, lo mencionas al final, pero no lo tocas.

## Reglas

- Comentarios XML (`///`) en español, en todas las clases y miembros públicos.
- `<summary>` explica **para qué sirve** y **cuándo usarlo**, no repite el
  nombre del método. "Obtiene el cliente por id" es inútil;
  "Busca un cliente en el inventario en memoria; devuelve null si no existe"
  es útil.
- `<param>` por cada parámetro, explicando qué se espera y qué formato.
- `<returns>` que describa el valor devuelto **y qué significa null**.
- `<exception>` si el método puede lanzar.
- En los métodos con reglas de negocio, añade `<remarks>` explicando la regla.

## Al terminar

Resume en una lista qué archivos tocaste y cuántos miembros documentaste.
Si detectaste algún problema de lógica, ponlo bajo el título
**Observaciones (no corregidas)**.
````

🧪 **Úsalo:** selecciona el agente **Documentador** y escribe:

```
Documenta ContosoBiker.Api/Servicios/BicicletaServicio.cs
```

Después revisa el diff. ¿Respetó la regla de no tocar la lógica?

> 🧠 **Pregunta para el grupo:** ¿por qué el Documentador sí tiene `edit` y el Revisor no? ¿Qué pasaría si le diéramos `edit` al Revisor? Discútanlo: es una decisión de diseño real que van a tomar en sus equipos.

---

### Paso 4.7 · Orquestación: agentes que llaman a otros agentes

Un agente puede delegar en otros mediante el campo **`agents`**. Esto permite dividir un trabajo grande en especialistas.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/agents/auditor.agent.md:

---
name: Auditor de calidad
description: Audita una funcionalidad completa de Contoso Biker combinando revisión de API y documentación.
tools: ['agent', 'search/codebase']
agents: ['Revisor de API', 'Documentador']
---

# Auditor de calidad de Contoso Biker

Coordinas una auditoría completa de la funcionalidad que se te indique.
No revisas ni documentas tú: **delegas** y después integras los resultados.

## Proceso

1. Localiza todos los archivos relacionados con la funcionalidad indicada:
   su modelo, su servicio, sus endpoints y sus pruebas.
2. Invoca al agente **Revisor de API** sobre el modelo y los endpoints.
   Pídele su tabla de hallazgos.
3. Si la revisión señala falta de documentación, invoca al agente
   **Documentador** sobre los archivos afectados.
4. Integra todo en un informe final.

## Informe final

- **Alcance**: qué archivos entraron en la auditoría
- **Hallazgos**: la tabla consolidada del revisor
- **Documentación**: qué se documentó, si aplicó
- **Plan de acción**: las tres cosas a arreglar primero, en orden de impacto

Sé honesto: si la funcionalidad está bien, dilo. No infles el informe.
````

🧪 **Úsalo:** selecciona **Auditor de calidad** y escribe:

```
Audita la funcionalidad completa de bicicletas.
```

Verás en el chat cómo el auditor **invoca a los otros dos agentes** y espera sus resultados antes de redactar el informe.

> ⚠️ **Dos requisitos que se olvidan siempre:**
> 1. Para que un agente pueda delegar, la herramienta **`agent` debe estar en su lista `tools`**. Si no, el campo `agents` se ignora en silencio.
> 2. Los nombres en `agents` deben coincidir **exactamente** con el campo `name` de los otros agentes, respetando mayúsculas y acentos.

> 🎓 **Concepto — por qué dividir en varios agentes.** Cada agente empieza con contexto limpio y una sola responsabilidad. Un agente que "lo hace todo" acaba con instrucciones contradictorias y resultados peores. Es el mismo principio de responsabilidad única que aplicas a tus clases.

---

### Paso 4.8 · MCP: darle herramientas que no tenía

Un agente solo puede usar las herramientas que existen. **MCP (Model Context Protocol)** es el estándar que permite conectar Copilot a sistemas externos: bases de datos, APIs, navegadores, tu gestor de incidencias.

La configuración vive en `.vscode/mcp.json`:

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .vscode/mcp.json con este contenido:

{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp"
    }
  }
}
````

📍 **Formas de añadir servidores MCP:**

| Método | Cómo |
|--------|------|
| Galería de extensiones | `Ctrl+Shift+X` → escribe `@mcp` → **Install** |
| Paleta de comandos | `Ctrl+Shift+P` → **MCP: Add Server** |
| A mano | Editar `.vscode/mcp.json` |

🧪 **Pruébalo.** En modo Agent, con el servidor de GitHub configurado:

```
Busca en el repositorio dotnet/aspnetcore las incidencias abiertas
relacionadas con la validación de Minimal APIs, y resume las tres más
relevantes para nuestro proyecto.
```

Copilot usará las herramientas del servidor MCP de GitHub para consultar datos reales.

**Para restringir qué herramientas MCP puede usar un agente**, inclúyelas en `tools`:

```yaml
tools: ['search/codebase', 'github/*']
```

> El formato `<servidor>/*` incluye todas las herramientas de ese servidor MCP.

> 🛡️ **Seguridad, en serio.** Un servidor MCP local **ejecuta código en tu máquina**. Añade solo servidores de origen confiable. Y nunca escribas claves de API directamente en `mcp.json`: usa variables de entrada o un archivo de entorno.

> 💡 **Nota práctica:** un archivo `.mcp.json` en la raíz del repositorio (con la clave `mcpServers` en vez de `servers`) es el formato portable, que funciona también fuera de VS Code. Úsalo si quieres compartir la configuración con el resto del equipo independientemente de su editor.

---

### Paso 4.9 · El agente en la nube: delegar trabajo completo

Hasta ahora todo pasó en tu máquina. GitHub tiene además un **agente en la nube** que trabaja **en tu repositorio**, en su propio entorno, y abre pull requests con el resultado.

**La diferencia:**

| | Agente en el editor | Agente en la nube |
|--|--------------------|-------------------|
| Dónde corre | Tu máquina | Entorno efímero de GitHub Actions |
| Qué produce | Cambios en tu copia local | Una rama y un pull request |
| Tú qué haces | Supervisas en vivo | Escribes la incidencia y revisas el PR |
| Ideal para | Trabajo con el que iteras | Tareas bien acotadas y repetitivas |

📍 **Cómo asignarle una incidencia:**

1. En GitHub, entra a tu repositorio
2. Pestaña **Issues** → abre o crea una incidencia
3. En el panel derecho, **Assignees**
4. Selecciona **Copilot**
5. Rellena el campo de **prompt opcional** con las indicaciones concretas
6. Elige el repositorio y la rama base
7. Si tienes agentes personalizados, puedes elegir uno desde el desplegable

🧪 **Ejercicio.** Crea una incidencia en tu repositorio con este contenido:

```markdown
Título:
Añadir búsqueda de clientes por ciudad

Descripción:
El endpoint GET /api/clientes devuelve todos los clientes sin posibilidad de
filtrar. El panel de administración necesita poder buscar por ciudad.

Aceptación:
- GET /api/clientes acepta el parámetro opcional ?ciudad=
- La comparación ignora mayúsculas, minúsculas y acentos
- Si no se envía el parámetro, el comportamiento actual no cambia
- El endpoint queda documentado en OpenAPI describiendo el nuevo filtro
- Hay pruebas de integración para: con filtro que encuentra resultados,
  con filtro sin resultados, y sin filtro
- `dotnet test` pasa en verde
```

> ⚠️ **El detalle que más frustra a la gente:** cuando asignas una incidencia, el agente recibe el **título, la descripción y los comentarios que existan en ese momento**. **No lee comentarios que añadas después.** Si necesitas darle más información, escríbela como comentario **en el pull request** que abra, no en la incidencia.

> 💡 El agente en la nube también lee tu `.github/copilot-instructions.md`, tus `.github/instructions/*.instructions.md` y tu `AGENTS.md`. **Todo el trabajo de los pasos 4.1 y 4.2 se aplica también aquí.** Por eso vale la pena hacerlo bien.

---

### Paso 4.10 · Preparar el entorno del agente en la nube *(bonus)*

El agente en la nube arranca en un contenedor Linux limpio. Si tu proyecto necesita algo instalado antes —como el SDK de .NET 10— se lo dices con un flujo de trabajo especial.

🤖 **PROMPT en modo Agent:**

````
Crea el archivo .github/workflows/copilot-setup-steps.yml:

name: "Copilot Setup Steps"

on:
  workflow_dispatch:
  push:
    paths:
      - .github/workflows/copilot-setup-steps.yml
  pull_request:
    paths:
      - .github/workflows/copilot-setup-steps.yml

jobs:
  # El trabajo DEBE llamarse copilot-setup-steps o Copilot no lo usará.
  copilot-setup-steps:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Instalar el SDK de .NET 10
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: '10.0.x'

      - name: Restaurar dependencias
        run: dotnet restore
````

> ⚠️ **Dos reglas estrictas:**
> 1. El trabajo **tiene que llamarse `copilot-setup-steps`**. Con cualquier otro nombre, Copilot lo ignora.
> 2. El archivo **debe estar en la rama por defecto** para que surta efecto.

> 💡 Puedes validarlo manualmente desde la pestaña **Actions** del repositorio antes de asignar ninguna incidencia.

---

### 🛠️ Solución de problemas del Ejercicio 4

| Problema | Solución |
|----------|----------|
| Mi agente no aparece en el desplegable | Debe estar en `.github/agents/` con extensión `.agent.md`, y `user-invocable` no debe ser `false` |
| El agente ignora mis instrucciones | Revisa que el encabezado esté entre `---` al principio del archivo y que el YAML sea válido |
| El agente edita archivos y no debería | Falta restringir `tools`. **Sin `tools`, tiene acceso a todo** |
| El agente no puede editar y debería | Añade `'edit'` a su lista `tools` |
| `agents:` no hace nada | Falta incluir `'agent'` en `tools` |
| Un subagente no se encuentra | El nombre debe coincidir exactamente con el campo `name` del otro agente, con acentos y mayúsculas |
| Las instrucciones `applyTo` no se aplican | El patrón glob es relativo a la raíz del espacio de trabajo y usa `/` como separador, también en Windows |
| El servidor MCP no arranca | `Ctrl+Shift+P` → **MCP: List Servers** para ver su estado y su registro de salida |
| No puedo asignar una incidencia a Copilot | Necesitas permiso de escritura y que el agente en la nube esté habilitado en el repositorio |
| `copilot-setup-steps.yml` se ignora | El trabajo debe llamarse exactamente así y el archivo debe estar en la rama por defecto |
| Tengo archivos `.chatmode.md` antiguos | Renómbralos a `.agent.md` y muévelos a `.github/agents/` |

---

### 🧭 Cierre del ejercicio: qué te llevas

| Si necesitas… | Usa… |
|---------------|------|
| Una regla para todo el repositorio | `.github/copilot-instructions.md` |
| Una regla solo para ciertos archivos | `.github/instructions/*.instructions.md` con `applyTo` |
| Que otras herramientas de IA respeten tus reglas | `AGENTS.md` |
| Un rol especializado y reutilizable | `.github/agents/*.agent.md` |
| Que ese rol no pueda romper nada | El campo `tools` |
| Dividir un trabajo grande entre especialistas | El campo `agents` + la herramienta `agent` |
| Conectar Copilot a un sistema externo | Un servidor MCP en `.vscode/mcp.json` |
| Delegar una tarea completa y recibir un PR | Asignar la incidencia a Copilot en GitHub |

> 🌟 **La idea que cierra el taller:** los agentes no son "prompts guardados". Son **la forma de codificar el criterio de tu equipo** para que se aplique igual a todo el mundo, todos los días, sin depender de que alguien se acuerde de la convención.

---

## 📖 Referencia rápida

### Referencias de contexto

| Referencia | Qué aporta |
|------------|------------|
| `#codebase` | Búsqueda en todo el espacio de trabajo |
| `#file:ruta/archivo.cs` | Un archivo concreto |
| `#selection` | La selección actual del editor |
| `#fetch https://...` | El contenido de una página web |
| `#githubRepo owner/repo` | Código de un repositorio público |
| `#usages` | Dónde se usa un símbolo |
| `#terminalLastCommand` | La salida del último comando |

> `@workspace` pertenece a versiones anteriores. Hoy su equivalente es `#codebase`.

### Comandos de chat

| Comando | Para qué |
|---------|----------|
| `/explain` | Explicar código |
| `/fix` | Proponer y aplicar una corrección |
| `/tests` | Generar pruebas |
| `/doc` | Generar documentación (chat en línea del editor) |
| `/plan` | Investigar y proponer un plan |
| `/new` | Crear la estructura de un proyecto |
| `/agents` | Configurar agentes |
| `/instructions` | Configurar instrucciones |
| `/create-agent` | Generar un agente nuevo |
| `/clear` | Empezar una conversación nueva |
| `/models` | Abrir el selector de modelo |
| `/help` | Ver comandos y agentes disponibles |

### Atajos de teclado

| Atajo (Windows / Linux) | Acción |
|-------------------------|--------|
| `Ctrl+Alt+I` | Abrir el panel de Copilot Chat |
| `Ctrl+I` | Chat en línea (en el editor o en la terminal) |
| `Ctrl+Shift+Alt+L` | Chat rápido |
| `Tab` | Aceptar la sugerencia en línea |
| `Esc` | Descartar la sugerencia |
| `Ctrl+→` | Aceptar solo la siguiente palabra |
| `Ctrl+Shift+P` | Paleta de comandos (**Chat: New Custom Agent**, **MCP: Add Server**…) |

> En macOS: `⌃⌘I` para el panel, `⌘I` para el chat en línea.

### Archivos de personalización

| Archivo | Ubicación | Para qué |
|---------|-----------|----------|
| `copilot-instructions.md` | `.github/` | Reglas de todo el repositorio |
| `*.instructions.md` | `.github/instructions/` | Reglas por patrón de archivo (`applyTo`) |
| `AGENTS.md` | raíz del repositorio | Reglas compartidas entre herramientas de IA |
| `*.agent.md` | `.github/agents/` | Agentes personalizados |
| `mcp.json` | `.vscode/` | Servidores MCP |
| `copilot-setup-steps.yml` | `.github/workflows/` | Entorno del agente en la nube |

### Comandos de .NET usados en el taller

```powershell
dotnet --version                              # versión del SDK
dotnet new sln -n ContosoBiker                # solución (genera .slnx en .NET 10)
dotnet new webapi -n Proyecto -o Proyecto     # Web API con Minimal APIs
dotnet new webapi --use-controllers -o Proy   # variante con controladores
dotnet new xunit -n Pruebas -o Pruebas        # proyecto de pruebas
dotnet sln add ruta/Proyecto.csproj           # añadir a la solución
dotnet add Pruebas.csproj reference Api.csproj  # referencia entre proyectos
dotnet add Api.csproj package Scalar.AspNetCore # paquete NuGet
dotnet restore                                # restaurar dependencias
dotnet build                                  # compilar
dotnet run --launch-profile http              # ejecutar
dotnet test                                   # ejecutar todas las pruebas
dotnet test -v normal                         # ver el nombre de cada prueba
dotnet test --filter "FullyQualifiedName~Cliente"  # ejecutar un subconjunto
```

### URLs de la aplicación

| URL | Qué es |
|-----|--------|
| `http://localhost:5080/` | Panel de administración |
| `http://localhost:5080/scalar` | Documentación interactiva de la API |
| `http://localhost:5080/openapi/v1.json` | Documento OpenAPI |
| `http://localhost:5080/api/clientes` | Endpoint de clientes |
| `http://localhost:5080/api/bicicletas` | Endpoint de bicicletas |
| `http://localhost:5080/api/estadisticas` | Resumen del panel |

### Hábitos que mejoran tus resultados

| Hábito | Ejemplo |
|--------|---------|
| Nombres descriptivos | `ObtenerBicicletasDisponibles()` en lugar de `GetData()` |
| Comentarios con intención | `// Valida que la bicicleta esté libre antes de rentarla` |
| Contexto de negocio explícito | Menciona "Contoso Biker", "renta", "inventario" |
| Apuntar a un archivo ejemplo | `#file:Modelos/Cliente.cs` + "sigue este mismo estilo" |
| Iterar en vez de rehacer | "Ahora añade `WithSummary` a todos los endpoints" |
| Describir el contrato completo | Qué entra, qué sale y qué pasa cuando falla |
| Ejecutar las pruebas tras cada cambio | "Ejecuta `dotnet test` y confirma que todo pasa" |
| Revisar siempre el diff | Nunca aceptes código que no puedas explicar |

---

## 🆘 ¿Te quedaste atrás?

Tranquilidad: el valor de este taller está en **experimentar con Copilot**, no en terminar cada línea de código.

| Situación | Qué hacer |
|-----------|-----------|
| No terminé el Ejercicio 1 | Pide a quien imparte el código de referencia para poder seguir con el Ejercicio 2 |
| No terminé el Ejercicio 2 | El Ejercicio 3 solo necesita la API. Puedes hacer pruebas sin frontend |
| No terminé el Ejercicio 3 | El Ejercicio 4 funciona con cualquier código que tengas. **No te lo pierdas: es la parte más diferencial** |
| No hice el bonus de rentas | Es opcional por diseño. No afecta a nada |
| Copilot me generó algo distinto a mi compañero | **Es normal y esperado.** Compárenlos: entender por qué difieren enseña más que el código en sí |
| Voy muy adelantado | Prueba el Paso 1.8 (rentas), crea un tercer agente propio, o conecta un servidor MCP distinto |

> 🎓 **Recomendación para quien imparte:** mantén una rama `solucion` en el repositorio del taller con el código completo de referencia. Así quien se quede atrás puede descargar lo que le falta y reincorporarse al grupo sin perderse el resto.

---

## ✅ Checklist final

Al terminar deberías tener:

### Personalización de Copilot

- [ ] `.github/copilot-instructions.md` — reglas de todo el proyecto
- [ ] `.github/instructions/pruebas.instructions.md` — convenciones de pruebas
- [ ] `.github/instructions/frontend.instructions.md` — convenciones del frontend
- [ ] `AGENTS.md` — guía del proyecto para agentes de IA
- [ ] `.github/agents/revisor-api.agent.md` — agente revisor (solo lectura)
- [ ] `.github/agents/documentador.agent.md` — agente documentador
- [ ] `.github/agents/auditor.agent.md` — agente orquestador
- [ ] `.vscode/mcp.json` — al menos un servidor MCP configurado
- [ ] `.github/workflows/copilot-setup-steps.yml` — entorno del agente en la nube *(bonus)*

### Backend

- [ ] `ContosoBiker.slnx` — solución
- [ ] `ContosoBiker.Api/Program.cs` — servicios, pipeline y endpoints
- [ ] `ContosoBiker.Api/Modelos/Cliente.cs` — con DataAnnotations
- [ ] `ContosoBiker.Api/Modelos/Bicicleta.cs` — con DataAnnotations
- [ ] `ContosoBiker.Api/Modelos/Renta.cs` *(⭐ bonus)*
- [ ] `ContosoBiker.Api/Servicios/ClienteServicio.cs`
- [ ] `ContosoBiker.Api/Servicios/BicicletaServicio.cs`
- [ ] `ContosoBiker.Api/Servicios/RentaServicio.cs` *(⭐ bonus)*
- [ ] `ContosoBiker.Api/Endpoints/ClientesEndpoints.cs`
- [ ] `ContosoBiker.Api/Endpoints/BicicletasEndpoints.cs`
- [ ] `ContosoBiker.Api/Endpoints/RentasEndpoints.cs` *(⭐ bonus)*

### Documentación de la API

- [ ] `/scalar` accesible y funcional
- [ ] El título dice "API de Contoso Biker"
- [ ] Los endpoints están agrupados por etiquetas
- [ ] Cada endpoint muestra su resumen y sus códigos de respuesta
- [ ] Un POST con datos inválidos devuelve 400 con mensajes en español

### Frontend

- [ ] `ContosoBiker.Api/wwwroot/index.html`
- [ ] Tarjetas de indicadores con datos reales de la API
- [ ] Tabla de clientes con alta, edición y baja
- [ ] Tabla de bicicletas con insignias de estado y precios formateados
- [ ] Los errores de la API se muestran al usuario en español

### Pruebas

- [ ] `ContosoBiker.Tests/ContosoBiker.Tests.csproj` usa `Microsoft.NET.Sdk.Web`
- [ ] `ContosoBiker.Tests/ClienteServicioTests.cs`
- [ ] `ContosoBiker.Tests/BicicletaServicioTests.cs`
- [ ] `ContosoBiker.Tests/ApiIntegracionTests.cs`
- [ ] `dotnet test` pasa en verde
- [ ] El resultado es el mismo en dos ejecuciones seguidas

---

## 🙋 Preguntas frecuentes

### ¿Por qué datos en memoria y no una base de datos?

| Ventaja | Detalle |
|---------|---------|
| Sin instalación | Nadie pierde media hora con SQL Server o PostgreSQL |
| Instantáneo | Las operaciones no esperan a disco ni a red |
| Portable | Se comporta igual en cualquier máquina del grupo |
| Enfocado | El taller trata de Copilot, no de configurar bases de datos |

**¿Y cómo migro a una base de datos real?** Pídeselo a Copilot:

```
#codebase Migra los servicios en memoria a Entity Framework Core con SQLite.
Mantén la misma interfaz pública de cada servicio para no romper los endpoints
ni las pruebas existentes. Añade las migraciones y actualiza Program.cs.
```

### ¿Por qué Scalar y no Swagger?

Hasta .NET 8, las plantillas de Web API incluían Swashbuckle, que generaba tanto el documento como la interfaz de Swagger UI. En **.NET 9 se retiró de las plantillas** (su mantenimiento comunitario se había detenido) y se sustituyó por `Microsoft.AspNetCore.OpenApi`, que **genera el documento pero no incluye interfaz**.

Así que hoy eliges tú la capa visual. Scalar es moderna, se integra en una línea y funciona bien con OpenAPI 3.1. Swagger UI y ReDoc siguen siendo alternativas válidas.

> ⚠️ Por buena práctica de seguridad, **estas interfaces solo deben exponerse en desarrollo**. Por eso en el taller van dentro del `if (app.Environment.IsDevelopment())`.

### ¿Copilot genera código distinto para cada persona?

Sí, y es **intencional**. Copilot considera tu contexto: los archivos que tienes abiertos, el código que ya escribiste, tus comentarios, tu estilo. Dos personas con el mismo prompt pueden obtener soluciones distintas y **ambas correctas**. Compararlas es uno de los mejores ejercicios del taller.

### ¿Cómo consigo que genere código en español?

1. Déjalo escrito en `.github/copilot-instructions.md` (Paso 1.2)
2. Escribe tus comentarios y nombres de variables en español
3. Si aún así se va al inglés, añade al prompt: *"Sigue las instrucciones de `.github/copilot-instructions.md`"*

### ¿Qué hago cuando sugiere algo incorrecto?

1. **No lo aceptes por inercia.** Léelo.
2. Pulsa `Esc` para descartar y reformula con más contexto.
3. Pregúntale por qué: *"¿por qué usaste ese enfoque? ¿hay alternativas?"*
4. Recuerda que **Copilot es un asistente, no una autoridad**. La responsabilidad del código es tuya.

Un caso real de este taller: Copilot suele sugerir `public partial class Program { }` en `Program.cs` porque lo ha visto miles de veces en ejemplos de .NET 6 a 9. En .NET 10 **sobra**. Detectar eso es exactamente la habilidad que este taller quiere entrenar.

### ¿Mi interfaz de Copilot se ve distinta a la del taller?

Es muy probable, y no es un problema. Copilot evoluciona rápido: los modos, los íconos y la ubicación de los selectores cambian entre versiones. Los **conceptos** de este material —modos, referencias de contexto, instrucciones, agentes, MCP— son estables. Si algo no lo encuentras, consulta la [documentación oficial de VS Code](https://code.visualstudio.com/docs/copilot/overview) o pregunta a quien imparte.

### ¿Cuál es la diferencia entre un "chat mode" y un "agente personalizado"?

Son lo mismo con distinto nombre. Los *custom chat modes* pasaron a llamarse *custom agents*. Si tienes archivos `.chatmode.md`, renómbralos a `.agent.md` y colócalos en `.github/agents/`.

### ¿Qué modelo debo elegir?

Copilot ofrece varios modelos y **la lista cambia cada pocas semanas**, así que este material no recomienda ninguno en concreto. Usa el selector de modelo del campo de chat y prueba:

- Para preguntas rápidas y código sencillo, un modelo ligero responde antes
- Para tareas complejas de varios archivos, un modelo de razonamiento acierta más
- La opción **Auto** elige por ti según la complejidad de la petición

En agentes personalizados puedes fijar una preferencia con el campo `model`, incluso como lista ordenada de alternativas.

### ¿Esto sirve si mi equipo no programa en C#?

Sí. La parte de C# es el vehículo; lo que se enseña —prompting con intención, referencias de contexto, instrucciones, agentes, MCP— es **independiente del lenguaje**. Para adaptarlo, cambia el stack del Ejercicio 1 y ajusta los archivos de instrucciones.

---

## 📚 Recursos adicionales

### GitHub Copilot

- [Documentación de GitHub Copilot](https://docs.github.com/en/copilot)
- [Copilot en VS Code](https://code.visualstudio.com/docs/copilot/overview)
- [Personalización: instrucciones, agentes y skills](https://code.visualstudio.com/docs/agent-customization/custom-instructions)
- [Agentes personalizados](https://code.visualstudio.com/docs/agent-customization/custom-agents)
- [Servidores MCP en VS Code](https://code.visualstudio.com/docs/agent-customization/mcp-servers)
- [Agente en la nube de GitHub](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent)
- [github/awesome-copilot](https://github.com/github/awesome-copilot) — instrucciones y agentes de la comunidad

### .NET y ASP.NET Core

- [Novedades de ASP.NET Core en .NET 10](https://learn.microsoft.com/aspnet/core/release-notes/aspnetcore-10.0)
- [Minimal APIs](https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis/overview)
- [OpenAPI en ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/openapi/aspnetcore-openapi)
- [Validación en Minimal APIs](https://learn.microsoft.com/aspnet/core/fundamentals/validation)
- [Archivos estáticos](https://learn.microsoft.com/aspnet/core/fundamentals/static-files)
- [Pruebas de integración](https://learn.microsoft.com/aspnet/core/test/integration-tests)
- [Ciclo de vida y soporte de .NET](https://learn.microsoft.com/dotnet/core/releases-and-support)

### Herramientas del taller

- [Scalar para ASP.NET Core](https://github.com/scalar/scalar)
- [xUnit](https://xunit.net/)
- [Bootstrap 5](https://getbootstrap.com/docs/5.3/)
- [Model Context Protocol](https://modelcontextprotocol.io/)

### El siguiente nivel

- **Copilot en la terminal** — pulsa `Ctrl+I` dentro de la terminal integrada de VS Code y describe el comando que necesitas
- **Mensajes de commit** — deja que Copilot los redacte a partir de tus cambios desde el panel de control de versiones
- **Revisión de código** — solicita una revisión de Copilot en tus pull requests
- **Agent Skills** — capacidades reutilizables en `.github/skills/<nombre>/SKILL.md`, invocables con `/<nombre>`
- **Enlaces (hooks)** — comandos que se ejecutan automáticamente en puntos concretos de una sesión de agente

---

## 👥 Créditos

**Taller:** Fundamentos de GitHub Copilot con C# y ASP.NET Core

**Escenario:** Contoso Biker, una red ficticia de renta de bicicletas creada para fines educativos. Cualquier parecido con una empresa real es casualidad.

**Stack:** GitHub Copilot · C# 14 · .NET 10 · ASP.NET Core Minimal APIs · OpenAPI 3.1 · Scalar · xUnit · Bootstrap 5

**Duración:** 3 horas · 4 ejercicios prácticos

**Idioma:** español

### Inspiración

La estructura pedagógica de este taller —el recorrido Ask → Agent, los momentos wow, las tablas de solución de problemas y el enfoque de "complejidad baja, aprendizaje alto"— está inspirada en el workshop [armandoblanco/workshop-githubcopilot-basico](https://github.com/armandoblanco/workshop-githubcopilot-basico), que usa Python y Flask sobre un escenario bancario. Gracias por compartirlo. 🙌

Esta versión reescribe el contenido para **C# y ASP.NET Core**, cambia el dominio a la renta de bicicletas y añade un **cuarto módulo completo dedicado a la creación de agentes**.

### Verificación del contenido

Todos los comandos, rutas, snippets y comportamientos descritos en los Ejercicios 1 a 3 fueron **ejecutados y verificados** contra el SDK de .NET `10.0.401`: la solución compila, la aplicación arranca en el puerto 5080, `/`, `/scalar` y los endpoints responden correctamente, y la batería de pruebas pasa completa.

El contenido del Ejercicio 4 está basado en la documentación oficial vigente de VS Code y GitHub. Ten presente que **la interfaz y la nomenclatura de Copilot evolucionan rápido**: si algo no coincide con lo que ves, el concepto sigue siendo válido pero conviene contrastar con la documentación enlazada.

---

> 🎉 **¡Gracias por participar!** Ya tienes lo necesario para usar GitHub Copilot como par de programación —y, más importante, para **configurarlo a la medida de tu equipo**.
>
> 🚲 *Contoso Biker: porque el mejor código, como la mejor ruta, se disfruta más en buena compañía.*

