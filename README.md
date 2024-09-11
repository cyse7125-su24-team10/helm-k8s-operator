# helm-k8s-operator

This Helm chart deploys a Kubernetes operator for monitoring and managing GitHub releases, specifically designed for CVE (Common Vulnerabilities and Exposures) tracking.

## Table of Contents

- [helm-k8s-operator](#helm-k8s-operator)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [Installing the Chart](#installing-the-chart)
  - [Configuration](#configuration)
  - [Custom Resource Definitions](#custom-resource-definitions)
    - [GitHubRelease CRD](#githubrelease-crd)
    - [GitHubReleasesMonitor CRD](#githubreleasesmonitor-crd)
  - [Contributing](#contributing)
  - [License](#license)

## Prerequisites

- Kubernetes 1.29+
- Helm 3.0+
- Docker Hub access (for pulling images)

## Installing the Chart

To install the chart with the release name `cve-operator`:

```bash
helm install cve-operator ./cve-operator
```

After installing the chart, you need to apply a GitHubReleasesMonitor custom resource. Here's an example of how to do this:

1. Create a file named `githubreleasesmonitor-sample.yaml` with the following content:

```yaml
apiVersion: cve-release.skynetx.me/v1
kind: GitHubReleasesMonitor
metadata:
  labels:
    app.kubernetes.io/name: cve-operator
    app.kubernetes.io/managed-by: kustomize
  name: githubreleasesmonitor-sample
spec:
  url: "https://github.com/CVEProject/cvelistV5/releases"
  monitorfrom: "now"
  dockersecret: "docker-hub-pat-reference-name"
  kafkauser: "user1"
  kafkapassword: "kafka-password"
  kafkabroker0: "kafka-controller-0.kafka-controller-headless.kafka.svc.cluster.local:9092"
  kafkabroker1: "kafka-controller-1.kafka-controller-headless.kafka.svc.cluster.local:9092"
  kafkabroker2: "kafka-controller-2.kafka-controller-headless.kafka.svc.cluster.local:9092"
  image: "vkneu7/pinecone-loader:v1.0.0"
  pineconekey: "Your-Pinecone-Project-Key"
  pineconecloud: "aws"
  pineconeenv: "cve"
  pineconeregion: "us-east-1"
```

2. Apply the GitHubReleasesMonitor custom resource:

```bash
kubectl apply -f githubreleasesmonitor-sample.yaml
```

This will create a GitHubReleasesMonitor resource that the CVE operator will use to monitor and retrieve hourly GitHub releases data from [CVE](https://github.com/CVEProject/cvelistV5/releases) and load it into the Pinecone database.

Note: Make sure to replace the placeholder values (e.g., "Your-Pinecone-Project-Key") with your actual configuration details before applying the resource.

## Configuration

The configuration options for this chart are defined in the Custom Resource Definitions. Please refer to the CRD sections below for details on configurable parameters.

## Custom Resource Definitions

This chart includes two Custom Resource Definitions (CRDs):

### GitHubRelease CRD

The `GitHubRelease` CRD represents a specific GitHub release to be processed. It includes the following fields:

- `url`: The GitHub repository URL to monitor
- `image`: Docker image of the [cve-operator](https://github.com/cyse7125-su24-team10/cve-operator)
- `dockersecret`: Secret reference name for pulling Docker images
- `kafkabroker0`, `kafkabroker1`, `kafkabroker2`: Kafka broker addresses
- `kafkauser`, `kafkapassword`: Kafka authentication credentials
- `pineconecloud`, `pineconeenv`, `pineconekey`, `pineconeregion`: Pinecone configuration

### GitHubReleasesMonitor CRD

The `GitHubReleasesMonitor` CRD defines parameters for monitoring GitHub releases. It includes all fields from the `GitHubRelease` CRD, plus:

- `monitorfrom`: Specifies the start date for monitoring. Options are:
  - `now`: Start monitoring from the time the CRD is created
  - `date`: Start monitoring from a specific date (format: YYYY-MM-DD)

This CRD is responsible for creating individual `GitHubRelease` resources for each new release detected.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request to the [cve-operator repository](https://github.com/cyse7125-su24-team10/cve-operator).

## License

This chart is licensed under the Apache License 2.0 - see the [LICENSE](LICENSE) file for details.