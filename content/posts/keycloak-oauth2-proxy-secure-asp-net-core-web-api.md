
+++
authors = ["Jun Wu"]
title = "Keycloak + OAuth2 Proxy to Protect Your ASP.NET Core Web API - 2024"
date = "2024-02-29"
description = "a guide for Keycloak & OAuth2 Proxy & ASP.NET Core Web API."
tags = [
    "oauth2-proxy",
    "keycloak",
    "ASP.NET Core"
]
categories = [
    "beckend",
    "middleware"
]
series = ["ASP.NET Core","Keycloak","oauth2-proxy"]
+++

## Introduction
Keycloak, OAuth2 Proxy, and ASP.NET Core Web API work together to secure your web application. 
Let's dive into how these components interact using a mermaid sequence diagram.

{{<mermaid>}}
sequenceDiagram
    participant User
    participant Keycloak
    participant OAuth2 Proxy
    participant ASP.NET Core Web API

    User -> Keycloak: Login
    Keycloak -> User: Redirect to OAuth2 Proxy
    User -> OAuth2 Proxy: Request
    OAuth2 Proxy -> ASP.NET Core Web API: Forward Request
    ASP.NET Core Web API -> OAuth2 Proxy: Get Roles from Headers
    OAuth2 Proxy -> User: Response

{{</mermaid>}}


OAuth2 Proxy Configuration
OAuth2 Proxy forwards headers to the upstream service by configuring:

```conf
pass_access_token = true
pass_authorization_header = true
```

### Get access token from headers in ASP.NET Core Web API

```c#
    public class CustomAuthenticationHandler : AuthenticationHandler<AuthenticationSchemeOptions>
    {
        public CustomAuthenticationHandler(
            IOptionsMonitor<AuthenticationSchemeOptions> options,
            ILoggerFactory logger,
            UrlEncoder encoder) : base(options, logger, encoder)
        {

        }

        protected override Task<AuthenticateResult> HandleAuthenticateAsync()
        {
            // Try to get the token from the header
            if (!Request.Headers.TryGetValue("x-forwarded-access-token", out var token))
            {
                return Task.FromResult(AuthenticateResult.Fail("Header Not Found."));
            }

            try
            {
                // Attempt to decode the JWT token
                var tokenHandler = new JwtSecurityTokenHandler();
                var jwtToken = tokenHandler.ReadJwtToken(token);
                var claims = jwtToken.Claims.Select(claim => new Claim(claim.Type, claim.Value)).ToList();

                var identity = new ClaimsIdentity(claims, Scheme.Name);

                var resourceAccessClaim = jwtToken.Claims.FirstOrDefault(c => c.Type == "resource_access")?.Value;
                if (resourceAccessClaim != null)
                {
                    var resourceAccessData = JsonSerializer.Deserialize<Dictionary<string, object>>(resourceAccessClaim);
                    if (resourceAccessData != null && resourceAccessData.TryGetValue("<your_realm_client_name_here>", out var expressMiddlewareData))
                    {
                        var expressMiddlewareDict = JsonSerializer.Deserialize<Dictionary<string, object>>(expressMiddlewareData.ToString());
                        if (expressMiddlewareDict != null && expressMiddlewareDict.TryGetValue("roles", out var roles))
                        {
                            var parsedRoles = JsonSerializer.Deserialize<List<string>>(roles.ToString());
                            if (parsedRoles != null)
                            {
                                foreach (var role in parsedRoles)
                                {
                                    identity.AddClaim(new Claim(ClaimTypes.Role, role));
                                }
                            }
                        }
                    }
                }

                var principal = new ClaimsPrincipal(identity);
                var ticket = new AuthenticationTicket(principal, Scheme.Name);
                return Task.FromResult(AuthenticateResult.Success(ticket));
            }
            catch
            {
                return Task.FromResult(AuthenticateResult.Fail("Invalid Token."));
            }
        }
    }
```

then you can config authentication and authorization 

```c#
        public static void Main(string[] args)
        {
            var builder = WebApplication.CreateBuilder(args);

            // Add services to the container.

            builder.Services.AddControllers();
            // Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
            builder.Services.AddEndpointsApiExplorer();
            builder.Services.AddSwaggerGen();
            // use custom authentication schema
            builder.Services.AddAuthentication("CustomScheme").AddScheme<AuthenticationSchemeOptions, CustomAuthenticationHandler>("CustomScheme", options => { });
            builder.Services.AddHttpContextAccessor();

            var app = builder.Build();

            // Configure the HTTP request pipeline.
            if (app.Environment.IsDevelopment())
            {
                app.UseSwagger();
                app.UseSwaggerUI();
            }

            app.UseAuthentication();
            app.UseAuthorization();

            app.MapControllers();

            app.Run();
        }
```

now you can protect you controller or action like this:

```c#
    [ApiController]
    [Authorize(AuthenticationSchemes = "CustomScheme")]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private readonly IHttpContextAccessor _httpContextAccessor;

        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        private readonly ILogger<WeatherForecastController> _logger;

        public WeatherForecastController(ILogger<WeatherForecastController> logger, IHttpContextAccessor httpContextAccessor)
        {
            _logger = logger;
            _httpContextAccessor = httpContextAccessor;
        }

        [Authorize(Roles = "admin")]
        [HttpGet(Name = "GetWeatherForecast")]
        public IEnumerable<WeatherForecast> Get()
        {
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
```


## Summary
### Pros:
- Enhanced security with Keycloak and OAuth2 Proxy
- Centralized authentication and authorization management
- Seamless integration with ASP.NET Core Web API
### Cons:
- Increased complexity in setup and configuration
- Potential performance overhead due to additional layers
- Dependency on external services for authentication and authorization


By leveraging Keycloak, OAuth2 Proxy, and ASP.NET Core Web API, you can create a robust and secure web application environment.