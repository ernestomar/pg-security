# Autenticación y Autorización en Base de datos

## Correr el contendor Docker para PostgreSql.

``` bash
docker run --name pg-internship -e POSTGRES_PASSWORD=123456 -p 5432:5432 -d postgres:15.2
```

## Inicializar la Base de Datos.
Para crear la base de datos y las tablas que mencionas en PostgreSQL, puedes seguir los siguientes pasos. Por favor, ten en cuenta que debes adaptar este código a tus necesidades específicas y asegurarte de que tienes los permisos necesarios para crear y modificar tablas en la base de datos.

Nos conetamos al motor de Postgres

```bash
$ docker exec -it pg-internship bash
```

Luego dentro del contenedor cambiamos de usuario

```bash
root@ba597b82a000:/# su - postgres
postgres@ba597b82a000:~$ psql template1
```

Primero, crea la base de datos:

```sql
CREATE DATABASE internship_db;

\c internship_db

```

A continuación, crea las tablas SE\_USER, SE\_GROUP y SE\_ROLE, incluyendo las llaves surrogadas y los campos de auditoría:

```sql
CREATE TABLE SE_USER (
  SE_USER_ID SERIAL PRIMARY KEY,
  USERNAME VARCHAR(255) UNIQUE NOT NULL,
  PASSWORD VARCHAR(255) NOT NULL,
  SE_GROUP_ID INTEGER,
  SE_ROLE_ID INTEGER,
  TX_HOST VARCHAR(255),
  TX_USER VARCHAR(255),
  TX_DATE TIMESTAMP
);

CREATE TABLE SE_GROUP (
  SE_GROUP_ID SERIAL PRIMARY KEY,
  GROUP_NAME VARCHAR(255) UNIQUE NOT NULL,
  TX_HOST VARCHAR(255),
  TX_USER VARCHAR(255),
  TX_DATE TIMESTAMP
);

CREATE TABLE SE_ROLE (
  SE_ROLE_ID SERIAL PRIMARY KEY,
  ROLE_NAME VARCHAR(255) UNIQUE NOT NULL,
  TX_HOST VARCHAR(255),
  TX_USER VARCHAR(255),
  TX_DATE TIMESTAMP
);

```

Luego, crea la tabla histórica H\_SE\_USER:

```
CREATE TABLE H_SE_USER (
  H_SE_USER_ID SERIAL PRIMARY KEY,
  SE_USER_ID INTEGER NOT NULL,
  USERNAME VARCHAR(255),
  PASSWORD VARCHAR(255),
  SE_GROUP_ID INTEGER,
  SE_ROLE_ID INTEGER,
  TX_HOST VARCHAR(255),
  TX_USER VARCHAR(255),
  TX_DATE TIMESTAMP
);

```

A continuación, crea un trigger para guardar el historial de cambios en la tabla H\_SE\_USER cada vez que se actualiza o inserta un registro en la tabla SE\_USER:

```
CREATE OR REPLACE FUNCTION insert_update_history() RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO H_SE_USER (
    SE_USER_ID,
    USERNAME,
    PASSWORD,
    SE_GROUP_ID,
    SE_ROLE_ID,
    TX_HOST,
    TX_USER,
    TX_DATE
  ) VALUES (
    NEW.SE_USER_ID,
    NEW.USERNAME,
    NEW.PASSWORD,
    NEW.SE_GROUP_ID,
    NEW.SE_ROLE_ID,
    NEW.TX_HOST,
    NEW.TX_USER,
    NEW.TX_DATE
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER save_history
AFTER INSERT OR UPDATE ON SE_USER
FOR EACH ROW EXECUTE PROCEDURE insert_update_history();

```

Con esto, se creará un registro en la tabla H\_SE\_USER cada vez que se realice una operación de inserción o actualización en la tabla SE\_USER, lo cual te permitirá llevar un registro histórico de los cambios en los usuarios.