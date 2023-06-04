# VIDEO 04 - Debug en Typescript

En este vídeo hemos visto cómo podemos hacer debug en Typescript haciendo uso de Visual Studio Code. Para ello hemos creado un fichero launch.json que indicará a Visual Studio que se conecte al proceso node:

```tsx
{
  // Use IntelliSense para saber los atributos posibles.
  // Mantenga el puntero para ver las descripciones de los existentes atributos.
  // Para más información, visite: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "attach",
      "name": "Enganchar a un proceso",
      "skipFiles": [
        "<node_internals>/**"
      ],
    }
  ]
}
```

Por otro lado le hemos dicho a ts-node-dev que se ponga en modo inspección en nuestro package.json:

```tsx
"start": "ts-node-dev --inspect -- ./src/index.ts",
```

Gracias a esto hemos podido depurar bien el error que nos daba y hemos arreglado nuestro middleware de errores para que devuelva un mensaje personalizado cuando nos falte por rellenar un campo:

```tsx
// Middleware de gestión de errores
  app.use((err: ErrorRequestHandler, req: Request, res: Response, next: NextFunction) => {
    console.log("*** INICIO DE ERROR ***");
    console.log(`PETICIÓN FALLIDA: ${req.method} a la url ${req.originalUrl}`);
    console.log(err);
    console.log("*** FIN DE ERROR ***");

    // Truco para quitar el tipo a una variable
    const errorAsAny: any = err as unknown as any;

    if (err?.name === "ValidationError") {
      res.status(400).json(err);
    } else if (errorAsAny.errmsg && errorAsAny.errmsg?.indexOf("duplicate key") !== -1) {
      res.status(400).json({ error: errorAsAny.errmsg });
    } else if (errorAsAny?.code === "ER_NO_DEFAULT_FOR_FIELD") {
      res.status(400).json({ error: errorAsAny?.sqlMessage });
    } else {
      res.status(500).json(err);
    }
  });
```

