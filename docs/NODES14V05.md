# VIDEO 05 - OneToMany en TypeORM

En este vídeo vamos a ver cómo podemos crear relaciones en SQL haciendo uso de TypeORM.

<https://typeorm.io/#creating-a-many-to-one--one-to-many-relation>

Hemos creado una entidad Team para almacenar los datos de un equipo y sus jugadores:

```tsx
import { Entity, PrimaryGeneratedColumn, Column, OneToMany } from "typeorm";
import { Player } from "./Player";

@Entity()
export class Team {
  @PrimaryGeneratedColumn()
    id: number;

  @Column()
    name: string;

  @Column()
    city: string;

  // JUGADORES
  @OneToMany(type => Player, player => player.team, { cascade: true })
    players: Player[];
}
```

También hemos modificado los jugadores para reflejar esa relación:

```tsx
import { Entity, PrimaryGeneratedColumn, Column, ManyToOne } from "typeorm";
import { Team } from "./Team";

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

  @ManyToOne(type => Team, team => team.players)
    team: Team;
}
```

Tras esto hemos creado un router para equipos:

```tsx
import { Router, type NextFunction, type Request, type Response } from "express";

// Typeorm
import { Team } from "../models/typeorm/Team";
import { AppDataSource } from "../databases/typeorm-datasource";
import { type Repository } from "typeorm";

const teamRepository: Repository<Team> = AppDataSource.getRepository(Team);

// Router
export const teamRouter = Router();

// CRUD: READ
teamRouter.get("/", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const teams: Team[] = await teamRepository.find({ relations: ["players"] });
    res.json(teams);
  } catch (error) {
    next(error);
  }
});

teamRouter.get("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const team = await teamRepository.findOne({
      where: {
        id: idReceivedInParams,
      },
      relations: ["players"],
    });

    if (!team) {
      res.status(404).json({ error: "Team not found" });
    }

    res.json(team);
  } catch (error) {
    next(error);
  }
});

// CRUD: CREATE
teamRouter.post("/", async (req: Request, res: Response, next: NextFunction) => {
  try {
    // Construimos team
    const newTeam = new Team();

    // Asignamos valores
    Object.assign(newTeam, req.body);

    const teamSaved = await teamRepository.save(newTeam);

    res.status(201).json(teamSaved);
  } catch (error) {
    next(error);
  }
});

// CRUD: DELETE
teamRouter.delete("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const teamToRemove = await teamRepository.findOne({
      where: {
        id: idReceivedInParams,
      },
      relations: ["players"]
    });

    if (!teamToRemove) {
      res.status(404).json({ error: "Team not found" });
    } else {
      // Quitar a los jugadores este equipo
      for (const player of teamToRemove.players) {
        player.team = null as any;
        await AppDataSource.manager.save(player);
      }

      await teamRepository.remove(teamToRemove);
      res.json(teamToRemove);
    }
  } catch (error) {
    next(error);
  }
});

teamRouter.put("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const teamToUpdate = await teamRepository.findOneBy({
      id: idReceivedInParams,
    });

    if (!teamToUpdate) {
      res.status(404).json({ error: "Team not found" });
    } else {
      // Asignamos valores
      Object.assign(teamToUpdate, req.body);

      const updatedTeam = await teamRepository.save(teamToUpdate);
      res.json(updatedTeam);
    }
  } catch (error) {
    next(error);
  }
});
```

Y hemos modificado las queries de jugadores para reflejar los datos de su equipo:

```tsx
// CRUD: READ
playerRouter.get("/", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const players: Player[] = await playerRepository.find({ relations: ["team"] });
    res.json(players);
  } catch (error) {
    next(error);
  }
});

playerRouter.get("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const player = await playerRepository.findOne({
      where: {
        id: idReceivedInParams,
      },
      relations: ["team"],
    });

    if (!player) {
      res.status(404).json({ error: "Player not found" });
    }

    res.json(player);
  } catch (error) {
    next(error);
  }
});
```

También hemos modificado nuestro seed para que cree jugadores y equipos:

```tsx
import { AppDataSource } from "../databases/typeorm-datasource";
import { Player } from "../models/typeorm/Player";
import { Team } from "../models/typeorm/Team";

export const teamAndPlayerSeed = async (): Promise<void> => {
  // Nos conectamos a la BBDD
  const dataSource = await AppDataSource.initialize();
  console.log(`Tenemos conexión!! Conectados a ${dataSource?.options?.database as string}`);

  // Eliminamos los datos existentes
  await AppDataSource.manager.delete(Player, {});
  await AppDataSource.manager.delete(Team, {});
  console.log("Eliminados jugadores y equipos");

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
  // await AppDataSource.manager.save(player1Entity);
  // await AppDataSource.manager.save(player2Entity);

  // Creamos equipo
  const team = {
    name: "Dream Team",
    city: "Madrid",
    players: [player1Entity, player2Entity]
  };

  // Creamos entidad equipo
  const teamEntity = AppDataSource.manager.create(Team, team);

  // Guardamos el equipo en BBDD
  await AppDataSource.manager.save(teamEntity);

  console.log("Creados los dos jugadores y equipo");

  // Cerramos la conexión
  await AppDataSource.destroy();
  console.log("Cerrada conexión SQL");
}

void teamAndPlayerSeed();
```

Recuerda que puedes encontrar en este repositorio todo el código que hemos visto durante la sesión:

<https://github.com/The-Valley-School/node-s14-typeorm-for-sql>

