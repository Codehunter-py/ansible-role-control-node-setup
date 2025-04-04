# Ansible Role: Control Node Setup

This Ansible role sets up a control node with Python virtual environments for both vSphere Automation SDK and Ansible-Core. It installs necessary system packages, sets up Python virtual environments, configures proxies, and ensures correct permissions. It also adds Git repository sync as a cron job and prepares the shell environment for the `ansible` service user.

## Requirements

- Ansible 2.9+
- Python 3
- Supported OS: CentOS, RHEL

## Role Variables

The following variables are configurable within your playbook or inventory to customize the role's behavior:

| Variable                    | Description                                                                                       | Default / Example Value                                      |
|-----------------------------|---------------------------------------------------------------------------------------------------|--------------------------------------------------------------|
| `required_packages`         | List of system packages required for the environment.                                            | `['python3', 'git', 'sshpass']`                              |
| `package_state`             | State of the system packages (`present`, `latest`, etc.).                                        | `present`                                                    |
| `virtualenv_sdk_packages`   | Python packages for the vSphere SDK environment.                                                 | `['pip', 'setuptools', 'pynetbox', 'netaddr', 'requests']`   |
| `virtualenv_requirements`   | Python packages with specific versions for the Ansible-Core environment.                         | `['pynetbox', 'pytz', 'netaddr', 'ansible']`                 |
| `virtualenv_sdk_path`       | Path to the vSphere SDK virtual environment.                                                     | `/opt/vsphere-automation-sdk-env`                            |
| `vsphere_sdk_url`           | URL for the vSphere SDK package.                                                                 | `git+https://github.com/vmware/vsphere-automation-sdk-python.git` |
| `venv_path`                 | Path for the Ansible-Core virtual environment.                                                   | `/opt/venv_ansible`                                          |
| `requirements_file`         | Path for the `requirements.txt` file.                                                            | `/opt/venv_ansible/requirements.txt`                         |
| `srv_ansible_home`          | Home directory for the `ansible` user.                                                           | `/home/ansible`                                              |
| `proxy_env`                 | Proxy settings to be configured in `.bashrc` for the service user.                               | `{http_proxy, https_proxy, no_proxy}`                        |
| `service_user`              | The service account name to be configured.                                                       | `ansible`                                                    |
| `git_repo_url`              | URL of the Git repository to clone and sync.                                                     | `git@github.com:example/automation.git`                      |
| `git_repo_path`             | Destination path where the Git repo will be cloned.                                              | `/opt/ansible-projects`                                      |
| `git_cron_state`            | Whether the Git sync cronjob should be `present` or `absent`.                                    | `present`                                                    |

## Usage Example

Include this role in your playbook and set the required variables:

### Example Playbook

```yaml
- name: Install and Configure Ansible Control Node
  hosts: sites_prod
  become: true
  gather_facts: true
  roles:
    - role: ansible-control
      vars:
        service_user: ansible
        required_packages:
          - python3
          - git
          - sshpass
        package_state: "latest"
        venv_path: "/opt/venv_ansible"
        virtualenv_requirements:
          - pynetbox
          - pytz
          - netaddr
          - ansible
        virtualenv_sdk_path: "/opt/vsphere-automation-sdk-env"
        vsphere_sdk_url: "git+https://github.com/vmware/vsphere-automation-sdk-python.git"
        proxy_env:
          http_proxy: "http://proxy.example.com:8080"
          https_proxy: "http://proxy.example.com:8080"
          no_proxy: "localhost,127.0.0.1"
        git_repo_url: "git@github.com:example/automation.git"
        git_repo_path: "/opt/ansible-projects"
        git_cron_state: "present"
```
### Role Tasks

- System Package Installation: Installs the required system packages using dnf.
- Proxy Configuration: Sets up proxy environment variables in .bashrc for the srv-ansible user.
- vSphere SDK Virtual Environment Setup: Creates and configures a Python virtual environment for the vSphere SDK.
- Ansible-Core Virtual Environment Setup: Creates and configures a Python virtual environment for Ansible-Core, including a requirements.txt file for specified packages.
- Permissions Management: Ensures that both virtual environment directories have the correct ownership and permissions.
- Service User Shell Configuration: Adds the following to .bashrc of the service user: 
  - Proxy environment variables
  - REQUESTS_CA_BUNDLE setting
  - Activation line for ansible-core virtualenv
- Git Repository Sync & Cron Setup:
  - Clones the specified Git repository if not already present.
  - Creates a cron job (hourly) to:
  - Checkout and reset the develop branch
  - Pull latest changes and clean up untracked files
  - Ensures the service user is allowed to use cron via /etc/cron.allow.

Author Information
------------------

[Ibrahim Musayev](https://github.com/Codehunter-py)
