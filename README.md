# Sistema de Venta de Entradas y Recomendación de Eventos

Trabajo Práctico Integrador de **Bases de Datos 2026** — Licenciatura en Ciencia de Datos en Organizaciones, UNLP.

**Autores:** Lautaro Pennetta y Franco Suárez

## De qué se trata

Armamos el backend de datos de una plataforma de venta de entradas para recitales y eventos, usando tres bases de datos NoSQL desde Python. La idea del trabajo no era usar las tres porque sí, sino decidir qué parte del problema resuelve mejor cada una:

| Base | Qué guarda | Por qué |
|------|-----------|---------|
| **MongoDB** | Usuarios, artistas, lugares, eventos y compras | Son los datos permanentes del sistema. Los separamos en colecciones y los relacionamos por identificadores. |
| **Redis** | Sesiones, reservas temporales, ranking de búsquedas y caché | Son datos que vencen solos: la sesión dura 30 minutos, la reserva 10 y la caché 5. |
| **Neo4j** | Amistades entre usuarios, artistas que sigue cada uno y artistas similares | Las recomendaciones dependen de recorrer relaciones (amigos de amigos, artistas parecidos), que es justo lo que resuelve bien un grafo. |

Las tres bases no se comunican entre sí: el notebook consulta una, toma los identificadores del resultado y con eso consulta la otra.

## Qué incluye el notebook

1. **Carga inicial** de las tres bases, con índices únicos en MongoDB y restricciones de unicidad en Neo4j.
2. **Flujo de compra**: reservar entradas, confirmar la compra y cancelar la reserva. Probamos una compra exitosa, un pedido sin disponibilidad, una cancelación y una reserva vencida.
3. **12 consultas** que combinan las tres bases (eventos por género, recomendaciones por amigos, gasto por usuario, búsquedas más populares, etc.).
4. **Estadísticas con gráficos** en Matplotlib: seguidores por artista, recaudación por evento, entradas vendidas por lugar y géneros más buscados.
5. **Modificaciones** que mantienen la consistencia entre bases: aumento de precios, cancelación de un evento con reembolso, eliminación de un usuario y de relaciones de amistad.

## Algunas decisiones que tomamos

- **La disponibilidad no se guarda, se calcula**: cupo − entradas vendidas (MongoDB) − reservas vigentes (Redis). Así, cuando una reserva vence, las entradas vuelven a estar disponibles sin tener que ejecutar nada.
- **Cada compra guarda su precio**, para que un aumento posterior no cambie lo que ya pagaron los compradores anteriores.
- **Una sola relación `AMIGO_DE` por pareja** en Neo4j, recorrida sin importar la dirección, para no duplicar amistades.
- **Al eliminar un usuario conservamos sus compras** (desvinculadas) para no perder el historial de ventas.
- Agregamos al dataset una usuaria, un artista, un lugar y un evento **sin actividad**, para comprobar que las estadísticas muestran también los casos en cero.

Las decisiones y los problemas que encontramos están explicados en detalle en el informe.

## Cómo ejecutarlo

Se necesita [Docker Desktop](https://www.docker.com/products/docker-desktop/) (o Docker Engine con `docker compose` v2).

```bash
cp .env.example .env
docker compose up --build
```

Después abrir JupyterLab en `http://localhost:8888`, entrar a `notebooks/tp_integrador.ipynb` y ejecutar **Restart Kernel and Run All Cells**. La primera celda limpia las tres bases, así que se puede correr de cero las veces que haga falta.

Si aparece un error de autenticación en MongoDB, probablemente los volúmenes se crearon antes del `.env`. Se soluciona con:

```bash
docker compose down -v
docker compose up --build
```

## Estructura

```
├── docker-compose.yml        # Python + JupyterLab, MongoDB, Redis y Neo4j
├── .env.example              # credenciales de ejemplo
├── requirements.txt          # librerías de Python
├── python/                   # Dockerfile del contenedor de Jupyter
└── notebooks/
    ├── tp_integrador.ipynb   # el trabajo práctico
    └── test_connections.ipynb
```

## Límites del trabajo

El notebook corre las operaciones de a una, así que no contemplamos usuarios reservando exactamente al mismo tiempo, y no hay una transacción que abarque las tres bases a la vez: la consistencia se mantiene coordinando cada operación desde Python.

---

El entorno de Docker está basado en el stack provisto por la cátedra.
