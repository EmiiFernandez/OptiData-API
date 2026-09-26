# OptiData API

API REST para gestionar pacientes y prescripciones de lentes de una óptica. La armé para modelar con código un flujo que conozco de mi trabajo en oftalmología: el paciente, su receta con los valores de cada ojo y el pedido de los anteojos.

**Herramientas:** Java 17 · Spring Boot 3.5 · Spring Data JPA · MySQL · Flyway · MapStruct · Swagger (springdoc) · Docker

## Problema

En una óptica, los datos de una receta (esfera, cilindro, eje, adición, distancia pupilar) se cargan muchas veces a mano y sin validación, y después cuesta encontrar el historial de un paciente. Quería una API que guarde esos datos con reglas claras y permita consultarlos por paciente o por documento.

## Datos

- El modelo tiene cuatro tablas: `patients`, `prescriptions`, `orders` y `sales`.
- Una prescripción guarda los valores del ojo derecho (OD) y del izquierdo (OI): esfera, cilindro y eje (0 a 180), más adición, distancia pupilar, tipo de lente y diagnóstico.
- Los diagnósticos y tipos de lente son enumeraciones con traducción al español: 15 diagnósticos (miopía, presbicia, astigmatismo, glaucoma, catarata, entre otros) y 8 tipos de lente.
- Flyway crea las tablas y carga datos de prueba **inventados**: 30 pacientes, 30 prescripciones, 29 pedidos y 18 ventas. No hay datos reales de pacientes.

## Método

- **Capas:** controller → service → repository, con DTOs de entrada y salida y mappers de MapStruct entre entidades y DTOs.
- **Validaciones:** campos obligatorios, largo de nombre y apellido, email válido, fecha de nacimiento en el pasado y formato de teléfono (Bean Validation).
- **Reglas de negocio:** no se puede repetir el número de documento, y una prescripción solo se crea para un paciente existente.
- **Errores:** un manejador global devuelve respuestas uniformes para recursos no encontrados, duplicados y datos inválidos.
- **Migraciones:** el esquema y los datos de prueba se versionan con Flyway (`src/main/resources/db/migration`).

## Resultados

Endpoints disponibles, documentados en Swagger:

| Método | Ruta | Qué hace |
|---|---|---|
| POST | `/api/pacientes` | Crea un paciente |
| GET | `/api/pacientes` | Lista los pacientes |
| GET | `/api/pacientes/{id}` | Busca un paciente por ID |
| POST | `/api/prescripciones` | Crea una prescripción para un paciente |
| GET | `/api/prescripciones` | Lista las prescripciones |
| GET | `/api/prescripciones/with-patient` | Lista las prescripciones con los datos del paciente |
| GET | `/api/prescripciones/{documentNumber}` | Devuelve el historial de prescripciones de un paciente por su documento |

Las tablas de pedidos y ventas ya están en el modelo y en los datos de prueba, pero todavía no tienen endpoints.

## Cómo ejecutarlo

Requisitos: Java 17 y Docker.

```bash
git clone https://github.com/EmiiFernandez/OptiData-API.git
cd OptiData-API
cp .env.template .env          # usuario, contraseña y base de datos locales
docker compose up -d           # levanta MySQL en el puerto 3306
./mvnw spring-boot:run         # en Windows: mvnw.cmd spring-boot:run
```

Al arrancar, Flyway crea las tablas y carga los datos de prueba. La documentación queda en `http://localhost:8080/swagger-ui.html`.

`./mvnw test` también necesita la base levantada, porque el test de contexto se conecta a MySQL.

Si ya tenías una base `optidata` de una ejecución anterior y Flyway da error de validación, bórrala (`DROP DATABASE optidata;`) y vuelve a arrancar.

## Estructura del repo

```
OptiData-API/
├── src/main/java/com/ef/optidata/
│   ├── controller/      # endpoints REST
│   ├── service/         # lógica de negocio
│   ├── repository/      # acceso a datos (Spring Data JPA)
│   ├── entity/          # entidades y enumeraciones
│   ├── dto/             # objetos de entrada y salida
│   ├── mapper/          # MapStruct
│   ├── exception/       # errores y manejador global
│   └── config/          # traducción de mensajes
├── src/main/resources/
│   ├── db/migration/    # scripts de Flyway (esquema y datos de prueba)
│   ├── messages/        # textos en inglés y español
│   └── application.properties
├── docker-compose.yml   # MySQL local
├── .env.template
└── pom.xml
```

## Próximos pasos

- Endpoints para pedidos y ventas, con el cambio de estado del pedido (pendiente, en producción, listo, entregado).
- Validar rangos clínicos en las prescripciones (por ejemplo, esfera y cilindro en pasos de 0,25).
- Tests de servicios y de controladores con una base en memoria, para que el build no dependa de MySQL.
- Endpoints de consulta agregada (prescripciones por diagnóstico o tipo de lente) para alimentar un dashboard.

---

Emilia Fernández · [LinkedIn](https://www.linkedin.com/in/emiliafernandez) · Licencia MIT
