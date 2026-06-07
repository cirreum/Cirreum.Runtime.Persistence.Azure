# Cirreum.Runtime.Persistence.Azure

[![NuGet Version](https://img.shields.io/nuget/v/Cirreum.Runtime.Persistence.Azure.svg?style=flat-square&labelColor=1F1F1F&color=003D8F)](https://www.nuget.org/packages/Cirreum.Runtime.Persistence.Azure/)
[![NuGet Downloads](https://img.shields.io/nuget/dt/Cirreum.Runtime.Persistence.Azure.svg?style=flat-square&labelColor=1F1F1F&color=003D8F)](https://www.nuget.org/packages/Cirreum.Runtime.Persistence.Azure/)
[![GitHub Release](https://img.shields.io/github/v/release/cirreum/Cirreum.Runtime.Persistence.Azure?style=flat-square&labelColor=1F1F1F&color=FF3B2E)](https://github.com/cirreum/Cirreum.Runtime.Persistence.Azure/releases)
[![License](https://img.shields.io/github/license/cirreum/Cirreum.Runtime.Persistence.Azure?style=flat-square&labelColor=1F1F1F&color=F2F2F2)](https://github.com/cirreum/Cirreum.Runtime.Persistence.Azure/blob/main/LICENSE)
[![.NET](https://img.shields.io/badge/.NET-10.0-003D8F?style=flat-square&labelColor=1F1F1F)](https://dotnet.microsoft.com/)

**Simplified `Cirreum.Persistence.Azure` configuration for Cirreum runtime applications**

## Overview

**Cirreum.Runtime.Persistence.Azure** provides a unified persistence layer configuration for the Cirreum framework. It simplifies database integration by offering a single extension method that automatically registers and configures multiple persistence providers with built-in health checks.

## Installation

```bash
dotnet add package Cirreum.Runtime.Persistence.Azure
```

## Usage

Add persistence to your application with a single line:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add persistence support
builder.AddPersistence();

var app = builder.Build();
```

This automatically:
- Registers Azure Cosmos DB persistence services
- Configures health checks for database connectivity
- Prevents duplicate service registration
- Integrates with the Cirreum service provider infrastructure

## Supported Providers

| Provider | Package | Database |
|----------|---------|----------|
| Azure Cosmos DB | `Cirreum.Persistence.Azure` | NoSQL document database |

## Features

- **Simple Integration**: One method to configure all persistence needs
- **Declarative Indexing Policies**: Entity attribute-driven Cosmos DB indexing configuration (included/excluded paths, composite indexes, spatial indexes)
- **Health Checks**: Automatic health check registration for monitoring
- **Duplicate Prevention**: Smart registration prevents service conflicts
- **Extensible Design**: Built on the Cirreum service provider pattern for easy extension

## Declarative Indexing Policy

When `IsAutoResourceCreationEnabled` is `true`, Cosmos DB containers are auto-created with indexing policies defined on your entity classes via attributes from [`Cirreum.Persistence.NoSql`](https://github.com/cirreum/Cirreum.Persistence.NoSql):

```csharp
[Container("tasks")]
[PartitionKeyPath("/clientId")]
[IndexingPolicy(IndexingMode.Consistent, Automatic = true)]
[ExcludedPath("/description/*")]
[ExcludedPath("/*")]
public record TaskItem : Entity {

    [IncludedPath]
    [CompositeIndex("type-client-date", CompositePathSortOrder.Ascending, position: 0)]
    public string Type { get; set; }

    [IncludedPath]
    [CompositeIndex("type-client-date", CompositePathSortOrder.Ascending, position: 1)]
    public string ClientId { get; set; }

    [IncludedPath]
    [CompositeIndex("type-client-date", CompositePathSortOrder.Descending, position: 2)]
    public DateTimeOffset CreatedAt { get; set; }

}
```

Entities without `[IndexingPolicy]` use the Cosmos DB default policy (auto-index all paths). For full attribute documentation, see [Cirreum.Persistence.NoSql](https://github.com/cirreum/Cirreum.Persistence.NoSql) and [Cirreum.Persistence.Azure](https://github.com/cirreum/Cirreum.Persistence.Azure).

## Configuration

Configure persistence providers in `appsettings.json`:

```json
{
  "ServiceProviders": {
    "Persistence": {
      "Azure": {
        "default": {
          "Name": "MyCosmosDb",
          "DatabaseId": "my-database",
          "OptimizeBandwidth": true,
          "IsAutoResourceCreationEnabled": true
        }
      }
    }
  },
  "ConnectionStrings": {
    "MyCosmosDb": "AccountEndpoint=https://your-account.documents.azure.com:443/;AccountKey=..."
  }
}
```

The `Name` property resolves the connection string via `Configuration.GetConnectionString(name)`. For production, store connection strings in Azure Key Vault using the naming convention `ConnectionStrings--{Name}`.

For detailed configuration options, see the individual provider documentation:
- [Cirreum.Persistence.NoSql](https://github.com/cirreum/Cirreum.Persistence.NoSql) (abstractions)
- [Cirreum.Persistence.Azure](https://github.com/cirreum/Cirreum.Persistence.Azure) (implementation)

## Versioning

Cirreum.Runtime.Persistence follows [Semantic Versioning](https://semver.org/):

- **Major** - Breaking API changes
- **Minor** - New features, backward compatible
- **Patch** - Bug fixes, backward compatible

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

**Cirreum Foundation Framework**  
*Layered simplicity for modern .NET*