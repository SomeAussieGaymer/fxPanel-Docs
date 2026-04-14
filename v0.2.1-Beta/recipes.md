# Recipes

YAML-based deployment automation for setting up servers in under 60 seconds. Recipes are "jailed" to the target folder and cannot write outside it. At the end of deployment, the target folder is checked for a `server.cfg` and a `resources` folder.

---

## Recipe Format

Recipes are YAML files that define a sequence of automated steps to deploy a server. They are validated with Zod schemas and support variable interpolation.

```yaml
name: My Server Recipe
version: 1.0.0
author: YourName
description: A basic FiveM server setup

$engine: 3
$minFxVersion: 7290
$onesync: on

variables:
    dbHost: localhost
    dbUsername: root
    dbPassword: ''
    dbName: null

tasks:
  - action: download_github
    src: https://github.com/user/repo
    ref: main
    dest: ./resources/[my-resources]

  - action: download_file
    url: https://example.com/asset.zip
    path: ./temp/asset.zip

  - action: unzip
    src: ./temp/asset.zip
    dest: ./resources/[assets]

  - action: write_file
    file: ./server.cfg
    data: |
      endpoint_add_tcp "0.0.0.0:{{serverEndpoints}}"
      sv_maxclients {{maxClients}}
      sv_licenseKey "{{svLicense}}"

  - action: remove_path
    path: ./temp
```

### Meta Fields

| Field | Description |
|-------|-------------|
| `name` | Recipe display name |
| `version` | Recipe version (semver) |
| `author` | Recipe author name |
| `description` | Recipe description (recommended under 256 characters) |
| `$engine` | Recipe engine version |
| `$minFxVersion` | Minimum required FXServer build number |
| `$onesync` | OneSync mode requirement |
| `$steamRequired` | Whether Steam is required to join |

---

## Context Variables

These variables are automatically populated during deployment and can be used in `write_file` and `replace_string` actions with `{{variableName}}` syntax.

| Variable | Description |
|----------|-------------|
| `{{deploymentID}}` | Unique ID for this deployment (e.g. PlumeESX_BBC957) |
| `{{serverName}}` | Configured server name |
| `{{recipeName}}` | Populated from the recipe metadata |
| `{{recipeAuthor}}` | Populated from the recipe metadata |
| `{{recipeVersion}}` | Populated from the recipe metadata |
| `{{recipeDescription}}` | Populated from the recipe metadata |
| `{{dbHost}}` | Database host address |
| `{{dbPort}}` | Database port |
| `{{dbUsername}}` | Database username |
| `{{dbPassword}}` | Database password |
| `{{dbName}}` | Database name |
| `{{dbDelete}}` | Whether to delete existing database |
| `{{dbConnectionString}}` | Full database connection string |
| `{{svLicense}}` | CFX license key |
| `{{serverEndpoints}}` | Server endpoint configuration |
| `{{maxClients}}` | Maximum player slots |

### Custom Variables

You can define custom variables in the recipe's `variables` block. These are loaded into the deployer context and can be overridden by user input in step 2.

```yaml
variables:
    dbHost: localhost
    dbUsername: root
    dbPassword: ''
    dbName: null
```

---

## Available Actions

Tasks are executed sequentially — any failure stops the process. Every task can include a `timeoutSeconds` option to increase its default timeout.

### Download

| Action | Description |
|--------|-------------|
| `download_github` | Clone or download a GitHub repository. Supports optional `ref` (branch/tag/commit) and `subpath`. Uses GitHub API to resolve default branch if ref is omitted. |
| `download_file` | Download a file from a URL to a specific path. |
| `unzip` | Extract a ZIP archive to a destination folder. |

### File System

| Action | Description |
|--------|-------------|
| `move_path` | Move a file or directory. Supports optional `overwrite` flag. |
| `copy_path` | Copy a file or directory. Supports optional `overwrite` and `errorOnExist` flags. |
| `remove_path` | Delete a file or directory. Silently does nothing if path doesn't exist. |
| `ensure_dir` | Create a directory if it doesn't exist. |
| `write_file` | Write content to a file (supports variable interpolation). Optional `append` mode. |
| `replace_string` | Find and replace text in a file or array of files. Supports `template` (default), `all_vars`, and `literal` modes. |

### Database

| Action | Description |
|--------|-------------|
| `connect_database` | Connect to a MySQL/MariaDB server using context variables. Creates the database if `dbName` is null. Must be called before `query_database`. |
| `query_database` | Execute a SQL query (from a file path or inline string) against the connected database. |

### Utility

| Action | Description |
|--------|-------------|
| `waste_time` | Wait for a specified number of seconds. Useful for rate-limit delays between downloads. |
| `load_vars` | Load variables from a JSON file into the deployer context. |
