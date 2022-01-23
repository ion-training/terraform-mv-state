# terraform-mv-state
rename a resource without deleting it from state using 'terraform state move' command

# How to use this repo
clone
```
git clone https://github.com/ion-training/terraform-mv-state.git
```
```
cd terraform-mv-state
```

# create resources defined in main.tf
initialize
```
terraform init
```

terraform plan and apply
```
terraform plan
```
```
terraform apply -auto-approve
```

Two resources created
- random_pet.name, generated a name
- null_resource.hello, takes the name and prints it on the screen


# Objective
From
```
$ tree
.
├── LICENSE
├── README.md
├── main.tf
└── terraform.tfstate

0 directories, 4 files

$ cat main.tf
resource "random_pet" "name" {
 length    = "4"
 separator = "-"
}

resource "null_resource" "hello" {
  provisioner "local-exec" {
    command = "echo Hello ${random_pet.name.id}"
  }
}

$
```

To this:\
_"terraform apply" should say nothing about `delete/create`._
```
$ tree
.
├── LICENSE
├── README.md
├── main.tf
├── name_generate
│   └── main.tf
├── terraform.tfstate
└── touch

1 directory, 6 files

$ cat main.tf
module "gen_a_name" {
    source = "./name_generate/"
}

resource "null_resource" "hello" {
  provisioner "local-exec" {
    command = "echo Hello ${module.gen_a_name.rand_name_output}"
  }
}

$ cat ./name_generate/main.tf 
resource "random_pet" "name" {
 length    = "4"
 separator = "-"
}

output "rand_name_output" {
  value = random_pet.name.id
}
```


# Steps: move resource random_pet.name in a module
_next "terraform apply" should say nothing about `delete/create`._\
Create name_generate dir and main.tf in it
```
mkdir name_generate
```
```
touch name_generate/main.tf
```
Move this the random_pet.name config from main.tf into ./name_generate/main.tf.\ 
Add also output to be referenced from toot module.
```
resource "random_pet" "name" {
 length    = "4"
 separator = "-"
}

output "rand_name_output" {
  value = random_pet.name.id
}
```
Create module resource in main.tf that references the module ./name_generate/
```
module "gen_a_name" {
    source = "./name_generate/"
}
```
View state resources
```
terraform state list
```

Move resource state random_pet.name to module
```
terraform state mv random_pet.name module.gen_a_name.random_pet.name
```

# Verify
```
terraform init
```
```
terraform plan
```
```
terraform apply
```

# Sample output
```
$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/random...
- Finding latest version of hashicorp/null...
- Installing hashicorp/random v3.1.0...
- Installed hashicorp/random v3.1.0 (signed by HashiCorp)
- Installing hashicorp/null v3.1.0...
- Installed hashicorp/null v3.1.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
$ terraform plan 

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.hello will be created
  + resource "null_resource" "hello" {
      + id = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 4
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform
apply" now.
$ terraform apply -auto-approve

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.hello will be created
  + resource "null_resource" "hello" {
      + id = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 4
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.
random_pet.name: Creating...
random_pet.name: Creation complete after 0s [id=positively-strangely-credible-pig]
null_resource.hello: Creating...
null_resource.hello: Provisioning with 'local-exec'...
null_resource.hello (local-exec): Executing: ["/bin/sh" "-c" "echo Hello positively-strangely-credible-pig"]
null_resource.hello (local-exec): Hello positively-strangely-credible-pig
null_resource.hello: Creation complete after 0s [id=4930653742174473605]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
[ion:~/Documents/dev/terraform-proj/terraform-mv-state] main(2)* ± mkdir name_generate
[ion:~/Documents/dev/terraform-proj/terraform-mv-state] main(+3/-4,2)* ± touch name_generate/main.tf
[ion:~/Documents/dev/terraform-proj/terraform-mv-state] main(+3/-4,2)* ± terraform state list
null_resource.hello
random_pet.name
$ terraform state mv random_pet.name module.gen_a_name.random_pet.name
Move "random_pet.name" to "module.gen_a_name.random_pet.name"
Successfully moved 1 object(s).
$ terraform init
Initializing modules...
- gen_a_name in name_generate

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/random from the dependency lock file
- Reusing previous version of hashicorp/null from the dependency lock file
- Using previously-installed hashicorp/random v3.1.0
- Using previously-installed hashicorp/null v3.1.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
[ion:~/Documents/dev/terraform-proj/terraform-mv-state] main(+3/-4,2)* ± terraform plan
module.gen_a_name.random_pet.name: Refreshing state... [id=positively-strangely-credible-pig]
null_resource.hello: Refreshing state... [id=4930653742174473605]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
[ion:~/Documents/dev/terraform-proj/terraform-mv-state] main(+3/-4,2)* ± terraform apply
module.gen_a_name.random_pet.name: Refreshing state... [id=positively-strangely-credible-pig]
null_resource.hello: Refreshing state... [id=4930653742174473605]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
$
```
