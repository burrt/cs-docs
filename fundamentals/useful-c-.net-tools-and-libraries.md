# Useful C#/.NET tools and libraries

I think it would be useful to list a number of tools and libraries that I use when setting up new services as well as the reference documentation. This may get out of date soon however it will at least give me a reference point to revisit. Some "templates" I've created:

* [AWS Lambda - .NET Console app](https://github.com/burrt/AwsLambdaDotnet)
* [AWS Lambda - .NET Web Api](https://github.com/burrt/AwsLambdaDotnetWebApi)
* [Shared services - Terraform](https://github.com/burrt/tf-shared-core-services)

## Logging

* [Serilog](https://github.com/serilog/serilog)
  * for json logs, it is particularly useful for logging services like AWS CloudWatch:
    * [blog 1](https://nblumhardt.com/2016/07/serilog-2-0-json-improvements/)
    * [blog series](https://nblumhardt.com/2016/06/structured-logging-concepts-in-net-series-1/)
  * [.NET/C# request logging improvements](https://github.com/serilog/serilog-aspnetcore#request-logging)
  * [Serilog.Enrichers.Sensitive](https://github.com/serilog-contrib/Serilog.Enrichers.Sensitive) - some protection against PII
  * [Destructurama.Attributed](https://github.com/destructurama/attributed) - a more powerful and flexible library, probably at a cost of runtime

## HTTP

* .NET's `HttpClient` is quite sufficient
* [Polly](https://github.com/App-vNext/Polly) - HTTP Request retries
* [Serilog HTTP request logging](https://github.com/serilog/serilog-aspnetcore)
  * It does require a few tweaks with [ASP.NET Core HTTP logging](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/http-logging)
  * Refer to the above templates

### Swagger/Open API

* .NET 9+ uses OpenAPI libraries for Swagger

## Database

Sometimes ORM (e.g. EF) isn't necessary, for mysql, the [MySqlConnector](https://github.com/mysql-net/MySqlConnector) seems quite handy.

## Testing

* [xUnit](https://github.com/xunit/xunit)/NUnit
* [Moq](https://github.com/devlooped/moq)
* [Specflow](https://github.com/SpecFlowOSS/SpecFlow) - deprecated

## Static code analysis

* SonarQube/Cloud - for code quality, coverage and security
* Veracode - security
* CodeQL - security
* Dependabot - security
* Snyk - security

## Observability

Microsoft has some new open source tools - [read here](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/observability-with-otel)

There are other built-in support for health checks, resource utilisation monitoring as well that is worth looking into. These now seem to have some integrations with more common open source tools such as Prometheus, Grafana, Kibana etc.
