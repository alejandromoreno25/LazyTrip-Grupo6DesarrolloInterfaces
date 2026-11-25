# Componentes de Selección de Fecha y Hora para Itinerarios de Viaje

Para dotar a la aplicación de gestión de itinerarios de viaje de una interfaz intuitiva y profesional en Java Swing, se han integrado y adaptado dos librerías externas especializadas en la selección temporal: **JCalendar** para la gestión de fechas y **Java Swing TimePicker** (junto con su dependencia **TimingFramework**) para la selección de horas.

A continuación, se detalla la arquitectura interna, las clases principales y el funcionamiento técnico de cada uno de los componentes integrados en el proyecto.

---

## 1. Componente de Selección de Fecha (`JCalendar`)

El componente encargado de la selección de fechas proviene del repositorio oficial de Kai Toedter (`https://github.com/toedter/jcalendar`). Esta librería sigue la especificación **JavaBeans**, lo que facilita su integración visual y programática en entornos de desarrollo Swing[cite: 1].

### Estructura y Clases Principales

El paquete `com.toedter.calendar` organiza el selector en una jerarquía modular de componentes visuales y lógicos[cite: 1]:

* **`JDateChooser` (`com.toedter.calendar.JDateChooser`):** Es el componente principal utilizado en los formularios del itinerario. Combina un campo de texto editable (`IDateEditor`) con un botón desplegable (`JButton`) que abre un menú emergente (`JPopupMenu`) conteniendo el calendario completo[cite: 1].
* **`JCalendar` (`com.toedter.calendar.JCalendar`):** Representa el panel completo del calendario, integrando en su interior la selección de día, mes y año, además de botones opcionales de acción rápida (*Hoy* y *Sin fecha*)[cite: 1].
* **`JDayChooser` (`com.toedter.calendar.JDayChooser`):** Gestiona la cuadrícula de días del mes, permitiendo la navegación mediante teclado o ratón, el resaltado de domingos y días festivos, y la validación visual de fechas[cite: 1].
* **`JMonthChooser` y `JYearChooser`:** Componentes dedicados a la selección del mes (mediante un `JComboBox` o `JSpinner`) y del año (basado en `JSpinField`), sincronizados automáticamente con `JDayChooser` mediante eventos `PropertyChangeListener`[cite: 1].
* **`JTextFieldDateEditor` y `JSpinnerDateEditor`:** Implementaciones de la interfaz `IDateEditor` que gestionan el formato de entrada (`SimpleDateFormat`), las máscaras de texto (`MaskFormatter`) y la validación en tiempo real de la fecha escrita por el usuario[cite: 1].

### Características Aplicadas al Proyecto

* **Validación de Rangos (`DateUtil` y `MinMaxDateEvaluator`):** Permite restringir las fechas seleccionables (`setSelectableDateRange`, `setMinSelectableDate`, `setMaxSelectableDate`), evitando que el usuario programe etapas del viaje en fechas pasadas o incoherentes con el itinerario general[cite: 1].
* **Internacionalización (`JLocaleChooser` y `UTF8ResourceBundle`):** Soporte nativo para múltiples configuraciones regionales (`Locale`), adaptando automáticamente los nombres de los meses, los días de la semana y las etiquetas de los botones al español (`es_ES`)[cite: 1].
* **Evaluadores Personalizados (`IDateEvaluator`):** Interfaz que permite marcar visualmente fechas especiales (por ejemplo, días de salida, llegada o festivos en destino) con colores y *tooltips* personalizados[cite: 1].

---

## 2. Componente de Selección de Hora (`Swing TimePicker`)

Para la elección precisa de las horas de salida, llegada o actividades del itinerario, se ha incorporado el componente desarrollado por DJ-Raven (`https://github.com/DJ-Raven/java-swing-timepicker`). Este componente ofrece un selector de reloj analógico interactivo de estilo *Material Design*[cite: 2].

### Arquitectura y Clases Principales

El componente se estructura bajo el paquete `com.raven.swing` y un sistema de eventos en `com.raven.event`[cite: 2]:

* **`TimePicker` (`com.raven.swing.TimePicker`):** Es el contenedor principal del selector. Incluye una cabecera (`panelHeader`) que muestra la hora y los minutos seleccionados, un interruptor para alternar entre formato de **12 horas (AM/PM)** y **24 horas** (`jCB24hour`), y botones de confirmación (`cmdOK`) y cancelación (`cmdCancel`). Puede vincularse directamente a un `JTextField` (`setDisplayText`) o mostrarse como ventana emergente (`showPopup`)[cite: 2].
* **`TimeComponent` (`com.raven.swing.TimeComponent`):** Es el núcleo gráfico del reloj. Dibuja de forma personalizada mediante `Graphics2D` (con suavizado *antialiasing*) la esfera del reloj, la distribución trigonométrica de las horas y minutos, y la manecilla indicadora. Gestiona la interacción mediante `MouseAdapter` y `MouseMotionAdapter`, permitiendo tanto el clic directo como el arrastre fluido de la manecilla[cite: 2].
* **`TimePickerButton`, `TimePickerLabel` y `TimePickerMenu`:** Componentes Swing personalizados que dotan al reloj de estética moderna, incluyendo efectos visuales de pulsación (*ripple effect*) en los botones y un menú emergente limpio (`JPopupMenu`)[cite: 2].

### Gestión de Eventos e Interpolación

* **Sistema de Eventos (`com.raven.event`):**
  * `EventTimeSelected`: Notifica en tiempo real (`hourSelected`, `minuteSelected`) cuando el usuario selecciona una hora o minuto en la esfera, realizando la transición automática de selección de hora a minutos (`changeToMinute`)[cite: 2].
  * `EventTimeChange`: Gestiona el cambio de estado entre la vista de horas y la vista de minutos (`timeChange`)[cite: 2].
  * `EventTimePicker`: Emite el evento final (`timeSelected`) cuando el usuario confirma la hora elegida[cite: 2].
* **Dependencia de Animación (`TimingFramework-0.55.jar`):** El reloj utiliza la librería `org.jdesktop.animation.timing` (`Animator` y `TimingTargetAdapter`) para interpolar el movimiento de la manecilla del reloj y las ondas de clic en los botones, logrando transiciones suaves y fluidas en la interfaz de usuario[cite: 2, 3].

---

## 3. Resumen de Integración en el Itinerario

| Característica | Selector de Fecha (`JCalendar`) | Selector de Hora (`TimePicker`) |
| :--- | :--- | :--- |
| **Paquete base** | `com.toedter.calendar`[cite: 1] | `com.raven.swing`[cite: 2] |
| **Clase principal** | `JDateChooser` / `JCalendar`[cite: 1] | `TimePicker` / `TimeComponent`[cite: 2] |
| **Tipo de dato devuelto** | `java.util.Date` / `java.util.Calendar`[cite: 1] | `String` formateado (`HH:mm` / `hh:mm aa`) o `java.util.Date`[cite: 2] |
| **Dependencias externas** | Ninguna (Java Swing estándar)[cite: 1] | `TimingFramework-0.55.jar`[cite: 2, 3] |
| **Uso en el proyecto** | Selección de fechas de inicio, fin y etapas del viaje | Selección de horarios de vuelos, reservas y actividades |


Se ha desarrollado un índice para llevar a cabo el plan de validación y pruebas:

## Estructura del Plan de Pruebas

1. **Introducción y Estrategia (Enfoque TDD)**
   * Explicación breve de por qué se diseñan las pruebas antes de tener el código final, basándose en las maquetas y los requisitos de las 8 pantallas de LazyTrip.
2. **Herramientas de Automatización**
   * **JUnit 5:** Qué es y cómo se usará para comprobar los cambios de estado internos (variables en memoria, sesión de usuario, validaciones de fechas y presupuesto).
   * **TestFX (o AssertJ-Swing):** Qué es y cómo se usará para simular acciones reales del usuario en pantalla (clics, escritura, movimiento de barras) y comprobar los elementos gráficos.
3. **Tabla de Casos de Prueba (Diseño Funcional y Gráfico)**
   * Lista estructurada (ID, Pantalla, Acción del usuario, Datos de entrada y Resultado esperado) cubriendo los puntos clave:
     * Bloqueo de *"Mis viajes"* sin iniciar sesión.
     * Validación de fechas (ida y vuelta) y número de viajeros.
     * Actualización en tiempo real del presupuesto (*slider*) y selección de hotel.
     * Separación de actividades y fotos al cambiar entre los botones de días (`DÍA 1`, `DÍA 2`...).
     * Opciones de guardado según si el usuario está registrado (Perfil) o no (PDF).
4. **Scripts Automatizados (Código de Prueba)**
   * **4.1. Script de Estado Interno (JUnit):** Código Java con métodos `@Test` y aserciones (`assertEquals`, `assertTrue`, `assertFalse`) para validar la lógica interna.
   * **4.2. Script de Interfaz Gráfica (TestFX):** Código Java simulando la interacción con botones, campos de texto y contenedores visuales (`clickOn`, `write`, `verifyThat`).
5. **Criterios de Validación y Estado de Ejecución**
   * Condiciones mínimas para dar por superadas las pruebas (ej. 100 % de éxito en validaciones críticas) y una pequeña tabla de control indicando su estado actual (Pendiente de integración con el código fuente).

---

## 1. Introducción a los Casos de Prueba

Para comenzar con el plan de validación, se ha realizado una investigación acerca de la metodología más adecuada para esta parte. Por ello, se seguirá la metodología **TDD** [1], estrategia que consiste en redactar los casos de prueba y definir los criterios de sus resultados antes de implementar el código fuente [1].

## 2. Herramientas de Automatización

Para ejecutar la estrategia de pruebas desde **LazyTrip**, se basarán en la combinación de las dos siguientes herramientas para la validación tanto de la lógica interna como de la respuesta de la interfaz:

* **JUnit 5:** Es el estándar de referencia para el desarrollo y ejecución de pruebas unitarias en Java. En este proyecto se utilizará para verificar los cambios de estado internos del sistema y sus métodos, asegurando que las reglas funcionen correctamente. Esto se comprobará mediante aserciones automáticas en las que se validará la lógica de los componentes de la aplicación [2].
* **TestFX (o AssertJ-Swing):** Son librerías especializadas en la automatización de pruebas funcionales sobre interfaces gráficas en Java. Estas librerías se emplearán para simular el comportamiento real de un usuario frente a las 8 pantallas. Posteriormente, la herramienta verificará de forma automática que los elementos gráficos reaccionan según lo diseñado (apertura de diálogos modales, actualización de etiquetas en pantalla o visibilidad de fotografías según el día seleccionado) [3][4].

# Documentación de los Casos de Prueba Funcionales para el Flujo de Planificación y Gestión de Viajes

| ID | Pantalla | Escenario y Precondición | Acción y Datos de Entrada | Resultado Esperado (UI y Estado Interno) |
| :--- | :--- | :--- | :--- | :--- |
| **CP-01** | P1: Inicio y Acceso | Acceder al historial sin estar con la sesión iniciada. | Pulsar el botón **Mis viajes** en la pantalla principal. | Se bloquea la navegación hacia P6 y se despliega la ventana modal `LoginDialog` solicitando credenciales. |
| **CP-02** | P1: Inicio y Acceso | Inicio de flujo de planificación de itinerario. | Pulsar el contenedor/botón principal **Nuevo viaje**. | Se instancia un nuevo objeto limpio en memoria y la vista transiciona a P2. |
| **CP-03** | P2: Destino y Fechas | No validación de la fecha (fecha de vuelta anterior a la de ida). | Destino: `"Madrid"`<br>Llegada: `20/07/2026`<br>Salida: `15/07/2026`<br>Pulsar **Continuar**. | El controlador rechaza la transición, mantiene P2 activa y muestra un mensaje visual de error en el selector de fechas. |
| **CP-04** | P2: Destino y Fechas | Validación de límite en el número de viajeros (rango permitido: 1 a 10). | Introducir `0` o `12` en el selector de viajeros y pulsar **Continuar**. | Impide el avance a P3 y notifica que el número de viajeros debe situarse entre 1 y 10 personas. |
| **CP-05** | P3: Presupuesto y Estilo | Actualización reactiva en tiempo real del presupuesto por persona. | Desplazar la barra deslizante hasta el valor elegido (ej. `600`). | La variable interna se actualiza a `600` y la etiqueta superior muestra `"600 €"` al instante. |
| **CP-06** | P3: Presupuesto y Estilo | Selección de categoría temática para filtrar sugerencias. | Pulsar el botón/etiqueta elegido. | El componente cambia su estilo visual a estado activo/seleccionado y guarda el filtro en `ViajeDTO`. |
| **CP-07** | P4: Alojamiento | Sincronización entre listado izquierdo y panel derecho de detalle. | Hacer clic sobre el primer hotel disponible en la lista izquierda. | El panel derecho se refresca con el nombre, valoración y enlace externo de reserva del hotel seleccionado. |
| **CP-08** | P4: Alojamiento | Retorno rápido para modificar parámetros iniciales sin perder estado. | Pulsar el botón **Cambiar destino y fechas**. | La aplicación regresa directamente a P2 manteniendo precargados los datos de ciudad, fechas y viajeros anteriores. |
| **CP-09** | P5: Itinerario | Incorporación de actividades sugeridas y ordenación cronológica. | Pulsar **Añadir** en una actividad de las `12:00` y otra de las `09:30`. | Ambas actividades se insertan en la agenda central ordenadas por hora (`09:30` primero) y se dibujan sus marcadores en el mapa. |
| **CP-10** | P5: Itinerario | Pestañas por día y aislamiento de agendas. | Con actividades en **DÍA 1**, pulsar el botón selector **DÍA 2**. | La vista central se limpia para mostrar la agenda del Día 2 sin borrar ni mezclar las actividades del Día 1. |
| **CP-11** | P5: Itinerario | Seleccionar la opción de descargar PDF y la de guardar en tu historial. | Evaluar barra inferior con `isAutenticado = false` y luego con `isAutenticado = true`. | Como invitado solo habilita **Guardar en PDF**; como usuario registrado habilita **Guardar en perfil** (persistiendo en P6). |
| **CP-12** | P6: Mis Viajes | Acceso a un viaje guardado en mis viajes. | Hacer clic sobre la tarjeta de un viaje almacenado en la lista. | Se carga el objeto completo del viaje seleccionado y la aplicación navega hacia la vista de resumen en P7. |
| **CP-13** | P7: Resumen del Viaje | Resumen por días y navegación hacia galería multimedia. | Comprobar tarjetas informativas y pulsar el botón **Fotos** del menú. | Las etiquetas muestran fechas, hotel y presupuesto exactos del viaje y se abre la Pantalla 8 (P8). |
| **CP-14** | P8: Subida de Fotos | Aislamiento de galería fotográfica entre diferentes jornadas del viaje. | Cargar 2 imágenes en **DÍA 1** y posteriormente pulsar el botón **DÍA 2**. | Las miniaturas del Día 1 dejan de renderizarse en pantalla, mostrando únicamente las del Día 2 hasta volver a pulsar Día 1. |

---

# Plan de Validación y Pruebas de Componentes Propios

## 1. Introducción y Propósito del Diseño de Pruebas

En el marco del **Sprint 3: Modularidad y Componentes Propios** del proyecto **LazyTrip**, el objetivo técnico principal del equipo es desarrollar controles visuales personalizados, encapsulados y reutilizables que sustituyan a los componentes estándar de la paleta de Java. Para garantizar la calidad del software antes de integrar estas piezas en las pantallas definitivas de la aplicación, se ha diseñado una batería de pruebas siguiendo la metodología **TDD (*Test-Driven Development*)** y los criterios de calidad del rol de *QA Tester*.

### ¿Para qué sirve este plan y por qué lo hemos estructurado así?

1. **Aislamiento y Encapsulación (Pruebas Unitarias antes que de Sistema):**
   Al tratarse de componentes independientes (diseñados para empaquetarse en un fichero `.jar` reutilizable), no deben depender de una base de datos ni del resto de pantallas de LazyTrip para funcionar. Por ello, se han dividido las pruebas en niveles complementarios:
   * **Validación de Estado Interno (JUnit):** Comprueba que el "cerebro" del componente inicializa sus valores por defecto y aplica las reglas de negocio sin fallos.
2. **Reutilización en Múltiples Contextos (Ida y Vuelta del Viaje):**
   Tanto el selector de fechas como el reloj analógico se utilizarán en distintas casillas del formulario (fecha/hora de salida y fecha/hora de llegada). Validar cada componente de forma aislada garantiza que, al instanciarlos varias veces en la misma pantalla, cada control mantendrá su propio estado de forma íntegra.
3. **Independencia de Implementación (Enfoque Agnóstico):**
   El diseño de los casos de prueba se centra en el comportamiento observable y en los tipos de datos encapsulados, evitando acoplarse a nombres internos de variables privadas. Esto permite ejecutar las pruebas de forma inmediata en cuanto los componentes estén codificados y detectar errores tempranos antes de la integración final.

---

## 2. ¿Qué queremos probar exactamente?

Los objetivos concretos de verificación se agrupan en cuatro pilares funcionales:

* **Integridad de inicialización:** Asegurar que, al instanciar cualquiera de los dos controles, sus contenedores emergentes (*popups*) permanecen cerrados y sus valores por defecto son limpios y coherentes.
* **Sincronización Vista-Modelo:** Verificar que toda selección realizada gráficamente en el calendario desplegable o en la esfera del reloj se traduce de inmediato en el formato de texto esperado en la barra principal (`dd/MM/yyyy` o `HH:mm`) y en sus atributos internos.
* **Prevención de errores de usuario:** Comprobar que el calendario bloquea o corrige rangos temporales incoherentes (como seleccionar una fecha de fin anterior a la fecha de inicio).
* **Dualidad de interacción en el reloj:** Validar tanto la conmutación superior entre modos (`Horas` / `Minutos`) como los dos mecanismos de selección de minutos: selección rápida discreta (saltos de 5 en 5 mediante clic en marcas principales) y selección continua de precisión (arrastre libre de la aguja para minutos intermedios).

---

## 3. Matriz de Casos de Prueba por Componente

### 3.1. Componente A: Selector de Rango de Fechas

**Descripción funcional:** Barra de texto izquierda acompañada de un icono de calendario a la derecha que despliega una cuadrícula emergente para seleccionar el inicio y fin del viaje.

| ID | Escenario | Acción y Datos de Entrada | Resultado Esperado (Estado Interno y Visual) |
| :--- | :--- | :--- | :--- |
| **CA-01** | Inicialización por defecto | Instanciar el componente sin interacción previa. | La barra de texto izquierda aparece vacía (o con *placeholder*), el desplegable está oculto y las fechas internas son `null`. |
| **CA-02** | Apertura del calendario | Hacer clic en el icono de calendario situado a la derecha de la barra. | Se despliega el contenedor emergente con la cuadrícula de días del mes. |
| **CA-03** | Selección de rango válido | Clic en fecha de inicio (`10/08/2026`) y fecha de fin (`15/08/2026`). | La línea de texto izquierda se rellena automáticamente con `"10/08/2026 - 15/08/2026"` y ambos atributos internos quedan guardados. |
| **CA-04** | Validación de rango invertido | Seleccionar `20/08/2026` como inicio y luego clicar `12/08/2026` como fin. | El componente impide que la fecha final sea anterior a la inicial (bloquea el rango o reinicia la fecha de inicio). |
| **CA-05** | Cierre automático al completar rango | Completar la selección de las dos fechas (inicio y fin). | El desplegable se oculta automáticamente dejando visible la barra de texto con el rango seleccionado. |

### 3.2. Componente B: Selector de Hora con Reloj Analógico

**Descripción funcional:** Ventana emergente con cabecera interactiva de Horas/Minutos y esfera analógica que permite seleccionar múltiplos de 5 mediante clic o minutos exactos mediante el arrastre de la aguja.

| ID | Escenario | Acción y Datos de Entrada | Resultado Esperado (Estado Interno y Visual) |
| :--- | :--- | :--- | :--- |
| **CB-01** | Inicialización por defecto | Instanciar el componente selector de hora. | El *popup* del reloj está cerrado, la cabecera muestra la hora inicial (ej. `00:00`) y la esfera activa por defecto es la de Horas. |
| **CB-02** | Apertura y conmutación de cabecera | Abrir el *popup*, clicar arriba en **Minutos** y luego volver a clicar en **Horas**. | Al clicar en Minutos cambia la vista a la esfera de minutos; al clicar en Horas vuelve a mostrar la esfera horaria. |
| **CB-03** | Selección de horas en la esfera | En modo Horas, clicar sobre el número `14` (o posición horaria correspondiente). | La cabecera superior actualiza las horas a `14` y el estado interno guarda la hora seleccionada. |
| **CB-04** | Selección rápida de minutos (de 5 en 5) | En modo Minutos, clicar sobre las marcas principales (`1`→`05`, `3`→`15`, `6`→`30`). | El componente calcula el múltiplo ($\text{posición} \times 5$), actualiza la cabecera a `:05`, `:15` o `:30` y guarda el valor. |
| **CB-05** | Selección de minuto intermedio por arrastre | En modo Minutos, arrastrar la aguja manualmente hasta un punto intermedio (ej. minuto `18`). | La aguja sigue el ángulo del ratón, la cabecera muestra `:18` y la variable interna almacena el minuto exacto (`18`). |
| **CB-06** | Volcado final de hora completa | Confirmar selección de hora (`14`) y minutos (`18`). | El campo principal del componente muestra `"14:18"` para llegada/salida del viaje. |
