# Load Balancer Annotations

This file defines a list of [Service][4] `type: LoadBalancer` annotations which are
supported by the `oci-cloud-controller-manager`.

All annotations are prefixed with `service.beta.kubernetes.io/` or `oci.oraclecloud.com/` or `oci-network-load-balancer.oraclecloud.com/` (for OCI Network Load Balancer specific annotations). For example:

```yaml
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  annotations:
    oci.oraclecloud.com/load-balancer-type: "lb"
    service.beta.kubernetes.io/oci-load-balancer-shape: "400Mbps"
    service.beta.kubernetes.io/oci-load-balancer-subnet1: "ocid..."
    service.beta.kubernetes.io/oci-load-balancer-subnet2: "ocid..."
    oci.oraclecloud.com/oci-network-security-groups: "ocid1..."
spec:
  ...
```

## Load balancer properties

| Name                                        | Description                                                                                                                                                                                                                                        | Default                                          |
| -----                                       | -----------                                                                                                                                                                                                                                        | -------                                          |
| `oci-load-balancer-internal`                | Create an [internal load balancer][1]. Cannot be modified after load balancer creation.                                                                                                                                                            | `false`                                          |
| `oci-load-balancer-shape`                   | A template that determines the load balancer's total pre-provisioned capacity (bandwidth) for ingress plus egress traffic. Available shapes include `100Mbps`, `400Mbps`, `8000Mbps` and `flexible`. Use `oci lb shape list` to get the list of shapes supported on your account | `"100Mbps"`                                      |
| `oci-load-balancer-shape-flex-min`                   | A template that determines the load balancer's minimum pre-provisioned capacity (bandwidth) for ingress plus egress traffic. Only used when `oci-load-balancer-shape` is set to `flexible`  | `N/A`                                      |
| `oci-load-balancer-shape-flex-max`                   | A template that determines the load balancer's maximum pre-provisioned capacity (bandwidth) for ingress plus egress traffic. Only used when `oci-load-balancer-shape` is set to `flexible`  | `N/A`                                      |
| `oci-load-balancer-subnet1`                 | The OCID of the one required regional subnet to attach the load balancer to OR The OCID of the first [subnet][2] of the two required Availability Domain specific subnets to attach the load balancer to. Must be in separate Availability Domains.                                                               | Value provided in config file                    |
| `oci-load-balancer-subnet2`                 | The OCID of the second [subnet][2] of the two required subnets to attach the load balancer to. Must be in separate Availability Domains.                                                            | Value provided in config file                    |
| `oci-load-balancer-health-check-retries`    | The number of retries to attempt before a backend server is considered "unhealthy".                                                                                                                                                                | `3`                                              |
| `oci-load-balancer-health-check-timeout`    | The maximum time, in milliseconds, to wait for a reply to a [health check][6]. A [health check][6] is successful only if a reply returns within this timeout period.                                                                               | `3000`                                           |
| `oci-load-balancer-health-check-interval`   | The interval between [health checks][6] requests, in milliseconds.                                                                                                                                                                                 | `10000`                                          |
| `oci-load-balancer-connection-idle-timeout` | The maximum idle time, in seconds, allowed between two successive receive or two successive send operations between the client and backend servers.                                                                                                | `300` for TCP listeners, `60` for HTTP listeners |
| `oci-load-balancer-security-list-management-mode` | Specifies the [security list mode](##security-list-management-modes) (`"All"`, `"Frontend"`,`"None"`) to configure how security lists are managed by the CCM.                            | `"All"`            
| `oci-load-balancer-backend-protocol` | Specifies protocol on which the listener accepts connection requests. To get a list of valid protocols, use the [`ListProtocols`][5] operation.                          | `"TCP"`            
| `oci-network-security-groups` | Specifies Network Security Groups' OCIDs to be associated with the loadbalancer. Please refer [here][8] for NSG details.                      | `N/A`            

Note: 
- Only one annotation `oci-load-balancer-subnet1` should be passed if it is a regional subnet.
- `oci-network-security-groups` uses `oci.oraclecloud.com/` as prefix.
## TLS-related

| Name | Description | Default |
| ---- | ----------- | ------- |
| `oci-load-balancer-tls-secret` | A reference in the form `<namespace>/<secretName>` to a Kubernetes [TLS secret][3]. | `""` |
| `oci-load-balancer-ssl-ports` | A `,` separated list of port number(s) for which to enable SSL termination. | `""` |

## Security List Management Modes
| Mode | Description | 
| ---- | ----------- | 
| `"All"` | CCM will manage all required security list rules for load balancer services | 
| `"Frontend"` | CCM will manage  only security list rules for ingress to the load balancer. Requires that the user has setup a rule that allows inbound traffic to the appropriate ports for kube proxy health port, node port ranges, and health check port ranges.  | 
| `"None`" | Disables all security list management. Requires that the user has setup a rule that allows inbound traffic to the appropriate ports for kube proxy health port, node port ranges, and health check port ranges. *Additionally, requires the user to mange rules to allow inbound traffic to load balancers.* | 

Note:
- If an invalid mode is passed in the annotation, then the default (`"All"`) mode is configured.
- If an annotation is not specified, the mode specified in the cloud provider config file is configured.  

## Network Load Balancer

For example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-nlb
  annotations:
    oci-network-load-balancer.oraclecloud.com/security-list-management-mode: "All"
    oci.oraclecloud.com/load-balancer-type: nlb
spec:
  selector:
    app: example-nlb
  ports:
    - port: 8088
      targetPort: 80
  type: LoadBalancer
  externalTrafficPolicy: Local
```

Note:
- The only security list management mode allowed when backend protocol is UDP is "None"
- `externalTrafficPolicy` should be "Local" for preserving source IP
- We recommend to set the `security-list-management-mode` as "None" and configure NSG / Security rules on your own.

## Network Load Balancer Specific Annotations

| Name                                                                          | Description                                                                                                                                                   | Default
| -----                                                                         | -----------                                                                                                                                                   | -------                                         
| `oci-network-load-balancer.oraclecloud.com/internal`                          | Create an [internal network load balancer][1]. Cannot be modified after load balancer creation.                                                               | `false`
| `oci-network-load-balancer.oraclecloud.com/subnet`                            | The OCID of the required regional or AD specific subnet to attach the network load balancer.	                                                                | Value set for the cluster
| `oci-network-load-balancer.oraclecloud.com/oci-network-security-groups`       | Specifies Network Security Groups' OCIDs to be associated with the network load balancer.	                                                                    | `""`
| `oci-network-load-balancer.oraclecloud.com/initial-freeform-tags-override`    | Specifies one or multiple Freeform tags to apply to the OCI Network Load Balancer.                                                                            | `""`
| `oci-network-load-balancer.oraclecloud.com/initial-defined-tags-override`     | Specifies one or multiple Defined tags to apply to the OCI Network Load Balancer.                                                                             | `""`
| `oci-network-load-balancer.oraclecloud.com/health-check-retries`              | The number of retries to attempt before a backend server is considered "unhealthy".	                                                                        | `3`
| `oci-network-load-balancer.oraclecloud.com/health-check-timeout`              | The maximum time, in milliseconds, to wait for a reply to a health check. A health check is successful only if a reply returns within this timeout period.    | `3000 ms`
| `oci-network-load-balancer.oraclecloud.com/health-check-interval`             | The interval between health checks requests, in milliseconds.		                                                                                            | `3000 ms`
| `oci-network-load-balancer.oraclecloud.com/backend-policy`                    | The network load balancer policy for the backend set. Valid values: "TWO_TUPLE", "THREE_TUPLE", or "FIVE_TUPLE"		                                        | `"FIVE_TUPLE"`
| `oci-network-load-balancer.oraclecloud.com/security-list-management-mode`     | Specifies the security list mode ("All", "Frontend","None") to configure how security lists are managed.		                                                | `"None"`


[1]: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
[2]: https://docs.us-phoenix-1.oraclecloud.com/Content/Network/Tasks/managingVCNs.htm
[3]: https://kubernetes.io/docs/concepts/services-networking/ingress/#tls
[4]: https://kubernetes.io/docs/concepts/services-networking/service/
[5]: https://docs.cloud.oracle.com/iaas/api/#/en/loadbalancer/20170115/LoadBalancerProtocol/ListProtocols
[6]: https://docs.cloud.oracle.com/en-us/iaas/api/#/en/loadbalancer/20170115/HealthChecker/
[7]: https://docs.oracle.com/en-us/iaas/api/#/en/loadbalancer/20170115/LoadBalancerPolicy/ListPolicies
[8]: https://docs.oracle.com/en-us/iaas/Content/Network/Concepts/networksecuritygroups.htm
