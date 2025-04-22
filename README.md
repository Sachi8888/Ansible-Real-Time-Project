#  🧾 Ansible Project: Automate EC2 Instance Lifecycle on AWS
## ✅ Objective

This project automates:

- Creating EC2 instances using loops
- Setting up password-less SSH between Ansible control node and EC2 instances
- Shutting down only Ubuntu instances using Ansible conditionals

## 📋 Prerequisites
Before starting this project, ensure you have the following:

- ✅ Ansible installed on your control node (Ubuntu/Linux preferred)
- ✅ An AWS account with access credentials (Access Key ID & Secret Key)
- ✅ An IAM user with permissions to create EC2 instances
- ✅ Ansible collections:
  
  ```
  ansible-galaxy collection install amazon.aws
  ```
  
- ✅ AWS CLI configured on the control node:

  ```
  aws configure
  ```
  
- ✅ A valid AWS key pair (.pem) for SSH access

- ✅ Basic understanding of writing Ansible playbooks

- Control node (local machine or EC2 instance) with:

    1.AWS CLI configured
    2.Python packages: boto, boto3, botocore

```
sudo apt update && sudo apt install -y ansible python3-pip
pip install boto boto3 botocore
```

## 📁 Project Structure

```
ansible-ec2-project/
├── group_vars/
│   └── all/
│       └── pass.yml             ← Encrypted using Ansible Vault
├── ec2_create.yaml              ← Creates EC2 instances
├── ec2_stop.yaml                ← Stops only Ubuntu EC2 instances
├── inventory.ini                ← Contains public IPs of instances
├── vault.pass                   ← DO NOT COMMIT!
└── .gitignore                   ← Prevents leaking secrets
```
## Project Setup
### 📌 Task 1: Create 3 EC2 Instances using Ansible Loops
- Create a playbook ec2_create.yml

```
---
- hosts: localhost
  connection: local

  tasks:
  - name: create EC2 instances
    amazon.aws.ec2_instance:
      name: "{{ item.name }}"
      key_name: "ansible_begin"
      instance_type: t2.micro
      security_group: default
      region: us-east-1
      aws_access_key: "{{ec2_access_key}}"
      aws_secret_key: "{{ec2_secret_key}}"
      network:
        assign_public_ip: true
      image_id: "{{ item.image }}"
    loop:
      - { image: "ami-084568db4383264d4", name: "ansible-ec2-1" }
      - { image: "ami-084568db4383264d4", name: "ansible-ec2-2" }
      - { image: "ami-0e449927258d45bc4", name: "ansible-ec2-3" }
```

- Run the playbook:

```
ansible-playbook ec2_create.yml  --vault-password-file vault.pass
```

#### 👉 Note down public IPs and add them to inventory.ini:

```
[all]
ec2-user@<PUBLIC_IP_OF_AMAZON_LIUNX_EC2>
ubuntu@<PUBLIC_IP_OF_UBUNTU_EC2-1>
ubuntu@<PUBLIC_IP_OF_UBUNTU_EC2-2>
```

### 📌 Task 2: Set Up Passwordless SSH Authentication
- Using Public Key:

```
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

- ssh-copy-id: This is the command used to copy your public key to a remote machine.

- -f: This flag forces the copying of keys, which can be useful if you have keys already set up and want to overwrite them.

- "-o IdentityFile ": This option specifies the identity file (private key) to use for the connection. The -o flag passes this option to the underlying ssh command.

- ubuntu@: This is the username (ubuntu) and the IP address of the remote server you want to access.

### 📌 Task 3: Shutdown Ubuntu Instances Only Using Conditionals
- Create a playbook ec2_stop.yml

```
---
- hosts: all
  become: true

  tasks:
    - name: Shutdown ubuntu instances only
      ansible.builtin.command: /sbin/shutdown -t now
      when:
        ansible_os_family == "Debian"
```

- Run the playbook:

```
ansible-playbook -i inventory ec2_stop.yml --vault-password-file vault.pass
```

At last don’t forget to terminate your ec2 instances after completing the project.

## 📸 Screenshots
- #### Creating IAM user with permission to Create EC2 Instances which named as ansible_admin:
  
  ![IAM_User](https://github.com/user-attachments/assets/fda12150-dce0-40f1-b6f2-588163ebca69)

-  #### Creation of EC2 instances after running ec2_create.yml file:

  ![ansible_instances](https://github.com/user-attachments/assets/bc6ae15f-2367-465a-822a-825a92ea3a0f)

- #### ansible output:

  ![ansible_output](https://github.com/user-attachments/assets/947f2f23-dbe6-4800-b060-f742ade98135)

- #### Shutting down only Ubuntu instances after running ec2_stop.yml file:

  ![Debian_stop](https://github.com/user-attachments/assets/136d47a4-e2d5-4365-81b2-2050999ce66f)



