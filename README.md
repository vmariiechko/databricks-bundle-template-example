# Databricks Bundle Template Example

A Databricks Asset Bundle for multi-environment deployment with user, stage, and prod environments.

## Template Source

This project was generated from the Databricks Asset Bundles template:<br>
https://github.com/vmariiechko/databricks-bundle-template

For template updates, fixes, and release notes, check the template repository.

## Getting Started

**New to this project?** Follow the [QUICKSTART.md](QUICKSTART.md) for step-by-step deployment instructions.

### Prerequisites

Before deploying, ensure you have:

1. **Databricks CLI** minimum version v0.274.0 installed ([reference docs](https://docs.databricks.com/aws/en/dev-tools/cli/install))
   ```bash
   pip install databricks-cli
   databricks --version
   ```

2. **Unity Catalog** with these pre-existing catalogs (created by your platform/infra team):
   - `dev_analytics` (development - shared by `user` and targets)
   - `stage_analytics` (pre-production)
   - `prod_analytics` (production)

### Quick Deploy

```bash
# Validate configuration
databricks bundle validate -t user

# Deploy to your personal environment
databricks bundle deploy -t user

# Run the sample job
databricks bundle run my_data_project_ingestion -t user
```

## Project Structure

```
databricks-bundle-template-example/
├── databricks.yml              # Bundle configuration
├── variables.yml               # Shared variables (catalogs, SPs)
├── resources/
│   ├── my_data_project_ingestion.job.yml       # ETL ingestion job
│   ├── my_data_project_pipeline.pipeline.yml   # LDP pipeline
│   ├── my_data_project_pipeline_trigger.job.yml
│   └── schemas.yml             # Unity Catalog schemas
├── src/
│   ├── jobs/                   # Job Python scripts
│   └── pipelines/              # LDP notebook code
├── tests/                      # Unit tests (run by CI pipeline)
├── templates/                  # Cluster config examples
├── .github/workflows/          # GitHub Actions workflows
├── bundle_init_config.json     # Template config used during generation
└── docs/                       # Setup guides
```

## Environments

| Target | Purpose | Catalog | Schema Isolation |
|--------|---------|---------|------------------|
| `user` | Personal development | `dev_analytics` | `<username>_bronze/silver/gold` |
| `stage` | Pre-production testing | `stage_analytics` | `bronze/silver/gold` |
| `prod` | Production | `prod_analytics` | `bronze/silver/gold` |

### Deployment Commands

```bash
databricks bundle validate -t <target>   # Validate
databricks bundle deploy -t <target>     # Deploy
databricks bundle run <job> -t <target>  # Run job
databricks bundle destroy -t <target>    # Cleanup
```

## Configuration

### Key Files

| File | Purpose |
|------|---------|
| `databricks.yml` | Targets, permissions, resource includes |
| `variables.yml` | Catalogs, service principals |
| `resources/*.yml` | Job and pipeline definitions |

### Compute
This bundle uses **classic clusters**. Cluster configurations are in resource files.
See `templates/cluster_configs.yml` for configuration examples.

### Multi-Workspace Setup

By default, all environments deploy to the same workspace using Unity Catalog for isolation.

**For a separate production workspace**, update `databricks.yml`:

```yaml
targets:
  prod:
    workspace:
      host: https://your-prod-workspace.azuredatabricks.net
```

### Service Principals

Service principals are **only required for CI/CD deployments** (stage, prod targets).

The `user` target runs as your personal identity and works immediately without SP configuration.

**To configure SPs for CI/CD:**

1. Create service principals in your Databricks workspace
2. Search for `SP_PLACEHOLDER` in `variables.yml`
3. Replace with your service principal application IDs
4. Grant Unity Catalog permissions (`USE CATALOG`, `CREATE SCHEMA`)

> **Important**: Service principals need Unity Catalog permissions before CI/CD can deploy. See [docs/CI_CD_SETUP.md](docs/CI_CD_SETUP.md#unity-catalog-prerequisites) for detailed setup instructions.

## Customization

### Replace Sample Code

The `src/` directory contains sample code demonstrating the medallion architecture pattern. Replace with your business logic:

- `src/jobs/ingest_to_raw.py` - Data ingestion
- `src/jobs/transform_to_silver.py` - Transformations
- `src/pipelines/bronze.py` - LDP bronze layer
- `src/pipelines/silver.py` - LDP silver layer

### Add New Resources

Create new job files in `resources/`:

```yaml
# resources/my_job.job.yml
resources:
  jobs:
    my_job:
      name: "${bundle.target} My Job"
      tasks:
        - task_key: main
          job_cluster_key: job_cluster
          spark_python_task:
            python_file: ../src/jobs/my_script.py
```

## Troubleshooting

### "Catalog not found"

Catalogs must be pre-existing (created by a metastore admin or platform team).
Verify that the required catalogs exist and you have `USE CATALOG` permission:
```sql
SHOW CATALOGS;
-- Required: dev_analytics, stage_analytics, prod_analytics
```

### Service Principal Errors (CI/CD targets only)

If deploying to stage, or prod:
1. Ensure `SP_PLACEHOLDER` values in `variables.yml` are replaced
2. Verify the SP exists in your workspace
3. The `user` target does not require SP configuration

## CI/CD

This project includes pre-configured CI/CD pipelines for **GitHub Actions**.

| Pipeline Stage | Trigger | Action |
|---------------|---------|--------|
| Bundle CI | Pull Request to `main` | Runs unit tests and validates bundle |
| Staging CD | Merge to `main` | Deploys to staging |
| Production CD | Merge to `release` | Deploys to production |

**Setup required:** See [docs/CI_CD_SETUP.md](docs/CI_CD_SETUP.md) for configuration instructions.

## Testing

### Unit Tests

Run unit tests locally:

```bash
# Install development dependencies
pip install -r requirements_dev.txt

# Run tests
pytest tests/ -V
```

Unit tests are located in the `tests/` directory and run automatically in the CI pipeline.

### Integration Tests

For data quality validation, use SDP/LDP expectations in your pipeline code.
See the pipeline notebooks in `src/pipelines/` for examples.

## Resources

- [Unity Catalog](https://docs.databricks.com/aws/en/data-governance/unity-catalog)
- [Databricks Asset Bundles](https://docs.databricks.com/aws/en/dev-tools/bundles)
- [Lakeflow Declarative Pipelines](https://docs.databricks.com/aws/en/ldp/)
- [CI/CD on Databricks](https://docs.databricks.com/aws/en/dev-tools/ci-cd)
