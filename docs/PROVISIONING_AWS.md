# OpenShift on AWS EC2 using CASL

* Gather prerequisites
* Set up local environment
* Build inventory
* Provision a cluster
* Scale a Cluster
* Delete a Cluster

## Gather prerequisites

In addition to _cloning this repo_, you'll need the following:

* An AWS account with the proper policies & permissions to create & delete the following:
  * EC2 resources (instances, volumes, VPCs, subnets, security groups, etc.)
  * (Optional) Route 53 zones
  * (Optional) S3 Buckets
* A Key Pair created in EC2. [Click here for instructions](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#having-ec2-create-your-key-pair). Make sure you created it in the right region.
* An _environment id_ (`env_id`) and _DNS Domain_ (`dns_domain`) under which the cluster will be created. The two will be concatenated together to form the cluster identity in the form of `env_id.dns_domain` and will be used as the base for all other hostnames created.


## Set up a local environment

* Install Docker and Ansible
+
On RHEL/CentOS:
+
```
yum install -y docker ansible
```
+
On Fedora:
```
dnf install -y docker ansible
```
NOTE: If you plan to run docker as yourself (non-root), your username must be added to the `docker` user group.
* Clone this repo
```
cd ~/src/
git clone https://github.com/redhat-cop/casl-ansible.git
```
* Run `ansible-galaxy` to pull in the necessary requirements for the CASL provisioning of OpenShift on AWS
```
cd ~/casl-ansible
ansible-galaxy install -r casl-requirements.yml -p roles
```

## Build Inventory

* Create a new inventory from the sample in this repo.
```
cp -r casl-ansible/inventory/sample.aws.example.com.d/ ~/c1-ocp.example.com/
```
* Edit `~/c1-ocp.example.com/inventory/group_vars/all.yml`. At a minimum, you must set values for the following
```
...
env_id: "<REPLACE WITH VALID ENV ID - i.e: env1>"
...
aws_image_name: <REPLACE WITH VALID AMI>
...
dns_domain: "<REPLACE WITH A VALID ROUTE53 DNS DOMAIN>"
...
aws_region: <REPLACE WITH VALID AWS REGION>
...
rhsm_username: '<REPLACE WITH VALID RHSM USERNAME>'
rhsm_password: '<REPLACE WITH VALID RHSM PASSWORD>'

```

Other available parameters to use the AWS provision can be found in the Role's [README](../roles/manage-aws-infra/README.md)

* Modify 'regions' entry (line 13) in the inventory 'ec2.ini' file to match the 'aws_region' variable in your inventory
* Modify 'instance_filters' entry (line 14) in the inventory 'ec2.ini' file to match the 'env_id' variable in your inventory's `all.yml`

Cool! Now you're ready to provision OpenShift clusters on AWS

## Provision an OpenShift Cluster

As an example, we'll provision the `sample.aws.example.com` cluster defined in the `~/src/casl-ansible/inventory` directory.

> **Note**: *It is recommended that you use a different inventory similar to the ones found in the `~src/casl-ansible/inventory` directory and keep it elsewhere. This allows you to update/remove/change your casl-ansble source directory without losing your inventory. Also note that it may take some effort to get the inventory just right, hence it is very beneficial to keep it around for future use without having to redo everything.*

The following is just an example on how the `sample.aws.example.com` inventory can be used:

1) Edit `~/src/casl-ansible/inventory/sample.aws.example.com.d/inventory/group_vars/all.yml` to match your AWS environment. See comments in the file for more detailed information on how to fill these in.

2) Edit `~/src/casl-ansible/inventory/sample.aws.example.com.d/inventory/group_vars/OSEv3.yml` for your AWS specific configuration. See comments in the file for more detailed information on how to fill these in.

3) Run the `end-to-end` provisioning playbook via our [AWS installer container image](../images/installer-aws/). ** COMING SOON **

```
docker run -u `id -u` \
      -v $HOME/.ssh/id_rsa:/opt/app-root/src/.ssh/id_rsa:Z \
      -v $HOME/src/:/tmp/src:Z \
      -e INVENTORY_DIR=/tmp/src/casl-ansible/inventory/sample.aws.example.com.d/inventory \
      -e PLAYBOOK_FILE=/tmp/src/casl-ansible/playbooks/openshift/end-to-end.yml \
      -e OPTS="-e aws_key_name=my-key-name" -t \
      redhatcop/installer-aws
```

> **Note 1:** The `aws_key_name` variable at the end should specify the name of your AWS keypair - as noted under AWS Specific Requirements above.

> **Note 2:** The above bind-mounts will map files and source directories to the correct locations within the control host container. Update the local paths per your environment for a successful run.

Done! Wait till the provisioning completes and you should have an operational OpenShift cluster. If something fails along the way, either update your inventory and re-run the above `end-to-end.yml` playbook, or it may be better to [delete the cluster](https://github.com/redhat-cop/casl-ansible#deleting-a-cluster) and re-start.

## Updating a Cluster

Once provisioned, a cluster may be adjusted/reconfigured as needed by updating the inventory and re-running the `end-to-end.yml` playbook.

## Scaling Up and Down

A cluster's Infra and App nodes may be scaled up and down by editing the following parameters in the `hosts` or `all.yml` file and then re-running the `end-to-end.yml` playbook as shown above.

```
aws_num_nodes=X
aws_num_infra=Y
```

## Deleting a Cluster

A cluster can be decommissioned/deleted by re-using the same inventory with the `delete-cluster.yml` playbook found alongside the `end-to-end.yml` playbook.

```
docker run -u `id -u` \
      -v $HOME/.ssh/id_rsa:/opt/app-root/src/.ssh/id_rsa:Z \
      -v $HOME/src/:/tmp/src:Z \
      -e INVENTORY_DIR=/tmp/src/casl-ansible/inventory/sample.casl.example.com.d/inventory \
      -e PLAYBOOK_FILE=/tmp/src/casl-ansible/playbooks/openshift/delete-cluster.yml \
      -e OPTS="-e delete_vpc=true"
      redhatcop/installer-aws
```

> **Note:** While deleting an AWS Cluster, the `delete_vpc` variable must be set to `true` in order to remove the VPC used with the cluster. Check role [README](../roles/manage-aws-infra/README.md) for further information.
