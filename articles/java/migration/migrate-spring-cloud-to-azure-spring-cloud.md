---
title: Migrate Spring Cloud applications to Azure Spring Cloud
description: This guide describes what you should be aware of when you want to migrate an existing Spring Cloud application to run on Azure Spring Cloud.
ms.author: karler
ms.topic: conceptual
ms.date: 02/09/2022
ms.custom: devx-track-java
recommendations: false
zone_pivot_group_filename: java/java-zone-pivot-groups.json
zone_pivot_groups: spring-cloud-tier-selection
---

# Migrate Spring Cloud applications to Azure Spring Cloud

This guide describes what you should be aware of when you want to migrate an existing Spring Cloud application to run on Azure Spring Cloud.

## Pre-migration

To ensure a successful migration, before you start, complete the assessment and inventory steps described in the following sections.

If you can't meet any of these pre-migration requirements, see the following companion migration guides:

* Migrate executable JAR applications to containers on Azure Kubernetes Service (guidance planned)
* Migrate executable JAR Applications to Azure Virtual Machines (guidance planned)

### Inspect application components

[!INCLUDE [determine-whether-and-how-the-file-system-is-used-azure-spring-cloud](includes/determine-whether-and-how-the-file-system-is-used-azure-spring-cloud.md)]

#### Determine whether any of the services contain OS-specific code

[!INCLUDE [determine-whether-your-application-contains-os-specific-code](includes/determine-whether-your-application-contains-os-specific-code-no-title.md)]

[!INCLUDE [switch-to-a-supported-platform-azure-spring-cloud](includes/switch-to-a-supported-platform-azure-spring-cloud.md)]

[!INCLUDE [identify-spring-boot-versions](includes/identify-spring-boot-versions.md)]

For any applications using Spring Boot 1.x, follow the [Spring Boot 2.0 migration guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide) to update them to a supported Spring Boot version. For supported versions, see the [Spring Boot and Spring Cloud versions](/azure/spring-cloud/how-to-prepare-app-deployment#spring-boot-and-spring-cloud-versions) section of [Prepare an application for deployment in Azure Spring Cloud](/azure/spring-cloud/how-to-prepare-app-deployment).

#### Identify Spring Cloud versions

Examine the dependencies of each application you're migrating to determine the version of the Spring Cloud components it uses.

##### Maven

In Maven projects, the Spring Cloud version is typically set in the `spring-cloud.version` property:

```xml
  <properties>
    <java.version>1.8</java.version>
    <spring-cloud.version>Hoxton.SR3</spring-cloud.version>
  </properties>
```

##### Gradle

In Gradle projects, the Spring Cloud version is typically set in the "extra properties" block:

```gradle
ext {
  set('springCloudVersion', "Hoxton.SR3")
}
```

You'll need to update all applications to use supported versions of Spring Cloud. For a list of supported versions, see the [Spring Boot and Spring Cloud versions](/azure/spring-cloud/how-to-prepare-app-deployment#spring-boot-and-spring-cloud-versions) section of [Prepare an application for deployment in Azure Spring Cloud](/azure/spring-cloud/how-to-prepare-app-deployment).

[!INCLUDE [identify-logs-metrics-apm-azure-spring-cloud.md](includes/identify-logs-metrics-apm-azure-spring-cloud.md)]

#### Identify Zipkin dependencies

Determine whether your application has explicit dependencies on Zipkin. Look for dependencies on the `io.zipkin.java` group in your Maven or Gradle dependencies.

### Inventory external resources

Identify external resources, such as data sources, JMS message brokers, and URLs of other services. In Spring Cloud applications, you can typically find the configuration for such resources in one of the following locations:

* In the *src/main/directory* folder, in a file typically called *application.properties* or *application.yml*.
* In the Spring Cloud Config repository identified in the previous step.

[!INCLUDE [inventory-databases-spring-boot](includes/inventory-databases-spring-boot.md)]

[!INCLUDE [identify-jms-brokers-in-spring](includes/identify-jms-brokers-in-spring.md)]

After you've identified the broker or brokers in use, find the corresponding settings. In Spring Cloud applications, you can typically find them in the *application.properties* and *application.yml* files in the application directory, or in the Spring Cloud Config server repository.

[!INCLUDE [jms-broker-settings-examples-in-spring](includes/jms-broker-settings-examples-in-spring.md)]

[!INCLUDE [identify-external-caches-azure-spring-cloud](includes/identify-external-caches-azure-spring-cloud.md)]

#### Identity providers

Identify all identity providers and all Spring Cloud applications that require authentication and/or authorization. For information on how identity providers may be configured, consult the following:

* For OAuth2 configuration, see the [Spring Cloud Security quickstart](https://spring.io/projects/spring-cloud-security).
* For Auth0 Spring Security configuration, see the [Auth0 Spring Security documentation](https://auth0.com/docs/quickstart/backend/java-spring-security5/01-authorization).
* For PingFederate Spring Security configuration, see the [Auth0 PingFederate instructions](https://auth0.com/authenticate/java-spring-security/ping-federate/).

#### Resources configured through VMware Tanzu Application Service (TAS) (formerly Pivotal Cloud Foundry)

For applications managed with TAS, external resources, including the resources described earlier, are often configured via TAS service bindings. To examine the configuration for such resources, use the [TAS (Cloud Foundry) CLI](https://docs.cloudfoundry.org/cf-cli/) to view the `VCAP_SERVICES` variable for the application.

```bash
# Log into TAS, if needed (enter credentials when prompted)
cf login -a <API endpoint>

# Set the organization and space containing the application, if not already selected during login.
cf target org <organization name>
cf target space <space name>

# Display variables for the application
cf env <Application Name>
```

Examine the `VCAP_SERVICES` variable for configuration settings of external services bound to the application. For more information, see the [TAS (Cloud Foundry) documentation](https://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html#VCAP-SERVICES).

#### All other external resources

It isn't feasible for this guide to document every possible external dependency. After the migration, it's your responsibility to verify that you can satisfy every external dependency of your application.

[!INCLUDE [inventory-configuration-sources-and-secrets-spring-cloud](includes/inventory-configuration-sources-and-secrets-spring-cloud.md)]

[!INCLUDE [inspect-the-deployment-architecture-spring-cloud](includes/inspect-the-deployment-architecture-spring-cloud.md)]

## Migration

### Remove explicit configuration server settings

In the services you're migrating, find any explicit assignments of Eureka settings and remove them. Such settings typically appear in *application.properties* or *application.yml* files:

**application.yml**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://myusername:mysecretpassword@localhost:8761/eureka/
```

If a setting like this appears in your application configuration, remove it. Azure Spring Cloud will automatically inject the connection information of its configuration server.

### Create an Azure Spring Cloud instance and apps

Provision an Azure Spring Cloud instance in your Azure subscription. Then, provision an app for every service you're migrating. Don't include the Spring Cloud registry and configuration servers. Do include the Spring Cloud Gateway service. For instructions, see [Quickstart: Deploy your first application to Azure Spring Cloud](/azure/spring-cloud/quickstart).

::: zone pivot="sc-standard-tier"

### Prepare the Spring Cloud Config server

Configure the configuration server in your Azure Spring Cloud instance. For more information, see [Set up a Spring Cloud Config Server instance for your service](/azure/spring-cloud/how-to-config-server).

> [!NOTE]
> If your current Spring Cloud Config repository is on the local file system or on premises, you'll first need to migrate or replicate your configuration files to a private cloud-based repository, such as GitHub, Azure Repos, or BitBucket.

[!INCLUDE [ensure-console-logging-and-configure-diagnostic-settings-azure-spring-cloud](includes/ensure-console-logging-and-configure-diagnostic-settings-azure-spring-cloud.md)]

[!INCLUDE [configure-persistent-storage-azure-spring-cloud](includes/configure-persistent-storage-azure-spring-cloud.md)]

::: zone-end

::: zone pivot="sc-enterprise-tier"

### VMware Tanzu components

In Enterprise tier, Application Configuration Service for VMware Tanzu® is provided to support externalized configuration for your apps. Managed Spring Cloud Config Server isn't available in Enterprise tier and is only available in Standard and Basic tier of Azure Spring Cloud.

#### Application Configuration Service for Tanzu

[Application Configuration Service for Tanzu](https://docs.pivotal.io/tcs-k8s/0-1/) is one of the commercial VMware Tanzu components. Application Configuration Service for Tanzu is Kubernetes-native, and totally different from Spring Cloud Config Server. Application Configuration Service for Tanzu enables the management of Kubernetes-native ConfigMap resources that are populated from properties defined in one or more Git repositories.

In Enterprise tier, there's no Spring Cloud Config Server, but you can use Application Configuration Service for Tanzu to manage centralized configurations. For more information, see [Use Application Configuration Service for Tanzu](/azure/spring-cloud/how-to-enterprise-application-configuration-service)

To use Application Configuration Service for Tanzu, do the following steps for each of your apps:

1. Add an explicit app binding to declare that your app needs to use Application Configuration Service for Tanzu.

   > [!NOTE]
   > When you change the bind/unbind status, you must restart or redeploy the app to make the change take effect.

1. Set config file patterns. Config file patterns enable you to choose which application and profile the app will use. For more information, see the [Pattern](/azure/spring-cloud/how-to-enterprise-application-configuration-service#pattern) section of [Use Application Configuration Service for Tanzu](/azure/spring-cloud/how-to-enterprise-application-configuration-service).

   Another option is to set the config file patterns at the same time as your app deployment, as shown in the following example:

   ```azurecli
      az spring-cloud app deploy \
          --name <app-name> \
          --artifact-path <path-to-your-JAR-file> \
          --config-file-pattern <config-file-pattern>
   ```

Application Configuration Service for Tanzu runs on Kubernetes. To help enable a transparent local development experience, we provide the following suggestions.

* If you already have a Git repository to store your externalized configuration, you can set up Spring Cloud Config Server locally as the centralized configuration for your application. After Config Server starts, it will clone the Git repository and provide the repository content through its web controller. For more information, see [Spring Cloud Config](https://cloud.spring.io/spring-cloud-config/reference/html) in the Spring documentation. The `spring-cloud-config-client` provides the ability for your application to automatically pick up the external configuration from the Config Server.

* If you don't have a Git repository or you don't want to set up Config Server locally, you can use the configuration file directly in your project. We recommend that you use a profile to isolate the configuration file so that it's used only in your development environment. For example, use `dev` as the profile. Then, you can create an *application-dev.yml* file in the *src/main/resource* folder to store the configuration. To get your app to use this configuration, start the app locally with `--spring.profiles.active=dev`.

#### Tanzu Service Registry

[VMware Tanzu® Service Registry](https://docs.vmware.com/en/Spring-Cloud-Services-for-VMware-Tanzu/index.html) is one of the commercial VMware Tanzu components. Tanzu Service Registry provides your Enterprise-tier apps with an implementation of the Service Discovery pattern, one of the key tenets of a microservice-based architecture. Your apps can use Tanzu Service Registry to dynamically discover and call registered services. Using Tanzu Service Registry is preferable to hand-configuring each client of a service, which can be difficult, or adopting some form of access convention, which can be brittle in production. For more information, see [Use Tanzu Service Registry](/azure/spring-cloud/how-to-enterprise-service-registry).

::: zone-end

### Migrate Spring Cloud Vault secrets to Azure KeyVault

You can inject secrets directly into applications through Spring by using the Azure KeyVault Spring Boot Starter. For more information, see [How to use the Spring Boot Starter for Azure Key Vault](../spring-framework/configure-spring-boot-starter-java-app-with-azure-key-vault.md).

> [!NOTE]
> Migration may require you to rename some secrets. Update your application code accordingly.

### Migrate all certificates to KeyVault

Azure Spring Cloud doesn't provide access to the JRE keystore, so you must migrate certificates to Azure KeyVault, and change the application code to access certificates in KeyVault. For more information, see [Get started with Key Vault certificates](/azure/key-vault/certificates/certificate-scenarios) and [Azure Key Vault Certificate client library for Java](/java/api/overview/azure/security-keyvault-certificates-readme).

### Remove application performance management (APM) integrations

Eliminate any integrations with APM tools/agents. For information on configuring performance management with Azure Monitor, see the [Post-migration](#post-migration) section.

### Replace explicit Zipkin dependencies with Spring Cloud Starters

If any of the migrated applications has explicit Zipkin dependencies, remove them and replace them with Spring Cloud Starters. For information on Azure Application Insights, see the [Post-migration](#post-migration) section.

### Disable metrics clients and endpoints in your applications

Remove any metrics clients used or any metrics endpoints exposed in your applications.

### Deploy the services

Deploy each of the migrated Spring apps (not including the Spring Cloud Config and Registry servers), as described in [Quickstart: Deploy your first application to Azure Spring Cloud](/azure/spring-cloud/quickstart).

### Configure per-service secrets and externalized settings

You can inject any per-service configuration settings into each service as environment variables. Use the following steps in the Azure portal:

1. Navigate to the Azure Spring Cloud Instance and select **Apps**.
1. Select the service to configure.
1. Select **Configuration**.
1. Enter the variables to configure.
1. Select **Save**.

![Spring Cloud App Configuration Settings](media/migrate-spring-cloud-to-azure-spring-cloud/spring-cloud-app-configuration-settings.png)

### Migrate and enable the identity provider

If any of the Spring Cloud applications require authentication or authorization, ensure they're configured to access the identity provider:

* If the identity provider is Azure Active Directory, no changes should be necessary.
* If the identity provider is an on-premises Active Directory forest, consider implementing a hybrid identity solution with Azure Active Directory. For guidance, see the [Hybrid identity documentation](/azure/active-directory/hybrid/).
* If the identity provider is another on-premises solution, such as PingFederate, consult the [Custom installation of Azure AD Connect](/azure/active-directory/hybrid/how-to-connect-install-custom) topic to configure federation with Azure Active Directory. Alternatively, consider using Spring Security to use your identity provider through [OAuth2/OpenID Connect](https://docs.spring.io/spring-security/reference/index.html) or [SAML](https://docs.spring.io/spring-security/reference/index.html).

### Update client applications

Update the configuration of all client applications to use the published Azure Spring Cloud endpoints for migrated applications.

## Post-migration

* Consider adding a deployment pipeline for automatic, consistent deployments. Instructions are available [for Azure Pipelines](/azure/spring-cloud/how-to-cicd), [for GitHub Actions](/azure/spring-cloud/how-to-github-actions), and [for Jenkins](/azure/jenkins/tutorial-jenkins-deploy-cli-spring-cloud-service).

* Consider using staging deployments to test code changes in production before they're available to some or all of your end users. For more information, see [Set up a staging environment in Azure Spring Cloud](/azure/spring-cloud/how-to-staging-environment).

* Consider adding service bindings to connect your application to supported Azure databases. These service bindings would eliminate the need for you to provide connection information, including credentials, to your Spring Cloud applications.

* Consider using Azure Application Insights to monitor performance and interactions of your applications. For more information, see [Application Insights Java In-Process Agent in Azure Spring Cloud](/azure/spring-cloud/how-to-application-insights).

* Consider adding Azure Monitor alert rules and action groups to quickly detect and address aberrant conditions. For more information, see [Tutorial: Monitor Spring Cloud resources using alerts and action groups](/azure/spring-cloud/tutorial-alerts-action-groups).

* Consider replicating the Azure Spring Cloud deployment in another region for lower latency and higher reliability and fault tolerance. Use [Azure Traffic Manager](/azure/traffic-manager) to load balance among deployments or use [Azure Front Door](/azure/frontdoor) to add SSL offloading and Web Application Firewall with DDoS protection.

* If geo-replication isn't necessary, consider adding an [Azure Application Gateway](/azure/application-gateway) to add SSL offloading and Web Application Firewall with DDoS protection.

* If your applications use legacy Spring Cloud Netflix components, consider replacing them with current alternatives:

   | Legacy                        | Current                                                |
   |-------------------------------|--------------------------------------------------------|
   | Spring Cloud Eureka           | Spring Cloud Service Registry                          |
   | Spring Cloud Netflix Zuul     | Spring Cloud Gateway                                   |
   | Spring Cloud Netflix Archaius | Spring Cloud Config Server                             |
   | Spring Cloud Netflix Ribbon   | Spring Cloud Load Balancer (client-side load balancer) |
   | Spring Cloud Hystrix          | Spring Cloud Circuit Breaker + Resilience4J            |
   | Spring Cloud Netflix Turbine  | Micrometer + Prometheus                                |
