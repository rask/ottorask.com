+++
title = "DigitalOcean automation with Terraform and Ansible"
date = 2018-02-28T16:30:00Z
slug = "digitalocean-automation-with-terraform-and-ansible"
+++

This post will walk you through a very basic setup of setting up automated
infrastructure and server provisioning on
{{ link(label="DigitalOcean", url="https://m.do.co/c/d0d4086e8abb", external=true) }}
using
{{ link(label="Hashicorp Terraform", url="https://www.terraform.io/", external=true) }}
and
{{ link(label="RedHat Ansible", url="https://www.ansible.com/", external=true) }}.

Terraform is a tool developed by Hashicorp that allows you to define your server
and cloud infrastructure using configuration. It makes automating infrastructure
dead simple and repeatable.

Ansible is a tool for configuration and software provisioning on a set of
servers of your choosing. You can provide Ansible with a static list of servers
to manage or then you can use a dynamic inventory script to fetch a list of
servers from a 3rd party service or a cloud provider themselves.

I work on an Ubuntu machine but these steps should be universal enough for you
to alter for other environments.

## What are we going to set up?

In this post, I will walk you through setting up a single Nginx web server on
DigitalOcean with a domain name, a tag, and a firewall. First, we will check out
how to configure Terraform and then we proceed to Ansible by first checking out
dynamic inventory and then provisioning itself.

This is a really simple example but will hopefully give you a solid introduction
of how and where Terraform and Ansible can be used.

If you know your way around documentation for both Terraform and Ansible you can
follow this walkthrough with something else than DigitalOcean. AWS has a free
tier which you can use to try this out with.

## Setting up Terraform

I will not be walking through installing Terraform as they have a good document
on that on their site. Instead, I will assume you have it installed already.

Terraform requires a DigitalOcean API token from you, so go ahead and create one
in your DO admin panel. DO will create a token and show you the token value
once, so make sure you save the token in some safe location.

**NOTE**: I will be using tokens quite unsafely in some parts of this
walkthrough, but I will show you a better way to handle secrets at the end of
the post.

First, we create a working directory for our configuration.

    $ mkdir /home/user/terraform-example
    $ cd /home/user/terraform-example

## Creating a droplet

Now you should be setup for an initial configuration. Create a file called
`main.tf` and append the following:

    provider "digitalocean" {
        token = "enter your secret DO API token here"
    }

    resource "digitalocean_droplet" "nginx_server" {
        name = "nginx-server"
        image = "ubuntu-16-04-x64"
        size = "512mb"
        region = "ams3"
        ipv6 = true
        private_networking = false
    }

This is already a completely operational example. Let’s run it and see what
happens.

Make sure you’re currently in the `terraform-example` directory. Then run

    $ terraform init

This command initializes backends and providers for you. If there are errors
then you need to find a solution for those.

After initialization, you can check out the current plan of action with

    $ terraform plan

This command outputs the current situation and wanted changes to your
infrastructure. If the configuration is OK you should see one new addition (the
droplet).

If the changes are OK in your eyes you can start making the changes!

    $ terraform apply

Terraform will take a while to let the droplet spawn properly. After you’re
greeted with a success message you can hop over to your DigitalOcean admin panel
and the droplet should be there, with the name `nginx-server`. Yay!

To get rid of the droplet and save some money you can destroy your
infrastructure.

    $ terraform destroy

After the command succeeds you should notice that the droplet is gone (forever)
in your DO admin panel as well.

## Tagging our servers

Tagging might seem redundant for a single server, but trust me: the sooner you
start tagging things the better.

We will create a single tag called `nginx` and apply that to our droplet.
Terraform’s DO provider allows creating tags with Terraform.

Inside `main.tf` create a new DO tag:

    resource "digitalocean_tag" "nginx" {
        name = "nginx"
    }

Now Terraform can create an **unbound** tag to be used with droplets. Let’s use
it with our droplet. Append a `tags` config value into our droplet config:

    resource "digitalocean_droplet" "nginx_server" {
        name = "nginx-server"
        image = "ubuntu-16-04-x64"
        size = "512mb"
        region = "ams3"
        ipv6 = true
        private_networking = false
        tags = ["${digitalocean_tag.nginx.name}"]
    }

(Notice how we use a reference to the tag resource, not just a string.)

Now if you run

    $ terraform plan && terraform apply

Our server should be tagged with `nginx`. Remember: tags are useful if not
required when working with automation systems.

## Adding an SSH key

Now you have configured an empty Ubuntu 16.04 server. It does not even have an
SSH key for you to log in with. Adding an SSH key is quite simple with DO and
Terraform. First, generate an SSH key pair with `ssh-keygen` locally (if you
haven’t already). Then supply the public counterpart to your DigitalOcean
account. When the public key is added to DO, you’re presented with a hash ID of
the key in the format of `12:34:45:ab:cd:ef:...`. This information is used to
tell Terraform and the DO API what SSH keys to inject to new servers.

Open up the `main.tf` file again and append a new configuration value to your
`nginx_server` droplet:

    resource "digitalocean_droplet" "nginx_server" {
        name = "nginx-server"
        image = "ubuntu-16-04-x64"
        size = "512mb"
        region = "ams3"
        ipv6 = true
        private_networking = false
        tags = ["${digitalocean_tag.nginx.name}"]
        ssh_keys = ["your hash id here"]
    }

Make sure you wrap the ID with `[]` characters or Terraform may start whining
about it.

Let’s see if that worked. Create the server with

    $ terraform plan && terraform apply

and then attempt to access the server with SSH. You can get the public IP of the
new instance from DO admin panel for instance.

    $ ssh root@123.123.123.123

If you can get into the server then great, it worked!

## Giving the server a domain name

(For example’s sake I will be using `example.com` as the TLD in these examples.)

First, make sure you’ve redirected your domain’s `NS` records to DigitalOcean’s
nameservers. After that, we can modify records with Terraform.

Adding a domain to a droplet with Terraform requires two new configs: a domain,
and a record. A domain is a reference to a domain name and a record tells where
a domain should be pointed. Create the following configuration inside `main.tf`

    resource "digitalocean_domain" "examplecom" {
        name = "example.com"
        ip_address = "${digitalocean_droplet.nginx_server.ipv4_address}"
    }

    resource "digitalocean_record" "examplecom" {
        name = "examplecom"
        type = "A"
        domain = "${digitalocean_domain.examplecom.name}"
        value = "${digitalocean_droplet.nginx_server.ipv4_address}"
    }

See how we first define a domain, then a record for the domain. Also notice the
use of variables and properties which we have defined elsewhere, e.g. the
droplet IP to which we want to point the domain to.

If your DNS configurations are right and the domains are manageable in DO admin
panel this config should work right now. Run

    $ terraform plan && terraform apply

And validate that the domain and record are visible in DO admin panel. You can
try accessing the domain with your browser but no web server is installed to
respond yet.

## Securing with a firewall

DigitalOcean offers infra-level firewalls for your droplets. We can create
firewalls and assign them to droplets with Terraform as well. As we are working
on a thing called `nginx_server` we can quite safely assume that this will be a
regular web server with traffic moving through ports 80 and 443. Additionally,
we want to allow SSH access in case someone needs to go in and check something.

In `main.tf` append a new firewall as such:

    resource "digitalocean_firewall" "webserver" {
        name = "webserver-firewall"
        droplet_ids = ["${digitalocean_droplet.nginx_server.id}"]

        inbound_rule = [
            {
                protocol = "tcp"
                port_range = "22"
            },
            {
                protocol = "tcp"
                port_range = "80"
            },
            {
                protocol = "tcp"
                port_range = "443"
            }
        ]

        outbound_rule = [
            {
                protocol = "tcp"
                port_range = "53"
            },
            {
                protocol = "udp"
                port_range = "53"
            }
        ]
    }

With `droplet_ids` we can define what droplets we use the firewall with. In
addition to inbound rules for HTTP(S) and SSH, we allow DNS traffic to external
networks in case someone on the server needs DNS information.

With this last piece of configuration in place, we can run a final `apply` and
check if the firewall appears in our DO panels. If it does then we have a server
ready for provisioning with Ansible.

## Setting up Ansible

As with Terraform, I presume you already have Ansible installed. We will be
using Ansible 2.4+ in this walkthrough.

### Getting inventory from DigitalOcean

We have created a server to DO with Terraform and now we need to fetch it back
for Ansible.
{{ link(label="Ansible luckily offers a nifty script for that in their GitHub repository", url="https://github.com/ansible/ansible/tree/devel/contrib/inventory", external=true) }}.

In `/etc/ansible` copy over the `digital_ocean.ini` and `digital_ocean.py`
files, and make the Python script executable with `chmod` or something.

Now take your secret DO API token and open the new INI file. Find a line where
the DO API token is defined and write the actual API token in. This will be used
to interact with the DO API when fetching your inventory.

To see if the inventory script is working you can issue a command:

    $ /etc/ansible/digital_ocean.py --pretty

This should output a bunch of information about your droplet(s). Among the data
you should see `nginx-server`, and other info we defined in our Terraform
configuration.

One of the most important informations is the droplet tag. When we defined our
droplet in Terraform we applied a tag called `nginx` to the droplet. This will
be vital when using Ansible.

### Creating a playbook

Playbooks are Ansible’s way of collecting a bunch of tasks together. We will use
a very simple playbook for this example. Create a new playbook file as such:

    $ mkdir /home/user/ansible-example
    $ touch /home/user/ansible-example/playbook.yml

Into the playbook insert the following configuration:

    ---
    - name: pretasks
      hosts: all
      remote_user: root
      gather_facts: false
      tasks:
        - name: install python
          raw: apt-get install python-minimal aptitude -y

    - name: nginx servers
      hosts: nginx
      remote_user: root
      tasks:
        - name: install nginx
          apt:
            name: nginx
            state: present

First, we make sure some regular Python interpreter is installed on all target
systems or otherwise Ansible cannot do preparing work like gather facts from the
servers (note the `gather_facts` setting, we prevent failure if Python is not
available this way).

The second task set is the most interesting one: we take a group of servers by
tag and do something on them. We defined the DigitalOcean tag `nginx` in our
Terraform config and now we can use it with Ansible!

We just install Nginx on the server with no other configuration.

This is an actual working playbook that we can run. Before running we need to go
over how to run this playbook properly against our DigitalOcean servers.

1.  Make sure we properly installed the `digital_ocean.py` script with proper
    INI API token config,
2.  We have a valid SSH private key which can access the DigitalOcean servers
    (this should be the same key pair we used when defining `ssh_keys` inside
    Terraform config).

To run the playbook, use a command as follows:

    $ sudo ansible-playbook ./playbook.yml \
        -i /etc/ansible/digital_ocean.py \
        --private-key /home/user/.ssh/id_rsa

Ansible should start working on your changes. If there are errors make sure you
have proper tags and tokens and keys configured.

If everything went OK you should see no `unreachable` or `failed` results. You
have just installed Nginx with Ansible to a server spawned with Terraform.
Congratulations!

Go check out `example.com` (your domain), you should see a Nginx welcome page.

Now that you’re done trying things out you can go back to the
`terraform-example` directory and just run

    $ terraform destroy

To get rid of all resources that are consuming money. Verify that they’re gone
in the DO admin panel.

## Securing secrets (a little at least)

Right now we have configured the DigitalOcean API token straight into your main
Terraform configuration file. This is bad and a different approach should be
taken when using secret variables in your setups.

Terraform provides a fileformat valled `.tfvars`. If you create a file called
`terraform.tfvars` in the same directory as your `main.tf` file Terraform will
read variables from it automatically.

Let’s move the DO API token to a `terraform.tfvars` file and make sure it does
not end up anywhere it should not. Open `main.tf` and adjust the DO provider
configuration section to look as follows:

    variable "digitalocean_api_token" {}

    provider "digitalocean" {
    token = "${var.digitalocean_api_token}"
    }

Now, inside `terraform.tfvars` add the following:

    digitalocean_api_token = "your secret DO API token"

You may now need to rerun `terraform init` to make sure the config change is
respected. If you now run `plan` and `apply` Terraform should read the API token
from `terraform.tfvars` instead of having it hardcoded into the configuration.
Step one done!

Next step will be to prevent the `terraform.tfvars` from leaking too easily.
There are many ways but a common one in cases where the configuration is
committed to a Git repository is to simply gitignore the file inside
`.gitignore`:

    *.tfvars

Now when you add secret variables into the file it will not be synced with the
rest of the world.

## In closing

Hopefully, you learned something while reading this walkthrough. This was a
really simple setup, especially when there was only a single server to setup and
configure.

I’ll leave it to you to try out how you can provision two similar servers or two
different servers using tags and Ansible.

(Please do let me know if there are errors in the post, I will fix those as soon
as possible.)