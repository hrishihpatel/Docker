
### Contents

[Spring Cloud Config Pull](#spring-cloud-pull)<br/>


<a name=spring-cloud-pull></a>
# Spring Cloud Config Pull

```
docker pull hyness/spring-cloud-config-server
```

# Configuing Spring Clod Config Server
Spring Cloud Config Server is a normal Spring Boot appication, it can be configured through all the way a Spring Boot application can be configured. You may use environment variables or you can mout configuration in the provided volume. The configuration file must be named **application** and may be a properties or yamle file.

# Configuration Example

## Git 

```
docker run -it -p 8888:8888 -e SPRING_CLOUD_CONFIG_SERVER_GIT_URI=<GitURL> -name springcloud hyness/spring-cloud-config-server
```

