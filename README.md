# Aiven Terraform Provider

The Terraform provider for [Aiven.io](https://aiven.io/), an open source data platform as a service.

**See the [official documentation](https://registry.terraform.io/providers/aiven/aiven/latest/docs) to learn about all the possible services and resources.**

## Quick start

- [Signup for Aiven](https://console.aiven.io/signup?utm_source=github&utm_medium=organic&utm_campaign=terraform&utm_content=signup)
- [Create your authentication token](https://aiven.io/docs/platform/howto/create_authentication_token)
- Create a file named `main.tf` with the content below:

```hcl
terraform {
  required_providers {
    aiven = {
      source  = "aiven/aiven"
      version = "x.y.z" # check out the latest version in the release section
    }
  }
}

provider "aiven" {
  api_token = "your-api-token"
}

resource "aiven_pg" "postgresql" {
  project                = "your-project-name"
  service_name           = "postgresql"
  cloud_name             = "google-europe-west3"
  plan                   = "startup-4"

  termination_protection = true
}

output "postgresql_service_uri" {
  value     = aiven_pg.postgresql.service_uri
  sensitive = true
}
```

- Run these commands in your terminal:

```bash
terraform init
terraform plan
terraform apply
psql "$(terraform output -raw postgresql_service_uri)"
```

Voilà, a PostgreSQL database.

## A word of caution

Recreating stateful services with Terraform will possibly **delete** the service and all its data before creating it again. Whenever the Terraform plan indicates that a service will be **deleted** or **replaced**, a catastrophic action is possibly about to happen.

Some properties, like **project** and the **resource name**, cannot be changed and it will trigger a resource replacement.

To avoid any issues, **please set the `termination_protection` property to `true` on all production services**, it will prevent Terraform to remove the service until the flag is set back to `false` again. While it prevents a service to be deleted, any logical databases, topics or other configurations may be removed **even when this section is enabled**. Be very careful!

## Policy Validation with OPA

We provide [Open Policy Agent (OPA)](https://www.openpolicyagent.org/) policy bundles to help validate your Terraform configurations and prevent common issues before deployment.

### Quick Start with Policies

1. **Download the latest policy bundle** from our [releases page](https://github.com/aiven/terraform-provider-aiven/releases):
   ```bash
   curl -LO https://github.com/aiven/terraform-provider-aiven/releases/latest/download/aiven-terraform-provider-policies-1.0.0.tar.gz
   ```

2. **Install Conftest**:
   ```bash
   brew install conftest  # macOS
   # or download from https://github.com/open-policy-agent/conftest/releases
   ```

3. **Validate your Terraform plan**:
   ```bash
   terraform plan -out=tfplan.out
   terraform show -json tfplan.out > tfplan.json
   conftest test --bundle aiven-terraform-provider-policies-1.0.0.tar.gz tfplan.json
   ```

### Available Policies

Our policy bundle helps prevent:
- **Duplicate resource conflicts** - Prevents creating multiple resources that target the same entity
- **Autoscaler integration conflicts** - Avoids issues when removing autoscalers while modifying services
- **ClickHouse grant duplicates** - Ensures only one grant resource per role/user combination

### CI/CD Integration

See our [detailed usage guide](opa/policies/USAGE.md) for examples of integrating these policies into:
- GitHub Actions
- GitLab CI
- Azure DevOps
- And combining with your own custom policies

## Contributing

Bug reports and patches are very welcome, please post them as GitHub issues and pull requests at https://github.com/aiven/terraform-provider-aiven. Please review the guides below.

- [Contributing guidelines](CONTRIBUTING.md)
- [Code of conduct](CODE_OF_CONDUCT.md)

Please see our [security](SECURITY.md) policy to report any possible vulnerabilities or serious issues.

## License

terraform-provider-aiven is licensed under the MIT license. Full license text is available in the [LICENSE](LICENSE) file. Please note that the project explicitly does not require a CLA (Contributor License Agreement) from its contributors.

## Credits

The original version of the Aiven Terraform provider was written and maintained by [Jelmer Snoeck](https://github.com/jelmersnoeck).
