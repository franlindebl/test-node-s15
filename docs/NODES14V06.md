# VIDEO 06 - Ejercicio: Creación Alumno y Curso con TypeORM

En este ejercicio debes utilizar TypeORM para crear entidades Alumno y Curso. Estas entidades estarán relacionadas: un curso podrá tener varios alumnos, pero un alumno no estará inscrito en más de un curso.

Los alumnos tendrán esta información:

```tsx
// Creamos dos estudiantes
  const student1 = {
    firstName: "Juan",
    lastName: "Perez",
  };

  const student2 = {
    firstName: "Ana",
    lastName: "Lopez",
  };
```

Y los cursos serán así:

```tsx
const course = {
    name: "Matemáticas",
    department: "Números",
    students: [student1Entity, student2Entity]
  };
```

Sigue los pasos de los videos vistos en esta sesión y crea:

- CRUD de estudiantes
- CRUD de cursos
- Seed de estudiantes y cursos

Recuerda que puedes encontrar en este repositorio todo el código que hemos visto durante la sesión:

<https://github.com/The-Valley-School/node-s14-typeorm-for-sql>