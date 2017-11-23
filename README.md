# Nginx Reverse-Proxy Setup & Management
This repo contains files that aid with the setting up of an Nginx-based reverse-proxy that has SSL enable via letsencrypt. The main management script is [`proxy-manage`](proxy_manager/proxy-manage), and has the config [`.env`](proxy_manager/.env). This script does most of the work, however there are some parts you will need to modify (the script's env file, the [docker compose](proxy_manager/docker-compose.rproxy.yml) file, [`nginx.def.conf`](nginx_config/setup/nginx.def.conf), and [`nginx.conf`](nginx_config/running/nginx.conf)), and there is some interaction, copying, etc you need to do to get the certificates working. The full instructs are below.

## Pre-Requisites
The script requires docker-compose to be installed under root (this can be done through pip). The compose file is version 3.4, however it will probably work with lower versions (however I haven't tested this).

## Script usage
(Brief note: the script is designed to be ran from the current directory. If this doesn't work for you, change the variables near the top of the script to an absolute path, and it will then run regardless of the current directory.)

To use this script properly, copy it and the env file to a directory on the server computer, edit the env file to your specifications, and run it under sudo with the argument `init-rp`. This will set up the nginx proxy container, using the docker-compose file, and the volume directories. After, copy [`nginx_config/setup/*`](nginx_config/setup) into the `$NGINX_VOLUMES/conf.d` directory in order to setup the base certification server, then restart the `$RPROXY_NAME` container to enable it.

Once this is done, run the script under sudo again with the argument `init-cert` in order to initialise the certbot. After, set up a docker network for the webserver and start up the webserver that you are proxying into the network. Edit the docker-compose file to enable the network options, and set the network name to the network the webserver is operation in. Then run the script with the argument `update-rp` in order to update the rproxy container. (This argument can be used to just update the nginx container version as well, since it automatically pulls the latest alpine image and prunes un-needed images (if you don't want this prune, disable that line before using this script))

Once this is done, delete the content of the `$NGINX_VOLUMES/conf.d` folder, and copy in [`nginx_config/running/nginx.conf`](nginx_config/running/nginx.conf), and edit it to suit your server configuration. You can then restart the `$RPROXY_NAME` container to enable it (you can use the `update-rp` argument to achieve this).

The normal [`nginx.conf`](nginx_config/running/nginx.conf) file has a default HTTP and HTTPS server config set up, however you should only need to modify the proxy name, webserver hostname, and SSL cert locations, since the HTTP server is designed to accept request for any domain (and it will process all ACME requests through this). 

You can also run more than 1 domain at once by copying the HTTPS server and proxy config and changing the server name (you will also have to init-cert with these extra domains, however if the rproxy container is running, you should be able to just re-run the argument with an updated [`.env`](proxy_manager/.env), just ensure that all domains that are being run are specified in this config file, as it is used for renewal).

When you have done, the reverse proxy should be working with SSL enabled.

## Renewal
To setup the renew, add something like the following to the sudo crontab:

```
0 0 * * 0,5 (cd <script copy location> && ./proxy-manage renew)
```

This will execute the renew script every Monday & Thursday. The cd command is used to keep the relative paths that the script uses. The log of this renew is stored in `$RENEW_LOG`.