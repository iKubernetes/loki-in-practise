# 在Kubernetes上部署Loki

本示例提供了Loki的两种模式的部署案例，具体使用有选其一即可：

- loki：Simple Scalable模式，提供了values.yaml文件，需要使用helm进行部署
  - 存储类型默认为s3，需要对接到MinIO上
- loiki-monolithic-mode：Monolithic模式，资源配置文件，需要使用kubectl部署
  - 依赖于Kubernetes集群上的默认存储类来创建PVC和PV；
  - 存储类型默认为filesystem
