You are advised to install latest version 4 (`stable` branch). Earlier version 3 (`v3` branch) is **neither supported, nor maintained anymore**. If you are still at `v3`, please upgrade as soon as possible.

# General prerequisites

* Server running Debian or Ubuntu (referred to as managed node from here on).
* Acquaintance with [Ansible](https://docs.ansible.com/). This includes having installed Ansible (and related stuff) on the control machine and met the requirements on the managed node (see: [Intro Installation](https://docs.ansible.com/ansible/intro_installation.html)).
* SSH access as *root* from the control machine to the managed node [using shared keys](https://wiki.debian.org/SSH#Using_shared_keys).
* [Git](https://git-scm.com/) installed on the control machine.

# Installing from scratch

Clone, checkout `origin/stable` and update submodules with Git:
```
git clone --recursive https://github.com/leihs/leihs.git
cd leihs
git checkout origin/stable
git submodule update --init --recursive
```

Create your own Ansible inventory:
```
cp -R zhdk-inventory my-inventory
cp my-inventory/host_vars/zhdk-leihs-prod.yml my-inventory/host_vars/my-host.yml
cp my-inventory/prod-hosts my-inventory/my-hosts
```

Copy your public SSH key:
```
cp ~/.ssh/my-key.pub my-inventory/ssh_keys/my-key.pub
```

Now edit `my-inventory/my-hosts` to have the following content:
```yaml
[leihs_server]
my-host
```

Then edit `my-inventory/host_vars/my-host` to have the following content:
```yaml
ansible_host: my-host-name
ansible_user: root

leihs_send_mails: Yes
db_backup_on_deploy: Yes
db_backup_keep_days: 4
db_backup_nigthly_enabled: Yes

ssh_keys_present:
  - "{{lookup('file', inventory_dir + '/ssh_keys/my-key')}}"

database_dump_restore_file_path: '{{inventory_dir}}/../legacy/tmp/db_production.pgbin'
```

Replace `my-inventory`, `my-hosts`, `my-host`, `my-host-name` and `my-key.pub` according to your taste and needs.

**IMPORTANT**: Do not change the content of `my-inventory/group_vars/all.yml` which is a symlink to `deploy/all.yml`. 

Install with Ansible:
```
ansible-playbook -v -i my-inventory/my-hosts deploy/deploy_play.yml
```

## First admin user and password

You will be provided with a simple GUI interface to create your first admin user and password when you open leihs in the browser.

## Hooking up to LDAP for logins

It is possible to use LDAP for logins. The procedure is described under [[LDAP configuration]].

# Upgrading

From within the `leihs` directory *on the control machine*, checkout the desired version tag and update submodules with Git:
```
git fetch --all
git checkout <tag>
git submodule update --recursive
```

If the control machine does not have the `leihs` directory yet, then do the following:
```
git clone --recursive https://github.com/leihs/leihs.git
cd leihs
git checkout <tag>
git submodule update --recursive
```

Install the new version with Ansible:
```
ansible-playbook -v -i <PATH_TO_YOUR_INVENTORY> deploy/deploy_play.yml
```

# Upgrading from v3

Install the new server from scratch as described [above](https://github.com/leihs/leihs/wiki/Deployment#installing-from-scratch).

Ensure your old `v3` server is at version [3.45.1](https://github.com/leihs/leihs/releases/tag/3.45.1). For this follow our upgrade instructions described [here](https://github.com/leihs/leihs/wiki/Upgrades).

Specify the old and new hosts accordingly. From within the new `leihs` directory, do the following:
```
cp my-inventory/host_vars/leihs-v3-prod.yml my-inventory/host_vars/my-old-host.yml
```

Edit `my-inventory/my-hosts` to have the following content:
```yaml
[leihs_v3_server]
my-old-host

[leihs_server]
my-host
```

Then edit `my-inventory/host_vars/my-old-host.yml` to have the following content:
```yaml
ansible_host: my-old-host-name
ansible_user: root
leihs_v3_root_dir: path-to-leihs-directory-on-my-old-host
leihs_v3_ruby_dir: path-to-rbenv-version-on-my-old-host
leihs_v3_user: leihs-user-on-my-old-host
```
Replace `my-old-host`, `my-old-host-name`, `path-to-leihs-directory-on-my-old-host`, `path-to-rbenv-version-on-my-old-host` and `leihs-user-on-my-old-host` according to your taste and needs.

**IMPORTANT**: Do not change the content of `my-inventory/group_vars/all.yml` which is a symlink to `deploy/all.yml`. 

Export your data from your `v3` server and import it into the new server.
```
ansible-playbook deploy/import-v3-into-v4_play.yml -i my-inventory/my-hosts
```

# Troubleshooting

Having carefully followed the described steps, Ansible playbooks should run smoothly. However, the world is not perfect, so in case of troubles one may take a look at [the possible caveats](https://github.com/leihs/leihs/wiki/Deployment-Caveats).
