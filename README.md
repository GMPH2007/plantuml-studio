# 🎨 PlantUML Studio IDE

> **Entorno de desarrollo web profesional, reactivo y en tiempo real para modelado y diagramación UML.**  
> Desarrollado y diseñado por **[**GMPH2007**](https://github.com/GMPH2007)**.

[![En vivo en Vercel](https://img.shields.io/badge/Demo_En_Vivo-Vercel-black?style=for-the-badge&logo=vercel)](https://plantuml-studio-wheat.vercel.app)
[![Licencia](https://img.shields.io/badge/Licencia-MIT-blue?style=for-the-badge)](#licencia)
[![Seguridad HTTPS](https://img.shields.io/badge/Seguridad-TLS_1.3_%7C_HTTPS-success?style=for-the-badge)](https://plantuml-studio-wheat.vercel.app)

---

## 🌐 Enlace del Proyecto en Producción
🔗 **Acceso público en la nube:** [https://plantuml-studio-wheat.vercel.app](https://plantuml-studio-wheat.vercel.app)

---

## 📖 Descripción del Proyecto

**PlantUML Studio IDE** es una aplicación web interactiva de alto rendimiento desarrollada para crear, editar, diagnosticar y exportar diagramas UML en tiempo real. 

Nació como respuesta a las limitaciones de los editores web tradicionales de PlantUML (demoras en compilación, publicidad invasiva, ausencia de números de línea, nula adaptación a pantallas móviles y mensajes de error crípticos del motor en inglés).

Este proyecto ofrece una experiencia inspirada en **Visual Studio Code**, completamente en español, sin publicidad, con diagnósticos asistidos paso a paso y optimizada tanto para computadoras de escritorio como para teléfonos móviles y tablets.

---

## 🚀 Características Principales

- ⚡ **Compilación y Renderizado Ultra Rápido (0 ms con Caché):** Renderizado en tiempo real con sistema de caché en memoria (`Map`) y cancelación de peticiones obsoletas mediante `AbortController`.
- ✏️ **Editor con Gutter Numerado:** Numeración de líneas sincronizada con scroll táctil y de rueda del ratón.
- 🛠️ **Diagnóstico Inteligente de Errores y Auto-Fix (1 Clic):** Analizador léxico en el cliente que detecta la línea exacta del fallo, explica en español qué pasó y ofrece un botón de reparación automática.
- 📱 **100% Responsivo para Celulares y Tablets:**
  - Sistema de pestañas móviles táctiles: `[✏️ Editor Código]`, `[👁️ Diagrama]` y `[📱 Dividido]`.
  - Controles de zoom interactivos (`[-]`, `[100%]`, `[+]`) de 40% a 300%.
  - Barra deslizante de snippets rápidos (Actor, Participante, Base de Datos, Flechas, Alt/Else, Notas, Clases).
- 📋 **Exportación en Alta Definición (HD 2.5x):**
  - Copiado directo al portapapeles como imagen PNG transparente o con fondo.
  - Descarga instantánea en formatos PNG y SVG vectorial.
- 🌓 **Sistema Dual de Temas:**
  - Modo Oscuro (*Dark Neón / Catppuccin inspired*) y Modo Claro (*Moderno Azul / Clean Slate*).
  - Persistencia de preferencias del usuario en `localStorage`.
- 🛡️ **Seguridad y Privacidad Estricta:**
  - Todo el código se procesa de forma efímera en el navegador. No se almacena código privado en bases de datos.
  - Conexión cifrada de grado bancario (TLS 1.3 / AES-128).
  - Cabeceras de seguridad HTTP configuradas en `vercel.json`.

---

## 🛠️ ¿Cómo se Hizo? (Arquitectura Técnica y Metodología)

El proyecto fue desarrollado bajo una arquitectura **SPA (Single Page Application)** moderna utilizando **Vanilla Web Standards** para garantizar máxima velocidad, portabilidad y cero dependencias pesadas:

```
┌─────────────────────────────────────────────────────────────┐
│                      PLANTUML STUDIO                        │
├──────────────────────────────┬──────────────────────────────┤
│       PANEL DE EDICIÓN       │       PANEL DE PREVIEW       │
│  • Textarea + Gutter         │  • Renderizado SVG Vectorial │
│  • Linter Léxico en Cliente  │  • Zoom Matricial (40%-300%) │
│  • Debounce 180ms            │  • Canvas Rasterizer (2.5x)  │
│  • AbortController           │  • Tarjeta de Error Auto-Fix │
└──────────────┬───────────────┴──────────────┬───────────────┘
               │                              │
               ▼                              ▼
      [ Caché en Memoria ]          [ Portapapeles / PNG ]
               │
               ▼ (HTTP POST HTTPS / TLS 1.3)
      [ Motor Kroki / PlantUML ]
```

### 1. Frontend & Diseño Responsivo (HTML5 + CSS3)
- **Variables CSS (`:root` & `body.light-mode`):** Permiten cambiar de tema visual instantáneamente en toda la aplicación con una sola clase.
- **CSS Grid & Flexbox:** Maquetación a 2 columnas en pantallas de escritorio (`1.05fr` para editor y `1.25fr` para diagrama).
- **Media Queries para Móviles (`@media (max-width: 820px)`):**
  - Transforma el grid en una arquitectura de pestañas independientes para evitar scroll vertical excesivo.
  - Ajuste de tipografía a `16px` para eliminar el comportamiento molesto de auto-zoom en iOS Safari y Android Chrome al interactuar con el teclado.
  - `touch-action: manipulation` y `-webkit-overflow-scrolling: touch` para desplazamiento fluido.

### 2. Motor de Compilación y Concurrencia (JavaScript ES6+)
- **Debounce de 180 ms:** Agrupa las pulsaciones de teclado del usuario antes de disparar la compilación, ahorrando peticiones innecesarias.
- **Cancelación con `AbortController`:** Si el usuario sigue escribiendo mientras una petición de renderizado está en curso, la señal aborta la petición previa para evitar carreras de datos (*race conditions*).
- **Caché en Memoria (`Map`):** Almacena en memoria RAM las combinaciones de código ya generadas. Al volver a un estado previo o borrar/deshacer, la respuesta es inmediata (**0 ms de latencia**).

### 3. Analizador Léxico y Corrección de Errores
- Los motores remotos de PlantUML a menudo reportan líneas desfasadas cuando hay palabras divididas (por ejemplo: presionar Enter dentro de `skinparam` separándolo en dos líneas).
- Se implementó un algoritmo cliente con expresiones regulares (`VALID_START_REGEX`, `ORPHAN_STYLE_REGEX`, `ARROW_REGEX`) que:
  1. Detecta directivas divididas entre líneas contiguas.
  2. Valida la ortografía de palabras clave (`skinparam`, `participant`, `actor`).
  3. Resalta las líneas exactas en color rojo en la numeración (`.gutter-num.error-active`).
  4. La función `executeAutoFix()` ejecuta la transformación del AST/código y recompila automáticamente sin intervención manual.

### 4. Rasterización Gráfica y Portapapeles (Canvas API)
- Para exportar a PNG en alta calidad:
  1. Se clona el elemento SVG generado y se serializa a XML con `XMLSerializer`.
  2. Se carga como blob de imagen en un elemento `Image()` asíncrono.
  3. Se dibuja sobre un `HTMLCanvasElement` aplicando un multiplicador de escala de **2.5x DPI**.
  4. Se convierte a blob PNG (`canvas.toBlob()`) y se transfiere al portapapeles mediante la API moderna `navigator.clipboard.write([new ClipboardItem(...)])`.

### 5. Despliegue en la Nube y CI/CD
- **Alojamiento:** Vercel Edge Network.
- **Integración:** Repositorio en GitHub vinculado para despliegues continuos.
- **Seguridad:** Archivo `vercel.json` con cabeceras estrictas:
  - `X-Frame-Options: DENY` (Anti Clickjacking)
  - `X-Content-Type-Options: nosniff` (Anti MIME sniffing)
  - `Referrer-Policy: strict-origin-when-cross-origin`
  - `Permissions-Policy: camera=(), microphone=(), geolocation=()`

---

## 📂 Estructura de Archivos

```plaintext
plantuml-studio/
├── index.html              # Aplicación IDE completa (SPA: HTML + CSS + JS)
├── vercel.json             # Configuración de cabeceras HTTP de seguridad
├── logo.png                # Logotipo oficial de alta resolución
├── logo.svg                # Logotipo vectorial escalable
├── favicon.svg             # Favicon para pestañas de navegador
├── README.md               # Documentación académica y técnica
├── diagrama_claro.puml      # Plantilla de ejemplo con tema Moderno Azul
├── diagrama_oscuro.puml     # Plantilla de ejemplo con tema Dark Neón
└── diagrama_actividad.puml  # Plantilla de ejemplo para diagramas de actividad
```

---

## 💻 Cómo Ejecutar el Proyecto

### Opción 1: Directamente en la Web (Sin instalar nada)
Ingresa con cualquier navegador moderno a:  
👉 **[https://plantuml-studio-wheat.vercel.app](https://plantuml-studio-wheat.vercel.app)**

### Opción 2: En tu Computadora Local
1. Clona el repositorio:
   ```bash
   git clone https://github.com/GMPH2007/plantuml-studio.git
   ```
2. Entra a la carpeta del proyecto:
   ```bash
   cd plantuml-studio
   ```
3. Abre el archivo `index.html` en tu navegador favorito (Chrome, Edge, Firefox, Safari) o utiliza una extensión como *Live Server* en VS Code.

---

## 👨‍🏫 Información Académica y Autoría

| Rol | Nombre | Identificador / Enlace |
| :--- | :--- | :--- |
| **Docente** | **Ing. Javier Eduardo Jaramillo Atoche** | Docente de Ingeniería |
| **Estudiante / Desarrollador** | **GMPH2007** | [@GMPH2007](https://github.com/GMPH2007) |
| **Repositorio Oficial** | **PlantUML Studio** | [github.com/GMPH2007/plantuml-studio](https://github.com/GMPH2007/plantuml-studio) |
| **Aplicación Web en Vivo** | **Vercel Cloud** | [plantuml-studio-wheat.vercel.app](https://plantuml-studio-wheat.vercel.app) |

> 🏷️ **Marca de agua oficial:** Todos los diagramas generados y exportados por la aplicación incorporan automáticamente la firma de autoría con los datos del docente y estudiante.

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Libre para uso educativo, académico y profesional.
