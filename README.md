// Program.cs

// --- 1. Data Model (Correction: Added 'global' to make it accessible) ---
// Defining the record globally is the correct pattern for minimal API top-level statements.
global record Todo(int Id, string Title, bool IsComplete);

// --- 2. In-Memory Data Store and Initial Setup ---
// The 'todos' list acts as our database for this simple assignment.
var todos = new List<Todo>
{
    new Todo(1, "Create ASP.NET API", true),
    new Todo(2, "Debug and test endpoints", false),
    new Todo(3, "Submit assignment", false)
};

// Create the web application builder
var builder = WebApplication.CreateBuilder(args);

// --- 3. Service Configuration (Necessary for DI/Swagger) ---
// Add services to the container.
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(); // API documentation

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

// Security Best Practice: Enforce HTTPS redirection
app.UseHttpsRedirection();

// --- 4. API Endpoints (CRUD Operations) ---

// GET /todos: Retrieve all To-Do items
app.MapGet("/todos", () =>
{
    return Results.Ok(todos);
})
.WithName("GetAllTodos")
.WithDescription("Retrieves a list of all existing To-Do items.");

// GET /todos/{id}: Retrieve a specific To-Do item
app.MapGet("/todos/{id:int}", (int id) =>
{
    var todo = todos.FirstOrDefault(t => t.Id == id);
    return todo is null ? Results.NotFound() : Results.Ok(todo);
})
.WithName("GetTodoById")
.WithDescription("Retrieves a single To-Do item by ID.");

// POST /todos: Create a new To-Do item
app.MapPost("/todos", (Todo newTodo) =>
{
    // Generate a new unique ID
    int newId = todos.Any() ? todos.Max(t => t.Id) + 1 : 1;
    var todoToAdd = newTodo with { Id = newId }; // Use 'with' to create a new record with the new Id

    todos.Add(todoToAdd);

    // Return 201 Created status and the location of the new resource
    return Results.Created($"/todos/{todoToAdd.Id}", todoToAdd);
})
.WithName("CreateTodo")
.WithDescription("Creates a new To-Do item.");

// PUT /todos/{id}: Update an existing To-Do item
app.MapPut("/todos/{id:int}", (int id, Todo updatedTodo) =>
{
    var existingTodo = todos.FirstOrDefault(t => t.Id == id);
    if (existingTodo is null)
    {
        return Results.NotFound();
    }

    // Remove the old item and add the new one
    todos.Remove(existingTodo);
    // Ensure the updatedTodo uses the correct route ID
    todos.Add(updatedTodo with { Id = id }); 

    return Results.NoContent(); // 204 success with no content
})
.WithName("UpdateTodo")
.WithDescription("Updates an existing To-Do item by ID.");

// DELETE /todos/{id}: Delete a To-Do item
app.MapDelete("/todos/{id:int}", (int id) =>
{
    int removedCount = todos.RemoveAll(t => t.Id == id);

    return removedCount == 0 ? Results.NotFound() : Results.NoContent();
})
.WithName("DeleteTodo")
.WithDescription("Deletes a To-Do item by ID.");

// --- 5. Run the Application ---
app.Run();
