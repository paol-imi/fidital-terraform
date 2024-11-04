# Task definitions

## Relations

One microservices version is binded to exactly one image, and that image is binded to more task definition to provide different configurations based on

- Customer (acarpia, fidital...)
- Environment (uat, prod...)

```txt
Microservice <--1to1--> Image <--1toN--> task definitions
```

**`TODO:`** Find a way to have a single source of truth for the common configurations that do not depend on [Customer, Environment]. (Most likely terraform is the answer)

### Naming conventions

```txt
ENVIRONMENT-CUSTOMER-MICROSERVICE_NAME
```

**`TODO:`** should we change it? what makes a convention good?

**`TODO:`** refactor old definitions

## Configuration

Task definitions combine a microservices image on ecr with:

- env variables
- ports binding
- resources expected usage (memory, cpu...)
- aws related settings

### Version

Each combination of [Customer, Environment] determine the image version to pick.

**`TODO:`** define a convention.

### Env Variables

In general, variables that depends on the [Customer, Environment] should not be binded to that in the application.yml, e.g. `UAT-ACARPIA-VAR` should just be `VAR`. This not only remove duplicates and bad abstractions, but also enable proper reuse of configurations.

**`TODO:`** Refactor all the spring boot services, for now even the new ones have this convention to ease the first deployment.

#### Port

In local environments, web servers must bind their port to a non-default one to avoid collisions. This is not an issue in a cloud setting, so each container should just map to the 8080 port. **New task definitions will do so and ld ones should be refactored.**

**`TODO:`** Which standard to use in the application.yml? e.g. `port: ${APPLICATION_PORT:8081}`? Where `APPLICATION_PORT` is set in the cloud environment and `8081` is the fallback for local testing?

#### Cloudwatch logs

**`TODO:`**: Logs should be grouped by Environment (e.g. distinguish prod by uat) other than by Customer. Refactor old tasks.
