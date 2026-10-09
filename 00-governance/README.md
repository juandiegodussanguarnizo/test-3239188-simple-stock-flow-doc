Propósito

Esta carpeta contiene las reglas generales para organizar, documentar y mantener el proyecto.

Fuente de referencia

El archivo spec/data-model.md es la fuente principal para reconstruir la documentación del sistema.

Reglas de documentación

Cada afirmación debe estar respaldada por una sección del modelo de datos.

Las afirmaciones no respaldadas por la fuente deben identificarse como supuestos.

No se deben inventar funcionalidades, entidades ni reglas de negocio.

La documentación debe ser coherente entre todas las carpetas.

Las reglas implementadas en PostgreSQL deben distinguirse de las reglas aplicadas únicamente en el dominio y de las tareas pendientes.

Estructura documental

00-governance: gobierno y convenciones.

01-context: contexto y alcance.

02-domain: entidades, reglas y lenguaje del negocio.

03-product: problema, visión y objetivos.

04-requirements: requisitos funcionales y no funcionales.

05-architecture: arquitectura, componentes y decisiones técnicas.
