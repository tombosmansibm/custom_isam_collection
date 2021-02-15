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