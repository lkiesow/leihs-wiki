You are advised to install latest version 4 (`stable` branch). Earlier version 3 (`v3` branch) is **neither supported, nor maintained anymore**. If you are still at `v3`, please upgrade as soon as possible.

# General prerequisites

* Server running Debian or Ubuntu (referred to as managed node from here on).
* Acquaintance with [Ansible](https://docs.ansible.com/). This includes having installed Ansible (and related stuff) on the control machine and met the requirements on the managed node (see: [Intro Installation](https://docs.ansible.com/ansible/intro_installation.html)).
* SSH access as *root* from the control machine to the managed node (public/private key-pair setup).

# Installing from scratch

See [Leihs Hosting Guide](https://github.com/leihs/leihs-instance/blob/master/GUIDE.md)
for detailed instructions on how to setup leihs for the first time, as well as upgrading.


# Troubleshooting

Having carefully followed the described steps, Ansible playbooks should run smoothly. However, the world is not perfect, so in case of troubles one may take a look at [the possible caveats](https://github.com/leihs/leihs/wiki/Deployment-Caveats).
