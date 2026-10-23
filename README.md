# sml-salesinsights-snowflake

AtScale SML catalog "Sales Insights - Snowflake": the AdventureWorks Internet Sales demo on Snowflake, with row security, security on measures, and MDX examples.

This repository holds the Snowflake version of the Sales Insights demo, written in SML (Semantic Modeling Language, the YAML format that AtScale uses to define a catalog as code). The shared customer, date, geography, gender, product, and inventory objects come from the package [sml-salesinsights-common](https://github.com/muellermarco/sml-salesinsights-common). The repository is connected to the [atscale-cicd-pipeline](https://github.com/muellermarco/atscale-cicd-pipeline).

## Technologies

- AtScale 2026.9.2
- sml-cli 2025.12.0 (pinned in the pipeline's `validate` workflow)
- SML catalog `version: 1.3` (see `catalog.yml`)
- Snowflake, through the AtScale connection `Snowflake Partner`
- GitHub Actions (reusable workflows from `muellermarco/atscale-cicd-pipeline`)

## Catalog at a glance

| Item | Value |
|---|---|
| Catalog | `Sales Insights - Snowflake` (`catalog.yml`) |
| Connection | AtScale connection `Snowflake Partner`. Database and schema: see `connections/Snowflake.yml`. |
| Package | `Common-Objects` from sml-salesinsights-common, branch `snowflake`, commit `db71a1d` (see `package.yml`) |

## Models

| Model | Type | Purpose |
|---|---|---|
| `Sales` | Model | Sales metrics by customer, date, order, and product |
| `Internet Sales` | Composite model | Combines `Sales` and the package model `Inventory` |
| `Sales RLS` | Model | `Sales` with the security dimensions and row security from the package |
| `Internet Sales RLS` | Composite model | Combines `Sales RLS` and the package model `Inventory RLS` |
| `RetailSecurityOnMeasures` | Model | Two fact datasets, row security on one of them |
| `MDX Examples` | Model | Calculated metrics that show MDX patterns |

## Demo scenarios

- **Row-level security (RLS)**: query `Sales RLS` or `Internet Sales RLS` as different users. Each user sees only the countries and product lines that the security tables allow.
- **Security on measures**: in `RetailSecurityOnMeasures`, row security `SECURITY_PRODUCTLINE_GROUP` applies only to `factinternetsales2`. The order quantity metric is filtered by product line; the sales amount metric is not.
- **MDX examples**: `MDX Examples` contains calculations such as `MDX AVG unrelated measures`, `MDX Customer count Multi Tuple`, `MDX Name Reference`, and `m_mdx_ignore_filter_dimension`.

## Repository contents

| Folder | Count | Contents |
|---|---|---|
| `connections/` | 1 | `Snowflake` |
| `datasets/` | 3 | `dimorder`, `factinternetsales`, `factinternetsales2` |
| `dimensions/` | 1 | `Order Dimension` |
| `metrics/` | 18 | Base metrics, for example sales amount, order quantity, customer count |
| `calculations/` | 144 | Calculated metrics, mainly time-relative (Retail 4-4-5 and custom 4-4-5 periods), plus the MDX examples |
| `models/` | 6 | See the table above |

The package adds more datasets, dimensions, models, and row security objects at install time.

## How the pipeline deploys this repo

| Event | Workflow | Result |
|---|---|---|
| Pull request | `validate` | sml-cli installs the package and validates the catalog. No deploy. |
| Merge to `main` | `deploy-qa` | The catalog deploys to the QA instance. |
| Tag `v*` | `release-live` | The catalog deploys to the live instance. |

QA and live deploy the catalog as `Sales Insights - Snowflake_main`.

## Change the model and release it

1. Create a branch from `main`.
2. Edit the SML files.
3. Install the package: `npx -y sml-cli@2025.12.0 install .`.
4. Validate: `npx -y sml-cli@2025.12.0 validate .`.
   Result: the output contains `Validation SUCCESSFUL`.
5. Open a pull request to `main`.
   Result: the `validate` check is green.
6. Merge the pull request.
   Result: `deploy-qa` deploys the catalog to the QA instance.
7. Push a release tag, for example `git tag v1.4 && git push origin v1.4`.
   Result: `release-live` deploys the catalog to the live instance.

## License

MIT License. See [LICENSE](LICENSE).
