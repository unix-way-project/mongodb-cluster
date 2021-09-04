
This project is written for educational purposes to deploy MongoDB Highly Available Cluster in the cloud.

Cloud is initialized using terraform for Digital Ocean Cloud.
Terraform produces inventory file as it's output ready to be used with Ansible.

Servers are initialized using Ansible.
Load balancer is deployed on VM using HAProxy service
