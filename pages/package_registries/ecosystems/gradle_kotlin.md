# Gradle (Kotlin)

Buildkite Package Registries provides registry support for Gradle-based Java packages (using the [Maven Publish Plugin](https://docs.gradle.org/current/userguide/publishing_maven.html)), using the Gradle Kotlin DSL. If you're using Gradle's Groovy DSL, refer to the [Gradle (Groovy)](/docs/package-registries/gradle-groovy) page.

Once your Java source registry has been [created](/docs/package-registries/manage-registries#create-a-source-registry), you can publish/upload packages (generated from your application's build) to this registry by configuring your `build.gradle.kts` file with the Gradle snippet presented on your Java registry's details page.

To view and copy the required `build.gradle.kts` configurations:

1. Select **Package Registries** in the global navigation to access the **Registries** page.
1. Select your Java source registry on this page.
1. Select the **Publish Instructions** tab and on the resulting page, in the **Using Gradle with `maven-publish` plugin** section, select **Gradle (Kotlin)** to expand this section.
1. Use the copy icon at the top-right of the code box to copy the Gradle code snippet and paste it into the appropriate area/s of your `build.gradle.kts` file.

    These `build.gradle.kts` file configurations contain the:
    <%= render_markdown partial: 'package_registries/ecosystems/gradle_file_configurations' %>

1. You can then run the `gradle publish` command to publish the package to this source registry.

## Publish a package

The following steps describe the process above:

1. Copy the following Gradle (Kotlin) snippet, paste it into your `build.gradle.kts` file, and modify accordingly:

    ```kotlin
    plugins {
      `maven-publish`
      `java-library`
    }

    publishing {
      publications {
          create<MavenPublication>("maven") {
              // MODIFY: Define your Maven coordinates of your package
              groupId = "com.name.domain.my"
              artifactId = "my-java-package-name"
              version = "my-java-package-version"

              from(components["java"])
          }
      }

      repositories {
        maven {
          url = uri("https://packages.buildkite.com/{org.slug}/{registry.slug}/maven2/")
          authentication {
            create<HttpHeaderAuthentication>("header")
          }

          credentials(HttpHeaderCredentials::class) {
            name = "Authorization"
            value = "Bearer registry-write-token"
          }
        }
      }
    }
    ```

    where:
    <%= render_markdown partial: 'package_registries/ecosystems/java_package_domain_name_version' %>

    <%= render_markdown partial: 'package_registries/org_slug' %>

    <%= render_markdown partial: 'package_registries/ecosystems/java_registry_slug' %>

    <%= render_markdown partial: 'package_registries/ecosystems/java_registry_write_token' %>

1. Publish your package:

    ```bash
    gradle publish
    ```

## Access a package's details

<%= render_markdown partial: 'package_registries/ecosystems/access_java_package_details_page' %>

<%= render_markdown partial: 'package_registries/ecosystems/package_details_page_sections' %>

### Downloading a package

A Java package can be downloaded from the package's details page. To do this:

1. [Access the package's details](#access-a-packages-details).
1. Select **Download**.

<h3 id="access-a-packages-details-installing-a-package"></h3>

### Installing a package from a source registry

A Java package can be installed using code snippet details provided on the package's details page. To do this:

1. [Access the package's details](#access-a-packages-details).
1. Ensure the **Installation** (tab) > **Gradle (Kotlin)**  section is displayed.
1. Copy the code snippet, paste this into the `build.gradle.kts` Gradle file, and modify the required values accordingly.

    You can then run `gradle install` on this modified script file to install this package.

This code snippet is based on this format:

```kotlin
repositories {
  maven {
    url = uri("https://packages.buildkite.com/{org.slug}/{registry.slug}/maven2/")
    authentication {
      create<HttpHeaderAuthentication>("header")
    }

    credentials(HttpHeaderCredentials::class) {
      name = "Authorization"
      value = "Bearer registry-read-token"
    }
  }
}

dependencies {
  implementation("com.name.domain.my:my-java-package-name:my-java-package-version")
}
```

where:

<%= render_markdown partial: 'package_registries/org_slug' %>

<%= render_markdown partial: 'package_registries/ecosystems/registry_slug' %>

- `registry-read-token` is your [API access token](https://buildkite.com/user/api-access-tokens) or [registry token](/docs/package-registries/manage-registries#configure-registry-tokens) used to download packages from your Java source registry. Ensure this access token has the **Read Packages** REST API scope, which allows this token to download packages from any registry your user account has access to within your Buildkite organization.

    **Note:** Both the `authentication` and `credentials` sections are not required for registries that are publicly accessible.

<%= render_markdown partial: 'package_registries/ecosystems/java_package_domain_name_version' %>

### Installing a package from a composite registry

If your Java source registry is an upstream of a [composite registry](/docs/package-registries/manage-registries#composite-registries), you can install one of its packages using the code snippet details provided on the composite registry's **Setup & Usage** page. To do this:

1. Select **Package Registries** in the global navigation to access the **Registries** page.
1. Select your Java composite registry on this page.
1. Select the **Setup & Usage** tab to display the **Usage Instructions** page.
1. Ensure the **Gradle (Kotlin)** tab is displayed.
1. Copy the relevant code snippets, and paste them into the `build.gradle.kts` Gradle file, modifying their values as required. Learn more about this is [Configuring the `build.gradle.kts` Gradle file](#configuring-the-build-dot-gradle-dot-kts-gradle-file), below.

    To install packages from any of this composite registry's configured upstreams, define each of these packages in their own `implementation()` line within `dependencies { }` of your `build.gradle.kts` Gradle file, as you would when [installing packages from a source registry](#access-a-packages-details-installing-a-package-from-a-source-registry), and run `gradle install` on this modified script file.

<h4 id="configuring-the-build-dot-gradle-dot-kts-gradle-file">Configuring the build.gradle.kts Gradle file</h4>

The `build.gradle.kts` code snippet is based on this format:

```kotlin
repositories {
  // ...
  maven {
    url = uri("https://packages.buildkite.com/{org.slug}/{registry.slug}/maven2/")
    authentication {
      create<HttpHeaderAuthentication>("header")
    }
    credentials(HttpHeaderCredentials::class) {
      name = "Authorization"
      value = "Bearer registry-read-token"
    }
  }
}
```

where:

<%= render_markdown partial: 'package_registries/org_slug' %>

<%= render_markdown partial: 'package_registries/ecosystems/registry_slug' %>

- `registry-read-token` is your [API access token](https://buildkite.com/user/api-access-tokens) or [registry token](/docs/package-registries/manage-registries#configure-registry-tokens) used to download packages from your Java composite registry. Ensure this access token has the **Read Packages** REST API scope, which allows this token to download packages from any registry your user account has access to within your Buildkite organization.

    To avoid having to store the actual token value in this file (and mitigate its exposure to continuous integration environments), you can reference it using an environment variable. For example, if you set this token value in the environment variable `REGISTRY_READ_TOKEN`, like:

    ```bash
    export REGISTRY_READ_TOKEN="YOUR-ACTUAL-TOKEN-VALUE"
    ```

    you can reference this in the `value` field above as:

    ```kotlin
    value = "Bearer ${System.getenv('REGISTRY_READ_TOKEN')}"
    ```

If you have added the official public registry to this Java composite registry, ensure that any references to the default `mavenCentral()` repository (in your `build.gradle.kts` or other relevant `.gradle.kts` files) have been removed, since Buildkite Package Registries itself handles the connection to the Maven Central repository through your Java composite registry.
