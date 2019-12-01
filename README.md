[![Build Status](https://travis-ci.org/wrouesnel/docker.openstf.svg?branch=master)](https://travis-ci.org/wrouesnel/docker.openstf)

# OpenSTF for Standalone Deployment

This is a self-contained container which builds and deploys [OpenSTF](https://openstf.io/).

## New Deployment Instructions

1. Stand up the container.
1. Run database migrations.
1. Create a superuser.

## LDAP Parameters

## WebUI Hooks

The following webui hooks are available to carry out container operations,
protected behind the admin interface and require POST commands:

## Environment Variables

* `HOSTNAME`
  Persistent hostname of this node. If unset defaults to the docker hostname.

* `ADMIN_USER=admin`
  Username of the admin user for accessing online management functions of the
  container (namely, the WebUI of alertmanager and cockroachdb).

* `ADMIN_PASSWORD`
  Password of the admin user for accessing online management functions of the
  container (namely, the WebUI of alertmanager and cockroachdb).

* `WEBUI_DNS_NAME`
  DNS name which the Web UI is being hosted under.

* `WEBUI_SSL_SERVER_CERT=`
  WebUI server identity certificate. May be a literal PEM certificate or a file path
  inside the container.
* `WEBUI_SSL_SERVER_KEY=`
  WebUI server identity key. May be a literal PEM certificate or a file path
  inside the container.
* `WEBUI_SSL_SERVER_CERTCHAIN=`
  WebUI server identity certificate chain. May be a literal PEM certificate or a file path
  inside the container.

* `SMTP_SMARTHOST=`
  `hostname:port` to use for sending alert emails.
* `SMTP_FROM`
  from address to set for from emails. If unset, defaults to `alerts@{{API_DNS_NAME}}`
* `SMTP_USERNAME=`
  username to use to login to the SMTP server.
* `SMTP_PASSWORD=`
  password to use to login to the SMTP server.
  
* `ALERT_EMAIL_ADDRESS=`
  email address to send alerts to.
  
* `RETHINKDB_ADMIN_PASSWORD` initial admin password to generate for rethinkDB. Automatically
  generated if unset.

* `ADB_PRIVATE_KEY` file path or literal ADB private key to set for the ADB server. Automatically
  generated if unset.

* `X_STF_PUBLIC_ADDR=` is the address STF should announce the URLs it is hosted at. If unset, defaults
  to the IP address of the container.

* `X_STF_REAPER_HEARTBEAT_INTERVAL=30000` heartbeat interval passed to the reaper service.

* `X_STF_APP_PORT=3100`

* `X_STF_AUTH_PORT=3200`

* `X_STF_STORAGE_PLUGIN_APK_PORT=3300`

* `X_STF_STORAGE_PLUGIN_IMAGE_PORT=3400`

* `X_STF_STORAGE_TEMP_PORT=3500`

* `X_STF_WEBSOCKET_PORT=3600`

* `X_STF_API_PORT=3700`

* `X_STF_PROVIDER_HEARTBEAT_INTERVAL=10000`

* `X_STF_PROVIDER_MIN_PORT=20000`

* `X_STF_PROVIDER_MAX_PORT=25000`

* `X_STF_APPSIDE_ADDR=127.0.0.1`
  Application-side services.

* `X_STF_DEVSIDE_ADDR=127.0.0.1`
  Device side services.

* `DEV_ALLOW_SELF_SIGNED=no`
  Allows a blank value for `SSL_SERVER_CERT` and `SSL_SERVER_KEY`. This causes
  the container to generate a self-signed certificate on startup. It is not a
  production configuration, but useful for development.
* `DEV_ALLOW_EPHEMERAL_DATA=no`
  Normally the container explicitely disallows ephemeral storge. This option
  overrides the check if set to `yes` to allow `/data` to not be a mountpoint.
* `DEV_STANDALONE=no`
  Disables any SSL client certificate checks and forces the node to trust only
  its local certificate. Useful for testing when using the standalone schema.
* `DEV_ALLOW_DEFAULT_TRUST=no`
  Set to `yes` to allow default platform SSL certificates to be used if
  `PLATFORM_TLS_TRUST_CERTIFICATES` is blank.
* `DEV_NO_ALERT_EMAILS=no`
  Set to `yes` to allow no addresses for sending alert emails to be specified.
* `DEV_NO_SMARTHOST=no`
  Set to `yes` to allow no the SMTP smarthost to be blank.
* `DEV_NO_SMARTHOST_TLS=no`
  Set to `yes` to disable TLS for smarthosts.
* `DEV_ENTRYPOINT_DEBUG=no"
  Set to `yes` to set `bash -x` on the entrypoint. 


## Hacking

`make run-it` to startup the container with most dev options enabled.
`make enter-it` to override the entrypoint and get a shell.
`make exec-into` will try and start a shell in an already running container.
