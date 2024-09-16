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

Configure CORS startup class inside the ConfigureService method.  

```csharp
public void ConfigureServices(IServiceCollection services)  
services.AddCors(options =>  
{  
   options.AddPolicy("Policy11",  
   builder => builder.WithOrigins("http://hello.com"));  
});  
```

Enable CORS using middleware in the Configure method.

```csharp
public void Configure(IApplicationBuilder app)  
{  
   app.UseCors("AllowMyOrigin");  
} 
```

[CORS In .NET Core](https://www.c-sharpcorner.com/article/cors-in-dotnet-core/)

**Implementation:**

To enable CORS there are  three ways to do so:  
- **Middleware using a named policy or default policy.**
  This following code enables the default CORS policy :
  ```csharp
  public class Startup {  
    public void ConfigureServices(IServiceCollection services) {  
        services.AddCors(options => {  
            options.AddDefaultPolicy(builder => {  
                builder.WithOrigins("http://hello.com", "http://www.test.com");  
            });  
        });  
        services.AddControllers();  
    }  
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env) {  
        if (env.IsDevelopment()) {  
            app.UseDeveloperExceptionPage();  
        }  
        app.UseRouting();  
        app.UseCors();  
        app.UseEndpoints(endpoints => {  
            endpoints.MapControllers();  
        });  
    }  
}```

- **Using Endpoint Routing**

Using endpoint routing, CORS can be apply per-endpoint basis using the RequireCors.
```csharp
public class Startup {  
    readonly string allowCors = "_myOrigins";  
    public void ConfigureServices(IServiceCollection services) {  
        services.AddCors(options => {  
            options.AddPolicy(name: allowCors, builder => {  
                builder.WithOrigins("http://hello.com");  
            });  
        });  
    }  
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env) {  
        if (env.IsDevelopment()) {  
            app.UseDeveloperExceptionPage();  
        }  
        app.UseRouting();  
        app.UseCors();  
        app.UseAuthorization();  
        app.UseEndpoints(endpoints => {  
            endpoints.MapGet("/foo", context => context.Response.WriteAsync("foo")).RequireCors(MyAllowSpecificOrigins);  
        });  
    }  
}
```

- **Using [EnableCors] attribute.**
  
This [EnableCors] attribute allow CORS for selected endpoints, so it will not impact the all endpoints,
 
This attribute will be applied on the following places:
**1). Global :** You can enable CORS globally for all controllers by adding the CorsAuthorizationFilterFactory filter in the ConfigureServices method,
```csharp
public void ConfigureServices(IServiceCollection services) {  
    services.AddMvc();  
    services.Configure < MvcOptions > (options => {  
        options.Filters.Add(new CorsAuthorizationFilterFactory("Policy1"));  
    });  
}
```

**2). Controller :** To apply the CORS policy for a particular controller we need to add the [EnableCors] attribute at controller level.  
```csharp
[EnableCors("Policy1")]  
public class HomeController : Controller  
{  
}  
```

**3). Action Method :**  
```csharp
public class TestController: ControllerBase {  
        [EnableCors("Policy2")]  
        [HttpGet]  
        public ActionResult < IEnumerable < string >> Get() {  
                return new string[] {  
                    "apple",  
                    "mango"  
                };  
            }  
            [EnableCors("Policy1")]  
            [HttpGet("{id}")]  
        public ActionResult < string > Get(int id) {  
            return "test"  
        } 
```

Sometimes we need to add multiple CORS.  
The following code will create two CORS policies:  

```csharp
public void ConfigureServices(IServiceCollection services) {  
        services.AddCors(options => {  
            options.AddPolicy("Policy1", builder => {  
                builder.WithOrigins("http://hello.com");  
            });  
            options.AddPolicy("Policy2", builder => {  
                builder.WithOrigins("http://www.test.com").AllowAnyHeader().AllowAnyMethod();  
            });  
        });  
```

[Cross-Origin Resource Sharing in .NET](https://medium.com/@darshana-edirisinghe/cross-origin-resource-sharing-in-net-f8d0aa802b5f)

**Practical Example of CORS ASP.NET Core**  
CORS and ASP.NET Core go together like bread and butter, cookies and milk, or… well, you get the idea. To make this partnership crystal clear, let’s look at a practical example where we configure an ASP.NET Core application to allow requests from specific origins.  

```csharp
/*
In Startup.cs configure services method, we add a CORS policy
*/
public void ConfigureServices(IServiceCollection services)
{
    services.AddCors(options =>
    {
        options.AddPolicy("AllowSpecificOrigin",
            builder =>
            {
                builder.WithOrigins("http://example.com", "http://example2.com")
                    .AllowAnyHeader()
                    .AllowAnyMethod();
            });
    });
}
```

[Enable CORS in ASP.NET Core in the Easiest Way](https://www.bytehide.com/blog/cors-aspnet-core)

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

**What is Routing?**  
- Routing is a pattern matching system.  
- Routing maps incoming request (from browser) to a particular resource (controller & action method).
                 domain.com/Home/About
                 domain.com/about-us
- MVC routing can be defined as a pattern-matching scheme that is used for mapping incoming requests of browsers to a definite MVC controller action.

**The MVC routing has 3 parameters :**
   - The first parameter determines the name of the route (ControllerName). 
   - The second parameter determines a specific pattern with which the URL matches 
     (ActionMethodName). 
   - The third parameter is responsible for providing default values for its placeholders 
     (Parameter).

**How Routing works?**
- We define a route for each action method. 
- All the routes are stored in route table
- Each incoming request is mapped to this route table.
- If a URL match is found then the request goes to the related controller action method.
- If the URL is not found then the application returns 404 pages.

![image](https://github.com/user-attachments/assets/917c400e-940c-4782-8f81-fb47f6a7b6f9)

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

- OWIN defines a standard interface between .NET web servers and web applications.
- The goal of the OWIN interface is to decouple server and application.
- Prior to this standard in .NET, there was a tight coupling between .NET applications and Internet Information Server (IIS), which led to great difficulties when trying to expand to different web application/server technologies.
- The introduction of OWIN has created an abstraction between application and server that completely decouples one from the other.
- OWIN extends its support as: 
a). It supports Authentication functionality related with Cookies.  
b). It also supports Expiry states.  
c). It supports Expiry state of session etc.  
d). It supports the security protections using such secure tokens.

- The contract defined by OWIN for application/server communication can be found within the IAppBuilder interface and at its core is boiled down into two pieces: the environment dictionary and a function to register middleware.  
The environment dictionary outlines the state of the request/response and simplifies it down to a mapping of string keys to objects within the Properties property. 

- With .Net 4.7 I used to log my http request and response conditionally through the help of OWIN. 

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

**e. Real-time Communication Services:**    
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

[Dependency Injection and using AddTransient, AddScoped and AddSingleton in an ASP.NET Core application](https://alexb72.medium.com/dependency-injection-and-using-addtransient-addscoped-and-addsingleton-in-an-asp-net-2ae09e45c983)

[Navigating Dependency Lifetimes: A Practical Comparison of AddTransient, AddScoped, and AddSingleton in .NET](https://nshyamprasad.medium.com/navigating-dependency-lifetimes-a-practical-comparison-of-addtransient-addscoped-and-8b825a465dc5)

**Advanced Use of DI: Scoped, Transient, and Singleton Lifetimes**

**Singleton services**, such as cache or configuration services, can store game state across sessions.  
**Scoped services** are useful for maintaining game-related state during a single request, such as player session data.  
**Transient services** are ideal for lightweight operations that don’t require state preservation between requests.  

## 10). Extension Methods. What are the use of extension method : Run(), Use() and Next()?

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
Entity Framework is a tool we use to access the database. EF is an Object Relational Mapper (ORM) we use to map the objects of our applications with the Relational Data.  

![image](https://github.com/user-attachments/assets/1ecc98c4-a8ce-434a-a2f8-6492385fcda6)

Earlier, we had to do all these mapping manually which involved lots of steps. We work at a higher level of Abstraction in Entity Framework.  

Entity Framework provides a class - DbContext which is the Gateway to our database. A DbContext can have one or more DbSets. These DBSets represent tables in our database. 

![image](https://github.com/user-attachments/assets/bbcb8b46-0b4c-4acc-9be2-173d5f149452)
           
We use LINQ Query to query these DbSets. Entity Framework converts these LINQ Queries to SQL Queries at runtime. We don’t have to do these translations manually. All of these are done by EF behind the scenes.

![image](https://github.com/user-attachments/assets/9fb5f640-deae-41cf-9b83-82ed7d01b846)

So, Entity Framework is responsible for opening a connection to the database, reads the data and maps it to the objects and adds them to the DbSet and returns the DBContext back to us. 

![image](https://github.com/user-attachments/assets/67141e8f-a167-455e-94b2-bca15f735b54)

As we add, modify or remove objects in these DbSets, EF keeps track of these changes and when we ask to persist these changes, again it will automatically generate the SQL Statements and execute them on our Database. 

There are 2 different ways/workflow for the EF - 
  a). Database First Approach 
  b). CodeFirst Approach 

[Entity Framework Core Model](https://www.learnentityframeworkcore5.com/entity-framework-core-model)
  
[One-to-Many Relationships using Fluent API in Entity Framework Core](https://www.entityframeworktutorial.net/efcore/configure-one-to-many-relationship-using-fluent-api-in-ef-core.aspx)

[An Insightful Dive into Entity Framework Core](https://www.linkedin.com/pulse/insightful-dive-entity-framework-core-heart-data-management-adi-inbar/?trackingId=CKg92x8pLiei6Du9gH2AjQ%3D%3D)

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

## 11.3). DbContext, DbSet & DTOs

The **DbContext** is simply the way for the developers to incorporate Entity Framework based data to the application. It allows you to make database connections inside an application model and allows the developer to link the model properties to the database table using a connection string.

[DbContext](https://www.learnentityframeworkcore5.com/dbcontext)

[ASP.NET Core REST API DbContext](https://www.pragimtech.com/blog/blazor/asp.net-core-rest-api-dbcontext/)

The DbContext in Entity Framework Core consist of the following features and responsibilities:

- Database Management
- Database Connections
- Entity Set
- Querying
- Validation

In Entity Framework Core, the **DbSet** represents the set of entities. In a database, a group of similar entities is called an Entity Set. The DbSet is responsible for performing all the basic CRUD (Create, Read, Update and Delete) operations on each of the Entity.

[DbSet](https://www.learnentityframeworkcore5.com/dbset)

**What Is a DTO?**  
A DTO (Data Transfer Object) is an object that defines how data will be sent between applications.  
It’s used only to send and receive data and does not contain in itself any business logic.  

**Why Use DTOs?**
The use of DTOs is very common in web development with ASP.NET Core as they provide solutions for many needs. Below are some of them:  
- Separate the service layer from the database layer  
- Hide specific properties that clients don’t need to receive  
- Omit properties to reduce the payload size  
- Manipulate nested objects to make them more convenient for clients  

[DTO (Data Transfer Object)](https://www.telerik.com/blogs/dotnet-basics-dto-data-transfer-object)

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

### A). LINQ Tutorial

- **LINQ Query Syntax in C#**

```csharp
int[] Num = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
IEnumerable<int> result = from numbers in Num  
                                where numbers >3  
                                select numbers;  
```

- **Lambda Expresssion**

// List to store the countries type of string  
```csharp
List<string> countries = new List<string>();

countries.Add("India");
countries.Add("US");
countries.Add("Australia");
countries.Add("Russia");

//use lambda expression to show the list of the countries  
IEnumerable<string> result = countries.Select(x => x);

//foreach loop to display the countries  
foreach (var item in result)    
{
    Console.WriteLine(item);
}
```

### B). Aggregate Function

- **LINQ Min () Function Syntax in C#**

```csharp
int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
int minimumNum = a.Min();  
```

- **LINQ Max () Function Syntax in C#**

```csharp
int[] a = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
int MaximumNum = a.Max();  
```

- **LINQ Sum () Function Syntax in C#**

```csharp
int[] Num = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };  
int Sum = Num.Sum();  
```


### C). LINQ Sorting Operators

Sorting Operators available in LINQ are:

1. ORDER BY  
2. ORDER BY DESCENDING  
3. THEN BY  
4. THEN BY DESCENDING  
5. REVERSE  

- **Syntax of LINQ OrderBy operator**

```csharp
var studentname = Objstudent.OrderBy(x => x.Name);
```

**LINQ OrderBy Operator Example**
```csharp
using System;  
using System.Collections;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Threading.Tasks;  
  
namespace ConsoleApp1  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            List<Student> Objstudent = new List<Student>(){  
        new Student() { Name = "Suresh Dasari", Gender = "Male", Subjects = new List<string> { "Mathematics", "Physics" } },  
        new Student() { Name = "Rohini Alavala", Gender = "Female", Subjects = new List<string> { "Entomology", "Botany" } },  
        new Student() { Name = "Praveen Kumar", Gender = "Male", Subjects = new List<string> { "Computers", "Operating System", "Java" } },  
        new Student() { Name = "Sateesh Chandra", Gender = "Male", Subjects = new List<string> { "English", "Social Studies", "Chemistry" } },  
        new Student() { Name = "Madhav Sai", Gender = "Male", Subjects = new List<string> { "Accounting", "Charted" } }  
        };  
            var studentname = Objstudent.OrderBy(x => x.Name);
 
            foreach (var student in student name)  
            {  
                Console.WriteLine(student.Name);  
            }  
                Console.ReadLine();  
    }  
}  
    class Student  
    {  
        public string Name { get; set; }  
        public string Gender { get; set; }  
        public List<string> Subjects { get; set; }  
    }  
}  
```

- **Syntax of LINQ OrderByDescending operator**

```csharp
var studentname = Objstudent.OrderByDescending(x => x.Name);  
```

**LINQ OrderByDescending Operator Example**

```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace ConsoleApp1
{
    class Program
    {
        static void Main(string[] args)
        {
            //create object of Student class and create a list of the student information  
            List<Student> Objstudent = new List<Student>()
            {
                new Student() { Name = "Akshay", Gender = "Male", Subjects = new List<string> { "Mathematics", "Physics" } },
                new Student() { Name = "Vaishali", Gender = "Female", Subjects = new List<string> { "Computer", "Botany" } },
                new Student() { Name = "Arpita", Gender = "FMale", Subjects = new List<string> { "Economics", "Operating System", "Java" } },
                new Student() { Name = "Shubham", Gender = "Male", Subjects = new List<string> { "Account", "Social Studies", "Chemistry" } },
                new Student() { Name = "Himanshu", Gender = "Male", Subjects = new List<string>{ "English", "Charted" } }
    };
            /*OrderByDescending() operator is used to print the name of the student in the descending form*/
            var studentname = Objstudent.OrderByDescending(x => x.Name);

            //foreach loop is used to print the name of the student  
            foreach (var student in studentname)
            {
                Console.WriteLine(student.Name);
            }
            Console.ReadLine();
        }
    }
    //create a class student  
    class Student
    {
        public string Name { get; set; }
        public string Gender { get; set; }
        public List<string> Subjects { get; set; }
    } 
}
```


[Mastering C# LINQ Guide](https://www.bytehide.com/blog/linq-csharp)

[LINQ Interview Questions and Answers](https://vasistadotnet.wordpress.com/wp-content/uploads/2015/02/linq-interview-questions-answers-by-shailendra-chauhan.pdf)

[Top 100 LINQ Interview Questions](https://github.com/Devinterview-io/linq-interview-questions) 
 
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

**Here is a basic overview of how JWT-based authentication works in a web application:**  
**1. User Authentication:** When a user logs in, the server validates the provided credentials (username and password). If the credentials are valid, the server generates a JWT token containing claims such as user ID, roles, expiration time, etc.  
**2. JWT Structure:** A JWT token is a compact, URL-safe string composed of three parts: header, payload, and signature.  
**3. Token Storage:** The client typically stores the JWT token in a secure manner, such as in an HTTP-only cookie or in local storage.   
**4. Server Validation:** The server validates the token upon receiving a request by checking the signature and verifying the claims.   
**5. Token Refresh:** To avoid frequent logins, a refresh token mechanism may be implemented. The client can request a new JWT token using a refresh token without re-entering credentials.   
**6. Token Expiration:** JWT tokens have an expiration time, which helps mitigate the risk of token misuse.  

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

**a). Authentication Example:** "In one of my projects, I used ASP.NET Core Identity for user authentication. We allowed users to sign in using either their email and password or via Google using OAuth 2.0. For the web API, we implemented JWT tokens to authenticate users, where the token was validated with every API request."    

**b). Authorization Example:** "For authorization, we implemented a role-based system where different user roles had different access levels. For example, Admin users could manage products, while regular users could only view them. We used the [Authorize] attribute in ASP.NET Core to protect specific API endpoints and ensured the users' roles were checked before performing certain actions."     

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

The Kestrel web server is the latest web server as part of ASP.NET Core. Kestrel is open-source , asynchronous I/O based server .It is used to host on any platform ASP.NET applications. It’s a command-line interface and listening server.  

![image](https://github.com/user-attachments/assets/642dbf97-2d29-4b7e-9240-6eb9382692a2)

The internal web server is called Kestrel and the external web server can be IIS, Apache or Nginx .Security purpose kestrel web server supports Secure Sockets Layer.  

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

State Management is a programming technique for User Interface in which the state of a single UI control completely or partially depends on the state of all the other UI controls.  

There are two types of State management in ASP net. They are :  
- Server-side  
- Client-side  

These are further subdivided into the following -  
**Server-Side**  
**a). Session :** It is used to store identity and information; information is stored in the server using Sessionid.  
**b). Application :** This is mainly used to store user activity in server memory and application events.  
**c). Cache :** This is stored on the server-side, and it is used to implement page caching and data caching.  

**Client-Side**  
**a). Cookies :** Cookies are small text files that are stored on the user's computer, and they can be used to store data such as user preferences, session information, and so on.  
**b). Viewstate :** It is used to manage page-level state and is used for storing, sending, and receiving information.  
**c). Control state :** We use Control State to use the view state without the possibility of it being disabled by the user.  
**d). Query String :** The query string is used to store the value in the URL.  
**e). Hidden Field :** Hidden fields are HTML input elements that are used to store data that is not meant to be seen or edited by the user.  

[What Is State Management: Applications, Types, Example and More](https://www.simplilearn.com/tutorials/asp-dot-net-tutorial/state-management-in-asp-net)

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


[.Net Core vs .Net Framework: Key Differences, Features, and more](https://www.mygreatlearning.com/blog/net-core-vs-net-framework/)

Use .NET Core for Server Applications When:
- When a project needs to be integrated across multiple platforms.
- The project requires the development of microservices
- The project necessitates extensive utilization of the Command Line Interface (CLI).

Prefer .NET Framework for Server Applications when:
- Applications are already run on .NET Framework
- Applications that require technologies such as workflow, webforms, or WCF may not be compatible with .NET Core

## 18). Filters and it's type

- Web API includes filters to add extra logic before or after the action method executes. Filters can be used to provide cross-cutting features such as logging, exception handling, performance measurement, authentication and authorization.
- Filters are actually attributes that can be applied on the Web API controller or one or more action methods.

![image](https://github.com/user-attachments/assets/adb92003-1d8b-4832-ac7c-d0ad4e8fb09d)

- Sometimes we want to execute some logic either before the execution of the action method or after the execution. We can use Action Filter for such a scenario.
- Filters define the logic which is executed before or after the execution of the action method.
- Action Filters are attributes which we can apply to the action methods.

  Following are the MVC action filter types:  
  **a). Authorization filter (implements IAuthorizationFilter) :** Authorization filters are used to implement authentication and authorization for controller actions. For example, the 
Authorize filter is an example of an Authorization filter.  
  **b). Action filter  (implements IActionFilter) :** Action filters contain logic that is executed before and after a controller action executes. You can use an action filter, for instance, to modify the view data that a controller action returns.  
  **c). Result filter  (implements IResultFilter) :** Result filters contain logic that is executed before and after a view result is executed. For example, you might want to modify a view result right before the view is rendered to the browser.  
  **d). Exception filter  (implementsIExceptionFilter attribute) :** Exception filters are the last type of filter to run. You can use an exception filter to handle errors raised by either your controller actions or controller action results. You also can use exception filters to log errors.  

![image](https://github.com/user-attachments/assets/b3776787-ab79-4e20-bde5-b0a5f652f3c9)

[Action Filters in MVC [Types of Filters with Examples]](https://www.upgrad.com/blog/action-filters-in-mvc/)

- Action filters are a type of filter in ASP.NET Core that are used to inject custom logic before or after the execution of a controller action method.  
[Understanding Action Filters in ASP.NET Core](https://medium.com/@kefasogabi/understanding-action-filters-in-asp-net-core-a-comprehensive-guide-with-code-samples-ec1f1f2af425)

## 19). What is MVC Architecture? How to create Controllers?

**MVC (Model-View-Controller)** separates the logic of the application from the display. MVC, with its ‘separation of concerns principle, not only creates a solid framework for web applications but also ensures that different aspects of the application are neatly organized, simplifying future scalability.  

The three parts of MVC are:   
**a). Model:** Defines the structure of the data. The Model is the business layer. Model provides the data.      
**b). View:** Handles the user interface and data presentation. The View is the display layer. View presents something to the user i.e User Interface.    
**c). Controller:** Updates the model and view based on user input. The Controller is input control. Controller coordinates between Model and the View. 
      Whatever data the Model has, the Controller passes that data to the View.  


<p align="center">
  <img src="https://github.com/user-attachments/assets/91fdf570-51ef-497f-9a11-8e7fd13f4af5" width="500" height="350" />
</p>

[The MVC Architecture](https://medium.com/@sadikarahmantanisha/the-mvc-architecture-97d47e071eb2)

- MVC Architecture is used to manage Code.
- The ASP.NET MVC framework is suitable for building complex but lightweight web applications.
- It facilitates rapid and efficient application development.
- The framework has a loosely coupled architecture that allows bigger teams of web designers and developers to work parallely in the development projects.

[What Is ASP.NET MVC and What Are Its Main Features?](https://www.matridtech.net/what-is-asp-net-mvc-and-what-are-its-main-features/)

**What is the role of the Controller in ASP.NET MVC?**

- The controller is the interface between the model and view components. 
- It manages and responds to the input and interaction from a user. 
- Controller renders the suitable view to the client, executes the relevant action method, obtains data from the model and fills the view, gets data from the view and updates the model.

As an example, the Customer controller manages all the interactions and inputs received from the Customer View and updates the database by using the Customer model. The Customer controller is also utilized for viewing the Customer-specific data.

![image](https://github.com/user-attachments/assets/a7c9ee7a-f685-445f-a6c0-01b61fbbcf63)

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

Different types of action results returned by action methods in the MVC controller. In the MVC Controller file you have many action methods. Each action method can return different return types of results lik e contentresult,javascript,json or view.  

[Basic return types of ActionResults in ASP.NET MVC](http://www.usmtechworld.com/actionreturntypes)  

Basic return types of action results in ASP.NET MVC are :-  
- **ViewResult -** If you want to return a view in an action method , you should use View as the return type of that method.  
- **PartialViewResult -** If you want to return partialview in action method,you should use partialviewresult as return type of that method.  
- **Contentresult -** If you want to return your content to the view then you should use Content as the return type of the action method.  
- **Emptyresult -** This emptyresult returns nothing in the view page.  
- **Fileresult -** If you want to return a file to the view then you should use File as the return type of the action method.  
- **Json result -** If you want to return JSON data to the view then you should use JSON as the return type of the action method.  
- **Javascript result -** If you want to return javascript to the view then you should use JavaScript as the return type of the action method.  

![image](https://github.com/user-attachments/assets/3f833388-c2b4-4882-a3ea-5b2407febd66)

**Content negotiation** is the process of selecting the best resource for a response when multiple resource representations are available. Content negotiation is an HTTP feature. Examples are - IActionResult. [Content Negotiation in Web API](https://code-maze.com/content-negotiation-web-api/)

[Controller Action Return Types in ASP.NET Core Web API](https://dotnettutorials.net/lesson/controller-action-return-types-core-web-api/)

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

[Repository Pattern Implementation in ASP.NET Core](https://medium.com/net-core/repository-pattern-implementation-in-asp-net-core-21e01c6664d7) 
- The repository pattern is a software design pattern that acts as an abstraction layer between your data access layer and the business logic layer in an ASP.NET Core Web API .  
- It hides the details of how exactly the data is saved or retrieved from the underlying data source.   
- The details of how the data is stored and retrieved is in the respective repository.   
- This means your business logic doesn’t care whether it’s talking to SQL Server, Oracle, or even a mock object for testing purposes.  

[How to perform Repository pattern in ASP.NET MVC?](https://www.ifourtechnolab.com/blog/how-to-perform-repository-pattern-in-asp-net-mvc#:~:text=The%20repository%20is%20used%20to,can%20facilitate%20automated%20unit%20testing.)

- The Repository pattern is the most popular pattern for creating an enterprise level application. 
- The repository is used to create an abstraction layer between the data access layer and the business logic layer of an application.Implementation of repository patterns can help to abstract your application from changes in the data store and can facilitate automated unit testing.
- Repository directly communicates with Database (Data Access Layer (DAL)) and fetches the data and provides it to the logical layer (Business logic layer (BAL)). 
- The purpose of the Repository is to isolate the data access layer (DAL) and the Business Logic Layer (BAL). 
- Instead of writing entire data access logic in a controller, write this logic in a different class known as a repository. This will make your code maintainable and understandable.

![image](https://github.com/user-attachments/assets/ac941a3d-7b8f-488f-b8ad-fde8597e33f2)

[Implement Repository Base and Unit of Work in C#](https://dev.to/1001binary/implement-repository-base-and-unit-of-work-in-c-2ncg)

[Repository Pattern C# ultimate guide: Entity Framework Core, Clean Architecture, DTOs, Dependency Injection, CQRS](https://medium.com/@codebob75/repository-pattern-c-ultimate-guide-entity-framework-core-clean-architecture-dtos-dependency-6a8d8b444dcb)

[Implementing ASP.NET CRUD APIs with Repository Pattern and Unit of Work](https://medium.com/@tahatasleem01/implementing-asp-net-crud-apis-with-repository-pattern-and-unit-of-work-b34a86d91baf)

[A Comprehensive Guide to Repository Pattern in .NET: Implementation and Best Practices](https://medium.com/@dhananjay_1891/a-comprehensive-guide-to-repository-pattern-in-net-implementation-and-best-practices-d67c3a92e618)

## 23). Singleton Design Pattern. Singleton VS Static Class.  

[Singleton Pattern in C#](https://dev.to/kalkwst/singleton-pattern-in-c-1dh0)

**Definition :**  
- In software engineering, the singleton pattern is a Creational software design pattern that restricts the instantiation of a class to a single instance(object). The Singleton pattern is used to ensure that a class has only one instance.
- Throughout the lifetime of the application the instance will remain same.  
- One of the well-known "Gang of Four" design patterns, which describe how to solve recurring problems in object-oriented software, the pattern is useful when exactly one object is needed to coordinate actions across a system.  
- More specifically, the singleton pattern allows objects to:  
1). Ensure they only have one instance  
2). Provide easy access to that instance  
3). Control their instantiation (for example, hiding the constructors of a class)  

[Singleton Design Pattern | Singleton Class | Hindi](https://www.youtube.com/watch?v=UjP8YPTVONU)

**Why we need Singleton design Pattern?**
- When there is single resource throughout the application, for example database, log file etc.
- When there is a single resource and there is very high chance for deadlock.
- When we want to pass instance from one class to another class.

**Implementation :**  
- In .NET, the Singleton pattern is implemented using a private constructor and a static field that holds the single instance of the class.  
- Class should be sealed and its constructor should be private.  

![image](https://github.com/user-attachments/assets/4c4e67a9-2d3f-46cd-a884-b101035ad3fa)

♉ 𝐖𝐡𝐲 𝐒𝐞𝐚𝐥𝐞𝐝 ?
We want only one Instance of Singleton class , when a class inherits a call to parent constructor comes which is eventually causing a object creation (Kind of objection creation not exactly). So to avoid it we have made Singleton class sealed. Now no other class can inherit from it.

♉ 𝐖𝐡𝐲 𝐂𝐨𝐧𝐬𝐭𝐫𝐮𝐜𝐭𝐨𝐫 𝐢𝐬 𝐏𝐫𝐢𝐯𝐚𝐭𝐞 ?
To avoid multiple instance creation of Singleton class we have made constructor private. When constructor is private no one can create object of that class using new Singleton() out of that class.

♉ 𝐇𝐨𝐰 𝐚𝐫𝐞 𝐰𝐞 𝐬𝐮𝐫𝐞 𝐭𝐡𝐚𝐭 𝐨𝐧𝐥𝐲 𝐨𝐧𝐞 𝐈𝐧𝐬𝐭𝐚𝐧𝐜𝐞 𝐰𝐨𝐮𝐥𝐝 𝐛𝐞 𝐜𝐫𝐞𝐚𝐭𝐞𝐝  
▶ Sealed class make sure that no class can inherit it

▶ Constructor is private so no way for direct instantiation.

▶ The read only keyword ensures that the lazy property is initialized only once, either during the static constructor or before the class is first accessed, and is never modified again. But only read only keyword does not guarantee it right ! That’s why we have made property of only getter type you would note it does not have setter in it.

[Singleton Design Pattern Implementation in C# (Thread Safe)](https://medium.com/@mwaseemzakir/singleton-design-pattern-implementation-in-c-thread-safe-4b0fd536d821)

**Pros and Cons of Singleton Pattern :**

![image](https://github.com/user-attachments/assets/a0f7e3f9-d299-477c-a3fa-3f8fe434db91)

**C# Code:**  
```csharp
public sealed class Singleton
{
private static readonly Singleton instance = new Singleton();
private Singleton() { }
public static Singleton Instance
{
get { return instance; }
}
public void SomeMethod()
{
Console.WriteLine( Singleton method called.”);
}
}
```

Let's see another piece of Code :

```csharp
using System;
namespace SingleTonExample
{
public sealed class SingleTonClass
{
private static SingleTonClass instance;
private static object obj;

private singleTonclass() { }

public static singleTonclass GetInstance()
{
lock (obj)
{
if (instance == null)
{
instance = new Singletonclass()il |
}
}
return instance;
}
}
class Program
{
static void Main(string[] args)
{
singleTonClass s = SingleTonClass.GetInstance();
}
}
}
```

**Singleton Vs Static Class**

The big difference between a singleton and a bunch of static methods is that singletons can implement interfaces.

[C# : Singleton Vs Static Class](https://rajeevdotnet.blogspot.com/2015/12/c-singleton-vs-static-class.html)

Other differences as below:
- Singleton object stores in Heap but, static object stores in stack
- We can clone the object of Singleton but, we cannot clone the static class object
- Singleton class follow the OOP(object oriented principles) but not static class
- We can implement interface with Singleton class but not with Static class.

The Singleton pattern has several advantages over static classes. 
- A singleton can extend classes and implement interfaces, while a static class cannot (it can extend classes, but it does not inherit their instance members). 
- A singleton can be initialized lazily or asynchronously while a static class is generally initialized when it is first loaded, leading to potential class loader issues. 

However the most important advantage, though, is that singletons can be handled polymorphic ally without forcing their users to assume that there is only one instance.

## 24). REST Api. How to create REST API?

REST, or REpresentational State Transfer, is an architectural style for providing standards between computer systems on the web, making it easier for systems to communicate with each other. [How To Build a RESTful API with ASP.NET Core](https://medium.com/net-core/how-to-build-a-restful-api-with-asp-net-core-fb7dd8d3e5e3)

REST relies on client-server relationship. This essentially means that client application and server application must be able to evolve separately without any dependency on each other.

REST is stateless. That means the communication between the client and the server always contains all the information needed to perform the request. There is no session state in the server, it is kept entirely on the client’s side.

REST provides a uniform interface between components. Resources expose directory structure-like URIs.

REST is not strictly related to HTTP, but it is most commonly associated with it. There are four basic HTTP verbs we use in requests to interact with resources in a REST system:  

- GET — retrieve a specific resource (by id) or a collection of resources  
- POST — create a new resource  
- PUT — update a specific resource (by id)  
- DELETE — remove a specific resource by id  

Our API will manage movie records stored in a relational database as described in the table below:

![image](https://github.com/user-attachments/assets/8cc8745a-1b8f-4ac8-bae7-3657454f1ac4)

In a REST system, representations transfer JSON or XML to represent data objects and attributes.

🎮 You can implement a matchmaking service using REST APIs or SignalR to find and assign players to games based on their skills, region, or preferences.

[How To Build a RESTful API with ASP.NET Core](https://medium.com/net-core/how-to-build-a-restful-api-with-asp-net-core-fb7dd8d3e5e3)

[Create rest API in .Net Core](https://medium.com/@sagarkumar2499/create-rest-api-in-net-core-b2aed00416fd)

## 25). What are HTTP Verbs? 

HTTP verbs are a set of standardized HTTP methods used to specify the desired action to be performed on a resource. 
In REST API, these verbs are the means by which clients communicate their intentions to the server. 
The most commonly used HTTP verbs in REST API are :

1. GET : The GET method is used to request data from the server.
2. POST : POST is employed to create a new resource on the server.
3. PUT : The PUT method is used to update an existing resource on the server.
4. DELETE : DELETE is used to remove a resource from the server.

HTTP Methods like Put, GET, POST, PATCH , DELETE , OPTIONS explained in Hindi. This is part of the API Testing with Rest Assured Series.

✅ **What is GET Request?**  
The HTTP GET method is used to *read* (or retrieve) a representation of a resource. In the “happy” (or non-error) path, GET returns a representation in XML or JSON and an HTTP response code of 200 (OK). In an error case, it most often returns a 404 (NOT FOUND) or 400 (BAD REQUEST).  

✅ **What is POST Request?**  
The POST verb is most-often utilized to *create* new resources. In particular, it's used to create subordinate resources. That is, subordinate to some other (e.g. parent) resource. In other words, when creating a new resource, POST to the parent and the service takes care of associating the new resource with the parent, assigning an ID (new resource URI), etc.  


✅ **What is PATCH Request?**  
PATCH is used for *modify* capabilities. The PATCH request only needs to contain the changes to the resource, not the complete resource.  


✅ **What is Delete Request?**  
DELETE is pretty easy to understand. It is used to *delete* a resource identified by a URI.  


✅ **What is PUT Request?**  
PUT is most-often utilized for *update* capabilities, PUT-ing to a known resource URI with the request body containing the newly-updated representation of the original resource.  

However, PUT can also be used to create a resource in the case where the resource ID is chosen by the client instead of by the server. In other words, if the PUT is to a URI that contains the value of a non-existent resource ID. Again, the request body contains a resource representation. Many feel this is convoluted and confusing. Consequently, this method of creation should be used sparingly, if at all.  

✅ **Put VS Patch vs post**  
POST creates an item in a collection. PUT replaces an item. PATCH modifies an item.   
 
[HTTP Verbs in REST API](https://medium.com/@alrazak/understanding-http-verbs-in-rest-api-f6080711d580)

## 26). What is Data Annotations(Validations)? Client Side and Server Side Validations.


## 27). What is Pagination?  


## 28). Improve (Optimize) performance of Dot Net Application.

<p align="center">
  <img src="https://github.com/user-attachments/assets/7803c0d0-dcc5-42d1-90f9-a11907e915ea" width="500" height="250" />
</p>

**1. Caching Data :** Caching is a technique used in software development to store data that is computationally expensive or frequently accessed temporarily. The server is called, and the reply obtained is saved. As a result, the next time a request is made for a similar response, instead of going to the server, the data is verified against the cached data, and if they match, the stored data is obtained.  
Output Caching is an approach used for improving the performance of an MVC application. It is used for enabling its users to cache the data sent back by the controller method so that the data used earlier does not get generated each time while invoking the same controller method.
It has advantages to use Output Caching as it cuts down database server round trips, minimizes server round trips as well as reduces the network traffic.

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

[Controller Action Return Types in ASP.NET Core Web API](https://dotnettutorials.net/lesson/controller-action-return-types-core-web-api/)

## Minor Projects

#### 1). CRUD Operations

- [ASP.NET Core Web Application](https://www.youtube.com/watch?v=T-e554Zt3n4)

- [CRUD Operations using ASP.NET Core MVC](https://www.youtube.com/watch?v=SfWuOFEatYc)

#### 2). Custom Identity

- [Custom Identity in Asp.Net Core MVC](https://www.youtube.com/watch?v=93ssXlCPcuI&t=2158s)

## ASP.NET MVC

- ▶️ [Learn ASP.NET MVC (.NET 6)](https://www.youtube.com/watch?v=H14S7x8q_vQ&list=PLqVWQ84m1Q7EiKKyOpiWSVcpG3qUmJ0Xc&index=1)

- ▶️ [Full Course - Learn ASP.NET Core MVC in .NET 8 | CRUD Operations](https://www.youtube.com/watch?v=BzlPrVB_DwA)

## Best Channels PlayLists

- ▶️ [Code Unparalleled .NET Playlist](https://www.youtube.com/@CodeUnparalleled/playlists)

