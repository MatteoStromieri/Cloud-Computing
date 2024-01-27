# ReplicaSet
A ReplicaSet is defined with fields, including a selector that specifies how to identify Pods it can acquire, a number of replicas indicating how many Pods it should be maintaining, and a pod template specifying the data of new Pods it should create to meet the number of replicas criteria. A ReplicaSet then fulfills its purpose by creating and deleting Pods as needed to reach the desired number. When a ReplicaSet needs to create new Pods, it uses its Pod template.

A ReplicaSet is linked to its Pods via the Pods' [metadata.ownerReferences](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#owners-dependents) field, which specifies what resource the current object is owned by. All Pods acquired by a ReplicaSet have their owning ReplicaSet's identifying information within their ownerReferences field. It's through this link that the ReplicaSet knows of the state of the Pods it is maintaining and plans accordingly.

A ReplicaSet identifies new Pods to acquire by using its selector. If there is a Pod that has no OwnerReference or the OwnerReference is not a [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) and it matches a ReplicaSet's selector, it will be immediately acquired by said ReplicaSet.

>[!Deployments]
>Deployments should be used instead of using directly ReplicaSets.
>
>A _Deployment_ provides declarative updates for [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) and [ReplicaSets](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).
>
>You describe a _desired state_ in a Deployment, and the Deployment [Controller](https://kubernetes.io/docs/concepts/architecture/controller/) changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

>[!Controller]
> A **Controller** is a _control loop_ is a non-terminating loop that regulates the state of a system.
# Service
- [Documentation](https://kubernetes.io/docs/concepts/services-networking/service/)

In Kubernetes, a Service is a method for exposing a network application that is running as one or more [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) in your cluster. You can run code in Pods, whether this is a code designed for a cloud-native world, or an older app you've containerized. You use a Service to make that set of Pods available on the network so that clients can interact with it.

From one moment to the next, you don't know how many of those Pods are working and healthy; you might not even know what those healthy Pods are named. Kubernetes [Pods](https://kubernetes.io/docs/concepts/workloads/pods/) are created and destroyed to match the desired state of your cluster. Pods are ephemeral resources (you should not expect that an individual Pod is reliable and durable).

Each Pod gets its own IP address (Kubernetes expects network plugins to ensure this). For a given Deployment in your cluster, the set of Pods running in one moment in time could be different from the set of Pods running that application a moment later.

This leads to a problem: if some set of Pods (call them "backends") provides functionality to other Pods (call them "frontends") inside your cluster, how do the frontends find out and keep track of which IP address to connect to, so that the frontend can use the backend part of the workload? The solution to this problem is *Services*.

Each Service object defines a logical set of endpoints (usually these endpoints are Pods) along with a policy about how to make those pods accessible. The set of Pods targeted by a Service is usually determined by a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) that you define.
## Defining a service
You can create, view or modify Service definitions using the Kubernetes API. Usually you use a tool such as `kubectl` to make those API calls for you.

For example, suppose you have a set of Pods that each listen on TCP port 9376 and are labelled as `app.kubernetes.io/name=MyApp`. You can define a Service to publish that TCP listener:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```
Applying this manifest creates a new Service named "my-service" with the default ClusterIP [service type](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types). The Service targets TCP port 9376 on any Pod with the `app.kubernetes.io/name: MyApp` label. Kubernetes assigns this Service an IP address (the _cluster IP_), that is used by the virtual IP address mechanism.

The controller for that Service continuously scans for Pods that match its selector, and then makes any necessary updates to the set of EndpointSlices for the Service.

>[!Note]
>A Service can map _any_ incoming `port` to a `targetPort`. By default and for convenience, the `targetPort` is set to the same value as the `port` field.

>[!Note]
>In Kubernetes, an EndpointSlice contains references to a set of network endpoints. The control plane automatically creates EndpointSlices for any Kubernetes Service that has a [selector](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/) specified. These EndpointSlices include references to all the Pods that match the Service selector.
>
>If there are so many endpoints for a Service that a threshold is reached, then Kubernetes adds another empty EndpointSlice and stores new endpoint information there.

## Multi-port Services
For some Services, you need to expose more than one port. Kubernetes lets you configure multiple port definitions on a Service object. When using multiple ports for a Service, you must give all of your ports names so that these are unambiguous. For example:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```
## Service types
>[!Note]
>The `type` must be specified under the `spec` inside the `yaml` file of the Service

The available `type` values and their behaviors are the following:

**ClusterIP**
Exposes the Service on a cluster-internal IP. Choosing this value makes the Service only reachable from within the cluster. This is the default that is used if you don't explicitly specify a `type` for a Service. You can expose the Service to the public internet using an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) or a [Gateway](https://gateway-api.sigs.k8s.io/).
>[!Note]
>If you define a Service that has the `.spec.clusterIP` set to `"None"` then Kubernetes does not assign an IP address.

You can specify your own cluster IP address as part of a `Service` creation request. To do this, set the `.spec.clusterIP` field. For example, if you already have an existing DNS entry that you wish to reuse, or legacy systems that are configured for a specific IP address and difficult to re-configure.

The IP address that you choose must be a valid IPv4 or IPv6 address from within the `service-cluster-ip-range` CIDR range that is configured for the API server.

**NodePort**
If you set the `type` field to `NodePort`, the Kubernetes control plane allocates a port from a range specified by `--service-node-port-range` flag (default: 30000-32767). Each node proxies that port (the same port number on every Node) into your Service.

For a node port Service, Kubernetes additionally allocates a port (TCP, UDP or SCTP to match the protocol of the Service). Every node in the cluster configures itself to listen on that assigned port and to forward traffic to one of the ready endpoints associated with that Service. You'll be able to contact the `type: NodePort` Service, from outside the cluster, by connecting to any node using the appropriate protocol (for example: TCP), and the appropriate port (as assigned to that Service).

If you want a **specific port number**, you can specify a value in the `nodePort` field. The control plane will either allocate you that port or report that the API transaction failed. This means that you need to take care of possible port collisions yourself. You also have to use a valid port number, one that's inside the range configured for NodePort use.

You can **set up nodes in your cluster to use a particular IP address** for serving node port services. You might want to do this if each node is connected to multiple networks (for example: one network for application traffic, and another network for traffic between nodes and the control plane).

If you want to specify particular IP address(es) to proxy the port, you can set the `--nodeport-addresses` flag for kube-proxy or the equivalent `nodePortAddresses` field of the [kube-proxy configuration file](https://kubernetes.io/docs/reference/config-api/kube-proxy-config.v1alpha1/) to particular IP block(s).

This flag takes a comma-delimited list of IP blocks (e.g. `10.0.0.0/8`, `192.0.2.0/25`) to specify IP address ranges that kube-proxy should consider as local to this node.

**LoadBalancer**
Exposes the Service externally using an external load balancer. Kubernetes does not directly offer a load balancing component; you must provide one.

On cloud providers which support external load balancers, setting the `type` field to `LoadBalancer`provisions a load balancer for your Service. The actual creation of the load balancer happens asynchronously, and information about the provisioned balancer is published in the Service's `.status.loadBalancer` field.

Traffic from the external load balancer is directed at the backend Pods. The cloud provider decides how it is load balanced.

**ExternalName**
Maps the Service to the contents of the `externalName` field (for example, to the hostname `api.foo.bar.example`). The mapping configures your cluster's DNS server to return a `CNAME` record with that external hostname value. No proxying of any kind is set up.
# DNS inside Kubernetes
Kubernetes creates DNS records for Services and Pods. You can contact Services with consistent DNS names instead of IP addresses.

Kubernetes publishes information about Pods and Services which is used to program DNS. Kubelet configures Pods' DNS so that running containers can lookup Services by name rather than IP.

Services defined in the cluster are assigned DNS names. By default, a client Pod's DNS search list includes the Pod's own namespace and the cluster's default domain
## DNS namespaces
A DNS query may return different results based on the namespace of the Pod making it. DNS queries that don't specify a namespace are limited to the Pod's namespace. Access Services in other namespaces by specifying it in the DNS query.

For example, consider a Pod in a `test` namespace. A `data` Service is in the `prod` namespace. A query for `data` returns no results, because it uses the Pod's `test` namespace. A query for `data.prod` returns the intended result, because it specifies the namespace.
# VirtualIPs and Service Proxies
Every [node](https://kubernetes.io/docs/concepts/architecture/nodes/) in a Kubernetes [cluster](https://kubernetes.io/docs/reference/glossary/?all=true#term-cluster) runs a [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/) The `kube-proxy` component is responsible for implementing a _virtual IP_ mechanism for [Services](https://kubernetes.io/docs/concepts/services-networking/service/) of `type` other than [`ExternalName`](https://kubernetes.io/docs/concepts/services-networking/service/#externalname)

Each instance of kube-proxy watches the Kubernetes [control plane](https://kubernetes.io/docs/reference/glossary/?all=true#term-control-plane) for the addition and removal of Service and EndpointSlice [objects](https://kubernetes.io/docs/concepts/overview/working-with-objects/#kubernetes-objects). For each Service, kube-proxy calls appropriate APIs (depending on the kube-proxy mode) to configure the node to capture traffic to the Service's `clusterIP` and `port`, and redirect that traffic to one of the Service's endpoints (usually a Pod, but possibly an arbitrary user-provided IP address). A control loop ensures that the rules on each node are reliably synchronized with the Service and EndpointSlice state as indicated by the API server.
# Gateway API
Make network services available by using an extensible, role-oriented, protocol-aware configuration mechanism.

Gateway API has three stable API kinds:
- **GatewayClass:** Defines a set of gateways with common configuration and managed by a controller that implements the class.
- **Gateway:** Defines an instance of traffic handling infrastructure, such as cloud load balancer.
- **HTTPRoute:** Defines HTTP-specific rules for mapping traffic from a Gateway listener to a representation of backend network endpoints. These endpoints are often represented as a [Service](https://kubernetes.io/docs/concepts/services-networking/service/).

A Gateway object is associated with exactly one GatewayClass; the GatewayClass describes the gateway controller responsible for managing Gateways of this class. One or more route kinds such as HTTPRoute, are then associated to Gateways. A Gateway can filter the routes that may be attached to its `listeners`, forming a bidirectional trust model with routes.

![[gateway-kind-relationships.svg]]
## GatewayClass
Gateways can be implemented by different controllers, often with different configurations. A Gateway must reference a GatewayClass that contains the name of the controller that implements the class.

A minimal GatewayClass example:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: example-class
spec:
  controllerName: example.com/gateway-controller
```
In this example, a controller that has implemented Gateway API is configured to manage GatewayClasses with the controller name `example.com/gateway-controller`. Gateways of this class will be managed by the implementation's controller.
## Gateway
A Gateway describes an instance of traffic handling infrastructure. It defines a network endpoint that can be used for processing traffic, i.e. filtering, balancing, splitting, etc. for backends such as a Service. For example, a Gateway may represent a cloud load balancer or an in-cluster proxy server that is configured to accept HTTP traffic.

A minimal Gateway resource example:
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```
In this example, an instance of traffic handling infrastructure is programmed to listen for HTTP traffic on port 80. Since the `addresses` field is unspecified, an address or hostname is assigned to the Gateway by the implementation's controller. This address is used as a network endpoint for processing traffic of backend network endpoints defined in routes.
## HTTPRoute
The HTTPRoute kind specifies routing behavior of HTTP requests from a Gateway listener to backend network endpoints. For a Service backend, an implementation may represent the backend network endpoint as a Service IP or the backing Endpoints of the Service. An HTTPRoute represents configuration that is applied to the underlying Gateway implementation. For example, defining a new HTTPRoute may result in configuring additional traffic routes in a cloud load balancer or in-cluster proxy server.

A minimal HTTPRoute example:

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /login
    backendRefs:
    - name: example-svc
      port: 8080
```

In this example, HTTP traffic from Gateway `example-gateway` with the Host: header set to `www.example.com` and the request path specified as `/login` will be routed to Service `example-svc` on port `8080`.
## Request flow
![[gateway-request-flow.svg]]
In this example, the request flow for a Gateway implemented as a reverse proxy is:
1. The client starts to prepare an HTTP request for the URL `http://www.example.com`
2. The client's DNS resolver queries for the destination name and learns a mapping to one or more IP addresses associated with the Gateway.
3. The client sends a request to the Gateway IP address; the reverse proxy receives the HTTP request and uses the Host: header to match a configuration that was derived from the Gateway and attached HTTPRoute.
4. Optionally, the reverse proxy can perform request header and/or path matching based on match rules of the HTTPRoute.
5. Optionally, the reverse proxy can modify the request; for example, to add or remove headers, based on filter rules of the HTTPRoute.
6. Lastly, the reverse proxy forwards the request to one or more backends.
---
# Lecture notes
**nodePort**
Praticamente la porta dell'ip associato al servizio viene esposta su tutti i nodi del nostro cluster. Quando arrivano richieste per l'IP del servizio su quella porta, le richieste vengono forwardade a uno dei pod nel nostro cluster tramite il KubeProxy del nodo che è stato contattato. Il KubeProxy fa il redirecting a un nodo che ha il servizio che stiamo richiedendo.

Quando richiediamo la traduzione di un nome al DNS otteniamo l'ip del servizio nodePort, allora contattiamo un nodo con quell'ip usando quella particolare porta, il kubeproxy capisce che stiamo contattando un pod nell'insieme di quelli associati a quel particolare IP:Port e allora rimanda il traffico a un nodo, se non riceve risposta allora lo  

Non è il loadbalancer perfetto, abbiamo bisogno di un componente di terze parti per ottenere HA
