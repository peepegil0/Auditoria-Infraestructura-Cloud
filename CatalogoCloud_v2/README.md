# Catálogo Cloud con XML y XSD

## Descripción del proyecto

Este proyecto consiste en la creación de un archivo XML que contiene el catálogo de servidores de la empresa ficticia TechNova Cloud, organizado por centros de datos, y un archivo XSD que se encarga de validar que todos los datos tengan el formato y la estructura correctos.

El objetivo es modelar un escenario real de infraestructura cloud aplicando restricciones avanzadas de XSD: expresiones regulares para validar formatos, agrupaciones reutilizables, control de cardinalidad, elementos excluyentes y unicidad de identificadores.

## Estructura del repositorio

El repositorio contiene los siguientes archivos:

* **CatalogoCloud_v2.xml** → Archivo XML con el catálogo de servidores de TechNova Cloud.
* **CatalogoCloud_v2.xsd** → Esquema XSD que define la estructura y las reglas de validación.
* **README.md** → Documento explicativo del proyecto.

## Validaciones realizadas

### Expresiones regulares (pattern)

En el archivo XSD se utilizan **expresiones regulares** para validar el formato de varios campos:

* **ID del servidor**: se valida con `srv-[a-z]+-[0-9]{2}`, obligando a que comience por `srv-`, seguido de letras minúsculas, un guion y exactamente dos dígitos. Ejemplos válidos: `srv-web-01`, `srv-backup-03`.
* **Zona geográfica**: se valida con `[a-z]{2}-[a-z]+-\d+`, siguiendo el estándar de regiones cloud. Ejemplos válidos: `eu-sur-1`, `us-east-2`.
* **Rack**: se valida con `[A-Z]\d{1,2}`, exigiendo una letra mayúscula seguida de uno o dos dígitos. Ejemplos válidos: `A1`, `B12`.
* **Versión del sistema operativo**: se valida con `\d+(\.\d+)*`, permitiendo versiones simples o con puntos. Ejemplos válidos: `22.04`, `9`, `15`.
* **Estado del servidor**: se valida con el patrón `activo|mantenimiento|apagado`, limitando los valores únicamente a esas tres opciones.
* **Unidad de RAM**: se valida con `MB|GB|TB`, permitiendo solo esas tres unidades de medida como atributo.
* **Tipo de disco**: se valida con `NVMe|SSD|HDD`, asegurando que solo se usen tipos de almacenamiento reconocidos.
* **DHCP**: se valida con `si|no`, limitando el valor del elemento a esas dos opciones.

### Agrupaciones y estructura avanzada

Además de las expresiones regulares, el XSD aplica varias técnicas de estructuración:

* **`attributeGroup` (AtributosServidor)**: los atributos `id`, `rack` y `estado` del elemento `<servidor>` se han extraído a un grupo reutilizable, evitando repetición de código en el esquema.
* **`group` (ComponentesHardware)**: los elementos `cpu`, `ram`, `gpu` y `discos` se han agrupado en un bloque reutilizable con `xs:sequence`, garantizando que siempre aparezcan en ese orden exacto.
* **`minOccurs` y `maxOccurs`**: la `<gpu>` tiene `minOccurs="0"` para hacerla opcional, ya que no todos los servidores disponen de tarjeta gráfica. Los `<disco>` tienen un máximo de 8 por servidor (`maxOccurs="8"`).
* **`choice` (elemento `<red>`)**: cada servidor debe tener O BIEN una `<ip_fija>` O BIEN un `<dhcp>`. El validador rechaza cualquier XML que intente incluir los dos a la vez.
* **`all` (elemento `<auditoria>`)**: los tres campos de auditoría (`fecha_revision`, `tecnico` y `nota`) pueden aparecer en cualquier orden dentro del XML, pero los tres son obligatorios.
* **`unique` (UnicoID)**: garantiza que ningún atributo `@id` se repita en todo el documento, añadiendo una segunda capa de protección más allá del formato.

## Tecnologías utilizadas

* **XML** para almacenar y estructurar los datos del catálogo de servidores.
* **XSD** (XML Schema Definition) para definir la estructura, los tipos y las reglas de validación.
* **Expresiones regulares**, comprobadas a través de https://regex101.com/

## Bibliografía

* https://www.w3schools.com/xml/schema_intro.asp
* https://www.w3schools.com/xml/schema_complex_indicators.asp
* https://regex101.com/
* https://www.w3.org/TR/xmlschema-0/
