# test-manager-library-3145555-docs

> **Prueba técnica · Ficha ADSO 3145555**
> Horario: **miércoles 14 de octubre de 2026, de 6:50 a. m. a 11:00 a. m.** (hora Colombia). Entrega a las **11:00 a. m.**

Este repositorio contiene el **SDD** de *Manager Library* en tres variantes coordinadas: Python, .NET y el stack del reto, PHP/Laravel con React. Las reglas de producto y el contrato son comunes; cada plan y arquitectura traduce esos requisitos a su stack.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer la variante apropiada del spec en [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs): `spec-php/` para el reto; `spec-python/` y `spec-.net/` como referencias tecnológicas del mismo producto.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El reto se implementa en React y PHP/Laravel. Las variantes Python y .NET son referencias; [`spec-php/`](spec-php/) describe el desarrollo del stack que se entrega:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Las tres variantes conservan las mismas historias, criterios de aceptación, reglas, contrato HTTP y modelo de datos. Solo cambian las decisiones de implementación necesarias para cada stack.

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
| [`spec-php/`](spec-php/) | SDD para el stack del reto: API PHP/Laravel y SPA React |
| [`spec-python/`](spec-python/) | El mismo producto como referencia de implementación Python |
| [`spec-.net/`](spec-.net/) | El mismo producto como referencia de implementación .NET |

Las tres versiones describen el **mismo producto** (las mismas historias, reglas de negocio, modelo y endpoints). `spec-php/` es el destino del reto; Python y .NET sirven como referencias comparativas.

### Paridad entre variantes

`spec.md`, `data-model.md`, `api-contract.md` y los cuatro ADR deben conservar el mismo contenido en las tres carpetas. Las tareas deben cubrir los mismos criterios y flujos funcionales, aunque su descomposición, comandos y evidencia técnica pueden adaptarse al stack. `constitution.md`, `plan.md` y los documentos de arquitectura pueden diferir en rutas de proyecto, framework y herramientas de verificación, pero no en garantías de negocio ni en el contrato compartido.

Todo cambio que afecte al producto o al contrato se aplica a las tres variantes en el mismo cambio y se revisa en paralelo. Las diferencias restantes deben explicarse por tecnología. No se crea un artefacto adicional que duplique requisitos o endpoints: los documentos compartidos de cada variante siguen siendo las fuentes de verdad.

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

En cualquiera de las tres carpetas, en este orden:

1. `constitution.md` — principios innegociables.
2. `spec.md` — qué debe hacer el sistema: actores, historias, criterios de aceptación, reglas de negocio.
3. `plan.md`, `architecture.md` y `arquitectura-panoramica.md` — cómo se construye.
4. `data-model.md` y `api-contract.md` — el modelo de datos y el contrato de la API.
5. `tasks.md` — qué hay que hacer y con qué evidencia se da por hecho.
6. `adr/` — las decisiones de arquitectura, con sus alternativas y consecuencias.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-manager-library-3145555-docs`](https://github.com/code-sena/test-manager-library-3145555-docs) | El SDD: `spec-php/`, `spec-python/` y `spec-.net/` |
| [`test-manager-library-3145555-api`](https://github.com/code-sena/test-manager-library-3145555-api) | Backend en PHP (Laravel) |
| [`test-manager-library-3145555-app`](https://github.com/code-sena/test-manager-library-3145555-app) | Frontend en React |
| [`test-manager-library-3145555-page`](https://github.com/code-sena/test-manager-library-3145555-page) | Sitio público estático de presentación |
| [`test-manager-library-3145555-infra`](https://github.com/code-sena/test-manager-library-3145555-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-manager-library-3145555-tool`](https://github.com/code-sena/test-manager-library-3145555-tool) | Utilidades: sembrador de datos de demostración |
