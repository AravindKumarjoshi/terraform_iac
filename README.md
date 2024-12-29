# terraform_iac

## Designing the CI section of the pipeline
The CI part of our pipeline contains five blocks:

 - Collect code from VCS.
 - Execute validation. We will run `terraform fmt` and `terraform validate` commands in this step.
 - Execute SAST test with tfsec.
 - Create plan using `terraform plan` command.
 - Store plan as artifact in artifact storage.
```
 - name: Prepare environment
  run: |
    terraform init -backend-config="environments/preprod_backend.hcl"
- name: Create Terraform plan for Preprod
  run: terraform plan -var-file="environments/preprod.tfvars" -out ${{env.preprod_artifact_name }}
  ```
    

Note:
we see three external elements in our pipeline:
  - We have to configure trigger from VCS.
  - We have to download and install `tfsec`.
  - We have to configure and manage artifact storage.
  - Terraform state file  is already needed in this phase as we execute `terraform plan`
  - To deal/manage in `CI` with `Terraform state` file, we need to add required resource to manage the state file.
    Eg:  these may require another Terraform template for creation and management


## Designing the CD section of the pipeline
  Our CD will also contain five stages:
 - Collect plan from artifact storage.
```
- name: Prepare environment
  run: |
    terraform init -backend-config="environments/preprod_backend.hcl"
- name: Download artifact for deployment
  uses: actions/download-artifact@v3
  with:
    name: preprod-execution-plan
- name: Execute terraform apply
  run: terraform apply ${{ env.preprod_artifact_name }}
```
 - Execute against “preprod” environment.
 - Test if deployment was successful.
 - Execute against “prod” environment.
 - Test if deployment was successful.

Note: 
 - Here we can see three external components. Artifact storage, which we already mentioned, and two environments.
 - possibly two different AWS accounts / Azure subscription.

## Pipeline using GitHub Action
 Action : hashicorp/setup-terraform@v2

 ```
  - name: Setup Terraform
  uses: hashicorp/setup-terraform@v2
  with:
     terraform_version: 1.5.4
```

