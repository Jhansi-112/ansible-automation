# Ansible Build Server Automation

Set up a Linux build server with Ansible roles: Java, Maven and Docker.
Ansible works over SSH. Nothing is installed on the server for Ansible.

## What this project does

- One control node (WSL Ubuntu) runs Ansible.
- One remote server (AWS EC2, Ubuntu) is set up.
- Three roles install the tools for a Java CI pipeline.

## Project structure

```
ansible-automation/
├── ansible.cfg          # Ansible settings
├── inventory.ini        # Server list (group: build_servers)
├── build_server.yml     # Main playbook, calls the 3 roles
├── java_role/           # Installs OpenJDK 17, sets JAVA_HOME
├── maven_role/          # Installs Maven
└── docker_role/         # Installs Docker, log rotation, handler
```

## Roles

| Role | What it does | Main variables |
|---|---|---|
| java_role | Installs OpenJDK, sets JAVA_HOME with a template | java_version, java_package |
| maven_role | Installs Maven | maven_package |
| docker_role | Installs Docker, copies daemon.json, restarts Docker only on change (handler), adds users to the docker group | docker_users, docker_log_max_size, docker_log_max_file |

## How to run

1. Install Ansible: `sudo apt install -y ansible`
2. Create an Ubuntu EC2 server and set up SSH key login.
3. Put the server IP and key path in `inventory.ini`.
4. Test: `ansible build_servers -m ping`
5. Run: `ansible-playbook build_server.yml`

## Useful commands

```bash
ansible-playbook build_server.yml --syntax-check
ansible-playbook build_server.yml --check --diff
ansible-playbook build_server.yml --tags java
ansible-playbook build_server.yml --tags install
ansible-playbook build_server.yml -e "docker_log_max_size=20m"
```

## Concepts used

- Idempotency: the second run shows changed=0.
- Handlers and notify: Docker restarts only when its config changes.
- Tags: run only part of a playbook.
- Jinja2 templates: java.sh.j2 and daemon.json.j2.
- Variable priority: -e is higher than vars, and vars is higher than defaults.

## Work tracking

Each step has a Jira ticket (ANS-1 to ANS-10). Commit messages start with the ticket number. Changes go to main through pull requests.

## Credits

