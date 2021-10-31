# Load Balancer GKE Terrafrom
cloud shell > gcloud auth list
update terraform > wget https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip
unzip > unzip terraform_0.13.0_linux_amd64.zip 
move executable to local bin > sudo mv terraform /usr/local/bin/
verify version > terraform -v
clone sample> gsutil -m cp -r gs://spls/gsp233/* .
navigate > cd tf-gke-k8s-service-lb
review main.tf > cat main.tf and cat k8s.tf
start > terraform init
apply changes > terraform apply -> yes
Navigate > Kubernate engine > check if a service is created
-----------------------------------------------------------
# HTTPS Content-Based Load Balancer with Terraform
cloud shell> wget https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip
unzip terraform_0.13.0_linux_amd64.zip
> sudo mv terraform /usr/local/bin/
> git clone https://github.com/GoogleCloudPlatform/terraform-google-lb-http.git
> cd ~/terraform-google-lb-http/examples/multi-backend-multi-mig-bucket-https-lb
> terraform init
> terraform plan -out=tfplan -var 'project=<PROJECT_ID>'
> terraform apply tfplan
> verify at network > load balncing > ....
> EXTERNAL_IP=$(terraform output | grep load-balancer-ip | cut -d = -f2 | xargs echo -n)
> echo https://${EXTERNAL_IP}
-------------------------------------------------------------------
# Modular Load Balancing with Terraform - Regional Load Balancer
>wget https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip
>unzip terraform_0.13.0_linux_amd64.zip 
>sudo mv terraform /usr/local/bin/
>terraform -v
>git clone https://github.com/GoogleCloudPlatform/terraform-google-lb
cd ~/terraform-google-lb/examples/basic
> export GOOGLE_PROJECT=$(gcloud config get-value project)
> terraform init
> terrafrom plan > enter project id
> terraform apply > ener project id > yes
> EXTERNAL_IP=$(terraform output | grep load_balancer_default_ip | cut -d = -f2 | xargs echo -n)
> echo "http://${EXTERNAL_IP}"
------------------------------------------------------------------------
# Custom Providers with Terraform
> touch provider.go
package main
import (
        "github.com/hashicorp/terraform-plugin-sdk/helper/schema"
)
func Provider() *schema.Provider {
        return &schema.Provider{
                ResourcesMap: map[string]*schema.Resource{},
        }
}
> touch main.go
package main
import (
        "github.com/hashicorp/terraform-plugin-sdk/plugin"
        "github.com/hashicorp/terraform-plugin-sdk/terraform"
)
func main() {
        plugin.Serve(&plugin.ServeOpts{
                ProviderFunc: func() terraform.ResourceProvider {
                        return Provider()
                },
        })
}
> go mod init gopath/src/github.com/hashicorp/terraform-plugin-sdk
> go get github.com/hashicorp/terraform-plugin-sdk/helper/schema
go get github.com/hashicorp/terraform-plugin-sdk/plugin
go get github.com/hashicorp/terraform-plugin-sdk/terraform
> go build -o terraform-provider-example
> ./terraform-provider-example
> touch resource_server.go
package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/helper/schema"
)

func resourceServer() *schema.Resource {
	return &schema.Resource{
		Create: resourceServerCreate,
		Read:   resourceServerRead,
		Update: resourceServerUpdate,
		Delete: resourceServerDelete,
		Schema: map[string]*schema.Schema{
			"address": &schema.Schema{
				Type:     schema.TypeString,
				Required: true,
			},
		},
	}
}
func resourceServerCreate(d *schema.ResourceData, m interface{}) error {
	return nil
}
func resourceServerRead(d *schema.ResourceData, m interface{}) error {
	return nil
}
func resourceServerUpdate(d *schema.ResourceData, m interface{}) error {
	return nil
}
func resourceServerDelete(d *schema.ResourceData, m interface{}) error {
	return nil
}

and provider.go as >

package main

import (
	"github.com/hashicorp/terraform-plugin-sdk/helper/schema"
)

func Provider() *schema.Provider {
	return &schema.Provider{
		ResourcesMap: map[string]*schema.Resource{
			"example_server": resourceServer(),
		},
	}
}

> go build -o terraform-provider-example
> ./terraform-provider-example
*** Move the Provider to Plugins Directory
> mkdir -p ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.0/linux_amd64
> cp terraform-provider-example ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.0/linux_amd64
> touch main.tf
# This is required for Terraform 0.13+
terraform {
  required_providers {
    example = {
      version = "~> 1.0.0"
      source  = "example.com/qwiklabs/example"
    }
  }
}
resource "example_server" "my-server" {
    address = "1.2.3.4"
}
> terraform init
> terraform plan
> update resource server
func resourceServerCreate(d *schema.ResourceData, m interface{}) error {
        address := d.Get("address").(string)
        d.SetId(address)
        return nil
}
> go build -o terraform-provider-example
> mkdir -p ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.1/linux_amd64
cp terraform-provider-example ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.1/linux_amd64
> update main.tf
# This is required for Terraform 0.13+
terraform {
  required_providers {
    example = {
      version = "~> 1.0.1"
      source  = "example.com/qwiklabs/example"
    }
  }
}
> terraform init -upgrade
> terraform plan
> terraform apply
> go build -o terraform-provider-example
> mkdir -p ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.2/linux_amd64
cp terraform-provider-example ~/.terraform.d/plugins/example.com/qwiklabs/example/1.0.2/linux_amd64
> terraform init -upgrade
> terrafrom destroy
--------------------------------------------------
# Cloud SQL with Terraform
> mkdir sql-with-terraform
cd sql-with-terraform
gsutil cp -r gs://spls/gsp234/gsp234.zip .
> unzip gsp234.zip
> cat main.tf
> terraform init
> terraform plan -out=tfplan
> terraform apply tfplan
 create cloud proxy ***
> wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
> chmod +x cloud_sql_proxy
> export GOOGLE_PROJECT=$(gcloud config get-value project)
> ./cloud_sql_proxy -instances=${MYSQL_CONN_NAME}=tcp:3306
> cd ~/sql-with-terraform
> echo MYSQL_PASSWORD=$(terraform output -json | jq -r '.generated_user_password.value')
> mysql -udefault -p --host 127.0.0.1 default
---------------------------------------------------
# Building a VPN Between Google Cloud and AWS with Terraform 
in gcloud
> git clone https://github.com/GoogleCloudPlatform/autonetdeploy-multicloudvpn.git
> cd autonetdeploy-multicloudvpn
> gcloud IAM > Service Account
> compute engine default> manage keys > add new keys > json >create > download json
> upload json
> ./gcp_set_credentials.sh ~/[PROJECT_ID]-[UNIQUE_ID].json
> export username=`whoami`
mkdir /home/$username/.aws/
touch /home/$username/.aws/credentials_autonetdeploy
>nano /home/$username/.aws/credentials_autonetdeploy
>[default]
>aws_access_key_id=<Your AWS Access Key
> aws_secret_access_key=<Your AWS Secret Key>
>  cat /home/$username/.aws/credentials_autonetdeploy
> export TF_VAR_aws_credentials_file_path=/home/$username/.aws/credentials_autonetdeploy
> export PROJECT_ID=$(gcloud config get-value project)
gcloud config set project $PROJECT_ID
> ./gcp_set_project.sh
> cd terraform
terraform init
> terraform plan
> ssh-keygen -t rsa -f ~/.ssh/vm-ssh-key -C $username
> chmod 400 ~/.ssh/vm-ssh-key
> gcloud compute config-ssh --ssh-key-file=~/.ssh/vm-ssh-key
> copy compute engine >meta data > ssh key value
> AWS > EC2 > Import Key pair
> gcloud ...
  cd ~/autonetdeploy-multicloudvpn/terraform
  >terraform validate
  >terraform plan
  > terraform apply
  > terraform output
  > terraform show
  > gcloud compute instances list
  > ssh -i ~/.ssh/vm-ssh-key [GCP_EXTERNAL_IP]
  > ping -c 5 google.com
curl ifconfig.co/ip
  > /tmp/run_iperf_to_ext.sh
  > /tmp/run_iperf_to_int.sh
  >ssh -i ~/.ssh/vm-ssh-key ubuntu@[AWS_EXTERNAL_IP]
  >ping -c 5 google.com
curl ifconfig.co/ip
  >/tmp/run_iperf_to_ext.sh
  >/tmp/run_iperf_to_int.sh
