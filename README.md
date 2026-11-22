# Práctica: Préstamo de equipo en Odoo (`prestamo_unicah`)

- **Estudiante:** Cristian Bonilla
- **Número de cuenta:** 0801200600485

## Respuestas

### Paso 1. Catálogo de equipos
1. **¿Qué pasa con un equipo cuando se desmarca el campo Activo? ¿Por qué es mejor que borrarlo?**
   El registro se archiva: deja de mostrarse en las listas y búsquedas predeterminadas de Odoo, pero permanece guardado en la base de datos. Es mejor que borrarlo porque conserva la integridad referencial y el historial de los préstamos anteriores en los que se utilizó ese equipo.
2. **¿Para qué sirve `copy=False` en el campo `codigo`?**
   Evita que el código de inventario se copie automáticamente cuando un usuario duplica un equipo desde la interfaz, previniendo códigos duplicados en el catálogo.

### Paso 2. Registro de préstamos
1. **Este modelo usa `_name` y `_inherit` al mismo tiempo. ¿Qué hace cada uno aquí?**
   `_name = "prestamo.prestamo"` crea un modelo nuevo con su propia tabla en PostgreSQL (`prestamo_prestamo`), mientras que `_inherit = ["mail.thread"]` incorpora por herencia los campos y métodos del *chatter* para registrar mensajes e historial de cambios sin crear un modelo distinto.
2. **¿Qué hace `tracking=True` en el campo `state`?**
   Registra automáticamente en el *chatter* cada cambio de estado del préstamo, indicando el estado anterior, el estado nuevo, el usuario y la fecha y hora.

### Paso 3. Equipos del préstamo
1. **En `fields.One2many("prestamo.linea", "prestamo_id")`, ¿qué indica cada parámetro?**
   `"prestamo.linea"` indica el modelo hijo donde se guardan las líneas, y `"prestamo_id"` indica el campo `Many2one` en ese modelo hijo que apunta al registro padre (`prestamo.prestamo`).
2. **¿Qué pasa con las líneas cuando se borra un préstamo? ¿Qué parte del código lo define?**
   Todas las líneas asociadas se eliminan automáticamente en cascada junto con el préstamo. Lo define el parámetro `ondelete="cascade"` en el campo `prestamo_id` de `prestamo.linea`.

### Paso 4. Flujo de estados
1. **Los botones ya se ocultan según el estado. ¿Por qué los métodos vuelven a validar el estado?**
   Porque ocultar un botón en XML solo restringe la interfaz visual; un usuario o llamada externa vía RPC podría ejecutar el método directamente o desde una pestaña desactualizada. Validar en Python garantiza que la regla de negocio se cumpla siempre en el servidor.
2. **¿Qué hace `prestamo.linea_ids.equipo_id.write(...)`? ¿Por qué no hace falta un `for` para cada equipo?**
   Actualiza el campo `estado` de todos los equipos incluidos en las líneas en una sola consulta SQL. No hace falta un bucle `for` porque en Odoo el acceso encadenado devuelve un *recordset* con todos los equipos y `.write()` opera sobre conjuntos de registros.

### Paso 5. Reglas del préstamo
1. **El campo `equipo_id` ya filtra los equipos disponibles. ¿Por qué hace falta el TODO 5?**
   Porque entre el momento en que se guarda el préstamo en estado *Borrador* y el momento en que se pulsa **Prestar**, otro usuario pudo haber prestado ese mismo equipo o enviado a mantenimiento; por ello se vuelve a verificar que siga disponible al confirmar la entrega.
2. **¿En qué momento se ejecuta un método con `@api.constrains`? ¿Y la validación de `action_prestar`?**
   Un método con `@api.constrains` se ejecuta automáticamente cada vez que se crea (`create`) o modifica (`write`) un registro en alguno de los campos indicados. En cambio, la validación de `action_prestar` se ejecuta únicamente cuando se presiona el botón **Prestar**.

### Paso 6. Contacto, kanban y búsqueda
1. **¿Qué pasa si arrastra una tarjeta del kanban a otra columna? ¿Por qué?**
   El campo `estado` del equipo cambia automáticamente al valor de la columna destino porque la vista kanban está agrupada por ese campo (`default_group_by="estado"`).
2. **El campo `prestamo_ids` se agregó a `res.partner`. ¿Se crea una columna nueva en la tabla de contactos? Explique.**
   No se crea ninguna columna nueva en la tabla `res_partner`, porque `One2many` es un campo relacional inverso virtual; la llave foránea física (`partner_id`) reside en la tabla `prestamo_prestamo`.