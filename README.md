# Config Server Prometheus bug reproducer

This project is a reproducer for the Spring Cloud Config issue
[spring-cloud/spring-cloud-config#2752](https://github.com/spring-cloud/spring-cloud-config/issues/2752).

## Steps to reproduce

### Application start

First, start the application with the Spring profile `native`.

The particular profile isn't related to the bug, it's just the
easiest to use in a minimal complete verifiable example.

### Http Requests

With `curl` (or any other client) execute the requests

```
curl http://localhost:8080/application/default
```

followed by

```
curl http://localhost:8080/application/default/label
```

The client receives the expected responses.

### Warning in Logs

On the second request, the application will log the following warning:

> The meter (MeterId{name='spring.cloud.config.environment.find.active', tags=[tag(spring.cloud.config.environment.application=application),tag(spring.cloud.config.environment.class=org.springframework.cloud.config.server.environment.SearchPathCompositeEnvironmentRepository),tag(spring.cloud.config.environment.label=label),tag(spring.cloud.config.environment.profile=default)]}) registration has failed: Prometheus requires that all meters with the same name have the same set of tag keys. There is already an existing meter named 'spring.cloud.config.environment.find.active' containing tag keys [spring.cloud.config.environment.application, spring.cloud.config.environment.class, spring.cloud.config.environment.profile]. The meter you are attempting to register has keys [spring.cloud.config.environment.application, spring.cloud.config.environment.class, spring.cloud.config.environment.label, spring.cloud.config.environment.profile]. Note that subsequent logs will be logged at debug level.

The problem is that Prometheus expects tags of a meter to be set consistently.
Currently, the Config Server will only set the label tag when a label is used by the client.

### Dropped observations

When checking the response of the end-point for the meter `spring_cloud_config_environment_find_active`,
the second request with the label is missing from the statistics.