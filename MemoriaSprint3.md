# 1. Componentes de Selección de Fecha y Hora para Itinerarios de Viaje (T-3.1.1 / T-3.2.1)

Para dotar a la aplicación de gestión de itinerarios de viaje (**Lazy Trip**) de una interfaz intuitiva y profesional en Java Swing, se han integrado y adaptado dos librerías externas especializadas en la selección temporal: **JCalendar** para la gestión de fechas y **Java Swing TimePicker** (junto con su dependencia **TimingFramework**) para la selección de horas.

A continuación, se detalla la arquitectura interna, las clases principales y el funcionamiento técnico de cada uno de los componentes integrados en el proyecto.


## 1.1. Componente de Selección de Fecha (JCalendar)

El componente encargado de la selección de fechas proviene del repositorio oficial de Kai Toedter (https://github.com/toedter/jcalendar). Esta librería sigue la especificación JavaBeans, lo que facilita su integración visual y programática en entornos de desarrollo Swing.

<img width="237" height="175" alt="image" src="https://github.com/user-attachments/assets/1f0ce962-663a-4032-8060-b7babbcbfbd1" />


### Arquitectura y Clases Principales

El paquete `com.toedter.calendar` organiza el selector en una jerarquía modular de componentes visuales y lógicos:

* **JDateChooser (`com.toedter.calendar.JDateChooser`):** Es el componente principal utilizado en los formularios del itinerario. Combina un campo de texto editable (`IDateEditor`) con un botón desplegable (`JButton`) que abre un menú emergente (`JPopupMenu`) conteniendo el calendario completo.
* **JCalendar (`com.toedter.calendar.JCalendar`):** Representa el panel completo del calendario, integrando en su interior la selección de día, mes y año, además de botones opcionales de acción rápida (*Hoy* y *Sin fecha*).
* **JDayChooser (`com.toedter.calendar.JDayChooser`):** Gestiona la cuadrícula de días del mes, permitiendo la navegación mediante teclado o ratón, el resaltado de domingos y días festivos, y la validación visual de fechas.
* **JMonthChooser y JYearChooser:** Componentes dedicados a la selección del mes (mediante un `JComboBox` o `JSpinner`) y del año (basado en `JSpinField`), sincronizados automáticamente con `JDayChooser` mediante eventos `PropertyChangeListener`.
* **JTextFieldDateEditor y JSpinnerDateEditor:** Implementaciones de la interfaz `IDateEditor` que gestionan el formato de entrada (`SimpleDateFormat`), las máscaras de texto (`MaskFormatter`) y la validación en tiempo real de la fecha escrita por el usuario.


### Características Aplicadas al Proyecto

* **Validación de Rangos (`DateUtil` y `MinMaxDateEvaluator`):** Permite restringir las fechas seleccionables (`setSelectableDateRange`, `setMinSelectableDate`, `setMaxSelectableDate`), evitando que el usuario programe etapas del viaje en fechas pasadas o incoherentes con el itinerario general.
* **Internacionalización (`JLocaleChooser` y `UTF8ResourceBundle`):** Soporte nativo para múltiples configuraciones regionales (`Locale`), adaptando automáticamente los nombres de los meses, los días de la semana y las etiquetas de los botones al español (`es_ES`).
* **Evaluadores Personalizados (`IDateEvaluator`):** Interfaz que permite marcar visualmente fechas especiales (por ejemplo, días de salida, llegada o festivos en destino) con colores y *tooltips* personalizados.


## 1.2. Componente de Selección de Hora (Swing TimePicker)

Para la elección precisa de las horas de salida, llegada o actividades del itinerario, se ha incorporado el componente desarrollado por DJ-Raven (https://github.com/DJ-Raven/java-swing-timepicker). Este componente ofrece un selector de reloj analógico interactivo de estilo *Material Design*.

<img width="300" height="415" alt="image" src="https://github.com/user-attachments/assets/c1e2fe76-8405-4eb7-bdb2-2d625ff8763a" />


### Arquitectura y Clases Principales

El componente se estructura bajo el paquete `com.raven.swing` y un sistema de eventos en `com.raven.event`:

* **TimePicker (`com.raven.swing.TimePicker`):** Es el contenedor principal del selector. Incluye una cabecera (`panelHeader`) que muestra la hora y los minutos seleccionados, un interruptor para alternar entre formato de 12 horas (AM/PM) y 24 horas (`jCB24hour`), y botones de confirmación (`cmdOK`) y cancelación (`cmdCancel`). Puede vincularse directamente a un `JTextField` (`setDisplayText`) o mostrarse como ventana emergente (`showPopup`).
* **TimeComponent (`com.raven.swing.TimeComponent`):** Es el núcleo gráfico del reloj. Dibuja de forma personalizada mediante `Graphics2D` (con suavizado *antialiasing*) la esfera del reloj, la distribución trigonométrica de las horas y minutos, y la manecilla indicadora. Gestiona la interacción mediante `MouseAdapter` y `MouseMotionAdapter`, permitiendo tanto el clic directo como el arrastre fluido de la manecilla.
* **TimePickerButton, TimePickerLabel y TimePickerMenu:** Componentes Swing personalizados que dotan al reloj de estética moderna, incluyendo efectos visuales de pulsación (*ripple effect*) en los botones y un menú emergente limpio (`JPopupMenu`).


### Gestión de Eventos e Interpolación

* **Sistema de Eventos (`com.raven.event`):**
  * **EventTimeSelected:** Notifica en tiempo real (`hourSelected`, `minuteSelected`) cuando el usuario selecciona una hora o minuto en la esfera, realizando la transición automática de selección de hora a minutos (`changeToMinute`).
  * **EventTimeChange:** Gestiona el cambio de estado entre la vista de horas y la vista de minutos (`timeChange`).
  * **EventTimePicker:** Emite el evento final (`timeSelected`) cuando el usuario confirma la hora elegida.
* **Dependencia de Animación (`TimingFramework-0.55.jar`):** El reloj utiliza la librería `org.jdesktop.animation.timing` (`Animator` y `TimingTargetAdapter`) para interpolar el movimiento de la manecilla del reloj y las ondas de clic en los botones, logrando transiciones suaves y fluidas en la interfaz de usuario.


## 1.3. Resumen de Integración en el Itinerario

| Componente | Paquete Base | Clase Principal | Tipo de Dato Devuelto | Dependencias Externas | Uso en el Proyecto |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Selector de Fecha (JCalendar)** | `com.toedter.calendar` | `JDateChooser` / `JCalendar` | `java.util.Date` / `java.util.Calendar` | Ninguna (Java Swing estándar) | Selección de fechas de inicio, fin y etapas del viaje |
| **Selector de Hora (TimePicker)** | `com.raven.swing` | `TimePicker` / `TimeComponent` | `String` formateado (`HH:mm` / `hh:mm aa`) o `java.util.Date` | `TimingFramework-0.55.jar` | Selección de horarios de vuelos, reservas y actividades |



# 2. Plan de Validación, Calidad y Empaquetado de Componentes Propios (T-3.3.1 / T-3.4.1)

En el marco del *Sprint 3: Modularidad y Componentes Propios* del proyecto **LazyTrip**, el objetivo técnico es desarrollar controles visuales personalizados, encapsulados y reutilizables (`CampoCalendario` y `CampoReloj`) que sustituyan a los componentes estándar de Swing. Para garantizar la fiabilidad del software antes de integrarlo en las pantallas definitivas de la aplicación, se ha adoptado la metodología **TDD (Test-Driven Development o Desarrollo Guiado por Pruebas)** [1].

El enfoque TDD consiste en invertir el ciclo tradicional de programación: antes de escribir el código funcional de cada componente, se redactan las pruebas unitarias que definen su comportamiento esperado (fase *Red*), se implementa el código mínimo necesario para superar dichas pruebas (fase *Green*) y, finalmente, se optimiza y limpia la estructura interna manteniendo todas las verificaciones en verde (fase *Refactor*) [1]. Gracias a esta metodología, bajo el rol de **QA Tester**, se ha construido una estrategia integral de aseguramiento de la calidad apoyada en **JUnit 5 (JUnit Jupiter)** [2] y automatización de interfaz gráfica mediante `java.awt.Robot` [3], culminando con la preparación del entorno para su distribución en `.jar` y documentación técnica Javadoc [4].


## 2.1. Diseño del Plan de Pruebas y Criterios de Validación

Al tratarse de componentes independientes pensados para empaquetarse en una librería reutilizable y emplearse en distintos contextos del formulario de viajes (fecha y hora de salida y de llegada), el diseño de las pruebas bajo TDD se ha estructurado en dos niveles complementarios:

* **Validación de Estado Interno (JUnit 5):** Comprueba que el modelo encapsulado de cada componente inicializa sus valores por defecto en `null`, aplica las reglas de formateo (`dd/MM/yyyy` y `HH:mm`) y responde correctamente ante actualizaciones programáticas o limpieza de datos [2].
* **Validación de Interacción Gráfica (`java.awt.Robot`):** La clase `java.awt.Robot` forma parte del paquete nativo AWT de Java y sirve para generar eventos de entrada del sistema a bajo nivel (movimientos reales del puntero, pulsaciones de botones del ratón y teclas) con el fin de automatizar pruebas sobre interfaces gráficas de usuario [3]. En este plan, se utiliza para simular la interacción humana real sobre Swing y verificar la apertura de los menús emergentes (`JPopupMenu`), la respuesta visual de los botones vectoriales y la manipulación directa de los selectores internos (`JCalendar` y `TimePicker`).

Los objetivos concretos de verificación se agrupan en cuatro pilares:

1. **Integridad de inicialización:** Garantizar que, al instanciar `CampoCalendario` o `CampoReloj`, sus ventanas emergentes permanecen ocultas y sus atributos internos son `null`.
2. **Sincronización Vista-Modelo:** Verificar que cualquier selección realizada en el calendario o en la esfera analógica se refleja de inmediato en el campo de texto izquierdo (`txtFecha` o `txtHora`) con el formato esperado.
3. **Interacción analógica y discreta en el reloj:** Validar tanto la selección de horas y minutos en múltiplos de 5 como el arrastre continuo de la aguja sobre la esfera en modo 24 horas.
4. **Encapsulación y aislamiento:** Confirmar que ambos controles funcionan de forma autónoma sin dependencias externas de base de datos ni acoplamiento a otras vistas.


## 2.2. Ejecución de Pruebas Unitarias y de Integración

Para automatizar la ejecución en NetBeans (entorno Ant), se integró el motor `junit-platform-console-standalone-1.10.2.jar` en el *Test Classpath* del proyecto y se configuró el objetivo `-do-test-run` en `build.xml` para lanzar `ConsoleLauncher` y generar los informes XML en `build/test/results/` [2].

La batería funcional se implementó en el paquete `main.test` mediante las suites `CalendarioEstadoTest` y `RelojEstadoTest`. En ellas se combinan aserciones de JUnit 5 con el control de eventos nativos de `java.awt.Robot`, permitiendo calcular las coordenadas exactas en pantalla de cada componente, desplazar el cursor automáticamente y ejecutar clics o arrastres reales sin intervención manual [3].


### Matriz de Pruebas Funcionales de CampoCalendario (`CalendarioEstadoTest`)

| ID | Escenario | Acción y Datos de Entrada | Resultado Esperado y Verificado |
| :--- | :--- | :--- | :--- |
| **CA-01** | Inicialización por defecto | Instanciar `CampoCalendario` sin interacción previa. | `getFecha()` devuelve `null`, `getTextoMostrado()` está vacío (`""`) y el `JCalendar` permanece oculto. |
| **CA-02** | Apertura del calendario | `Robot` desplaza el cursor al icono derecho y ejecuta clic izquierdo (`BUTTON1_DOWN_MASK`). | Se despliega el `JPopupMenu` y `JCalendar.isShowing()` pasa a `true`. |
| **CA-03** | Asignación y formateo | Asignar la fecha `2026-11-25` mediante `setFecha()`. | El estado interno almacena `LocalDate.of(2026, 11, 25)` y la barra muestra `"25/11/2026"`. |
| **CA-04** | Limpieza de estado | Invocar `setFecha(null)` tras haber seleccionado una fecha previa. | El atributo interno vuelve a `null` y el campo de texto queda vacío (`""`). |


### Matriz de Pruebas Funcionales de CampoReloj (`RelojEstadoTest`)

| ID | Escenario | Acción y Datos de Entrada | Resultado Esperado y Verificado |
| :--- | :--- | :--- | :--- |
| **CB-01** | Inicialización por defecto | Instanciar `CampoReloj` sin interacción previa. | `getHora()` devuelve `null`, `getTextoMostrado()` está vacío (`""`) y `TimePicker.isShowing()` es `false`. |
| **CB-02** | Apertura del reloj analógico | `Robot` pulsa sobre el botón vectorial derecho del reloj. | Se despliega el popup emergente mostrando el selector analógico `TimePicker` en modo 24 horas. |
| **CB-03** | Selección en múltiplos de 5 | Establecer horas con minutos discretos (`14:05`, `14:15`, `14:30`). | El componente sincroniza el `TimePicker` y formatea la salida en `"14:05"`, `"14:15"` y `"14:30"`. |
| **CB-04** | Arrastre en esfera y volcado | `Robot` abre el selector, realiza un arrastre sobre la esfera y confirma minuto exacto (`14:18`). | La aguja responde al desplazamiento, `getHora().getMinute()` registra `18` y el campo muestra `"14:18"`. |

La ejecución completa de ambas suites concluyó con **8 de 8 pruebas superadas (`BUILD SUCCESSFUL`)**, validando tanto la lógica interna como el comportamiento gráfico en pantalla.


## 2.3. Pruebas de Seguridad Básica y Uso de Recursos

Con el fin de asegurar que la integración de los componentes en pantallas con múltiples formularios no degrade el rendimiento ni introduzca vectores de fallo por manipulación de entrada, se desarrolló la suite `SeguridadyMemoriaTest` bajo JUnit 5:

<img width="712" height="640" alt="image" src="https://github.com/user-attachments/assets/8bbe24a4-13f0-49f8-8bda-9631a32c1083" />


| ID | Área Evaluada | Procedimiento de Prueba | Criterio de Aceptación Verificado |
| :--- | :--- | :--- | :--- |
| **SM-01** | Consumo de memoria y fugas | Instanciar y descartar en bucle 200 objetos `CampoCalendario` y 200 `CampoReloj` (400 controles en total), invocar `System.gc()` y medir el delta de memoria heap mediante `Runtime`. | El incremento neto de memoria residual se mantiene por debajo del umbral definido (`<64 MB`) y el tiempo total de creación es inferior a `3 s`. |
| **SM-02** | Prevención de inyección de entrada | Inspeccionar los componentes `JTextField` internos (`txtFecha` y `txtHora`) de ambos controles. | Ambos campos tienen `isEditable() == false`, bloqueando la introducción libre por teclado de cadenas malformadas o intentos de inyección. |
| **SM-03** | Robustez ante nulos y límites | Probar `setFecha(null)`, `setHora(null)`, años bisiestos (`29/02/2028`) y horas límite (`00:00` y `23:59`). | Ninguna llamada lanza `NullPointerException` ni errores de parseo; los límites se formatean con exactitud. |

Con la incorporación de `SeguridadyMemoriaTest`, la batería global alcanzó **11 de 11 casos ejecutados con éxito**, acreditando la estabilidad de recursos y la protección frente a entradas no válidas.


## 2.4. Configuración del Entorno para Exportación a `.jar` y Javadoc (Tarea #43 / T-3.4.1)

Finalmente, se dejó configurado el entorno de desarrollo en NetBeans y el script de construcción Ant para facilitar la distribución de los componentes como librería independiente y su documentación técnica asociada [4], [5]:

* **Empaquetado en archivo `.jar`:** En las propiedades del proyecto (*Build → Packaging*), se activaron las opciones *Build JAR after Compiling* y *Copy Dependent Libraries*. Al ejecutar *Clean and Build* (`clean jar`), Ant genera automáticamente el artefacto `dist/LazyTripComponentes.jar` junto con las dependencias necesarias (`jcalendar-1.4.jar`, `swing-time-picker.jar` y `TimingFramework-0.55.jar`) en `dist/lib/`, permitiendo reutilizar los componentes en la aplicación principal de LazyTrip [5].
* **Generación de documentación técnica (Javadoc):** Se ha documentado el código fuente de `CampoCalendario` y `CampoReloj` con comentarios Javadoc (`/** ... */`) y etiquetas estándar (`@author`, `@version`, `@since`, `@param`, `@return`, `@throws`, `@deprecated`, `@see`) [6]. En *Build → Documenting*, se habilitaron las etiquetas `@author` y `@version`, y se configuraron los parámetros adicionales `-encoding UTF-8 -charset UTF-8 -docencoding UTF-8` para garantizar la correcta codificación de caracteres en español. Mediante la acción *Generate Javadoc*, el entorno genera en `dist/javadoc/` la documentación HTML completa de todas las clases, constructores y métodos públicos del paquete [4].



# 3. Manual Técnico, Guía de Instalación y Despliegue (Rol Release Manager - HU-3.4 / T-3.4.1 / T-3.5.1)

Bajo la responsabilidad del rol de **Release Manager**, se establece la documentación técnica, el procedimiento de instalación de la librería, la selección de herramientas para empaquetado comercial e instaladores y la aplicación de firmas digitales al software.


## 3.1. Manual Técnico de los Componentes LazyTrip

### Resumen de Componentes Desarrollados

| Componente | Función | Dónde se usa en LazyTrip |
| :--- | :--- | :--- |
| **`CampoCalendario`** | Seleccionar una fecha en un calendario emergente. | Fecha de llegada y fecha de salida del itinerario. |
| **`CampoReloj`** | Seleccionar una hora en un reloj analógico emergente. | Hora de llegada y hora de salida del itinerario. |


### Tecnologías y Herramientas Utilizadas

| Herramienta | Uso en el Proyecto |
| :--- | :--- |
| **Java SE (JDK 25)** | Lenguaje y plataforma de desarrollo. Incluye las herramientas `javac`, `jar` y `javadoc`. |
| **Swing** | Librería gráfica de la que heredan los componentes (`JComponent`, `JPanel`). |
| **`java.time` (`LocalDate`, `LocalTime`)** | Tipos de datos modernos para gestionar la fecha y la hora seleccionadas. |
| **JCalendar 1.4 (`com.toedter.calendar`)** | Librería externa que aporta el calendario emergente. |
| **TimePicker (`com.raven.swing`)** | Librería externa (`swing-time-picker.jar`) que aporta el reloj analógico emergente. |
| **Timing Framework 0.55** | Librería (`TimingFramework-0.55.jar`) de animaciones que necesita `TimePicker`. |
| **Apache NetBeans** | Entorno de desarrollo (IDE) y editor visual de interfaces. |
| **Apache Ant** | Sistema de construcción que utiliza NetBeans en los proyectos *Java with Ant*. |
| **JUnit 5** | Framework de pruebas unitarias para validar los componentes de forma aislada. |
| **Javadoc** | Herramienta del JDK que genera la documentación HTML a partir de los comentarios del código. |
| **Herramienta `jar`** | Herramienta del JDK que empaqueta las clases compiladas y el manifiesto (`META-INF/MANIFEST.MF`) en un único fichero `.jar` [7]. |


### Herencia y Requisitos de JavaBean

Ambos controles extienden de `javax.swing.JPanel` y cumplen las especificaciones estándar de JavaBeans para ser integrados en la paleta visual del IDE:

| Requisito JavaBean | Cómo se cumple en el proyecto |
| :--- | :--- |
| **Clase pública** | `public class CampoCalendario` / `public class CampoReloj` |
| **Constructor público sin parámetros** | `public CampoCalendario()` / `public CampoReloj()` |
| **Propiedades privadas con getters/setters** | Atributos encapsulados `fecha` (`getFecha` / `setFecha`) y `hora` (`getHora` / `setHora`) |
| **Serialización** | Implementa `Serializable` por herencia directa de `JPanel` |


### Estructura de Documentación Javadoc

Todo el código público está documentado mediante bloques Javadoc (`/** ... */`) empleando las etiquetas principales estándar [6]:

* **`@author`:** Indica el autor o autores de la clase.
* **`@version`:** Especifica la versión actual de la clase o API.
* **`@since`:** Indica desde qué versión está disponible la clase o método.
* **`@param`:** Describe cada parámetro que recibe un método (nombre y función).
* **`@return`:** Explica el valor que devuelve un método.
* **`@throws` / `@exception`:** Documenta las excepciones que un método puede lanzar.
* **`@deprecated`:** Señala que un método o clase ya no se recomienda y debe dejar de usarse, indicando alternativas.
* **`@see`:** Añade un enlace o referencia a otra clase, método o página relacionada.


### Estructura del Empaquetado y Dependencias

El componente se distribuye como un fichero `.jar`, un formato basado en ZIP que reúne las clases compiladas y el fichero de manifiesto `META-INF/MANIFEST.MF` con los metadatos del paquete [7]. Dado que las librerías de terceros no se incrustan dentro del `.jar` principal, deben acompañarlo en la carpeta `lib/`:

| Fichero / Directorio | Ubicación | Contenido |
| :--- | :--- | :--- |
| `LazyTripComponentes.jar` | `dist/` | Clases compiladas de `CampoCalendario` y `CampoReloj`. |
| `jcalendar-1.4.jar` | `dist/lib/` | Librería externa JCalendar. |
| `swing-time-picker.jar` | `dist/lib/` | Librería externa TimePicker. |
| `TimingFramework-0.55.jar` | `dist/lib/` | Librería de animaciones requerida por TimePicker. |
| Documentación HTML | `dist/javadoc/` | API completa del componente generada con Javadoc (`index.html`). |


## 3.2. Guía de Instalación y Generación de la Librería

### Requisitos Previos

| Elemento | Requisito |
| :--- | :--- |
| **JDK** | Java Development Kit (JDK 25). |
| **IDE** | Apache NetBeans (configurado con proyectos *Java with Ant*). |
| **Librerías externas** | `jcalendar-1.4.jar`, `swing-time-picker.jar` y `TimingFramework-0.55.jar`. |


### Paso 1: Preparar el Proyecto de la Librería

1. En NetBeans, acceder a **File → New Project → Java with Ant → Java Class Library** y asignarle el nombre `LazyTripComponentes`.
2. Dentro de *Source Packages*, crear el paquete `com.lazytrip.componentes` y alojar en él las clases `CampoCalendario.java` y `CampoReloj.java`.
3. Crear una carpeta `lib/` en la raíz del proyecto y copiar dentro los archivos `jcalendar-1.4.jar`, `swing-time-picker.jar` y `TimingFramework-0.55.jar`.
4. Hacer clic derecho sobre **Libraries → Add JAR/Folder** y seleccionar los tres archivos `.jar` anteriores.


### Paso 2: Compilar y Generar el Fichero `.jar`

1. Hacer clic derecho sobre el proyecto `LazyTripComponentes` y seleccionar **Clean and Build** (atajo `Mayús + F11`).
2. NetBeans ejecutará Ant y generará el fichero compilado en `LazyTripComponentes/dist/LazyTripComponentes.jar`, copiando automáticamente las dependencias a `dist/lib/`.


### Paso 3: Generar la Documentación Javadoc

1. Hacer clic derecho sobre el proyecto `LazyTripComponentes` y seleccionar **Generate Javadoc**.
2. La documentación HTML se creará en `dist/javadoc/`. Para visualizarla, abrir el archivo `index.html` en cualquier navegador web.


### Paso 4: Integración en el Proyecto Principal (T-3.5.1)

1. En el proyecto principal de **LazyTrip**, hacer clic derecho sobre **Libraries → Add JAR/Folder** e importar `LazyTripComponentes.jar` junto con los tres `.jar` de la carpeta `dist/lib/`.
2. Arrastrar o instanciar `CampoCalendario` y `CampoReloj` dentro de las vistas del formulario de gestión de itinerarios.


## 3.3. Generación de Instaladores Profesionales

Para empaquetar y distribuir la aplicación comercial resultante en entornos Windows mediante asistentes de instalación profesionales, se han evaluado dos herramientas principales:

* **Inno Setup (Opción Seleccionada):** Es el asistente de instalación gratuito más popular para Windows. Utiliza archivos de script sencillos (`.iss`), permite un alto grado de personalización visual de la interfaz del instalador y empaqueta toda la aplicación y sus dependencias en un único archivo ejecutable (`setup.exe`).
* **NSIS (Nullsoft Scriptable Install System):** Sistema de código abierto muy ligero y flexible, ideal para generar instaladores de tamaño mínimo cuando se requiere un control exhaustivo mediante scripts detallados.


## 3.4. Aplicación de Firmas Digitales al Software

La **firma digital** es un mecanismo criptográfico que actúa como equivalente electrónico avanzado de una firma manuscrita, garantizando la autenticidad del autor y la integridad del ejecutable frente a manipulaciones.


### Obtención del Certificado Digital

* **Ámbito institucional / nacional (España):** Puede solicitarse a través de la **Fábrica Nacional de Moneda y Timbre (FNMT)** o mediante el sistema **Cl@ve**.
* **Distribución comercial de software (Code Signing):** Para evitar advertencias de seguridad (*Windows SmartScreen*), se recomienda adquirir un certificado de firma de código emitido por una Autoridad de Certificación (CA) de confianza como **DigiCert**, o emplear servicios integrados en la nube como **Azure Trusted Signing**.


### Procedimiento de Firma con `SignTool.exe`

1. **Solicitar el certificado:** Obtener el archivo de certificado `.pfx` (o configurar el almacén en Azure Trusted Signing) a través de una Autoridad de Certificación de confianza.
2. **Instalar el Windows SDK:** Descargar e instalar el SDK de Windows para disponer de la herramienta oficial de línea de comandos `SignTool.exe`.
3. **Firmar el ejecutable:** Abrir la consola de comandos de Windows (CMD o PowerShell) y ejecutar el siguiente comando sustituyendo las rutas y la contraseña correspondientes:

```cmd
signtool.exe sign /f "ruta/de/tu-certificado.pfx" /p "tu-contraseña" /fd SHA256 /tr "[http://timestamp.digicert.com](http://timestamp.digicert.com)" /td SHA256 "ruta/de/tu-programa.exe"

```
# 4. Bibliografía

* **[1]** K. Beck, *Test-Driven Development: By Example*. Boston, MA, USA: Addison-Wesley Professional, 2002. [En línea]. Disponible en: https://www.oreilly.com/library/view/test-driven-development/0321146530/


* **[2]** JUnit Team, *JUnit 5 User Guide (Version 5.10.2)*, 2024. [En línea]. Disponible en: https://junit.org/junit5/docs/current/user-guide/


* **[3]** Oracle, *Java Platform, Standard Edition 17 API Specification: Class Robot (java.awt.Robot)*, 2024. [En línea]. Disponible en: https://docs.oracle.com/en/java/javase/17/docs/api/java.desktop/java/awt/Robot.html


* **[4]** Oracle, *The Javadoc Tool Documentation & Technical Guide*, 2024. [En línea]. Disponible en: https://docs.oracle.com/en/java/javase/17/javadoc/javadoc.html


* **[5]** The Apache Software Foundation, *Apache Ant Manual: Jar Task & Build Configuration*, 2024. [En línea]. Disponible en: https://ant.apache.org/manual/Tasks/jar.html


* **[6]** Aprender a Programar, *Documentar proyectos Java con Javadoc: comentarios, símbolos y tags (`@deprecated`, `@param`, etc.)*, 2024. [En línea]. Disponible en: https://www.aprenderaprogramar.com/index.php?option=com_content&view=article&id=646:documentar-proyectos-java-con-javadoc-comentarios-simbolos-tags-deprecated-param-etc-cu00680b&catid=68&Itemid=188


* **[7]** Oracle, *The Java Tutorials: Packaging Programs in JAR Files*, 2024. [En línea]. Disponible en: https://docs.oracle.com/javase/tutorial/deployment/jar/


* **[8]** Alura Cursos, *Duda sobre exportación de fichero `.jar` en NetBeans*, Foro de la comunidad. [En línea]. Disponible en: https://app.aluracursos.com/forum/topico-duda-sobre-jar-netbeans-219312

