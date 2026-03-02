# Quick Start Guide for my_data_project

## 1. Prerequisites Check

```bash
# Verify Databricks CLI is installed
databricks --version

# Authenticate to your workspace
databricks auth login
```

## 2. Validate Configuration

```bash
databricks bundle validate -t user
```

If validation fails, check:
- Unity Catalog `dev_analytics` exists and you have `USE CATALOG` permission

## 3. Deploy to User Environment

```bash
databricks bundle deploy -t user
```

## 4. Run Sample Workloads

```bash
# Run the ingestion job
databricks bundle run my_data_project_ingestion -t user

# Trigger the pipeline
databricks bundle run my_data_project_pipeline_trigger -t user
```

## 5. Verify in Workspace

Check your Databricks workspace:
- **Jobs**: Look for `[user <yourname>] my_data_project Ingestion Job`
- **Pipelines**: Look for `[user <yourname>] my_data_project ETL Pipeline`

## 6. Cleanup (Optional)

```bash
databricks bundle destroy -t user
```

## Next Steps

### Configure Service Principals (for CI/CD)

Before deploying to stage, or prod:

1. Create service principals in your Databricks workspace
2. Search for `SP_PLACEHOLDER` in `variables.yml`
3. Replace with your service principal application IDs

### Set Up CI/CD Pipeline

For automated deployment via CI/CD, see [docs/CI_CD_SETUP.md](docs/CI_CD_SETUP.md).

### Deploy to Higher Environments
```bash
databricks bundle deploy -t stage
databricks bundle deploy -t prod
```
> **Multi-Workspace Setup**: If using a separate prod workspace, update `workspace.host` in `databricks.yml`. See [README.md](README.md) for details.

## Troubleshooting

### "Catalog not found" Error

Catalogs must be pre-existing (created by a metastore admin or platform team).
Verify that the `dev_analytics` catalog exists and you have access:
```sql
SHOW CATALOGS;
```

### Service Principal Errors

> **Note**: The `user` target does not require service principals.

For stage, prod targets:
- Search for `SP_PLACEHOLDER` in `variables.yml` and replace with your SP IDs
- Ensure the SP exists in your workspace before deploying
