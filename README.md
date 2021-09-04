
This project is written for educational purposes to deploy MongoDB Highly Available Cluster in the Cloud.

Cloud is initialized using terraform for Digital Ocean Cloud provider.
Terraform produces inventory file as it's output ready to be used with Ansible.

Servers are initialized using Ansible.
Load balancer is deployed on VM using HAProxy service

Installation process:
1. Genarate Personal Access token to access DigitalOcean cloud.
2. Generate SSH Key pair on your machine and add public key to the DigitalOcean public keys list.
3. Deploy virtual machines on the cloud.

Note: by default it will deploy following configuration:
* 1 droplet for Load Balancer
* 3 droplets for Config replicaset
* 2 replicasets 3 droplets each formed as a MongoDB shards

```
cd terraform/
terraform init
terraform apply -var do_token=<DigitalOcean Token> -var do_ssh_key=<SSH Key Name>
```

4. Please check ansible inventory file is properly created at terraform's directory: inventory.ini
5. Move to ansible directory and run the ansible automation
```
cd ../ansible
```

Optionaly you can validate if all servers are created properly and accessible:

```
ansible --user root --private-key ~/.ssh/unixway-keypair -i ../terraform/inventory.ini all -m ansible.builtin.shell -a 'hostname'
```

Initialize MongoDB cluster:

Important: generate strong password for admin user.
```
export MONGO_ADMIN_PASSWORD=admin

ansible-playbook --user root --private-key ~/.ssh/unixway-keypair -i ../terraform/inventory.ini ./mongodb.yml
```

Checking connectivity to cluster:
1. Get IP of Load Balancer droplet to conect to.
2. Username admin with password admin is created by default

```
mongo --host <IP address> --port 27017 --username admin --password admin
```

When you login to the MongoDB console as admin user, run following command to show the status of the shards:
```
sh.status()
```
You must see the list of all your shards properly initialized.
```
...
shards:
      {  "_id" : "rs0",  "host" : "rs0/10.0.0.3:27018,10.0.0.6:27018,10.0.0.7:27018",  "state" : 1 }
      {  "_id" : "rs1",  "host" : "rs1/10.0.0.2:27018,10.0.0.5:27018,10.0.0.8:27018",  "state" : 1 }
...
```

Important: Cloud resources are not cheap, please do not leave droplets running without requirements. You can always destroy all droplets with terraform:
```
terraform destroy -var do_token=<DigitalOcean Token>
```

Important: this automation is made for educational purposes so we ignored all security aspects to protect our workloads. All MongoDB nodes are using internal ports for communication, however, all MongoDB ports are publicly avaialble and anyone can login to the cluster as anonymous user.
