You are advised to install `v4`. Earlier version `v3` is **neither supported, nor maintained anymore**. If you are still at `v3`, please upgrade as soon as possible.

# Installing v4 from scratch

## Prerequisites

* Debian / Ubuntu server.
* Acquaintance with [Ansible](https://docs.ansible.com/).
* SSH key setup on the server (root) and on the client running ansible playbooks.

## Procedure

1. Clone, checkout `v4` and update submodules with Git:
```
git clone --recursive https://github.com/leihs/leihs.git
cd leihs
git checkout origin/v4
git submodule update --init --recursive
```

2. Create your own Ansible inventory. See our example in `zhdk-inventory`.
3. Install with Ansible:
```
ansible-playbook -v -i <YOUR_OWN_INVENTORY> deploy_play.yml
```

### First admin user and password

You will be provided with a simple GUI interface to create your first admin user and password when you open leihs in the browser.

### Hooking up to LDAP for logins

It is possible to use LDAP for logins. The procedure is described under [[LDAP configuration]].

# Upgrading v4

1. Checkout the desired version tag and update submodules with Git:
```
git fetch
git checkout <tag>
git submodule update --recursive
```

2. Install the new version with Ansible:
```
ansible-playbook -v -i <YOUR_OWN_INVENTORY> deploy_play.yml
```

# Upgrading from v3 to v4

1. Install the new `v4` server as described above.

2. Ensure your `v3` server is at version [3.45.1](https://github.com/leihs/leihs/releases/tag/3.45.1). For this follow our upgrade instructions described [here](https://github.com/leihs/leihs/wiki/Upgrades).

3. Export your data from your `v3` server and import it into the new `v4` server:
```
ansible-playbook import-v3-into-v4_play.yml -i <YOUR_OWN_INVENTORY>
```
