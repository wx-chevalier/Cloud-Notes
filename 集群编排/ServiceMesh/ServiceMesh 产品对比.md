# 产品对比

- Linkerd（读作“link-dee”）：2016 年发布的元老级项目。Linkerd 最初是从 Twitter 开发的一个库中分离出来的，在领域内的另一个重量级项目 Conduit 加入后，便形成了 Linkerd 2.0 的基础。

- Envoy：由 Lyft 创建，Envoy 充当服务网格的“数据平面”，与“控制平面”相匹配，提供比较完整的服务网格服务。

- Istio：由 Lyft、IBM 和谷歌联合开发而成，是服务于 Envoy 等代理的“控制平面”。虽然默认是与 Envoy 成对匹配，但是它们都可以与其他平台配对使用。

- HashiCorp Consul：在 Consul 1.2 版本后，推出了名为 Connect 的功能，这个功能为 HashiCorp 的分布式系统的服务发现和配置部分，添加了服务加密和基于身份的授权的功能。这个使得使 HashiCorp Consul 成为非常完整的服务网格。