# Pisitos (Travel and Activities) - Automatización Inmobiliaria con RPA 🚀

<img width="1254" height="1254" alt="pisitos" src="https://github.com/user-attachments/assets/5e80667f-8753-41ef-aa35-f9575f4f973f" />

![Versión](https://img.shields.io/badge/Versi%C3%B3n-1.0-blue)
![Plataforma](https://img.shields.io/badge/Power_Automate-Desktop_%26_Cloud-green)
![Entorno](https://img.shields.io/badge/Entorno-Microsoft_365_%26_Chrome-orange)

Este repositorio contiene la solución automatizada de **RPA** para el proyecto académico **"Pisitos"**, enfocado en la monitorización, extracción y consolidación de ofertas inmobiliarias desde la plataforma Idealista de forma 100% desatendida.

El objetivo principal es eliminar la carga operativa manual de búsqueda de inmuebles, procesando criterios parametrizados y distribuyendo informes automatizados por correo electrónico.

---

## 📊 Arquitectura del Sistema

El robot está diseñado bajo el patrón de arquitectura desacoplada **Dispatcher-Performer**, garantizando una ejecución asíncrona, escalable y resiliente ante fallos de red.

[ Excel Entrada ] ──> ( Dispatcher ) ──> [ Cola en la Nube (ColBuscadorIdealista) ]
│
[ Informe Gmail ] <── ( Performer ) <── [ Navegación + Extracción Web (Chrome) ]


### 1. Componente Dispatcher (Preparador de Colas)
* **Función:** Inicializa y lee el archivo local de criterios (`idealista_criterio_entrada.xlsx`).
* **Procesamiento:** Transforma de forma secuencial cada fila de búsqueda (Municipio, Rango de Precios, Habitaciones) en un objeto estructurado bajo el estándar **JSON**.
* **Carga:** Inyecta los objetos en la cola de trabajo en la nube llamada `ColBuscadorIdealista`.

### 2. Componente Performer (Robot Ejecutor)
* **Ingesta:** Extrae las transacciones de la cola y las convierte en variables de filtrado.
* **Normalización:** Limpia automáticamente las cadenas de texto (conversión a minúsculas, eliminación de tildes y acentos) para evitar errores de codificación en la URL de Idealista.
* **Extracción:** Manipula el DOM del navegador Google Chrome, genera tablas virtuales en memoria y escribe los resultados controlando el límite máximo de registros configurado por el usuario (`UserInput`).
* **Persistencia y Salida:** Vuelca la información en una plantilla Excel local (`Reñas_limpio_idea.xlsx`) y la envía adjunta mediante SMTP de Gmail.

---

## 🛠️ Tecnologías y Aplicaciones Utilizadas

* **Power Automate Desktop (v. 2.x):** Motor principal de automatización del flujo local.
* **Power Automate Cloud:** Gestión y orquestación de la cola de peticiones.
* **Google Chrome:** Navegador web securizado para la extracción de datos.
* **Microsoft Excel (Microsoft 365):** Origen de criterios y destino consolidado de datos.
* **GMAIL (SMTP):** Servicio de mensajería para alertas y distribución de informes.

---

## 📐 Estructura de Datos (Modelos de Entrada y Salida)

### 📥 Entrada (`idealista_criterio_entrada.xlsx`)
El archivo de configuración debe contar con la siguiente estructura de columnas:
* **Localidad:** *(Texto opcional)* Para acotar áreas específicas.
* **Municipio:** *(Texto obligatorio)* Destino principal de búsqueda.
* **PrecioMin / PrecioMax:** *(Numérico)* Rango de filtrado económico.
* **Habitaciones:** *(Texto/Numérico)* Requisito de estancias (ej. "2", "3", "estudio").

### 📤 Salida (`Reñas_limpio_idea.xlsx`)
El informe consolidado se genera a partir de la celda `A2` con los siguientes campos extraídos:
* **Columna A:** Título / Descripción del Inmueble.
* **Columna B:** Precio.
* **Columna C:** Características principales (m², habitaciones, planta).
* **Columna D:** Ubicación / Dirección.
* **Columna E:** Enlace URL directo al anuncio.

---

## 🛡️ Gestión de Riesgos y Excepciones (Resiliencia)

El flujo incorpora robustos bloques de control de errores (**Try/Catch**) distribuidos en acciones críticas:

### 💼 Excepciones de Negocio (Business Exceptions)
* **BE-02 (Sin resultados):** Si los criterios no arrojan ofertas, el ítem se marca en la cola y el robot salta a la siguiente transacción sin detenerse.
* **BE-03 (Ubicación no normalizada):** Activación de la rutina de limpieza alfanumérica para evitar páginas 404.
* **BE-04 (Límite alcanzado):** Detiene controladamente la iteración al alcanzar el umbral `UserInput`.

### ⚙️ Excepciones de Sistema (System Exceptions)
* **SE-01 (Web caída / Timeout):** Ejecuta hasta un máximo de $N$ reintentos de refresco de página antes de derivar a revisión manual.
* **SE-02 (Bloqueo por Cookies):** Subrutina de inyección de clics automatizada para evadir pop-ups o cortinas de privacidad.
* **SE-03/SE-04 (Errores de E/S o SMTP):** Reintentos automáticos de guardado local y autenticación de correo para blindar la integridad del fichero frente a fallos de red.

---

## 🚨 Sistema de Alarmado e Informes

El robot centraliza todas sus comunicaciones hacia el buzón configurado del proyecto (`@gmail.com`):

1. **[INFORME] - Ejecución completada:** Envío desatendido con éxito que adjunta el archivo Excel con el resumen ejecutivo de inmuebles extraídos.
2. **[ALERTA CRÍTICA] - Error de Sistema:** Notificación inmediata ante excepciones de infraestructura (`SE-0X`). Cierra ordenadamente Chrome y Excel para liberar memoria y adjunta el log técnico de fallos.

---

## 📝 Pruebas de Validación (UAT)
Para certificar el paso del bot a producción, se establecen los siguientes criterios de éxito:
* **Success Rate:** Completar el 100% de las transacciones válidas sin excepciones de sistema durante 3 ejecuciones consecutivas.
* **Cierre Limpio:** Confirmación de que las instancias de procesos quedan completament
