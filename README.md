# Ansible

```
export SERVER_IP=<aa.bb.cc.dd>
export KEYNAME=deploy_key
export CERTBOT_EMAIL=user@example.com
```


Do the initial server provisioning to create the deploy user. NOTE that the
path to the deploy key is relative to the playbook directory containing the
`bootstrap.yml` file, not the current directory

```
ansible-playbook -e "public_key_path=../../${KEYNAME}.pub" \
   -i "${SERVER_IP}," playbooks/bootstrap.yml \
   --user root --ask-pass
```

Add the deploy key to `ssh-agent`

```
ssh-add $KEYNAME
```

Now create an `inventories.ini`, looking like this, where
`<<TOP_LEVEL_DOMAIN>>` is (e.g.) example.com:

```
[webservers]
<<TOP_LEVEL_DOMAIN>> ansible_user=deploy ansible_ssh_private_key_file=<<DEPLOY_KEY>>.pub
```


Specify the sites to create in json format:

```
export SITES='{sites: [{"domain":"example.com"},{"domain":"d1.example.com"}]}'
```

Do the deployment. Change to `certbot_staging=false` to actually install the
certificates once the script is seen working.

```
ansible-playbook -i ../inventories/inventory.ini \
    site.yml \
    -e 'email=${CERTBOT_EMAIL}' \
    -e 'certbot_staging=true' \
    -e 'certbot_auto_renew_cron=true' \
    -e ${SITES}
```
