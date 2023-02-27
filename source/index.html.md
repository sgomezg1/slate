---
title: API Knowledge USB - Documentación

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='https://www.usbbog.edu.co/'>Universidad San Buenaventura</a>
  - <a href='https://github.com/slatedocs/slate'>Documentación hecha en Slate</a>

includes:
  - errors

search: false

code_clipboard: true

meta:
  - name: description
    content: Documentación Knowledge USB API
---

# Introducción

¡Saludos! En este documento vamos a mostrar una guia correspondiente a los servicios web que están desarrollados para la aplicación Knowledge USB. Dashboard que permitirá a la universidad tomar decisiones con respecto a la gestión de conocimiento de la institución. Está diseñado este proyecto a manera de prototipo, en donde esto se puede editar a futuro y además, de ser un entregable del proyecto, es una guia necesaria para las personas que editen el proyecto a futuro, o necesiten de los servicios web.

Este proyecto ha sido desarrollado con Laravel (PHP) debido a la facilidad de desarrollo y la simple sintaxis del lenguaje de programación. Para editar los servicios, es necesario que la persona conozca PHP y sepa como funciona este framework, que está diseñado bajo el patrón de arquitectura MVC y es orientado a objetos.

Se usará MySQL como sistema gestor de base de datos.

Se mostrará en este documento los parametros a enviar a cada servicio, el cuerpo de las respuestas, los posibles errores generados por el API y su interpretación.

Esta documentación se ha creado gracias a [Slate](https://github.com/slatedocs/slate).

# Desplegar API Knowledge USB

Este proyecto, usa una base de datos igual al un antecedente directo de nuestro proyecto. Este proyecto al ser desarrollado con Laravel, nos permite poder crear las tablas por medio de la interfaz de comandos [Artisan] (https://laravel.com/docs/9.x/artisan) y llenarlas por medio de otros elementos que vienen listos para ser usados en el framework, como los son Seeders y Factories. A continuación, se listarán los pasos que deben seguirse de la forma descrita en este documento par asegurar un correcto despliegue del API desarrollada para el proyecto.

1. Tener un servidor que contenga Apache 2.0 como mínimo, PHP mínimo en su versión 7.4 y MySQL mínimo en versión 5.8; Adicionalmente instalar y configurar Composer en nuestro entorno de desarrollo (Sea Linux o Windos).

2. Clonar este [repositorio] (https://github.com/sgomezg1/KnowledgeUSBBackend). Podemos clonar esto directamente en nuestro servidor web si tenemos acceso a una terminal por SSH u otro método, lo cual es muy recomendable para hacer la instalación más sencilla.

3. En la raiz de nuestro proyecto, ejecutar el comando **composer install**. Si presenta fallas y en la raíz del proyecto existe un archivo llamado **composer-lock.json**, eliminarlo y ejecutar de nuevo el comando.

4. Crear un archivo llamado .env; podemos copiar el archivo .env.example y pegar su contenido en el nuevo archivo. En donde allí reemplazaremos los datos de conexión a la base de datos del archivo de ejemplo por los que nosotros tengamos configurados en nuestra base de datos.

5. Ejecutar los siguientes comandos:

- **php artisan config:cache**
- **php artisan migrate:fresh**
- **php artisan key:generate**
- **php artisan passport:install**
- **php artisan passport:keys --force**
- **php artisan config:cache**
- **php artisan storage:link**

6. Los anteriores comandos, nos permitieron tener toda la base de datos lista para autenticarnos y hacer el llenado correspondiente con datos de prueba. Recuerde que los datos generados acá los provee la librería [Faker] (https://github.com/fzaninotto/Faker) y no tienen nombres reales ni nada que sirva como información oficial para la universidad. Aclarado esto, procedemos a correr los siguientes comandos:

- **php artisan db:seed --class=FacultadSeeder**
- **php artisan db:seed --class=ProgramaSeeder**
- **php artisan db:seed --class=MateriumSeeder**
- **php artisan db:seed --class=TipoUsuariosSeeder**
- **php artisan usuario:rol**
- **php artisan db:seed --class=ClaseSeeder**
- **php artisan db:seed --class=LineasDeInvestigacionSeeder**
- **php artisan db:seed --class=GrupoInvestigacionSeeder**
- **php artisan db:seed --class=AreaConocimientoSeeder**
- **php artisan db:seed --class=EventoSeeder**
- **php artisan db:seed --class=TipoProyectoSeeder**
- **php artisan db:seed --class=MacroProyectoSeeder**
- **php artisan db:seed --class=ConvocatoriaSeeder**

Posteriormente, debemos usar el siguiente comando para crear un semillero, necesario para la correcta creación de un proyecto, ya que la columna semillero en la tabla proyectos no es nullable.

- **php artisan crear:semillero --cantidad=CANTIDAD_SEMILLEROS_A_CREAR --grupoInvestigacion=ID_GRUPO_INVESTIGACION_BD --lineaInvestigacion=NOMBRE_LINEA_INVESTIGACION_A_ASINGAR**

Y para crear cuantos proyectos necesitemos, debemos ejecutar el siguiente comando:

- **php artisan crear:proyecto**

Con estos comandos, tenemos lista nuestra base de datos para empezar a crear usuarios, proyectos y realizar otro tipo de operaciones necesarias para ejecutar el prototipo de nuestra aplicación. Cabe aclarar que si queremos crear más usuarios, podemos generarlos con el comando **php artisan usuario:rol**, específicado anteriormente.

Para poder crear un proyecto, debemos enviar estos parámetros:

| Parámetro           | Descripción                                           |
| ------------------- | ----------------------------------------------------- |
| --semillero         | ID de semillero a asignar, no obligatorio             |
| --tipoProyecto      | Asignar o no un tipo de proyecto                      |
| --areasConocimiento | Cantidad de areas de conocimiento a asignar           |
| --productos         | Cantidad productos a asignar                          |
| --presupuestos      | Cantidad de presupuestos a asignar                    |
| --participaciones   | Cantidad de participaciones del proyecto en un evento |
| --clases            | Cantidad de clases a asignar                          |
| --participantes     | Cantidad de participantes a asignar                   |
| --convocatorias     | Cantidad de convocatorias a asignar                   |

**NOTA: El despliegue de los archivos cambiará según en donde vayamos a desplegar nuestro proyecto. Si es en un entorno local, nos bastará con clonar el repositorio y seguir los pasos anteriormente mencionados, si es en un servidor web, se deben hacer los siguientes pasos.**

1. Mover el contenido de la carpeta public a la raiz de nuestro servidor web. Es decir, si tengo un hosting con dominio http://ejemplo-knowledge-usb.com, debo tomar el contenido de la carpeta public y moverlo a la carpeta public_html de nuestro hosting.

2. Los archivos restantes, debemos moverlos un nivel atras de la carpeta public, en donde crearemos una carpeta con el nombre knowledgeusbapi

# Autenticación

Es necesario iniciar sesión en el sistema para poder hacer uso de los servicios web disponibles en esta documentación. Para la autenticación, utilizamos [Laravel Passport] (https://laravel.com/docs/9.x/passport), que implementa la autenticación OAuth 2.0. Recibimos un token que servirá como parametro de seguridad obligatorio al hacer las peticiones al API, debido a que debemos asegurar nuestra aplicación de peticiones indeseadas. Para esto, TODAS las peticiones exceptuando las del siguiente listado deben ser enviadas con el siguiente header;

`Authorization: Bearer <MI_API_KEY>`

<b>Listado de servicios excluidos:</b>

`POST http://localhost:8000/api/auth/prevLogin`

`POST http://localhost:8000/api/auth/login`

<aside class="notice">
Se recomienda SIEMPRE crear un interceptor para validar que el token que se está enviando, no se haya expirado.<br>
TIEMPO DE EXPIRACIÓN DE TOKEN: 1 Semana.
</aside>

## Obtener roles para iniciar sesión

> Para autenticarse: Debes hacer esta petición:

```shell
# Copia este enlace y agrega los parametros requeridos
curl "http://localhost:8000/api/auth/prevLogin"

```

> Petición correcta:

```json

{
    "roles": [
        {
            "id": 1,
            "nombre": "nombre_rol",
            "descripcion": "",
        },
        {
            "id": 2,
            "nombre": "nombre_rol",
            "descripcion": ""
        }
        ...
    ]
}

```

> Petición con error:

```json
{
  "error_code": "INVALID_CREDENTIALS",
  "mensaje": "Error, usuario o contraseña incorrectas."
}
```

Esta petición retorna todos los roles disponibles de un usuario.

### Petición HTTP

`POST http://localhost:8000/api/auth/prevLogin`

### Parametros

| Parametro  | Default   | Requerido | Descripción                                                                                        |
| ---------- | --------- | --------- | -------------------------------------------------------------------------------------------------- |
| correo_est | No aplica | Si        | Correo institucional del usuario que obtendrá sus roles. Se espera una dirección de correo valida. |
| password   | No aplica | Si        |  Contraseña del correo.                                                                            |

<aside class="notice">
Se debe hacer esta petición primero, debido a que cada usuario puede tener asignado más de un rol. Debemos verificar que roles tiene disponibles el usuario para así poderse autenticar y acceder a partes específicas del sistema para el rol seleccionado.
</aside>

## Iniciar sesión

> Para autenticarse: Debes hacer esta petición:

```shell
# Copia este enlace y agrega los parametros requeridos
curl "http://localhost:8000/api/autho/login"

```

> Petición correcta:

```json
{
  "access_token": "TOKEN_RECIBIDO",
  "token_type": "Bearer",
  "expires_at": "FECHA_Y_HORA_DE_EXPIRACION"
}
```

> Petición con error:

```json
{
  "error_code": "INVALID_CREDENTIALS",
  "mensaje": "Error, usuario o contraseña incorrectas."
}
```

Esta petición nos retorna el token de autenticación.

### Petición HTTP

`POST http://localhost:8000/api/auth/login`

### Parametros

| Parametro  | Default   | Requerido | Descripción                                                                                        |
| ---------- | --------- | --------- | -------------------------------------------------------------------------------------------------- |
| correo_est | No aplica | Si        | Correo institucional del usuario que obtendrá sus roles. Se espera una dirección de correo valida. |
| password   | No aplica | Si        | Contraseña del correo.                                                                             |

<aside class = "notice">
El token recibido en este servicio es el que debemos enviar en el header de cada una de las peticiones realizadas a los endpoint diferentes a la lista anteriormente especificada. Sin este token, obtendremos error de permisos por parte del servidor. Se recomienda guardarlo en un estado o en LocalStorage en nuestro front-end.
</aside>

## Aceptar politicas de privacidad

> Persistencia para aceptación de politica de tratamiento de datos:

```shell
# Copia este enlace y agrega los parametros requeridos
curl "http://localhost:8000/api/auth/aceptar-politicas/{id}"

```

> Petición correcta:

```json
{
  "mensaje": "Politica de tratamiento de datos aceptada"
}
```

> Petición con error:

```json
{
  "error_code": "NO_EXISTING_USER",
  "mensaje": "El usuario que acepta la política de tratamiento de datos no existe en nuestro sistema."
}
```

Esta petición nos retorna el token de autenticación.

### Petición HTTP

`GET http://localhost:8000/api/auth/aceptar-politicas/{id}`

### Parametros

| Parametro | Default   | Requerido | Descripción                                                                                        |
| --------- | --------- | --------- | -------------------------------------------------------------------------------------------------- |
| id        | No aplica | Si        | Correo institucional del usuario que obtendrá sus roles. Se espera una dirección de correo valida. |

# Buscador

## Proyectos

```shell
curl "http://localhost:8000/api/proyectos/"
```

> Petición con resultados:

```json
{
  "success": true,
  "proyectos": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "participantes": [
        {
          "cedula": "247698",
          "cod_universitario": 6756,
          "correo_est": "glover.geovanni@yahoo.com",
          "nombres": "Larry Christiansen III",
          "apellidos": "Hahn",
          "telefono": "1-520-664-5570",
          "correo_personal": "rosalia.johnson@reichert.com",
          "semillero_id": null,
          "programa_id": 2
        }
      ],
      "area_conocimientos": [
        {
          "id": 1,
          "nombre": "Jovani Robel MD",
          "gran_area": "Deleniti sint.",
          "descripcion": "Et non veritatis.",
          "pivot": {
            "proyecto": 3,
            "area_conocimiento": 1
          }
        }
      ],
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        }
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "proyectos": []
}
```

Este servicio retorna todos los proyectos alojados en el sistema de información.

### Petición HTTP

`POST http://localhost:8000/api/proyectos/`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Proyecto por ID

```shell
curl "http://localhost:8000/api/proyectos/{id}"
```

> Petición con resultados

```json
{
  "success": true,
  "proyectos": {
    "id": 3,
    "titulo": "Miss",
    "estado": "En Desarrollo",
    "descripcion": "Architecto voluptate optio.",
    "macro_proyecto": {
      "id": 7,
      "nombre": "Dr. Cali Zboncak",
      "descripcion": "Asperiores vero sint sit deleniti quis ex incidunt inventore et.",
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "estado": "eius"
    },
    "fecha_inicio": "2023-01-17T00:00:00.000000Z",
    "fecha_fin": "2023-01-17T00:00:00.000000Z",
    "semillero": {
      "id": 3,
      "nombre": "Aletha Hintz PhD",
      "descripcion": "Ab nihil enim.",
      "fecha_fun": "2023-01-17T00:00:00.000000Z",
      "grupo_investigacion": 5,
      "lider_semillero": "670343019",
      "linea_investigacion": "Andres Quigley"
    },
    "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
    "visibilidad": 1,
    "ciudad": "East Sadie",
    "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
    "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
    "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
    "tipo_proyecto": {
      "nombre": "Inv. Independientes",
      "descripcion": ""
    },
    "tipo_conocimiento": "Tacito",
    "nombre_facultad": "Humanidades y Educación",
    "nombre_programa": "Licenciatura en Educación Infantil",
    "antecedentes": [],
    "area_conocimientos": [
      {
        "id": 1,
        "nombre": "Jovani Robel MD",
        "gran_area": "Deleniti sint.",
        "descripcion": "Et non veritatis.",
        "pivot": {
          "proyecto": 3,
          "area_conocimiento": 1
        }
      }
    ],
    "participantes": [
      {
        "cedula": "247698",
        "cod_universitario": 6756,
        "correo_est": "glover.geovanni@yahoo.com",
        "nombres": "Larry Christiansen III",
        "apellidos": "Hahn",
        "telefono": "1-520-664-5570",
        "correo_personal": "rosalia.johnson@reichert.com",
        "semillero_id": null,
        "programa_id": 2
      }
    ],
    "productos": [
      {
        "id": 5,
        "titulo_producto": "Ms.",
        "tipo_producto": "non",
        "url_repo": "https://source.unsplash.com/random/800x600",
        "fecha": "2023-01-17T00:00:00.000000Z",
        "proyecto": 3
      }
    ],
    "presupuestos": [
      {
        "id": 1,
        "monto": 1810533,
        "fecha": "2023-01-17T00:00:00.000000Z",
        "proyecto": 3,
        "descripcion": "Quia architecto exercitationem omnis non. Cum exercitationem deserunt corporis quae quibusdam est voluptas. Aliquam ea dolor molestiae ut quam expedita nihil. Autem qui molestiae et tempora optio et aut perferendis."
      }
    ]
  }
}
```

> Petición sin resultados

```json
{
  "success": false,
  "mensaje": "No hay un proyecto con este ID"
}
```

Esta petición regresa los datos de un proyecto por su ID.

### Petición HTTP

`GET  http://localhost:8000/api/proyectos/{id}`

### Parametros URL

| Parametro | Requerido | Descripción                |
| --------- | --------- | -------------------------- |
| ID        | No        | ID de proyecto a consultar |

## Obtener investigadores

```shell
curl "http://localhost:8000/api/investigadores"
```

> Petición con resultados

```json
{
  "success": true,
  "usuarios": [
    {
      "cedula": "247698",
      "nombres": "Larry Christiansen III",
      "apellidos": "Hahn",
      "participaciones": [
        {
          "id": 3,
          "titulo": "Miss",
          "estado": "En Desarrollo",
          "descripcion": "Architecto voluptate optio.",
          "macro_proyecto": 7,
          "fecha_inicio": "2023-01-17T00:00:00.000000Z",
          "fecha_fin": "2023-01-17T00:00:00.000000Z",
          "semillero": 3,
          "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
          "visibilidad": 1,
          "ciudad": "East Sadie",
          "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
          "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
          "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
          "tipo_proyecto": "Inv. Independientes",
          "tipo_conocimiento": "Tacito",
          "pivot": {
            "usuario": "247698",
            "proyecto": 3
          },
          "clases": [
            {
              "numero": 10424,
              "nombre": "Prof. Edna Gibson",
              "semestre": "4",
              "materia": "Hortense",
              "profesor": "670343019",
              "materium": {
                "catalogo": "Hortense",
                "nombre": "Eugene Spinka MD",
                "programa": 2,
                "programas": {
                  "id": 2,
                  "nombre": "Licenciatura en Educación Infantil",
                  "facultad_id": 2,
                  "director": null,
                  "facultad": {
                    "id": 2,
                    "nombre": "Humanidades y Educación",
                    "decano": null,
                    "coor_inv": null
                  }
                }
              }
            }, ...
          ]
        }
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "usuarios": []
}
```

Esta petición nos traerá los investigadores almacenados en el sistema.

### Petición HTTP

`POST http://localhost:8000/api/investigadores`

### Parametros URL

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Investigadores por número de documento

```shell
curl "http://localhost:8000/api/investigadores/{numDocumento}"
```

> Petición con resultados:

```json
{
    "success": true,
    "usuario": [
        {
            "cedula": "670343019",
            "cod_universitario": 78459,
            "correo_est": "eloise14@gmail.com",
            "nombres": "Michele Hayes PhD",
            "apellidos": "Strosin",
            "telefono": "+1.516.957.4995",
            "correo_personal": "blanda.isidro@yundt.com",
            "semillero_id": null,
            "programa_id": 6,
            "participaciones": [
                {
                    "id": 6,
                    "titulo": "Prof.",
                    "estado": "Finalizado",
                    "descripcion": "Modi est assumenda.",
                    "macro_proyecto": 10,
                    "fecha_inicio": "2023-01-17T00:00:00.000000Z",
                    "fecha_fin": "2023-01-17T00:00:00.000000Z",
                    "semillero": 5,
                    "retroalimentacion_final": "Labore est consequatur quia distinctio cupiditate impedit ducimus.",
                    "visibilidad": 1,
                    "ciudad": "Trantowville",
                    "metodologia": "Debitis temporibus consequuntur commodi magni dolores sapiente ut.",
                    "conclusiones": "Dolores deserunt molestiae nihil in quia illum et harum aut vero soluta voluptatem.",
                    "justificacion": "Ut qui cum quo eligendi minus recusandae sunt.",
                    "tipo_proyecto": "Proyecto de Aula",
                    "tipo_conocimiento": "Implicito",
                    "pivot": {
                        "usuario": "670343019",
                        "proyecto": 6
                    },
                    "clases": [
                        {
                            "numero": 39470,
                            "nombre": "Ayana Batz V",
                            "semestre": "3",
                            "materia": "Bell",
                            "profesor": "670343019",
                            "materium": {
                                "catalogo": "Bell",
                                "nombre": "Grayce Fadel",
                                "programa": 8,
                                "programas": {
                                    "id": 8,
                                    "nombre": "Ingeniería Mecatrónica",
                                    "facultad_id": 3,
                                    "director": null,
                                    "facultad": {
                                        "id": 3,
                                        "nombre": "Ingeniería",
                                        "decano": null,
                                        "coor_inv": null
                                    }
                                }
                            }
                        }, ...
                    ]
                }, ...
            ]
        }
    ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "usuario": []
}
```

Este servicio trae valores numéricos correspondientes al presupuesto asignado a los proyectos por mes.

### Petición HTTP

`GET http://localhost:8000/api/investigadores/{numDocumento}`

### Parametros URL

| Parametro    | Requerido | Descripción                                      |
| ------------ | --------- | ------------------------------------------------ |
| numDocumento | No        | Número de documento del investigador a consultar |

# Reportes

## Reporte proyectos con presupuesto

```shell
curl "http://localhost:8000/api/reportes/presupuestos" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que tienen presupuesto asignado junto con datos adicionales.

### Petición HTTP

`POST http://localhost:8000/api/reportes/presupuestos`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos por convocatoria

```shell
curl "http://localhost:8000/api/reportes/convocatorias" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que vienen por convocatoria.

### Petición HTTP

`POST http://localhost:8000/api/reportes/convocatorias`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos que requieren integrantes

```shell
curl "http://localhost:8000/api/reportes/integrantes" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que no tienen integrantes y necesitan que se asignen uno o más.

### Petición HTTP

`POST http://localhost:8000/api/reportes/integrantes`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos de semillero

```shell
curl "http://localhost:8000/api/reportes/semillero" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que son de semillero.

### Petición HTTP

`POST http://localhost:8000/api/reportes/semillero`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos de investigadores independientes

```shell
curl "http://localhost:8000/api/reportes/investigadores-independientes" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que son de investigadores independientes.

### Petición HTTP

`POST http://localhost:8000/api/reportes/investigadores-independientes`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos que son trabajo de grado

```shell
curl "http://localhost:8000/api/reportes/trabajo-de-grado" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        } ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que son trabajos de grado.

### Petición HTTP

`POST http://localhost:8000/api/reportes/trabajo-de-grado`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos por facultad

```shell
curl "http://localhost:8000/api/reportes/facultad/{idFacultad}" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        },
        {
          "numero": 51903,
          "nombre": "Yasmine Jacobi",
          "semestre": "7",
          "materia": "Sarina",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Sarina",
            "nombre": "Prof. Kirk Cole DVM",
            "programa": 10,
            "programas": {
              "id": 10,
              "nombre": "Ingeniería de Sistemas",
              "facultad_id": 3,
              "director": null,
              "facultad": {
                "id": 3,
                "nombre": "Ingeniería",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        },
        {
          "numero": 77383,
          "nombre": "Mack McCullough",
          "semestre": "2",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        }
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos que son de semillero.

### Petición HTTP

`POST http://localhost:8000/api/reportes/facultad/{idFacultad}`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| idFacultad       | Si        | ID de facultad a consultar.                                                           |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

## Reporte proyectos por programa

```shell
curl "http://localhost:8000/api/reportes/programa/{idPrograma}" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "RUTA_DE_REPORTE_GENERADO",
  "data": [
    {
      "id": 3,
      "titulo": "Miss",
      "estado": "En Desarrollo",
      "descripcion": "Architecto voluptate optio.",
      "macro_proyecto": 7,
      "fecha_inicio": "2023-01-17T00:00:00.000000Z",
      "fecha_fin": "2023-01-17T00:00:00.000000Z",
      "semillero": 3,
      "retroalimentacion_final": "Quia provident quod delectus incidunt et quo vel iusto occaecati maiores in.",
      "visibilidad": 1,
      "ciudad": "East Sadie",
      "metodologia": "Occaecati aut culpa inventore deleniti dolores aut iure.",
      "conclusiones": "Tempore culpa sequi amet sed dolores ratione quidem suscipit molestiae error odit.",
      "justificacion": "Quaerat corrupti debitis quae nihil eveniet saepe eum quibusdam possimus quas.",
      "tipo_proyecto": "Inv. Independientes",
      "tipo_conocimiento": "Tacito",
      "clases": [
        {
          "numero": 10424,
          "nombre": "Prof. Edna Gibson",
          "semestre": "4",
          "materia": "Hortense",
          "profesor": "670343019",
          "materium": {
            "catalogo": "Hortense",
            "nombre": "Eugene Spinka MD",
            "programa": 2,
            "programas": {
              "id": 2,
              "nombre": "Licenciatura en Educación Infantil",
              "facultad_id": 2,
              "director": null,
              "facultad": {
                "id": 2,
                "nombre": "Humanidades y Educación",
                "decano": null,
                "coor_inv": null
              }
            }
          }
        }, ...
      ]
    }
  ]
}
```

> Petición sin resultados

```json
{
  "success": true,
  "mensaje": "Reporte generado con exito",
  "ruta": "REPORTE_VACIO",
  "data": []
}
```

Este servicio genera un reporte de los proyectos por programa.

### Petición HTTP

`POST http://localhost:8000/api/reportes/programa/{idPrograma}`

### Parametros

| Parametro        | Requerido | Descripción                                                                           |
| ---------------- | --------- | ------------------------------------------------------------------------------------- |
| idPrograma       | Si        | ID del programa a consultar.                                                          |
| estado           | No        | Arreglo de posibles estados (En propuesta, En Progreso, En Correcciones, Finalizado). |
| facultad         | No        | Arreglo con uno o más ID de facultades.                                               |
| programa         | No        | Arreglo con uno o más ID de programas.                                                |
| areaConocimiento | No        | Arreglo con uno o más ID de areas de conocimiento.                                    |
| nombre           | No        | Nombre del proyecto. Puede ser similar o exacto.                                      |

# Dashboard

## Valores para tarjetas

```shell
curl "http://localhost:8000/api/graficos/elementos-dashboard" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "creados": 17,
  "finalizados": 6,
  "aula": 4,
  "grado": 3,
  "semillero": 17,
  "convocatorias": 0,
  "inv_independientes": 0,
  "presupuesto": 17,
  "implicitos": 8,
  "tacitos": 9,
  "ranking": {
    "ingenieria": 33,
    "humanidades": 14,
    "juridicas": 1,
    "economicas": 0,
    "psicologia": 0
  }
}
```

Este servicio trae todos los valores que vemos en las tarjetas del tablero de control de Knowledge USB.

### Petición HTTP

`GET http://localhost:8000/api/graficos/elementos-dashboard`

## Datos graficos proyectos finalizados por facultad

```shell
curl "http://localhost:8000/api/graficos/datos-graficas-finalizados-facultad" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "datos": [
    {
      "facultad": "Psicología",
      "finalizados": 0,
      "no_finalizados": 0
    },
    {
      "facultad": "Humanidades y Educación",
      "finalizados": 4,
      "no_finalizados": 10
    },
    {
      "facultad": "Ingeniería",
      "finalizados": 13,
      "no_finalizados": 20
    },
    {
      "facultad": "Ciencias Economicas",
      "finalizados": 0,
      "no_finalizados": 0
    },
    {
      "facultad": "Ciencias Juridicas",
      "finalizados": 1,
      "no_finalizados": 3
    }
  ]
}
```

Este servicio trae valores numéricos correspondientes a la cantidad de proyectos finalizados y no finalizados por facultad.

### Petición HTTP

`GET http://localhost:8000/api/graficos/datos-graficas-finalizados-facultad`

## Datos graficos proyectos trabajo y semillero por facultad

```shell
curl "http://localhost:8000/api/graficos/datos-graficas-grado-semillero-facultad" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "datos": [
    {
      "facultad": "Psicología",
      "proyectos_grado": 0,
      "semilleros": 0
    },
    {
      "facultad": "Humanidades y Educación",
      "proyectos_grado": 4,
      "semilleros": 2
    },
    {
      "facultad": "Ingeniería",
      "proyectos_grado": 5,
      "semilleros": 14
    },
    {
      "facultad": "Ciencias Economicas",
      "proyectos_grado": 0,
      "semilleros": 0
    },
    {
      "facultad": "Ciencias Juridicas",
      "proyectos_grado": 0,
      "semilleros": 2
    }
  ]
}
```

Este servicio trae valores numéricos correspondientes a la cantidad de proyectos de grado y semillero por facultad.

### Petición HTTP

`GET http://localhost:8000/api/graficos/datos-graficas-grado-semillero-facultad`

## Datos graficos presupuesto por mes para proyectos

```shell
curl "http://localhost:8000/api/graficos/datos-graficas-presupuesto-proyectos-por-mes" \
    -H Authorization: Bearer <TOKEN_JWT>
```

> Petición con resultados:

```json
{
  "success": true,
  "datos": [
    {
      "presupuesto": 13690289,
      "fecha": "Septiembre 2022"
    },
    {
      "presupuesto": 60171148,
      "fecha": "Octubre 2022"
    },
    {
      "presupuesto": 21203642,
      "fecha": "Noviembre 2022"
    },
    {
      "presupuesto": 53073563,
      "fecha": "Diciembre 2022"
    },
    {
      "presupuesto": 110911023,
      "fecha": "Enero 2023"
    }
  ]
}
```

Este servicio trae valores numéricos correspondientes al presupuesto asignado a los proyectos por mes.

### Petición HTTP

`GET http://localhost:8000/api/graficos/datos-graficas-presupuesto-proyectos-por-mes`
