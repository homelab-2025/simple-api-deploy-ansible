# Simple-API-Deploy-Ansible

This a is repo to deploy simple-api using ansible

Here is how to apply simple-api-deploy-ansible on AWX with AWX CLI

## 1. Install AWX CLI

```bash
pip install awxkit
```

## 2. Login to AWX

Set your AWX host URL:

```bash
export TOWER_HOST=http://<URL>:<PORT>/
```

Login and save your token:

```bash
awx login -k --conf.host $TOWER_HOST --conf.username admin --conf.password <your_admin_password> -f human
```

> Replace `<your_admin_password>` with your actual AWX admin password and execute the output command.

## 3. Create Organization

```bash
awx organizations create --name "MyOrganization"
```

## 4. Create Credentials

```bash
awx credentials create \
  --name "SSH Credentials" \
  --credential_type "Machine" \
  --inputs '{"username": "user", "password": "user", "become_method": "sudo", "become_password": "user"}' \
  --organization "MyOrganization"
```

> Replace the username and password as needed for your target hosts.

## 5. Create Project

```bash
awx projects create \
  --name "simple-api-project" \
  --scm_type "git" \
  --scm_url "https://github.com/homelab-2025/simple-api-deploy-ansible.git" \
  --scm_branch "main" \
  --organization "MyOrganization"
```

## 6. Create Inventories

```bash
awx inventories create --name "prod_inventory" --organization "MyOrganization"
awx inventories create --name "qual_inventory" --organization "MyOrganization"
```

## 7. Add Inventory Sources

```bash
awx inventory_sources create \
  --name "Source from project for prod" \
  --inventory "prod_inventory" \
  --source scm \
  --source_project "simple-api-project" \
  --source_path "inventories/prd/hosts.yml" \
  --update_on_launch true

awx inventory_sources create \
  --name "Source from project for qual" \
  --inventory "qual_inventory" \
  --source scm \
  --source_project "simple-api-project" \
  --source_path "inventories/qua/hosts.yml" \
  --update_on_launch true
```

Update inventory sources to pull latest hosts:

```bash
awx inventory_sources update --name "Source from project for prod"
awx inventory_sources update --name "Source from project for qual"
```

## 8. Create Job Templates

```bash
awx job_templates create \
  --name "simple-api-prod-template" \
  --project "simple-api-project" \
  --inventory "prod_inventory" \
  --playbook "playbook.yml" \
  --ask_credential_on_launch true \
  --ask_limit_on_launch true

awx job_templates create \
  --name "simple-api-qual-template" \
  --project "simple-api-project" \
  --inventory "qual_inventory" \
  --playbook "playbook.yml" \
  --ask_credential_on_launch true \
  --ask_limit_on_launch true
```

## 9. Launch a Job

```bash
awx job_templates launch "simple-api-prod-template" \
  --monitor \
  --limit all \
  --verbosity 2 \
  --credentials "SSH Credentials"
```

Or for qualification:

```bash
awx job_templates launch "simple-api-qual-template" \
  --monitor \
  --limit all \
  --verbosity 2 \
  --credentials "SSH Credentials"
```
