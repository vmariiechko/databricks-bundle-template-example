# Permissions Setup

Permissions were not included in this bundle configuration. This means:
- **No group variables** are defined in `variables.yml`
- **No `permissions:` blocks** exist on targets in `databricks.yml`
- **No schema `grants:`** are configured in any target

Access follows Databricks default object ownership and workspace access controls until you define explicit `permissions:` and schema `grants:`.

---

## Adding Permissions Later

If your team grows or compliance requirements change, you can add role-based access control:

### Step 1: Create Groups in Your Workspace

Create groups via **Settings → Identity and access → Groups** or CLI:
```bash
databricks groups create --display-name developers
databricks groups create --display-name qa_team
databricks groups create --display-name operations_team
databricks groups create --display-name analytics_team
```

### Step 2: Define Group Variables

Add to `variables.yml`:
```yaml
variables:
  developers_group:
    description: Developer group name
    default: "developers"
  qa_team_group:
    description: QA team group name
    default: "qa_team"
  operations_group:
    description: Operations team group name
    default: "operations_team"
  analytics_team_group:
    description: Analytics team group name
    default: "analytics_team"
```

### Step 3: Add Target Permissions

Add `permissions:` blocks to targets in `databricks.yml`:
```yaml
targets:
  stage:
    permissions:
      - level: CAN_VIEW
        group_name: ${var.developers_group}
      - level: CAN_RUN
        group_name: ${var.qa_team_group}
      - level: CAN_MANAGE
        service_principal_name: ${var.stage_service_principal}
```

### Step 4: Add Schema Grants

Add `grants:` to schema resource overrides per target in `databricks.yml`:
```yaml
targets:
  stage:
    resources:
      schemas:
        bronze_schema:
          name: ${var.schema_prefix}bronze
          catalog_name: ${var.catalog_name}
          grants:
            - principal: ${var.stage_service_principal}
              privileges:
                - ALL_PRIVILEGES
            - principal: ${var.developers_group}
              privileges:
                - USE_SCHEMA
                - SELECT
```

---

## SQL Grant Examples

For manual permission setup via SQL:
```sql
-- Grant schema access to a group
GRANT USE_SCHEMA ON SCHEMA catalog.bronze TO `developers`;
GRANT SELECT ON SCHEMA catalog.bronze TO `developers`;

-- Grant full access (typically for service principals)
GRANT ALL PRIVILEGES ON SCHEMA catalog.bronze TO `deploy_sp`;

-- View current grants
SHOW GRANTS ON SCHEMA catalog.bronze;
```

---

## Databricks Documentation

For detailed guidance on permissions and access control:

- [Unity Catalog Privileges](https://docs.databricks.com/aws/en/data-governance/unity-catalog/manage-privileges) — Managing schema and table grants
- [Managing Groups](https://docs.databricks.com/aws/en/admin/users-groups/manage-groups) — Creating and managing workspace groups
- [Bundle Permissions](https://docs.databricks.com/aws/en/dev-tools/bundles/permissions) — DABs resource-level RBAC configuration
- [Unity Catalog Best Practices](https://docs.databricks.com/aws/en/data-governance/unity-catalog/best-practices) — Security and governance patterns
