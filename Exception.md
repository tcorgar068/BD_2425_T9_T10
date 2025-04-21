## 🧱 Estructura básica del manejo de excepciones en PL/pgSQL

El manejo de excepciones se realiza dentro de un **bloque `BEGIN ... EXCEPTION ... END`**, que forma parte del bloque `DO`, `FUNCTION`, `PROCEDURE`, etc.

```sql
BEGIN
   -- Código que puede lanzar una excepción
EXCEPTION
   WHEN condición_de_error THEN
      -- Código a ejecutar cuando se produce la excepción
END;
```

---

## 🛑 Principales excepciones genéricas de PostgreSQL

Estas son algunas de las excepciones más comunes que puedes capturar:

| Excepción                  | Descripción |
|---------------------------|-------------|
| `division_by_zero`        | División entre cero. |
| `undefined_column`        | Columna no existe. |
| `undefined_table`         | Tabla no existe. |
| `raise_exception`         | Error general lanzado con `RAISE EXCEPTION`. |
| `unique_violation`        | Violación de restricción `UNIQUE`. |
| `foreign_key_violation`   | Violación de clave foránea. |
| `null_value_not_allowed`  | Valor `NULL` en campo `NOT NULL`. |
| `check_violation`         | Violación de una restricción `CHECK`. |
| `deadlock_detected`       | Se detectó un interbloqueo (deadlock). |
| `others`                  | Captura cualquier excepción no especificada. |

Puedes capturar múltiples excepciones con:

```sql
BEGIN
   -- código
EXCEPTION
   WHEN division_by_zero THEN
      RAISE NOTICE 'Error: división por cero';
   WHEN others THEN
      RAISE NOTICE 'Otro error: %', SQLERRM;
END;
```

---

## 🛠️ Cómo lanzar tus propios errores

Puedes crear errores personalizados usando el comando `RAISE`:

```sql
RAISE EXCEPTION 'Mensaje de error';
```

También puedes usar parámetros y códigos SQLSTATE personalizados:

```sql
RAISE EXCEPTION 'Error: el valor % no es válido', mi_variable
   USING ERRCODE = 'P0001';  -- códigos personalizados válidos: P0001-P9999
```

Opcionalmente puedes añadir:

- `MESSAGE`: mensaje del error.
- `DETAIL`: detalles adicionales.
- `HINT`: sugerencias al usuario.
- `ERRCODE`: código de error SQLSTATE.

Ejemplo:

```sql
RAISE EXCEPTION USING 
    MESSAGE = 'Valor fuera de rango', 
    DETAIL = 'El valor debe estar entre 1 y 100', 
    HINT = 'Revise los datos de entrada',
    ERRCODE = 'P0002';
```

---

## 💡 Variables útiles en el bloque EXCEPTION

Dentro del bloque `EXCEPTION`, puedes usar variables especiales:

- `SQLSTATE`: código del error SQL estándar (ej. `23505` para `unique_violation`)
- `SQLERRM`: mensaje del error

Ejemplo:

```sql
BEGIN
   -- código
EXCEPTION
   WHEN OTHERS THEN
      RAISE NOTICE 'Código de error: %, Mensaje: %', SQLSTATE, SQLERRM;
END;
```

---

## 🧪 Ejemplo completo

```sql
DO $$
DECLARE
    num1 INTEGER := 10;
    num2 INTEGER := 0;
    resultado INTEGER;
BEGIN
    resultado := num1 / num2;
    RAISE NOTICE 'Resultado: %', resultado;
EXCEPTION
    WHEN division_by_zero THEN
        RAISE EXCEPTION 'Intento de dividir por cero. num1=%, num2=%', num1, num2;
    WHEN others THEN
        RAISE NOTICE 'Error inesperado: %', SQLERRM;
END $$;
```

Perfecto, vamos a profundizar un poco más. Dividiré la explicación en tres partes:

---

## 🧩 1. Estructura de manejo de excepciones en PL/pgSQL

En PL/pgSQL, un bloque de código puede capturar excepciones mediante:

```sql
BEGIN
   -- Código que puede generar errores
EXCEPTION
   WHEN nombre_excepción THEN
      -- Código de recuperación
   WHEN OTHERS THEN
      -- Captura cualquier otra excepción no especificada
END;
```

Este bloque puede estar dentro de funciones, procedimientos o bloques `DO`.

---

## ⚠️ 2. Errores genéricos (excepciones del sistema)

PostgreSQL tiene una lista de errores predefinidos con nombres simbólicos, como `division_by_zero`, `unique_violation`, etc.

### 📌 Ejemplo: errores genéricos

Supongamos que tenemos una tabla con una restricción `UNIQUE` y tratamos de insertar un duplicado:

```sql
-- Creamos una tabla con restricción UNIQUE
DROP TABLE IF EXISTS empleados;
CREATE TABLE empleados (
    id SERIAL PRIMARY KEY,
    dni VARCHAR(10) UNIQUE NOT NULL,
    nombre TEXT NOT NULL
);

-- Insertamos un valor inicial
INSERT INTO empleados (dni, nombre) VALUES ('12345678A', 'Ana López');
```

Ahora un bloque PL/pgSQL que maneje errores:

```sql
DO $$
DECLARE
    v_dni empleados.dni%TYPE := '12345678A'; -- ya existe
    v_nombre empleados.nombre%TYPE := 'Carlos Ruiz';
BEGIN
    INSERT INTO empleados (dni, nombre) VALUES (v_dni, v_nombre);
    RAISE NOTICE 'Empleado insertado correctamente.';
EXCEPTION
    WHEN unique_violation THEN
        RAISE NOTICE 'Error: El DNI % ya está registrado.', v_dni;
    WHEN others THEN
        RAISE NOTICE 'Otro error: % - Código SQLSTATE: %', SQLERRM, SQLSTATE;
END $$;
```

### Explicación:

- **`unique_violation`** es una excepción específica para la violación de claves únicas.
- **`others`** es un comodín para capturar cualquier error no listado.

---

## 🚨 3. Creación de errores propios (personalizados)

Puedes lanzar tus propias excepciones con el comando `RAISE EXCEPTION`. Además, puedes adjuntar:

- `MESSAGE`: mensaje principal
- `DETAIL`: más información
- `HINT`: consejos para solucionar
- `ERRCODE`: código SQLSTATE personalizado (los personalizados empiezan por `P`)

### 📌 Ejemplo: lanzar errores propios con validación de datos

```sql
DO $$
DECLARE
    edad_usuario INTEGER := -5;
BEGIN
    IF edad_usuario < 0 THEN
        RAISE EXCEPTION 
            USING 
                MESSAGE = 'La edad no puede ser negativa.',
                DETAIL = 'Valor recibido: ' || edad_usuario,
                HINT = 'Verifica el valor ingresado en la variable edad_usuario.',
                ERRCODE = 'P0001';
    END IF;
    RAISE NOTICE 'Edad válida: %', edad_usuario;
EXCEPTION
    WHEN others THEN
        RAISE NOTICE 'ERROR PERSONALIZADO: % (%).', SQLERRM, SQLSTATE;
END $$;
```

### Resultado:

```
ERROR PERSONALIZADO: La edad no puede ser negativa. (P0001).
```

### 📝 Notas:

- `ERRCODE = 'P0001'` define un código de error personalizado.
- Puedes lanzar este tipo de errores cuando se incumple una regla de negocio.

---

## 🧪 Ejemplo combinado: errores del sistema y errores personalizados

```sql
DO $$
DECLARE
    v_num1 INTEGER := 10;
    v_num2 INTEGER := 0;
    resultado INTEGER;
BEGIN
    IF v_num2 = 0 THEN
        RAISE EXCEPTION 
            USING MESSAGE = 'División por cero prohibida.',
                  DETAIL = 'Intentaste dividir ' || v_num1 || ' entre ' || v_num2,
                  HINT = 'Asegúrate de que el divisor no sea cero.',
                  ERRCODE = 'P1000';
    END IF;

    resultado := v_num1 / v_num2;
    RAISE NOTICE 'Resultado: %', resultado;

EXCEPTION
    WHEN division_by_zero THEN
        RAISE NOTICE 'ERROR GENÉRICO: División por cero.';
    WHEN OTHERS THEN
        RAISE NOTICE 'ERROR CAPTURADO: % (%).', SQLERRM, SQLSTATE;
END $$;
```

Este bloque:

1. Lanza un error **personalizado** si detecta división por cero *antes* de hacer la operación.
2. Aun así, el `division_by_zero` se captura por si el control no fuera suficiente.
3. Y `others` sirve como red de seguridad.
