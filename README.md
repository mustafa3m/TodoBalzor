// code : 


Pages : 


  TodoList.razor

@page "/todo"

@inject AuthService AuthService

@inject NavigationManager NavigationManager

@inject TodoService TodoService


<div class="container mt-4">

    <h3 class="text-center mb-4">Application Todo</h3>


    @if (isLoading)

    {

        <div class="text-center">Loading inn...</div>

    }

    else if (!isAuthenticated)

    {

        <div class="alert alert-warning text-center">

            Du må være logget inn for å se Todo-listen.

        </div>

    }

    else

    {

        <div class="d-flex flex-column flex-md-row justify-content-center mb-4">

            <button class="btn btn-primary mx-2 mb-2 mb-md-0" @onclick="ShowTodos">Show Todos</button>

            <button class="btn btn-success mx-2" @onclick="ShowCompleted">Show Completed</button>

        </div>


        <div class="mt-4">

            <DynamicComponent Type="currentComponent" Parameters="componentParameters" />

        </div>

    }

</div>


@code {

    private Type currentComponent = typeof(TodoComponent);

    private Dictionary<string, object> componentParameters = new Dictionary<string, object>();

    private bool isAuthenticated;

    private bool isLoading = true;


    protected override async Task OnInitializedAsync()

    {

        isAuthenticated = await AuthService.IsAuthenticated();

        isLoading = false;


        if (!isAuthenticated)

        {

            NavigationManager.NavigateTo("/login"); // Omdiriger til innlogging

        }

    }


    private void ShowTodos() => currentComponent = typeof(TodoComponent);

    private void ShowCompleted() => currentComponent = typeof(CompletedComponent);

}


 



   TodoComponent.razor

@page "/todos"

@inject TodoService TodoService

@inject IJSRuntime JS // Inject JavaScript runtime

@inject NavigationManager navigation


<h3 class="text-center mb-4">Todo List</h3>


<div class="container mt-4">

    <div class="input-group mb-3">

        <input class="form-control " @bind="newTodoTitle" placeholder="Enter a new task..." />

        <button class="btn btn-primary" @onclick="AddTodo">Add Todo</button>

    </div>


    <ul class="list-group">

        @foreach (var todo in TodoService.Todos)

        {

            <li class="list-group-item d-flex justify-content-between align-items-center">

                <div class="d-flex align-items-center">

                    <input class="form-check-input me-2" type="checkbox" @onclick="() => CompleteTodo(todo)" />

                    <span>@todo.Title</span>

                </div>

                <button class="btn btn-danger btn-sm" @onclick="() => OpenDeleteModal(todo)">Delete</button>

            </li>

        }

    </ul>


    <!-- Modal for deletion confirmation -->

    <div class="modal fade" id="deleteModal" tabindex="-1" aria-labelledby="deleteModalLabel" aria-hidden="true">

        <div class="modal-dialog">

            <div class="modal-content">

                <div class="modal-header">

                    <h5 class="modal-title" id="deleteModalLabel">Delete Confirmation</h5>

                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>

                </div>

                <div class="modal-body">

                    Are you sure you want to delete this task?

                </div>

                <div class="modal-footer">

                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cancel</button>

                    <button type="button" class="btn btn-danger" @onclick="DeleteTodo">Delete</button>

                </div>

            </div>

        </div>

    </div>

</div>



    @code {

        private string? newTodoTitle;

        private TodoItem? currentTodoToDelete; // Holds the item for deletion





        private void AddTodo()

        {

            if (!string.IsNullOrWhiteSpace(newTodoTitle))

            {

                TodoService.AddTodo(newTodoTitle);

                newTodoTitle = string.Empty; // Clear input after adding

            }

        }



        private async Task OpenDeleteModal(TodoItem todo)

        {

            currentTodoToDelete = todo; // Set the current todo to delete

            await JS.InvokeVoidAsync("modalHelpers.showModal", "deleteModal"); // Show the modal

        }


        private async Task DeleteTodo()

        {

            if (currentTodoToDelete != null)

            {

                TodoService.RemoveTodoItem(currentTodoToDelete);

                currentTodoToDelete = null; // Clear the reference after deletion

                await JS.InvokeVoidAsync("modalHelpers.hideModal", "deleteModal"); // Hide the modal after deletion

                StateHasChanged(); // Force the UI to refresh

            }

        }


        private void CompleteTodo(TodoItem todo)

        {

            TodoService.MarkAsComplete(todo);

        }


        protected override async Task OnAfterRenderAsync(bool firstRender)

        {

            if (firstRender)

            {

                await JS.InvokeVoidAsync("modalHelpers.hideModal", "deleteModal");

            }

        }

    }




     Logout.razor

@page "/logout"

@inject AuthService AuthService

@inject NavigationManager navigation




<div class="container mt-4 text-center">

    <h3 class="mb-4">Logging out...</h3>

    <p class="text-muted">Please wait while we log you out.</p>

    <div class="spinner-border" role="status">

        <span class="visually-hidden">Loading...</span>

    </div>

</div>



@code {

    protected override async Task OnInitializedAsync()

    {

        await AuthService.Logout();

        navigation.NavigateTo("/login");

    }

}





    Login.razor


@page "/login"

@using System.ComponentModel.DataAnnotations;

@inject AuthService AuthService

@inject NavigationManager navigation


<h3>Login</h3>

@if (!string.IsNullOrEmpty(errorMessage))

{

    <div class="alert alert-danger">@errorMessage</div>

}


<EditForm Model="userLogin" OnValidSubmit="HandleLogin">

    <DataAnnotationsValidator />

    <ValidationSummary />

    <div class="mb-3">

        <label class="form-label">Username</label>

        <InputText id="username" class="form-control" @bind-Value="userLogin.Username" />

    </div>

    <div class="mb-3">

        <label class="form-label">Password</label>

        <InputText id="password" type="password" class="form-control" @bind-Value="userLogin.Password" />

    </div>

    <button type="submit" class="btn btn-primary">Login</button>

</EditForm>




@code {


    private UserLogin userLogin = new UserLogin();

    private string errorMessage;


    private async Task HandleLogin()

    {

        var success = await AuthService.Login(userLogin.Username, userLogin.Password);

        if (success)

        {

            navigation.NavigateTo("/todo");

        }

        else

        {

            errorMessage = "Invalid username or password. Please try again!";

        }

    }



    private class UserLogin

    {

        [Required(ErrorMessage = "Username is required, please!")]

        [StringLength(20, ErrorMessage = "Username cannot exceed 20 characters")]

        public string Username { get; set; }


        [Required(ErrorMessage = "Password is required, i wont let u go without it!")]

        public string Password { get; set; }

    }

}



   CompletedComponent .razor 


@page "/Completed"



@inject TodoService? TodoService


<div class="container mt-4">

    <h3 class="text-center mb-4">Completed Tasks</h3>


    <ul class="list-group">

        @foreach (var todo in TodoService.GetCompletedTodos())

        {

            <li class="list-group-item d-flex justify-content-between align-items-center">

                <span>@todo.Title</span>

                <button class="btn btn-warning btn-sm" @onclick="() => UndoComplete(todo)">Undo</button>

            </li>

        }

    </ul>

</div>


@code {

    private void UndoComplete(TodoItem todo)

    {

        TodoService.UndoComplete(todo);

    }

}






Service


AuthService.cs


using Blazored.LocalStorage;

using Microsoft.AspNetCore.Components.Forms;


public class AuthService

{

    private readonly ILocalStorageService _localStorage;


    private const string TokenKey = "authToken";


    public event Action OnAuthStateChanged;


    public AuthService(ILocalStorageService localStorage)

    {

        _localStorage = localStorage;

    }


    public async Task<bool> Login(string username, string password)

    {

        // statisk brukernavn og passord for demonstrasjon

        if (username == "admin" && password == "password")

        {

            await _localStorage.SetItemAsync(TokenKey, "fake-jwt-token");

            NotifyAuthStateChanged();

            return true;

        }


        return false;

    }


    public async Task Logout()

    {

        await _localStorage.RemoveItemAsync(TokenKey);

        NotifyAuthStateChanged();

    }


    public async Task<bool> IsAuthenticated()

    {

        var token = await _localStorage.GetItemAsync<string>(TokenKey);

        return !string.IsNullOrEmpty(token);

    }


    private void NotifyAuthStateChanged()

    {

        OnAuthStateChanged.Invoke();

    }

}



2 FOIS 


TodoService.cs

using System.Collections.Generic;

using System.Linq;




public class TodoService

{

    private readonly List<TodoItem> _todos = new List<TodoItem>();


    // Expose Todos as a read-only collection

    public IReadOnlyList<TodoItem> Todos => _todos;


    // Add a new todo

    public void AddTodo(string title)

    {

        var todo = new TodoItem { Title = title, IsCompleted = false };

        _todos.Add(todo);

    }


    // Mark a todo item as completed

    public void MarkAsComplete(TodoItem todo)

    {

        var existingTodo = _todos.FirstOrDefault(t => t == todo);

        if (existingTodo != null)

        {

            existingTodo.IsCompleted = true;

        }

    }


    // Remove a todo item

    public void RemoveTodoItem(TodoItem todo)

    {

        _todos.Remove(todo);

        Console.WriteLine($"Todo list after deletion: {string.Join(", ", _todos.Select(t => t.Title))}");

    }


    // Get the list of completed todos

    public List<TodoItem> GetCompletedTodos()

    {

        return _todos.Where(todo => todo.IsCompleted).ToList();

    }


    public void UndoComplete(TodoItem todo)

    {

        todo.IsCompleted = false;

    }

}



TodoItem.cs




public class TodoItem

{

    public int Id { get; set; } // Use an int for Id

    public string? Title { get; set; } // Nullable, if it's expected to be null at some point

    public bool IsCompleted { get; set; }

}



Program.cs

using Blazored.LocalStorage;

using Microsoft.AspNetCore.Components.Web;

using Microsoft.AspNetCore.Components.WebAssembly.Hosting;

using Microsoft.Extensions.DependencyInjection;

using TodoApplikasjonenDelFire;

using TodoApplikasjonenDelFire.Pages;


var builder = WebAssemblyHostBuilder.CreateDefault(args);

builder.RootComponents.Add<App>("#app");


builder.Services.AddSingleton<TodoService>();

builder.Services.AddBlazoredLocalStorage();

builder.Services.AddScoped<AuthService>();


await builder.Build().RunAsync();





js


modalHelpers.js

window.modalHelpers = {

    showModal: function (modalId) {

        var modalElement = document.getElementById(modalId);

        if (modalElement) {

            var modal = new bootstrap.Modal(modalElement);

            modal.show();

        } else {

            console.error(`Modal with ID ${modalId} not found.`);

        }

    },

    hideModal: function (modalId) {

        var modalElement = document.getElementById(modalId);

        if (modalElement) {

            var modal = bootstrap.Modal.getInstance(modalElement);

            if (modal) {

                modal.hide();

            } else {

                console.error(`No instance of modal with ID ${modalId} found.`);

            }

        } else {

            console.error(`Modal with ID ${modalId} not found.`);

        }

    }

};


css index


index.html

<!DOCTYPE html>

<html lang="en">




<head>

    <meta charset="utf-8" />

    <meta name="viewport" content="width=device-width, initial-scale=1.0" />

    <title>TodoApplikasjonenDelFire</title>

    <base href="/" />

    <link rel="stylesheet" href="css/bootstrap/bootstrap.min.css" />

    <link rel="stylesheet" href="css/app.css" />

    <link rel="icon" type="image/png" href="favicon.png" />

    <link href="TodoApplikasjonenDelFire.styles.css" rel="stylesheet" />

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>


</head>



<body>

    <div id="app">

        <svg class="loading-progress">

            <circle r="40%" cx="50%" cy="50%" />

            <circle r="40%" cx="50%" cy="50%" />

        </svg>

        <div class="loading-progress-text"></div>

    </div>


    <div id="blazor-error-ui">

        An unhandled error has occurred.

        <a href="" class="reload">Reload</a>

        <a class="dismiss">🗙</a>

    </div>



    <script src="https://code.jquery.com/jquery-3.6.0.min.js" integrity="sha384-KyZXEAg3QhqLMpG8r+Knujsl5/8K74cLlQ/0AMPTqV+O3C6C6gHs3Yt6c6/2OT2l" crossorigin="anonymous"></script>

    <script src="https://cdn.jsdelivr.net/npm/@popperjs/core@2.11.6/dist/umd/popper.min.js" integrity="sha384-oBqDVmMz4fnFO9gybL3CLd5Qx8UChgTfK8xt9Y67hs5J2ePmA0R/J4LkcTQiXKh" crossorigin="anonymous"></script>


    <!-- Load Bootstrap bundle first -->

    <script src="js/bootstrap/bootstrap.bundle.min.js"></script>


    <!-- Load your custom JavaScript file after Bootstrap -->

    <script src="js/modalHelpers.js"></script>


    <!-- Add Blazor's WebAssembly script if using Blazor WASM -->

    <script src="_framework/blazor.webassembly.js"></script>

</body>


</html>




app.css

html, body {

    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;

}


h1:focus {

    outline: none;

}


a, .btn-link {

    color: #0071c1;

}


.btn-primary {

    color: #fff;

    background-color: #1b6ec2;

    border-color: #1861ac;

}


.btn:focus, .btn:active:focus, .btn-link.nav-link:focus, .form-control:focus, .form-check-input:focus {

    box-shadow: 0 0 0 0.1rem white, 0 0 0 0.25rem #258cfb;

}


.content {

    padding-top: 1.1rem;

}


.valid.modified:not([type=checkbox]) {

    outline: 1px solid #26b050;

}


.invalid {

    outline: 1px solid red;

}


.validation-message {

    color: red;

}


#blazor-error-ui {

    background: lightyellow;

    bottom: 0;

    box-shadow: 0 -1px 2px rgba(0, 0, 0, 0.2);

    display: none;

    left: 0;

    padding: 0.6rem 1.25rem 0.7rem 1.25rem;

    position: fixed;

    width: 100%;

    z-index: 1000;

}


    #blazor-error-ui .dismiss {

        cursor: pointer;

        position: absolute;

        right: 0.75rem;

        top: 0.5rem;

    }


.blazor-error-boundary {

    background: url(data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNTYiIGhlaWdodD0iNDkiIHhtbG5zPSJodHRwOi8vd3d3LnczLm9yZy8yMDAwL3N2ZyIgeG1sbnM6eGxpbms9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkveGxpbmsiIG92ZXJmbG93PSJoaWRkZW4iPjxkZWZzPjxjbGlwUGF0aCBpZD0iY2xpcDAiPjxyZWN0IHg9IjIzNSIgeT0iNTEiIHdpZHRoPSI1NiIgaGVpZ2h0PSI0OSIvPjwvY2xpcFBhdGg+PC9kZWZzPjxnIGNsaXAtcGF0aD0idXJsKCNjbGlwMCkiIHRyYW5zZm9ybT0idHJhbnNsYXRlKC0yMzUgLTUxKSI+PHBhdGggZD0iTTI2My41MDYgNTFDMjY0LjcxNyA1MSAyNjUuODEzIDUxLjQ4MzcgMjY2LjYwNiA1Mi4yNjU4TDI2Ny4wNTIgNTIuNzk4NyAyNjcuNTM5IDUzLjYyODMgMjkwLjE4NSA5Mi4xODMxIDI5MC41NDUgOTIuNzk1IDI5MC42NTYgOTIuOTk2QzI5MC44NzcgOTMuNTEzIDI5MSA5NC4wODE1IDI5MSA5NC42NzgyIDI5MSA5Ny4wNjUxIDI4OS4wMzggOTkgMjg2LjYxNyA5OUwyNDAuMzgzIDk5QzIzNy45NjMgOTkgMjM2IDk3LjA2NTEgMjM2IDk0LjY3ODIgMjM2IDk0LjM3OTkgMjM2LjAzMSA5NC4wODg2IDIzNi4wODkgOTMuODA3MkwyMzYuMzM4IDkzLjAxNjIgMjM2Ljg1OCA5Mi4xMzE0IDI1OS40NzMgNTMuNjI5NCAyNTkuOTYxIDUyLjc5ODUgMjYwLjQwNyA1Mi4yNjU4QzI2MS4yIDUxLjQ4MzcgMjYyLjI5NiA1MSAyNjMuNTA2IDUxWk0yNjMuNTg2IDY2LjAxODNDMjYwLjczNyA2Ni4wMTgzIDI1OS4zMTMgNjcuMTI0NSAyNTkuMzEzIDY5LjMzNyAyNTkuMzEzIDY5LjYxMDIgMjU5LjMzMiA2OS44NjA4IDI1OS4zNzEgNzAuMDg4N0wyNjEuNzk1IDg0LjAxNjEgMjY1LjM4IDg0LjAxNjEgMjY3LjgyMSA2OS43NDc1QzI2Ny44NiA2OS43MzA5IDI2Ny44NzkgNjkuNTg3NyAyNjcuODc5IDY5LjMxNzkgMjY3Ljg3OSA2Ny4xMTgyIDI2Ni40NDggNjYuMDE4MyAyNjMuNTg2IDY2LjAxODNaTTI2My41NzYgODYuMDU0N0MyNjEuMDQ5IDg2LjA1NDcgMjU5Ljc4NiA4Ny4zMDA1IDI1OS43ODYgODkuNzkyMSAyNTkuNzg2IDkyLjI4MzcgMjYxLjA0OSA5My41Mjk1IDI2My41NzYgOTMuNTI5NSAyNjYuMTE2IDkzLjUyOTUgMjY3LjM4NyA5Mi4yODM3IDI2Ny4zODcgODkuNzkyMSAyNjcuMzg3IDg3LjMwMDUgMjY2LjExNiA4Ni4wNTQ3IDI2My41NzYgODYuMDU0N1oiIGZpbGw9IiNGRkU1MDAiIGZpbGwtcnVsZT0iZXZlbm9kZCIvPjwvZz48L3N2Zz4=) no-repeat 1rem/1.8rem, #b32121;

    padding: 1rem 1rem 1rem 3.7rem;

    color: white;

}


    .blazor-error-boundary::after {

        content: "An error has occurred."

    }


.loading-progress {

    position: relative;

    display: block;

    width: 8rem;

    height: 8rem;

    margin: 20vh auto 1rem auto;

}


    .loading-progress circle {

        fill: none;

        stroke: #e0e0e0;

        stroke-width: 0.6rem;

        transform-origin: 50% 50%;

        transform: rotate(-90deg);

    }


        .loading-progress circle:last-child {

            stroke: #1b6ec2;

            stroke-dasharray: calc(3.141 * var(--blazor-load-percentage, 0%) * 0.8), 500%;

            transition: stroke-dasharray 0.05s ease-in-out;

        }


.loading-progress-text {

    position: absolute;

    text-align: center;

    font-weight: bold;

    inset: calc(20vh + 3.25rem) 0 auto 0.2rem;

}


    .loading-progress-text:after {

        content: var(--blazor-load-percentage-text, "Loading");

    }


code {

    color: #c02d76;

}



@media (max-width: 576px) {

    .list-group-item {

        font-size: 14px; /* Smaller font on mobile */

    }

}




Layout


NavMenu.razor

@inject AuthService AuthService


<div class="top-row ps-3 navbar navbar-expand-lg navbar-dark bg-dark">

    <div class="container-fluid">

        <a class="navbar-brand" href="">TodoApplikasjonenDelTre</a>

        <button title="Navigation menu" class="navbar-toggler" @onclick="ToggleNavMenu">

            <span class="navbar-toggler-icon"></span>

        </button>

    </div>

</div>


<div class="@NavMenuCssClass nav-scrollable" @onclick="ToggleNavMenu">

    <nav class="flex-column">

        <div class="nav-item px-3">

            <NavLink class="nav-link" href="" Match="NavLinkMatch.All">

                <span class="bi bi-house-door-fill-nav-menu" aria-hidden="true"></span> Home

            </NavLink>

        </div>

        <div class="nav-item px-3">

            <NavLink class="nav-link" href="todo">

                <span class="bi bi-list-nested-nav-menu" aria-hidden="true"></span> Todo

            </NavLink>

        </div>


        @if (isAuthenticated)

        {

            <div class="nav-item px-3">

                <NavLink class="nav-link" href="login">

                    Login

                </NavLink>

            </div>


           

           

     

           

        }

        else

        {

            <div class="nav-item px-3">

                <NavLink class="nav-link" href="logout">

                    Logout

                </NavLink>

            </div>

       

           


           

        }

    </nav>

</div>


@code {

    private bool collapseNavMenu = true;

    private bool isAuthenticated;


    protected override async Task OnInitializedAsync()

    {

        isAuthenticated = await AuthService.IsAuthenticated();

        AuthService.OnAuthStateChanged += UpdateAuthState;

    }


    private void UpdateAuthState()

    {

        InvokeAsync(async () =>

        {

            isAuthenticated = await AuthService.IsAuthenticated();

            StateHasChanged();

        });

    }


    private string? NavMenuCssClass => collapseNavMenu ? "collapse navbar-collapse" : "navbar-collapse";


    private void ToggleNavMenu()

    {

        collapseNavMenu = !collapseNavMenu;

    }


    public void Dispose()

    {

        AuthService.OnAuthStateChanged -= UpdateAuthState;

    }

}




