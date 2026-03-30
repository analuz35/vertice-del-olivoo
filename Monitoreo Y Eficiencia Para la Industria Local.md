Documento de Requerimientos y Arquitectura de Software (SRS)

Proyecto: Sistema IoT de Monitoreo y Eficiencia Energética Industrial

Ubicación Objetivo: Cruz del Eje y Noroeste Provincial (Desmotadoras, Aceiteras, Fincas).
Plazo de Ejecución: 8 Meses (7 de desarrollo + 1 de inserción de mercado).
Versión del Documento: 1.0

1. Resumen Ejecutivo

El proyecto consiste en el desarrollo de una solución integral (Hardware + SaaS) para el monitoreo en tiempo real del consumo eléctrico en tableros industriales. Utilizando sensores no invasivos (IoT) y análisis de datos en la nube, el sistema permite a las industrias agropecuarias/metalurgia, etc. prevenir fallas en maquinaria pesada, evitar multas por energía reactiva y predecir el costo de su facturación eléctrica, optimizando sus costos operativos.

2. Planteo del Problema (Dolores a Resolver)

Facturación "Sorpresa": Las industrias locales (ej. cooperativas, aceiteras) reciben la factura de EPEC (Empresa Provincial de Energía de Córdoba) a mes vencido, sin capacidad de reacción ante picos de consumo.

Multas por Ineficiencia: Penalizaciones económicas por exceso de consumo en horarios pico o por mala compensación de energía reactiva (motores ineficientes).

Mantenimiento Reactivo (Quema de Motores): Un motor que empieza a fallar por desgaste mecánico consume más corriente antes de quemarse. Hoy, la industria se entera cuando la máquina ya se rompió, frenando la producción.

Falta de Conectividad Rural: Las soluciones estándar de domótica requieren Wi-Fi estable, algo escaso en galpones y fincas de la Ruta 38.

3. Arquitectura de Hardware y Comunicaciones (IoT)

Dado el entorno industrial, la captura de datos se basa en los siguientes principios:

Medición No Invasiva: Uso de sensores de núcleo partido (SCT-013) que se abrazan a los cables de fase en el tablero principal, eliminando el riesgo de electrocución y la necesidad de cortar el suministro para su instalación.

Edge Computing: El microcontrolador (ESP32) procesa miles de muestras por segundo localmente para calcular el valor RMS (Root Mean Square) real de la corriente, enviando al servidor solo el dato procesado para ahorrar ancho de banda.

Tolerancia a Fallos de Red (Offline-First): * Opción de módulo GSM (Chip celular SIM800L/SIM7600) para zonas sin Wi-Fi.

Almacenamiento local en memoria MicroSD (Data Logger). Si se corta internet, el ESP32 guarda los datos y realiza un bulk upload (sincronización masiva) al recuperar la conexión.

4. Requerimientos Funcionales (Software / SaaS)

Módulo 1: Ingesta y Procesamiento de Datos (Telemetría)

Recepción de datos de corriente (Amperaje), voltaje estimado y potencia (Watts) cada 5 segundos por cada sensor instalado.

Cálculo automático de KWh (Kilovatios-hora) acumulados.

Algoritmo de estimación de costo en Pesos (ARS) indexado al cuadro tarifario vigente de EPEC o la cooperativa local.

Módulo 2: Panel de Control en Tiempo Real (Dashboard B2B)

Visualización gráfica del consumo general de la planta vs. circuitos individuales (Ej: "Línea de Prensado" vs. "Cámaras de Frío").

Gráficos de perfil de carga diario (curvas de demanda) para identificar consumos "vampiro" o ineficiencias nocturnas.

Módulo 3: Motor de Alertas y Mantenimiento Predictivo

Línea Base Inteligente: El sistema establece un promedio histórico de consumo por máquina.

Detección de Anomalías: Si un motor consume un 15% más de su línea base durante más de 10 minutos, el sistema dispara una alerta.

Notificaciones Push y WhatsApp: Alertas automáticas al jefe de planta ("Alerta: Sobreconsumo detectado en Motor Bomba Riego 2").

Módulo 4: Reportes y Exportación

Generación de reportes gerenciales en PDF/Excel a fin de mes, detallando:

Consumo total vs. Mes anterior.

Horarios de mayor ineficiencia.

Huella de Carbono estimada (kg CO2 equivalentes ahorrados).

5. Requerimientos No Funcionales

Alta Frecuencia de Ingesta (High Throughput): El backend debe ser capaz de soportar la ingesta concurrente de datos (cientos de requests por minuto por fábrica) sin cuellos de botella.

Almacenamiento de Series Temporales: Uso de bases de datos optimizadas para Time-Series (TSDB), permitiendo consultas rápidas sobre millones de registros históricos.

Protocolo Ligero: Uso de MQTT (Message Queuing Telemetry Transport) en lugar de HTTP para la comunicación Hardware-Servidor, reduciendo la latencia y el consumo de datos móviles.

Seguridad IoT: Autenticación de dispositivos mediante tokens únicos y conexión cifrada (TLS/SSL) al broker MQTT.

6. Stack Tecnológico Sugerido

Hardware (Firmware): C++ (Framework Arduino/ESP-IDF), ESP32, Sensores SCT-013, Módulo RTC (Reloj en tiempo real) y MicroSD.

Comunicaciones: Broker MQTT (Eclipse Mosquitto).

Base de Datos: TimescaleDB (Extensión de PostgreSQL para series temporales) o InfluxDB.

Backend: Node.js (Express o NestJS).

Frontend Dashboard: React.js / Next.js / Vite.js (con librerías de gráficos avanzados como Recharts o Apache ECharts).

7. Plan de Ejecución y Roadmap (8 Meses)

Mes 1: Prototipado Hardware. Ensamblaje del ESP32 con sensores SCT-013, calibración de mediciones RMS y pruebas en tablero de prueba local.

Mes 2: Infraestructura Cloud. Configuración del Broker MQTT y la base de datos de series temporales (TimescaleDB).

Mes 3: Desarrollo Backend. Creación de endpoints para ingesta de datos, cálculo de KWh y motor de reglas de alertas.

Mes 4: Resiliencia IoT. Programación de la lógica "Offline" en el ESP32 (guardado en SD y reintento de conexión vía módulo GSM).

Mes 5-6: Frontend y Analytics. Desarrollo del Dashboard web para las industrias, integración de gráficos en tiempo real (WebSockets) y alertas vía WhatsApp API.

Mes 7: Pruebas de Estrés y QA. Simulación de 50 sensores enviando datos simultáneamente durante 7 días. Calibración fina de algoritmos de detección de anomalías.

Mes 8: Implementación Piloto. Instalación del hardware en 2 industrias o fincas locales (Ej: desmotadora o secadero) y validación de la estimación de costos vs. la factura real de luz.
