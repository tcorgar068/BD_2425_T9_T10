# **Lenguaje de Programación de PostgreSQL: PL/pgSQL**
El **PL/pgSQL** (Procedural Language/PostgreSQL) es el lenguaje de programación interno de PostgreSQL. Se usa para crear funciones, procedimientos almacenados, disparadores (**triggers**) y bloques anónimos dentro de la base de datos.

---

## **📌 1️⃣ Características de PL/pgSQL**
✅ Permite **control de flujo** (`IF`, `LOOP`, `WHILE`, etc.).  
✅ Soporta **variables** y **tipos de datos** nativos de PostgreSQL.  
✅ Permite **manejo de errores** con `EXCEPTION`.  
✅ Puede realizar consultas SQL dentro del código.  
✅ Compatible con **funciones y procedimientos almacenados**.  

---

# **📌 2️⃣ Declaración de Variables**
Las variables en PL/pgSQL se declaran con `DECLARE` dentro de un bloque `DO $$` o una función.

### **Ejemplo: Declaración de variables y asignación**
```sql
DO $$ 
DECLARE 
    v_salario NUMERIC(10,2) := 3000.50;
    v_empleado_id INT := 1001;
BEGIN
    RAISE NOTICE 'Empleado ID: %, Salario: %', v_empleado_id, v_salario;
END $$;
```
💡 **Explicación**:
- `DECLARE` define variables (`v_salario`, `v_empleado_id`).
- `RAISE NOTICE` muestra mensajes en la consola.

---

# **📌 3️⃣ Bloques Anónimos (`DO $$`)**
PL/pgSQL permite ejecutar código sin necesidad de una función.

### **Ejemplo: Bloque anónimo con `DO $$`**
```sql
DO $$ 
DECLARE 
    v_nombre TEXT := 'PostgreSQL';
BEGIN
    RAISE NOTICE 'Hola, %!', v_nombre;
END $$;
```
💡 **Explicación**:
- `DO $$` ejecuta código procedural sin necesidad de crear una función.
- `RAISE NOTICE` muestra un mensaje con el valor de `v_nombre`.

---

# **📌 4️⃣ Funciones en PL/pgSQL**
Las funciones devuelven un valor y se definen con `CREATE FUNCTION`.

### **Ejemplo: Función que devuelve el salario de un empleado**
```sql
CREATE OR REPLACE FUNCTION obtener_salario(emp_id INT) RETURNS NUMERIC AS $$
DECLARE
    v_salario NUMERIC;
BEGIN
    SELECT SALARIO INTO v_salario FROM EMPLE WHERE EMP_NO = emp_id;
    RETURN v_salario;
END $$ LANGUAGE plpgsql;
```
### **Ejecutar la función**
```sql
SELECT obtener_salario(1001);
```
💡 **Explicación**:
- `CREATE FUNCTION` crea una función `obtener_salario(emp_id INT)`.
- `SELECT ... INTO` almacena el salario del empleado en `v_salario`.
- `RETURN v_salario` devuelve el salario.

---

# **📌 5️⃣ Procedimientos almacenados (`PROCEDURE`)**
A diferencia de las funciones, **los procedimientos no devuelven valores**, pero pueden modificar datos.

### **Ejemplo: Procedimiento para actualizar salarios**
```sql
CREATE OR REPLACE PROCEDURE actualizar_salario(p_emp_id INT, p_incremento NUMERIC) AS $$
BEGIN
    UPDATE EMPLE SET SALARIO = SALARIO + p_incremento WHERE EMP_NO = p_emp_id;
END $$ LANGUAGE plpgsql;
```
### **Ejecutar el procedimiento**
```sql
CALL actualizar_salario(1001, 500);
```
💡 **Explicación**:
- `CREATE PROCEDURE` define un procedimiento `actualizar_salario(emp_id, incremento)`.
- `UPDATE EMPLE` aumenta el salario del empleado.

---

# **📌 6️⃣ Estructuras de Control**
PL/pgSQL soporta `IF`, `CASE`, `LOOPS`, `WHILE`, etc.

## **➤ 6.1 IF-THEN-ELSE**
```sql
DO $$ 
DECLARE 
    v_salario NUMERIC := 2500;
BEGIN
    IF v_salario < 3000 THEN
        RAISE NOTICE 'El salario es bajo.';
    ELSE
        RAISE NOTICE 'El salario es suficiente.';
    END IF;
END $$;
```
💡 **Explicación**:
- Evalúa si `v_salario < 3000` y muestra un mensaje.

---

## **➤ 6.2 CASE-WHEN**
```sql
DO $$ 
DECLARE 
    v_puesto TEXT := 'PROGRAMADOR';
BEGIN
    CASE v_puesto
        WHEN 'GERENTE' THEN RAISE NOTICE 'Es un gerente';
        WHEN 'PROGRAMADOR' THEN RAISE NOTICE 'Es un programador';
        ELSE RAISE NOTICE 'Puesto desconocido';
    END CASE;
END $$;
```
💡 **Explicación**:
- `CASE` evalúa `v_puesto` y muestra un mensaje dependiendo del valor.

---

## **➤ 6.3 LOOP (Ciclo infinito con `EXIT`)**
```sql
DO $$ 
DECLARE 
    i INT := 1;
BEGIN
    LOOP
        RAISE NOTICE 'Iteración %', i;
        i := i + 1;
        EXIT WHEN i > 5;
    END LOOP;
END $$;
```
💡 **Explicación**:
- Ejecuta un bucle **hasta que `i > 5`**.

---

## **➤ 6.4 WHILE (Repetir mientras se cumpla una condición)**
```sql
DO $$ 
DECLARE 
    i INT := 1;
BEGIN
    WHILE i <= 5 LOOP
        RAISE NOTICE 'Iteración %', i;
        i := i + 1;
    END LOOP;
END $$;
```
💡 **Explicación**:
- `WHILE` ejecuta el bloque **hasta que `i` sea mayor a 5**.

---

## **📌 7️⃣ Cursores (Recorrer Resultados)**
Los cursores permiten **iterar sobre resultados de una consulta**.

### **Ejemplo: Cursor para recorrer empleados**
```sql
DO $$ 
DECLARE 
    cur CURSOR FOR SELECT EMP_NO, APELLIDO FROM EMPLE;
    v_emp_no INT;
    v_apellido TEXT;
BEGIN
    OPEN cur;
    LOOP
        FETCH cur INTO v_emp_no, v_apellido;
        EXIT WHEN NOT FOUND;
        RAISE NOTICE 'Empleado: % - %', v_emp_no, v_apellido;
    END LOOP;
    CLOSE cur;
END $$;
```
💡 **Explicación**:
- `CURSOR FOR` define un cursor con empleados.
- `FETCH` extrae datos fila por fila.
- `EXIT WHEN NOT FOUND;` termina cuando no haya más filas.

---

## **📌 8️⃣ Manejo de Errores (`EXCEPTION`)**
PL/pgSQL permite capturar errores y manejar excepciones.

### **Ejemplo: Manejo de error en una división por cero**
```sql
DO $$ 
DECLARE 
    resultado NUMERIC;
BEGIN
    BEGIN
        resultado := 10 / 0;  -- Esto generará un error
    EXCEPTION
        WHEN division_by_zero THEN
            RAISE NOTICE 'Error: División por cero!';
    END;
END $$;
```
💡 **Explicación**:
- Captura el error `division_by_zero` y muestra un mensaje en `RAISE NOTICE`.

---

## **📌 9️⃣ Triggers (Disparadores)**
Los **triggers** ejecutan código automáticamente antes o después de una acción (`INSERT`, `UPDATE`, `DELETE`).

### **Ejemplo: Trigger que evita salarios menores a 1000**
```sql
CREATE OR REPLACE FUNCTION validar_salario() RETURNS TRIGGER AS $$
BEGIN
    IF NEW.SALARIO < 1000 THEN
        RAISE EXCEPTION 'El salario no puede ser menor a 1000';
    END IF;
    RETURN NEW;
END $$ LANGUAGE plpgsql;
```

```sql
CREATE TRIGGER trg_validar_salario
BEFORE INSERT OR UPDATE ON EMPLE
FOR EACH ROW
EXECUTE FUNCTION validar_salario();
```
💡 **Explicación**:
- `BEFORE INSERT OR UPDATE` verifica **antes de insertar o actualizar**.
- `RAISE EXCEPTION` impide salarios menores a `1000`.

---

# **📌 Resumen**
✅ **PL/pgSQL** permite programar dentro de PostgreSQL con **variables, control de flujo y funciones**.  
✅ Se pueden crear **funciones, procedimientos, cursores y triggers**.  
✅ Soporta **manejo de errores con `EXCEPTION`**.  
