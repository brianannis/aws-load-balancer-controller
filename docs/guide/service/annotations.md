## Service annotations

!!!note ""
    - Annotation keys and values can only be strings. All other types below must be string-encoded, for example:
        - boolean: `"true"`
        - integer: `"42"`
        - stringList: `"s1,s2,s3"`
        - stringMap: `"k1=v1,k2=v2"`
        - json: `"{ \"key\": \"value\" }"`

## Annotations
| Name                                                                                             | Type                    | Default                   | Notes                                                  |
|--------------------------------------------------------------------------------------------------|-------------------------|---------------------------|--------------------------------------------------------|
| [service.beta.kubernetes.io/load-balancer-source-ranges](#lb-source-ranges)                      | stringList              |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-type](#lb-type)                                    | string                  |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-nlb-target-type](#nlb-target-type)                 | string                  |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-name                                                | string                  |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-internal](#lb-internal)                            | boolean                 | false                     |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-proxy-protocol](#proxy-protocol-v2)                | string                  |                           | Set to `"*"` to enable                                 |
| service.beta.kubernetes.io/aws-load-balancer-ip-address-type                                     | string                  | ipv4                      | ipv4 \| dualstack                                      |
| service.beta.kubernetes.io/aws-load-balancer-access-log-enabled                                  | boolean                 | false                     |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-name                           | string                  |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-access-log-s3-bucket-prefix                         | string                  |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-cross-zone-load-balancing-enabled                   | boolean                 | false                     |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-ssl-cert                                            | stringList              |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-ssl-ports                                           | stringList              |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-ssl-negotiation-policy                              | string                  | ELBSecurityPolicy-2016-08 |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-backend-protocol                                    | string                  |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-additional-resource-tags                            | stringMap               |                           |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-healthy-threshold                       | integer                 | 3                         |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-unhealthy-threshold                     | integer                 | 3                         |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-timeout                                 | integer                 | 10                        |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-interval                                | integer                 | 10                        |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-protocol                                | string                  | TCP                       |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-port                                    | integer \| traffic-port | traffic-port              |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-healthcheck-path                                    | string                  | "/" for HTTP(S) protocols |                                                        |
| service.beta.kubernetes.io/aws-load-balancer-eip-allocations                                     | stringList              |                           | Public Facing lb only. Length/order must match subnets |
| service.beta.kubernetes.io/aws-load-balancer-private-ipv4-addresses                              | stringList              |                           | Internal lb only. Length/order must match subnets      |
| [service.beta.kubernetes.io/aws-load-balancer-target-group-attributes](#target-group-attributes) | stringMap               |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-subnets](#subnets)                                 | stringList              |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-alpn-policy](#alpn-policy)                         | stringList              |                           |                                                        |
| [service.beta.kubernetes.io/aws-load-balancer-target-node-labels](#target-node-labels)           | stringMap               |                           |                                                        |


## Traffic Routing
Traffic Routing can be controlled with following annotations:

- <a name="lb-type">`service.beta.kubernetes.io/aws-load-balancer-type`</a> specifies the load balancer type. This controller reconciles those service resources with this annotation set to either `nlb-ip` or `external`.

    !!!note ""
        - For `nlb-ip` type, controller will provision NLB with IP targets. This value is supported for backwards compatibility
        - For `external` type, NLB target type depend on the annotation [nlb-target-type](#nlb-target-type)

    !!!warning "limitations"
        - This annotation should not be modified after service creation.

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-type: external
        ```

- <a name="nlb-target-type">`service.beta.kubernetes.io/aws-load-balancer-nlb-target-type`</a> specifies the target type to configure for NLB. You can choose between
`instance` and `ip`.
    - `instance` mode will route traffic to all EC2 instances within cluster on the [NodePort](https://kubernetes.io/docs/concepts/services-networking/service/#nodeport) opened for your service.

        !!!note ""
            service must be of type "NodePort" or "LoadBalancer" for `instance` targets

    - `ip` mode will route traffic directly to the pod IP.

        !!!note ""
            network plugin must use native AWS VPC networking configuration for pod IP, for example [Amazon VPC CNI plugin](https://github.com/aws/amazon-vpc-cni-k8s).

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: instance
        ```

- <a name="subnets">`service.beta.kubernetes.io/aws-load-balancer-subnets`</a> specifies the [Availability Zone](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
the NLB will route traffic to. See [Network Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/network-load-balancers.html#availability-zones) for more details.

    !!!tip
        Subnets are auto-discovered if this annotation is not specified, see [Subnet Discovery](../../deploy/subnet_discovery.md) for further details.

    !!!note ""
        You must specify at least one subnet in any of the AZs, both subnetID or subnetName(Name tag on subnets) can be used.

    !!!warning "limitations"
        - Each subnets must be from a different Availability Zone
        - AWS has restrictions on disabling existing subnets for NLB. As a result, you might not be able to edit this annotation once the NLB gets provisioned.

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-subnets: subnet-xxxx, mySubnet
        ```

- <a name="alpn-policy">`service.beta.kubernetes.io/aws-load-balancer-alpn-policy`</a> allows you to configure the [ALPN policies](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/create-tls-listener.html#alpn-policies)
on the load balancer.

    !!!note "requirements"
        TLS listener forwarding to a TLS target group

    !!!tip "supported policies"
        - `HTTP1Only` Negotiate only HTTP/1.*. The ALPN preference list is http/1.1, http/1.0.
        - `HTTP2Only` Negotiate only HTTP/2. The ALPN preference list is h2.
        - `HTTP2Optional` Prefer HTTP/1.* over HTTP/2 (which can be useful for HTTP/2 testing). The ALPN preference list is http/1.1, http/1.0, h2.
        - `HTTP2Preferred` Prefer HTTP/2 over HTTP/1.*. The ALPN preference list is h2, http/1.1, http/1.0.
        - `None` Do not negotiate ALPN. This is the default.

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-alpn-policy: HTTP2Preferred
        ```

- <a name="target-node-labels">`service.beta.kubernetes.io/aws-load-balancer-target-node-labels`</a> specifies which nodes to include in the target group registration for `instance` target type.

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-target-node-labels: label1=value1, label2=value2
        ```

## Resource attributes
NLB resource attributes can be controlled via the following annotations:

- <a name="proxy-protocol-v2">service.beta.kubernetes.io/aws-load-balancer-proxy-protocol</a> specifies whether to enable proxy protocol v2 on the target group.
Set to '*' to enable proxy protocol v2. This annotation takes precedence over the annotation `service.beta.kubernetes.io/aws-load-balancer-target-group-attributes`
for proxy protocol v2 configuration.

    !!!note ""
        The only valid value for this annotation is `*`.

- <a name="target-group-attributes">`service.beta.kubernetes.io/aws-load-balancer-target-group-attributes`</a> specifies the
[Target Group Attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#target-group-attributes) to be configured.

    !!!example
        - set the deregistration delay to 120 seconds (available range is 0-3600 seconds)
            ```
            service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: deregistration_delay.timeout_seconds=120
            ```
        - enable source IP affinity
            ```
            service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: stickiness.enabled=true,stickiness.type=source_ip
            ```
        - enable proxy protocol version 2
            ```
            service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: proxy_protocol_v2.enabled=true
            ```
        - enable connection termination on deregistration
            ```
            service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: deregistration_delay.connection_termination.enabled=true
            ```
        - enable [client IP preservation](https://docs.aws.amazon.com/elasticloadbalancing/latest/network/load-balancer-target-groups.html#client-ip-preservation)
            ```
            service.beta.kubernetes.io/aws-load-balancer-target-group-attributes: preserve_client_ip.enabled=true
            ```

## Access control
Load balancer access can be controllerd via following annotations:

- <a name="lb-source-ranges">`service.beta.kubernetes.io/load-balancer-source-ranges`</a> specifies the CIDRs that are allowed to access the NLB.

    !!!tip
        we recommend specifying CIDRs in the service `Spec.LoadBalancerSourceRanges` instead

    !!!note "Default"
        - `0.0.0.0/0` will be used if the IPAddressType is "ipv4"
        - `0.0.0.0/0` and `::/0` will be used if the IPAddressType is "dualstack"

    !!!warning ""
        This annotation will be ignored in case preserve client IP is not enabled.
        - preserve client IP is disabled by default for `IP` targets
        - preserve client IP is enabled by default for `instance` targets

    !!!example
        ```
        service.beta.kubernetes.io/load-balancer-source-ranges: 10.0.0.0/24
        ```

- <a name="lb-internal">`service.beta.kubernetes.io/aws-load-balancer-internal`</a> specifies whether the NLB will be internet-facing or internal.

    !!!example
        ```
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
        ```
