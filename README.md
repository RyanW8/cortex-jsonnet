# Jsonnet for Cortex

This repo has the jsonnet for deploying cortex and the related monitoring in Kubernetes.

To generate the YAMLs for deploying Cortex:

1. Make sure you have tanka and jb installed:

    Follow the steps at https://tanka.dev/install. If you have `go` installed locally you can also use:

    ```console
    $ # make sure to be outside of GOPATH or a go.mod project
    $ GO111MODULE=on go get github.com/grafana/tanka/cmd/tk
    $ GO111MODULE=on go get github.com/jsonnet-bundler/jsonnet-bundler/cmd/jb
    ```

1. Initialise the Tanka, and install the Cortex and Kubernetes Jsonnet libraries.

    ```console
    $ mkdir <name> && cd <name>
    $ tk init --k8s=false
    $ # The k8s-alpha library supports Kubernetes versions 1.14+
    $ jb install github.com/jsonnet-libs/k8s-alpha/1.18
    $ cat <<EOF > lib/k.libsonnet
      (import "github.com/jsonnet-libs/k8s-alpha/1.18/main.libsonnet")
      + (import "github.com/jsonnet-libs/k8s-alpha/1.18/extensions/kausal-shim.libsonnet")
      EOF
    $ jb install github.com/grafana/cortex-jsonnet/cortex
    ```
1. Use the example monitoring.jsonnet.example:

    ```console
    $ cp vendor/cortex/cortex-manifests.jsonnet.example environments/default/main.jsonnet
    ```

1. Check what is in the example:

    ```console
    $ cat environments/default/main.jsonnet
    ...
    ```

1. Generate the YAML manifests:

    ```console
    $ tk show environments/default
    ```

    To output YAML manifests to `./manifests`, run:

    ```console
    $ tk export environments/default manifests
    ```

    To generate the dashboards and alerts for Cortex:

    ```console
    $ cd cortex-mixin
    $ jb install
    $ mkdir out
    $ jsonnet -S alerts.jsonnet > out/alerts.yaml
    $ jsonnet -J vendor -S dashboards.jsonnet -m out
    $ jsonnet -J vendor -S recording_rules.jsonnet > out/rules.yaml
    ```
