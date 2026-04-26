# Orchestrating PostgreSQL with .NET Aspire

[![CI](https://github.com/konradcinkusz/dotNET-Aspire-postgress-article/actions/workflows/ci.yml/badge.svg)](https://github.com/konradcinkusz/dotNET-Aspire-postgress-article/actions/workflows/ci.yml)
[![PDF](https://img.shields.io/badge/PDF-download-blue)](https://github.com/konradcinkusz/dotNET-Aspire-postgress-article/releases/latest/download/Aspire_PostgreSQL_Setup.pdf)

A practical technical article documenting every step required to add a .NET Aspire AppHost project to an existing .NET 9 solution, using the HobbitGame Blazor application as the worked example. The article covers PostgreSQL container setup with persistent named Docker volumes, pgAdmin 4 integration, ServiceDefaults for OpenTelemetry, and secure credential management via .NET User Secrets.

## Quick start

```sh
git clone https://github.com/konradcinkusz/dotNET-Aspire-postgress-article.git
cd dotNET-Aspire-postgress-article

# Compile the PlantUML diagram once
plantuml -tpng figures/aspire-wiring.plantuml

# Build the article
latexmk -xelatex main.tex
```

The resulting PDF (`main.pdf`) renders the full article with inline C# and XML listings, two reference tables, one architecture diagram, and a bibliography.

## Repository structure

```
.
├── .github/workflows/ci.yml           # PDF build and GitHub Release on tag push
├── chapters/
│   ├── 01-introduction.tex
│   ├── 02-theoretical-background.tex
│   ├── 03-apphost-project.tex
│   ├── 04-program-wiring.tex
│   ├── 05-servicedefaults.tex
│   ├── 06-web-project-integration.tex
│   ├── 07-secrets-and-credentials.tex
│   ├── 08-discussion.tex
│   ├── 09-conclusion.tex
│   └── 10-references.tex
├── figures/
│   └── aspire-wiring.plantuml         # Figure 1 source (compiled to PNG by CI)
├── main.tex                           # entry point
├── .gitignore
└── README.md
```

## Article outline

| Section | Topic |
|---------|-------|
| 1. Introduction | Problem statement: local database setup friction; what Aspire solves |
| 2. Background | DistributedApplication model, resource types, container lifetimes, WaitFor, ServiceDefaults |
| 3. AppHost project | `.csproj` structure, secondary SDK, `UserSecretsId`, solution registration |
| 4. Program.cs wiring | Canonical `Program.cs`; table of every builder call and its omission failure mode |
| 5. ServiceDefaults | Library structure, `Extensions.cs`, OpenTelemetry export, health endpoints |
| 6. Web project integration | NuGet additions, `Program.cs` changes, DbContext registration, migration ordering |
| 7. Secrets and credentials | How Aspire generates passwords, User Secrets store, pgAdmin credentials, pitfall table |
| 8. Discussion | Production scope, EnsureCreated vs. MigrateAsync, team environments, Docker Compose relation |
| 9. Conclusion | Summary and onboarding impact |

## Master prompt

This article was produced using the following master prompt, which can be reused to apply the same Aspire setup to any other .NET solution. Replace the placeholders with values relevant to your project:

```
I have an existing .NET <NET_VERSION> solution.
Solution file: <PATH_TO_SLN>
Web / API project: <WEB_PROJECT_NAME> (csproj path: <PATH_TO_WEB_CSPROJ>)

I want you to add a .NET Aspire AppHost project that:
1. Orchestrates a PostgreSQL container with a persistent named Docker volume.
2. Adds a pgAdmin 4 container for browser-based DB inspection during development.
3. Sets ContainerLifetime.Persistent on the Postgres resource.
4. Registers a logical database named "<DB_RESOURCE_NAME>".
5. Wires the existing <WEB_PROJECT_NAME> project via WithReference and WaitFor.
6. Exposes the web app's HTTP/HTTPS endpoints externally.
7. Adds a ServiceDefaults library with OpenTelemetry, HTTP resilience, and service discovery.
8. Adds Aspire.Npgsql.EntityFrameworkCore.PostgreSQL and registers AddNpgsqlDbContext<AppDbContext>("<DB_RESOURCE_NAME>").
9. Stores generated secrets in .NET User Secrets.
10. All projects target .NET <NET_VERSION>.
```

| Placeholder | HobbitGame value | Description |
|-------------|-----------------|-------------|
| `<NET_VERSION>` | `9` | Target framework major version |
| `<PATH_TO_SLN>` | `HobbitGame.sln` | Relative path to solution file |
| `<WEB_PROJECT_NAME>` | `HobbitGame.Web` | The project Aspire orchestrates |
| `<PATH_TO_WEB_CSPROJ>` | `src/HobbitGame.Web/HobbitGame.Web.csproj` | Relative csproj path |
| `<SOLUTION_PREFIX>` | `HobbitGame` | Common prefix for AppHost / ServiceDefaults names |
| `<DB_RESOURCE_NAME>` | `hobbitdb` | Logical database name |
| `<ASPIRE_VERSION>` | `9.0.0` | NuGet version for all `Aspire.*` packages |

## Releasing

Pushing a tag of the form `v*` (e.g. `v1.0.0`) triggers the GitHub Actions workflow in `.github/workflows/ci.yml`. The workflow compiles the PlantUML diagram, runs XeLaTeX via `latexmk`, and publishes the resulting PDF as a GitHub Release artifact.

```sh
git tag v1.0.0
git push origin v1.0.0
```
