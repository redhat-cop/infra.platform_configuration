# Ansible Role ansible.gateway_configuration.services

## Description

An Ansible Role to configure gateway API routes (called Service) on Ansible Automation gateway.
They define http port and path (starting with prefix /api/) used in gateway and
http port and path in the destination service (gateway, controller, hub, eda).

## Variables

Detailed description of variables are provided in the [top-level README](../../README.md)

Variables specific for this role are following:

| Variable Name                                     |                    Default Value                    | Required | Description                                                                                                                                                    |                                                      |
|:--------------------------------------------------|:---------------------------------------------------:|:--------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------:|
| `gateway_services` (Alias: services)              |             [below](#service-arguments)             |   yes    | Data structure describing your service entries described below.                                                                                                |        [more](../../README.md#data-variables)        |
| `gateway_configuration_services_secure_logging`   |  `gateway_configuration_secure_logging` OR `false`  |    no    | Whether or not to include the sensitive service role tasks in the log. Set this value to `True` if you will be providing your sensitive values from elsewhere. |   [more](../../README.md#secure-logging-variables)   |
| `gateway_configuration_services_enforce_defaults` | `gateway_configuration_enforce_defaults` OR `false` |    no    | Whether or not to enforce default option values on only the service role.                                                                                      |      [more](../../README.md#enforcing-defaults)      |
| `gateway_configuration_services_async_retries`    |    `gateway_configuration_async_retries` OR `30`    |    no    | This variable sets the number of retries to attempt for the role.                                                                                              | [more](../../README.md#asynchronous-retry-variables) |
| `gateway_configuration_services_async_delay`      |     `gateway_configuration_async_delay` OR `1`      |    no    | This sets the delay between retries for the role.                                                                                                              | [more](../../README.md#asynchronous-retry-variables) |

## Data Structure

### Service Arguments

Options for the `gateway_services` variable:

| Variable Name         |    Default Value    | Required | Type | Description                                                                                                                                       |
|:----------------------|:-------------------:|:--------:|:----:|:--------------------------------------------------------------------------------------------------------------------------------------------------|
| `name`                |         N/A         |   yes    | str  | The name of the api                                                                                                                               |
| `new_name`            |         N/A         |    no    | str  | Setting this option will change the existing name (looked up via the name field)                                                                  |
| `description`         |         ""          |    no    | str  | Description of the service                                                                                                                        |
| `api_slug`            |         ""          |    no    | str  | URL slug for the gateway API path for the Controller, Hub and EDA services (gateway API route requires value "gateway", but the slug is not used) |
| `http_port`           |         N/A         |    no    | str  | ID or name referencing the [Http Port](../http_ports/README.md)                                                                                   |
| `service_cluster`     |         N/A         |    no    | str  | ID or name referencing the [Service Cluster](../service_clusters/README.md)                                                                       |
| `is_service_https`    |       `false`       |    no    | bool | Flag whether or not the service cluster uses https                                                                                                |
| `enable_gateway_auth` | N/A (`true` by API) |    no    | bool | If false, the AAP gateway will not insert a gateway token into the proxied request                                                                |
| `service_path`        |         ""          |    no    | str  | URL path on the AAP Service cluster to route traffic to                                                                                           |
| `service_port`        |         N/A         |    no    | int  | Port on the service cluster to route traffic to                                                                                                   |
| `order`               |  "" (`50` by API)   |    no    | int  | The order to apply the routes in lower numbers are first. Items with the same value have no guaranteed order                                      |
| `tags`                |         ""          |    no    | str  | Comma-separated string, selects which (tagged) nodes receive traffic from this route                                                              |
| `state`               |      `present`      |    no    | str  | [more](../../README.md#state-variable)                                                                                                            |

**Unique value:**

- `name`
- `http_port` + `service_cluster`

**Note**: field `gateway_path` is inferred from the `api_slug`, always starts with `/api/` and is read only.

## Usage

#### Json Example

- Check that Controller API Route exists
- Create or update gateway API Route on proxy port (http port) with id 1 and path '/' proxying gateway on path '
  /api/v1/' and port 9000
- Create or update EDA API Route on proxy port (http port) 8000 and path '/api/eda/' proxying Event Driven
  Automation on path '/api/v2/' and port 9000. Lookup for existing name "EDA API", but create/update with different name

```json
{
  "gateway_services": [
    {
      "name": "Controller API",
      "state": "exists"
    },
    {
      "name": "gateway API",
      "http_port": 1,
      "service_cluster": "gateway",
      "service_path": "/api/v1/",
      "service_port": 9000
    },
    {
      "name": "EDA API",
      "new_name": "Event Driven Automation API",
      "http_port": "Port 8000",
      "api_slug": "eda",
      "service_cluster": "eda",
      "service_path": "/api/v2/",
      "service_port": 9000
    }
  ]
}
```

### Yaml Example

- Remove all gateway Services (resp. their proxy configurations)

File name: `data/gateway_services.yml`

```yaml
---
gateway_state: absent
gateway_service_clusters:
- name: Controller API
- name: Hub API
- name: EDA API
- name: Gateway API
```

## Run Playbook

File name: [manage_data.yml](../../README.md#example-ansible-playbook) can be found in the top-level README.

```shell
ansible-playbook manage_data.yml -e @data/gateway_services.yml
```

## License

[GPLv3](https://github.com/redhat-cop/infra.platform_configuration/blob/main/COPYING)
