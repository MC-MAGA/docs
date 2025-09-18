# macOS hosted agents

macOS instances for Buildkite hosted agents are only offered with [Apple silicon](https://en.wikipedia.org/wiki/Apple_silicon) architecture. Please contact support if you have specific needs for Intel machines.

To accommodate different workloads, instances are capable of running up to 4 hours. If you require longer running agents, please contact support at support@buildkite.com.

## Sizes

Buildkite offers a selection of macOS instance types (each based on a different size combination of virtual CPU power and memory capacity, known as an _instance shape_), allowing you to tailor your hosted agents' resources to the demands of your jobs.

<%= render_markdown partial: 'shared/hosted_agents/hosted_agents_instance_shape_table_mac' %>

## macOS instance software support

All standard macOS [Tahoe](#macos-tahoe), [Sequoia](#macos-sequoia) and [Sonoma](#macos-sonoma) version instances have their own respective Xcode and runtime software versions available by default (listed below). Each macOS version also has its own set of [Homebrew packages](#homebrew-packages) with specific versions optimized for that operating system. If you have specific requirements for software that is not listed here, please contact support.

Updated Xcode versions will be available one week after Apple offers them for download. This includes Beta, Release Candidate (RC), and official release versions.

## macOS Tahoe

- 26.0

### Xcode

- 26.0
- 16.4

### Runtimes

#### iOS

- 26.0
- 18.6
- 17.5

#### tvOS

- 26.0
- 17.5
- 16.4

#### visionOS

- 26.0
- 2.5
- 1.2

#### watchOS

- 26.0
- 11.5
- 10.5

## macOS Sequoia

- 15.6.1

### Xcode

- 26.0
- 16.4
- 16.3
- 16.2
- 16.1
- 16.0
- 15.4

### Runtimes

#### iOS

- 26.0
- 18.6
- 18.5
- 18.4 RC
- 18.2 RC
- 18.1
- 18.0
- 17.5
- 16.4
- 15.5

#### tvOS

- 26.0
- 18.5
- 18.4 RC
- 18.2 RC
- 18.1
- 18.0
- 17.5
- 16.4

#### visionOS

- 26.0
- 2.5
- 2.4 RC
- 2.2 RC
- 2.1
- 2.0
- 1.2
- 1.1
- 1.0

#### watchOS

- 26.0
- 11.5
- 11.4 RC
- 11.2 RC
- 11.1
- 11.0
- 10.5
- 9.4

## macOS Sonoma

- 14.6.1

### Xcode

- 16.3
- 16.2
- 16.1
- 16.0
- 15.4
- 15.3
- 15.2
- 15.1
- 14.3.1

### Runtimes

#### iOS

- 18.4 RC
- 18.2 RC
- 18.1
- 18.0
- 17.5
- 17.4
- 17.2
- 16.4
- 16.2
- 15.5

#### tvOS

- 18.4 RC
- 18.2 RC
- 18.1
- 18.0
- 17.5
- 17.4
- 17.2
- 16.4

#### visionOS

- 2.4 RC
- 2.2 RC
- 2.1
- 2.0
- 1.2
- 1.1
- 1.0

#### watchOS

- 11.4 RC
- 11.2 RC
- 11.1
- 11.0
- 10.5
- 10.4
- 10.2
- 9.4

## Homebrew packages

Package versions vary by macOS operating system. Check your hosted agents base image page for specific version details for each macOS version.

- ant
- applesimutils
- aria2
- awscli
- azcopy
- azure-cli
- bazelisk
- bicep
- carthage
- cmake
- cocoapods
- curl
- deno
- docker
- fastlane
- gcc@13
- gh
- git
- git-lfs
- gmp
- gnu-tar
- gnupg
- go
- gradle
- httpd
- jq
- kotlin
- libpq
- llvm
- llvm@15
- maven
- mint
- nginx
- node
- openssl@3
- p7zip
- packer
- perl
- php
- pkgconf
- postgresql@14
- python@3.13
- r
- rbenv
- rbenv-bundler
- ruby
- rust
- rustup
- selenium-server
- swiftformat
- swiftlint
- tmux
- unxip
- wget
- xcbeautify
- xcodes
- yq
- zstd

## Git mirror cache

The Git mirror cache is a specialized type of cache volume designed to accelerate Git operations by caching the Git repository between builds. This is useful for large repositories that are slow to clone.

These volumes are attached on a best-effort basis depending on their locality, expiration and current usage, and therefore, should not be relied upon as durable data storage. By default, a Git mirror cache is created for each repository.

### Enabling Git mirror cache

To enable Git mirror cache for your hosted agents:

1. Select **Agents** in the global navigation to access the **Clusters** page.
1. Select the cluster in which to enable the Git mirror cache feature.
1. Select **Cache Storage**, then select the **Settings** tab.
1. Select **Enable Git mirror**, then select **Save cache settings** to enable Git mirrors for the selected hosted cluster.

Once enabled, the Git mirror cache will be used for all hosted jobs using Git repositories in that cluster. A separate cache volume will be created for each repository.

<%= image "hosted-agents-git-mirror.png", width: 1760, height: 436, alt: "Hosted agents git mirror setting displayed in the Buildkite UI" %>

### Deleting Git mirror cache

Deleting a cache volume may affect the build time for the associated pipelines until the new cache is established.

To delete a git mirror cache:

1. Select **Agents** in the global navigation to access the **Clusters** page.
1. Select the cluster whose Git mirror cache is to be deleted.
1. Select **Cache Storage**, then select the **Volumes** tab to view a list of all exiting cache volumes.
1. Select **Delete** for the Git mirror cache volume you wish to remove.
1. Confirm the deletion by selecting **Delete Cache Volume**.
