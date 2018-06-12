# Upgrading from v3 to v4

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
