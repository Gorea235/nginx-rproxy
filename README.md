# Nginx Reverse-Proxy Setup & Management
This repo contains file that aid with the setting up of a Nginx-based reverse-proxy that has SSL enable via letsencrypt. The main management script is [proxy-manage](proxy_manager/proxy-manage), and has the config [proxy-manage.conf](proxy_manager/proxy-manage.conf). This script does most of the work, however there are some parts you will need to modify (the script's conf file, and the `vars.*.conf` files under the [nginx config](nginx_config/running/includes)), and there is some interaction, copying, etc you need to do to get the certificates working. The full instructs are below.

## Script usage
(Brief note: the script is designed to be ran from the current directory. If this doesn't work for you, change the variables near the top of the script to an absolute path, and it will then run regardless of the current directory.)

To use this script properly, copy it and the conf file to a directory on the server computer, edit the conf file to your specifications, and run it under sudo with the argument `init-rp`. This will set up the nginx proxy container and the volume directories. After, copy [`nginx_config/setup/*`](nginx_config/setup) into the `$NGINX_VOLUMES/conf.d` directory in order to setup the base certification server, then restart the `$RPROXY_NAME` container to enable it.

Once this is done, run the script under sudo again with the argument `init-cert` in order to initialise the certbot. Once this is done, delete the content of the `$NGINX_VOLUMES/conf.d` folder, and copy in [`nginx_config/running/*`](nginx_config/running), then edit the `var.*.conf` files in the includes folder to the variables of the server, then restart the `$RPROXY_NAME` container to enable it.

Once this is done, the reverse proxy should be working with SSL enabled.

To setup the renew, add something like the following to the sudo crontab:

```
0 0 * * 0,5 (cd <script copy location> && ./proxy-manage renew)
```

This will execute the renew script every Monday & Thursday. The cd command is used to keep the relative paths that this script uses. The log of this renew is stored in `$RENEW_LOG`.