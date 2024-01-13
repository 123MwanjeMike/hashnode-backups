---
title: "Configure a MongoDB cluster with Ansible"
seoTitle: "Configure a MongoDB cluster with Ansible"
seoDescription: "With this Ansible role, you will be able to configure your ansible sharded cluster from scratch in a matter of only seconds."
datePublished: Fri Jan 12 2024 20:45:49 GMT+0000 (Coordinated Universal Time)
cuid: clrb3ykvj000009gvb9n91xwf
slug: configure-a-mongodb-cluster-with-ansible
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/cijiWIwsMB8/upload/b0a31d2b1d23b671ba2cf5ae28115d5f.jpeg
tags: mongodb, ansible, ansible-playbook, mongodb-cluster

---

Manual server configuration is always a nightmare, especially when there is a large fleet of servers to manage. However, with Infrastructure as Code(IaC) automation tools like Ansible, all that pain goes away. In this blog, we shall leverage the automation tool to create a MongoDB sharded cluster from scratch in just a few minutes, using [this Ansible role](https://github.com/123MwanjeMike/ansible-mongodb).

### Prerequisites

1. Ansible installed on your control node
    
2. Seven (or more) target servers running any Ubuntu OS version
    
3. SSH access to the target servers
    

## **Getting our hands dirty**

### Install the Ansible Role

As I mentioned earlier, we shall be using an existing Ansible role and so we won't have to write all the playbooks by ourselves from scratch. However, to install the role, we shall need a file that has it's installation information. A *requirements.yml* file . Run the commands below to create the file and the directory, *ansible\_mongodb,* which we shall be working.

```bash
mkdir ~/ansible_mongodb
echo "-   name: 123mwanjemike.ansible_mongodb
    src: https://github.com/123MwanjeMike/ansible-mongodb
    version: v1.0.4" > ~/ansible_mongodb/requirements.yml
```

<details data-node-type="hn-details-summary"><summary>Note on version v1.0.4</summary><div data-type="detailsContent"><em>The latest version of the role, 123mwanjemike.ansible_mongodb, at the time of writing this blog, was v1.0.4. There may however be more recent </em><a target="_blank" rel="noopener noreferrer nofollow" href="https://github.com/123MwanjeMike/ansible-mongodb/releases" style="pointer-events: none"><em>release versions</em></a><em> at the time you read this which are more recommended to use instead.</em></div></details>

You can now install the role using the Ansible Galaxy command below.

```bash
ansible-galaxy install -r ~/ansible_mongodb/requirements.yml
```

Output:

![Output from running the ansible-galaxy install command](https://cdn.hashnode.com/res/hashnode/image/upload/v1705081849770/0bda5aeb-5031-40a0-9ef2-18d672b0514e.png align="center")

### The Inventory File

Create an inventory using the command below

```bash
mkdir ~/ansible_mongodb/inventory
touch ~/ansible_mongodb/inventory/hosts
```

Next, populate the inventory with the target servers grouped according to the respective replicaSets. That is to say, the Config servers replicaSet servers in their group, and the Shard servers in the corresponding group.

Also, make sure to use the Fully Qualified Domain Names(FQDN) of the servers when adding them to the inventory. Below is how my inventory looks like.

```yaml
all:
  children:            
    mongo_cluster:
      children:
        mongos_routers:
          hosts:
            mongos-router-0.europe-north1-b.c.oceanic-muse-408212.internal:
        config_servers:
          hosts:
            mongod-cfgsvr-0.europe-north1-b.c.oceanic-muse-408212.internal:
            mongod-cfgsvr-1.europe-north1-c.c.oceanic-muse-408212.internal:
            mongod-cfgsvr-2.europe-north1-a.c.oceanic-muse-408212.internal:
        shard_0:
          hosts:
            mongod-shard-0-0.europe-north1-b.c.oceanic-muse-408212.internal:
            mongod-shard-0-1.europe-north1-c.c.oceanic-muse-408212.internal:
            mongod-shard-0-2.europe-north1-a.c.oceanic-muse-408212.internal:
```

### The Variables

There will be a number of variables at play when using this role. Let's get them right.

**Host Variables**  
Run the commands below to put them in place

1. All hosts
    
    ```bash
    mkdir ~/ansible_mongodb/inventory/group_vars
    echo "ansible_python_interpreter: /usr/bin/python3" > ~/ansible_mongodb/inventory/group_vars/all.yml
    ```
    
2. Router(s)
    
    ```bash
    echo "cluster_role: 'router'" > ~/ansible_mongodb/inventory/group_vars/mongos_routers.yml
    ```
    
3. Config Servers
    
    ```bash
    echo "cluster_role: 'config'
    
    replica_set:
      name: 'cfgsvr'
      group: 'config_servers'" > ~/ansible_mongodb/inventory/group_vars/config_servers.yml
    ```
    
4. Shard\_0 Servers
    
    ```bash
    echo "cluster_role: 'shard'
    
    replica_set:
      name: 'shard-0'
      group: 'shard_0'" > ~/ansible_mongodb/inventory/group_vars/shard_0.yml
    ```
    

**Configuration Variables**

You can find [the role's preset configuration variables](https://github.com/123MwanjeMike/ansible-mongodb/blob/develop/vars/main.yaml) in the `var/main.yml` file at the path where the role was installed. For example in my case, it was installed at `~/.ansible/roles/123mwanjemike.ansible_mongodb`.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705087388438/29a4d481-d23b-464a-a219-9e558f8ebd5b.png align="center")

You can change them as you wish, especially the `mongo_configs` versions and public keys given that they point to Ubuntu Jammy's MongoDB version 7.0 public keys.

**Cluster Variables**

As for the MongoDB cluster configuration variables, they have been documented on the role's README on GitHub [here](https://github.com/123MwanjeMike/ansible-mongodb#flags-and-variables).

### A Playbook

We're almost good to go. We now just need a playbook to install and configure the MongoDB sharded cluster. I build from the [sample playbook](https://github.com/123MwanjeMike/ansible-mongodb#sample-playbook) provided in the README of the role on GitHub. Below is my `~/ansible_mongodb/sample_playbook.yml` file

```yaml
- hosts: mongo_cluster
  become: true
  vars:
    adminUser: "myAdminUser"
    adminPass: "" # Secret password here
    new_shard:
      name: "shard-0"
      group: "shard_0"
    tgt_db: "myDatabase"
    roles: ["readWrite", "userAdmin"]
    userName: "targetUser"
    userPass: "" # Secret password here
    keyfile_src: "./keyfile"
    cfg_server:
      name: "cfgsvr"
      group: "config_servers"

  pre_tasks:
    - name: Generate random string with OpenSSL on ansible controller
      shell: openssl rand -base64 756 > keyfile
      delegate_to: localhost
      args:
        creates: keyfile

  roles:
    - { role: 123mwanjemike.ansible_mongodb, flags: ["install_mongo"] }
    - { role: 123mwanjemike.ansible_mongodb, flags: ["configure_mongo"] }
    - { role: 123mwanjemike.ansible_mongodb, flags: ["clear_logs"] }
    - { role: 123mwanjemike.ansible_mongodb, flags: ["prepare_members"] }
    - { role: 123mwanjemike.ansible_mongodb, flags: ["start_mongo"] }
    - {
        role: 123mwanjemike.ansible_mongodb,
        flags: ["init_replica"],
        when: cluster_role != 'router',
      }
    - {
        role: 123mwanjemike.ansible_mongodb,
        flags: ["create_admin"],
        when: cluster_role != 'router',
        delegate_to: "{{ groups[replica_set.group] | first }}",
      }
    - {
        role: 123mwanjemike.ansible_mongodb,
        flags: ["add_shard"],
        when: cluster_role == 'router',
        run_once: true,
      }
    - {
        role: 123mwanjemike.ansible_mongodb,
        flags: ["create_database"],
        when: cluster_role == 'router',
        run_once: true,
      }
```

---

The moment of truth has now come. We shall be running the playbook with the command below and see what happens.

```bash
cd ~/ansible_mongodb/
ansible-playbook sample_playbook.yml -i inventory/hosts
```

Output:

![Play recap](https://cdn.hashnode.com/res/hashnode/image/upload/v1705091039925/b72995d4-c256-4286-88b3-2afc4ade11e6.png align="center")

Checking out the cluster 🎉💃🏽🕺🏽😎

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1705132956920/4322c80f-a0ac-49e1-8bf6-a2ebd5bd9978.png align="center")

# Closing Remarks

With this Ansible role, we've been able to create and configure a MongoDB sharded cluster in under 10 minutes without writing any single configuration for the servers by ourselves.

Incase you were wondering, I am indeed the author of the role we have just used to create the cluster. I am also working on [this Terraform project](https://github.com/123MwanjeMike/mongo-cluster-terraform) that can be used to create the server instances for the sharded cluster used herein. Currently, I have only set up the provider for Google Cloud Platform(GCP) but will be adding Azure, AWS, and other service providers soon. I hope you enjoy them as much as I did when creating them.

*Cheers!*

## Additional resources

1. [Ansible](https://www.ansible.com/)
    
2. [\[Ansible\] Roles](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.html#roles)
    
3. [What Is a “Fully Qualified Domain Name”?](https://www.ssl.com/faqs/what-is-a-fully-qualified-domain-name/)
    
4. [\[MongoDB\] Sharded Cluster Components](https://www.mongodb.com/docs/manual/core/sharded-cluster-components/#sharded-cluster-components)
    
5. [What is a virtual private cloud (VPC)?](https://www.cloudflare.com/learning/cloud/what-is-a-virtual-private-cloud/)