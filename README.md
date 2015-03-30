# virtualmin-nginx-config-hook

A [Virtualmin](https://www.virtualmin.com/) hook to generate a nginx server config file post-Apache server setup.

## Why?

- [x] You have a dedicated server or a VPS
- [x] You have a LAMP stack beautifully set up and configured
- [x] You use Virtualmin as the web hosting control panel of choice
- [x] You have Apache proxied by a public-facing nginx, because performance
- [x] You have to manually create and manage those nginx `.conf` files after setting up a host in Virtualmin, and
- [x] You ~~are too lazy for this shit~~ feel like your time can be used for something better

## How?
Virtualmin can be configured to run a command (_hook_) before and after making changes to a server. In the command context, a lot of global variables are pre-populated by Virtualmin, ready to use. For example:

```bash
uid=500
name=1
postgres=0
web_port=8000
limit_mysql=1
limit_ssl=0
```

Take a look at `sample-dom-file` for a more complete example. 

All of these variable names will be available with a `$VIRTUALSERVER_` prefix, e.g. `$VIRTUALSERVER_UID`, `$VIRTUALSERVER_DOM` etc. Making use of them, we can write a shellscript to automate creating and managing the nginx config files on the fly.

## No, really, how?
Oh ok...

```bash
git clone git@github.com:phanan/virtualmin-nginx-config-hook.git
chmod +x virtualmin-nginx-config-hook/hook
```

Then, [configure the post domain modification script](https://www.virtualmin.com/documentation/developer/prepost) to point to `hook`'s full path. Now whenever you create or modify a (sub)domain via Virtualmin interface, these things should happen, in that order:

1. A file in the format of `{domain_name}.conf` is generated under `{nginx_conf_dir}/sites-available`
1. A symbolic link pointing to said file is created under `{nginx_conf_dir}/sites-enabled`
1. `nginx` service is then reloaded and ready to serve

When you delete the (sub)domain, both the file and its symlink are removed.

## Configuration
The hook works with the assumption that your nginx is configured to read domain config files from `sites-enabled` and `sites-available` directories, following the (arguably) [most common practice](https://www.digitalocean.com/community/tutorials/how-to-configure-the-nginx-web-server-on-a-virtual-private-server). Additionally, these variables can be configured to fit your needs:

```bash
NGINX_CONF_DIR="/etc/nginx"
NGINX_PORT=80
NGINX_SSL_PORT=443
```

Other than that... not so much, sorry.

## Known Limitations
* Upon Virtualmin domain update, the corresponding `.conf` file's content will be completely reset. If you have any custom modifications, they will be gone into oblivion
* Domain disabling and re-enabling are not supported yet.

<hr>

## Disclaimer and License
MIT © [Phan An](http://phanan.net). 

This package is created and shared with hope that it may help someone out there, but its owner shouldn't take any responsibility for any claim, damages or other liability whatsoever. See `LICENSE`.
