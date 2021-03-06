# Hyperledger Fabric CA

[Hyperledger Fabric CA](http://hyperledger-fabric-ca.readthedocs.io/) is a Certificate Authority node for the [Hyperledger](https://www.hyperledger.org/) Fabric permissioned blockchain framework. Learn more about it by visiting the [user's guide](http://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#).

## TL;DR;

```bash
$ helm install stable/hlf-ca
```

## Introduction

The Hyperledger Fabric CA can be installed as either a Root CA, or an intermediate CA (by pointing to a parent CA, which can itself be a Root CA or an intermediate).

This CA can then be used to register and enroll identities for clients, admins and nodes of the Hyperledger Fabric network.

## Prerequisites

- Kubernetes 1.9+
- PV provisioner support in the underlying infrastructure.
- A running [PostgreSQL Chart](https://github.com/kubernetes/charts/tree/master/stable/postgresql) to host the Hyperledger Fabric CA data, in a database defined under the settings `db.database`.

## Installing the Chart

To install the chart with the release name `org1-ca`:

```bash
$ helm install stable/hlf-ca --name org1-ca
```

The command deploys the Hyperledger Fabric CA on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

### Custom parameters

Specify each parameter using the `--set key=value[,key=value]` argument to `helm install`. For example:

```bash
$ helm install stable/hlf-ca --name org1-ca --set adminUsername=ca-admin,adminPassword=secretpassword
```

The above command creates a CA Admin user named `ca-admin` with password `secretpassword`.

Alternatively, a YAML file can be provided while installing the chart. This file specifies values to override those provided in the defualt values.yaml. For example,

```bash
$ helm install stable/hlf-ca --name org1-ca -f my-values.yaml
```

## Updating the chart

When updating the chart, make sure you provide the `adminPassword`, otherwise `helm update` will generate a new random (and invalid) password.

```bash
$ export CA_PASSWORD=$(kubectl get secret --namespace {{ .Release.Namespace }} org1-ca -o jsonpath="{.data.CA_PASSWORD}" | base64 --decode; echo)
$ helm upgrade org1-ca stable/hlf-ca --set adminPassword=$CA_PASSWORD
```

## Uninstalling the Chart

To uninstall/delete the `org1-ca` deployment:

```bash
$ helm delete org1-ca
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following table lists the configurable parameters of the Hyperledger Fabric CA chart and default values.

| Parameter                          | Description                                     | Default                                                    |
| ---------------------------------- | ------------------------------------------------ | ---------------------------------------------------------- |
| `image.repository`                 | `hlf-ca` image repository                        | `hyperledger/fabric-ca`                                    |
| `image.tag`                        | `hlf-ca` image tag                               | `x86_64-1.1.0`                                             |
| `image.pullPolicy`                 | Image pull policy                                | `IfNotPresent`                                             |
| `service.port`                     | TCP port                                         | `7054`                                                     |
| `service.type`                     | K8S service type exposing ports, e.g. `ClusterIP`| `ClusterIP`                                                |
| `ingress.enabled`                  | If true, Ingress will be created                 | `false`                                                    |
| `ingress.annotations`              | Ingress annotations                              | `{}`                                                       |
| `ingress.path`                     | Ingress path                                     | `/`                                                        |
| `ingress.hosts`                    | Ingress hostnames                                | `[]`                                                       |
| `ingress.tls`                      | Ingress TLS configuration                        | `[]`                                                       |
| `persistence.accessMode`           | Use volume as ReadOnly or ReadWrite              | `ReadWriteOnce`                                            |
| `persistence.annotations`          | Persistent Volume annotations                    | `{}`                                                       |
| `persistence.size`                 | Size of data volume                              | `1Gi`                                                      |
| `persistence.storageClass`         | Storage class of backing PVC                     | `default`                                                  |
| `adminUsername`                    | Admin Username for CA                            | `admin`                                                    |
| `adminPassword`                    | Admin Password                                   | Random 24 alphanumeric characters                          |
| `caName`                           | Name of CA                                       | `org1-ca`                                                  |
| `db.chart`                         | Name of a Database Chart holding CA data         | `` supports postgresql                                     |
| `db.database`                      | Name of the actual Database holding the CA data  | `fabric_ca`                                                |
| `config.hlfToolsVersion`           | Version of Hyperledger Fabric tools used         | `1.1.0`                                                    |
| `config.mountTLS`                  | If TLS secrets are generated, do we mount them?  | `false`                                                    |
| `config.debug`                     | Enable debug logging                             | `true`                                                     |
| `config.csr.ca.pathlength`         | Pathlength of CA certificate hierarchy           | `1`                                                        |
| `config.csr.names.c`               | Country to which CA belongs                      | `US`                                                       |
| `config.csr.names.st`              | State to which CA belongs                        | `North Carolina`                                           |
| `config.csr.names.l`               | Locality to which CA belongs                     | ``                                                         |
| `config.csr.names.o`               | Organization to which CA belongs                 | `Hyperledger`                                              |
| `config.csr.names.ou`              | Organizational Unit to which CA belongs          | `Fabric`                                                   |
| `config.intermediate`              | Structure defining that CA is intermediate       | `nil`                                                      |
| `config.intermediate.parent.chart` | Which hlf-ca chart acts as parent to this CA     | `nil`                                                      |
| `config.intermediate.parent.url`   | URL of parent CA                                 | `nil`                                                      |
| `config.intermediate.parent.port`  | Port of parent CA                                | `nil`                                                      |
| `config.affiliations`              | Affiliations for CA                              | `{org1: [] }`                                              |
| `resources`                        | CPU/Memory resource requests/limits              | `{}`                                                       |
| `nodeSelector`                     | Node labels for pod assignment                   | `{}`                                                       |
| `tolerations`                      | Toleration labels for pod assignment             | `[]`                                                       |
| `affinity`                         | Affinity settings for pod assignment             | `{}`                                                       |

## Persistence

The volume stores the Fabric_CA data and configurations at the `/var/hyperledger` path of the container.

The chart mounts a [Persistent Volume](http://kubernetes.io/docs/user-guide/persistent-volumes/) at this location. The volume is created using dynamic volume provisioning through a PersistentVolumeClaim managed by the chart.

## Feedback and feature requests

This is a work in progress and we are happy to accept feature requests. We are even happier to accept pull requests implementing improvements :-)
