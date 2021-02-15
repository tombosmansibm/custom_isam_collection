# Install AWX

## Docker

I've followed the instruction here https://github.com/ansible/awx/blob/devel/INSTALL.md#docker-compose

The only important thing is that you must prepare the virtual environment directory.
> # AWX custom virtual environment folder. Only usable for local install.
> custom_venv_dir=/data/venv/

I hope this allows you to prepare the virtual environment to run Ansible in, on your machine.
The AWX installation that runs in Docker should then pick it up.




# Start stop the awx
The docker compose result ends up here:

cd ~/.awx/awxcompose/

In that directory, you can run the "docker-compose" commands.
docker-compose logs


# Collections

There's problems with installing collections on AWX/Tower using requirements.yml at least for me , on version 17.0.1 with Ansible 2.9.17.

```
"stderr_lines": [
    "ERROR! Invalid collection name 'git@github.com:tombosmansibm/custom_isam_collection.git', name must be in the format <namespace>.<collection>. Please make sure namespace and collection name contains characters from [a-zA-Z0-9_] only."
```

While it works fine on Ansible 2.10+ to use the git url as "name", it does not on AWX/Tower.

So I followed instructions here to create a local galaxy server:
https://galaxy.ansible.com/docs/developers/contributing.html#setting-up-your-development-environment

This results in a local galaxy server running on port 8000: http://localhost:8000/

## Add namespace
To be able to publish on the public Galaxy server, you need to request a namespace first.

On the local machine, you can obviously create it yourself.

## Publish collection

Create an ansible.cfg file , for instance in your home directory (touch ~/.ansible.cfg)

> [galaxy]
> server_list = local_galaxy, release_galaxy
> 
> [galaxy_server.release_galaxy]
> url=https://galaxy.ansible.com/
> 
> [galaxy_server.local_galaxy]
> url=http://localhost:8000/
> token=<token>

````
ansible-galaxy collection publish /home/tbosmans/ansible/ansible_collections/custom/isam/custom-isam-1.0.15.tar.gz
````