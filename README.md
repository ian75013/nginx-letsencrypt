# Nginx + LetsEncrypt on Docker

## How nginx is initially setup

1. `EMAIL` environment variable specifies the email address that will recieve the notifications when there is a renewal needed
2. `SERVERS` environment variable specifies a space separated list of FQDNs for the certificate.  It is expected that the first one would be primary and will be put in the `/etc/letsencrypt/live` folder.
3. When the `/etc/letsencrypt/live folder` is missing the `init` script will start up `certbot` in standalone mode.  This will create the necessary files to enable SSL.  Otherwise we would require two configuration files for nginx (one with and one without SSL).

# Customization points

`/etc/conf.d` is expected to contain the virtual server specific configurations.  The default configuration will simply `return 204` for every request except for the `/.well-known/acme-challenge` URI which points to `/tmp/.well-known/acme-challenge` and that provides the challenges required by LetsEncrypt for renewals.

`/etc/stream.d` is expected to contain stream specific configurations which are useful for passing the request / response as is to an upstream server.  This is an example of an upstream server called `intranet` which the nginx server will route to if the request is for `i.trajano.net` the `default default_https` is needed to make it do the normal processing specified in the previous paragraph.

    upstream intranet {
        server intranet:443;
    }

    map $ssl_preread_server_name $upstream {
        default default_https;
        i.trajano.net intranet;
    }
