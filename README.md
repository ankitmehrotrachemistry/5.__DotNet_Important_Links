## Tell me about your Project
 
In my previous role, I developed the backend for a real-time multiplayer game using .NET Core. My primary responsibility was designing and developing scalable APIs that supported player interactions, matchmaking, and in-game events. We used a microservices architecture to handle key features like player matchmaking, real-time communication, and leaderboards. For real-time gameplay, I integrated SignalR to manage WebSocket connections, allowing players to communicate their moves instantly. 
     
We stored player profiles and game data in SQL Server while using Redis for caching active game sessions and leaderboards. The system was deployed on Azure with Kubernetes, allowing it to scale dynamically based on player demand. This ensured low-latency gameplay for over 1 million players. We also leveraged Azure Cloud Services to scale our servers dynamically based on player activity, ensuring low latency and high availability, especially during peak times.
     
For security, we used JWT tokens for player authentication, ensuring secure and stateless communication between the client and server.  

## Architecture of a Multiplayer Game Backend:
    
**Client-Server Model:** The client (game app) communicates with the server (backend) for operations like player registration, authentication, game state synchronization, and matchmaking. The server is responsible for managing game logic, player connections, and real-time communication.    
**Microservices Architecture:** .NET Core supports building modular systems where game functionalities like matchmaking, leaderboards, and player stats are managed in separate services. This architecture enhances scalability and maintainability.

## Real-Time Communication with SignalR:

For multiplayer games, real-time communication between the server and players is crucial. SignalR is a library built into ASP.NET Core that allows real-time communication via WebSockets or other protocols.    

**Use Case:** It is used to broadcast messages to all players (like game state updates) or send targeted messages (like player-specific notifications or movements).    
**Example:** In a multiplayer game where players need to synchronize actions (e.g., shooting, moving), SignalR helps keep all clients in sync    

```csharp
public class GameHub : Hub
{
    public async Task SendMove(string player, string move)
    {
        // Broadcast the player's move to all other connected players
        await Clients.Others.SendAsync("ReceiveMove", player, move);
    }
}
```

# DotNet_Important_Links

## 2). CORS (Cross Origin Resource Sharing)

- Cross-Origin Resource Sharing (CORS) is a security feature that allows or restricts web applications running at one domain to make requests for resources from a different domain.  
- In a .NET Core API, CORS is implemented using middleware.   
- A CORS policy defines which domains, HTTP methods, headers, and other options are permitted. 
- You need to register the CORS middleware in the Configure method of Startup.cs .

 [Cross-Origin Resource Sharing in .NET](https://medium.com/@darshana-edirisinghe/cross-origin-resource-sharing-in-net-f8d0aa802b5f)

```csharp
app.UseCors("AllowSpecificOrigin");
```

I needed to make an authenticated cross-origin request to an ASP.NET Core Identity application that was using cookie authentication. I showed how to configure the app to allow CORS requests, and how to use the JavaScript fetch() API to call the request. However, this still doesn't work as the ASP.NET Core Identity cookie is marked as SameSite=Lax (for good security reasons). In the final section I showed how to configure Identity to mark the cookie as SameSite=None. This has security implications, so you should be wary about doing this is in your production applications!

[Making authenticated cross-origin requests with ASP.NET Core Identity](https://andrewlock.net/making-authenticated-cross-origin-requests-with-aspnetcore-identity/)

<p align="center">
  <img src="https://github.com/user-attachments/assets/cdc2c0f6-481a-4285-93e6-ae8c82808d75" width="400" height="250" />
</p>

## 3). How to handle Exception except try-Catch?

- Instead of scattering try-catch blocks throughout your code, .NET Core allows you to handle exceptions globally.  
- This is done using :  
    A). Middleware  
    B). Filters     

**A). In ASP.NET Core, you can use Middleware for global exception handling.**

```csharp
public class ErrorHandlingMiddleware
{
    private readonly RequestDelegate _next;

    public ErrorHandlingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            HandleException(context, ex);
        }
    }

    private static void HandleException(HttpContext context, Exception ex)
    {
        // Log and respond to the error accordingly
        context.Response.StatusCode = 500;
        context.Response.ContentType = "application/json";
        // Serialize and write the exception details to the response
    }
}

// In Startup.cs
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<ErrorHandlingMiddleware>();
}
```

**B). We’ll focus on using a Global Filter.**

```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    public void OnException(ExceptionContext context)
    {
        var statusCode = context.Exception switch
        {
            NotFoundException => StatusCodes.Status404NotFound,

            ValidationException => StatusCodes.Status400BadRequest,

            UnauthorizedAccessException => StatusCodes.Status401Unauthorized,

            _ => StatusCodes.Status500InternalServerError
        };

        context.Result = new ObjectResult(new
        {
            error = context.Exception.Message,
            stackTrace = context.Exception.StackTrace
        })
        {
            StatusCode = statusCode
        };
    }
}
``` 
This filter will intercept exceptions and convert them to appropriate HTTP responses.

In the Startup.cs, register the global filter.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers(options =>
    {
        options.Filters.Add(new GlobalExceptionFilter());
    });
}
``` 

[Exception Handling in .NET Core Web API](https://medium.com/codenx/exception-handling-in-net-core-web-api-e0c4aad1db06)

[Middleware and Filters power in ASP.NET Core](https://binodmahto.medium.com/middleware-and-filters-power-in-asp-net-core-3c4e3349cedb)

## 4). Routing in WEB API and MVC

Routing in ASP.NET Core Web API is a powerful feature that allows you to define how HTTP requests are mapped to your API endpoints. It allows you to define the endpoints of your API and handle requests efficiently.

<p align="center">
  <img src="https://github.com/user-attachments/assets/a715b526-925d-42c9-a437-b069abab9763" width="300" height="250" />
</p>

In ASP.NET Core, routing is handled by the routing middleware, which is configured in the Startup.cs file. There are two primary types of routing:   
A). Convention-based Routing   
B). Attribute Routing  

**A). Convention-based Routing :** Convention-based routing defines routes globally in the Startup.cs file. This approach is useful when you want to apply a consistent routing pattern across your entire application.

**Defining Routes in Startup.cs**
In the Configure method of the Startup.cs file, you define routes using the UseEndpoints method:  
```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

**B). Attribute Routing :** Attribute routing allows you to define routes directly on controller actions using attributes. This approach offers more flexibility and fine-grained control over your API endpoints.  

**Using Route Attributes**
You can use the [Route] attribute to define routes on controllers and actions:

```csharp
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAllProducts()
    {
        // Implementation
    }

    [HttpGet("{id}")]
    public IActionResult GetProductById(int id)
    {
        // Implementation
    }
}
```

🎮 In a game server built using .NET Core, routing plays a crucial role in handling requests from clients (game players) and directing them to appropriate controllers or actions.

There are two primary ways to define routes in Web API:

**A). Convention-based routing** : It is typically set up in the Startup.cs file using the MapControllerRoute method.

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllerRoute(
            name: "default",
            pattern: "api/{controller=Home}/{action=Index}/{id?}");
    });
}
```

For example, if the player requests api/players/GetPlayer/123, the framework maps this to the PlayersController's GetPlayer method with the parameter 123.

**B). Attribute Based Routing** :
In attribute routing, you specify the URL pattern directly in the controller.

```csharp
[ApiController]
[Route("api/[controller]")]
public class PlayersController : ControllerBase
{
    // GET api/players/{id}
    [HttpGet("{id}")]
    public IActionResult GetPlayer(string id)
    {
        // Logic to retrieve player details by ID
        var player = PlayerService.GetPlayerById(id);
        return Ok(player);
    }

    // POST api/players/create
    [HttpPost("create")]
    public IActionResult CreatePlayer([FromBody] Player newPlayer)
    {
        // Logic to create a new player
        PlayerService.CreatePlayer(newPlayer);
        return Ok(newPlayer);
    }
}
```
GET api/players/123: Retrieves the player with ID 123.
POST api/players/create: Creates a new player.

[Http Routing in ASP.NET Core Web API](https://medium.com/@nwonahr/routing-in-asp-net-core-web-api-c9c6dcae5cbd)

## 5). Middleware and it's working

- Middleware is a component that sits between the web server and the application’s request pipeline. It processes incoming requests and generates outgoing responses.  
- Middleware can be used to perform a wide range of tasks such as authentication, logging, error handling, routing, and more.  
- Middleware can be added and ordered in the pipeline using the **UseMiddleware() method** in the **Configure() method** of the Startup class.  
- Middleware in .NET Core is like a series of checkpoints or gatekeepers that a request must pass through before reaching the endpoint, and again on its way back as a response. They are essential components in the request pipeline, responsible for everything from logging, authentication, to response compression.

[.Net Core Middleware Explained](https://medium.com/@shubhadeepchat/net-core-middleware-explained-8c21bf646700)

<p align="center">
  <img src="https://github.com/user-attachments/assets/6cca0303-5287-4c16-96a4-310f80dffa35" width="500" height="250" />
</p>

🎮 Middlewares allow you to insert custom logic into the request/response pipeline of your game server.
In the context of a game server, middlewares can handle:

**Request/Response validation:** Validate requests from clients and responses sent back to clients.
**Authentication and Authorization:** Ensure that only authenticated users can access the game server.

Middleware components in .NET Core are implemented as part of the RequestDelegate pipeline.

**Common Use Cases for Middleware in Game Server Logic**

**A). Authentication and Authorization Middleware**
Authentication and Authorization Middleware verifies JWT tokens or other forms of authentication, checking if a player has permission to access the game server.

```csharp
public class AuthenticationMiddleware
{
    private readonly RequestDelegate _next;

    public AuthenticationMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        var token = context.Request.Headers["Authorization"].FirstOrDefault()?.Split(" ").Last();
        if (string.IsNullOrEmpty(token) || !ValidateToken(token))
        {
            context.Response.StatusCode = 401; // Unauthorized
            await context.Response.WriteAsync("Invalid Token");
            return;
        }

        await _next(context); // Pass to the next middleware if authenticated
    }

    private bool ValidateToken(string token)
    {
        // Token validation logic
        return true; // Assume token is valid
    }
}
```

**B). Request Rate Limiting**
Rate limiting middleware helps prevent denial-of-service attacks and ensures fair use. This is especially important during in-game events where many players may be interacting simultaneously.

```csharp
public class RateLimitingMiddleware
{
    private static Dictionary<string, DateTime> _requestTracker = new Dictionary<string, DateTime>();
    private readonly RequestDelegate _next;
    private readonly TimeSpan _throttlePeriod = TimeSpan.FromSeconds(1); // 1 request per second

    public RateLimitingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        string playerId = GetPlayerId(context); // Get player identifier (e.g., from headers)

        if (_requestTracker.TryGetValue(playerId, out var lastRequestTime))
        {
            if (DateTime.UtcNow - lastRequestTime < _throttlePeriod)
            {
                context.Response.StatusCode = 429; // Too many requests
                await context.Response.WriteAsync("Rate limit exceeded");
                return;
            }
        }

        _requestTracker[playerId] = DateTime.UtcNow;
        await _next(context);
    }

    private string GetPlayerId(HttpContext context)
    {
        return context.Request.Headers["Player-Id"].FirstOrDefault();
    }
}
```
**C). Logging and Performance Monitoring**
Track the requests coming to your game server, logging critical events like player actions, matchmaking requests, or game state updates.

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        var request = context.Request;
        Console.WriteLine($"Request: {request.Method} {request.Path}");

        var startTime = DateTime.UtcNow;
        await _next(context); // Pass control to the next middleware
        var duration = DateTime.UtcNow - startTime;

        Console.WriteLine($"Response Status: {context.Response.StatusCode}, Duration: {duration.TotalMilliseconds} ms");
    }
}
```

## 6). Create Custom Middleware

C# Code Example: Custom Logging Middleware 📝

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}

public static class LoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseLoggingMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<LoggingMiddleware>();
    }
}

// In Startup.cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseLoggingMiddleware();
    // Other middleware registrations
}
```

🎮 **You can also create custom middlewares for game-specific scenarios such as:**

**A). Game State Middleware**
Used to manage the player's current game state and persist data as they progress.

**B). Matchmaking Middleware**
Track requests for matchmaking and either enqueue them or assign players to a match when conditions are met.

Example Middleware for Matchmaking:

```csharp
public class MatchmakingMiddleware
{
    private static Queue<string> _waitingPlayers = new Queue<string>();
    private readonly RequestDelegate _next;

    public MatchmakingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        var playerId = context.Request.Headers["Player-Id"].FirstOrDefault();
        if (!string.IsNullOrEmpty(playerId))
        {
            if (_waitingPlayers.Count > 0)
            {
                var opponentId = _waitingPlayers.Dequeue();
                await context.Response.WriteAsync($"Match found! You are playing against {opponentId}");
            }
            else
            {
                _waitingPlayers.Enqueue(playerId);
                await context.Response.WriteAsync("Waiting for a match...");
            }
        }
        else
        {
            await _next(context);
        }
    }
}
```

**Register Middlewares in the Pipeline**
Once you have created your middleware, you need to register it in the request pipeline within the Startup.cs or Program.cs:

```csharp
public class Startup
{
    public void Configure(IApplicationBuilder app)
    {
        // Custom Middlewares
        app.UseMiddleware<LoggingMiddleware>();
        app.UseMiddleware<AuthenticationMiddleware>();
        app.UseMiddleware<RateLimitingMiddleware>();
        app.UseMiddleware<MatchmakingMiddleware>();

        // Exception Handling Middleware
        app.UseMiddleware<ExceptionHandlingMiddleware>();

        // Endpoints
        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```
[Custom Middlewares With Dependency Injection In .NET Framework](https://medium.com/@ofirbarak96/custom-middlewares-with-dependency-injection-in-net-framework-b18f5b935e4d)

## 7). OWIN Middleware

OWIN defines a standard interface between .NET web servers and web applications. The goal of the OWIN interface is to decouple server and application.

With .Net 4.7 I used to log my http request and response conditionally through the help of OWIN. 

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
     app.UseCors(builder => builder
                .AllowAnyOrigin()
                .AllowAnyMethod()
                .AllowAnyHeader());
     app.UseHttpsRedirection();
     app.UseMiddleware<HttpHandlerMiddleware>();
     app.UseDefaultFiles();
     app.UseStaticFiles();
}
```
The above OWIN specification describes the five parts (or roles) of the application called as software actors. They are Server, Web Framework, Web Application, Middleware, and Host.

<p align="center">
  <img src="https://github.com/user-attachments/assets/b307ab92-5583-4862-bdee-7d9c7c385494" width="500" height="250" />
</p>

[Introduction to OWIN](https://www.tektutorialshub.com/owin/introduction-to-owin/)

[Create OWIN Middleware](https://www.tektutorialshub.com/asp-net/asp-net-owin-middleware/)

[Controller Action Return Types in ASP.NET Core Web API](https://dotnettutorials.net/lesson/controller-action-return-types-core-web-api/)

[.Net Core Using Middleware to log http request/responses](https://theochiu2010.medium.com/net-core-using-middleware-to-log-http-request-responses-f60364e2880)

## 8). Dependency Injection

Dependency Injection is the design pattern that helps us to create an application which loosely coupled. The main advantage of DI (Dependency Injection) is our application is loosely coupled and has provided greater maintainability, testability, and also re-usability. 
ASP.NET Core is designed from scratch to support Dependency Injection. ASP.NET Core injects objects of dependency classes through constructor or method by using built-in IoC container.  
The built-in container is represented by **IServiceProvider interface** implementation that supports constructor injection by default. 
The built-in IoC container supports three kinds of lifetimes:

**a). Singleton:** IoC container will create and share a single instance of a service throughout the application's lifetime.  
**b). Transient:** The IoC container will create a new instance of the specified service type every time you ask for it.  
**c). Scoped:** IoC container will create an instance of the specified service type once per request and will be shared in a single request.  

[Dependency Injection With .NET Core](https://kusham1998.medium.com/dependency-injection-with-net-core-a6b33e74f6df)

ASP.NET Core framework includes extension methods for each types of lifetime; AddSingleton(), AddTransient() and AddScoped() methods for singleton, transient and scoped lifetime respectively.

The following example shows the ways of registering types (service) using extension methods.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ILog, MyConsoleLogger>();
    services.AddSingleton(typeof(ILog), typeof(MyConsoleLogger));

    services.AddTransient<ILog, MyConsoleLogger>();
    services.AddTransient(typeof(ILog), typeof(MyConsoleLogger));

    services.AddScoped<ILog, MyConsoleLogger>();
    services.AddScoped(typeof(ILog), typeof(MyConsoleLogger));
}
```

[Mastering Dependency Injection in .NET Core](https://medium.com/@vndpal/mastering-dependency-injection-in-net-core-94aea0a4ab6c)

**Dependency Injection Video Demonstration** ▶️    
[ASP.NET CORE Tutorial For Beginners 31 - Dependency Injection (DI) in Hindi](https://www.youtube.com/watch?v=3nnESO6I3iE)

🎮 In a game server, DI allows you to:

- Inject services for handling game logic, player management, matchmaking, etc.
- Swap implementations, such as switching from in-memory storage to a database.
- Manage the lifetime of objects (e.g., singleton for game state services, transient for per-request services).

Common Use Cases in Game Servers
In a game server, you might use DI for various core components:

**a. Game Logic Services :**  
Services that handle game mechanics, rules, and interactions between players.
Examples: Combat systems, scoring, event handling.

```csharp
public interface IGameService
{
    void StartGame(string gameId);
    void EndGame(string gameId);
    void UpdateGameState(string gameId, string playerAction);
}

public class GameService : IGameService
{
    private readonly IGameRepository _gameRepository;
    
    public GameService(IGameRepository gameRepository)
    {
        _gameRepository = gameRepository;
    }

    public void StartGame(string gameId)
    {
        // Initialize game logic
        Console.WriteLine($"Game {gameId} started.");
    }

    public void EndGame(string gameId)
    {
        // Handle game ending logic
        Console.WriteLine($"Game {gameId} ended.");
    }

    public void UpdateGameState(string gameId, string playerAction)
    {
        // Update game state based on player action
        Console.WriteLine($"Game {gameId} updated with player action: {playerAction}");
    }
}
```

**b. Player Management :**  
Managing player profiles, authentication, session tracking, and in-game states.

```csharp
public interface IPlayerService
{
    void PlayerLogin(string playerId);
    void PlayerLogout(string playerId);
    Player GetPlayerProfile(string playerId);
}

public class PlayerService : IPlayerService
{
    private readonly IPlayerRepository _playerRepository;

    public PlayerService(IPlayerRepository playerRepository)
    {
        _playerRepository = playerRepository;
    }

    public void PlayerLogin(string playerId)
    {
        // Handle player login logic
        Console.WriteLine($"{playerId} logged in.");
    }

    public void PlayerLogout(string playerId)
    {
        // Handle player logout logic
        Console.WriteLine($"{playerId} logged out.");
    }

    public Player GetPlayerProfile(string playerId)
    {
        // Fetch player profile from repository
        return _playerRepository.GetPlayer(playerId);
    }
}
```

**c. Matchmaking Services**
Logic for pairing players based on skill levels, waiting time, or other criteria.

**d. Data Repositories**
Managing game data storage, such as player stats, inventory, match history, etc.
Can be backed by a database (e.g., SQL Server) or an in-memory store.

```csharp
public interface IPlayerRepository
{
    Player GetPlayer(string playerId);
    void SavePlayer(Player player);
}

public class PlayerRepository : IPlayerRepository
{
    // Assume we are using Entity Framework for SQL Server interaction
    private readonly GameDbContext _dbContext;

    public PlayerRepository(GameDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public Player GetPlayer(string playerId)
    {
        return _dbContext.Players.FirstOrDefault(p => p.PlayerId == playerId);
    }

    public void SavePlayer(Player player)
    {
        _dbContext.Players.Add(player);
        _dbContext.SaveChanges();
    }
}
```

**e. Real-time Communication Services**
Handling WebSocket connections or gRPC for real-time player interactions.

By using Dependency Injection in your .NET Core game server, you can manage your components and services more efficiently, reduce code duplication, improve testability, and handle complex game logic in a modular and maintainable way. Whether it’s player management, game state updates, or matchmaking, DI provides flexibility and robustness to your backend game server architecture.

## 9). AddScoped, AddTransient and AddSingleton

<p align="center">
  <img src="https://github.com/user-attachments/assets/9d3bd981-1c65-4eba-a4a9-e947a889a50a" width="400" height="250" />
</p>

**Service Lifetimes in .NET Core:**
Service lifetimes define how long instances of a service should be kept and reused. The framework provides three main service lifetimes: Singleton, Transient, and Scoped. Choosing the appropriate service lifetime is crucial for managing resources efficiently and ensuring the desired behavior of your application.

**A). AddSingleton:**  
The AddSingleton method is used to register a service as a singleton. A singleton instance is created once and reused throughout the lifetime of the application. This is beneficial when you want a single instance of a service to be shared across the entire application.

```csharp
services.AddSingleton<IMyService, MyService>();
```

**Real-world Example:** Consider a scenario where you have a configuration service that loads application settings from a file. You’d want to ensure that the configuration is loaded only once and shared across all components.

```csharp
services.AddSingleton<IConfigurationService, FileConfigurationService>();  
```

**B). AddTransient:**
The AddTransient method is used to register a service as transient. A new instance of the service is created every time it's requested. Transient services are suitable for lightweight and stateless operations.

```csharp
services.AddTransient<IMyService, MyService>();
```

**Real-world Example:** Imagine you have a logging service responsible for writing logs to a file. In this case, you may want a new instance of the logging service for every log entry to maintain isolation and avoid interference between log entries.

```csharp
services.AddTransient<ILoggingService, FileLoggingService>();
```

**C). AddScoped:**
The AddScoped method is used to register a service as scoped. A scoped instance is created once per request within the scope of an HTTP request. It means that the same instance is shared across different components within the same HTTP request.

```csharp
services.AddScoped<IMyService, MyService>();
```

**Real-world Example:** Consider a scenario where you have a user service responsible for handling user-related operations. Using AddScoped ensures that the same instance of the user service is used throughout the entire HTTP request, allowing you to maintain a consistent state for the current user.

```csharp
services.AddScoped<IUserService, UserIdentityService>();
```

- [Dependency Injection and using AddTransient, AddScoped and AddSingleton in an ASP.NET Core application](https://alexb72.medium.com/dependency-injection-and-using-addtransient-addscoped-and-addsingleton-in-an-asp-net-2ae09e45c983)

- [Navigating Dependency Lifetimes: A Practical Comparison of AddTransient, AddScoped, and AddSingleton in .NET](https://nshyamprasad.medium.com/navigating-dependency-lifetimes-a-practical-comparison-of-addtransient-addscoped-and-8b825a465dc5)

**Advanced Use of DI: Scoped, Transient, and Singleton Lifetimes**

**Singleton services**, such as cache or configuration services, can store game state across sessions.  
**Scoped services** are useful for maintaining game-related state during a single request, such as player session data.  
**Transient services** are ideal for lightweight operations that don’t require state preservation between requests.  

## 10). Extension Methods. What are the use of extension methods : Run(), Use() and Next() ?

Extension methods are a way to add new functionality to existing types without modifying their source code. They are static methods that can be called as if they are instance methods on the extended type. Extension methods are a powerful feature in C# that can help you write cleaner, more expressive code.  

**Key Characteristics:**
- Static Class: Extension methods must be defined in a static class.  
- Static Method: The method itself must be static.  
- First Parameter with this Keyword: The first parameter specifies the type being extended, and it must be preceded by the this keyword.  

**Benefits of Extension Methods:**
- Improved Readability and Maintainability: They help in writing fluent APIs and make code more readable.  
- Encapsulation of Logic: You can encapsulate frequently used logic, which simplifies code maintenance.  
- Flexibility: They allow you to "add" functionality to existing types, including those in third-party libraries or .NET's core libraries, without inheritance.  

**Common Uses:**  
- LINQ: Most LINQ operations are implemented as extension methods on IEnumerable<T> and IQueryable<T>.  
- Utility Functions: Often used for string manipulations, date/time operations, and other utility functions.  

[Mastering Extension Methods in C#](https://www.linkedin.com/pulse/mastering-extension-methods-c-pradeep-pandit-feawf/)

In one of my .NET Core projects, I created an extension method for validating email addresses. 

```csharp
public static class StringExtensions
{
    public static bool IsValidEmail(this string email)
    {
        return Regex.IsMatch(email, @"^[^@\s]+@[^@\s]+\.[^@\s]+$");
    }
}
```

Coming to the Extension Methods - Run(), Use() and Next()  
[Run, Use, and Next Method in ASP.NET Core](https://dotnettutorials.net/lesson/run-next-use-methods-in-asp-net-core/#:~:text=The%20Run%20method%20in%20ASP,in%20the%20request%20processing%20pipeline.)

🎮 We can use Extension Methods in Unity3D also in Game Development.

[11 Useful Unity C# Extension Methods](https://monoflauta.com/2021/07/27/11-useful-unity-c-extension-methods/)  

**Game server setup for handling player connections or game events**  
In a game server using .NET, app.Run is typically part of the setup in an ASP.NET Core application, where it starts the web host and begins listening for incoming HTTP requests. For a backend game server, especially in Unity or another game framework using .NET Core, app.Run is used to host the game server logic via a web API or WebSocket server.

Here’s an example of how you might use app.Run in a game server setup for handling player connections or game events via HTTP or WebSockets:

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = WebApplication.CreateBuilder(args);

// Add services like WebSockets or other game-related services
builder.Services.AddControllers();
builder.Services.AddWebSocketManager(); // If using WebSockets for real-time interactions

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseRouting();

// If using WebSockets
app.UseWebSockets();

// Endpoints for HTTP-based requests
app.UseEndpoints(endpoints =>
{
    endpoints.MapControllers();  // Map API routes for game functionality
});

// Run the game server
app.Run();
```

**app.UseRouting():** This is used to route incoming requests to the appropriate handlers (e.g., player connections, game events).
**app.UseWebSockets():** If you are handling real-time communication, WebSockets may be necessary for live multiplayer game servers.
**app.UseEndpoints():** Configures endpoints for different routes like /api/game/start for starting a game or /api/game/player/connect for player connections.    

In a multiplayer backend, app.Run effectively starts the game server, enabling it to handle player requests, game events, matchmaking, or real-time communication over WebSockets.

## 11.1). Entity Framework Core

In Entity Framework, the model is prepared according to the requirement of the user. It depends upon the number of classes and categories that will be embedded into the database.

- [Entity Framework Core Model](https://www.learnentityframeworkcore5.com/entity-framework-core-model)
  
- [One-to-Many Relationships using Fluent API in Entity Framework Core](https://www.entityframeworktutorial.net/efcore/configure-one-to-many-relationship-using-fluent-api-in-ef-core.aspx)

- [An Insightful Dive into Entity Framework Core](https://www.linkedin.com/pulse/insightful-dive-entity-framework-core-heart-data-management-adi-inbar/?trackingId=CKg92x8pLiei6Du9gH2AjQ%3D%3D)

- ▶️ [Complete 3 Hour ASP NET 6.0 and Entity Framework Core Course!](https://www.youtube.com/watch?v=7d2UMAIgOLQ&list=PLwhVruPHD9rxZ9U5K6vqUFkfrjaRhwEsV&index=12)

🎮  **Entity Framework (EF) is an Object-Relational Mapper (ORM)** used in .NET applications to interact with databases, simplifying data access by mapping database records to objects in your application. In the context of a game server, using Entity Framework allows you to efficiently manage player data, game statistics, match histories, and other persistent data without manually writing complex SQL queries.

Common Use Cases for Entity Framework in a Game Server
- **Player Profiles:** Storing and retrieving player information such as usernames, levels, stats, inventory, etc.
- **Match History:** Keeping records of past matches, player scores, and performance.
- **In-Game Currency/Inventory:** Managing virtual currencies or in-game assets tied to players.
- **Game States:** Saving the state of ongoing matches or sessions for persistence between server restarts.

## 11.2). DataBase First Approach and CodeFirst Approach

Entity Framework is an open-source object-relational mapping framework for ADO .NET, which is a data access technology, this means that you can use this technology to access data in the database.  

**A). Code First Approach**
The code-first approach is a way to design your application’s data models by creating them as C# classes for your models and then you use them to create your database.

In order to apply the code-first approach, you need to follow these steps:

- Define your model classes, each class corresponds to a table in your database.
- Create a DbContext class that inherits from the DbContext class provided by Entity Framework.
- Enable migrations.
- Create the database, through the package manager console, use the entity framework to create the database and the table. Also, you need to write a command for that which is the Update-Database command.

As a simple example, we will create a Person model class and a DbContext class:

```csharp
public class Person
{
    public int Id { get; set; }
    public string FullName { get; set; }
    public int Age { get; set; }
}
```

```csharp
public class MyDbContext : DbContext
{
    public DbSet<Person> Persons { get; set; }

    public MyDbContext() : base("TheConnectionString")
    {
    }
}
```

**B). Database First Approach**
The database first approach is a way to create the data models starting from an existing database. We generate our data models and the DbContext class based on an existing database schema.
An example of how to query data using the DbContext would be like this:

```csharp
public IEnumerable<Person> GetPersons()
        {
            using (myDBContext con = new MyDBContext())
            {
                con.Persons.Load();
                return con.Persons.Local.ToList();
            }
        }
```
[Exploring Database-First Approach with Entity Framework in .NET Core 6](https://medium.com/@certosinolab/exploring-database-first-approach-with-entity-framework-in-net-core-6-db86a822d72e)

[Code First Approach vs. Database First Approach](https://medium.com/codex/code-first-approach-vs-database-first-approach-a3830c0cc9b6)

🎮 Store persistent player data (e.g., profiles, achievements, leaderboards) using a database like SQL Server. Use Entity Framework Core (Code First) to interact with the database.

## 11.3). DbContext & DbSet

The **DbContext** is simply the way for the developers to incorporate Entity Framework based data to the application. It allows you to make database connections inside an application model and allows the developer to link the model properties to the database table using a connection string.

The DbContext in Entity Framework Core consist of the following features and responsibilities:

- Database Management
- Database Connections
- Entity Set
- Querying
- Validation

In Entity Framework Core, the **DbSet** represents the set of entities. In a database, a group of similar entities is called an Entity Set. The DbSet is responsible for performing all the basic CRUD (Create, Read, Update and Delete) operations on each of the Entity.

- [DbContext](https://www.learnentityframeworkcore5.com/dbcontext)
- [DbSet](https://www.learnentityframeworkcore5.com/dbset)

- [c# Entity framework core assignment solution: Add models and tables](https://www.linkedin.com/pulse/c-entity-framework-core-assignment-solution-add-models-adi-inbar-3r87f/?trackingId=dxMHMLu80K4bLf%2B8FpVmzA%3D%3D)

- [ASP.NET Core REST API DbContext](https://www.pragimtech.com/blog/blazor/asp.net-core-rest-api-dbcontext/)

- [DTO (Data Transfer Object)](https://www.telerik.com/blogs/dotnet-basics-dto-data-transfer-object)

🎮 In a game server, the GameDbContext would contain DbSets for various entities like Player, Game, and Match.

```csharp
public class GameDbContext : DbContext
{
    public DbSet<Player> Players { get; set; }
    public DbSet<Game> Games { get; set; }
    public DbSet<Match> Matches { get; set; }

    public GameDbContext(DbContextOptions<GameDbContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Additional configuration of entities (optional)
    }
}
```

Your entities represent the data you want to store. For example, a Player class might include information about a player, such as username, score, level, etc.

```csharp
public class Player
{
    public int PlayerId { get; set; } // Primary Key
    public string Username { get; set; }
    public int Level { get; set; }
    public int Score { get; set; }
    public List<Match> Matches { get; set; } // One-to-many relationship with Matches
}

public class Game
{
    public int GameId { get; set; } // Primary Key
    public string Title { get; set; }
    public List<Match> Matches { get; set; }
}

public class Match
{
    public int MatchId { get; set; } // Primary Key
    public int PlayerId { get; set; }
    public Player Player { get; set; } // Foreign Key relationship to Player
    public int GameId { get; set; }
    public Game Game { get; set; } // Foreign Key relationship to Game
    public DateTime MatchDate { get; set; }
    public int PlayerScore { get; set; }
}
```
In this example:

A Player can participate in many Matches.
Each Match is associated with both a Player and a Game.

In the Startup.cs file (or Program.cs in .NET 6+), configure Entity Framework to use your chosen database provider (e.g., SQL Server).

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Register the DbContext with SQL Server
        services.AddDbContext<GameDbContext>(options =>
            options.UseSqlServer(Configuration.GetConnectionString("GameDb")));

        // Register other services
        services.AddTransient<IPlayerService, PlayerService>();
        services.AddTransient<IGameService, GameService>();

        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

## 11.4). SQL and NoSQL

🎮 Multiplayer games often have global leaderboards. .NET Core can integrate with databases like Redis for fast leaderboard lookups and use SQL or NoSQL databases to store player performance stats.

```csharp
public class LeaderboardService
{
    private readonly IRedisCache _redis;

    public LeaderboardService(IRedisCache redis)
    {
        _redis = redis;
    }

    public async Task<IEnumerable<Player>> GetTopPlayers()
    {
        // Get top players from Redis cache
        return await _redis.GetTopPlayersAsync();
    }
}
```
**Performance Optimization:    
Asynchronous Programming:** Use async/await patterns to handle concurrent connections and I/O-bound operations, which is crucial in multiplayer games to prevent blocking threads.  
**Caching:** Use Redis for caching frequently accessed data, like player profiles or game states, to reduce database load.

For certain games, you’ll want to store persistent data, such as player stats, inventory, or achievements. This can be done using a SQL or NoSQL database, integrated with Entity Framework Core.

Example:
```csharp
public class PlayerContext : DbContext
{
    public DbSet<Player> Players { get; set; }
}

public class Player
{
    public string PlayerId { get; set; }
    public int Score { get; set; }
    public Vector2 Position { get; set; }
}

```

## 11.5). LINQ

- [Mastering C# LINQ Guide](https://www.bytehide.com/blog/linq-csharp)

- [LINQ Interview Questions and Answers](https://vasistadotnet.wordpress.com/wp-content/uploads/2015/02/linq-interview-questions-answers-by-shailendra-chauhan.pdf)

- [Top 100 LINQ Interview Questions](https://github.com/Devinterview-io/linq-interview-questions) 
 
## 11.6). Migrations, Seeding Data , Nullable and Entity States

- [Migrations in Entity Framework Core](https://www.entityframeworktutorial.net/efcore/entity-framework-core-migration.aspx)

- [Navigating Migrations and Delete Table in Entity Framework Core](https://www.linkedin.com/pulse/navigating-migrations-delete-table-entity-framework-core-adi-inbar/)

- [Harnessing Entity Framework Core: Add Migration in C#](https://www.linkedin.com/pulse/harnessing-entity-framework-core-add-migration-c-adi-inbar/?trackingId=VnxXh8XGAjpfVSfRaFG39A%3D%3D)

- [Reverting and Managing Migrations in Entity Framework Core with C#](https://www.linkedin.com/pulse/reverting-managing-migrations-entity-framework-core-c-adi-inbar/?trackingId=w3J1ieu5fLpAMDfIVaruhA%3D%3D)
 
- [Migration Seeding Data in Entity Framework Core with C#](https://www.linkedin.com/pulse/seeding-data-entity-framework-core-c-adi-inbar-tqwff/?trackingId=Bgt89Mt6yxrDPsRHDmpSOQ%3D%3D)

- [EF Core Seed Data](https://www.learnentityframeworkcore.com/migrations/seeding)
 
- [Understanding Nullable Fields and Renaming Columns in C# with Entity Framework Core](https://www.linkedin.com/pulse/understanding-nullable-fields-renaming-columns-c-entity-adi-inbar/?trackingId=%2BwnxToGDylnTUiTV54QnEA%3D%3D)

- [Entity States in Entity Framework](https://dotnettutorials.net/lesson/entity-state-in-entity-framework/)

## 12). JWT Authentication and Role Based Authorization

One of the key aspects of building web applications is implementing user authentication and authorization. We can implement authentication and authorization using JSON Web Tokens (JWT) in ASP.NET Core, along with a refresh token mechanism to extend the validity of the JWT.
JSON Web Token (JWT) is a widely used standard for representing claims securely between two parties. 
A JWT consists of three parts separated by dots:   
a). Header  
b). Payload  
c). Signature  

[Implementing Authentication and Authorization in ASP.NET Core using JWT Tokens and refresh token with .NET 7](https://medium.com/@kefasogabi/implementing-authentication-and-authorization-in-asp-net-e831c04b4d38)

[OAuth 2.0 and OpenID in simple terms](https://medium.com/@iamprovidence/oauth-2-0-and-openid-in-simple-terms-7196089a1b29)

🎮 In a multiplayer game, it’s important to authenticate users securely. .NET Core provides Identity and OAuth2.0 for managing player authentication and authorization.

**JWT Tokens:** Issue JWT tokens for player authentication. This is particularly useful in games, where the token can be passed in each request to verify the player’s identity without maintaining a session.

```csharp
public class AuthController : ControllerBase
{
    private readonly ITokenService _tokenService;

    public AuthController(ITokenService tokenService)
    {
        _tokenService = tokenService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login(LoginDto loginDto)
    {
        var token = await _tokenService.GenerateJwtToken(loginDto);
        return Ok(new { token });
    }
}
```

Use JWT Tokens for player authentication, especially in multiplayer environments where players need to remain authenticated across multiple sessions.

The server broadcasts the updated game state to all clients at regular intervals, ensuring that each client has the same view of the game world. The clients update their local copies of the game state based on these updates.

Example: Every 100ms, the server sends an update to all clients, informing them of the latest player positions, game events, etc.

```csharp
public async Task SyncGameState()
{
    while (true)
    {
        // Periodically broadcast the authoritative game state to all clients
        await Clients.All.SendAsync("GameStateUpdate", _gameState);

        // Wait for a short interval before sending the next update
        await Task.Delay(100); // Send updates every 100ms
    }
}
```


💻 **Explain how you Implement Authentication and Authorization in a Project:**         
**Authentication Example:** "In one of my projects, I used ASP.NET Core Identity for user authentication. We allowed users to sign in using either their email and password or via Google using OAuth 2.0. For the web API, we implemented JWT tokens to authenticate users, where the token was validated with every API request."    

**Authorization Example:** "For authorization, we implemented a role-based system where different user roles had different access levels. For example, Admin users could manage products, while regular users could only view them. We used the [Authorize] attribute in ASP.NET Core to protect specific API endpoints and ensured the users' roles were checked before performing certain actions."     

## 13). CSRF (Cross Site Request Forgery)

Cross-site request forgery (CSRF) is an attack that tricks a user's browser into sending a malicious HTTP request to another website. This malicious HTTP request looks like it was sent by the user, but it actually comes from the attacker.

[Cross-site request forgery (CSRF)](https://portswigger.net/web-security/csrf)

<p align="center">
  <img src="https://github.com/user-attachments/assets/4e1df98f-61a0-4a14-8d57-8f16e9865fc5" width="500" height="250" />
</p>

[How to secure legacy ASP.NET MVC against Cross-Site (CSRF) Attacks](https://www.red-gate.com/simple-talk/development/web/how-to-secure-legacy-asp-net-mvc-against-csrf-attacks/)

## 14). Kestrel Server

Kestrel is the default web server used in ASP.NET Core applications. It is designed to be fast and efficient, making it an ideal choice for modern web applications. Kestrel can handle HTTP requests and responses, providing a robust foundation for building web applications.

Why Use Kestrel?   
**a). Performance:** Kestrel is highly performant and can handle a large number of requests per second.  
**b). Cross-Platform:** Kestrel runs on Windows, macOS, and Linux, making it versatile for different deployment environments.  
**c). Asynchronous:** Built on top of libuv, Kestrel is designed to handle asynchronous I/O operations efficiently.  
**d). Default Server:** It’s the default web server in ASP.NET Core, which means it’s well-integrated and supported out of the box.  

**Configure Kestrel in Program.cs**  
Open the Program.cs file and configure Kestrel by calling the UseKestrel method on the WebHostBuilder.

```csharp
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
                webBuilder.UseKestrel(options =>
                {
                    options.ListenAnyIP(5000); // Listen on port 5000
                });
            });
}
```

**Configure the Startup Class**  
In the Startup.cs file, configure the services and middleware.

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllers();
        });
    }
}
```

**Create a Sample Controller**  
Create a sample controller to test the Kestrel server. Add a new WeatherForecastController.cs file in the Controllers folder.

```csharp
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;

namespace KestrelDemo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        [HttpGet]
        public IEnumerable<string> Get()
        {
            return new string[] { "Weather 1", "Weather 2" };
        }
    }
}
```

- [Kestrel Server in ASP.NET Core](https://medium.com/@siva.veeravarapu/kestrel-server-in-asp-net-core-cbba6911805b#:~:text=Kestrel%20is%20the%20default%20web%20server%20used%20in%20ASP.NET,IIS%2C%20Nginx%2C%20or%20Apache.)

## 15). State Management - Client and Server

There are two types of State management in ASP net. They are :  
- Server-side  
- Client-side  

These are further subdivided into the following -  
**Server-Side**  
a). Session  
b). Application  
c). Cache  

**Client-Side**  
a). Cookies   
b). Viewstate  
c). Control state  
d). Query String  
e). Hidden Field  

- [What Is State Management: Applications, Types, Example and More](https://www.simplilearn.com/tutorials/asp-dot-net-tutorial/state-management-in-asp-net)

![image](https://github.com/user-attachments/assets/74518738-e094-41d8-8d52-c7695cda37d0)

[What are the different Session state management options available in ASP.NET?](https://medium.com/@iammanolov98/what-are-the-different-session-state-management-options-available-in-asp-net-c08b7ef7dc49)

## 16). Session Management

- **Sessions** in Web Applications refer to the mechanism of storing user-specific data temporarily on the server side across multiple requests. 
- Unlike cookies, which are stored on the client side, session data is stored on the server side, enhancing security and reducing the risk of exposing sensitive information.
- Sessions typically generate a unique identifier (session ID) for each user session upon their first interaction with the application. This identifier (session ID) is stored as a cookie on the client side (usually), and the corresponding data is stored on the Web Server.
- When the client makes subsequent requests, this session ID is sent in the Request header.
- The server uses this identifier to retrieve session-specific data stored in memory (Temporary Caching Mechanism), a database, or another persistent storage mechanism.
- This data persists until the session expires (due to user inactivity or logout) or is manually cleared.

- Session state allows you to store user data on the server that can be accessed across multiple requests from the same client. You can configure session state in the Startup.cs file and then use it in a controller or a view.

[6.2 Using session and TempData for preserving user data](https://medium.com/@syantien/6-2-using-session-and-tempdata-for-preserving-user-data-5a3a6de32b76)

```csharp
// Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddSession(options =>
    {
        options.IdleTimeout = TimeSpan.FromMinutes(30);
    });
}

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseSession();
}

// In a controller
public IActionResult Index()
{
    HttpContext.Session.SetString("Name", "John Doe");
    return View();
}

```

To retrieve the data from the session:

```csharp
public IActionResult AnotherAction()
{
    var name = HttpContext.Session.GetString("Name");
    //...
}

```

## 17). .NET Core and .NET Framework

| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Mobile Development    | The .NET Framework currently does not support their development at all, and that is a problem. | NET Core has some support for mobile apps that is compatible with Xamarin and other open-source platforms for mobile applications.|
| Row 2    | Data 3   | Data 4   |


- [.Net Core vs .Net Framework: Key Differences, Features, and more](https://www.mygreatlearning.com/blog/net-core-vs-net-framework/)

## 18). Filters and it's type

- Action filters are a type of filter in ASP.NET Core that are used to inject custom logic before or after the execution of a controller action method.  
[Understanding Action Filters in ASP.NET Core](https://medium.com/@kefasogabi/understanding-action-filters-in-asp-net-core-a-comprehensive-guide-with-code-samples-ec1f1f2af425)

## 19). What is MVC Architecture? How to create Controllers?

MVC (Model-View-Controller) separates the logic of the application from the display. MVC, with its ‘separation of concerns principle, not only creates a solid framework for web applications but also ensures that different aspects of the application are neatly organized, simplifying future scalability.  

The three parts of MVC are:   
**a). Model:** Defines the structure of the data    
**b). View:** Handles the user interface and data presentation    
**c). Controller:** Updates the model and view based on user input    

<p align="center">
  <img src="https://github.com/user-attachments/assets/91fdf570-51ef-497f-9a11-8e7fd13f4af5" width="500" height="350" />
</p>

[The MVC Architecture](https://medium.com/@sadikarahmantanisha/the-mvc-architecture-97d47e071eb2)

🎮 Create controllers to handle player interactions. For example, you might have a PlayersController and an ActionsController.

**PlayersController.cs**
```csharp
using Microsoft.AspNetCore.Mvc;
using System.Linq;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class PlayersController : ControllerBase
{
    private readonly GameDbContext _context;

    public PlayersController(GameDbContext context)
    {
        _context = context;
    }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetPlayer(string id)
    {
        var player = await _context.Players.FindAsync(id);
        if (player == null)
        {
            return NotFound();
        }
        return Ok(player);
    }

    [HttpPost]
    public async Task<IActionResult> CreatePlayer([FromBody] Player player)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        _context.Players.Add(player);
        await _context.SaveChangesAsync();
        return CreatedAtAction(nameof(GetPlayer), new { id = player.PlayerId }, player);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdatePlayer(string id, [FromBody] Player player)
    {
        if (id != player.PlayerId)
        {
            return BadRequest();
        }

        _context.Entry(player).State = EntityState.Modified;
        await _context.SaveChangesAsync();
        return NoContent();
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeletePlayer(string id)
    {
        var player = await _context.Players.FindAsync(id);
        if (player == null)
        {
            return NotFound();
        }

        _context.Players.Remove(player);
        await _context.SaveChangesAsync();
        return NoContent();
    }
}
```

**ActionsController.cs**
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;

[ApiController]
[Route("api/[controller]")]
public class ActionsController : ControllerBase
{
    private readonly GameDbContext _context;

    public ActionsController(GameDbContext context)
    {
        _context = context;
    }

    [HttpPost]
    public async Task<IActionResult> RecordAction([FromBody] PlayerAction action)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest(ModelState);
        }

        _context.PlayerActions.Add(action);
        await _context.SaveChangesAsync();
        return Ok(action);
    }
}


## 20). Controller Action Methods in MVC

- [Controller Action Return Types in ASP.NET Core Web API](https://dotnettutorials.net/lesson/controller-action-return-types-core-web-api/)

## 21). What are Views? ViewData , ViewBag and TempData

There are two approaches to passing a weakly typed data into the views:

- ViewData
- ViewBag

**1). ViewData**

ViewData maintains data when you move from controller to view. It is also a dictionary object and derived from ViewDataDictionary. As Data is stored as Object in ViewData, while retrieving, the data it needs to be TypeCasted to its original type as the datas stored as objects and it also requires NULL checks while retrieving.

```csharp
ViewData[“Name"] = “CodingSikho";
```

ViewData is a dictionary object and we can get/set values using a key. ViewData exposes an instance of the ViewDataDictionary class.

Let’s create a controller action method and set a value for UserId inside ViewData:

```csharp
public class ViewDataController : Controller
{
    public IActionResult Index()
    {
        ViewData["UserId"] = 101;
        return View();
    }
}
```

Now let’s try to access the userId value inside the View:

```html
@{
    ViewData["Title"] = "Index";
    var userId = ViewData["UserId"]?.ToString();
}
<h1>ViewData</h1>
User Id : @userId
```

Then let’s run the application and navigate to /viewdata:

<p align="center">
  <img src="https://github.com/user-attachments/assets/843ec7b1-e81e-4bd2-a1f7-0cf845e304e8" width="500" height="250" />
</p>

We can see the UserId value is read from ViewData and displayed on the page.

**2). ViewBag** 

The ViewBag is a dynamic type property of ControllerBase class which is the base class of all the controllers. Castings not required when you use ViewBag.  
ViewBag only transfers data from controller to view, not visa-versa. ViewBag values will be null if redirection occurs.  
ViewBag support any number of properties or values. If same value found then it will only consider last value assigned to the property.

```csharp
ViewBag.Name = “Coding Sikho";
```
ViewBag is similar to ViewData but it is a dynamic object and we can add data into it without converting to a strongly typed object. In other words, ViewBag is just a dynamic wrapper around the ViewData.

Let’s add a controller action method to set a few values in ViewBag:

```csharp
public class ViewBagController : Controller
{
    public IActionResult Index()
    {
        ViewBag.UserId = 101;
        ViewBag.Name = "John";
        ViewBag.Age = 31;
        return View();
    }
}
```
Then let’s access it from the View and display the values:

```csharp
{
    ViewData["Title"] = "Index";
    var userId = ViewBag.UserId;
    var name = ViewBag.Name;
    var age = ViewBag.Age;
}
<h1>ViewBag</h1>
User Id : @userId<br />
Name : @name<br /&gt;
Age : @age<br />

```
Now let’s run the application and navigate to /viewbag:

<p align="center">
  <img src="https://github.com/user-attachments/assets/01e5e09b-ae14-418b-b1e4-008e7c2ede5f" width="500" height="200" />
</p>

**3). TempData**

TempData is another way to store temporary data. It is meant to be a short-lived, single-use store for data between requests. It uses session state behind the scenes.

```csharp
public IActionResult Index()
{
    TempData["Message"] = "You successfully registered!";
    return RedirectToAction("Welcome");
}

public IActionResult Welcome()
{
    var message = TempData["Message"];
    // ...
}
```
TempData values are retained for a single request, so they are useful for scenarios such as redirecting between actions (Post-Redirect-Get pattern).

TempData internally uses session variable and stays for a subsequent HTTP Request. This means it maintains data when you move one controller to another controller or oneaction to another action. As this is a dictionary object null checking and typecastingis required while using it.

```csharp
TempData["Name"] = “Coding Sikho";
```

TempData gets destroyed immediately after it's used (once value is read from tempdata) in subsequent HTTP request, so no explicit action required, if you want preserve value in the subsequent request after using need to call **Keep method or Peek method**.

[ASP.NET MVC: Models, ViewData, ViewBag, and TempData Explained](https://www.linkedin.com/pulse/aspnet-mvc-models-viewdata-viewbag-tempdata-explained-ervis-trupja-ytn7f/)

**Keep and Peek**
The keep() and peek() method is used to read the data without deletion of the current read object.
You can use Keep() when prevent/hold the value depends on additional logic.
You can use Peek() when you always want to hold/prevent the value for another request.

```csharp
TempData["Name"] = "Coding Sikho"
TempData.Keep("Name");
TempData.Peek("Name");
```

[State Management in ASP.NET Core MVC](https://code-maze.com/state-management-in-asp-net-core-mvc/)

▶️ [TempData, ViewData, ViewBag in Asp.Net MVC | MVC for beginners](https://www.youtube.com/watch?v=lr7YTjpRF5g)

<p align="center">
  <img src="https://github.com/user-attachments/assets/f7b31ffa-1a68-4a79-84bf-06f3ab51624f" width="500" height="250" />
</p>

## 22). Repository Pattern

- [Repository Pattern Implementation in ASP.NET Core](https://medium.com/net-core/repository-pattern-implementation-in-asp-net-core-21e01c6664d7)

- [Implement Repository Base and Unit of Work in C#](https://dev.to/1001binary/implement-repository-base-and-unit-of-work-in-c-2ncg)

- [Repository Pattern C# ultimate guide: Entity Framework Core, Clean Architecture, DTOs, Dependency Injection, CQRS](https://medium.com/@codebob75/repository-pattern-c-ultimate-guide-entity-framework-core-clean-architecture-dtos-dependency-6a8d8b444dcb)

- [Implementing ASP.NET CRUD APIs with Repository Pattern and Unit of Work](https://medium.com/@tahatasleem01/implementing-asp-net-crud-apis-with-repository-pattern-and-unit-of-work-b34a86d91baf)

- [A Comprehensive Guide to Repository Pattern in .NET: Implementation and Best Practices](https://medium.com/@dhananjay_1891/a-comprehensive-guide-to-repository-pattern-in-net-implementation-and-best-practices-d67c3a92e618)

## 23). Singleton Design Pattern

- 

## 24). REST Api. How to create REST API?

🎮 You can implement a matchmaking service using REST APIs or SignalR to find and assign players to games based on their skills, region, or preferences.

- [How To Build a RESTful API with ASP.NET Core](https://medium.com/net-core/how-to-build-a-restful-api-with-asp-net-core-fb7dd8d3e5e3)

- [Create rest API in .Net Core](https://medium.com/@sagarkumar2499/create-rest-api-in-net-core-b2aed00416fd)

## 25). What are HTTP Verbs? 
 

## 26). What is Data Annotations(Validations)? Client Side and Server Side Validations.


## 27). What is Pagination?  


## 28). Improve (Optimize) performance of Dot Net Application.

<p align="center">
  <img src="https://github.com/user-attachments/assets/7803c0d0-dcc5-42d1-90f9-a11907e915ea" width="500" height="250" />
</p>

**1. Caching Data :** Caching is a technique used in software development to store data that is computationally expensive or frequently accessed temporarily. The server is called, and the reply obtained is saved. As a result, the next time a request is made for a similar response, instead of going to the server, the data is verified against the cached data, and if they match, the stored data is obtained.

**2. Memory optimizations :** In .NET application development, memory optimization is an essential factor to consider. Object pooling, reducing the object’s size, and preventing memory leaks are all ways that developers can increase the scalability of their applications and lessen memory-related problems.

**3. Disable View State :** Disabling View State is another tried-and-true method a seasoned .NET development company uses to enhance .NET application performance. The View State method is a state management technique used by .NET to maintain page and control data over round trips.

**4. Asynchronous programming :** Asynchronous programming enables you to run numerous processes simultaneously without stopping the main thread, which helps dedicated .NET developers to create responsive and scalable apps.

**5. Optimize database access :** Here are a few techniques that could be utilized to create code that would significantly improve performance.

- Try retrieving the data in one or two calls to the server rather than several.
- If it is not necessary, do not retrieve the data in advance. The application slows down due to the increased demand for the response.
- Use Entity Framework Core (EF Core) or other Object-Relational Mapping (ORM) technologies to make your database searches more efficient.
- Avoid making repeated database calls by using caching to save frequently accessed data.

**6. Reduce Network Round-Trip Time :** One of the most effective ways to boost the overall speed of your.NET application is to minimize the number of networks round-trips by compressing data, employing HTTP compression, and lowering the size of payloads. 

**7. Use JSON Serialization :** `System.Text.Json` is used by leading .NET development companies for JSON serialization to and deserialization from options. As a result, JSON can be read and written asynchronously without the need to wait for other processes to finish. Use JSON instead of NEwtonsoft because it enhances performance.

[Optimizing Performance in .NET Web Applications: Tips and Techniques](https://eluminoustechnologies.com/blog/optimizing-net-application-development/)

**How to Maximize Performance and Scalability of Your App?**  
https://medium.com/agoda-engineering/asp-net-core-performance-optimization-how-to-maximize-performance-and-scalability-of-your-app-d676668aebea

## 29). Memory Leaks

**What is Memory Leak?**
A memory leak in C# occurs when a program allocates memory by creating objects but fails to release them after they are no longer needed.  
This leads to a gradual increase in the memory consumption of the application, potentially causing it to slow down or crash if the system runs out of memory. Memory leaks are often subtle and can be difficult todetect and debug.

[What are memory leaks in C#](https://www.youtube.com/watch?v=zSGMGFSK6Ho)

<p align="center">
  <img src="https://github.com/user-attachments/assets/154ce1fa-052f-406e-a338-f4eb53b691f1" width="500" height="250" />
</p>

[8 Ways You can Cause Memory Leaks in .NET](https://michaelscodingspot.com/ways-to-cause-memory-leaks-in-dotnet/)

**Memory leaks can occur in the following scenarios:**

**1.Unmanaged Resources:** When using unmanaged resources like file handles,
network connections, or database connections, failing to properly release them
can lead to memory leaks.

**2.Event Handlers:** Not unregistering event handlers can prevent the garbage
collector from deallocating the memory used by the subscriber objects.

**3.Static Members:** Static fields and properties, if improperly managed, can hold
references to objects, preventing those objects from being garbage collected.

**4.Large Object Heap (LOH) Fragmentation:** Large objects are managed in a separate
heap in .NET. Improper management can lead to memory fragmentation, which
can indirectly cause memory leaks.

**5.References in Collections:** Holding references to objects in collections (like lists,
dictionaries, etc.) for longer than necessary can prevent the garbage collector
from reclaiming that memory.

**To prevent memory leaks in C#, you can follow these best practices:**

**1. Properly Dispose Unmanaged Resources:** Implement the IDisposable interface for
classes that use unmanaged resources and ensure that the Dispose method is
called appropriately, either explicitly or using a using statement.

**2. Detach Event Handlers:** Always detach event handlers when they are no longer
needed, especially in the case of long-lived publishers.

**3. Be Cautious with Static Members:** Avoid unnecessary static fields, or ensure that
they do not hold references to instances that should be garbage collected.

**4. Monitor and Optimize LOH Usage:** Be aware of memory allocation in the Large
Object Heap and optimize it to avoid fragmentation.

**5. Manage Collection Lifetimes:** Regularly review and clean up collections, removing
references to objects that are no longer needed.

**6. Use Weak References:** Where applicable, use weak references (WeakReference
class) for objects that do not need to be kept alive by the garbage collector.

**7. Profiling and Analysis Tools:** Utilize memory profiling and analysis tools to identify
potential leaks, especially in complex applications.

## 30). Unit Testing - NUnit and XUnit

- [Controller Action Return Types in ASP.NET Core Web API](https://dotnettutorials.net/lesson/controller-action-return-types-core-web-api/)

## Game Server using Dot Net Core




## Minor Projects

#### 1). CRUD Operations

- [ASP.NET Core Web Application](https://www.youtube.com/watch?v=T-e554Zt3n4)

- [CRUD Operations using ASP.NET Core MVC](https://www.youtube.com/watch?v=SfWuOFEatYc)

#### 2). Custom Identity

- [Custom Identity in Asp.Net Core MVC](https://www.youtube.com/watch?v=93ssXlCPcuI&t=2158s)

## 0). Advanced C# Concepts

- ▶️ [Advanced C# Topics](https://www.youtube.com/watch?v=VT9ueWBqquU&list=PLwhVruPHD9ryiH4kN0EHYeXQXIOHLBcJX&index=1)

## ASP.NET MVC

- ▶️ [Learn ASP.NET MVC (.NET 6)](https://www.youtube.com/watch?v=H14S7x8q_vQ&list=PLqVWQ84m1Q7EiKKyOpiWSVcpG3qUmJ0Xc&index=1)

- ▶️ [Full Course - Learn ASP.NET Core MVC in .NET 8 | CRUD Operations](https://www.youtube.com/watch?v=BzlPrVB_DwA)

## Best Channels PlayLists

- ▶️ [Code Unparalleled .NET Playlist](https://www.youtube.com/@CodeUnparalleled/playlists)

