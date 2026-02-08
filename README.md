# GitLab Groups Terraform Module

<img src="https://github.com/user-attachments/assets/a09f3eff-3e08-4a3c-b490-71117a40080b" width="380" height="80"/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<img width="380" height="90" alt="image" src="https://github.com/user-attachments/assets/4ac93e8c-4848-472e-ae13-2bd858811b48" />

A comprehensive Terraform module for creating and managing GitLab groups with full support for security settings, access controls, and organizational features.

## Features

- **Complete GitLab Group Management** - Full support for all `gitlab_group` resource features
- **Group Membership Management** - Add users to groups with different access levels (owner, maintainer, developer, reporter, guest)
- **Advanced Security Settings** - Two-factor authentication, IP restrictions, and push rules
- **Access Control** - Granular permissions for project/subgroup creation and membership
- **CI/CD Integration** - Shared runners configuration and minutes management
- **Organizational Structure** - Support for nested groups and inheritance
- **Communication Controls** - Email domain restrictions and notification settings
- **Branch Protection** - Default branch protection rules for all projects
- **Comprehensive Outputs** - All group information available for use in other resources

## Usage

### Basic Group

```hcl
module "basic_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "development-team"
  path        = "dev-team"
  description = "Development team group"
}
```

### Group with Members

```hcl
module "team_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "backend-team"
  path        = "backend-team"
  description = "Backend development team"
  
  # Add team members with different access levels
  owners      = [101, 102]           # User IDs for group owners
  maintainers = [201, 202, 203]      # User IDs for maintainers
  developers  = [301, 302, 303, 304] # User IDs for developers
  reporters   = [401]                # User IDs for reporters
  guests      = [501, 502]           # User IDs for guests
}
```

### Advanced Member Management

```hcl
module "project_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "project-alpha"
  path        = "project-alpha"
  description = "Project Alpha team"
  
  # Simple member lists
  owners     = [100]
  developers = [200, 201, 202]
  
  # Advanced member configuration with expiration dates
  members = {
    contractor_1 = {
      user_id      = 300
      access_level = "developer"
      expires_at   = "2024-12-31"
    }
    contractor_2 = {
      user_id      = 301
      access_level = "developer"
      expires_at   = "2024-12-31"
    }
    external_auditor = {
      user_id      = 400
      access_level = "reporter"
      expires_at   = "2024-06-30"
    }
  }
}
```

### Advanced Group with Security Features

```hcl
module "secure_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  # Basic Configuration
  name        = "security-team"
  path        = "sec-team"
  description = "Security team with enhanced controls"
  
  # Visibility and Access Control
  visibility_level               = "private"
  prevent_forking_outside_group = true
  membership_lock               = true
  
  # Security Settings
  require_two_factor_authentication = true
  two_factor_grace_period          = 24
  ip_restriction_ranges            = ["192.168.1.0/24", "10.0.0.0/8"]
  
  # Email Restrictions
  allowed_email_domains_list = ["company.com", "contractor.company.com"]
  
  # Push Rules
  push_rules = {
    prevent_secrets         = true
    commit_committer_check  = true
    member_check           = true
    author_email_regex     = "@(company|contractor)\\.com$"
    reject_unsigned_commits = true
  }
  
  # Default Branch Protection
  default_branch_protection_defaults = {
    allowed_to_push            = ["maintainer"]
    allow_force_push           = false
    allowed_to_merge           = ["maintainer"]
    developer_can_initial_push = false
  }
}
```

### Nested Subgroup

```hcl
# Parent group
module "engineering_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "Engineering"
  path        = "engineering"
  description = "Main engineering group"
  
  subgroup_creation_level = "maintainer"
  project_creation_level  = "developer"
}

# Child subgroup
module "backend_team" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "Backend Team"
  path        = "backend"
  description = "Backend development team"
  parent_id   = module.engineering_group.id
}
```

### Group with Shared Runners and CI/CD Limits

```hcl
module "cicd_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "CI/CD Projects"
  path        = "cicd-projects"
  description = "Group for CI/CD intensive projects"
  
  # Shared Runners Configuration
  shared_runners_setting              = "enabled"
  shared_runners_minutes_limit        = 10000
  extra_shared_runners_minutes_limit  = 5000
  
  # Enable Auto DevOps
  auto_devops_enabled = true
  
  # Large File Storage
  lfs_enabled = true
}
```

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.3.0 |
| gitlab | ~> 18.0 |

## Providers

| Name | Version |
|------|---------|
| gitlab | ~> 18.0 |

## Inputs

### Required

| Name | Description | Type |
|------|-------------|------|
| name | The name of the GitLab group | `string` |
| path | The path of the GitLab group (URL slug) | `string` |

### Optional

| Name | Description | Type | Default |
|------|-------------|------|---------|
| description | A description of the GitLab group | `string` | `null` |
| visibility_level | The visibility level of the group (private, internal, public) | `string` | `"private"` |
| parent_id | The ID of the parent group (creates nested group) | `number` | `null` |
| project_creation_level | Who can create projects (noone, owner, maintainer, developer, administrator) | `string` | `"maintainer"` |
| subgroup_creation_level | Who can create subgroups (owner, maintainer) | `string` | `"owner"` |
| owners | List of user IDs to add as owners | `list(number)` | `[]` |
| maintainers | List of user IDs to add as maintainers | `list(number)` | `[]` |
| developers | List of user IDs to add as developers | `list(number)` | `[]` |
| reporters | List of user IDs to add as reporters | `list(number)` | `[]` |
| guests | List of user IDs to add as guests | `list(number)` | `[]` |
| members | Map of users with custom access levels and optional expiration dates | `map(object)` | `{}` |
| require_two_factor_authentication | Require 2FA for all group members | `bool` | `false` |
| two_factor_grace_period | 2FA enforcement grace period (hours) | `number` | `48` |
| lfs_enabled | Enable Large File Storage for projects | `bool` | `true` |
| shared_runners_setting | Shared runners configuration | `string` | `null` |
| ip_restriction_ranges | IP addresses/subnets for access restriction | `list(string)` | `[]` |
| allowed_email_domains_list | Allowed email domains for group access | `list(string)` | `[]` |

<details>
<summary>View all input variables</summary>

For a complete list of all input variables, see the [variables.tf](./variables.tf) file. The module supports all GitLab group features including:

- **Membership Management**: owners, maintainers, developers, reporters, guests, members (with expiration)
- **Access Control**: membership_lock, share_with_group_lock, prevent_forking_outside_group
- **Features**: auto_devops_enabled, wiki_access_level, emails_enabled, mentions_disabled
- **CI/CD**: shared_runners_minutes_limit, extra_shared_runners_minutes_limit
- **Avatar**: avatar, avatar_hash
- **Advanced Security**: push_rules object with comprehensive commit validation
- **Branch Protection**: default_branch_protection_defaults object

</details>

## Outputs

### Basic Outputs

| Name | Description |
|------|-------------|
| id | The ID of the GitLab group |
| name | The name of the GitLab group |
| path | The path of the GitLab group |
| full_name | The full name of the GitLab group |
| full_path | The full path of the GitLab group |
| web_url | The web URL of the GitLab group |
| runners_token | Group registration token for runners (sensitive) |

### Structured Outputs

| Name | Description |
|------|-------------|
| group_info | Comprehensive group information object |
| access_settings | Access control and permission settings |
| security_settings | Security configuration |
| cicd_settings | CI/CD and shared runners configuration |
| feature_settings | Feature and integration settings |

### Membership Outputs

| Name | Description |
|------|-------------|
| members | All group members with their access levels |
| member_count | Total number of members added to the group |
| member_ids | List of all user IDs that are members of the group |

### Convenience Outputs

| Name | Description |
|------|-------------|
| namespace_id | Group ID for use as namespace_id (alias for id) |
| is_top_level_group | Boolean indicating if this is a top-level group |

<details>
<summary>View all outputs</summary>

For a complete list of all outputs, see the [outputs.tf](./outputs.tf) file.

</details>

## Examples

For comprehensive usage examples including enterprise setups, nested subgroups, and specialized configurations, see the [examples directory](https://github.com/sudo-terraform-modules/terraform-gitlab-groups/tree/main/examples).

The examples include:

- **Complete Configuration** (`main.tf`) - Full organizational structure with DevOps teams, SAP subgroups, and security configurations
- **Variables and Documentation** - Complete variable examples and setup instructions

### Creating a Project in the Group

```hcl
# Create the group
module "my_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "my-team"
  path        = "my-team"
  description = "My development team"
}

# Create a project in the group
resource "gitlab_project" "my_project" {
  name         = "awesome-app"
  description  = "Our awesome application"
  namespace_id = module.my_group.namespace_id
  
  visibility_level = module.my_group.visibility_level
}
```

### Enterprise Group with Full Security

```hcl
module "enterprise_group" {
  source  = "sudo-terraform-modules/groups/gitlab"
  version = "0.2.0"

  name        = "Enterprise Applications"
  path        = "enterprise-apps"
  description = "Enterprise-grade applications with full security"
  
  # Maximum security settings
  visibility_level                  = "private"
  require_two_factor_authentication = true
  two_factor_grace_period          = 0  # Immediate enforcement
  membership_lock                  = true
  share_with_group_lock           = true
  prevent_forking_outside_group   = true
  
  # Strict access controls
  project_creation_level  = "owner"
  subgroup_creation_level = "owner"
  
  # Corporate email requirements
  allowed_email_domains_list = ["company.com"]
  
  # Comprehensive push rules
  push_rules = {
    prevent_secrets              = true
    commit_committer_check      = true
    commit_committer_name_check = true
    member_check                = true
    reject_unsigned_commits     = true
    author_email_regex          = "@company\\.com$"
    commit_message_regex        = "^(feat|fix|docs|style|refactor|test|chore):"
    max_file_size              = 100  # 100 MB limit
    deny_delete_tag            = true
  }
  
  # Strict branch protection
  default_branch_protection_defaults = {
    allowed_to_push            = ["no one"]
    allow_force_push           = false
    allowed_to_merge           = ["maintainer"]
    developer_can_initial_push = false
  }
  
  # Restricted runners
  shared_runners_setting = "disabled_and_unoverridable"
}
```

## Module Structure

```text
├── .github/
│   └── workflows/          # CI/CD pipeline configurations
├── .gitignore              # Git ignore patterns
├── LICENSE                 # Module license
├── README.md               # Module documentation
├── examples/               # Usage examples and configurations
│   ├── main.tf             # Complete example with all features
│   ├── variables.tf        # Example variable definitions
│   └── terraform.tfvars    # Example variable values
├── main.tf                 # GitLab group resource configuration
├── outputs.tf              # Output value definitions
├── variables.tf            # Input variable definitions with validation
└── versions.tf             # Terraform and provider version constraints
```

## Validation and Best Practices

The module includes comprehensive input validation:

- **Path Validation**: Ensures group path contains only valid characters
- **Access Level Validation**: Validates all access levels against GitLab's allowed values
- **Email Regex Validation**: Ensures push rule email patterns are valid
- **Numeric Constraints**: Validates time periods, file sizes, and limits
- **Conditional Validation**: Context-aware validation based on other variables

## Contributing

We welcome contributions! Please see our [Contributing Guide](.github/contributing.md) for details on how to contribute to this module, including development setup, testing guidelines, and submission requirements.

## Security Considerations

When using this module in production:

1. **Review Default Values**: Ensure default settings align with your security requirements
2. **Enable 2FA**: Consider enabling two-factor authentication for sensitive groups
3. **IP Restrictions**: Use IP restrictions for highly sensitive groups
4. **Push Rules**: Implement appropriate push rules for code quality and security
5. **Access Levels**: Use principle of least privilege for creation permissions
6. **Email Domains**: Restrict access to corporate email domains

## License

This module is licensed under the Apache 2.0 License. See [LICENSE](./LICENSE) file for details.

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history and changes.

## Maintainer

This module is maintained by [Amaan Ul Haq Siddiqui](https://github.com/amaanx86) and [Hameedullah Khan](https://github.com/hameedullah).

---

**Note**: This module requires GitLab 13.0+ for full feature compatibility. Some advanced features may require GitLab Premium or Ultimate subscriptions.
