# DuckLake + SQLMesh + PostgreSQL Demo

This project demonstrates how to integrate [DuckLake](https://ducklake.select/), [SQLMesh](https://sqlmesh.com/), [Neon PostgreSQL](https://neon.com/), and [Cloudflare R2](https://developers.cloudflare.com/r2/) to create a modern data lakehouse architecture with version control and state management.

## Overview

This setup showcases:
- **DuckLake**: A data lakehouse platform for storing and managing data in Parquet format
- **SQLMesh**: A DataOps framework for data transformations and pipeline management
- **Neon PostgreSQL**: Cloud-native PostgreSQL for catalog and state storage
- **Cloudflare R2**: Object storage for scalable, cost-effective data lake storage and integration with DuckLake
- **Custom SQLMesh Integration**: Uses a custom fork that adds `data_path` support for DuckDB catalogs

## Prerequisites

- Access to Neon PostgreSQL database

## Quick Start

### 1. Installation

Install dependencies using uv (or pip):

```bash
uv sync
```

### 2. Environment Setup

Set up your environment variables for Neon PostgreSQL and Cloudflare R2:

```bash
export PG__HOST="your-neon-host.neon.tech"
export PG__PORT="5432"
export PG__DATABASE="your-database"
export PG__USER="your-username"
export PG__PASSWORD="your-password"
export PG__ENDPOINT_ID="your-endpoint-id"

export R2__ACCOUNT_ID="your-account-id"
export R2__ACCESS_KEY_ID="your-key-id"
export R2__SECRET_ACCESS_KEY="your-secret-key"
```

### 3. Plan

```bash
uv run sqlmesh plan
```

## Configuration

### DuckLake Integration

The `config.yaml` configures DuckLake as a catalog within DuckDB:

```yaml
gateways:
  ducklake:
    connection:
      type: duckdb
      catalogs:
        lakehouse:
          type: ducklake
          path: postgres:host={{ env_var('PG__HOST') }} dbname={{ env_var('PG__DATABASE') }} user={{ env_var('PG__USER') }} password={{ env_var('PG__PASSWORD') }}
          data_path: data  # Custom enhancement for data storage
          encrypted: true # Custom enhancement for data encryption
      extensions:
        - ducklake
        - httpfs
      secrets:
        - type: r2
          account_id: {{ env_var('R2__ACCOUNT_ID') }}
          key_id: {{ env_var('R2__ACCESS_KEY_ID') }}
          secret: {{ env_var('R2__SECRET_ACCESS_KEY') }}
```

### Neon PostgreSQL State Storage

SQLMesh uses Neon PostgreSQL for storing execution state and metadata:

```yaml
state_connection:
  type: postgres
  host: {{ env_var('PG__HOST') }}
  port: {{ env_var('PG__PORT') }}
  database: {{ env_var('PG__DATABASE') }}
  user: {{ env_var('PG__USER') }}
  password: endpoint={{ env_var('PG__ENDPOINT_ID') }};{{ env_var('PG__PASSWORD') }}
  sslmode: require
```

> **NOTE:** Neon requires endpoint ID, which can be passed via the password field.

## Custom SQLMesh Fork

This project uses a custom fork of SQLMesh that adds `data_path` support for DuckDB catalogs:

- **Repository**: https://github.com/mattiasthalen/sqlmesh/tree/add_data_path_to_duckdb_catalog
- **Enhancement**: Enables specifying a local data directory for DuckLake storage
- **Benefit**: Improved data locality and performance for development/testing

## Resources

- [DuckLake Documentation](https://ducklake.select/)
- [SQLMesh Documentation](https://sqlmesh.com/)
- [Neon Documentation](https://neon.com/docs)
- [Custom SQLMesh Fork](https://github.com/mattiasthalen/sqlmesh/tree/add_data_path_to_duckdb_catalog)
