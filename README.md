# validate-urdf-action

**Validate ROS URDF files in CI/CD using the [RoboInfra API](https://roboinfra-dashboard.azurewebsites.net).**

Automatically catches URDF structural errors on every pull request — before they reach your simulation or hardware. Powered by 9 structural checks including joint validity, kinematic chain integrity, and duplicate detection.

---

## Quick Start

```yaml
# .github/workflows/validate-urdf.yml

name: Validate URDF

on: [push, pull_request]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: roboinfra/validate-urdf-action@v1
        with:
          api-key: ${{ secrets.ROBOINFRA_API_KEY }}
          file: urdf/robot.urdf
```

That's it. Add your API key as a GitHub secret and every push or PR runs URDF validation automatically.

---

## Get an API Key

1. Go to [roboinfra-dashboard.azurewebsites.net](https://roboinfra-dashboard.azurewebsites.net)
2. Register for a free account (50 validations/month, no credit card)
3. Go to **API Keys** → **Create Key**
4. In your GitHub repo: **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `ROBOINFRA_API_KEY`
   - Value: your `rk_...` key

---

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `api-key` | ✅ Yes | — | Your RoboInfra API key. Always use `${{ secrets.ROBOINFRA_API_KEY }}` — never hardcode. |
| `file` | ✅ Yes | — | Path to URDF file relative to repo root. Example: `urdf/robot.urdf` |
| `fail-on-invalid` | No | `true` | Set to `false` to print errors without failing the workflow. Useful for reporting-only mode. |
| `analyze` | No | `false` | Set to `true` to also run kinematic analysis (DOF, joint chain, end effectors). Requires Basic or Pro plan. |

---

## Outputs

| Output | Description |
|--------|-------------|
| `is-valid` | `true` or `false` |
| `errors` | Comma-separated list of validation errors. Empty string if valid. |
| `dof` | Degrees of freedom (only set when `analyze: true` and plan allows it). |

### Using outputs in later steps

```yaml
- uses: roboinfra/validate-urdf-action@v1
  id: urdf-check
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/robot.urdf
    analyze: true

- name: Print results
  run: |
    echo "Valid: ${{ steps.urdf-check.outputs.is-valid }}"
    echo "DOF: ${{ steps.urdf-check.outputs.dof }}"
    echo "Errors: ${{ steps.urdf-check.outputs.errors }}"
```

---

## Examples

### Basic — fail PR if URDF is invalid

```yaml
- uses: actions/checkout@v4

- uses: roboinfra/validate-urdf-action@v1
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/robot.urdf
```

### With kinematic analysis (Basic/Pro plan)

```yaml
- uses: roboinfra/validate-urdf-action@v1
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/robot.urdf
    analyze: true
```

### Report-only mode (never fails, just prints)

```yaml
- uses: roboinfra/validate-urdf-action@v1
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/robot.urdf
    fail-on-invalid: false
```

### Multiple URDF files in one workflow

```yaml
- uses: roboinfra/validate-urdf-action@v1
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/arm.urdf

- uses: roboinfra/validate-urdf-action@v1
  with:
    api-key: ${{ secrets.ROBOINFRA_API_KEY }}
    file: urdf/gripper.urdf
```

---

## What Gets Checked

9 structural checks run on every validation:

1. Root element must be `<robot>`
2. At least one `<link>` must exist
3. No duplicate link names
4. No duplicate joint names
5. All joint `parent` links must reference a defined link
6. All joint `child` links must reference a defined link
7. Joint `type` must be valid (`revolute`, `continuous`, `prismatic`, `fixed`, `floating`, `planar`)
8. `revolute` and `prismatic` joints must include a `<limit>` element
9. Exactly one root link — no cycles, no orphaned chains

---

## Plans

| Plan | Price | Validations/month | Kinematic Analysis |
|------|-------|------------------|--------------------|
| Free | $0 | 50 | — |
| Basic | $25/month | 500 | ✓ |
| Pro | $75/month | 5,000 | ✓ |

Get your key at [roboinfra-dashboard.azurewebsites.net](https://roboinfra-dashboard.azurewebsites.net)

---

## License

MIT