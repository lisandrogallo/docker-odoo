docker-odoo
===========

Dockerfile and configuration steps to launch Odoo version 8.0 container linked to a PostgreSQL container.

## Instalation

Install Docker following these steps (Ubuntu): http://docs.docker.com/installation/ubuntulinux/

## Configuration

Create a local directory to store configuration and data for the PostgreSQL container.
Run PostgreSQL container on a separate terminal:

    sudo docker run \
        --rm \
        --tty \
        --interactive \
        --name postgres \
        --volume /local_directory/data:/var/lib/postgresql/data \
        --volume /local_directory/log:/var/log/postgresql postgres:9.3

Link to PostgreSQL container and create 'odoo' user. A prompt will ask for a password to set:

    sudo docker run \
        --interactive \
        --tty \
        --link postgres:postgres \
        --rm postgres \
        bash -c 'exec createuser \
        -U postgres \
        -h "$POSTGRES_PORT_5432_TCP_ADDR" \
        -p "$POSTGRES_PORT_5432_TCP_PORT"\
        --pwprompt odoo \
        --createdb'

Build the Odoo image using the Dockerfile included on this repo:

    cd docker-odoo/
    sudo docker build -t odoo_server:8.0 .

Launch Odoo server connected to Postgres instance:

    sudo docker run \
        --rm \
        --interactive \
        --tty \
        --name odoo \
        --link postgres:odoo_database \
        --publish=8069:8069 \
        odoo_server:8.0
    
Then you can access Odoo at the following URL: http://localhost:8069

These containers will be deleted when they stop running. If you need to run them as daemons replace the **--interactive** and **--tty** options with **--detach**.

## Development

For using this docker image for development, you have to execute:

    sudo docker run \
        --rm \
        --interactive \
        --tty \
        --name odoo \
        --link postgres:odoo_database \
        --volume /local_module:/opt/odoo/sources/addons \
        --publish=8069:8069 \
        odoo_server:8.0

To update all modules on a specific database, yo have to execute:

    sudo docker run \
        --rm \
        --interactive \
        --tty \
        --name odoo \
        --link postgres:odoo_database \
        --volume /local_module:/opt/odoo/sources/addons \
        --publish=8069:8069 \
        odoo_server:8.0
        sudo -H -u odoo /opt/odoo/server/odoo.py \
        -c /opt/odoo/server/odoo.conf \
        --update=all -d database_name

For using a local Odoo configuration file, you have to execute:

    sudo docker run \
        --rm \
        --interactive \
        --tty \
        --name odoo \
        --link postgres:odoo_database \
        --volume /local_module:/opt/odoo/sources/addons \
        --volume /local_path/odoo.conf:/opt/odoo/odoo.conf \
        --publish=8069:8069 \
        odoo_server:8.0

## Troubleshooting

If you get the following error:

    OperationalError: FATAL:  no pg_hba.conf entry for host "172.17.0.7", user "odoo", database "postgres", SSL off

You can override it by adding this line to the *pg_hba.conf* file:

    echo "host  all  all  172.17.0.0/24  trust" >> /local_directory/data/pg_hba.conf
