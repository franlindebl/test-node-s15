# VIDEO 02 - Primera entidad y seed

En esta sesión hemos creado nuestra primera entidad de TypeORM para almacenar jugadores:

```tsx
import { Entity, PrimaryGeneratedColumn, Column } from "typeorm";

@Entity()
export class Player {
  @PrimaryGeneratedColumn()
    id: number;

  @Column()
    firstName: string;

  @Column()
    lastName: string;

  @Column()
    shirtNumber: number;
}
```

Además de eso hemos creado nuestro primer seed:

```tsx
import { AppDataSource } from "../databases/typeorm-datasource";
import { Player } from "../models/typeorm/Player";

export const playerSeed = async (): Promise<void> => {
  // Nos conectamos a la BBDD
  const dataSource = await AppDataSource.initialize();
  console.log(`Tenemos conexión!! Conectados a ${dataSource?.options?.database as string}`);

  // Eliminamos los datos existentes
  await AppDataSource.manager.delete(Player, {});
  console.log("Eliminados jugadores existentes");

  // Creamos dos players
  const player1 = {
    firstName: "Cristiano",
    lastName: "Ronaldo",
    shirtNumber: 7,
  };

  const player2 = {
    firstName: "Lionel",
    lastName: "Messi",
    shirtNumber: 10,
  };

  // Creamos las entidades
  const player1Entity = AppDataSource.manager.create(Player, player1);
  const player2Entity = AppDataSource.manager.create(Player, player2);

  // Las guardamos en base de datos
  await AppDataSource.manager.save(player1Entity);
  await AppDataSource.manager.save(player2Entity);

  console.log("Creados los dos jugadores");

  // Cerramos la conexión
  await AppDataSource.destroy();
  console.log("Cerrada conexión SQL");
}

void playerSeed();
```

Es importante que referenciemos el nuevo modelo e el AppDataSource o fallará:

```tsx
import "reflect-metadata";
import { DataSource } from "typeorm";
import { Player } from "../models/typeorm/Player";
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
  entities: [Player], // TODO
  migrations: [], // TODO
  subscribers: [], // TODO
});
```

Para ejecutar el seed hemos creado un comando en el package.json que hace uso de ts-node-dev pero sin el watch:

```tsx
"seed:player": "ts-node-dev ./src/seeds/Player.seed.ts --ignore-watch",
```

