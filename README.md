## Best Practices
- `terraform plan` before doing `terraform apply`
- Announce in chat with @here in DevOps channel before running `terraform apply` to let your team know about the change, Example: 
> @here running `terraform apply` in Production account to upgrade EKS cluster version. More info on this PR [pr-123](https://github.com/company/pr-123)
- Use a separate repository for private terraform modules so that we can version control the changes using git tags.
- Always check terraform public module before creating new one, most public module are well written and tested ( dont re-invent the wheel).
- Use public terraform module from [https://github.com/terraform-aws-modules](https://github.com/terraform-aws-modules)
- Avoid storing secrets while creating a resource, for example AWS RDS DB Password, instead create a temporary password and change it after resource is created, terraform state will store secrets plain text!
- Limit access to terraform state bucket access, encrypt it and enable versioning.
- Run `terraform fmt` before submitting Pull Requests.

## Terraform tools that makes your life easier:


## #1 tfenv

[tfenv](https://github.com/tfutils/tfenv) is Terraform version manager similar to [rbenv](https://github.com/rbenv/rbenv)

### Install tfenv

```
brew install tfenv
```
### Usage Example

```
# List tf remote versions
tfenv list-remote

# Install tf version
tfenv install 0.11.15

# Use tf version
tfenv use 0.11.15
```
### tfenv automatic switching

Add `.terraform-version` file to automatically switch between different terrafom version and control different version between different accounts. 

**Example:**

```
├── .
├── aws_prodution_account
    ├── resource_1
    │   ├── main.tf
    |   ├── variables.tf
    |   └── ...
    ├── resource_2
    │   ├── main.tf
    |   ├── variables.tf
    |   └── ...
    ├── .terraform-version
├── aws_staging_account
    ├── resource_1
    │   ├── main.tf
    |   ├── variables.tf
    |   └── ...
    ├── resource_2
    │   ├── main.tf
    |   ├── variables.tf
    |   └── ...
    ├── .terraform-version
├── README.md
└── ...
```

## #2 aws-vault

> Although it's not exactly specific for terraform but aws-vault helps alot when you have multiple aws accounts

[aws-vault](https://github.com/99designs/aws-vault) stores IAM credentials in your Os's secure keystor, generates temporary credentials to be used in shell.

Using [aws-vault](https://github.com/99designs/aws-vault) with terraform to easily switch between aws accounts and avoid hardcoding aws profile in terraform backend state code.
### Install aws-vault

```
brew install --cask aws-vault
```

### Usage Example

```
# Run simple aws command
aws-vault exec aws_example_account -- aws s3 ls

# Login to aws console using temporary credentials
aws-vault login aws_example_account

# Use tf version
aws-vault exec aws_example_account -- terraform apply
```

## #3 terraform pre-commit

Using [pre-commit framework](http://pre-commit.com/) with terraform, will help your code to be keeped clean, formated, updated document and checked for tf security issues  (optional with [tfsec](https://github.com/tfsec/tfsec) before commiting and pushing the code with git.

### Install precommit and related tools

```
brew install pre-commit gawk terraform-docs tflint coreutils checkov terrascan
```

### Install the pre-commit hook globally

```
DIR=~/.git-template
git config --global init.templateDir ${DIR}
pre-commit init-templatedir -t pre-commit ${DIR}
```
### Initialize git repo with terraform hooks

```
cd your_terraform_git_repo
git init # if new repo
cat <<EOF > .pre-commit-config.yaml
repos:
- repo: git://github.com/antonbabenko/pre-commit-terraform
  rev: <VERSION> # Get the latest from: https://github.com/antonbabenko/pre-commit-terraform/releases
  hooks:
    - id: terraform_fmt
    - id: terraform_docs
EOF
pre-commit install
# Test pre commit
pre-commit run --all-files
```
> Now, whenever you run `git commit` on terraform repo, pre-commit will run the hooks

### Auto generate docs using [terraform-docs](https://github.com/terraform-docs/terraform-docs) for your terraform modules with pre-commit

```
cd terraform_example_module

cat <<EOF > README.md
<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
lines here, will be replaced by terraform_docs when pre commit runs
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
EOF
pre-commit run --all-files
```

## #4 tfsec

Want static analysis for your terraform code to help spot potential security issues? then all you need is [tfsec](https://github.com/tfsec/tfsec)

### Install tfsec
```
brew install tfsec
```
### Usage Example

```
cd terraform_folder
tfsec .
```

### Add tfsec to you pre-commit config
Add `terraform_tfsec` hook to `.pre-commit-config.yaml`

***Example***

```
repos:
- repo: git://github.com/antonbabenko/pre-commit-terraform
  hooks:
    - ...
    - id: terraform_tfsec
```

### Ignoring some tfsec rules

You may wish to ignore some warnings from tfsec. you can simply add a comment containing tfsec:ignore:<RULE> to the offending line in your templates.

***For example, to ignore an open security group rule:***

```
resource "aws_security_group_rule" "my-rule" {
    type = "ingress"
    #tfsec:ignore:AWS006
    cidr_blocks = ["0.0.0.0/0"]
}
```
