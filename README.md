# Linux VM on Azure with Terraform

This project creates an Ubuntu virtual machine on Azure along with all the network resources it needs. Terraform describes the desired state and uses the Azure provider to create, query, and delete those resources.

## Resources created

- A resource group in `canadacentral`.
- A virtual network and a subnet.
- A static public IP address.
- A network interface connected to the subnet and the public IP.
- An Ubuntu 22.04 virtual machine (`Standard_B1s`).
- A network security group allowing SSH access on port 22.

The current SSH rule allows connections from any IP address. For a real environment, restrict `source_address_prefix` in `main.tf` to your public IP in CIDR format, for example `203.0.113.10/32`.

## Requirements

- An active Azure subscription with permissions to create resources.
- Terraform 1.1.0 or later.
- Azure CLI, to sign in from the terminal.

Sign in and, if you have more than one subscription, select the one you will use:

```bash
az login
az account set --subscription "SUBSCRIPTION_ID_OR_NAME"
```

## Main files

- `main.tf`: provider, resources, username and password variables, and the public IP output.
- `.terraform.lock.hcl`: verified provider versions; keep it in Git.
- `terraform.tfvars.example`: example variable configuration, without a real password.
- `terraform.tfvars`: local values; excluded from Git.
- `.terraform/`: plugins downloaded by Terraform; generated locally and excluded from Git.
- `terraform.tfstate`: local state of the managed resources; excluded from Git.

## Variables

`main.tf` declares `admin_username` for the administrator username and `admin_password` for its password. The password is marked as `sensitive = true`.

To set it locally, copy the example file and replace its value with a strong password that meets Azure's requirements:

```bash
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars`:

```hcl
admin_username = "admin_user"
admin_password = "REPLACE_WITH_A_STRONG_PASSWORD"
```

Terraform can also prompt for the variable when running `plan` or `apply` if it can't find it. `sensitive = true` hides the value in parts of Terraform's output, but it does not encrypt it inside the state. Protect `terraform.tfstate` and never publish `terraform.tfvars` or share the password.

## Initialize, review, and create

Run the following commands from the project folder:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

`init` downloads the provider and prepares the working directory. `fmt` applies Terraform's standard formatting. `validate` checks the configuration. `plan` shows the proposed changes without applying them. `apply` creates or updates the resources, and asks for confirmation before continuing.

When it finishes, get the public IP and connect via SSH:

```bash
terraform output -raw public_ip_address
ssh admin_user@PUBLIC_IP
```

Use the password configured in `terraform.tfvars`. On the first connection, SSH may ask you to confirm the remote machine's fingerprint; verify that it matches your VM before accepting it.

## Changes and destruction

After modifying `main.tf`, run `terraform plan` again to review the effect before applying it with `terraform apply`.

To delete the resources managed by this project:

```bash
terraform plan -destroy
terraform destroy
```

Terraform first deletes dependent resources, such as the VM and its network interface, and leaves the resource group for last. Azure may take some time to complete the deletion. Do not manually delete or edit `terraform.tfstate` while Terraform is working: the state is the record it uses to map the configuration to the real resources.