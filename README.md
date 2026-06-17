# Export and Import AAP Assets (AAP 2.4 → AAP 2.6)

## Overview

This repository provides a migration workflow for exporting configuration assets from **Ansible Automation Platform (AAP) 2.4** and importing them into **AAP 2.6**.

The migration process is intentionally divided into multiple stages to ensure credentials are handled securely and to minimize import failures caused by missing secrets.

The workflow consists of:

1. Export assets from AAP 2.4
2. Import credential types and credentials into AAP 2.6
3. Manually update credential secrets in AAP 2.6
4. Import the remaining configuration assets into AAP 2.6

---

## Prerequisites

### Source Environment (AAP 2.4)

* AAP 2.4 Controller URL
* Controller administrator credentials
* Access to create temporary API tokens

### Target Environment (AAP 2.6)

* AAP 2.6 Gateway/Controller URL
* Administrator credentials

### GitHub

The export process stores exported assets in a Git repository.

Required:

* GitHub Personal Access Token (PAT)
* Access to clone and push to the repository

---

## Repository Structure

```text
.
├── playbooks
│   ├── export-configs.yml
│   ├── import-cred.yml
│   └── import-confige.yml
├── roles
├── var_files
│   └── platform-info.yml
├── ansible.cfg
└── README.md
```

---

# Step 1 - Export Assets from AAP 2.4

The export playbook performs the following actions:

* Creates a temporary Controller API token
* Exports AAP 2.4 assets
* Normalizes exported data
* Organizes assets into a Configuration-as-Code structure
* Commits and pushes the exported assets to GitHub

Run:

```bash
ansible-playbook playbooks/export-configs.yml
```

### Required Environment Variables

```bash
export CONTROLLER_HOST=https://<AAP24_CONTROLLER>
export CONTROLLER_USERNAME=admin
export CONTROLLER_PASSWORD='<password>'
export GITHUB_TOKEN='<github_pat>'
```

### Exported Content

The export process captures assets such as:

* Organizations
* Teams
* Users
* Credential Types
* Credentials
* Inventories
* Hosts
* Groups
* Inventory Sources
* Projects
* Execution Environments
* Job Templates
* Workflow Job Templates
* Notification Templates
* Schedules
* Applications
* Roles and Permissions

> **Note:** Credential secret values are never exported.

---

# Step 2 - Import Credentials into AAP 2.6

Credentials must be imported before any other assets.

Run:

```bash
ansible-playbook playbooks/import-cred.yml
```

This playbook imports:

* Credential Types
* Credentials

At this stage, credential objects are created but their secret values are intentionally empty.

---

# Step 3 - Update Credential Secrets

After importing credentials, manually update all required secrets through the AAP 2.6 UI or API.

Examples include:

* Passwords
* API Tokens
* Vault Passwords
* SSH Private Keys
* Cloud Provider Secrets
* Automation Hub Tokens

This step is required before importing the remaining assets.

Failure to update credential secrets may result in:

* Project synchronization failures
* Inventory synchronization failures
* Job Template failures
* Workflow execution failures

---

# Step 4 - Import Remaining Assets

After credential secrets have been updated, import the remaining configuration.

Run:

```bash
ansible-playbook playbooks/import-confige.yml
```

This playbook imports:

* Organizations
* Teams
* Users
* Inventories
* Hosts
* Groups
* Inventory Sources
* Projects
* Execution Environments
* Job Templates
* Workflow Job Templates
* Notification Templates
* Schedules
* Applications
* Roles and Permissions

---

## Configuration

The import playbooks use configuration values defined in:

```text
var_files/platform-info.yml
```

Update this file with the connection information for the target AAP 2.6 environment before running the import playbooks.

Example:

```yaml
platform_host: https://aap26.example.com
platform_username: admin
platform_password: password
validate_certs: false
```

---

## Recommended Migration Order

Run the migration in the following order:

```bash
# Export from AAP 2.4
ansible-playbook playbooks/export-configs.yml

# Import credential types and credentials
ansible-playbook playbooks/import-cred.yml

# Manually update credential secrets in AAP 2.6

# Import remaining assets
ansible-playbook playbooks/import-confige.yml
```

---

## Credential Handling

For security reasons, AAP does not export credential secrets.

The migration process therefore:

1. Creates credential objects
2. Requires administrators to update secrets manually
3. Imports dependent assets only after credentials are functional

This approach ensures that sensitive information is never stored in Git repositories or exported configuration files.

---

## Notes

* Review exported assets before importing into production.
* Test the migration in a non-production environment first.
* Verify project synchronization after credential updates.
* Verify inventory source synchronization before running production workloads.
* Review imported job templates and workflow job templates for environment-specific settings.

---

## Disclaimer

This repository is intended to assist with migrating configuration assets between AAP environments. Review all exported and imported content to ensure it aligns with your organization's security, compliance, and operational requirements before use in production.
