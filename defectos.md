# Defectos — Pruebas de carga

## Defecto 01 — Conexiones rechazadas al iniciar el estrés

En el primer segundo del escenario stress, 5 peticiones fallaron con "connection refused" mientras los 200 VUs iniciales se conectaban a la vez. Después no volvió a pasar.

Evidencia (perf/results/k6-stress.log):

time="2026-09-20T23:08:32-05:00" level=warning msg="Request Failed" error="dial tcp 127.0.0.1:8080: connectex: No connection could be made because the target machine actively refused it."

Esperado: 0 errores. Obtenido: 5 de 2.619.695, por debajo del umbral del 1%.

Causa probable: la cola de conexiones pendientes de Tomcat (accept-count, 100 por defecto) se llena con la ráfaga inicial. Es lo que pasaría en producción tras un reinicio, cuando todos reconectan a la vez.

Estado: Abierto. Prioridad: Media. Mejora: subir server.tomcat.accept-count o suavizar la rampa inicial.

## Defecto 02 — El estrés no satura el servicio

Con 600 VUs el p95 fue 0,60 ms, casi igual que con 20. El script espera 100 ms entre peticiones, así que el límite está en el cliente, no en el servidor. Es un problema del diseño de la prueba.

Estado: Abierto. Prioridad: Media. Mejora: repetir con SLEEP_MS=0 o con el escenario arrival.

| ID | Escenario | Esperado | Obtenido | Estado |
|---|---|---|---|---|
| 01 | stress | 0 errores | 5 conexiones rechazadas | Abierto |
| 02 | stress | Encontrar saturación | p95 plano | Abierto |
