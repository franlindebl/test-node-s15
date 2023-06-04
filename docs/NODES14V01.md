# VIDEO 01 - Intro a TypeORM

ORM significa Object-Relational Mapping (Mapeo Objeto-Relacional en español). Es una técnica de programación que convierte datos entre el sistema de tipos utilizado en un lenguaje de programación orientado a objetos y el sistema de tipos utilizado en una base de datos relacional.

En términos más sencillos, un ORM es como un puente entre tu código y la base de datos. Te permite interactuar con los datos de la base de datos como si fueran objetos en tu lenguaje de programación. Esto significa que no tienes que escribir código SQL crudo en la mayoría de los casos, lo que puede hacer que tu código sea más fácil de leer, escribir y mantener.

Con un ORM, puedes hacer cosas como las siguientes:

- Definir modelos que representan las tablas de tu base de datos.
- Crear, leer, actualizar y borrar registros en tu base de datos simplemente trabajando con los objetos de tus modelos.
- Realizar consultas complejas utilizando un API más amigable para el desarrollador que el SQL crudo.

Es importante mencionar que los ORMs pueden variar en términos de características y capacidades. Algunos pueden ser bastante simples y solo proporcionar funcionalidades básicas de CRUD (Create, Read, Update, Delete), mientras que otros pueden tener características más avanzadas como el mapeo automático de relaciones entre tablas, migraciones de base de datos y mucho más.

TypeORM, es un ORM para Node.js que admite tanto JavaScript como TypeScript y tiene muchas características avanzadas, está preparado para trabajar con múltilples fuentes de bases de datos, como por ejemplo SQL o Mongo. Nosotros vamos a trabajar con SQL en esta ocasión.

<https://typeorm.io/>

En este vídeo hemos instalado la librería:

```tsx
npm i reflect-metadata 
npm i typeorm
```

Y hemos creado nuestro archivo de conexión con la base de datos:

```tsx
import "reflect-metadata";
import { DataSource } from "typeorm";
import dotenv from "dotenv";
dotenv.config();

const SQL_HOST: string = process.env.SQL_HOST as string;
const SQL_USER: string = process.env.SQL_USER as string;
const SQL_PASSWORD: string = process.env.SQL_PASSWORD as string;
const SQL_DATABASE: string = process.env.SQL_DATABASE as string;

export const AppDataSource = new DataSource({
  host: SQL_HOST,
  username: SQL_USER,
  password: SQL_PASSWORD,
  database: SQL_DATABASE,
  type: "mysql",
  port: 3306,
  synchronize: true,
  logging: false,
  entities: [], // TODO
  migrations: [], // TODO
  subscribers: [], // TODO
});
```

Y lo hemos importado en nuestro index.ts

```tsx
import { AppDataSource } from "./databases/typeorm-datasource";

// ... 
// ...

const datasource = await AppDataSource.initialize();

// ...
// ...

<p>Estamos usando TypeORM con la BBDD: ${datasource.options.database as string}</p>
```

