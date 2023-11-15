# Install by using the Knative Operator

Knative offers a [Kubernetes Operator](https://kubernetes.io/docs/concepts/extend-kubernetes/operator/) that facilitates the installation, configuration, and management of Knative components. The Serving and Eventing components can be installed individually or together on your Kubernetes cluster.


# Prerequisites

Before proceeding with the installation of Knative, ensure that the following prerequisites are met:

- **For Prototyping Purposes:** Knative is compatible with most local Kubernetes deployments. For instance, a local, single-node cluster with 3 CPUs and 4 GB of memory can be used. You can set up a local Knative distribution for development using the Knative Quickstart plugin.

- **For Production Purposes:** It is recommended to have:
  - At least 6 CPUs, 6 GB of memory, and 30 GB of disk storage for a single-node cluster.
  - For multi-node clusters, each node should have at least 2 CPUs, 4 GB of memory, and 20 GB of disk storage.
- A Kubernetes cluster running version v1.26 or newer.
- Installation of the `kubectl` CLI.
- Internet access for the Kubernetes cluster to fetch images. For private registry access, refer to [Deploying images from a private container registry](/docs/serving/deploying-from-private-registry/).


## Install the Knative Operator

Prior to installing the Knative Serving and Eventing components, the Knative Operator must be installed first.

To install the latest stable Operator release, execute the following command:

```bash
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.12.0/operator.yaml
```

### Verify your Knative Operator installation

1. Set the current namespace to default:

    ```bash
    kubectl config set-context --current --namespace=default
    ```

1. Check the Operator deployment status:

    ```bash
    kubectl get deployment knative-operator
    ```

    If the installation is successful, the deployment status should be:

    ```{.bash .no-copy}
    NAME               READY   UP-TO-DATE   AVAILABLE   AGE
    knative-operator   1/1     1            1           19h
    ```

### Track the log

To monitor the Operator logs, use the command:

```bash
kubectl logs -f deploy/knative-operator
```

## Install Knative Serving

For Knative Serving installation, a custom resource (CR) must be created, the networking layer added to the CR, and DNS configured.

### Create the Knative Serving custom resource

To create the custom resource for the latest Knative Serving version in the Operator, follow these steps:

1. Copy the following YAML into a file:

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: knative-serving
    ---
    apiVersion: operator.knative.dev/v1beta1
    kind: KnativeServing
    metadata:
      name: knative-serving
      namespace: knative-serving
    ```
    !!! note
If no version is specified using spec.version, the Operator defaults to the latest available version.

1. Apply the YAML file by running the command:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

    Where `<filename>` is the name of the file you created in the previous step.

### Install the networking layer
    Follow these steps to install Kourier and enable its Knative integration:

    1. Add spec.ingress.kourier and spec.config.network to your Serving CR YAML file:

        ```yaml
        apiVersion: operator.knative.dev/v1beta1
        kind: KnativeServing
        metadata:
          name: knative-serving
          namespace: knative-serving
        spec:
          # ...
          ingress:
            kourier:
              enabled: true
          config:
            network:
              ingress-class: "kourier.ingress.networking.knative.dev"
        ```

    1. Apply the YAML file for your Serving CR:

        ```bash
        kubectl apply -f <filename>.yaml
        ```


    1. Fetch the External IP or CNAME:


        ```bash
        kubectl --namespace knative-serving get service kourier
        ```

Save this information for DNS configuration.



### Verify the Knative Serving deployment

1. Monitor Knative deployments:

    ```bash
    kubectl get deployment -n knative-serving
    ```

    If Knative Serving is successfully deployed, all deployments will show READY status. Here is a sample output:

    ```{ .bash .no-copy }
    NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
    activator              1/1     1            1           18s
    autoscaler             1/1     1            1           18s
    autoscaler-hpa         1/1     1            1           14s
    controller             1/1     1            1           18s
    domain-mapping         1/1     1            1           12s
    domainmapping-webhook  1/1     1            1           12s
    webhook                1/1     1            1           17s
    ```

1. Check the status of Knative Serving Custom Resource:

    ```bash
    kubectl get KnativeServing knative-serving -n knative-serving
    ```

   If installed successfully, you should see:

    ```{ .bash .no-copy }
    NAME              VERSION             READY   REASON
    knative-serving   <version number>    True
    ```

### Configure DNS
Configure DNS to eliminate the need for curl commands with a host header. Knative provides a Kubernetes Job called default-domain to configure Knative Serving to use sslip.io as the default DNS suffix.

    ```bash
    kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.12.0/serving-default-domain.yaml
    ```

## Install Knative Eventing

Knative Eventing installation involves applying the custom resource (CR). Optionally, you can install the Knative Eventing component with different event sources.

### Create the Knative Eventing custom resource

To create the custom resource for the latest Knative Eventing version in the Operator, perform the following steps:

1. Copy the following YAML into a file:

    ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: knative-eventing
    ---
    apiVersion: operator.knative.dev/v1beta1
    kind: KnativeEventing
    metadata:
      name: knative-eventing
      namespace: knative-eventing
    ```

   !!! note
If no version is specified using spec.version, the Operator defaults to the latest available version.

1. Apply the YAML file by running the command:

    ```bash
    kubectl apply -f <filename>.yaml
    ```

Replace <filename> with the name of your created file.


### Verify the Knative Eventing deployment

1. Monitor Knative deployments:

    ```bash
    kubectl get deployment -n knative-eventing
    ```

    If Knative Eventing is successfully deployed, all deployments will show READY status. Here is a sample output:

    ```{.bash .no-copy}
    NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
    eventing-controller     1/1     1            1           43s
    eventing-webhook        1/1     1            1           42s
    imc-controller          1/1     1            1           39s
    imc-dispatcher          1/1     1            1           38s
    mt-broker-controller    1/1     1            1           36s
    mt-broker-filter        1/1     1            1           37s
    mt-broker-ingress       1/1     1            1           37s
    pingsource-mt-adapter   0/0     0            0           43s
    sugar-controller        1/1     1            1           36s
    ```

1. Check the status of Knative Eventing Custom Resource:

    ```bash
    kubectl get KnativeEventing knative-eventing -n knative-eventing
    ```

    If installed successfully, you should see:

    ```{.bash .no-copy}
    NAME               VERSION             READY   REASON
    knative-eventing   <version number>    True
    ```

