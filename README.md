# Marten Code Generation Issue

## Setup

This repro requires a working PostgreSQL server and database.
Simply run this command to create a new PostgreSQL server in Docker

```
docker run --name wolverine-repro -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -p 5432:5432 -d postgres
```

## Reproduction Steps

1. Open command line in `/src/Repro.Api` directory
2. Run `dotnet run -- codegen write`
3. Open the solution
4. Set a breakpoint inside `UpdateTodoEndpoint.Update()`
5. Start the application
6. Go to the Swagger UI (http://localhost:5255/swagger)
7. Create a new Todo

   ```
   PUT /todos
   {
      "description": "Foo"
   }
   ```

8. Copy the ID from the response of step 5.
9. Update the existing Todo with a new description

   ```
   POST /todos/{id}
   {
      "description": "Bar"
   }
   ```

10. Now the breakpoint should be hit, but the Todo could not be loaded properly (description is missing)

If we inspect the generated code, we can clearly see some fishy code like this
```
public Repro.Api.Domain.Todo Create(Marten.Events.IEvent @event, Marten.IQuerySession session)
{
    var todo = new Repro.Api.Domain.Todo();
    switch (@event)
    {
        case Marten.Events.IEvent<Repro.Api.Domain.Events.TodoCreated> event_TodoCreated1:
            todo = Repro.Api.Domain.Todo.Create(event_TodoCreated1.Data);
            break;
    }

    return null;
}
```

If you don't use static code generation, everything works fine. Then the generated code looks like this

```
public Repro.Api.Domain.Todo Create(Marten.Events.IEvent @event, Marten.IQuerySession session)
{
    switch (@event)
    {
        case Marten.Events.IEvent<Repro.Api.Domain.Events.TodoCreated> event_TodoCreated1:
            return Repro.Api.Domain.Todo.Create(event_TodoCreated1.Data);
            break;
    }

    return null;
}
```

## Teardown

Remove the created PostgreSQL server with this command

```
docker rm -f -v wolverine-repro
```
