# Guión de exposición (10 min) — Defensa de tesis

**Evaluación del rendimiento de un clúster de bajo costo basado en Raspberry Pi orquestado con Kubernetes**
Diego Oswaldo Márquez Paccha · Universidad Nacional de Loja

> Máximo 10 min. Ritmo ágil: ~15–20 s por slide, y las de sección casi de paso.
> ▸ marca cada **clic** en las slides con revelado por fragmentos.

---

## 1 · Carátula *(~15 s)*

Buenos días. Presento mi trabajo de titulación: *"Evaluación del rendimiento de un clúster de bajo costo basado en Raspberry Pi orquestado con Kubernetes"*, dirigido por el Ing. Cristian Narváez.

## 2 · Agenda *(rápida, ~8 s)*

Este es el recorrido: problema y objetivos, los dos objetivos del trabajo, y las conclusiones.

## 3 · Introducción *(~30 s)*

Los contenedores y Kubernetes son hoy el estándar, pero atados a la nube comercial, con costos que son una barrera para la academia. La alternativa: computación en el borde con Raspberry Pi y Kubernetes ligero. La motivación es democratizar el acceso a este cómputo con hardware asequible.

## 4 · Problemática y Pregunta *(~35 s)*

La pregunta que guía el trabajo: **¿cuál es la latencia y el throughput de un clúster con Kubernetes, bajo carga, en una arquitectura de bajo costo?**
▸ Contexto: la nube es una barrera de costo. ▸ Limitación: el hardware, sobre todo la red. ▸ Alternativa: Raspberry Pi con K3s. ▸ El vacío: hay poca evidencia de su rendimiento real y de qué lo limita. Eso vine a medir.

## 5 · Conceptos clave *(rápida, ~15 s)*

Tres ideas para seguir la charla: Kubernetes ligero, **K3s**; mis métricas, **latencia y throughput**; y el **thermal throttling**, la bajada de frecuencia por calor.

## 6 · Objetivos *(~25 s)*

El objetivo general: medir latencia y throughput bajo carga. Dos específicos: ▸ construir el prototipo físico, y ▸ orquestar el despliegue web y medirlo. La presentación sigue estos dos objetivos.

## 7 · (Sección) Objetivo 1 *(~5 s)*

Primer objetivo: la construcción del prototipo.

## 8 · Metodología O1 — Design Thinking *(~20 s)*

Construí el prototipo con Design Thinking, en tres iteraciones, hasta una carcasa que integrara los nodos con refrigeración y escalabilidad. A la derecha, las fases y el diseño 3D.

## 9 · Metodología O1 — Requisitos *(rápida, ~12 s)*

De ahí salieron los requisitos: refrigeración, puertos accesibles, switch integrado, telemetría, escalabilidad y bajo costo.

## 10 · Metodología O1 — Hardware y red *(~20 s)*

El clúster es heterogéneo: un maestro Raspberry Pi 4B y tres trabajadores Raspberry Pi 3, con IPs estáticas porque Kubernetes necesita enrutamiento determinista.

## 11 · Metodología O1 — Configuración base *(~15 s)*

Instalé Ubuntu Server y, clave para la fiabilidad, congelé el kernel y las actualizaciones para tener un entorno determinista. Luego instalé K3s.

## 12 · Resultados O1 — Prototipo *(~25 s)*

Este es el prototipo: carcasa de ocho piezas impresas en 3D que integra los cuatro nodos, los ventiladores y el switch, con telemetría OLED por nodo. Escalable y reproducible: el modelo 3D es público. *(Si llevas el clúster: "aquí lo tienen físicamente".)*

## 13 · Resultados O1 — Bajo costo *(~15 s)*

Costó unos 464 dólares, por debajo del servidor tradicional más barato, con ahorros de hasta 87 % frente a la nube. Se mantiene como bajo costo.

## 14 · Resultados O1 — Clúster operativo *(~15 s)*

K3s quedó operativo: los cuatro nodos *Ready* y NGINX en tres réplicas, una por trabajador. Base estable: cero fallos en 21 ejecuciones.

## 15 · (Sección) Objetivo 2 *(~5 s)*

Segundo objetivo: la evaluación del rendimiento.

## 16 · Metodología O2 — Diseño *(~20 s)*

Usé benchmarking. Variable independiente: con o sin refrigeración. Dependientes: latencia y throughput. La carga, NGINX; tres repeticiones por combinación: 18 ejecuciones.

## 17 · Metodología O2 — Niveles de carga *(~15 s)*

Generé la carga con Grafana K6 en tres niveles: 50, 200 y 500 usuarios virtuales, este último para llevar el sistema al estrés.

## 18 · Metodología O2 — Observabilidad *(~15 s)*

Monitoreé el interior con Prometheus y Grafana —CPU, red, temperatura por nodo— y exporté cada ejecución para reproducibilidad.

## 19 · Metodología O2 — Aislar el cuello de botella *(~20 s)*

Diseñé un experimento de control: en vez de dirigir el tráfico al maestro, lo repartí entre los trabajadores. Si mejora mucho, el cuello era la centralización; si no, es la red de los trabajadores.

## 20 · Resultados O2 — Rendimiento *(~30 s)*

Los resultados: en carga baja y media, holgura, latencia de 3 milisegundos. En carga alta, unas 915 peticiones por segundo, pero la latencia salta a 700 en el p95 —eso sí, cero fallos—. Escala hasta 200 usuarios y satura en 500.

## 21 · Resultados O2 — Comportamiento térmico *(~20 s)*

La refrigeración bajó hasta 39 grados. Pero atención: incluso sin ella, el maestro se quedó en 74 grados, por debajo del umbral de throttling. El throttling nunca se activó.

## 22 · Resultados O2 — Efecto de la refrigeración *(~20 s)*

Y aquí el giro: bajar 39 grados mejoró el throughput apenas 1.6 %. Consistente, pero sin relevancia práctica. Para cargas web, la temperatura no es el factor limitante.

## 23 · Resultados O2 — Telemetría interna *(~20 s)*

¿Qué lo limita entonces? El 99 % del tiempo de cada petición se va esperando el primer byte. La CPU al 28 %, el enlace al 8 %. Ni CPU ni ancho de banda: el retardo es *antes* de responder.

## 24 · Resultados O2 — Experimento complementario *(~20 s)*

Al distribuir el tráfico, el throughput subió apenas 3.4 % y la consistencia empeoró. No era la centralización: cada trabajador recibe el tráfico por su propia interfaz USB, igual de restringida.

## 25 · Resultados O2 — Factor limitante *(~30 s)*

*(Cada clic descarta una causa.)*
Descartando de forma sistemática: ▸ temperatura, no; ▸ CPU, no; ▸ ancho de banda, no; ▸ centralización, no. Lo que queda —y es la conclusión central— es el **hardware de red de los trabajadores**: en la Raspberry Pi 3 el Ethernet va por el bus USB, y eso penaliza el procesamiento por paquete.

## 26 · (Sección) Discusión *(~5 s)*

Contrasto ahora con la literatura.

## 27 · Discusión O1 — Costo *(~15 s)*

Mi costo está dentro del rango de la literatura y por debajo de un servidor tradicional. Confirma el bajo costo.

## 28 · Discusión O2 — Literatura *(~30 s)*

El contraste clave es con Benoit-Cattin: ellos sí alcanzaron el throttling, pero porque cargaron **inferencia de redes neuronales**, un trabajo de CPU intensivo —el rendimiento cayó de 16 a 10 fps y enfriar les dio 90 %—. Mi carga es web: deja el CPU al 28 %, nunca se calienta lo suficiente, y por eso enfriar me dio solo 1.6 %. Es decir, el throttling solo aparece con cargas que saturan el procesador; una carga web no lo hace. Y un clúster con Raspberry Pi 4 y Gigabit reportó latencias diez veces mayores, lo que muestra que la configuración pesa tanto como el hardware.

## 29 · Discusión O2 — Implicaciones *(~15 s)*

El mensaje de diseño: para cargas web, invertir en red antes que en refrigeración. La refrigeración se mantiene por fiabilidad del hardware, no por rendimiento.

## 30 · Discusión — Limitaciones *(~15 s)*

Limitaciones: tres repeticiones, un solo tipo de carga, clúster heterogéneo y pruebas de cinco minutos. Acotan el alcance de las conclusiones.

## 31 · (Sección) Conclusiones *(~5 s)*

Cierro con conclusiones y recomendaciones.

## 32 · Conclusiones — Respuesta *(~25 s)*

Respondiendo a la pregunta: bajo carga alta, 915 peticiones por segundo, latencia p95 de 700 milisegundos y cero fallos. Y el hallazgo central: el factor limitante no es la temperatura ni la centralización, sino la red de los trabajadores.

## 33 · Conclusiones — Objetivos *(~15 s)*

Ambos objetivos se cumplieron. La conclusión general: la infraestructura de bajo costo es viable, y en cargas web la red determina el rendimiento por encima del calor.

## 34 · Recomendaciones técnicas *(~15 s)*

Recomiendo: placas con Ethernet dedicado y switch Gigabit para mejorar; mantener la refrigeración por fiabilidad; y evaluar SSD por USB.

## 35 · Recomendaciones futuras *(~15 s)*

Y a futuro: un clúster homogéneo de control, más tipos de carga, más repeticiones, y valor docente del prototipo.

## 36 · Gracias *(~10 s)*

Con esto concluyo. Agradezco la atención del tribunal y quedo atento a sus preguntas.

---

## Notas para cumplir los 10 min

- **Pasa rápido** por agenda, conceptos y las cuatro slides de sección (7, 15, 26, 31): son de transición.
- **Invierte tu tiempo** en resultados y en el factor limitante (slides 20–25) y en las conclusiones (32); es lo que evalúa el tribunal.
- **No leas las tablas**: di solo la cifra clave de cada una.
- **Cronométrate** en un ensayo completo; si te pasas, recorta en metodología (slides 9, 11, 17, 18).
- **Preguntas probables:** por qué 3 repeticiones, por qué no RPi 4 homogéneas, cómo aislaste el switch, por qué NGINX. Respuestas cortas ya están en tus limitaciones y recomendaciones.
