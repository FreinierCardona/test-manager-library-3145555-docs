# test-manager-library-3145555-docs

> **Prueba técnica · Ficha ADSO 3145555**
> Horario: **miércoles 14 de octubre de 2026, de 6:50 a. m. a 11:00 a. m.** (hora Colombia). Entrega a las **11:00 a. m.**

Este repositorio contiene el **spec** de *Manager Library*. Es el único con contenido: los otros cinco empiezan vacíos.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Entrega

Cuenta lo que quede **subido a los forks antes de las 11:00 a. m.** Lo que llegue después no se revisa.
No hay prórroga: suban avances parciales con commits pequeños durante la prueba.

## Qué contiene este repositorio

| Carpeta | Contenido |
|---|---|
| [`spec-python/`](spec-python/) | El spec de Manager Library escrito para un backend Python |
| [`spec-.net/`](spec-.net/) | El mismo sistema especificado para un backend .NET |

Las dos versiones describen el **mismo producto** (las mismas historias, reglas de negocio y
endpoints); cambian las decisiones de tecnología. Ninguna de las dos es el stack del reto.

### Paridad entre variantes

`spec.md`, `data-model.md`, `api-contract.md` y los cuatro ADR deben conservar el mismo contenido en ambas carpetas. Las reglas funcionales y la evidencia de comportamiento de `tasks.md` también deben coincidir; solo pueden variar los comandos, frameworks y herramientas específicos de cada stack. `constitution.md`, `plan.md` y los documentos de arquitectura pueden diferir en decisiones técnicas, rutas de proyecto y herramientas de verificación, pero no en garantías de negocio ni en el contrato compartido.

Todo cambio que afecte al producto o al contrato se aplica a `spec-python/` y `spec-.net/` en el mismo cambio y se revisa en paralelo. Las diferencias restantes deben explicarse por tecnología. No se mantiene una tercera copia de requisitos o endpoints: estos documentos continúan siendo las fuentes de verdad.

### Glosario breve

| Término | Significado en estos documentos |
|---|---|
| DDL | Instrucciones SQL que crean o modifican el esquema de la base de datos |
| FK | Clave foránea que relaciona una fila con otra tabla |
| ORM | Herramienta que mapea objetos del programa a tablas y consultas de base de datos |
| JWT / HS256 | Token con claims firmado mediante HMAC-SHA-256; su contenido no está cifrado |
| CORS | Política del navegador que limita qué orígenes pueden llamar a la API |
| UTC | Referencia horaria común usada para almacenar y comparar instantes |
| Fake | Sustituto controlado de una dependencia externa para ejecutar pruebas sin ese servicio real |

## Cómo se lee el spec

En cualquiera de las dos carpetas, en este orden:

1. `constitution.md` — principios innegociables.
2. `spec.md` — qué debe hacer el sistema: actores, historias, criterios de aceptación, reglas de negocio.
3. `plan.md`, `architecture.md` y `arquitectura-panoramica.md` — cómo se construye.
4. `data-model.md` y `api-contract.md` — el modelo de datos y el contrato de la API.
5. `tasks.md` — qué hay que hacer y con qué evidencia se da por hecho.
6. `adr/` — las decisiones de arquitectura, con sus alternativas y consecuencias.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-manager-library-3145555-api`](https://github.com/code-sena/test-manager-library-3145555-api) | Backend en PHP (Laravel) |
| [`test-manager-library-3145555-app`](https://github.com/code-sena/test-manager-library-3145555-app) | Frontend en React |
| [`test-manager-library-3145555-page`](https://github.com/code-sena/test-manager-library-3145555-page) | Sitio público estático de presentación |
| [`test-manager-library-3145555-infra`](https://github.com/code-sena/test-manager-library-3145555-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-manager-library-3145555-tool`](https://github.com/code-sena/test-manager-library-3145555-tool) | Utilidades: sembrador de datos de demostración |
