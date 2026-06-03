<p align="center">
  <img src="https://img.shields.io/badge/Chef-Infra-F09820?style=for-the-badge&logo=chef&logoColor=white" alt="Chef"/>
  <img src="https://img.shields.io/badge/Platform-Ubuntu_24.04-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu"/>
  <img src="https://img.shields.io/badge/Cloud-AWS-232F3E?style=for-the-badge&logo=amazon-aws&logoColor=white" alt="AWS"/>
  <img src="https://img.shields.io/badge/IaC-Infrastructure_as_Code-4EAA25?style=for-the-badge&logo=gnu-bash&logoColor=white" alt="IaC"/>
</p>

<h1 align="center">🍳 Chef Infrastructure Automation</h1>

<p align="center">
  <strong>A complete, production-ready Chef automation project with cookbooks, recipes, and deployment guides</strong>
</p>

<p align="center">
  <a href="#-quick-start"><img src="https://img.shields.io/badge/Quick_Start-blue?style=flat-square" alt="Quick Start"/></a>
  <a href="#-architecture"><img src="https://img.shields.io/badge/Architecture-green?style=flat-square" alt="Architecture"/></a>
  <a href="#-cookbooks"><img src="https://img.shields.io/badge/Cookbooks-orange?style=flat-square" alt="Cookbooks"/></a>
  <a href="#-documentation"><img src="https://img.shields.io/badge/Documentation-purple?style=flat-square" alt="Docs"/></a>
  <a href="./CHEF_BEGINNER_GUIDE.md"><img src="https://img.shields.io/badge/📖_Beginner_Guide-red?style=flat-square" alt="Guide"/></a>
  <a href="./CHEF_COMMANDS_CHEATSHEET.md"><img src="https://img.shields.io/badge/🔪_Commands_Cheatsheet-yellow?style=flat-square" alt="Cheatsheet"/></a>
</p>

---

## 🎯 Overview

This repository contains a **complete Chef Infrastructure Automation** setup including:

- ✅ Production-ready cookbooks for web server provisioning
- ✅ End-to-end Server-Client model configuration on AWS
- ✅ Comprehensive beginner's guide (1200+ lines of documentation)
- ✅ Tested on Ubuntu 24.04 with AWS EC2 instances
- ✅ Bootstrap automation for scaling nodes

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Chef Infrastructure                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌──────────────────┐       ┌──────────────────────────────┐  │
│   │ CHEF WORKSTATION │       │        CHEF SERVER           │  │
│   │                  │       │                              │  │
│   │  • Write Code    │──────▶│  • Store Cookbooks           │  │
│   │  • Test Locally  │ knife │  • Manage Nodes              │  │
│   │  • Upload CBs    │upload │  • Authentication            │  │
│   │  • Bootstrap     │       │  • Run List Assignment       │  │
│   │                  │       │                              │  │
│   └──────────────────┘       └──────────────┬───────────────┘  │
│                                              │                   │
│                                    chef-client (pull)            │
│                                              │                   │
│                              ┌───────────────┼───────────────┐  │
│                              ▼               ▼               ▼  │
│                     ┌─────────────┐  ┌─────────────┐  ┌──────┐ │
│                     │   Node 1    │  │   Node 2    │  │  ... │ │
│                     │ (Web Server)│  │ (DB Server) │  │      │ │
│                     └─────────────┘  └─────────────┘  └──────┘ │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## ⚡ Quick Start

### Prerequisites

| Requirement | Specification |
|---|---|
| Cloud | AWS (ap-south-1) |
| OS | Ubuntu 24.04 LTS |
| Instance Type | 4 vCPU, 8 GB RAM, 30 GB disk |
| Ports | 22, 80, 443 |
| Network | All instances in same VPC |

### 1. Install Chef Workstation

```bash
sudo apt update
curl -L https://omnitruck.chef.io/install.sh | sudo bash -s -- -P chef-workstation
chef -v
```

### 2. Setup FQDN

```bash
# /etc/hosts on both machines
172.31.39.222   workstation.example.com
172.31.39.30    client.example.com
```

### 3. Generate Chef Repo

```bash
chef generate repo chef-repo
cd chef-repo/cookbooks
chef generate cookbook webserver
```

### 4. Bootstrap Client Node

```bash
knife bootstrap 172.31.39.30 \
  -N chef-client \
  --ssh-user ubuntu \
  --sudo \
  --ssh-identity-file ~/krishna_kubernetes.pem
```

---

## 📁 Project Structure

```
chef-repo/
├── cookbooks/
│   └── webserver/
│       ├── recipes/
│       │   ├── default.rb      # Main recipe (install + configure + service)
│       │   ├── install.rb      # Package installation
│       │   ├── configure.rb    # File configuration
│       │   └── service.rb      # Service management
│       ├── templates/
│       ├── attributes/
│       │   └── default.rb
│       ├── metadata.rb
│       └── README.md
├── roles/
├── environments/
├── data_bags/
├── CHEF_BEGINNER_GUIDE.md      # 📖 Complete beginner documentation
├── CHEF_COMMANDS_CHEATSHEET.md # 🔪 All Chef/Knife/Client commands
└── README.md                    # You are here
```

---

## 📚 Cookbooks

### webserver

Installs and configures Apache2 web server.

```ruby
# recipes/default.rb
package 'apache2' do
  action :install
end

file '/var/www/html/index.html' do
  content "Managed by Chef 🍳\n"
  action :create
end

service 'apache2' do
  action [:enable, :start]
end
```

### Run It

```bash
# Local mode (Chef Zero)
chef-client --local-mode --runlist "recipe[webserver]"

# With dry run
chef-client --local-mode --runlist "recipe[webserver]" --why-run

# Server-Client mode
knife cookbook upload webserver
knife node run_list add chef-client "recipe[webserver]"
```

---

## 🔧 Key Commands

<details>
<summary><strong>📋 Chef Workstation Commands</strong></summary>

```bash
chef generate repo <name>              # Generate chef repo
chef generate cookbook <name>           # Generate cookbook
chef generate recipe <cb> <recipe>     # Generate recipe
chef -v                                # Version info
cookstyle <path>                       # Lint check
```
</details>

<details>
<summary><strong>🔪 Knife Commands</strong></summary>

```bash
knife ssl fetch                        # Fetch SSL cert
knife ssl check                        # Verify SSL
knife cookbook upload <name>            # Upload cookbook
knife cookbook list                     # List cookbooks
knife node list                        # List nodes
knife node show <name>                 # Node details
knife node run_list add <node> "recipe[cb]"  # Assign recipe
knife bootstrap <IP> -N <name> --ssh-user <user> --sudo --ssh-identity-file <key>
```
</details>

<details>
<summary><strong>🏃 Chef Client Commands</strong></summary>

```bash
chef-client --local-mode <recipe.rb>                              # Run locally
chef-client --local-mode --runlist "recipe[cb::recipe]"           # Run with runlist
chef-client --local-mode --runlist "recipe[cb]" --why-run         # Dry run
chef-client                                                        # Server mode
chef-client --why-run                                              # Server dry run
```
</details>

---

## 📖 Documentation

| Document | Description |
|---|---|
| **[CHEF_BEGINNER_GUIDE.md](./CHEF_BEGINNER_GUIDE.md)** | Complete beginner's guide (1200+ lines) covering all Chef concepts |
| **[CHEF_COMMANDS_CHEATSHEET.md](./CHEF_COMMANDS_CHEATSHEET.md)** | All Chef, Knife, and Chef-Client commands (540+ lines) |
| **[Chef Docs](https://docs.chef.io)** | Official Chef documentation |
| **[Resources Reference](https://docs.chef.io/client/19/resources/bundled/)** | All built-in Chef resources |
| **[Supermarket](https://supermarket.chef.io)** | Community cookbooks |

---

## 🧪 Testing

```bash
# Syntax & Linting
cookstyle cookbooks/webserver/

# Unit Tests
chef exec rspec cookbooks/webserver/spec/

# Integration Tests (Test Kitchen)
cd cookbooks/webserver
kitchen test

# Smoke Test (Dry Run)
chef-client --local-mode --runlist "recipe[webserver]" --why-run
```

---

## 🚀 Deployment Workflow

```
 ┌─────────┐    ┌──────────┐    ┌──────────┐    ┌─────────┐    ┌────────┐
 │  Write  │───▶│  Lint    │───▶│  Test    │───▶│ Upload  │───▶│ Deploy │
 │  Code   │    │cookstyle │    │--why-run │    │  knife  │    │  chef  │
 └─────────┘    └──────────┘    └──────────┘    └─────────┘    └────────┘
```

1. **Write** — Create/edit recipes in cookbooks
2. **Lint** — `cookstyle cookbooks/mybook/`
3. **Test** — `chef-client --local-mode --why-run --runlist "recipe[mybook]"`
4. **Upload** — `knife cookbook upload mybook`
5. **Deploy** — `knife node run_list add <node> "recipe[mybook]"` → `chef-client` on node

---

## 🌐 Infrastructure Details

| Component | Details |
|---|---|
| **Region** | ap-south-1 (Mumbai) |
| **VPC** | vpc-09b0676901c3aee52 |
| **Workstation** | 172.31.39.222 (Ubuntu 24.04) |
| **Client Node** | 172.31.39.30 (Ubuntu 24.04) |
| **Security Group** | sg-064df3a92acddac32 |
| **Key Pair** | Krishna_kubernetes |

---

## 📊 Chef Concepts at a Glance

```
┌──────────────────────────────────────────────────────┐
│                  CHEF HIERARCHY                        │
├──────────────────────────────────────────────────────┤
│                                                       │
│  Resource    →  Smallest unit (package, file, etc.)  │
│      ↓                                                │
│  Recipe      →  Collection of resources (.rb file)   │
│      ↓                                                │
│  Cookbook    →  Collection of recipes + metadata      │
│      ↓                                                │
│  Run List    →  Ordered list of recipes for a node   │
│      ↓                                                │
│  Role        →  Reusable run list pattern            │
│      ↓                                                │
│  Environment →  Dev / Staging / Production           │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-cookbook`)
3. Write your cookbook with tests
4. Run `cookstyle` and fix any issues
5. Commit changes (`git commit -m 'Add new cookbook'`)
6. Push to branch (`git push origin feature/new-cookbook`)
7. Open a Pull Request

---

## 📝 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

## 👨‍💻 About

<p align="center">
  <strong>Developed by Krishna Yada</strong>
</p>

<p align="center">
  <a href="mailto:yadakrishna242@gmail.com"><img src="https://img.shields.io/badge/Email-yadakrishna242%40gmail.com-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email"/></a>
  <a href="https://github.com"><img src="https://img.shields.io/badge/GitHub-Krishna_Yada-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/></a>
  <a href="https://linkedin.com"><img src="https://img.shields.io/badge/LinkedIn-Krishna_Yada-0077B5?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn"/></a>
</p>

---

<p align="center">
  <img src="https://img.shields.io/badge/Status-Production_Ready-brightgreen?style=flat-square" alt="Status"/>
  <img src="https://img.shields.io/badge/Version-1.0.0-blue?style=flat-square" alt="Version"/>
  <img src="https://img.shields.io/badge/Last_Updated-June_2026-informational?style=flat-square" alt="Updated"/>
</p>

<p align="center"><em>Made with ❤️ and 🍳 by Krishna Yada</em></p>
