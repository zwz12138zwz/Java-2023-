# Java工程师 2023版（已完结，视频+课件+代码）
获课：xingkeit.top/16766/Java工程师 2023版（已完结，视频+课件+代码）


SpringCloud LoadBalancer：客户端负载均衡深度实战

在微服务架构中，一个服务往往部署多个实例来分担流量。如何让请求合理地落到不同实例上，是决定系统吞吐量和稳定性的关键问题。Spring Cloud LoadBalancer 作为 Spring 官方推出的客户端负载均衡组件，已经全面替代了进入维护模式的 Netflix Ribbon，成为当前微服务负载均衡的标准选型。

客户端负载均衡的核心思路

传统的服务端负载均衡（如 Nginx）由一个中间节点统一转发请求，而客户端负载均衡把"选择实例"这件事交给了调用方自己。每个服务消费者在本地维护一份可用实例列表，发起调用前自行根据策略挑选目标实例，再直接发起请求。这种模式去掉了中间转发层，减少了单点瓶颈，同时与服务发现组件（如 Nacos、Eureka）天然契合，能动态感知实例的上下线。

Spring Cloud LoadBalancer 正是基于这一思路设计的。它通过 LoadBalancerClient 接口，从注册中心获取目标服务的实例列表，再按配置的策略选出一个实例，将请求中的服务名替换为真实的 IP 和端口，最终完成调用。

从 Ribbon 到 LoadBalancer 的演进

Ribbon 曾是 Spring Cloud 生态中最主流的负载均衡组件，提供了丰富的策略和配置能力。但随着 Netflix 宣布其进入维护模式，Ribbon 的线程模型和配置复杂度逐渐成为短板。Spring 官方推出的 Spring Cloud LoadBalancer 采用响应式编程模型，与 WebFlux 无缝集成，配置更简洁，扩展性也更强。

在实战中，新版 Spring Cloud 项目需要手动引入 spring-cloud-starter-loadbalancer 依赖，否则启动时会直接报错。这一点是很多从旧版本迁移的开发者容易踩的坑。

与 OpenFeign 的集成实战

在企业级微服务项目中，远程调用几乎都采用 OpenFeign。OpenFeign 原生适配 Spring Cloud LoadBalancer，引入依赖后即可自动生效，无需额外配置。调用方只需在接口上使用 @FeignClient 注解声明目标服务名，框架会在底层自动完成服务发现、实例选择和 URL 重构。

相比 RestTemplate 需要手动拼接 URL 和整合负载均衡组件，OpenFeign 的声明式写法让远程调用和本地方法调用几乎没有区别，代码更简洁，日志调试也更方便。线上正式项目统一推荐使用 OpenFeign，RestTemplate 仅适合本地临时测试。

内置策略与自定义扩展

Spring Cloud LoadBalancer 默认采用轮询策略（RoundRobin），适合大多数均匀分配的场景。除此之外，还支持随机策略、基于响应时间的权重分配等。在实际生产中，不同实例的硬件配置往往不同，通过权重策略可以让性能更强的实例承担更多流量，避免资源浪费。

当内置策略无法满足需求时，可以通过实现 ReactorServiceInstanceLoadBalancer 接口来编写自定义负载均衡逻辑。比如实现"每个实例连续处理 N 次请求后再切换"的批量轮询策略，或者结合业务标签实现灰度路由。自定义策略通过 @LoadBalancerClient 注解绑定到具体服务，实现细粒度的策略控制。

生产环境的实战建议

在策略选择上，均匀分配场景用默认的轮询即可；实例性能差异大时使用加权策略；对可用性要求高的场景可以结合重试机制，在实例故障时快速切换到其他节点。

性能方面，建议将底层 HTTP 客户端从默认的 HttpURLConnection 替换为 Apache HttpClient 或 OkHttp，开启连接池复用，避免每次请求都重新建立连接。同时合理配置连接超时和读取超时时间，防止慢实例拖垮整个调用链路。

健康检查同样不可忽视。Spring Cloud LoadBalancer 支持自定义 HealthCheckServiceInstanceListSupplier，可以定期探测实例的存活状态，及时剔除不可用节点，避免请求打到"僵尸实例"上。

Spring Cloud LoadBalancer 的价值不仅在于把请求分发出去，更在于它提供了一套可扩展、可观测的负载均衡基础设施。理解它的原理和实战要点，是构建高可用微服务架构的必修课。

