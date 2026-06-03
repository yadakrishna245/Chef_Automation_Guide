# 🔪 Chef Commands Cheat Sheet

**A comprehensive reference for all Chef CLI, Knife, and Chef-Client commands**

---

## 📋 Table of Contents

- [Chef Workstation CLI](#chef-workstation-cli)
- [Chef Client Commands](#chef-client-commands)
- [Knife — SSL & Connectivity](#knife--ssl--connectivity)
- [Knife — Cookbook Management](#knife--cookbook-management)
- [Knife — Node Management](#knife--node-management)
- [Knife — Bootstrap](#knife--bootstrap)
- [Knife — Role Management](#knife--role-management)
- [Knife — Environment Management](#knife--environment-management)
- [Knife — Data Bag Management](#knife--data-bag-management)
- [Knife — Search](#knife--search)
- [Knife — User & Org Management](#knife--user--org-management)
- [Knife — SSH & Remote Execution](#knife--ssh--remote-execution)
- [Knife — Status & Reporting](#knife--status--reporting)
- [Cookstyle (Linting)](#cookstyle-linting)
- [Test Kitchen](#test-kitchen)
- [InSpec (Compliance)](#inspec-compliance)
- [Berkshelf (Dependency Management)](#berkshelf-dependency-management)
- [Ohai (System Profiling)](#ohai-system-profiling)
- [Chef Server CTL](#chef-server-ctl)

---

## Chef Workstation CLI

```bash
# Version & Info
chef -v                                          # Show all Chef component versions
chef --help                                      # Show help

# Generate Structures
chef generate repo <repo_name>                   # Generate full chef repo structure
chef generate cookbook <cookbook_name>             # Generate new cookbook
chef generate recipe <cookbook> <recipe_name>     # Generate new recipe in cookbook
chef generate template <cookbook> <template_name> # Generate ERB template
chef generate attribute <cookbook> <attr_name>    # Generate attribute file
chef generate file <cookbook> <file_name>         # Generate cookbook file
chef generate resource <cookbook> <res_name>      # Generate custom resource
chef generate helper <cookbook> <helper_name>     # Generate helper library

# Chef Shell (Interactive Ruby with Chef DSL)
chef-shell                                       # Start interactive Chef shell
chef-shell --solo                                # Start in solo mode
chef-shell --client                              # Start in client mode
chef-shell -z                                    # Start in local (zero) mode
```

---

## Chef Client Commands

```bash
# ─── Local Mode (Chef Zero) ───

chef-client --local-mode <recipe.rb>             # Run single recipe locally
chef-client --local-mode --runlist "recipe[cookbook::recipe]"  # Run with runlist
chef-client --local-mode --runlist "recipe[cookbook]"          # Default recipe
chef-client --local-mode --runlist "recipe[cb1],recipe[cb2]"  # Multiple recipes
chef-client --local-mode --runlist "recipe[cookbook]" --why-run  # Dry run (local)
chef-client -z -r "recipe[cookbook]"             # Short form (-z = local mode)

# ─── Server Mode (Production) ───

chef-client                                      # Normal run (pull from server)
chef-client --why-run                            # Dry run against server
chef-client -W                                   # Short form for --why-run

# ─── Logging & Debug ───

chef-client --log_level debug                    # Debug output
chef-client --log_level info                     # Info level logging
chef-client --log_level warn                     # Warning level only
chef-client -l debug                             # Short form
chef-client --logfile /var/log/chef-client.log   # Log to file
chef-client -L /var/log/chef-client.log          # Short form

# ─── Configuration ───

chef-client --config /path/to/client.rb          # Use specific config file
chef-client -c /path/to/client.rb                # Short form
chef-client --override-runlist "recipe[test]"    # Override node's run list once
chef-client -o "recipe[test]"                    # Short form

# ─── Run Control ───

chef-client --once                               # Run once then exit (for daemon mode)
chef-client --interval 1800                      # Run every 30 minutes
chef-client --splay 300                          # Random delay up to 5 min (avoid thundering herd)
chef-client --daemonize                          # Run as daemon
chef-client -d                                   # Short form for daemon

# ─── Node & Environment ───

chef-client --node-name <name>                   # Override node name
chef-client -N <name>                            # Short form
chef-client --environment <env_name>             # Override environment
chef-client -E production                        # Short form

# ─── JSON Attributes ───

chef-client --json-attributes /path/to/attrs.json  # Pass JSON attributes
chef-client -j /path/to/run_list.json              # Short form

# ─── Formats ───

chef-client --format doc                         # Document format output
chef-client --format min                         # Minimal output
chef-client --format progress                    # Progress bar output
chef-client -F doc                               # Short form

# ─── Useful Combinations ───

chef-client -z -r "recipe[webserver]" -l info    # Local + runlist + info logging
chef-client -z -r "recipe[webserver]" -W         # Local + runlist + why-run
chef-client --once -l info -F doc                # One-time run with doc format
```

---

## Knife — SSL & Connectivity

```bash
knife ssl fetch                                  # Download SSL cert from Chef Server
knife ssl check                                  # Verify SSL connection to server
knife ssl fetch -s https://chef-server.com       # Fetch from specific server URL
```

---

## Knife — Cookbook Management

```bash
# Upload & Download
knife cookbook upload <cookbook_name>              # Upload cookbook to server
knife cookbook upload <cb1> <cb2> <cb3>           # Upload multiple cookbooks
knife cookbook upload --all                       # Upload all cookbooks
knife cookbook download <cookbook_name>            # Download cookbook from server
knife cookbook download <name> <version>          # Download specific version

# List & Show
knife cookbook list                               # List all cookbooks on server
knife cookbook list -a                            # List all versions
knife cookbook show <cookbook_name>                # Show cookbook details
knife cookbook show <name> <version>              # Show specific version details
knife cookbook show <name> <ver> recipes          # Show recipes in cookbook
knife cookbook show <name> <ver> attributes       # Show attributes

# Delete
knife cookbook delete <cookbook_name>              # Delete cookbook (latest version)
knife cookbook delete <name> <version>            # Delete specific version
knife cookbook delete <name> --all                # Delete all versions
knife cookbook delete <name> --purge              # Delete and purge from server

# Bulk Operations
knife cookbook bulk delete "^old_"                # Bulk delete matching pattern
```

---

## Knife — Node Management

```bash
# List & Show
knife node list                                  # List all registered nodes
knife node show <node_name>                      # Show node details
knife node show <node_name> -l                   # Show node details (long format)
knife node show <node_name> -a <attribute>       # Show specific attribute
knife node show <node_name> -a platform          # Show platform
knife node show <node_name> -a ipaddress         # Show IP address
knife node show <node_name> -a fqdn              # Show FQDN
knife node show <node_name> -r                   # Show run list only
knife node show <node_name> -F json              # Output as JSON

# Run List Management
knife node run_list add <node> "recipe[cookbook]"                    # Add recipe
knife node run_list add <node> "recipe[cookbook::recipe]"            # Add specific recipe
knife node run_list add <node> "role[webserver]"                     # Add role
knife node run_list add <node> "recipe[cb1],recipe[cb2]"            # Add multiple
knife node run_list add <node> "recipe[new]" --after "recipe[base]" # Add after specific item
knife node run_list remove <node> "recipe[cookbook]"                 # Remove recipe
knife node run_list remove <node> "role[webserver]"                  # Remove role
knife node run_list set <node> "recipe[cb1],recipe[cb2]"            # Replace entire run list

# Edit & Delete
knife node edit <node_name>                      # Edit node (opens editor)
knife node delete <node_name>                    # Delete node from server
knife node delete <node_name> -y                 # Delete without confirmation

# Environment
knife node environment set <node> <environment>  # Set node environment

# Create
knife node create <node_name>                    # Create node manually
knife node from file <filename.json>             # Create node from JSON file
```

---

## Knife — Bootstrap

```bash
# Basic Bootstrap (Linux)
knife bootstrap <IP_or_FQDN> \
  -N <node_name> \
  --ssh-user <username> \
  --sudo \
  --ssh-identity-file <path/to/key.pem>

# Bootstrap with Run List
knife bootstrap <IP> \
  -N <node_name> \
  --ssh-user ubuntu \
  --sudo \
  --ssh-identity-file ~/key.pem \
  --run-list "recipe[base],recipe[webserver]"

# Bootstrap with Environment
knife bootstrap <IP> \
  -N <node_name> \
  --ssh-user ubuntu \
  --sudo \
  --ssh-identity-file ~/key.pem \
  --environment production

# Bootstrap with Password (instead of key)
knife bootstrap <IP> \
  -N <node_name> \
  --ssh-user root \
  --ssh-password 'MyPassword'

# Bootstrap Windows Node
knife bootstrap windows winrm <IP> \
  -N <node_name> \
  --winrm-user Administrator \
  --winrm-password 'Password'

# Bootstrap Options
knife bootstrap <IP> -N <name> --ssh-user <user> --sudo \
  --ssh-identity-file <key> \
  --node-ssl-verify-mode none \           # Skip SSL verification
  --bootstrap-version 18.0.0 \            # Specific Chef Client version
  --use-sudo-password \                   # Prompt for sudo password
  --json-attributes '{"key":"value"}' \   # Pass JSON attributes
  --hint <hint_name>                      # Provide Ohai hint
```

---

## Knife — Role Management

```bash
# CRUD Operations
knife role create <role_name>                    # Create role (opens editor)
knife role list                                  # List all roles
knife role show <role_name>                      # Show role details
knife role edit <role_name>                      # Edit role
knife role delete <role_name>                    # Delete role

# From File
knife role from file roles/<role_name>.rb        # Create/update role from file
knife role from file roles/<role_name>.json      # From JSON file

# Bulk Operations
knife role bulk delete "^test_"                  # Bulk delete matching pattern
```

---

## Knife — Environment Management

```bash
# CRUD Operations
knife environment create <env_name>              # Create environment
knife environment list                           # List all environments
knife environment show <env_name>                # Show environment details
knife environment edit <env_name>                # Edit environment
knife environment delete <env_name>              # Delete environment

# From File
knife environment from file environments/<env>.rb    # Create from Ruby file
knife environment from file environments/<env>.json  # Create from JSON file

# Compare
knife environment compare <env1> <env2>          # Compare cookbook versions
```

---

## Knife — Data Bag Management

```bash
# CRUD Operations
knife data bag create <bag_name>                 # Create data bag
knife data bag create <bag_name> <item_name>     # Create item (opens editor)
knife data bag list                              # List all data bags
knife data bag show <bag_name>                   # Show items in data bag
knife data bag show <bag_name> <item_name>       # Show specific item
knife data bag edit <bag_name> <item_name>       # Edit item
knife data bag delete <bag_name>                 # Delete data bag
knife data bag delete <bag_name> <item_name>     # Delete item

# From File
knife data bag from file <bag_name> <item.json>  # Create item from JSON file
knife data bag from file <bag_name> data_bags/<bag_name>/  # Load all items from dir

# Encrypted Data Bags
knife data bag create <bag> <item> --secret-file /path/to/secret  # Create encrypted
knife data bag show <bag> <item> --secret-file /path/to/secret    # Decrypt and show
knife data bag edit <bag> <item> --secret-file /path/to/secret    # Edit encrypted
```

---

## Knife — Search

```bash
# Basic Search
knife search node '*:*'                          # All nodes
knife search node 'name:<node_name>'             # By node name
knife search node 'platform:ubuntu'              # By platform
knife search node 'platform_version:24.04'       # By platform version
knife search node 'role:webserver'               # By role
knife search node 'recipes:apache2'              # By recipe
knife search node 'chef_environment:production'  # By environment
knife search node 'ipaddress:172.31.*'           # By IP pattern
knife search node 'fqdn:*.example.com'           # By FQDN pattern

# Complex Queries
knife search node 'platform:ubuntu AND role:webserver'      # AND
knife search node 'platform:ubuntu OR platform:centos'      # OR
knife search node 'NOT platform:windows'                    # NOT
knife search node 'role:web AND chef_environment:prod'      # Combined

# Output Formatting
knife search node '*:*' -a ipaddress             # Show only IP
knife search node '*:*' -a fqdn                  # Show only FQDN
knife search node '*:*' -a platform              # Show only platform
knife search node '*:*' -a run_list              # Show only run lists
knife search node 'role:web' -a ipaddress -a fqdn  # Multiple attributes

# Search Other Indexes
knife search role '*:*'                          # Search roles
knife search environment '*:*'                   # Search environments
knife search client '*:*'                        # Search clients
knife search <data_bag_name> '*:*'               # Search data bag items
```

---

## Knife — User & Org Management

```bash
# User Management
knife user list                                  # List all users
knife user show <username>                       # Show user details
knife user create <username> -f <key_file>       # Create user
knife user edit <username>                       # Edit user
knife user delete <username>                     # Delete user

# Client Management
knife client list                                # List all clients
knife client show <client_name>                  # Show client details
knife client create <client_name>                # Create client
knife client delete <client_name>                # Delete client
knife client reregister <client_name>            # Regenerate client key
```

---

## Knife — SSH & Remote Execution

```bash
# Run command on nodes matching search
knife ssh 'role:webserver' 'sudo chef-client'                    # Run chef-client
knife ssh 'name:*' 'uptime'                                      # Check uptime
knife ssh 'platform:ubuntu' 'sudo apt update'                    # Update all Ubuntu nodes
knife ssh 'chef_environment:prod' 'df -h'                        # Check disk space

# With SSH options
knife ssh 'role:web' 'sudo chef-client' -x ubuntu -i ~/key.pem  # Specify user & key
knife ssh 'name:*' 'hostname' --ssh-user ubuntu                  # Specify user
```

---

## Knife — Status & Reporting

```bash
knife status                                     # Show all nodes with last check-in time
knife status "role:webserver"                    # Filter by role
knife status --long                              # Detailed status
knife status --run-list                          # Include run lists
knife status "platform:ubuntu" -r               # Ubuntu nodes with run lists
```

---

## Cookstyle (Linting)

```bash
cookstyle .                                      # Lint current directory
cookstyle <file.rb>                              # Lint single file
cookstyle cookbooks/<cookbook_name>/              # Lint entire cookbook
cookstyle --auto-correct                         # Auto-fix issues
cookstyle -a                                     # Short form auto-correct
cookstyle --format progress                      # Progress format
cookstyle --format json                          # JSON output
```

---

## Test Kitchen

```bash
# Lifecycle Commands
kitchen create                                   # Create test instance
kitchen converge                                 # Run Chef on instance
kitchen verify                                   # Run tests
kitchen destroy                                  # Destroy instance
kitchen test                                     # Full cycle (create→converge→verify→destroy)

# Instance Management
kitchen list                                     # List all test instances
kitchen login                                    # SSH into test instance
kitchen login <instance_name>                    # SSH into specific instance

# Selective Operations
kitchen converge <instance_name>                 # Converge specific instance
kitchen verify <instance_name>                   # Verify specific instance
kitchen destroy <instance_name>                  # Destroy specific instance

# Debug
kitchen diagnose                                 # Show configuration
kitchen diagnose --all                           # Show all config details
```

---

## InSpec (Compliance)

```bash
# Run Tests
inspec exec <test_path>                          # Run InSpec tests
inspec exec <test_path> -t ssh://user@host -i key.pem  # Remote execution
inspec exec <profile_url>                        # Run from URL
inspec exec compliance://<profile>               # Run compliance profile

# Profile Management
inspec init profile <name>                       # Create new profile
inspec check <profile_path>                      # Validate profile
inspec vendor <profile_path>                     # Vendor dependencies

# Shell (Interactive)
inspec shell                                     # Interactive testing
inspec shell -t ssh://user@host -i key.pem      # Remote interactive

# Detect
inspec detect                                    # Detect OS of local machine
inspec detect -t ssh://user@host                 # Detect remote OS
```

---

## Berkshelf (Dependency Management)

```bash
berks install                                    # Install cookbook dependencies
berks update                                     # Update all dependencies
berks update <cookbook_name>                      # Update specific cookbook
berks upload                                     # Upload all cookbooks to server
berks upload <cookbook_name>                      # Upload specific cookbook
berks list                                       # List resolved dependencies
berks info <cookbook_name>                        # Show cookbook info
berks search <query>                             # Search Supermarket
berks vendor <path>                              # Vendor cookbooks to directory
berks verify                                     # Verify Berksfile syntax
berks outdated                                   # Show outdated cookbooks
```

---

## Ohai (System Profiling)

```bash
ohai                                             # Full system profile (JSON)
ohai platform                                    # Show OS platform
ohai platform_version                            # Show OS version
ohai hostname                                    # Show hostname
ohai fqdn                                        # Show FQDN
ohai ipaddress                                   # Show IP address
ohai memory                                      # Show memory info
ohai cpu                                         # Show CPU info
ohai filesystem                                  # Show filesystem info
ohai network                                     # Show network info
ohai network/interfaces/eth0                     # Show specific interface
ohai --directory /path/to/plugins                # Load custom plugins
```

---

## Chef Server CTL

```bash
# Service Management
sudo chef-server-ctl reconfigure                 # Apply configuration
sudo chef-server-ctl status                      # Show all service status
sudo chef-server-ctl restart                     # Restart all services
sudo chef-server-ctl start                       # Start all services
sudo chef-server-ctl stop                        # Stop all services
sudo chef-server-ctl tail                        # Tail all service logs

# User Management
sudo chef-server-ctl user-create <user> <first> <last> <email> '<password>' --filename <key.pem>
sudo chef-server-ctl user-list                   # List all users
sudo chef-server-ctl user-show <username>        # Show user details
sudo chef-server-ctl user-delete <username>      # Delete user

# Organization Management
sudo chef-server-ctl org-create <org> "<description>" --association_user <user> --filename <validator.pem>
sudo chef-server-ctl org-list                    # List organizations
sudo chef-server-ctl org-show <org_name>         # Show org details
sudo chef-server-ctl org-delete <org_name>       # Delete organization
sudo chef-server-ctl org-user-add <org> <user>   # Add user to org
sudo chef-server-ctl org-user-remove <org> <user> # Remove user from org

# Maintenance
sudo chef-server-ctl cleanse                     # Reset server (DESTRUCTIVE!)
sudo chef-server-ctl gather-logs                 # Collect logs for support
sudo chef-server-ctl test                        # Run built-in tests
sudo chef-server-ctl backup                      # Backup server data
sudo chef-server-ctl restore <backup_file>       # Restore from backup
```

---

*Created by Krishna Yada | June 2026*
