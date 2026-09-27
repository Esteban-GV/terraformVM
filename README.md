# VM Linux en Azure con Terraform

Este proyecto crea una máquina virtual Ubuntu en Azure y todos los recursos de red que necesita. Terraform describe el estado deseado y usa el proveedor de Azure para crear, consultar y eliminar esos recursos.

## Recursos creados

- Un grupo de recursos en `canadacentral`.
- Una red virtual y una subred.
- Una IP pública estática.
- Una interfaz de red conectada a la subred y a la IP pública.
- Una máquina virtual Ubuntu 22.04 (`Standard_B1s`).
- Un grupo de seguridad de red con acceso SSH por el puerto 22.

La regla SSH actual permite conexiones desde cualquier dirección IP. Para un entorno real, limita `source_address_prefix` en `main.tf` a tu IP pública con formato CIDR, por ejemplo `203.0.113.10/32`.

## Requisitos

- Una suscripción activa de Azure con permisos para crear recursos.
- Terraform 1.1.0 o posterior.
- Azure CLI, para iniciar sesión desde la terminal.

Inicia sesión y, si tienes más de una suscripción, selecciona la que usarás:

```bash
az login
az account set --subscription "ID_O_NOMBRE_DE_SUSCRIPCION"
```

## Archivos principales

- `main.tf`: proveedor, recursos, variables de usuario y contraseña, y salida de la IP pública.
- `.terraform.lock.hcl`: versiones verificadas del proveedor; se conserva en Git.
- `terraform.tfvars.example`: ejemplo de configuración de variables, sin una contraseña real.
- `terraform.tfvars`: valores locales; está excluido de Git.
- `.terraform/`: plugins descargados por Terraform; se genera localmente y está excluido de Git.
- `terraform.tfstate`: estado local de los recursos administrados; está excluido de Git.

## Variables

`main.tf` declara `admin_username` para el nombre de usuario administrador y `admin_password` para su contraseña. La contraseña está marcada como `sensitive = true`.

Para definirla localmente, copia el ejemplo y reemplaza su valor por una contraseña segura que cumpla los requisitos de Azure:

```bash
cp terraform.tfvars.example terraform.tfvars
```

Edita `terraform.tfvars`:

```hcl
admin_username = "admin_user"
admin_password = "REEMPLAZA_POR_UNA_CONTRASENA_SEGURA"
```

Terraform también puede pedir la variable al ejecutar `plan` o `apply` si no la encuentra. `sensitive = true` oculta el valor en parte de la salida de Terraform, pero no lo cifra dentro del estado. Protege `terraform.tfstate` y nunca publiques `terraform.tfvars` ni compartas la contraseña.

## Inicializar, revisar y crear

Ejecuta los siguientes comandos desde la carpeta del proyecto:

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

`init` descarga el proveedor y prepara el directorio de trabajo. `fmt` aplica el formato estándar de Terraform. `validate` comprueba la configuración. `plan` muestra los cambios propuestos sin aplicarlos. `apply` crea o actualiza los recursos, y pide confirmación antes de continuar.

Al terminar, consulta la IP pública y conéctate por SSH:

```bash
terraform output -raw public_ip_address
ssh admin_user@IP_PUBLICA
```

Usa la contraseña configurada en `terraform.tfvars`. En la primera conexión, SSH puede pedir confirmar la huella de la máquina remota; verifica que corresponde a tu VM antes de aceptarla.

## Cambios y destrucción

Después de modificar `main.tf`, vuelve a ejecutar `terraform plan` para revisar el efecto antes de aplicar con `terraform apply`.

Para eliminar los recursos administrados por este proyecto:

```bash
terraform plan -destroy
terraform destroy
```

Terraform elimina primero los recursos dependientes, como la VM y su interfaz de red, y deja para el final el grupo de recursos. Azure puede tardar un tiempo en completar la eliminación. No borres ni edites manualmente `terraform.tfstate` mientras Terraform esté trabajando: el estado es el registro que usa para relacionar la configuración con los recursos reales.

