
# Overview

 The World is imperfect, non-ideal. Maybe (problably?) your IT infrastructure is like that
 too. Infra as code can help improve that, but sometimes its tools/practices/users call for
 ideals you or your environment are not ready to meet right now.

 Ansible keeps things simple. And it helps you automate the things the way you do it today,
 whether it's a good or bad way. I think Ansible is better at theses than others.

 This is an imperfect example of imperfect automation for an imperfect environment. But one
 that you can improve by codifying and automating your infra RIGHT NOW.

 **Warning**: this ain't a complete or optimized BestPractical RT installation example. It's
  purpose is showcasing some ansible features/possibilities.

# Interesting things to note in here

 - This playbook is meant to be used interactively. It prompts for some info (vars_propmpt).
   But you can override that setting the var with --extra_vars

 - It sets several variables right upfront. Very readable. They can reference other variables,
   including the prompted ones.

 - I use ready roles I got from ansible galaxy to install mysql and nginx.
 - I then do my own stuff with tasks (not creating a role for that, that's my choice), but
   break them in other files (includes), mostly for readability/maintenability.
 - One of those includes even contains an attrocious configure/make/make install.

 - I use the script module to upload and run some hipothetically useful script I would have
   ready, so I would not need to invest time right now converting it to an ansible playbook.

 - For the RT user password, we generate a random one in the control machine and keep it saved
   to a file on the remote host, instead of keeping in our code. We then always read that file
   so we don't generate a new password in every run. THIS IS UGLY AS HELL. If I were to use this
   approach for real, we could at least make a custom python module to enable a cleaner code.
   But hey, use ansible vault or some lookup on a remote shared secret vault.

# Better ways

- Creating packages is easy with jordansissel/fpm. Avoid doing configure/make/make-install if
  possible.

- Place vars in groups_vars, preferably. If they are really host-bound, in the inventory itself
  then.

- For reusability, prefer creating roles, instead of including files.

- Use Ansible Vault for secrets (you can try it here), or some remote shared secret storage
  like Hashicorp's Vault, acessing it though a lookup plugin like hashi_vault or
  https://github.com/jhaals/ansible-vault.

# Examples

Using this vagrant repo, you can try it like this below:

```
 vagrant up
 ansible-playbook --sudo rt-server.yml \
   -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory
```

Calling it like this bellow will ask only for prompt_mysql_root_password (but you could set
that too, of course)

```
 ansible-playbook --sudo rt-server.yml \
   -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory \
   --extra-vars "prompt_mysql_bind_address=127.0.0.1 prompt_rt_version=4.4.0"
```

Using Ansible Vault for storing secrets (vault password is p@ssword):

```
 ansible-vault view encrypted-vars.yml
 ansible-vault edit encrypted-vars.yml
 ansible-playbook --sudo rt-server.yml --ask-vault-pass \
   -i .vagrant/provisioners/ansible/inventory/vagrant_ansible_inventory \
   --extra-vars "prompt_mysql_bind_address=127.0.0.1 prompt_rt_version=4.4.0 use_vault=true"
```
