# K8s Adimission Controller

Admission Controller 是一个拦截器，请求通过认证之后，请求被存储起来之前拦截发送给 Kuberenetes API Server 的请求。

> An admission controller is a piece of code that intercepts requests to the Kubernetes API server prior to persistence of the object, but after the request is authenticated and authorized.

简而言之，Kubernetes Admission Controller 是控制和强制使用集群的插件。可以将它们视为拦截（已认证）API 请求的 Gatekeeper，并且可以更改请求对象或完全拒绝该请求。准入控制过程分为两个阶段：首先执行 Mutating 阶段，然后执行 Validating 阶段。因此，Admission Controller 可以充当变异或验证控制器或两者的组合。例如，LimitRanger Admission Controller 可以使用默认资源请求和限制来扩展 Pod（更改阶段），并验证具有明确设置的资源要求的 Pod 不超过 LimitRange 对象中指定的每个命名空间限制（验证阶段）。

![Admission Controller Phases](https://s2.ax1x.com/2020/01/01/lGDzh6.png)

根据以上的流程，Kubernetes 将 AC 分为三种：

- validating，验证型。用于验证 K8s 的资源定义是否符合规则。
- mutating，修改型。用于修改 K8s 的资源定义，比如加个 label 什么的。
- 二者皆是，即同一个 AC，既是验证型又是修改型。

多个 Admission Controller 会形成一个 Admission Chain（链条），修改型的在前面先执行，验证型的在后面后执行，这样验证型的才能去验证修改的对不对。
我们可以查看目前启用的 AC：

```sh
$ kube-apiserver -h | grep enable-admission-plugins

--admission-control strings              Admission is divided into two phases. In the first phase, only mutating admission plugins run. In the second phase, only validating admission plugins run. The names in the below list may represent a validating plugin, a mutating plugin, or both. The order of plugins in which they are passed to this flag does not matter. Comma-delimited list of: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, DefaultStorageClass, DefaultTolerationSeconds, DenyEscalatingExec, DenyExecOnPrivileged, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, Initializers, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodPreset, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, ValidatingAdmissionWebhook. (DEPRECATED: Use --enable-admission-plugins or --disable-admission-plugins instead. Will be removed in a future version.)

--enable-admission-plugins strings       admission plugins that should be enabled in addition to default enabled ones (NamespaceLifecycle, LimitRanger, ServiceAccount, Priority, DefaultTolerationSeconds, DefaultStorageClass, PersistentVolumeClaimResize, MutatingAdmissionWebhook, ValidatingAdmissionWebhook, ResourceQuota). Comma-delimited list of admission plugins: AlwaysAdmit, AlwaysDeny, AlwaysPullImages, DefaultStorageClass, DefaultTolerationSeconds, DenyEscalatingExec, DenyExecOnPrivileged, EventRateLimit, ExtendedResourceToleration, ImagePolicyWebhook, Initializers, LimitPodHardAntiAffinityTopology, LimitRanger, MutatingAdmissionWebhook, NamespaceAutoProvision, NamespaceExists, NamespaceLifecycle, NodeRestriction, OwnerReferencesPermissionEnforcement, PersistentVolumeClaimResize, PersistentVolumeLabel, PodNodeSelector, PodPreset, PodSecurityPolicy, PodTolerationRestriction, Priority, ResourceQuota, SecurityContextDeny, ServiceAccount, StorageObjectInUseProtection, ValidatingAdmissionWebhook. The order of plugins in this flag does not matter.
```

# Adimission Webhook

Admission Controller 有着非常丰富的使用场景，譬如 Istio 就是采用 Admission Webhook 实现 SideCar 容器自动注入；我们也可以自动地为应用打标签，或者自动将 SideCar 容器注册到 Pod 中。

![收集应用日志的 Sidecar 容器](https://s2.ax1x.com/2020/01/01/lGrgC6.md.png)

Admission webhooks are HTTP callbacks that receive admission requests and do something with them. 用户可以定义两种 webhook，validating admission webhook、mutating admission webhook。一个用于验证，另一个用于修改。Webhook 回调，接收 API Server 发送的 admissionReview 请求，并返回 admissionResponse。典型的 ValidatingWebhookConfiguration 的资源定义如下：

```yml
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: <name of this configuration object>
webhooks:
  - name: <webhook name, e.g., pod-policy.example.io>
    rules:
      - apiGroups:
          - ""
        apiVersions:
          - v1
        operations:
          - CREATE
        resources:
          - pods
    clientConfig:
      service:
        namespace: <namespace of the front-end service>
        name: <name of the front-end service>
      caBundle: <pem encoded ca cert that signs the server cert used by the webhook>
```

其中 rules 定义了匹配规则，当发给 API Server 的请求满足该规则的时候，API Server 就会给 clientConfig 中配置的 service 发送 Admission 请求。如果存在多个 K8s 集群，我们也可以共享 WebHook API：

![WebHook API](https://s2.ax1x.com/2020/01/01/lGsEMF.md.png)

Cluster c1、c2 中的 Webhook 配置会指向各自集群内部的 service，这个 service 其实是 headless service，它指向的是 cluster A 的 service(需要暴露给其它集群能够访问，nodePort 也可以)，这样所有集群就共享一个 Webhook API 了。

# 编程控制

编写自己的 Webhook 需要完成以下步骤：

- 创建 TLS Certificate，即证书
- 编写服务端代码，服务端代码需要使用证书
- 根据证书创建 K8s sercret
- 创建 K8s Deployment 和 Service
- 创建 K8s WebhookConfiguration，其中需要使用之前创建的证书

可以参考 [k8s-examples/webhook](https://github.com/BE-Kits/k8s-examples) 中的相关示例。

AdmissionWebhook 可以像拦截器一样拦截 K8s api 请求，要实现修改功能用 MutatingAdmissionWebhook，实现验证功能用 ValidatingAdmissionWebhook。AdmissionReview 结构体 request 请求信息通过 AdmissionReview 的 Request 字段可以获取到；response 通过 AdmissionReview 的 Response 字段设置返回

```go
// AdmissionReview describes an admission review request/response.
 type AdmissionReview struct {
     metav1.TypeMeta `json:",inline"`
     // Request describes the attributes for the admission request.
     // +optional
     Request *AdmissionRequest `json:"request,omitempty" protobuf:"bytes,1,opt,name=request"`
     // Response describes the attributes for the admission response.
     // +optional
     Response *AdmissionResponse `json:"response,omitempty" protobuf:"bytes,2,opt,name=response"`
 }
```

mutating 是通过 json patch 方式实现的，对应的结构体定义如下：

```go
type patchOperation struct {
   Op    string      `json:"op"`
   Path  string      `json:"path"`
   Value interface{} `json:"value,omitempty"`
 }

 patches = append(patches, patchOperation{
   Op:    "add",
   Path: "/metadata/annotations",
   Value: true,
 })
```
