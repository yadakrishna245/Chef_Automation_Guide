# 🍳 Chef Infrastructure Automation — Complete Beginner's Guide

**Author:** Krishna Yada  
**Version:** 1.0  
**Last Updated:** June 2026

---

## 📋 Table of Contents

1. [What is Chef?](#1-what-is-chef)
2. [Chef Architecture](#2-chef-architecture)
3. [Chef Components Overview](#3-chef-components-overview)
4. [Environment Setup](#4-environment-setup)
5. [Chef Resources](#5-chef-resources)
6. [Chef DSL (Domain Specific Language)](#6-chef-dsl)
7. [Chef Recipes](#7-chef-recipes)
8. [Chef Cookbooks](#8-chef-cookbooks)
9. [Chef Run Lists](#9-chef-run-lists)
10. [Include Recipe & Dependencies](#10-include-recipe--dependencies)
11. [Chef Zero (Local Mode)](#11-chef-zero-local-mode)
12. [Server-Client Model](#12-server-client-model)
13. [Knife Commands Reference](#13-knife-commands-reference)
14. [Attributes & Data Bags](#14-attributes--data-bags)
15. [Roles & Environments](#15-roles--environments)
16. [Chef Supermarket](#16-chef-supermarket)
17. [Testing & Best Practices](#17-testing--best-practices)
18. [Troubleshooting](#18-troubleshooting)
19. [Quick Reference Cheat Sheet](#19-quick-reference-cheat-sheet)
20. [Useful Links](#20-useful-links)

---

## 1. What is Chef?

Chef is a powerful **Infrastructure as Code (IaC)** automation tool that transforms infrastructure management into reusable, testable, and versionable code.

### Key Concepts (Kitchen Analogy)

| Kitchen Term | Chef Term | Description |
|---|---|---|
| Kitchen Tools | **Resources** | Basic building blocks (package, file, service) |
| Recipe Card | **Recipe** | Set of instructions using resources (.rb file) |
| Cookbook | **Cookbook** | Collection of recipes organized together |
| Customer Order | **Run List** | What recipes to run on which node, in what order |
| Head Chef | **Chef Server** | Central hub that stores cookbooks and manages nodes |
| Sous Chef | **Chef Workstation** | Where you write and test your code |
| Cook | **Chef Client** | Agent on managed nodes that executes recipes |

### Why Chef?

- **Idempotent** — Run code multiple times, same result every time
- **Declarative** — Define WHAT you want, not HOW to do it
- **Cross-platform** — Linux, Windows, macOS, Cloud
- **Version controlled** — Infrastructure code lives in Git
- **Scalable** — Manage 5 or 5000 servers with the same code
- **Community** — Thousands of pre-built cookbooks on Supermarket

---

## 2. Chef Architecture

### Two Deployment Models

#### Model 1: Chef Zero (Local/Standalone Mode)

```
┌─────────────────────────────────┐
│       Chef Workstation          │
│                                 │
│  Recipe → Chef Client (local)   │
│         → Apply on same machine │
└─────────────────────────────────┘
```

- No server needed
- Good for testing and development
- Use `--local-mode` flag

#### Model 2: Server-Client Model (Production)

```
┌──────────────────┐         ┌──────────────────┐
│ Chef Workstation │────────▶│   Chef Server    │
│ (Write code)     │ Upload  │ (Store cookbooks)│
└──────────────────┘         └────────┬─────────┘
                                      │
                              ┌───────┴────────┐
                              │   Pull recipes │
                              ▼                ▼
                     ┌──────────────┐  ┌──────────────┐
                     │ Chef Client  │  │ Chef Client  │
                     │   (Node 1)   │  │   (Node 2)   │
                     └──────────────┘  └──────────────┘
```

### How Server-Client Model Works

1. **Developer** writes cookbooks on the **Workstation**
2. Cookbooks are **uploaded** to the **Chef Server** using `knife`
3. Each **Node** has a Chef Client agent installed
4. Chef Client **pulls** its assigned recipes from the server
5. Chef Client **converges** the node to the desired state
6. Chef Client reports back to the server

---

## 3. Chef Components Overview

| Component | Purpose | Installation |
|---|---|---|
| **Chef Workstation** | Author, test, and upload cookbooks | `curl -L https://omnitruck.chef.io/install.sh \| sudo bash -s -- -P chef-workstation` |
| **Chef Server** | Central repository for cookbooks, policies, node data | Separate installation (hosted or self-managed) |
| **Chef Client (Infra Client)** | Agent running on managed nodes | Bootstrap from workstation OR manual install |
| **Knife** | CLI tool to interact with Chef Server | Included with Chef Workstation |
| **Ohai** | System profiler that detects node attributes | Included with Chef Client |
| **InSpec** | Compliance and testing framework | Included with Chef Workstation |
| **Cookstyle** | Linting tool for Chef code | Included with Chef Workstation |

---

## 4. Environment Setup

### Prerequisites

| Requirement | Details |
|---|---|
| OS | Ubuntu 24.04 LTS (recommended) |
| CPU | 4 vCPUs (minimum 2) |
| RAM | 8 GB (minimum 4 GB) |
| Disk | 30 GB |
| Ports | 22 (SSH), 80 (HTTP), 443 (HTTPS) |
| Network | All servers on same VPC/Network |
| Time Sync | NTP enabled, all machines synchronized |

### Step 1: Launch EC2 Instances

Launch 2 Ubuntu 24.04 EC2 instances in the same VPC:
- **Chef Workstation** (e.g., 172.31.39.222)
- **Chef Client** (e.g., 172.31.39.30)

### Step 2: Configure FQDN & /etc/hosts

On **BOTH** machines:

```bash
# Edit hosts file
sudo vi /etc/hosts

# Add entries:
172.31.39.222   workstation.example.com
172.31.39.30    client.example.com
```

Set hostname:

```bash
# On Workstation
sudo hostnamectl set-hostname workstation.example.com

# On Client
sudo hostnamectl set-hostname client.example.com
```

### Step 3: Verify Connectivity

```bash
ping workstation.example.com
ping client.example.com
```

> **Note:** Ensure ICMP is allowed in your Security Group for ping to work.

### Step 4: Time Synchronization

```bash
# On both machines
sudo apt install -y chrony
sudo systemctl enable chrony
sudo systemctl start chrony

# Verify time sync
chronyc tracking
timedatectl
```

### Step 5: Install Chef Workstation

On the **Workstation** machine:

```bash
sudo apt update
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-workstation
```

Verify installation:

```bash
chef -v
```

Expected output:
```
Chef Workstation version: 24.x.x
Chef Infra Client version: 18.x.x
Chef InSpec version: 5.x.x
Chef CLI version: 5.x.x
Chef Habitat version: 1.x.x
Test Kitchen version: 3.x.x
Cookstyle version: 7.x.x
```

### Step 6: Generate Chef Repo

```bash
chef generate repo chef-repo
cd chef-repo
```

---

## 5. Chef Resources

### What are Resources?

Resources are the **fundamental building blocks** in Chef. They define a piece of infrastructure and its desired state.

### Types of Resources

| Category | Resources |
|---|---|
| **Package Management** | `package`, `apt_package`, `yum_package`, `dpkg_package` |
| **File Management** | `file`, `cookbook_file`, `template`, `remote_file`, `directory` |
| **Service Management** | `service`, `systemd_unit` |
| **User Management** | `user`, `group` |
| **Execution** | `execute`, `bash`, `powershell_script`, `script` |
| **System** | `cron`, `mount`, `hostname`, `timezone` |
| **Network** | `firewall`, `iptables_rule` |

### Resource Syntax

```ruby
type 'name' do
  attribute1  'value1'
  attribute2  'value2'
  action      :action_name
end
```

### Components of a Resource

| Component | Description | Example |
|---|---|---|
| **Type** | Resource keyword (no quotes) | `package`, `file`, `service` |
| **Name** | Identifier/title (in quotes) | `'apache2'`, `'/var/www/html/index.html'` |
| **Attributes** | Properties defining desired state | `content`, `owner`, `mode`, `version` |
| **Action** | What to do (prefixed with `:`) | `:install`, `:create`, `:start`, `:enable` |
| **do...end** | Ruby block encapsulating the resource | Required when specifying attributes |

### Common Resource Examples

#### Package Resource
```ruby
# Install a package
package 'apache2' do
  action :install
end

# Short form (action :install is default)
package 'apache2'

# Install specific version
package 'nginx' do
  version '1.18.0'
  action :install
end

# Remove a package
package 'telnet' do
  action :remove
end
```

#### File Resource
```ruby
# Create a file with content
file '/var/www/html/index.html' do
  content '<h1>Hello from Chef!</h1>'
  owner   'www-data'
  group   'www-data'
  mode    '0644'
  action  :create
end

# Delete a file
file '/tmp/old_file.txt' do
  action :delete
end
```

#### Service Resource
```ruby
# Start and enable a service
service 'apache2' do
  action [:enable, :start]
end

# Restart a service
service 'nginx' do
  action :restart
end

# Stop a service
service 'mysql' do
  action :stop
end
```

#### Directory Resource
```ruby
directory '/opt/myapp' do
  owner  'deploy'
  group  'deploy'
  mode   '0755'
  recursive true
  action :create
end
```

#### User Resource
```ruby
user 'deploy' do
  comment  'Deployment User'
  home     '/home/deploy'
  shell    '/bin/bash'
  password '$6$encrypted_password_hash'
  action   :create
end
```

#### Execute Resource
```ruby
execute 'update_apt' do
  command 'apt-get update'
  action  :run
end
```

#### Template Resource
```ruby
template '/etc/nginx/nginx.conf' do
  source 'nginx.conf.erb'
  owner  'root'
  group  'root'
  mode   '0644'
  notifies :restart, 'service[nginx]'
end
```

### Resource Actions Reference

| Resource | Common Actions | Default Action |
|---|---|---|
| `package` | `:install`, `:remove`, `:upgrade`, `:purge` | `:install` |
| `file` | `:create`, `:delete`, `:touch` | `:create` |
| `service` | `:start`, `:stop`, `:restart`, `:enable`, `:disable` | `:nothing` |
| `directory` | `:create`, `:delete` | `:create` |
| `user` | `:create`, `:remove`, `:modify`, `:lock`, `:unlock` | `:create` |
| `execute` | `:run`, `:nothing` | `:run` |

### Guards (Conditional Execution)

```ruby
# Only run if condition is true
package 'apache2' do
  action :install
  only_if { node['platform'] == 'ubuntu' }
end

# Don't run if condition is true
execute 'install_app' do
  command './install.sh'
  not_if { File.exist?('/opt/app/installed') }
end
```

### Notifications

```ruby
# Notify another resource when this one changes
template '/etc/apache2/apache2.conf' do
  source 'apache2.conf.erb'
  notifies :restart, 'service[apache2]', :delayed
end

# Subscribe to changes in another resource
service 'apache2' do
  action [:enable, :start]
  subscribes :restart, 'template[/etc/apache2/apache2.conf]', :immediately
end
```

---

## 6. Chef DSL

### Chef Domain Specific Language

Chef DSL is based on **Ruby** but simplified for infrastructure automation. You don't need to be a Ruby expert — just understand the basics.

### DSL Structure (6 Steps)

```ruby
# Step 1: Resource Type (keyword, no quotes)
# Step 2: Resource Name (string, in quotes)
# Step 3: do (opens Ruby block)
# Step 4: Attributes (key-value pairs)
# Step 5: Action (colon + action name)
# Step 6: end (closes Ruby block)

file '/tmp/hello.txt' do          # Steps 1, 2, 3
  content 'Hello, Chef World!'    # Step 4
  mode    '0644'                  # Step 4
  action  :create                 # Step 5
end                               # Step 6
```

### Ruby Basics for Chef

```ruby
# Variables
my_port = 8080
app_name = 'myapp'

# String interpolation (double quotes only)
file "/opt/#{app_name}/config.yml" do
  content "port: #{my_port}"
end

# Conditionals
if node['platform'] == 'ubuntu'
  package 'apache2'
else
  package 'httpd'
end

# Arrays
['git', 'curl', 'wget'].each do |pkg|
  package pkg do
    action :install
  end
end

# Comments start with #
# This is a comment
```

---

## 7. Chef Recipes

### What is a Recipe?

A recipe is a **collection of resources** that describes a particular configuration. It's a `.rb` file containing Chef DSL code.

### Recipe Development Workflow

```
┌──────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────┐
│  CREATE  │────▶│ SYNTAX CHECK │────▶│  SMOKE TEST  │────▶│   RUN    │
│  (.rb)   │     │ (cookstyle)  │     │  (--why-run) │     │ (apply)  │
└──────────┘     └──────────────┘     └──────────────┘     └──────────┘
```

### Step 1: Create a Recipe

```bash
vi webserver.rb
```

```ruby
#
# Recipe: webserver.rb
# Description: Install and configure Apache web server
#

# Install Apache
package 'apache2' do
  action :install
end

# Create custom landing page
file '/var/www/html/index.html' do
  content "This is my webpage managed by Chef!\n"
  owner   'www-data'
  group   'www-data'
  mode    '0644'
  action  :create
end

# Start and enable Apache service
service 'apache2' do
  action [:enable, :start]
end
```

### Step 2: Syntax Check

```bash
cookstyle webserver.rb
```

### Step 3: Smoke Test (Dry Run)

```bash
chef-client --local-mode webserver.rb --why-run
```

> `--why-run` shows what WOULD happen without making actual changes.

### Step 4: Apply the Recipe

```bash
chef-client --local-mode webserver.rb
```

---

## 8. Chef Cookbooks

### What is a Cookbook?

A cookbook is the **organizational unit** in Chef. It contains:

```
cookbook_name/
├── recipes/           # Recipe files (.rb)
│   └── default.rb    # Default recipe (runs if no recipe specified)
├── templates/         # ERB template files
├── files/             # Static files to distribute
├── attributes/        # Default attribute values
│   └── default.rb
├── libraries/         # Custom Ruby code
├── resources/         # Custom resources
├── spec/              # Unit tests
├── test/              # Integration tests
├── metadata.rb        # Cookbook metadata (name, version, dependencies)
├── chefignore         # Files to ignore
├── Berksfile          # Dependency management
└── README.md          # Documentation
```

### Create a Cookbook

```bash
cd /home/ubuntu/chef-repo/cookbooks
chef generate cookbook webserver
```

### Edit the Default Recipe

```bash
cd webserver/recipes/
vi default.rb
```

```ruby
#
# Cookbook:: webserver
# Recipe:: default
#
# Copyright:: 2026, Krishna Yada, All Rights Reserved.

# Install Apache2
package 'apache2' do
  action :install
end

# Create landing page
file '/var/www/html/index.html' do
  content "This is my webpage for Chef Configuration tool\n"
  action :create
end

# Start Apache2 service
service 'apache2' do
  action [:enable, :start]
end
```

### Test and Run

```bash
# Syntax check
cookstyle recipes/default.rb

# Dry run
chef-client --local-mode --runlist "recipe[webserver::default]" --why-run

# Real run
chef-client --local-mode --runlist "recipe[webserver::default]"
```

### metadata.rb Explained

```ruby
name 'webserver'
maintainer 'Krishna Yada'
maintainer_email 'yadakrishna242@gmail.com'
license 'All Rights Reserved'
description 'Installs/Configures webserver'
version '1.0.0'
chef_version '>= 16.0'

# Dependencies
depends 'apt'
depends 'firewall'
```

---

## 9. Chef Run Lists

### What is a Run List?

A run list defines **which recipes** to execute on a node and in **what order**.

### Run List Syntax

```
"recipe[cookbook_name::recipe_name]"
```

### Examples

```bash
# Run default recipe from webserver cookbook
chef-client --local-mode --runlist "recipe[webserver::default]"

# Short form (default recipe assumed)
chef-client --local-mode --runlist "recipe[webserver]"

# Run a specific recipe
chef-client --local-mode --runlist "recipe[webserver::apache]"

# Run multiple recipes (order matters!)
chef-client --local-mode --runlist "recipe[base::default],recipe[webserver::default],recipe[monitoring::default]"

# With why-run (dry run)
chef-client --local-mode --runlist "recipe[webserver::default]" --why-run
```

### Run List Order Matters!

```bash
# This installs base packages FIRST, then configures web server
--runlist "recipe[base],recipe[webserver],recipe[firewall]"
```

Think of it like a restaurant order:
1. **Appetizer** (base setup) → served first
2. **Main Course** (application) → served second
3. **Dessert** (monitoring/cleanup) → served last

---

## 10. Include Recipe & Dependencies

### Include Recipe

Use `include_recipe` to call one recipe from another. This helps break large recipes into smaller, reusable pieces.

#### Example: Split webserver into multiple recipes

**recipes/default.rb** (main entry point):
```ruby
include_recipe 'webserver::install'
include_recipe 'webserver::configure'
include_recipe 'webserver::service'
```

**recipes/install.rb**:
```ruby
package 'apache2' do
  action :install
end
```

**recipes/configure.rb**:
```ruby
file '/var/www/html/index.html' do
  content "Welcome to Chef-managed server!\n"
  action :create
end
```

**recipes/service.rb**:
```ruby
service 'apache2' do
  action [:enable, :start]
end
```

### Cookbook Dependencies

When your cookbook depends on another cookbook, declare it in `metadata.rb`:

```ruby
# metadata.rb
depends 'apt'
depends 'ntpserver'
depends 'firewall'
```

Then use include_recipe to call recipes from dependent cookbooks:

```ruby
# recipes/default.rb
include_recipe 'apt::default'
include_recipe 'ntpserver::default'
```

---

## 11. Chef Zero (Local Mode)

### What is Chef Zero?

Chef Zero is a **lightweight, in-memory Chef Server** that runs locally. Perfect for:
- Learning Chef
- Testing recipes before production
- Standalone server configuration
- CI/CD pipelines

### Using Chef Zero

```bash
# Run a recipe file directly
chef-client --local-mode myrecipe.rb

# Run with runlist
chef-client --local-mode --runlist "recipe[webserver]"

# Dry run
chef-client --local-mode --runlist "recipe[webserver]" --why-run

# With specific log level
chef-client --local-mode --runlist "recipe[webserver]" --log_level info
```

### When to Use Chef Zero vs Server-Client

| Feature | Chef Zero | Server-Client |
|---|---|---|
| Setup complexity | None | High |
| Nodes managed | 1 (local) | Unlimited |
| Centralized management | No | Yes |
| Use case | Dev/Test | Production |
| Requires network | No | Yes |

---

## 12. Server-Client Model

### Architecture

```
┌───────────────────────────────────────────────────────────────┐
│                        WORKFLOW                                 │
│                                                                │
│  Workstation ──(knife upload)──▶ Chef Server ◀──(pull)── Nodes│
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Setup Overview

#### On Chef Workstation:

```bash
# 1. Create chef config directory
mkdir -p /root/.chef

# 2. Create knife configuration
cat > /root/.chef/knife.rb << 'EOF'
current_dir = File.dirname(__FILE__)
node_name                "admin"
client_key               "/root/.chef/admin.pem"
chef_server_url          "https://chef-server.example.com/organizations/myorg"
cookbook_path             ["#{current_dir}/../cookbooks"]
ssl_verify_mode          :verify_none
EOF

# 3. Fetch SSL certificate
knife ssl fetch

# 4. Verify connectivity
knife ssl check

# 5. List nodes
knife node list
```

### Bootstrap a Client Node

Bootstrap installs Chef Client on a remote node AND registers it with the Chef Server:

```bash
knife bootstrap <CLIENT_IP> \
  -N <NODE_NAME> \
  --ssh-user ubuntu \
  --sudo \
  --ssh-identity-file /path/to/private_key.pem
```

### Server-Client Workflow

```bash
# 1. Create cookbook on workstation
chef generate cookbook mycookbook

# 2. Write your recipe
vi cookbooks/mycookbook/recipes/default.rb

# 3. Syntax check
cookstyle cookbooks/mycookbook

# 4. Test locally first
chef-client --local-mode --runlist "recipe[mycookbook]" --why-run

# 5. Upload to Chef Server
knife cookbook upload mycookbook

# 6. Verify upload
knife cookbook list

# 7. Assign run list to node
knife node run_list add chef-client "recipe[mycookbook]"

# 8. On the client node, run chef-client to converge
# (SSH to client or wait for automatic run)
chef-client
```

### Managing Nodes

```bash
# List all nodes
knife node list

# Show node details
knife node show chef-client

# Edit node run list
knife node run_list add chef-client "recipe[webserver]"
knife node run_list remove chef-client "recipe[old_cookbook]"

# Delete a node
knife node delete chef-client
knife client delete chef-client
```

---

## 13. Knife Commands Reference

### SSL & Connectivity

| Command | Description |
|---|---|
| `knife ssl fetch` | Download SSL certificate from Chef Server |
| `knife ssl check` | Verify SSL connectivity |

### Cookbooks

| Command | Description |
|---|---|
| `knife cookbook upload <name>` | Upload cookbook to server |
| `knife cookbook list` | List all cookbooks on server |
| `knife cookbook show <name>` | Show cookbook details |
| `knife cookbook delete <name>` | Delete cookbook from server |
| `knife cookbook download <name>` | Download cookbook from server |

### Nodes

| Command | Description |
|---|---|
| `knife node list` | List all registered nodes |
| `knife node show <name>` | Show node details |
| `knife node run_list add <node> "recipe[cookbook]"` | Add recipe to node's run list |
| `knife node run_list remove <node> "recipe[cookbook]"` | Remove recipe from run list |
| `knife node delete <name>` | Delete node |

### Bootstrap

| Command | Description |
|---|---|
| `knife bootstrap <IP> -N <name> --ssh-user <user> --sudo --ssh-identity-file <key>` | Bootstrap a new node |

### Search

| Command | Description |
|---|---|
| `knife search node '*:*'` | Search all nodes |
| `knife search node 'platform:ubuntu'` | Search nodes by platform |
| `knife search node 'role:webserver'` | Search nodes by role |

---

## 14. Attributes & Data Bags

### Attributes

Attributes define specific values about a node. They have a **precedence order**.

#### Attribute Precedence (lowest to highest)

1. `default` — Cookbook attributes
2. `normal` — Set on node (persists)
3. `override` — Force a value
4. `automatic` — Ohai-detected (highest priority)

#### Defining Attributes

**attributes/default.rb:**
```ruby
default['webserver']['port'] = 80
default['webserver']['document_root'] = '/var/www/html'
default['webserver']['server_name'] = 'localhost'
```

**Using attributes in recipes:**
```ruby
template '/etc/apache2/ports.conf' do
  source 'ports.conf.erb'
  variables(
    port: node['webserver']['port']
  )
end
```

### Data Bags

Data bags store **global JSON data** accessible by all nodes (e.g., user lists, database credentials).

```bash
# Create a data bag
knife data bag create users

# Create an item in the data bag
knife data bag from file users admin.json
```

**data_bags/users/admin.json:**
```json
{
  "id": "admin",
  "username": "deploy",
  "ssh_keys": ["ssh-rsa AAAA..."],
  "groups": ["sudo", "www-data"]
}
```

**Using in a recipe:**
```ruby
admin = data_bag_item('users', 'admin')

user admin['username'] do
  home "/home/#{admin['username']}"
  shell '/bin/bash'
  action :create
end
```

---

## 15. Roles & Environments

### Roles

Roles define patterns of configuration that can be assigned to multiple nodes.

```bash
# Create a role
knife role create webserver
```

**roles/webserver.rb:**
```ruby
name "webserver"
description "Web Server Role"
run_list(
  "recipe[base]",
  "recipe[apache]",
  "recipe[monitoring]"
)
default_attributes(
  "apache" => {
    "port" => 80
  }
)
```

```bash
# Assign role to node
knife node run_list add mynode "role[webserver]"
```

### Environments

Environments map to your deployment stages (dev, staging, production).

**environments/production.rb:**
```ruby
name "production"
description "Production Environment"
cookbook_versions(
  "webserver" => "= 2.0.0",
  "database"  => ">= 1.5.0"
)
default_attributes(
  "app" => {
    "debug" => false
  }
)
```

```bash
# Set node environment
knife node environment set mynode production
```

---

## 16. Chef Supermarket

### What is Supermarket?

**Chef Supermarket** (https://supermarket.chef.io) is the community cookbook repository — like npm for Node.js or PyPI for Python.

### Using Community Cookbooks

**Berksfile:**
```ruby
source 'https://supermarket.chef.io'

metadata
cookbook 'apt'
cookbook 'nginx'
cookbook 'java'
```

```bash
# Install dependencies
berks install

# Upload to Chef Server
berks upload
```

### Popular Community Cookbooks

| Cookbook | Purpose |
|---|---|
| `apt` | APT package management |
| `yum` | YUM package management |
| `nginx` | Nginx web server |
| `java` | Java installation |
| `docker` | Docker management |
| `mysql` | MySQL database |
| `postgresql` | PostgreSQL database |
| `users` | User management |
| `ntp` | Time synchronization |
| `firewall` | Firewall rules |

---

## 17. Testing & Best Practices

### Testing Tools

| Tool | Purpose | Command |
|---|---|---|
| **Cookstyle** | Linting/style check | `cookstyle .` |
| **ChefSpec** | Unit testing | `chef exec rspec` |
| **Test Kitchen** | Integration testing | `kitchen test` |
| **InSpec** | Compliance testing | `inspec exec profile/` |

### Test Kitchen (Integration Testing)

**.kitchen.yml:**
```yaml
driver:
  name: vagrant

provisioner:
  name: chef_zero

platforms:
  - name: ubuntu-24.04

suites:
  - name: default
    run_list:
      - recipe[webserver::default]
```

```bash
kitchen create    # Create VM
kitchen converge  # Run Chef
kitchen verify    # Run tests
kitchen destroy   # Cleanup
kitchen test      # All in one
```

### Best Practices

1. **Always test locally first** — Use `--why-run` before applying
2. **Use cookstyle** — Follow linting recommendations
3. **Version your cookbooks** — Semantic versioning in metadata.rb
4. **Use templates over files** — ERB templates are more flexible
5. **Keep recipes small** — Use `include_recipe` to split logic
6. **Use attributes** — Don't hardcode values in recipes
7. **Use guards** — `only_if` and `not_if` for idempotency
8. **Test with Kitchen** — Integration tests catch real issues
9. **Document everything** — README.md in every cookbook
10. **Use Berkshelf** — Manage cookbook dependencies

---

## 18. Troubleshooting

### Common Issues & Solutions

| Problem | Cause | Solution |
|---|---|---|
| `No knife configuration file found` | Missing knife.rb | Create `/root/.chef/knife.rb` |
| `private key could not be loaded` | Missing .pem file | Copy admin.pem from Chef Server |
| `Connection refused` | Server not running or port blocked | Check security groups, verify services |
| `SSL certificate verify failed` | Certificate not fetched | Run `knife ssl fetch` |
| `Cookbook not found` | Not uploaded to server | Run `knife cookbook upload <name>` |
| `ping not working` | ICMP not allowed in Security Group | Add ICMP rule to security group |
| `Permission denied (publickey)` | Wrong key file or permissions | Check key file path, `chmod 400 key.pem` |
| `Node already exists` | Previous bootstrap | `knife node delete <name>` then re-bootstrap |

### Debug Commands

```bash
# Verbose Chef Client output
chef-client --local-mode -l debug --runlist "recipe[cookbook]"

# Check Chef Server status
knife status

# Verify node registration
knife node show <node_name>

# Check cookbook syntax
cookstyle cookbooks/mycookbook/

# Test connectivity
knife ssl check
knife node list
```

---

## 19. Quick Reference Cheat Sheet

### Workstation Commands

```bash
# Generate structures
chef generate repo <name>
chef generate cookbook <name>
chef generate recipe <cookbook> <recipe>
chef generate template <cookbook> <template>
chef generate attribute <cookbook> <attribute>

# Version check
chef -v

# Cookbook linting
cookstyle <path>
```

### Chef Client Commands

```bash
# Local mode (Chef Zero)
chef-client --local-mode <recipe.rb>
chef-client --local-mode --runlist "recipe[cookbook::recipe]"
chef-client --local-mode --runlist "recipe[cookbook]" --why-run

# Server mode (on node)
chef-client                    # Normal run
chef-client --why-run          # Dry run
chef-client --log_level debug  # Debug output
```

### Knife Commands

```bash
# SSL
knife ssl fetch
knife ssl check

# Cookbooks
knife cookbook upload <name>
knife cookbook list
knife cookbook show <name>
knife cookbook delete <name>

# Nodes
knife node list
knife node show <name>
knife node run_list add <node> "recipe[cookbook]"
knife node run_list remove <node> "recipe[cookbook]"
knife node delete <name>

# Bootstrap
knife bootstrap <IP> -N <name> --ssh-user <user> --sudo --ssh-identity-file <key>

# Search
knife search node '*:*'
knife search node 'platform:ubuntu'

# Roles
knife role list
knife role show <name>

# Data Bags
knife data bag list
knife data bag show <name>
```

---

## 20. Useful Links

| Resource | URL |
|---|---|
| Chef Documentation | https://docs.chef.io |
| Chef Resources Reference | https://docs.chef.io/client/19/resources/bundled/ |
| Chef Supermarket | https://supermarket.chef.io |
| Chef Downloads | https://downloads.chef.io |
| Manage Organizations | https://manage.chef.io/organizations |
| Chef GitHub | https://github.com/chef/chef |
| Learn Chef (Official) | https://learn.chef.io |

---

## 🎯 Learning Path Summary

```
Beginner:
  Resources → Recipes → Chef Zero (local mode) → Cookbooks → Run Lists

Intermediate:
  Server-Client Model → Knife → Bootstrap → Attributes → Templates

Advanced:
  Roles → Environments → Data Bags → Custom Resources → Testing → CI/CD
```

---

*Created by Krishna Yada | June 2026*
