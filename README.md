# TodoApi3

This is a C# API project.

https://learn.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-8.0&tabs=visual-studio-code#create-an-api-project:~:text=Open%20the%20integrated,Visual%20Studio%20Code.

- Open Git bash.  Download the most recent version if you do not have it.
- Make sure you have visual studio code installed.
- Change directories to the folder that will contain the project folder.
- Run the following commands:
```
dotnet new web -o TodoApi3
cd TodoApi3
code -r ../TodoApi3
```
- If a dialogue box appears, select **Yes**
- Run this command to trust the certificate:
```
dotnet dev-certs https --trust
```
- Select **Yes** if you agree to trust the certificate.
- Press to **Run** and then **Run without debugging** to run the app.  Make sure to install the C sharp extension.
NuGet-packages
==============
- Run the commands
```
dotnet add package Microsoft.EntityFrameworkCore.InMemory
dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore
```
- Create a folder named ```Todo.cs``` with this code:
```
public class Todo
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```
- Create a file named ```TodoDb.cs with this code:
```
using Microsoft.EntityFrameworkCore;

class TodoDb : DbContext
{
    public TodoDb(DbContextOptions<TodoDb> options)
        : base(options) { }

    public DbSet<Todo> Todos => Set<Todo>();
}
```
- Replace the contents of ```Program.cs``` with this code:
```
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<TodoDb>(opt => opt.UseInMemoryDatabase("TodoList"));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();
var app = builder.Build();

app.MapGet("/todoitems", async (TodoDb db) =>
    await db.Todos.ToListAsync());

app.MapGet("/todoitems/complete", async (TodoDb db) =>
    await db.Todos.Where(t => t.IsComplete).ToListAsync());

app.MapGet("/todoitems/{id}", async (int id, TodoDb db) =>
    await db.Todos.FindAsync(id)
        is Todo todo
            ? Results.Ok(todo)
            : Results.NotFound());

app.MapPost("/todoitems", async (Todo todo, TodoDb db) =>
{
    db.Todos.Add(todo);
    await db.SaveChangesAsync();

    return Results.Created($"/todoitems/{todo.Id}", todo);
});

app.MapPut("/todoitems/{id}", async (int id, Todo inputTodo, TodoDb db) =>
{
    var todo = await db.Todos.FindAsync(id);

    if (todo is null) return Results.NotFound();

    todo.Name = inputTodo.Name;
    todo.IsComplete = inputTodo.IsComplete;

    await db.SaveChangesAsync();

    return Results.NoContent();
});

app.MapDelete("/todoitems/{id}", async (int id, TodoDb db) =>
{
    if (await db.Todos.FindAsync(id) is Todo todo)
    {
        db.Todos.Remove(todo);
        await db.SaveChangesAsync();
        return Results.NoContent();
    }

    return Results.NotFound();
});

app.Run();
```
Swagger 
=======
- Run this command:
```
dotnet add package NSwag.AspNetCore
```
- Add the following highlighted code before app is defined in line var app = builder.Build();
```
using NSwag.AspNetCore;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<TodoDb>(opt => opt.UseInMemoryDatabase("TodoList"));
builder.Services.AddDatabaseDeveloperPageExceptionFilter();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddOpenApiDocument(config =>
{
    config.DocumentName = "TodoAPI";
    config.Title = "TodoAPI v1";
    config.Version = "v1";
});
var app = builder.Build();
```
- Add the following highlighted code to the next line after app is defined in line var app = builder.Build();
```
var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.UseOpenApi();
    app.UseSwaggerUi(config =>
    {
        config.DocumentTitle = "TodoAPI";
        config.Path = "/swagger";
        config.DocumentPath = "/swagger/{documentName}/swagger.json";
        config.DocExpansion = "list";
    });
}
```
- Run the app.
- Go to https://localhost:```your_port_number```/swagger/index.html
- Select **Post/todoitems** > **Try it out**
- Enter the following without specifying an id:
```
{
  "name":"walk dog",
  "isComplete":true
}
```
- Select **Execute**.
- Test the endpoints for **GET**, **PUT** and **DELETE**.