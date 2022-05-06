---
ms.date: 05-05-2022
ms.author: v-yonghuiye
---

## Spring Native Support

Spring Native provides support for compiling Spring Boot applications to native executables using the [GraalVM][graalvm] [native-image][graalvm-native-docs] compiler. The native images will bring many advantages, such as instant startup, instant peak performance, and reduced memory consumption. Some Spring Cloud Azure features can also benefit from the Spring Native support, the goal is that Spring Cloud Azure applications can be built as native images without any code modification. See the [Spring Native][spring-native-overview] for more details.


### Support

Spring Cloud Azure has been validated against GraalVM and Spring Native, and provides the beta version support. You can try it on your projects if they are using those supported dependencies, and [raise bugs][azure-sdk-java-issues] or [contribute pull requests][spring-cloud-azure-native-configuration] if something goes wrong on Spring Cloud Azure. See the [Spring Native Support][spring-native-support] for more details.

#### Spring Native

Spring Cloud Azure `4.1.0-beta.1` has been tested against Spring Native `0.11.4` and GraalVM `22.0.0`.

#### Spring Cloud Azure Native

NOTE: Spring Native `0.11.4` has been tested against Spring Cloud Azure Native Configuration `4.0.0-beta.1`.

Spring Cloud Azure provides a dependency `spring-cloud-azure-native-configuration` that is an extension of Spring Native configuration for Spring Cloud Azure libraries, the Spring Native AOT plugin will combine the `spring-native-configuration` and `spring-cloud-azure-native-configuration` to build applications to native executables. No extra modifications are needed to the code that uses Spring Cloud Azure libraries apart from adding the dependency, it only covers to the code at the Spring Cloud Azure libraries.

Supported features:

* `Azure App Configuration clients auto-configuration`
* `Azure Event Hubs clients auto-configuration`
* `Azure Key Vault Certificates clients auto-configuration`
* `Azure Key Vault Secrets clients auto-configuration`
* `Azure Storage Blob clients auto-configuration`
* `Azure Storage File Share clients auto-configuration`
* `Azure Storage Queue clients auto-configuration`
* `Spring Integration for Azure Event Hubs`
* `Spring Integration for Azure Storage Queue`

#### Limitations

The Spring Cloud Azure support for Spring Native is still in the early stages and continues to be updated, the following features are not yet supported:

* `Azure Cosmos clients auto-configuration`
* `Azure Service Bus clients auto-configuration`
* `Spring Data for Azure Cache for Redis`
* `Spring Data for Azure Cosmos`
* `Spring Cloud Stream for Azure Event Hubs`
* `Spring Cloud Stream for Azure Service Bus`
* `Spring Kafka for Azure Event Hubs`
* `Spring Integration for Azure Service Bus`

NOTE: Not all the native image options are supported by Spring Native, see the [Native image options][spring-native-image-options] for more details.

WARNING: Spring Cloud Azure `4.1.0-beta.1` is not validated for building native executables based on Gradle Kotlin.

### Project Setup

The Spring Cloud Azure applications can enable Spring Native support according to [Getting started][spring-native-getting-started], the only additional processing required is to add the following dependency to the POM file.

TIP:  Note that this dependency `com.azure.spring:spring-cloud-azure-native-configuration` is not managed in `com.azure.spring:spring-cloud-azure-dependencies`.

Maven dependency:

```xml
<dependency>
  <groupId>com.azure.spring</groupId>
  <artifactId>spring-cloud-azure-native-configuration</artifactId>
  <version>4.0.0-beta.1</version>
</dependency>
```

Gradle dependency:

```groovy
dependencies {
    implementation "com.azure.spring:spring-cloud-azure-native-configuration:4.0.0-beta.1"
}
```

### Build the native application

There are two main ways to build a Spring Boot native application with Spring Cloud Azure libraries.

#### Build with Buildpacks

The native application can be built as follows:

Build with Maven:
```shell
mvn spring-boot:build-image
```

Build with Gradle:
```shell
gradle bootBuildImage
```

See the [Getting started with Buildpacks][spring-native-getting-started-buildpacks] for more details.

#### Build with Native Build Tools

The native application can be built as follows:

Build with Maven:
```shell
mvn -Pnative -DskipTests package
```

Build with Gradle:
```shell
gradle nativeCompile
```

See the [Getting started with Native Build Tools][spring-native-getting-started-native-build-tools] for more details.

### Run the native application

There are two main ways to run a native executable.

TIP: Assuming the project artifact id is `spring-cloud-azure-sample`, the project version is `0.0.1-SNAPSHOT`, the custom image name can be specified using the *<image><name>custom-image-name</name></image>* configuration element in Spring Boot plugin if you are using [Cloud Native Buildpacks][spring-boot-container-images.buildpacks] or using the *<imageName>custom-image-name</imageName>* configuration element if you are using link:{graalvm-native-buildtools}[native-maven-plugin].

#### Run with Buildpacks

To run the application, you can use `docker` the usual way as shown in the following example:

```shell
docker run --rm -p 8080:8080 spring-cloud-azure-sample:0.0.1-SNAPSHOT
```

#### Run with Native Build Tools

To run your application, invoke the following:

Run with Maven:

```cmd
target\spring-cloud-azure-sample
````

Run with Gradle:
```cmd
build\native\nativeCompile\spring-cloud-azure-sample
````

### Samples

Please refer to [storage-blob-native][azure-spring-sample-storage-blob-native].

Here are other verified samples that support Spring Native, see [Spring Cloud Azure Samples][azure-spring-samples] for more details.

> [!div class="mx-tdBreakAll"]
> | Library Artifact ID                                     | Supported Example Projects                                                                                      |
> |---------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
> | spring-cloud-azure-starter-appconfiguration             | [appconfiguration-client][appconfiguration-client]                                                              |
> | spring-cloud-azure-starter-eventhubs                    | [eventhubs-client][eventhubs-client]                                                                            |
> | spring-cloud-azure-starter-integration-eventhubs        | [storage-queue-integration][storage-queue-integration], [storage-queue-operation][storage-queue-operation]      |
> | spring-cloud-azure-starter-integration-storage-queue    | [appconfiguration-client][appconfiguration-client]                                                              |
> | spring-cloud-azure-starter-keyvault-secrets             | [property-source][property-source], [secret-client][secret-client]                                              |
> | spring-cloud-azure-starter-storage-blob                 | [storage-blob-sample][storage-blob-sample]                                                                      |
> | spring-cloud-azure-starter-storage-file-share           | [storage-file-sample][storage-file-sample]                                                                      |
> | spring-cloud-azure-starter-storage-queue                | [storage-queue-client][storage-queue-client]                                                                    |

<!-- URL links -->
[graalvm]: https://www.graalvm.org/
[graalvm-docs]: https://www.graalvm.org/reference-manual
[graalvm-native-docs]: https://www.graalvm.org/reference-manual/native-image
[graalvm-native-buildtools]: https://github.com/graalvm/native-build-tools
[spring-cloud-azure-native-configuration]: https://github.com/Azure/azure-sdk-for-java/tree/main/sdk/spring-experimental/spring-cloud-azure-native-configuration
[azure-sdk-java-issues]: https://github.com/Azure/azure-sdk-for-java/issues
[spring-native-overview]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#overview
[spring-native-support]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#support
[spring-native-image-options]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#native-image-options
[spring-native-getting-started]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#getting-started
[spring-native-getting-started-buildpacks]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#getting-started-buildpacks
[spring-native-getting-started-native-build-tools]: https://docs.spring.io/spring-native/docs/0.11.4/reference/htmlsingle/#getting-started-native-build-tools
[spring-boot-container-images.buildpacks]: https://docs.spring.io/spring-boot/docs/2.6.6/reference/html/container-images.html#container-images.buildpacks
[azure-spring-samples]: https://github.com/Azure-Samples/azure-spring-boot-samples#run-samples-based-on-spring-native
[azure-spring-sample-storage-blob-native]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/spring-native/storage-blob-native
[appconfiguration-client]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/appconfiguration/spring-cloud-azure-starter-appconfiguration/appconfiguration-client
[eventhubs-client]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/eventhubs/spring-cloud-azure-starter-eventhubs/eventhubs-client
[storage-queue-integration]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/storage/spring-cloud-azure-starter-integration-storage-queue/storage-queue-integration
[storage-queue-operation]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/storage/spring-cloud-azure-starter-integration-storage-queue/storage-queue-operation
[appconfiguration-client]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/appconfiguration/spring-cloud-azure-starter-appconfiguration/appconfiguration-client
[property-source]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/keyvault/spring-cloud-azure-starter-keyvault-secrets/property-source
[secret-client]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/keyvault/spring-cloud-azure-starter-keyvault-secrets/secret-client
[storage-blob-sample]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/storage/spring-cloud-azure-starter-storage-blob/storage-blob-sample
[storage-file-sample]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/storage/spring-cloud-azure-starter-storage-file-share/storage-file-sample
[storage-queue-client]: https://github.com/Azure-Samples/azure-spring-boot-samples/tree/main/storage/spring-cloud-azure-starter-storage-queue/storage-queue-client
