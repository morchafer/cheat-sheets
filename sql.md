# SQL Cheat Sheet

## DDL (Data Definition Language)

### Create & Drop

- Crear una base de datos

  ```sql
    CREATE DATABASE nombreDB;
  ```

- Crear una base de datos IF NOT EXISTS

  ```sql
    CREATE DATABASE IF NOT EXISTS nombreDB;
  ```

- Eliminar una base de datos

  ```sql
    DROP DATABASE nombreDB;
  ```

- Eliminar una base de datos IF EXISTS

  ```sql
    DROP DATABASE IF EXISTS nombreDB;
  ```

- Cambiar el espacio de trabajo a otra base de datos

  ```sql
    USE nombreDB;
  ```

- Crear una tabla sin atributos

  ```sql
    CREATE TABLE clientes (
      clienteID INT,
      Nombre VARCHAR(30),
      Ciudad VARCHAR(30),
      Telefono VARCHAR(30)
    );
  ```

- Crear una tabla con atributos

  ```sql
    CREATE TABLE IF NOT EXISTS dbTemporal.productos (
      productoID INT NOT NULL PRIMARY KEY UNIQUE AUTO_INCREMENT,
      Nombre VARCHAR(40),
      Categoria VARCHAR(50),
      Precio  INT NOT NULL
    );
  ```

- Crear una tabla temporal (solo existirá durante la sesión activa)

  La siguiente sentencia crea una tabla temporal a partir de la tabla productos.

  Las tablas temporales no se muestran en el esquema de la base de datos.

  ```sql
    CREATE TEMPORARY TABLE tmpProductos SELECT * FROM productos;
  ```

- Crear una tabla a partir de otra, con la misma estructura de atributos pero sin los registros

  ```sql
    CREATE TABLE tblLikeCalientes LIKE clientes;
  ```

- Crear una tabla con constraints a nivel de columna

  ```sql
    CREATE TABLE empleados (
      empleadoID INT AUTO_INCREMENT UNIQUE NOT NULL PRIMARY KEY,
      Nombre VARCHAR(50) NOT NULL,
      Apellido VARCHAR(50) NOT NULL,
      Telefono CHAR(10) NOT NULL,
      Domicilio VARCHAR(50) NOT NULL
    );
  ```

- Crear una tabla con constraints a nivel de tabla (ejemplos empleados, productos y detalleDeVenta)

  ```sql
    CREATE TABLE empleados (
      empleadoID INT AUTO_INCREMENT UNIQUE NOT NULL,
      Nombre VARCHAR(50) NOT NULL,
      Apellido VARCHAR(50) NOT NULL,
      Telefono CHAR(10) NOT NULL,
      Domicilio VARCHAR(50) NOT NULL,
      CONSTRAINT pkEmpleados PRIMARY KEY (empleadoID),
      CONSTRAINT uqNombre UNIQUE (Nombre)
    );
  ```

  ```sql
    CREATE TABLE productos (
      productoID INT PRIMARY KEY NOT NULL UNIQUE AUTO_INCREMENT,
      Nombre VARCHAR(50) NOT NULL,
      precioVenta DECIMAL(6, 2) NOT NULL,
      Existencia BOOL NOT NULL DEFAULT FALSE
    );
  ```

  ```sql
    CREATE TABLE detalleDeVenta (
      ventaID INT PRIMARY KEY AUTO_INCREMENT UNIQUE NOT NULL,
      fechaVenta DATETIME NOT NULL,
      clienteID INT NOT NULL,
      productoID INT NOT NULL,
      cantidadVendida INT NOT NULL,
      precioVenta DECIMAL(6, 2) NOT NULL,

      CONSTRAINT fkDetalleVentaProductos
      FOREIGN KEY (productoID)
      REFERENCES productos(productoID)
      ON DELETE NO ACTION,

      CONSTRAINT fkDetalleVentaClientes
      FOREIGN KEY (clienteID)
      REFERENCES clientes(clienteID)
      ON DELETE NO ACTION
    );
  ```

### Alter (Add, Modify, Change, Drop)

- Mostrar la descripción de una tabla

  ```sql
    DESC clientes;
  ```

- Agregar una columna a una tabla

  ```sql
    ALTER TABLE clientes
    ADD Correo VARCHAR(30);
  ```

- Agregar una columna a una tabla en una parte de ella en específico

  ```sql
    ALTER TABLE clientes
    ADD Domicilio VARCHAR(50) AFTER Ciudad;
  ```

- Agregar una columna al principio de una tabla

  ```sql
    ALTER TABLE clientes
    ADD Apellido VARCHAR(30) FIRST;
  ```

- Borrar una columna de una tabla (no se puede eliminar una columna que vulnere la Integridad Referencial)

  ```sql
    ALTER TABLE clientes
    DROP COLUMN Apellido;
  ```

- Modificar la capacidad de un campo (no se puede modificar si el valor infringe los registros que ya tengan información)

  ```sql
    ALTER TABLE clientes
    MODIFY Nombre VARCHAR(50);
  ```

- Modificar el tipo de dato de un campo (error si el tipo de dato no coincide con registros existentes)

  ```sql
    ALTER TABLE clientes
    MODIFY Nombre CHAR(30) NULL;
  ```

- Modificar el valor por defecto cuando un registro se inserte

  ```sql
    ALTER TABLE clientes
    MODIFY Nombre VARCHAR(50) NOT NULL DEFAULT 'Fulanito de tal';
  ```

- Modificar el nombre de un campo

  ```sql
    ALTER TABLE clientes
    CHANGE COLUMN Correo Email VARCHAR(30);
  ```

- Agregar Primary Key a una tabla (el campo ya debe existir para asignarle Primary Key)

  ```sql
    ALTER TABLE tmpproductos
    ADD PRIMARY KEY (productoID);
  ```

- Agregar Foreign Key a una tabla

  ```sql
    ALTER TABLE tmpproductos
    ADD
      CONSTRAINT fkTempProdID
      FOREIGN KEY (tempProdID)
      REFERENCES productos(productoID)
      ON DELETE NO ACTION;
  ```

- Borrar la Primary Key de una tabla

  ```sql
    ALTER TABLE tmpproductos
    DROP PRIMARY KEY;
  ```

- Borrar la(s) Foreign Key de una tabla

  ```sql
    ALTER TABLE tmpproductos
    DROP FOREIGN KEY fkTempProdID;
  ```

- Renombrar una tabla

  ```sql
    RENAME TABLE clientes TO losclientes;
  ```

- Borrar toda la información de una tabla con TRUNCATE (la destruye y la vuelve a construir, por lo tanto reinicia el registro de IDs)

  ```sql
    TRUNCATE TABLE detalledeventa;
  ```

- Eliminar la tabla de la base de datos

  ```sql
    DROP TABLE tblclientes;
  ```

- Columnas generadas mediante campos calculados (valores dinámicos)

  _GENERATED ALWAYS & VIRTUAL_ son opciones por defecto.

  _VIRTUAL_ -> Este campo (por defecto) implica que en situaciones de INSERT o UPDATE el cálculo se hará de forma virtual, por lo tanto no será guardado en la DB y no ocupará espacio.

  _STORED_ -> A diferencia de VIRTUAL, STORED especifica que SÍ se guardará en la DB como un dato más.

  ```sql
    CREATE TABLE ejemploColumnaGenerada (
      ID INT(11) NOT NULL AUTO_INCREMENT,
      Cantidad DOUBLE NOT NULL,
      Precio DOUBLE NOT NULL,
      TotalSinDescuento DOUBLE AS (Cantidad  * Precio) NOT NULL,
      DiezPorCientoDcto DOUBLE GENERATED ALWAYS AS ((Cantidad * Precio) / 10) VIRTUAL NOT NULL,
      PRIMARY KEY (ID)
    );

    INSERT INTO ejemplocolumnagenerada (ID, Cantidad, Precio) VALUES (0, 10, 100);
    INSERT INTO ejemplocolumnagenerada (id, cantidad, PrEcio) VALUES (0, 20, 200);

    UPDATE ejemplocolumnagenerada
    SET Precio = 300
    WHERE ID = 1;

    SELECT * FROM ejemplocolumnagenerada;
  ```

- Mostrar Ruta Segura

  ```sql
    SHOW VARIABLES LIKE "secure_file_priv";
  ```

- Cargar archivos CSV

  ```sql
    LOAD DATA INFILE 'C:\\archivos\\tblPersonas.csv'
    INTO TABLE personas
    CHARACTER SET latin1
    FIELDS TERMINATED BY ','
    ENCLOSED BY '"'
    LINES TERMINATED BY '\r\n'
    IGNORE 1 ROWS;
  ```

## DML (Data Manipulation Language)

- Seleccionar todos los registros de una tabla

  ```sql
    SELECT * FROM personas;
  ```

- Seleccionar columnas específicas de una tabla

  ```sql
    SELECT nombre, correo FROM personas;
  ```

- Seleccionar columnas específicas de una tabla y ordenar alguna de ellas de forma descendente

  ```sql
    SELECT nombre, correo
    FROM personas
    ORDER BY nombre DESC;
  ```

- Seleccionar columnas específicas de una tabla y ordenar alguna de ellas de forma ascendente

  ```sql
    SELECT nombre, correo
    FROM personas
    ORDER BY nombre ASC;
  ```

- Seleccionar columnas específicas de una tabla con condición y ordenamiento ascendente

  Forma 1:

  ```sql
    SELECT fechaES, montoIE
    FROM entradasalidadinero
    WHERE fechaES >= "2018-01-01 00:00:00" && fechaES <= "2018-01-31 00:00:00"
    ORDER BY fechaES ASC;
  ```

  Forma 2:

  ```sql
    SELECT fechaES, montoIE
    FROM entradasalidadinero
    WHERE fechaES BETWEEN '2018/01/01' AND '2018/01/31'
    ORDER BY fechaES ASC;
  ```

- Seleccionar columnas específicas de una tabla con condición, operación y alias

  ```sql
    SELECT id, fechaES, montoIE, montoIE * 2 AS "montoX2"
    FROM entradasalidadinero
    WHERE montoIE >= 500.00
  ```

- Concatenación con CONCAT

  ```sql
    SELECT CONCAT(nombre, " || ", correo, " || ", NOW())
    AS "Nombre & Correo"
    FROM personas
  ```

- Función LEFT

  ```sql
    SELECT LEFT(nombre, 4)
    AS "Primeras 4 letras del nombre"
    FROM personas;
  ```

- Función DATE_FORMAT

  ```sql
    SELECT
      date_format(fechaES, "%d/%m/%y") AS "Fecha con formato dd/mm/yy",
      date_format(fechaES, "%e-%b-%Y") AS "Fecha con formato d-month-yyyy"
    FROM entradasalidadinero;
  ```

- Función ROUND

  ```sql
    SELECT round(montoIE + .56, 1) AS "Valor redondeado" FROM entradasalidadinero;
  ```

- Seleccionar valores distintos (únicos)

  ```sql
    SELECT DISTINCT year(fechaES) FROM entradasalidadinero;
  ```

- Cláusula WHERE

  ```sql
    SELECT * FROM personas WHERE right(correo, 11) = "hotmail.com";

    SELECT * FROM personas WHERE left(nombre, 1) < 'K' ORDER BY nombre ASC;

    SELECT * FROM personas WHERE id != 8 AND left(telefono, 3) = "493" AND id % 2 = 0;

    SELECT * FROM personas WHERE NOT left(telefono, 3) = '493';

    SELECT * FROM personas WHERE right(correo, 9) = "gmail.com" OR right(correo, 9) = "yahoo.com";

    SELECT * FROM personas WHERE nombre IN('JUAN PEREZ GARCIA', 'JESUS DE LA CUEVA RODARTE');

    SELECT * FROM personas WHERE telefono IS NULL;

    SELECT * FROM entradasalidadinero WHERE idPersonas IN
      (SELECT id FROM personas WHERE nombre IN ('JUAN PEREZ GARCIA', 'JESUS DE LA CUEVA RODARTE'));

    SELECT * FROM entradasalidadinero
    WHERE (fechaES BETWEEN "2018-01-01" AND "2018-12-31") AND (montoIE BETWEEN 100 AND 2000)
    ORDER BY montoIE ASC;

    SELECT * FROM personas
    WHERE left(nombre, 1) BETWEEN "A" AND "P"
    ORDER BY nombre ASC;
  ```

- Expresiones Regulares con REGEXP

  ```sql
    SELECT * FROM personas WHERE correo REGEXP "gmail"; -- Contiene

    SELECT * FROM employees WHERE first_name REGEXP "^Ma"; -- Inicia con

    SELECT * FROM employees WHERE first_name REGEXP "ta$"; -- Termina con

    SELECT * FROM employees WHERE first_name REGEXP "rs|sn"; -- Contiene una de ambas opciones

    SELECT * FROM employees WHERE first_name REGEXP "m[eo]"; -- Contiene la combinación de me o mo

    SELECT * FROM employees WHERE first_name REGEXP "n[a-f]"; -- Contiene la combinación de na, nb, nc, nd, ne o nf

    SELECT * FROM employees WHERE first_name REGEXP "mar[iy][ao]"; -- Contiene la combinación de mar seguido de i | y seguido de a | o

    SELECT * FROM employees WHERE first_name REGEXP "[a-z][aeiou]n$"; -- Termina con la combinación de cualquier letra seguido de una vocal y la letra n
  ```

- Expresiones Regulares con LIKE (LIKE no es case-sensitive)

  ```sql
    SELECT * FROM employees WHERE first_name LIKE "Man%"; -- Inicia con "Man"

    SELECT * FROM employees WHERE first_name LIKE "Ma__u%"; -- Inicia con la combinación de Ma seguido de dos caracteres cualquiera seguido de la letra u

    SELECT * FROM employees WHERE first_name LIKE "%ton"; -- Termina con "ton"

    SELECT * FROM employees WHERE first_name LIKE "%uan%"; -- Contiene la cadena "uan" en cualquier parte

    SELECT * FROM employees WHERE first_name LIKE "_____"; -- Busca por los first_names que contengan solo 5 caracteres
  ```

- Ordenamiento múltiple y con alias

  ```sql
    SELECT last_name, birth_date
    FROM employees
    ORDER BY last_name ASC, birth_date ASC;

    SELECT birth_date, concat(first_name, " ", last_name) AS "NombreCompleto"
    FROM employees
    ORDER BY NombreCompleto ASC;

    SELECT birth_date,
           concat(first_name, " ", last_name) AS "NombreCompleto",
           round(datediff(now(), birth_date)/365) AS Edad
    FROM employees
    ORDER BY Edad DESC;
  ```

- Establecer el campo a ordenar por su índice en vez de su nombre

  Este ejemplo ordena el campo con el índice 2 (NombreCompleto) en forma ascendente (valor por defecto)

  ```sql
    SELECT birth_date,
          concat(first_name, " ", last_name) AS "NombreCompleto",
          round(datediff(now(), birth_date)/365) AS Edad
    FROM employees
    ORDER BY 2;
  ```

- Ordenar por uno o varios valores en específico de un campo

  ```sql
    SELECT * FROM titles
    WHERE emp_no BETWEEN 10001 AND 10500
    ORDER BY field(title, "Engineer", "Senior Staff") DESC;
  ```

- Uso de la cláusula LIMIT

  Esta consulta retorna 6 filas a partir de la 4ta (se especifica 3 pero tal fila no es tomada en cuenta en el resultado).

  ```sql
    SELECT * FROM pendientes
    ORDER BY id ASC
    LIMIT 3, 6;
  ```

- INNER JOIN con dos tablas

  ```sql
    SELECT
      entradasalidadinero.fechaES,
      entradasalidadinero.montoIE,
      entradasalidadinero.idPersonas,
      personas.nombre
    FROM entradasalidadinero
    INNER JOIN personas ON entradasalidadinero.idPersonas = personas.id;
  ```

- INNER JOIN con varias tablas

  ```sql
    SELECT
        entradasalidadinero.fechaES AS FechaEntradaSalida,
        entradasalidadinero.montoIE AS MontoIngresoEgreso,
        entradasalidadinero.idPersonas AS PersonaID,
        personas.nombre AS NombrePersona,
        ingresosegresos.descripcion AS IngresosEgresosDesc,
        tipoingeg.descripcion AS Tipo,
        grupoingeg.observaciones AS Observaciones
    FROM entradasalidadinero
    INNER JOIN personas ON entradasalidadinero.idPersonas = personas.id
    INNER JOIN ingresosegresos ON entradasalidadinero.idIngresosEgresos = ingresosegresos.id
    INNER JOIN tipoingeg ON ingresosegresos.idTipoIngEg  = tipoingeg.id
    INNER JOIN grupoingeg ON ingresosegresos.idGrupoIngEg = grupoingeg.id;
  ```

- ALIAS en INNER JOIN

  ```sql
    SELECT
        esDinero.fechaES AS FechaEntradaSalida,
        esDinero.montoIE AS MontoIngresoEgreso,
        esDinero.idPersonas AS PersonaID,
        p.nombre AS NombrePersona,
        ie.descripcion AS IngresosEgresosDesc,
        ieType.descripcion AS Tipo,
        ieGroup.observaciones AS Observaciones
    FROM entradasalidadinero AS esDinero
    INNER JOIN personas AS p ON esDinero.idPersonas = p.id
    INNER JOIN ingresosegresos AS ie ON esDinero.idIngresosEgresos = ie.id
    INNER JOIN tipoingeg AS ieType ON ie.idTipoIngEg  = ieType.id
    INNER JOIN grupoingeg AS ieGroup ON ie.idGrupoIngEg = ieGroup.id;
  ```

- INNER JOIN con tablas entre diferentes bases de datos

  Las DBs deben estar en el mismo servidor y con acceso permitido a ellas.

  ```sql
    SELECT
        personas.nombre,
        personas.correo,
        personas.telefono,
        personas.id,
        employees.employees.emp_no,
        employees.employees.birth_date,
        employees.employees.gender,
        employees.salaries.salary
    FROM personas
    INNER JOIN employees.employees ON personas.id = employees.employees.emp_no
    INNER JOIN employees.salaries ON employees.employees.emp_no = employees.salaries.emp_no;
  ```

- Técnica Self Join - Unión de una tabla con ella misma (se necesita asignar un alias a ella misma)

  ```sql
    SELECT
        personas.idPersonasJefe,
        personas.nombre,
        personas.correo,
        personas.telefono
    FROM personas
    INNER JOIN personas AS pers ON personas.idPersonasJefe = pers.idPersonasJefe;
  ```

- LEFT Y RIGHT JOIN

  ```sql
    SELECT
        personas.id,
        personas.nombre,
        personas.correo,
        employees.employees.gender,
        employees.employees.birth_date
    FROM personas
    LEFT JOIN employees.employees ON personas.id = employees.employees.emp_no;

    SELECT
        personas.id,
        personas.nombre,
        personas.correo,
        employees.employees.gender,
        employees.employees.birth_date
    FROM personas
    RIGHT JOIN employees.employees ON personas.id = employees.employees.emp_no;
  ```

- Palabras reservadas USING & NATURAL

  _USING_ puede sustituir a _ON_ cuando el campo que relaciona ambas tablas se llama igual y tiene el mismo tipo de dato.

  _NATURAL_ toma las columnas que posean el mismo nombre entre dos tablas y las utiliza para realizar un join.

  ```sql
    SELECT
        employees.first_name,
        employees.last_name,
        titles.title,
        salaries.salary
    FROM employees
    INNER JOIN  titles USING(emp_no)
    INNER JOIN salaries USING(emp_no);

    SELECT
        employees.first_name,
        employees.last_name,
        titles.title,
        salaries.salary
    FROM employees
    NATURAL JOIN  titles
    INNER JOIN salaries USING(emp_no);
  ```

- Unir tablas de forma nativa. Sin utilizar INNER JOIN.

  ```sql
    SELECT
        entradasalidadinero.fechaES AS FechaEntradaSalida,
        entradasalidadinero.montoIE AS MontoIngresoEgreso,
        entradasalidadinero.idPersonas AS PersonaID,
        personas.nombre AS NombrePersona,
        ingresosegresos.descripcion AS IngresosEgresosDesc,
        tipoingeg.descripcion AS Tipo,
        grupoingeg.observaciones AS Observaciones
    FROM
        entradasalidadinero,
        personas,
        ingresosegresos,
        tipoingeg,
        grupoingeg
    WHERE
        entradasalidadinero.idPersonas = personas.id AND
        entradasalidadinero.idIngresosEgresos = ingresosegresos.id AND
        ingresosegresos.idTipoIngEg  = tipoingeg.id AND
        ingresosegresos.idGrupoIngEg = grupoingeg.id;
  ```

- Instrucción UNION para mostrar los resultados de dos consultas de diferentes tablas en un mismo Result Grid.

  ```sql
    SELECT id, nombre, correo, telefono FROM personas
    UNION ALL
    SELECT id, concat(nombre, ' ', apaterno, ' ', amaterno), correo, telefono FROM clientes
    ORDER BY id;
  ```

- Subqueries Escalares (aquellas que retornan un único registro de un solo campo)

  ```sql
    SELECT *
    FROM entradasalidadinero
    WHERE montoIE > (SELECT avg(montoIE) FROM entradasalidadinero);
  ```

- Subqueries De Listas (aquellas que retornan una lista de registros de un solo campo)

  ```sql
    SELECT idPersonas, idIngresosEgresos, montoIE
    FROM entradasalidadinero
    WHERE entradasalidadinero.idPersonas IN(SELECT id FROM personas WHERE personas.nombre LIKE '%Juan%');
  ```

- Palabra reservada ALL en Subqueries

  ```sql
    SELECT * FROM entradasalidadinero
    WHERE montoIE > ALL(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);

    SELECT * FROM entradasalidadinero
    WHERE montoIE < ALL(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);

    SELECT * FROM entradasalidadinero
    WHERE montoIE = ALL(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 3);

    SELECT * FROM entradasalidadinero
    WHERE montoIE != ALL(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);
  ```

- Palabra reservada ANY en Subqueries (funciona igual que SOME)

  ```sql
    SELECT * FROM entradasalidadinero
    WHERE montoIE > ANY(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);

    SELECT * FROM entradasalidadinero
    WHERE montoIE < ANY(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);

    SELECT * FROM entradasalidadinero
    WHERE montoIE = ANY(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 1);

    SELECT * FROM entradasalidadinero
    WHERE montoIE != ANY(SELECT montoIE FROM entradasalidadinero WHERE idIngresosEgresos = 3);
  ```

- Subqueries Correlacionadas (aquellas que se ejecutan una vez por cada registro de la consulta padre)

  ```sql
    SELECT * FROM entradasalidadinero AS esd
    WHERE montoIE <
      (SELECT AVG(montoIE) FROM entradasalidadinero
      WHERE idPersonas = esd.idPersonas);
  ```

- Subqueries Correlacionadas con EXISTS

  ```sql
    SELECT *
    FROM personas
    WHERE NOT EXISTS (SELECT * FROM usuarios WHERE personas.id = usuarios.idPersonas);

    SELECT *
    FROM personas
    WHERE NOT EXISTS (SELECT * FROM pendientes WHERE personas.id = pendientes.idPersonaQueAsigno);

    SELECT *
    FROM personas
    WHERE NOT EXISTS (SELECT * FROM entradasalidadinero WHERE personas.id = entradasalidadinero.idPersonas);
  ```

- Uso de Subqueries en el FROM

  ```sql
    SELECT  aux.idPersonas AS ID,
            usuarios.id AS UserID,
            aux.nombre AS NombreCompleto,
            usuarios.nombre AS NombreDeUsuario,
            aux.correo AS CorreoElectronico,
            SumaIE
    FROM (  SELECT
              idPersonas,
              nombre,
              correo,
              sum(montoIE) AS SumaIE
            FROM
              entradasalidadinero
            INNER JOIN
              personas
            ON
              entradasalidadinero.idPersonas = personas.id
            GROUP BY 1, 2
         ) AS aux
    INNER JOIN usuarios
    ON aux.idPersonas = usuarios.idPersonas AND usuarios.idPersonas IN(1, 2, 3);
  ```

- Uso de Subqueries con Técnica CTE (Common Table Expression)

  ```sql
    WITH aux AS
    (SELECT
      idPersonas,
      nombre,
      correo,
      sum(montoIE) AS SumaIE
    FROM
      entradasalidadinero
    INNER JOIN
      personas
    ON
      entradasalidadinero.idPersonas = personas.id
    GROUP BY 1, 2
    )

    SELECT
      aux.idPersonas AS ID,
      usuarios.id AS UserID,
      aux.nombre AS NombreCompleto,
      usuarios.nombre AS NombreDeUsuario,
      aux.correo AS CorreoElectronico,
      SumaIE
    FROM aux
    INNER JOIN usuarios
    ON aux.idPersonas = usuarios.idPersonas AND usuarios.idPersonas IN(1, 2, 3);
  ```

- Sentencia INSERT INTO

  Creando una copia de la tabla 'personas' en 'tmpPersonas' pero sin registros.

  _Nota:_ se crea la estructura de la tabla, más no las claves primarias o foráneas y demás.

  ```sql
    CREATE TABLE tmpPersonas AS
    SELECT * FROM personas
    WHERE id = -1;

    INSERT INTO tmppersonas VALUES(0, "Fernando Daniel", "fernando@gmail.com", "3221234567", 2);
    INSERT INTO tmppersonas VALUES(0, "Juan Perez", "perez@gmail.com", "3221234567", 2);

    INSERT INTO usuarios(nombre, password, idPersonas)
    VALUES("Fulano de tal", "equisde", 5);

    INSERT INTO tmppersonas VALUES
    (1, "Messi", "messi@messi.com", "3221234567", 1),
    (2, "Pique", "pique@pique.com", "9876543212", 2),
    (3, "Ronaldinho", "ronaldinho@ronaldinho.com", "3221377156", 3),
    (4, "Cristiano", "cristiano@cristiano.com", "1231231232", 4);
  ```

- Realizar inserciones a una tabla basadas en una consulta

  ```sql
    INSERT INTO tmppersonas
    SELECT * FROM personas WHERE correo REGEXP "@hotmail.com$";
  ```

- Sentencia UPDATE

  ```sql
    UPDATE entradasalidadinero
    SET montoIE = montoIE * 1.10
    WHERE fechaES = "2018-01-03";
  ```

- Sentencia UPDATE con una Subquery

  ```sql
    UPDATE usuarios
    SET password = "nuevopass"
    WHERE idPersonas IN(
      SELECT id FROM personas
      WHERE correo REGEXP "@hotmail.com$");
  ```

- Sentencia UPDATE con varias Subqueries

  ```sql
    UPDATE personas
    SET personas.nombre = (SELECT tmppersonas.nombre FROM tmppersonas WHERE tmppersonas.id = personas.id)
    WHERE personas.id = (SELECT tmppersonas.id FROM tmppersonas WHERE tmppersonas.id = personas.id);
  ```

- Sentencia DELETE

  ```sql
    DELETE FROM tmppersonas
    WHERE length(nombre) >= 15;

    DELETE FROM usuarios
    WHERE idPersonas IN(SELECT id FROM personas WHERE id <= 3);
  ```

- Borrar todo de una tabla (si la Integridad Referencial lo permite)

  ```sql
    DELETE FROM grupoingeg;
  ```

- Funciones de Agregación

  ```sql
    SELECT
        count(*) AS "Total de registros",
        round(avg(montoIE)) AS "Promedio IE",
        round(sum(montoIE)) AS "Suma IE",
        min(montoIE) AS "Monto mínimo",
        max(montoIE) AS "Monto máximo"
    FROM entradasalidadinero
    WHERE year(fechaES) > "2018";

    SELECT min(nombre), max(nombre) FROM personas;
  ```

- GROUP BY, ORDER BY & HAVING

  ```sql
    SELECT
        personas.nombre AS "NombreDeLaPersona",
        ingresosegresos.descripcion AS "DescripciónIE",
        sum(entradasalidadinero.montoIE) AS "Suma IE",
        count(*) AS "NúmeroDeRegistros"
    FROM entradasalidadinero
    INNER JOIN personas ON personas.id = entradasalidadinero.idPersonas
    INNER JOIN ingresosegresos ON ingresosegresos.id = entradasalidadinero.idIngresosEgresos
    WHERE personas.nombre IN("ANDRES PEREZ LOPEZ", "BERNARDO FUENTES CORREA")
    GROUP BY NombreDeLaPersona, DescripciónIE
    HAVING sum(montoIE) >= 1000 AND NúmeroDeRegistros = 2
    ORDER BY NombreDeLaPersona;
  ```

- WITH ROLLUP & GROUPING con Condicional IF

  ```sql
    SELECT
        personas.nombre AS "NombreDeLaPersona",
        ingresosegresos.descripcion AS "DescripciónIE",
        IF(GROUPING(personas.nombre) = 1, "TotalGeneral", " ") AS "TGen",
        IF(GROUPING(ingresosegresos.descripcion) = 1, "TotalPorGrupo", " ") AS "TGrupo",
        sum(entradasalidadinero.montoIE) AS "Suma IE",
        count(*) AS "NúmeroDeRegistros"
    FROM entradasalidadinero
    INNER JOIN personas ON personas.id = entradasalidadinero.idPersonas
    INNER JOIN ingresosegresos ON ingresosegresos.id = entradasalidadinero.idIngresosEgresos
    WHERE personas.nombre IN("ANDRES PEREZ LOPEZ", "BERNARDO FUENTES CORREA")
    GROUP BY NombreDeLaPersona, DescripciónIE WITH ROLLUP;
  ```

- Clausula OVER (Window Functions)

  ```sql
    SELECT
        idPersonas, fechaES, montoIE,
        sum(montoIE) OVER() AS TotalMontoIE,
        count(idPersonas) OVER() AS TotalDePersonas,
        sum(montoIE) OVER(PARTITION BY idPersonas ORDER BY montoIE ASC) AS TotalPorPersona,
        count(idPersonas) OVER(PARTITION BY idPersonas) AS ConteoDePersonas
    FROM entradasalidadinero
    WHERE year(fechaES) = 2018 and month(fechaES) = 1;
  ```

- Uso de FRAMES - ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

  ```sql
    SELECT
        personas.nombre,
        ingresosegresos.descripcion,
        entradasalidadinero.montoIE,
        sum(entradasalidadinero.montoIE) OVER() AS TotalMontoIE,
        count(*) OVER() AS TotalDeRegistros,
        count(*) OVER(PARTITION BY personas.nombre) AS TotalDeRegistrosPorPersona,
        sum(entradasalidadinero.montoIE) OVER(PARTITION BY personas.nombre) AS MontoIEPorPersona,
        sum(entradasalidadinero.montoIE)
          OVER (PARTITION BY personas.nombre
                ORDER BY entradasalidadinero.montoIE ASC
                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) -- Suma el actual registro con el anterior
          AS Acumulados
    FROM entradasalidadinero
    INNER JOIN personas ON personas.id = entradasalidadinero.idPersonas
    INNER JOIN ingresosegresos ON ingresosegresos.id = entradasalidadinero.idIngresosEgresos
    WHERE YEAR(fechaES) = 2018 AND MONTH(fechaES) = 1;
  ```

- Uso de FRAMES - RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW

  ```sql
    SELECT
        personas.nombre,
        ingresosegresos.descripcion,
        entradasalidadinero.montoIE,
        sum(entradasalidadinero.montoIE) OVER() AS TotalMontoIE,
        count(*) OVER() AS TotalDeRegistros,
        count(*) OVER(PARTITION BY personas.nombre) AS TotalDeRegistrosPorPersona,
        sum(entradasalidadinero.montoIE) OVER(PARTITION BY personas.nombre) AS MontoIEPorPersona,
        sum(entradasalidadinero.montoIE)
            OVER  (PARTITION BY personas.nombre
                  ORDER BY entradasalidadinero.fechaES DESC
                  RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) -- Suma el actual registro con el anterior
                  -- pero condicionado hasta encontrar dos fechas iguales (por el ordenamiento por fechaES).
                  -- Si encuentra dos o más fechas iguales, las toma como un frame.
            AS Acumulados
    FROM entradasalidadinero
    INNER JOIN personas ON personas.id = entradasalidadinero.idPersonas
    INNER JOIN ingresosegresos ON ingresosegresos.id = entradasalidadinero.idIngresosEgresos
    WHERE YEAR(fechaES) = 2018 AND MONTH(fechaES) = 1;
  ```

## DCL (Data Control Language)

- Muestra los privilegios del actual usuario

  ```sql
    SHOW privileges;
  ```

- Crea un usuario y se le asigna un password

  ```sql
    CREATE USER fulanodetal@localhost identified by "12345";
  ```

- Selecciona usuario y host de la tabla user de la db mysql

  ```sql
    USE mysql;
    SELECT user, host
    FROM user;
  ```

- Asignar permisos a un usuario sobe una base de datos específica

  ```sql
    GRANT ALL
    ON bdpendientes.*
    TO fulanodetal@localhost;
  ```

- Ver permisos de un usuario en específico

  ```sql
    SHOW GRANTS FOR fulanodetal@localhost;
  ```

- Crear un usuario y asignarle permisos específicos

  ```sql
    CREATE USER sofia@localhost identified by "12345678";
    GRANT SELECT, INSERT
    ON bdpendientes.*
    TO sofia@localhost WITH GRANT OPTION;
  ```

- Revocar permisos a un usuario

  ```sql
    REVOKE ALL, GRANT OPTION FROM fulanodetal@localhost;
  ```

- Renombrar un nombre de usuario

  ```sql
    RENAME USER sofia@localhost TO laura@localhost;
  ```

- Eliminar un usuario

  ```sql
    DROP USER laura@localhost;
  ```

## Vistas (Views)

Además de la instrucción SELECT, en las Vistas también se pueden utilizar instrucciones como UPDATE, DELETE O INSERT, sin embargo el uso más frecuente es con SELECT, para mostrar información sin tener que reconstruir la Query completa.

- Crear una Vista con una sola Tabla implicada

  ```sql
    CREATE VIEW personas_view AS
    SELECT nombre, correo, telefono
    FROM personas;
  ```

- Ejecutar una Vista

  ```sql
    SELECT * FROM personas_view;
  ```

- Crea una Vista nueva o la reemplaza si la Vista la existe

  ```sql
    CREATE OR REPLACE VIEW personas_view AS
    SELECT nombre, correo
    FROM personas;
  ```

- Eliminar una Vista

  ```sql
    DROP VIEW personas_view;
  ```

## Procedimientos Almacenados (Stored Procedures)

- Crear un Procedimiento

  ```sql
    DELIMITER //
    CREATE PROCEDURE test()
    BEGIN
      -- SELECT en este sentido funciona como un Return en Programación
      SELECT
        'Esta es una prueba' AS mensajeTest,
        'Hola Mundo' AS mensajeTest2,
        'Adios' AS mensajeTest3;
    END //
    DELIMITER ;
  ```

- Llamar un Procedimiento

  ```sql
    CALL test();
  ```

- Eliminar un Procedimiento

  ```sql
    DROP PROCEDURE test;
  ```

- Procedimiento Almacenado - Ejemplo 1

  ```sql
    DELIMITER //
    CREATE PROCEDURE getTotalPendientesNuevos()
    BEGIN
        SELECT count(*) AS TotalPendientesNuevos
        FROM estatuspendiente
        INNER JOIN pendientes ON estatuspendiente.id = pendientes.idEstatusPendiente
        WHERE estatuspendiente.estatus = "Nuevo";
    END //
    DELIMITER ;

    CALL getTotalPendientesNuevos();
  ```

- Procedimiento Almacenado - Ejemplo 2

  ```sql
    DELIMITER //
    CREATE PROCEDURE getTotalPendientesNuevos()
    BEGIN
        DECLARE TotalPendientesNuevos INT;
        SELECT count(*) INTO TotalPendientesNuevos
        FROM estatuspendiente
        INNER JOIN pendientes ON estatuspendiente.id = pendientes.idEstatusPendiente
        WHERE estatuspendiente.estatus = "Nuevo";

        SELECT TotalPendientesNuevos;
    END //
    DELIMITER ;

    CALL getTotalPendientesNuevos();
  ```

- Procedimiento Almacenado - Con parámetros IN - OUT & instrucción IF - ELSEIF

  ```sql
    DELIMITER //
    CREATE PROCEDURE totalPendientes(IN IDPersonaAsignado INT, OUT Frase VARCHAR(100), OUT Persona VARCHAR(50))
    BEGIN
        DECLARE totalPendientes INT;

        SELECT count(*)
        INTO totalPendientes
        FROM pendientes
        WHERE pendientes.idPersonaAsignado = IDPersonaAsignado;

        SELECT nombre
        INTO Persona
        FROM personas
        WHERE personas.id = IDPersonaAsignado;

        IF totalPendientes = 0 OR totalPendientes = null THEN
          SET Frase = 'Sin pendientes!';
        ELSEIF totalPendientes > 1 AND totalPendientes < 10 THEN
          SET Frase = concat('Hay ', totalPendientes, ' pendientes. ', 'La persona no puede atenderte.');
        ELSEIF totalPendientes >= 11 THEN
          SET Frase = concat('# de Pendientes: ', totalPendientes, '. Imposible de atender. Persona muy ocupada.');
        END IF;
    END //
    DELIMITER ;

    CALL totalPendientes(3, @Frase, @Persona);
    SELECT @Frase, @Persona;
  ```

- Procedimiento Almacenado - Con parámetros IN - OUT & instrucción CASE - Forma 1

  ```sql
    DELIMITER //
    CREATE PROCEDURE totalPendientesCase(IN IDPersonaAsignado INT, OUT Frase VARCHAR(100), OUT Persona VARCHAR(50))
    BEGIN
        DECLARE totalPendientes INT;

        SELECT count(*)
        INTO totalPendientes
        FROM pendientes
        WHERE pendientes.idPersonaAsignado = IDPersonaAsignado;

        SELECT nombre
        INTO Persona
        FROM personas
        WHERE personas.id = IDPersonaAsignado;

      CASE
        WHEN totalPendientes > 1 AND totalPendientes < 10 THEN
          SET Frase = concat('Hay ', totalPendientes, ' pendientes.', 'La persona no puede atenderte.');
        WHEN totalPendientes >= 11 THEN
          SET Frase = concat('# de Pendientes: ', totalPendientes, '. Imposible de atender. Persona muy ocupada.');
        ELSE
          SET Frase = 'Sin pendientes!';
      END CASE;
    END //
    DELIMITER ;

    CALL totalPendientesCase(4, @Frase, @Persona);
    SELECT @Frase, @Persona;
  ```

- Procedimiento Almacenado - Con parámetros IN - OUT & instrucción CASE - Forma 2

  ```sql
    DELIMITER //
    CREATE PROCEDURE totalPendientesCase2(IN IDPersonaAsignado INT, OUT Frase VARCHAR(100), OUT Persona VARCHAR(50))
    BEGIN
        DECLARE totalPendientes INT;

        SELECT count(*)
        INTO totalPendientes
        FROM pendientes
        WHERE pendientes.idPersonaAsignado = IDPersonaAsignado;

        SELECT nombre
        INTO Persona
        FROM personas
        WHERE personas.id = IDPersonaAsignado;

      CASE totalPendientes
        WHEN 7 THEN
          SET Frase = '7 pendientes';
        WHEN 70 THEN
          SET Frase = '70 pendientes';
        ELSE
          SET Frase = 'Diferente de 7 y 70';
      END CASE;
    END //
    DELIMITER ;

    CALL totalPendientesCase2(2, @Frase, @Persona);
    SELECT @Frase, @Persona;
  ```

- Procedimiento Almacenado - Ciclo WHILE

  ```sql
    DELIMITER //
    CREATE PROCEDURE ejemploConWhile()
    BEGIN
        DECLARE Counter INT DEFAULT 1;
        DECLARE Str VARCHAR(100) DEFAULT '';

        counterLoop: WHILE Counter <= 5 DO
            SET Str = concat(Str, 'Counter in ', Counter, ' | ');
            IF Counter = 5 THEN
              LEAVE counterLoop;
            END IF;
            SET Counter = Counter + 1;
        END WHILE counterLoop;

        SELECT Str, Counter;
    END //
    DELIMITER ;

    CALL ejemploConWhile();
  ```

- Procedimiento Almacenado - Ciclo REPEAT

  ```sql
    DELIMITER //
    CREATE PROCEDURE ejemploConRepeat()
    BEGIN
        DECLARE Counter INT DEFAULT 1;
        DECLARE Str VARCHAR(100) DEFAULT '';

        REPEAT
            SET Str = concat(Str, 'Counter in ', Counter, ' | ');
            SET Counter = Counter + 1;
        UNTIL Counter > 5
        END REPEAT;

      SELECT Str, Counter;
    END //
    DELIMITER ;

    CALL ejemploConRepeat();
  ```

- Procedimiento Almacenado - Ciclo LOOP

  ```sql
    DELIMITER //
    CREATE PROCEDURE ejemploConLoop()
    BEGIN
        DECLARE Counter INT DEFAULT 1;
        DECLARE Str VARCHAR(100) DEFAULT '';

        counterLoop: LOOP
            IF Counter = 5 THEN
              LEAVE counterLoop;
            END IF;
            SET Str = concat(Str, 'Counter in ', Counter, ' | ');
            SET Counter = Counter + 1;
        END LOOP counterLoop;

      SELECT Str, Counter;
    END //
    DELIMITER ;

    CALL ejemploConLoop();
  ```

- Procedimiento Almacenado - Cursores

  ```sql
    DELIMITER //
    CREATE PROCEDURE cursores()
    BEGIN
        DECLARE id_persona INT;
        DECLARE telefono_persona VARCHAR(30);
        DECLARE row_not_found TINYINT DEFAULT FALSE;
        DECLARE update_count INT DEFAULT 0;

        -- Declarando el cursor
        DECLARE cursor_persona CURSOR FOR
        SELECT id, telefono
        FROM personas
        WHERE correo LIKE '%hotmail%';

        -- Declarando un manejador de errores para cuando el cursor no encuentre más renglones
        DECLARE CONTINUE HANDLER FOR NOT FOUND
        SET row_not_found = TRUE;

        -- Operando en el cursor
        OPEN cursor_persona;
        WHILE row_not_found = FALSE DO

            -- Obteniendo los valores del renglón y almacenándolos en variables
            FETCH cursor_persona INTO id_persona, telefono_persona;

            IF length(telefono_persona) > 0 THEN
              UPDATE pendientes
              SET observaciones = concat('Actualmente la persona tiene el número ', telefono_persona)
              WHERE idPersonaAsignado = id_persona;
            ELSE
              UPDATE pendientes
              SET observaciones = 'Favor de solicitar el número de teléfono'
              WHERE idPersonaAsignado = id_persona;

              SET update_count = update_count + 1;
            END IF;
        END WHILE;
        CLOSE cursor_persona;

        SELECT concat(update_count, ' personas no tienen teléfono.') AS NúmeroDePersonasSinTeléfono;
    END //
    DELIMITER ;

    CALL cursores();
  ```

## Funciones Almacenadas (Stored Functions)

```sql
  DELIMITER //
  CREATE FUNCTION montosIngresosEgresos(idIE INT)
  RETURNS DECIMAL(8, 2)
  READS SQL DATA
  BEGIN
      DECLARE Sumatoria DECIMAL(8, 2);
      SELECT sum(montoIE) INTO Sumatoria
      FROM entradasalidadinero
      WHERE idIngresosEgresos = idIE;

      IF Sumatoria > 0 THEN
        RETURN Sumatoria;
      ELSE
        RETURN 0;
      END IF;
  END //
  DELIMITER ;

  SELECT id, descripcion, montosIngresosEgresos(id) FROM ingresosegresos;
```

## Triggers

- Ejemplo 1 - BEFORE INSERT

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_before_insert
    BEFORE INSERT ON personas
    FOR EACH ROW
    BEGIN
      IF new.correo LIKE '%gmail%' THEN
        SIGNAL SQLSTATE "HY000"
        SET MESSAGE_TEXT = 'Se ingresó un correo con dominio "gmail". Correo inválido.';
      ELSE
        SET new.correo = lower(new.correo);
      END IF;
    END //
    DELIMITER ;
  ```

- Ejemplo 2 - AFTER INSERT

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_after_insert
    AFTER INSERT ON personas
    FOR EACH ROW
    BEGIN
      INSERT INTO log_eventos
      VALUES ('personas', 'after insert', new.correo, 'n/a', 'completado', now());
    END //
    DELIMITER ;
  ```

- Ejemplo 3 - BEFORE UPDATE

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_before_update
    BEFORE UPDATE ON personas
    FOR EACH ROW
    BEGIN
      IF new.correo LIKE '%gmail%' THEN
        SIGNAL SQLSTATE "HY000"
        SET MESSAGE_TEXT = 'Se ingresó un correo con dominio "gmail". Correo inválido.';
      ELSE
        SET new.correo = lower(new.correo);
        INSERT INTO log_eventos
        VALUES ('personas', 'before update', new.correo, old.correo, 'completado', now());
      END IF;
    END //
    DELIMITER ;
  ```

- Ejemplo 4 - AFTER UPDATE

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_after_update
    AFTER UPDATE ON personas
    FOR EACH ROW
    BEGIN
      INSERT INTO log_eventos
      VALUES ('personas', 'after update', new.correo, old.correo, 'completado', now());
    END //
    DELIMITER ;
  ```

- Ejemplo 5 - BEFORE DELETE

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_before_delete
    BEFORE DELETE ON personas
    FOR EACH ROW
    BEGIN
      INSERT INTO log_eventos
      VALUES ('personas', 'before delete', 'n/a', old.correo, 'completado', now());
    END //
    DELIMITER ;
  ```

- Ejemplo 6 - AFTER DELETE

  ```sql
    DELIMITER //
    CREATE TRIGGER correo_lowercase_after_delete
    AFTER DELETE ON personas
    FOR EACH ROW
    BEGIN
      INSERT INTO log_eventos
      VALUES ('personas', 'after delete', 'n/a', old.correo, 'completado', now());
    END //
    DELIMITER ;
  ```

## SQL Dinámico

```sql
  DELIMITER //
  CREATE PROCEDURE dynamicSelect(IN args VARCHAR(100), IN tblName VARCHAR(50))
  BEGIN
      SET @qry = concat('SELECT ', args, ' FROM ', tblName);
      PREPARE select_statement FROM @qry;
      EXECUTE select_statement;
      DEALLOCATE PREPARE select_statement;
  END //
  DELIMITER ;

  CALL dynamicSelect('*', 'personas');
```

## Transactions

```sql
  DELIMITER //
  CREATE PROCEDURE transactionExample()
  BEGIN
      DECLARE sql_error TINYINT DEFAULT FALSE;
      DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
      SET sql_error = TRUE;

      START TRANSACTION;
      -- Aquí se ejecutan las instrucciones INSERT, UPDATE o DELETE, según sea necesario
      -- Si una sola instrucción falla, se ejecuta el Rollback, y por lo tanto, ninguna instrucción se completa
      -- Si ninguna falla, se ejecuta el Commit
      DELETE FROM tmppersonas;
      INSERT INTO tmppersonas SELECT * FROM personas;
      UPDATE tmppersonas SET telefono = '450';
      INSERT INTO tmppersonas VALUES (0, "Maria", "correo@correo.com", '123456789', 3);

      IF sql_error THEN
        ROLLBACK;
        SELECT 'La transacción no fue ejecutada' AS "Status";
      ELSE
        COMMIT;
        SELECT 'La transacción se ejecutó con éxito' AS "Status";
      END IF;
  END //
  DELIMITER ;

  CALL transactionExample();
```

## Character Sets & Collations

```sql
  SHOW CHAR SET LIKE "%utf8mb4%";
  SHOW VARIABLES LIKE "character_set_server";
  SHOW VARIABLES LIKE "character_set_database";
  SHOW COLLATION;
  SHOW VARIABLES LIKE "collation_server";

  SELECT TABLE_NAME, TABLE_COLLATION
  FROM INFORMATION_SCHEMA.TABLES
  WHERE TABLE_SCHEMA = "bdpendientes";
```
