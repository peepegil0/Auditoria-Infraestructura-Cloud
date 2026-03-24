# Catálogo Cloud con XML y XSD

## Descripción del proyecto

Este proyecto consiste en la refactorización del esquema XSD del catálogo de servidores de TechNova Cloud y su archivo XML asociado. Partiendo de una versión inicial con validaciones RegEx básicas, el objetivo ha sido evolucionar el esquema aplicando los indicadores avanzados de XSD: agrupaciones reutilizables, control de cardinalidad, elementos excluyentes, orden flexible y garantía de unicidad.

El resultado es un esquema más limpio, más fácil de mantener y más robusto frente a errores de introducción de datos.

## Estructura del repositorio

El repositorio contiene los siguientes archivos:

* **CatalogoCloud_v2.xml** → Archivo XML con el catálogo de servidores refactorizado.
* **CatalogoCloud_v2.xsd** → Esquema XSD que define la estructura y las reglas de validación.
* **README.md** → Documento explicativo del proyecto.
  
## Misiones aplicadas

### Misión 1 — attributeGroup (AtributosServidor)

Los tres atributos del elemento `<servidor>` (`id`, `rack` y `estado`) se han extraído a un `<xs:attributeGroup name="AtributosServidor">` en la raíz del XSD. Desde el elemento `<servidor>` se llama con una sola línea mediante `ref="AtributosServidor"`. Esto evita repetir la declaración de los tres atributos cada vez que se reutilice el elemento.

### Misión 2 — group, sequence

Los elementos `cpu`, `ram`, `gpu` y `discos` se han agrupado en `<xs:group name="ComponentesHardware">`. El indicador `<xs:sequence>` obliga a que aparezcan siempre en ese orden exacto. El elemento `<hardware>` ya no los declara directamente, sino que los referencia con `<xs:group ref="ComponentesHardware" />`.

### Misión 3 — minOccurs y maxOccurs

La `<gpu>` tiene `minOccurs="0"` para hacerla opcional, ya que no todos los servidores disponen de tarjeta gráfica. Los `<disco>` tienen `maxOccurs="8"`, limitando el número máximo de discos por servidor a ocho.

### Misión 4 — choice (elemento `<red>`)

Justo después de `<software>`, cada servidor incluye un elemento `<red>`. Mediante `<xs:choice>`, el validador obliga a que contenga O BIEN una `<ip_fija>` O BIEN un `<dhcp>` (con valor `si` o `no`). Si alguien intenta incluir los dos a la vez, el validador da error.

### Misión 5 — all (elemento `<auditoria>`)

Al final de cada `<servidor>` se ha añadido un elemento `<auditoria>` con tres campos: `fecha_revision`, `tecnico` y `nota`. El indicador `<xs:all>` permite que esos tres campos aparezcan en cualquier orden dentro del XML. Los tres siguen siendo obligatorios.

### Misión 6 — unique (UnicoID)

Se ha añadido la restricción `<xs:unique name="UnicoID">` al final del elemento raíz `<catalogo_cloud>`. Esta restricción recorre todos los elementos `<servidor>` del documento y garantiza que ningún atributo `@id` se repita, aunque su formato sea válido según la RegEx.

### ¿Cuántas líneas de código habéis ahorrado al usar grupos?

El grupo `ComponentesHardware` ocupa **72 líneas** en su definición. Sin este grupo, esas 72 líneas tendrían que estar declaradas directamente dentro del elemento `<hardware>`. Al usar el grupo, se reduce a **1 sola línea** (`<xs:group ref="ComponentesHardware" />`), lo que supone un **ahorro de 71 líneas** en el código.

De forma similar, el `attributeGroup` `AtributosServidor` agrupa **3 atributos** en un bloque de **5 líneas** y los sustituye por una única línea de referencia dentro del `<servidor>`. Si el esquema tuviera varios elementos que compartiesen esos atributos, el ahorro se multiplicaría por cada uso adicional.

En total, el uso de grupos en este proyecto ha permitido ahorrar aproximadamente **72 líneas** de código repetido, haciendo el esquema más limpio y fácil de mantener.

### ¿Qué error da VS Code si intentáis poner dos servidores con el mismo ID?

Al intentar validar un XML que contiene dos servidores con el mismo `id` (por ejemplo, dos veces `srv-web-01`), da error.

Esto ocurre porque la restricción `<xs:unique name="UnicoID">` sirve como un portero que recorre todos los atributos `@id` del documento tras aplicar la validación de formato. Aunque la RegEx acepte el valor `srv-web-01` como correcto en ambos casos, el `unique` detecta que el mismo valor aparece dos veces y lanza el error de duplicado. El formato puede ser válido y aun así fallar la validación de unicidad: son dos capas de control independientes.

## Tecnologías utilizadas

* **XML** para almacenar y estructurar los datos del catálogo de servidores.
* **XSD**  para validar la estructura, los tipos y las reglas.
