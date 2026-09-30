# Guion de la demostracion (video breve, 4 a 6 minutos)

Objetivo: mostrar el recorrido completo Fuente -> Kafka -> Beam -> Salida y
evidenciar el comportamiento ante eventos normales, duplicados y tardios.

Preparar cuatro terminales en la carpeta del proyecto.

## 0. Introduccion (30 s)

- Presentar el caso: total de ventas confirmadas por tienda y por minuto, en
  tiempo casi real, para operacion de retail.
- Mostrar el diagrama `docs/diagrama-arquitectura.png` y nombrar las cuatro
  etapas.

## 1. Levantar la infraestructura (45 s)

Terminal 1:

```bash
docker compose up -d
docker compose logs init-topics
```

Señalar los tres topicos creados: `ventas.raw`, `ventas.totales`,
`ventas.invalidos`.

## 2. Arrancar el pipeline (45 s)

Terminal 2:

```bash
make pipeline
```

Explicar en una frase: lee de `ventas.raw` con KafkaIO, ventanea por tiempo de
evento, deduplica con estado, agrega y publica en `ventas.totales`.

## 3. Consumir los totales (30 s)

Terminal 3:

```bash
make consumer
```

Queda a la espera; aca se veran aparecer y actualizarse los totales por ventana.

## 4. Producir eventos y observar (2 a 3 min)

Terminal 4:

```bash
make producer
```

Mientras corre, el productor imprime en consola cuando inyecta un DUPLICADO, un
TARDIO o un INVALIDO. Ir narrando y volviendo a la terminal 3 del consumidor:

- Eventos normales: el total de cada tienda por ventana va creciendo; cada
  ventana muestra su pane EARLY, luego ON_TIME.
- Duplicado: cuando el productor marca "DUPLICADO event_id=...", mostrar que el
  total de esa tienda y ventana NO cambia (la deduplicacion lo descarta).
- Tardio: cuando marca "TARDIO Ns", mostrar que el total de una ventana ya
  cerrada se corrige con un pane LATE (revision), siempre que este dentro de los
  120 s de lateness.
- Invalido: mostrar el topico dead-letter recibiendo el evento sin `monto`:

```bash
docker compose exec kafka /opt/kafka/bin/kafka-console-consumer.sh \
  --bootstrap-server kafka:19092 --topic ventas.invalidos --from-beginning --max-messages 3
```

## 5. Cierre (30 s)

- Mostrar el estado final materializado en la terminal del consumidor: una fila
  por ventana (upsert por `tienda_id|window_start`).
- Mencionar la semantica declarada (at-least-once + idempotencia, no
  exactly-once) y remitir al documento tecnico.

Detener todo:

```bash
docker compose down -v
```

## Plan B si Docker no esta disponible

Grabar el smoke test en modo archivo, que corre el pipeline completo sin Kafka:

```bash
make smoke
```

Mostrar la verificacion de totales, la deduplicacion y el dead-letter en la
salida del script.
