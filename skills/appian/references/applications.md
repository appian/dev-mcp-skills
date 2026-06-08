# Applications

## CLI Commands

```bash
# Create an application
echo '{"name":"Case Management"}' | appian apps create
echo '{"name":"Case Management","prefix":"CM","description":"Manages support cases"}' | appian apps create

# List applications
appian apps list
appian apps list | jq '.[] | select(.name | contains("Case"))'

# Get application details
appian apps get <uuid>

# Delete an application (removes container only, objects preserved)
appian apps delete <uuid>
```

## Create JSON Schema

```json
{
  "name": "Case Management",
  "prefix": "CM",
  "description": "Optional description"
}
```

- `name` (required) — application display name
- `prefix` (optional) — naming prefix for objects (e.g., "CM"). Auto-generated from name if omitted.
- `description` (optional)

## What Creation Produces

Creating an application auto-generates:
- **PREFIX Administrators** group — developers and admins
- **PREFIX Users** group — all end users (includes Administrators as members)
- Default rule folder, document folder (Knowledge Center), and process model folder

Use `--format uuids` to capture the app UUID for subsequent commands:
```bash
APP=$(echo '{"name":"Case Management","prefix":"CM"}' | appian apps create --format uuids)
```

## Discovery After Creation

After creating an app, discover its auto-generated objects:
```bash
# Find the process model folder (needed for PM creation)
appian pm list --app $APP  # shows the PM folder context

# Find groups
appian apps get $APP | jq '.defaultObjects'
```

## Application Prefix Convention

The prefix propagates to all objects in the application:
- Record types: `CM Case`, `CM Customer` (prefix + Title Case singular; plural name has no prefix)
- Interfaces: `CM_Dashboard`, `CM_CaseForm`
- Expression rules: `CM_GetFullName`, `CM_IsEligible`
- Process models: `CM Create Case`, `CM Reassign Case`
- Constants: `CM_ADMIN_GROUP`, `CM_STATUS_OPEN`
- Groups: `CM Administrators`, `CM Case Managers`
- Sites: `CM Site` (internal), `Case Management` (display name)
