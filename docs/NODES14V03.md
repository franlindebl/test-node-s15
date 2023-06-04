# VIDEO 03 - CRUD completo de Player

En este vídeo hemos creado el CRUD para nuestra nueva entidad Player.

Nuestro fichero player.routes.ts ha quedado así:

```tsx
import { Router, type NextFunction, type Request, type Response } from "express";

// Typeorm
import { Player } from "../models/typeorm/Player";
import { AppDataSource } from "../databases/typeorm-datasource";
import { type Repository } from "typeorm";

const playerRepository: Repository<Player> = AppDataSource.getRepository(Player);

// Router
export const playerRouter = Router();

// CRUD: READ
playerRouter.get("/", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const players: Player[] = await playerRepository.find();
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
    });

    if (!player) {
      res.status(404).json({ error: "Player not found" });
    }

    res.json(player);
  } catch (error) {
    next(error);
  }
});

// CRUD: CREATE
playerRouter.post("/", async (req: Request, res: Response, next: NextFunction) => {
  try {
    // Construimos player
    const newPlayer = new Player();

    // Asignamos valores
    Object.assign(newPlayer, {
      firstName: req.body.firstName,
      lastName: req.body.lastName,
      shirtNumber: req.body.shirtNumber,
    });

    const playerSaved = await playerRepository.save(newPlayer);

    res.status(201).json(playerSaved);
  } catch (error) {
    next(error);
  }
});

// CRUD: DELETE
playerRouter.delete("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const playerToRemove = await playerRepository.findOneBy({
      id: idReceivedInParams,
    });

    if (!playerToRemove) {
      res.status(404).json({ error: "Player not found" });
    } else {
      await playerRepository.remove(playerToRemove);
      res.json(playerToRemove);
    }
  } catch (error) {
    next(error);
  }
});

playerRouter.put("/:id", async (req: Request, res: Response, next: NextFunction) => {
  try {
    const idReceivedInParams = parseInt(req.params.id);

    const playerToUpdate = await playerRepository.findOneBy({
      id: idReceivedInParams,
    });

    if (!playerToUpdate) {
      res.status(404).json({ error: "Player not found" });
    } else {
      // Asignamos valores
      Object.assign(playerToUpdate, req.body);

      const updatedPlayer = await playerRepository.save(playerToUpdate);
      res.json(updatedPlayer);
    }
  } catch (error) {
    next(error);
  }
});
```

