---
nav_title: Getting started with app registration
title: Getting started with app registration
---

## Using the plugin

You can follow the [basic examples](/hub/kong-inc/application-registration/how-to/basic-example/) to enable
your plugin using your preferred method. Or, follow this guide for some specific scenarios.

### Prerequisites

Before using this plugin, [set an authorization provider strategy in `kong.conf`](/gateway/latest/kong-enterprise/dev-portal/applications/auth-provider-strategy/). 

### Enable automatic registration approval

You can enable `auto_approve` so that application registration requests are
automatically approved.

The following example creates an instance of the plugin with `auto_approve` enabled.
Replace `<service>` with your service name or ID, and `<my_service_display_name>` with the
`display_name` of your service:

```
curl -X POST http://localhost:8001/services/<service>/plugins \
    --data "name=application-registration"  \
    --data "config.display_name=<my_service_display_name>" \
    --data "config.auto_approve=true
```

Or, you can update your current configuration with a `PATCH` request.
Replace `<plugin_id>` with the `id` of your plugin.

```
curl -X PATCH http://localhost:8001/plugins/<plugin_id> \
  --data "config.auto_approve=true"
```

### Enable proxy with consumer's credential

Activating application registration will restrict access to the service using consumer credentials. To regain this functionality, you can enable `enable_proxy_with_consumer_credential`. It will introduce
an additional method for accessing the service using the consumer's credential.

```
curl -X POST http://localhost:8001/services/<service>/plugins \
    --data "name=application-registration"  \
    --data "config.display_name=<my_service_display_name>" \
    --data "config.enable_proxy_with_consumer_credential=true"
```

Or, update your current configuration with a `PATCH` request.
Replace `<plugin_id>` with the `id` from the existing application-registration instance.

```
curl -X PATCH http://localhost:8001/plugins/<plugin_id> \
  --data "config.enable_proxy_with_consumer_credential=true"
```

And then, create a plugin `key-auth` and route for the service.

```
curl -X POST http://localhost:8001/services/<service>/routes \
    --data "path=/test"
```

```
curl -X POST http://localhost:8001/services/<service>/plugins \
    --data "name=key-auth" \
    --data "config.key_names=apikey"
```

Create a consumer and a consumer's credential for the `key-auth`.

```
curl -X POST http://localhost:8001/consumers \
    --data "username=test"
```

```
curl -X POST http://localhost:8001/consumers/<consumer_id>/key-auth \
    --data "key:<apikey>"
```

When `enable_proxy_with_consumer_credential` is enabled, constructing a request to the service should yield a 200 response, rather than the 401 response when disabled.

```
curl http://localhost:8000/test?apikey=<apikey>
```

The route should still be accessible with the `client_id` of the Application.

```
curl http://localhost:8000/test?apikey=<client_id>
```

### Enable show issuer URL

Enable `show_issuer` to expose the **Issuer URL** in the **Service Details** dialog.

{:.note}
> **Note:** Exposing the [Issuer URL](/gateway/latest/kong-enterprise/dev-portal/applications/enable-application-registration#show-url-issuer) is essential
for the [Authorization Code Flow](/gateway/latest/kong-enterprise/dev-portal/authentication/3rd-party-oauth/#ac-flow) 
configured for third-party identity providers.

Update your current configuration with a `PATCH` request. Replace `<plugin_id>` with the `id` of your plugin.

```
curl -X PATCH http://localhost:8001/plugins/<plugin_id> \
  --data "config.show_issuer=true"
```
