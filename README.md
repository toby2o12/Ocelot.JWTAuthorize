# Ocelot.JWTAuthorize
<img src="https://github.com/axzxs2001/Ocelot.JWTAuthorize/blob/master/Ocelot.JWTAuthorize/Ocelot.JWTAuthorize/githublogo.png" alt="GitHub" title="Ocelot.JwtAuthorize" width="360" height="200" />

[![NuGet Badge](https://buildstats.info/nuget/Ocelot.JwtAuthorize)](https://www.nuget.org/packages/Ocelot.JwtAuthorize/)
[![GitHub license](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/axzxs2001/Ocelot.JWTAuthorize/blob/master/LICENSE)


This library is used in the verification project when Ocelot is used as an API gateway. In the Ocelot project, the API project, the verification project, and the injection function can be used.


### 1. add the following sections to the appsetting. Json file for each project
```json
{
  "JwtAuthorize": {  
    "Secret": "ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890",
    "Issuer": "ocelot",
    "Audience": "everyone",
    "PolicyName": "permission",
    "DefaultScheme": "Bearer",
    "IsHttps": false,
    "RequireExpirationTime": true
  }
}
```

### 2. API Project 

>#### PM>Install-Package Ocelot.JWTAuthorize
Startup.cs
```c#
public void ConfigureServices(IServiceCollection services)
{
     services.AddApiJwtAuthorize((context) =>
     {
          //validate permissions return(permit) true or false(denied)
          return true;
     });
     services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```
API Controller
```C#
    [Authorize("permission")]
    [Route("api/[controller]")]
    [ApiController]
    public class ValuesController : Controller
```
### 3. Authorize Project

>#### PM>Install-Package Ocelot.JWTAuthorize
startup.cs
```C#
 public void ConfigureServices(IServiceCollection services)
{
      services.AddTokenJwtAuthorize();
      services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```
LoginController.cs
```C#
[HttpPost]
public IActionResult Login([FromBody]LoginModel loginModel)
{        
        if (loginModel.UserName == "gsw" && loginModel.Password == "111111")
        {
             var claims = new Claim[] {
                 new Claim(ClaimTypes.Name, "gsw"),
                 new Claim(ClaimTypes.Role, "admin")                  
             };     
             //DateTime.Now.AddSeconds(1200) is expiration time
             var token = _tokenBuilder.BuildJwtToken(claims, DateTime.Now.AddSeconds(1200));      
             return new JsonResult(new { Result = true, Data = token });
         }
         else
         {
             return new JsonResult(new
             {
                 Result = false,
                 Message = "Authentication Failure"
             });
         }
 }
```

### 4. Ocelot Project

>#### PM>Install-Package Ocelot.JWTAuthorize
Startup.cs
```C#
public void ConfigureServices(IServiceCollection services)
{
       services.AddOcelotJwtAuthorize();
       services.AddOcelot(Configuration);
       services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
```
