Ansible Workstation

Completed By: GINIKA NDUDUISI

Project Objective

I set up this workstation as a clean, repeatable Ansible control node for my DMI DevOps Micro Internship. Ansible can't run as a control node on native Windows, so I built the environment inside Ubuntu on WSL2. The goal is a setup where Ansible and its tools are isolated in one project, linted automatically before every commit, and ready to connect to managed hosts over SSH.

Controller Environment

Host machine:	Windows
Platform:	WSL2 (kernel 6.6.87.2-microsoft-standard-WSL2)
Operating system:	Ubuntu 24.04.5 LTS (Noble Numbat)
Python:	3.12.3
Project location:	~/ansible-onboarding

Installed Tools

Ansible (ansible-core) | 2.21.5 |
| Ansible Lint | 26.10.1 |
| YAML Lint | 1.38.0 |
| Pre-commit | 4.7.0 |
| Git | 2.43.0 |
| OpenSSH | 9.6p1 |

The project uses a Python virtual environment (.venv) so that Ansible and the linting tools are installed inside the project, not system-wide. This keeps tool versions separate from Ubuntu's own Python packages, lets each project pin its own versions through requirements.txt, and means the environment can be deleted and rebuilt at any time without affecting the rest of the system.

To activate it:

bash
cd ~/ansible-onboarding
source .venv/bin/activate

The prompt shows (.venv) while the environment is active. To leave it, run deactivate.

To rebuild it from scratch:

bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt


VS Code Configuration

VS Code is connected to Ubuntu through WSL, and these extensions are installed on the WSL side:

Ansible (redhat.ansible)
YAML (redhat.vscode-yaml)
Python (ms-python.python)
Pylance (ms-python.vscode-pylance)
Python Debugger (ms-python.debugpy)
Python Environments (ms-python.vscode-python-envs)
EditorConfig (editorconfig.editorconfig)
Claude Code (anthropic.claude-code)

The workspace settings in .vscode/settings.json point both the Python extension and the Ansible extension to ${workspaceFolder}/.venv/bin/python. This means VS Code uses the same interpreter, Ansible version and linters as the terminal, so errors in the editor match what pre-commit reports. The settings also turn on Ansible lint validation and YAML validation, trim trailing whitespace on save, and format on save.

Ansible Configuration

ansible.cfg sits in the project root, so Ansible picks it up automatically whenever commands run from this folder. It sets project defaults so they don't need to be typed on every command:

inventory = inventories/ points to the project's inventory folder.
roles_path looks for roles in roles/ and ./.ansible/roles.
retry_files_enabled = False stops .retry files from cluttering the project.
interpreter_python = auto_silent lets Ansible detect Python on managed hosts without warnings.
forks = 10 and timeout = 30 control how many hosts run in parallel and how long to wait for a connection.
YAML-formatted output (callback_result_format = yaml) makes task results easier to read.
Under [ssh_connection], pipelining and SSH connection reuse (ControlMaster/ControlPersist) make runs faster.

Note: host_key_checking = False is set only because this is a controlled training environment where lab hosts are created and destroyed often. In a real or production environment, host-key checking should stay enabled so the controller can detect if a server's identity changes.

SSH Configuration
Private key: ~/.ssh/id_ed25519 (ED25519, permissions 600)
Public key: ~/.ssh/id_ed25519.pub (this is the part copied to managed hosts)
SSH client config: ~/.ssh/config

The private key's contents are never shared, committed or displayed.

SSH agent: the agent holds the unlocked private key in memory for the session. Once the key is added with ssh-add, Ansible and SSH can use it for every connection without asking for the passphrase each time, and the key file doesn't need to be passed on every command.

known_hosts: ~/.ssh/known_hosts stores the fingerprints of servers the controller has connected to before. On later connections, SSH compares the server's fingerprint to the saved one and warns if it has changed, which protects against connecting to an impostor server.

Git and Pre-commit
.gitignore

.gitignore stops local and sensitive files from being committed:

.venv/, so the virtual environment stays local and is rebuilt from requirements.txt instead
__pycache__/, *.pyc and .cache/, which are generated cache files
.ansible/ and *.retry, which are local Ansible files
.DS_Store, an operating-system file
id_rsa, id_ed25519, *.pem and *.key, as a safety net in case a private key is ever copied into the project folder
Pre-commit hooks

Configured in .pre-commit-config.yaml:

Hook	Repository	Version
yamllint	adrienverge/yamllint	v1.38.0
ansible-lint	ansible/ansible-lint	v26.8.0 (set to language_version: python3)
When the hooks run

After pre-commit install, the hooks run automatically every time I run git commit. They check the staged files, and if any hook fails, the commit is blocked until the issues are fixed. They can also be run manually on the whole project at any time:

bash
pre-commit run --all-files
Verification Commands
bash
# Environment
cat /etc/os-release
uname -r
python3 --version

# Ansible and linting tools
ansible --version
ansible-lint --version
yamllint --version

# Pre-commit
pre-commit --version
pre-commit run --all-files

# Git and SSH
git --version
ssh -V

# SSH agent
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh-add -l

ssh-add -l should list the ED25519 key, which confirms that the agent is running and holding the key.

Team-Friendly Feature

The pre-commit configuration is committed to the repository, so every teammate who clones the project gets the same linters at the same pinned versions. After running pre-commit install once, their commits are checked against the same YAML and Ansible rules as mine, without anyone needing to remember to run the linters. Together with requirements.txt and the shared .vscode/settings.json, this means everyone's workstation behaves the same way, and style or syntax problems are caught before they reach the shared repository.

Pitfall Avoided

I avoided installing Ansible globally. Every Ansible and linting tool is installed inside the project's .venv, and .venv/ is listed in .gitignore so it is never committed. This keeps the system Python clean, lets the exact tool versions be rebuilt from requirements.txt, and keeps a large, machine-specific folder out of the repository.

I also moved the project from the Windows drive (/mnt/c/...) into my Linux home folder. On /mnt/c, WSL shows every folder as world-writable, and Ansible refuses to load an ansible.cfg from a world-writable directory. Keeping the project at ~/ansible-onboarding means ansible.cfg is loaded as expected.
