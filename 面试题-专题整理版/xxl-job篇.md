[toc]

---

## 1. xxl-job的原理

<img src="./assets/image-20251017170310124.png" alt="image-20251017170310124" style="zoom:80%;" />

xxl-job使用db锁来控制哪个定时任务执行，底层通过数据库来做的，性能差一些，收费版还可以不使用数据库，而是Redis

Elastic-job使用的是zookeeper，而不是数据库，没有xxl-job功能多，没有后台管理；可以支持每个节点占多少分片，xxl-job不支持（自动分配）

