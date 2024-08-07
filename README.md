# Self-Service Active Directory
## _The good way account management on Active Directory_


## Introduce

> This application is closed source, the codes of the software have not been shared.

Self-Service AD (SSAD) is a web-based password management solution. It eliminates the users dependency on administrators to change or reset their passwords. It reduces the workload of the help desk and in turn reduces the cost incurred by the company.

Every day help desk and IT personnel waste time handling minor problems that regular users can solve by themselves. Every password reset, create user, user info update, enable/disable user in Active Directory involves skilled personnel whereas users can easily perform these tasks for themselves. Providing users with the possibility of Active Directory self-service saves human resources and streamlines updating of the active directory information. This app helps your Core IT Staff focus on the daily challenges to run and improve your Environment.

This application used in 9 different organizations, the password reset workload of IT personnel has decreased by 65% with the self-service password reset feature in the last 3 years.

In cases where the user does not have an active directory account, the phone number is not registered or is not up to date, or the account is inactive, it automatically creates a ticket to the IT department and if the IT department approves the ticket, it automatically performs all the necessary active directory user transactions and sends the information to the user's phone via SMS. 

With automatically created tickets, IT employees can perform transactions with the tickets in the portal without having to perform manual transactions with the active Directory console or 3rd party Active Directory applications.


## Features

What can IT departments do?
- Can create and reset accounts through the administration panel.
- Can inspect organizational units and user details.
- Can enable/disable/lock and unlock users.
- Can list all user actions via portal.
- Can list locked users,password expired users and never password expires users.
- Can manage profile update requests.
- Access can be restricted for the unwanted user.
- System administrators can create authorized delegate users for certain organizational units for IT department employees. Thus, delegated users can only create and view users within the organizational units for which they are authorized.
- Can store and view the contracts signed by users by uploading them to the portal.

What can users do?
- It can perform password reset, password change,password check,disable user and account opening operations via portal.

**How self-service password reset works**
- A user who has forgotten their password initiates a password reset request from either the Self-Service AD web portal.
- Self-Service AD checks the LDAP user status.
- After successful identity verification, a random password is generated according to the predetermined password criteria and sent to the registered mobile phone number of the user.
- The user will then be able to log in to their account using the newly reset password.
  

### Advantages of Self-Service AD application

- Does not require a server installation that is a member of the Active Directory domain.
- The delegate user does not need to be a member of the Domain Admin group. For delegation, it is sufficient to define a delegation user with limited authority to the OUs and groups that the portal can process.
- Since it has a distributed architecture, projects can be located on different servers (UI, API, Background services).

### Prerequisites

```sh
Microsoft Active Directory
MongoDB
RabbitMQ
Redis
SQL Server
```

## Project structure

Project  | Technology  | Type | Description
------------- | ------------- | ------------- | -------------
SelfServiceAD.Business  | .NET 8.0  | Class Library | Business layer
SelfServiceAD.Caching  | .NET 8.0  | Class Library | Caching layer
SelfServiceAD.Core  | .NET 8.0  | Class Library | Core layer
SelfServiceAD.Data  | .NET 8.0  | Class Library | Data access layer
SelfServiceAD.Entities | .NET 8.0 | Class Library | Entities
SelfServiceAD.LdapServices | .NET 8.0  | Class Library | LDAP communication layer
SelfServiceAD.Messaging | .NET 8.0  | Class Library | Message broker services
SelfServiceAD.WebAPI | .NET 8.0 | Web Application | All API endpoints for UI
SelfServiceAD.WebUI  | .NET 8.0 | Web Application | Razor pages

## Database structure

Description  | Database type  | Purpose
------------- | ------------- | -------------
Application database | MS SQL Server  | Stores application data,identity information and active directory organizational unit and group paths and other application-related details
Hangfire database| MS SQL Server  | Stores Hangfire server data
Log database | MongoDB | Stores application log data


## Layers

### Business Layer

The Business project provides a large number of classes that handle business rules, validations, LDAP calls, message queue operations, constants, option definitions, and caching operations used throughout the application. It also provides CRUD operations by calling the repositories in the Data layer.

**System.DirectoryServices** and **System.DirectoryServices.AccountManagement** libraries are used for active directory account management.

The `JsonStringLocalizer` helper methods in the Core project and the Business layer throughout the application contain the Response class, which is the common return model of WebAPI endpoints.

Example response class:

```
{
    "data": [
        {
            "companyId": 73,
            "description": "Company A"
        },
        {
            "companyId": 74,
            "description": "Company B"
        }
    ],
    "statusCode": 200,
    "resultCode": 0,
    "isSuccess": true,
    "message": null,
    "hasData": true
}
```

### Data access layer

In the Data project, the necessary contexts have been written to access the MSSQL database using Repository pattern and EntityFramework, and the MongoDB database with MongoClient. UnitOfWork pattern is used for SQL database. Sample data is added to the empty database with the seeding classes in the `/Migrations` directory.

### Entities layer

DTO, Model and Entities were written in the Entities project. Each class implements an abstract class.

Defined interfaces|
-------------|
IModel |
IDto |
ISqlBaseEntity|
INoSqlBaseEntity|

### WebUI and WebAPI projects

The WebUI project communicates with the SQL database and the endpoints in the WebAPI project. Classes that communicate with API services make HTTP client requests through the classes added to the `/ApiServices` directory of the WebUI project. A separate ApiService class is written for each controller.

API endpoints authenticate with JWT. It is configured for JWT authentication with the `/Extensions/AddIdentityExtension` method. Some endpoints related to background services require authentication with API key. (Example: User export operations, Caching services..) `ApiKey` methods in `/Attributes` directory are ApiKey validation at Controller level. (Example: `CacheController,NotificationsController,UserImportsController` ). API key definitions are stored in the `appSettings.json` file. (Example: NotificationApiKey, UserExportApiKey)

Localization related settings are defined in `/Core/Helpers/JsonStringLocalizer` and `/Core/Helpers/JsonStringLocalizerFactory` classes.

The necessary settings for CORS configuration have been added to the `Program.cs` file as middleware with the `/Extensions/AddCorsOptionExtensions` extension method.

It has been added to the `Program.cs` file as middleware with the `/Extensions/AddSwaggerExtensions` and `/Extensions/AddSwaggerAuthorized` extension methods for the Swagger implementation. The swagger user and password are set in the 'SwaggerOptions' node of the `appSettings.json` file.

FluentAPI validations for model validations are written in classes added to the `Business/ValidationRules/FluentValidation` directory. Validations within WebAPI and WebUI projects have been added to the `Program.cs` file as middleware with the `/Extensions/AddValidatorsExtensions` extension method.

Validations in `/WebAPI/` and `/WebUI/` projects are written in object mapping classes added to the `/Mapping/` directory.

The logging operations are written in the `WebUI/Filters/TimerFilter` class to monitor the action performances. Detailed information such as the execution time of the action performances are sent to the RabbitMQ message queue with this class. The logging messages are read and written to MongoDB by the background service worker named `LoggingServices`.

Database contexts used for WebAPI and WebUI projects have been added to the Program.cs file as middleware with the `/Extensions/AddDbContextsExtensions` extension method.

Classes related to Hangfire implementations have been added to the Program.cs file as middleware with the `/Extensions/AddJobsExtensions` extension method in the WebUI project.

In order to map the settings used in the application from the `appSettings.json` file, the classes in the `Business/Abstract/Options` and `Business/Concrete/Options` directory are written. By using [Options pattern](https://docs.microsoft.com/en-us/dotnet/core/extensions/options), the records in the `appSettings.json` file are structured and implemented with Dependency Injection.

RateLimiting related settings are defined in `ClientRateLimiting` node of `appSettings.json` file.

ElasticAPM is used to monitor the performance of applications. ElasticApm related settings are defined in the `ElasticApm` node of the `appSettings.json` file.

In the WebAPI project, the '/Extensions/IpSafeMiddleware' class has been added as middleware to the 'Program.cs' file so that only certain IP addresses in the 'IpOptions' node of the 'appSettings.json' file can access the API.

The JWT settings used for authentication are in the `appSettings.json` file `TokenOptions` node.

Serilog-related settings are defined in the `Serilog` node of the `appSettings.json` file for dynamic logging. By default, File logging and MongoDB sinks are defined.

**Redis** is preferred as the cache technology in the application architecture. The data required for the control of active directory users are used by keeping them in the cache.

JSON files are used to store localized strings and implement middleware to switch languages ​​via language keys in the header. Via IDistributedCache, strings are retrieved from the cache with the JsonStringLocalizer class, which implements the IStringLocalizer abstract class. Strings are created in JSON format in the /Resources/ directory of WebAPI and WebUI projects. The language in which the application will start broadcasting as soon as it stands up is defined in the `Language` node of the `appSettings.json` file.
In the WebUI project, the language settings can be changed dynamically with the help of the `ViewComponents/LanguageSwitcherViewComponent` class by selecting them from the drop down list added to the layouts. Static MVC pages are displayed in the selected language.
In addition, after the language selection made in the WebUI project, the `Accept-Language` header information for API requests is automatically set with the help of API call classes in the `/ApiServices` directory.

### Notification services

In the NotificationServices project, the services required for sending SMS, Slack and E-mail are written.  [Twilio](https://www.twilio.com/),[Teknomart](http://www.teknomart.com.tr/),[NetGSM](https://www.netgsm.com).Also, Generic SMS service has been written for companies that support HTTP GET method.

**RabbitMQ** is preferred as the message broker in the application architecture. Asynchronous endpoints such as sending SMS are made through services that process RabbitMQ message queues. 

The application deserializes the pending notifications in the message queue and sends the incoming object via the relevant service. Since the messages waiting in the message queu are of a generic nature, a service can be started from Twilio by standing up for the Twilio service at the time of T, or sending from the Generic sms service can be started by standing up for the Generic sms service. After the services are running, they are sent to the logging queue and written to MongoDB by the background service named NotificationServices.

## Technology stack

Self Service Active Directory Application uses a number of projects to work properly:

- [AutoMapper.Extensions.Microsoft.DependencyInjection](https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/)
- [FluentValidation](https://www.nuget.org/packages/FluentValidation/)
- [Hangfire.Core](https://www.nuget.org/packages/Hangfire.Core/1.7.28)
- [jQuery](https://www.nuget.org/packages/jQuery/)
- [Microsoft.AspNetCore.Authentication.JwtBearer](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.JwtBearer/6.0.5)
- [Microsoft.AspNetCore.Identity](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/)
- [Microsoft.AspNetCore.Identity.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore/6.0.5)
- [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore/6.0.5)
- [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/6.0.5)
- [MongoDB.Driver](https://www.nuget.org/packages/MongoDB.Driver/)
- [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/)
- [RabbitMQ.Client](https://www.nuget.org/packages/RabbitMQ.Client/)
- [RestSharp](https://www.nuget.org/packages/RestSharp/107.3.0)
- [Serilog](https://www.nuget.org/packages/Serilog/2.11.0)
- [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/)
- [Swashbuckle.AspNetCore](https://www.nuget.org/packages/Swashbuckle.AspNetCore/)
- [System.DirectoryServices](https://www.nuget.org/packages/System.DirectoryServices/6.0.0)
- [System.DirectoryServices.AccountManagement](https://www.nuget.org/packages/System.DirectoryServices.AccountManagement/6.0.0)

