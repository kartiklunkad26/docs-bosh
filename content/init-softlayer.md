This document shows how to create a new [environment](terminology.md#environment) on IBM Cloud Infrastructure (previously called SoftLayer).

## Step 1: Prepare an IBM Cloud Infrastructure Environment {: #prepare }

To prepare your IBM Cloud Infrastructure environment:

* [Create a SoftLayer account](#account)
* [Generate a SoftLayer API Key](#api-key)
* [Access SoftLayer VPN](#vpn)
* [Order VLANs](#vlan)

---
### Create a SoftLayer account {: #account }

If you do not have an SoftLayer account, [create one for one month free](https://www.softlayer.com/promo/freeCloud).

Use the login credentials received in your provided email to login to SoftLayer [Customer Portal](https://control.softlayer.com).

---
### Generate an API Key {: #api-key }

API keys are used to securely access the SoftLayer API. Follow [Generate an API Key](http://knowledgelayer.softlayer.com/procedure/generate-api-key) to generate your API key.

---
### Access SoftLayer VPN {: #vpn }

To access SoftLayer Private network, you need to access SoftLayer VPN. Follow [VPN Access](http://www.softlayer.com/vpn-access) to access the VPN. You can get your VPN password from your [user profile](https://control.softlayer.com/account/user/profile). Follow [VPN Access](http://www.softlayer.com/vpn-access) to access the VPN.

---
### Order VLANs {: #vlan }

VLANs provide the ability to partition devices and subnets on the network. To order VLANs, login to SoftLayer [Customer Portal](https://control.softlayer.com) and navigate to Network > IP Management > VLANs. Once on the page, click the "Order VLAN" link in the top-right corner. Fill in the pop-up window to order the VLANs as you need. The VLAN IDs are needed in the deployment manifest.

---
## Step 2: Deploy {: #deploy }

1. Install [BOSH CLI v2](cli-v2.md).

2. Establish VPN to make sure you can connect to IBM Cloud Infrastructure over private network from your workstation where to run BOSH CLI. 

3. Apply a portable IP from Softlayer. Let's say it's `10.0.0.6`.

4. Use `bosh create-env` command to deploy the Director.

    ```shell
    # Create a worksapce directory to keep state
    mkdir -p bosh-workspace && cd bosh-workspace

    # Clone Director templates
    git clone https://github.com/cloudfoundry/bosh-deployment

    # Fill below variables (replace example values) and deploy the Director
    sudo bosh create-env bosh-deployment/bosh.yml \
        --state=state.json \
        --vars-store=creds.yml \
        -o bosh-deployment/softlayer/cpi.yml \  
        -v director_name=bosh \
        -v internal_cidr=10.0.0.0/24 \        
        -v internal_gw=10.0.0.1 \             
        -v internal_ip=10.0.0.6 \
        -v sl_datacenter= \
        -v sl_vm_domain= \
        -v sl_vm_name_prefix= \
        -v sl_vlan_public= \
        -v sl_vlan_private= \
        -v sl_username= \
        -v sl_api_key=
    ```

5. Connect to the Director.

    ```shell
    # Configure local alias
    bosh alias-env bosh-1 -e 10.0.0.6 --ca-cert <(bosh int ./creds.yml --path /director_ssl/ca)

    # Log in to the Director
    export BOSH_CLIENT=admin
    export BOSH_CLIENT_SECRET=`bosh int ./creds.yml --path /admin_password`

    # Query the Director for more info
    bosh -e bosh-1 env
    ```

6. Save the deployment state files stored in your workspace directory `bosh-workspace` for future update/deletion. See [Deployment state](cli-envs.md#deployment-state) for details.
