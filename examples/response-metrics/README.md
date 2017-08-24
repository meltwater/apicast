# Reporting metrics on responses

APIcast supports metrics on requests out or the box. However, sometimes you need to capture metrics on the responses you're returning to the user.

To carry this out, you'll need to capture data from the response on it's way out of the 3scale gateway.

This is possible by creating a custom module and overriding the `post_action` function of `apicast`.

## How it works

There are two code snipped that do the following:

1. `apicast_response_metrics.lua` is a custom module (see [this example](https://github.com/3scale/apicast/tree/master/examples/custom-module) for more info) that overrides the default APIcast's post_action phase handler to include the following logic:
  * It checks if the request was for a specific path, the path we wish to collect metrics on - in this case `/v1/documents`
  * It extracts a custom header `x-document-count` from the response (which was added in the application code)
  * It calls a custom path `/report_metric`, passing the `document_count` and `user_key` to a record the metric
2. `response_metrics.conf` is a configuration file that should be added to `apicast.d` directory to be included in the configuration.
  * It contains `/report_metric` that is used to make the `POST` request to report custom metrics to 3scale

## How it works

You'll need to set up a custom metric in your 3scale Admin Portal. See the 3scale documentation on how to [Create new metric](https://support.3scale.net/docs/access-control/api-definition-methods-metrics).


## Adding the customization to APIcast

**Note:** the example commands are supposed to be run from the root of the local copy of the `apicast` repository.

### Native APIcast

Place `apicast_response_metrics.lua` to `apicast/src`, and `response_metrics.conf` to `apicast/apicast.d` and start APIcast:

```
THREESCALE_PORTAL_ENDPOINT=https://ACCESS-TOKEN@ACCOUNT-admin.3scale.net APICAST_MODULE=apicast_response_metrics bin/apicast
```

### Docker

Attach the above files as volumes to the container and set `APICAST_MODULE` environment variable.

```
docker run --name apicast --rm -p 8080:8080 -v $(pwd)/examples/response-metrics/apicast_response_metrics.lua:/opt/app-root/src/src/apicast_response_metrics.lua:ro -v $(pwd)/examples/response-metrics/response_metrics.conf:/opt/app-root/src/apicast.d/response_metrics.conf:ro -e THREESCALE_PORTAL_ENDPOINT=https://ACCESS-TOKEN@ACCOUNT-admin.3scale.net -e APICAST_MODULE=apicast_response_metrics quay.io/3scale/apicast:master
```
