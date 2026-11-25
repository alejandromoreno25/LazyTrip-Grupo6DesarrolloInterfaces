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
