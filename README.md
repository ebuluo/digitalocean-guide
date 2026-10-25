# DigitalOcean Droplet 选购与配置指南（2026年版）：架构差异、计费陷阱与理性选型

We earn commissions when you shop through the links below.

通过下方链接购买服务，我们可能获得佣金，您无需为此额外付费。

---

在当今的云计算生态中，寻找一台用来跑代码、部署后端或搭建微服务的云主机，选择往往呈现两个极端：一端是以 AWS、Google Cloud 为代表的超大规模公有云，功能极度强大，但控制台复杂庞大，网络网关、IAM 策略和计费账单足以让中小型团队和独立开发者望而却步；另一端则是各类极其廉价的非托管型单一 VPS 服务商，虽然价格低廉，但缺乏私有网络、云防火墙、自动化快照和托管数据库等配套支撑。

DigitalOcean 正好处在这两个极端之间。它的核心计算单元称为 **Droplet**，本质上就是预装了主流 Linux 发行版的虚拟专用服务器（VPS）。对于不想深陷公有云繁琐配置泥潭、又需要稳定基础设施与标准 API 的开发者而言，DigitalOcean 一直是一个兼具实用性与操作体验的经典选择。

本文将梳理当前收录的 **163 款 bundled CPU Droplet** 规格，拆解 **Basic、General Purpose、CPU-Optimized、Memory-Optimized、Storage-Optimized** 五大系列的底层硬件差异、真实的计费规则与常见误区，并提供一套务实的选型参考与完整的分组参数目录。

---

## 一、产品定位与网络现实：它适合谁，不适合谁？

### 1. 为什么选择 DigitalOcean？
- **直观干净的控制台**：创建一台机器只需几十秒，网络拓扑、监控图表和安全组一目了然，极大地降低了认知负担。
- **可预测的捆绑账单**：不同于按调用量、复杂 I/O 读写单独拆碎计费的模式，bundled CPU Droplet 将计算核心、内存、本地磁盘及一定额度的月度出站流量打包销售，成本模型非常透明。
- **完善的开发者生态**：提供开箱即用的 VPC（私有网络）、免费平台级 Cloud Firewalls（云防火墙）、Floating IP（浮动 IP）、Volumes（块存储卷）、托管数据库（PostgreSQL/MySQL/Redis）以及官方维护的 Terraform Provider 和命令行工具 `doctl`。

### 2. 必须看清的边界与网络认知
- **非面向中国大陆网络优化的线路**：DigitalOcean 的数据中心分布在北美（纽约、旧金山、多伦多）、欧洲（阿姆斯特丹、法兰克福、伦敦）及亚太（新加坡、班加罗尔、悉尼）等地。**平台并未针对中国大陆三大运营商做任何专门的路由优化（如 CN2 GIA、AS9929、CMIN2 等）**。国内公网直连访问属于标准国际路由，延迟与稳定性受国际骨干网波动影响较大。
  - **适合场景**：面向海外用户或出海业务的 Web 应用、API 接口服务、全球分布式节点、开发测试集群、爬虫与数据中继、跨国团队的开发环境。
  - **不适合场景**：面向中国大陆普通终端用户且不使用 CDN 加速的无备案直连建站、对低延迟有极致要求的游戏加速代理。
- **实际可用性与价格提醒**：本文列出的价格基于当前收录目录基准。不同规格的地区可选性、最终下单价格与库存情况，以 [DigitalOcean 官网总入口](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) 后台创建页面为准。需要特别提醒的是：官方推荐链接为通用的新客注册与控制台入口，并不会预先指定某款特定套餐，在进入后台后需手动勾选本文对应的系列和规格代码。

---

## 二、必须看懂的计费规则与常见“扣费陷阱”

很多新手在初次接触云主机时，往往因为对计费周期的理解偏差而产生意外账单。以下是使用 DigitalOcean 时必须牢记的几个计费核心点：

### 1. 秒级计费与 672 小时月度上限
- bundled CPU Droplets 采用**按秒计费**（Per-second Billing），单次生命周期最低按 **60 秒或 0.01 美元（取较高者）**计算。这意味着临时拉起一台高配实例跑测试，只要几分钟就会被正常释放，不必为一整小时甚至整月买单。
- 在一个月度账单周期内，Droplet 的累计计费上限为 **672 小时（相当于 28 天）**。一旦实例在当月持续运行满 672 小时，后续直到月底的时间都不会再产生额外的按小时算力费用。
- **特别区分**：本文详述的是针对标准 bundled CPU Droplets 的计费逻辑；DigitalOcean 旗下的 v5 系列等新型架构或特定产品线并不适用这套捆绑月上限计费机制，切勿将两者的计费模型混淆。

### 2. 最大误区：关机（Power Off）依然会全额扣费！
这是云新手最容易踩的坑：**在操作系统内执行 `poweroff` 或在控制台点击“Power Off”，Droplet 依然在计费！**

为什么？因为关机只是停止了虚拟机的 CPU 运算，但平台依然在底层物理宿主机上为你独占保留着 CPU 核心分配、内存配额、固定的外网 IPv4 地址以及机载高速 NVMe/SSD 磁盘。只要这些物理资源被锁定、没有交还给公用资源池，计费时钟就不会停止。

> **停止计费的唯一方法：Destroy（销毁实例）**  
> 如果某台机器你暂时不用，但希望彻底停止产生账单，正确的做法是：  
> 1. 先对该 Droplet 拍一个手动快照（Snapshot）；  
> 2. 在控制台点击 **Destroy** 彻底释放该实例；  
> 3. 下次需要恢复时，从该快照直接恢复（Restore）出一台全新的 Droplet 即可。

### 3. 流量池机制与额外附加成本
- **入站流量（Inbound）全免**：任何流向你 Droplet 的数据均不收费。
- **出站流量配额与流量池（Bandwidth Pooling）**：
  - 每台 Droplet 都自带定量的每月出站流量额度（从 500 GiB 到 12,000 GiB 不等）。
  - 在同一个账户（或团队）下，所有同构 Droplet 的出站流量会自动汇聚成一个共享的**全球流量池**。例如你拥有两台自带 1,000 GiB 额度的机器，只要账户总出站流量不超过 2,000 GiB，即使其中一台用了 1,800 GiB、另一台只用了 100 GiB，也不会产生超额罚款。
  - 一旦超出流量池配额，超额出站流量按 **0.01 美元/GiB** 计费。
- **独立计费组件**：
  - **自动备份（Backups）**：开启后由平台每周自动生成还原点，费用通常为 Droplet 月费的 20% 左右。
  - **快照（Snapshots）**：按实际存储容量计费（通常约为 0.06 美元/GiB/月）。
  - **独立块存储卷（Volumes Block Storage）**：用于灵活挂载、动态扩容的独立网络磁盘，通常按 0.10 美元/GiB/月计费。

---

## 三、五大系列核心差别：共享与独占，以及如何取舍？

在挑选具体规格前，首先要理解 DigitalOcean 实例的核心硬件划分。

```
                    ┌─────────────────────────┐
                    │  DigitalOcean Droplets  │
                    └────────────┬────────────┘
                                 │
         ┌───────────────────────┴───────────────────────┐
         │                                               │
┌────────┴────────┐                             ┌────────┴────────┐
│   Basic 系列    │                             │   Dedicated     │
│   (共享 vCPU)   │                             │   (专用 vCPU)   │
└────────┬────────┘                             └────────┬────────┘
         │                                               │
   ┌─────┴─────┐                   ┌─────────────┬───────┴─────┬─────────────┐
   │  Regular  │                   │             │             │             │
   │Premium AMD│              ┌────┴────┐   ┌────┴────┐   ┌────┴────┐   ┌────┴────┐
   │Prem. Intel│              │ General │   │  CPU-   │   │ Memory- │   │Storage- │
   └───────────┘              │ Purpose │   │Optimized│   │Optimized│   │Optimized│
                              │  (1:4)  │   │  (1:2)  │   │  (1:8)  │   │(1:8+NVMe│
                              └─────────┘   └─────────┘   └─────────┘   └─────────┘
```

### 1. 核心分水岭：共享 vCPU（Shared）vs 专用 vCPU（Dedicated）
- **Basic（共享 vCPU）**：虚拟处理核心与其他用户的虚拟机共享底层物理核心。平台允许多核短时间突发（Burst），但如果有程序长时间维持 100% 满载，调度器会进行限频以保护物理机上的“邻居”。适合绝大多数轻量 Web 服务、开发测试、个人博客、微型数据库或低并发脚本。
- **其他四大系列（专用 vCPU）**：General Purpose、CPU-Optimized、Memory-Optimized、Storage-Optimized 均采用物理核心超线程独占机制。即使你的代码 24 小时跑满 100% 算力，也不会受到平台限频，性能表现极其平稳，适合企业级生产环境。

### 2. 处理器与磁盘分级：Regular vs Premium Intel vs Premium AMD
在 Basic 和部分专用系列中，你能看到底层硬件分类：
- **Regular（标准型）**：采用成熟代际的 Intel Xeon 处理器，搭配标准 SATA/NVMe SSD。它的单位成本最低，算力中规中矩，适合对极致单核主频不敏感的常规负载。
- **Premium Intel（增强型 Intel）**：采用较新代际的 Intel Xeon Scalable 处理器，单核睿频更高，全系标配高性能 NVMe SSD。如果你的应用严重依赖单线程瓶颈（例如 Node.js 单进程服务、某些 Python/PHP 框架或未分库的轻量关系型数据库），它的响应速度明显优于 Regular。
- **Premium AMD（增强型 AMD）**：采用 AMD EPYC 系列处理器，同样标配 NVMe SSD。在多线程并发吞吐、多任务计算和程序编译打包场景下，AMD 的多核综合表现非常出色。

### 3. 五大系列的内存配比与典型用途

| 系列名称 | vCPU:内存 黄金比例 | 适用核心场景 | 选型权衡点 |
| :--- | :--- | :--- | :--- |
| **Basic** | 1:0.5 至 1:4 宽泛范围 | 开发测试、个人站点、微服务、轻量应用 | 最便宜（$4起），但 CPU 不保独占，忌长时间满载 |
| **CPU-Optimized** | **1:2**（如 2核4G、4核8G） | CI/CD 构建机、视频转码、数学运算、批处理 | 计算性能极致，但不适合大内存驻留型应用 |
| **General Purpose** | **1:4**（如 2核8G、4核16G） | 主力 Web 服务、中型电商、高并发 API 网关 | 业界公认最均衡的生产配比，抗突发能力强 |
| **Memory-Optimized** | **1:8**（如 2核16G、4核32G） | Redis 缓存池、Elasticsearch、JVM 重度应用 | 单位内存成本摊薄，适合只缺内存不缺算力的业务 |
| **Storage-Optimized** | **1:8 + 大容量机载 NVMe** | 亿级 NoSQL、大型 MySQL、时序分析、日志索引 | 本地超高速 NVMe 直接挂载，摆脱网络块存储 IOPS 瓶颈 |

---

## 四、从创建到上线：标准工程化操作步骤

为了避免不必要的支出和安全漏洞，在进入具体规格挑选前，建议遵循以下标准流程创建 Droplet：

1. **设置账单告警（Billing Alerts）**：登录后，在控制台的 **Billing -> Account Alerts** 页面，设置一个月度消费警报金额（如 20 美元或 50 美元）。当账户产生意外出站流量或忘记销毁测试机时，系统会第一时间发邮件提醒。
2. **强制使用 SSH 密钥（SSH Keys）**：创建页面中切勿使用简单密码认证。在本地生成密钥对（推荐 `ssh-keygen -t ed25519`），将公钥上传至 DigitalOcean。创建机器时勾选该公钥，根用户（root）密码登录将被默认禁用，彻底阻断扫描爆破攻击。
3. **选择就近的数据中心（Datacenter Region）**：根据你的主要访问群体物理位置选择区域。例如面向美洲选 NYC（纽约）或 SFO（旧金山）；面向欧洲选 FRA（法兰克福）或 AMS（阿姆斯特丹）；面向东南亚或澳洲选 SGP（新加坡）或 SYD（悉尼）。
4. **选择系统镜像（Image）**：生产环境推荐选择最新版本的 **Ubuntu LTS** 或 **Debian**。除非明确图省事，否则建议通过裸系统搭配自动化配置脚本来搭建环境，避免 1-Click 应用镜像中残留不必要的陈旧依赖。
5. **挂载云防火墙（Cloud Firewall）**：在 **Networking -> Firewalls** 中创建一条免费的平台级防火墙策略。将入站规则限制为仅允许 80、443 端口，并将 SSH（22端口）限制为你个人的固定公网 IP 或特定堡垒机网段，在流量到达虚拟机操作系统之前将其拦在外部网络层。
6. **彻底销毁不再使用的实例**：测试完成后，右键实例菜单选择 **Destroy**，确认红色按钮完成释放，确保物理资源被归还，停止计费。

---

## 五、全系列 163 款规格参数与选型指南

接下来将进入完整的 163 款规格参数目录。你可以根据实际预算与技术架构，在对应的分组中找到适合的型号代码（Slug）。

---

### basic

**选型建议**：  
Basic 是出镜率最高、也是绝大多数独立开发者与中小型项目的起点。如果只是搭建一个个人技术博客、部署一个自动化监控探针或跑简单的低频爬虫，入门级的 [Basic Regular · 1 vCPU / 0.5 GiB / 10 GiB · s-1vcpu-512mb-10gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$4/月）或 [Basic Regular · 1 vCPU / 1 GiB / 25 GiB · s-1vcpu-1gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$6/月）性价比最高。需要注意的是，0.5 GiB 内存跑现代 Linux 发行版非常吃紧，连 `apt update` 或编译一个 npm 包都可能触发 OOM Killer，生产环境下建议至少从 1 GiB 内存起步。

如果你打算在 Basic 上运行带有少量并发的动态后端（如 Spring Boot、Django、Rails 或搭配 SQLite/MySQL），建议优先选择配备了 NVMe 固态硬盘的 [Basic Premium AMD · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$14/月）或同配置的 [Basic Premium Intel · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$14/月）。每月多付两美元，换来的是明显更低的磁盘 I/O 延迟和更高的单核执行效率。

#### 完整规格表：Basic（31款）

| 套餐 / 规格标识 | 配置与流量 | 价格与周期 |
| --- | --- | --- |
| [Basic Regular · 1 vCPU / 0.5 GiB / 10 GiB · s-1vcpu-512mb-10gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 0.5GiB内存 · 10GiB SSD · 500GiB/月 | US$4/月上限；参考US$0.00595/小时 |
| [Basic Regular · 1 vCPU / 1 GiB / 25 GiB · s-1vcpu-1gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 1GiB内存 · 25GiB SSD · 1000GiB/月 | US$6/月上限；参考US$0.00893/小时 |
| [Basic Premium AMD · 1 vCPU / 1 GiB / 25 GiB · s-1vcpu-1gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 1GiB内存 · 25GiB NVMe SSD · 1000GiB/月 | US$7/月上限；参考US$0.01042/小时 |
| [Basic Premium Intel · 1 vCPU / 1 GiB / 25 GiB · s-1vcpu-1gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 1GiB内存 · 25GiB NVMe SSD · 1000GiB/月 | US$7/月上限；参考US$0.01042/小时 |
| [Basic Premium Intel · 1 vCPU / 1 GiB / 35 GiB · s-1vcpu-1gb-35gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 1GiB内存 · 35GiB NVMe SSD · 1000GiB/月 | US$8/月上限；参考US$0.0119/小时 |
| [Basic Regular · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 2GiB内存 · 50GiB SSD · 2000GiB/月 | US$12/月上限；参考US$0.01786/小时 |
| [Basic Premium AMD · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 2GiB内存 · 50GiB NVMe SSD · 2000GiB/月 | US$14/月上限；参考US$0.02083/小时 |
| [Basic Premium Intel · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 2GiB内存 · 50GiB NVMe SSD · 2000GiB/月 | US$14/月上限；参考US$0.02083/小时 |
| [Basic Premium Intel · 1 vCPU / 2 GiB / 70 GiB · s-1vcpu-2gb-70gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 1vCPU · 2GiB内存 · 70GiB NVMe SSD · 2000GiB/月 | US$16/月上限；参考US$0.02381/小时 |
| [Basic Regular · 2 vCPU / 2 GiB / 60 GiB · s-2vcpu-2gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 2GiB内存 · 60GiB SSD · 3000GiB/月 | US$18/月上限；参考US$0.02679/小时 |
| [Basic Premium AMD · 2 vCPU / 2 GiB / 60 GiB · s-2vcpu-2gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 2GiB内存 · 60GiB NVMe SSD · 3000GiB/月 | US$21/月上限；参考US$0.03125/小时 |
| [Basic Premium Intel · 2 vCPU / 2 GiB / 60 GiB · s-2vcpu-2gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 2GiB内存 · 60GiB NVMe SSD · 3000GiB/月 | US$21/月上限；参考US$0.03125/小时 |
| [Basic Premium Intel · 2 vCPU / 2 GiB / 90 GiB · s-2vcpu-2gb-90gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 2GiB内存 · 90GiB NVMe SSD · 3000GiB/月 | US$24/月上限；参考US$0.03571/小时 |
| [Basic Regular · 2 vCPU / 4 GiB / 80 GiB · s-2vcpu-4gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 80GiB SSD · 4000GiB/月 | US$24/月上限；参考US$0.03571/小时 |
| [Basic Premium AMD · 2 vCPU / 4 GiB / 80 GiB · s-2vcpu-4gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 80GiB NVMe SSD · 4000GiB/月 | US$28/月上限；参考US$0.04167/小时 |
| [Basic Premium Intel · 2 vCPU / 4 GiB / 80 GiB · s-2vcpu-4gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 80GiB NVMe SSD · 4000GiB/月 | US$28/月上限；参考US$0.04167/小时 |
| [Basic Premium Intel · 2 vCPU / 4 GiB / 120 GiB · s-2vcpu-4gb-120gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 120GiB NVMe SSD · 4000GiB/月 | US$32/月上限；参考US$0.04762/小时 |
| [Basic Premium AMD · 2 vCPU / 8 GiB / 100 GiB · s-2vcpu-8gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 100GiB NVMe SSD · 5000GiB/月 | US$42/月上限；参考US$0.0625/小时 |
| [Basic Premium Intel · 2 vCPU / 8 GiB / 160 GiB · s-2vcpu-8gb-160gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 160GiB NVMe SSD · 5000GiB/月 | US$48/月上限；参考US$0.07143/小时 |
| [Basic Regular · 4 vCPU / 8 GiB / 160 GiB · s-4vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 160GiB SSD · 5000GiB/月 | US$48/月上限；参考US$0.07143/小时 |
| [Basic Premium AMD · 4 vCPU / 8 GiB / 160 GiB · s-4vcpu-8gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 160GiB NVMe SSD · 5000GiB/月 | US$56/月上限；参考US$0.08333/小时 |
| [Basic Premium Intel · 4 vCPU / 8 GiB / 160 GiB · s-4vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 160GiB NVMe SSD · 5000GiB/月 | US$56/月上限；参考US$0.08333/小时 |
| [Basic Premium Intel · 4 vCPU / 8 GiB / 240 GiB · s-4vcpu-8gb-240gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 240GiB NVMe SSD · 6000GiB/月 | US$64/月上限；参考US$0.09524/小时 |
| [Basic Premium AMD · 4 vCPU / 16 GiB / 200 GiB · s-4vcpu-16gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 200GiB NVMe SSD · 8000GiB/月 | US$84/月上限；参考US$0.125/小时 |
| [Basic Premium Intel · 4 vCPU / 16 GiB / 320 GiB · s-4vcpu-16gb-320gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 320GiB NVMe SSD · 8000GiB/月 | US$96/月上限；参考US$0.14286/小时 |
| [Basic Regular · 8 vCPU / 16 GiB / 320 GiB · s-8vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 320GiB SSD · 6000GiB/月 | US$96/月上限；参考US$0.14286/小时 |
| [Basic Premium AMD · 8 vCPU / 16 GiB / 320 GiB · s-8vcpu-16gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 320GiB NVMe SSD · 6000GiB/月 | US$112/月上限；参考US$0.16667/小时 |
| [Basic Premium Intel · 8 vCPU / 16 GiB / 320 GiB · s-8vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 320GiB NVMe SSD · 6000GiB/月 | US$112/月上限；参考US$0.16667/小时 |
| [Basic Premium Intel · 8 vCPU / 16 GiB / 480 GiB · s-8vcpu-16gb-480gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 480GiB NVMe SSD · 9000GiB/月 | US$128/月上限；参考US$0.19048/小时 |
| [Basic Premium AMD · 8 vCPU / 32 GiB / 400 GiB · s-8vcpu-32gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 400GiB NVMe SSD · 10000GiB/月 | US$168/月上限；参考US$0.25/小时 |
| [Basic Premium Intel · 8 vCPU / 32 GiB / 640 GiB · s-8vcpu-32gb-640gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 640GiB NVMe SSD · 10000GiB/月 | US$192/月上限；参考US$0.28571/小时 |

---

### cpu-optimized

**选型建议**：  
CPU-Optimized 系列严格恪守 **1:2 的 vCPU/内存配比**（即 1 核对应 2 GiB 内存），且全系采用**专用 vCPU**。这意味着你的虚拟机独享分配的物理硬件核心，不会出现共享核心时由于宿主机超售带来的性能毛刺。

这一系列是纯算力消耗型业务的理想工作马。典型场景包括：
- GitLab CI / GitHub Actions 的自托管 Runner 构建节点
- 视频编解码、FFmpeg 流式分片转码服务器
- 批处理任务队列、大数据清洗工人节点（Worker Nodes）
- 无状态的纯高并发 API 计算后端

入门推荐关注 [CPU-Optimized Regular · 2 vCPU / 4 GiB / 25 GiB · c-2](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$42/月），对于大多数持续运行的打包工具已绰绰有余。如果有较高磁盘读写或需要更强单核算力，可以上调至搭载 NVMe 的 [CPU-Optimized Premium Intel · 4 vCPU / 8 GiB / 50 GiB · c-4-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$109/月）。

#### 完整规格表：CPU-Optimized（36款）

| 套餐 / 规格标识 | 配置与流量 | 价格与周期 |
| --- | --- | --- |
| [CPU-Optimized Regular · 2 vCPU / 4 GiB / 25 GiB · c-2](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 25GiB SSD · 4000GiB/月 | US$42/月上限；参考US$0.0625/小时 |
| [CPU-Optimized Regular · 2 vCPU / 4 GiB / 50 GiB · c2-2vcpu-4gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 50GiB SSD · 4000GiB/月 | US$47/月上限；参考US$0.06994/小时 |
| [CPU-Optimized Regular · 2 vCPU / 4 GiB / 125 GiB · c5-2vcpu-4gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 4GiB内存 · 125GiB SSD · 4000GiB/月 | US$62/月上限；参考US$0.09226/小时 |
| [CPU-Optimized Regular · 4 vCPU / 8 GiB / 50 GiB · c-4](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 50GiB SSD · 5000GiB/月 | US$84/月上限；参考US$0.125/小时 |
| [CPU-Optimized Regular · 4 vCPU / 8 GiB / 100 GiB · c2-4vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 100GiB SSD · 5000GiB/月 | US$94/月上限；参考US$0.13988/小时 |
| [CPU-Optimized Premium Intel · 4 vCPU / 8 GiB / 50 GiB · c-4-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 50GiB NVMe SSD · 5000GiB/月 | US$109/月上限；参考US$0.1622/小时 |
| [CPU-Optimized Premium Intel · 4 vCPU / 8 GiB / 100 GiB · c2-4vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 100GiB NVMe SSD · 5000GiB/月 | US$122/月上限；参考US$0.18155/小时 |
| [CPU-Optimized Regular · 4 vCPU / 8 GiB / 250 GiB · c5-4vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 250GiB SSD · 5000GiB/月 | US$124/月上限；参考US$0.18452/小时 |
| [CPU-Optimized Premium Intel · 4 vCPU / 8 GiB / 250 GiB · c5-4vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 8GiB内存 · 250GiB NVMe SSD · 5000GiB/月 | US$161/月上限；参考US$0.23958/小时 |
| [CPU-Optimized Regular · 8 vCPU / 16 GiB / 100 GiB · c-8](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 100GiB SSD · 6000GiB/月 | US$168/月上限；参考US$0.25/小时 |
| [CPU-Optimized Regular · 8 vCPU / 16 GiB / 200 GiB · c2-8vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 200GiB SSD · 6000GiB/月 | US$188/月上限；参考US$0.27976/小时 |
| [CPU-Optimized Premium Intel · 8 vCPU / 16 GiB / 100 GiB · c-8-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 100GiB NVMe SSD · 6000GiB/月 | US$218/月上限；参考US$0.3244/小时 |
| [CPU-Optimized Premium Intel · 8 vCPU / 16 GiB / 200 GiB · c2-8vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 200GiB NVMe SSD · 6000GiB/月 | US$244/月上限；参考US$0.3631/小时 |
| [CPU-Optimized Regular · 8 vCPU / 16 GiB / 500 GiB · c5-8vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 500GiB SSD · 6000GiB/月 | US$248/月上限；参考US$0.36905/小时 |
| [CPU-Optimized Premium Intel · 8 vCPU / 16 GiB / 500 GiB · c5-8vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 16GiB内存 · 500GiB NVMe SSD · 6000GiB/月 | US$322/月上限；参考US$0.47917/小时 |
| [CPU-Optimized Regular · 16 vCPU / 32 GiB / 200 GiB · c-16](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 200GiB SSD · 7000GiB/月 | US$336/月上限；参考US$0.5/小时 |
| [CPU-Optimized Regular · 16 vCPU / 32 GiB / 400 GiB · c2-16vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 400GiB SSD · 7000GiB/月 | US$376/月上限；参考US$0.55952/小时 |
| [CPU-Optimized Premium Intel · 16 vCPU / 32 GiB / 200 GiB · c-16-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 200GiB NVMe SSD · 7000GiB/月 | US$437/月上限；参考US$0.6503/小时 |
| [CPU-Optimized Premium Intel · 16 vCPU / 32 GiB / 400 GiB · c2-16vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 400GiB NVMe SSD · 7000GiB/月 | US$489/月上限；参考US$0.72768/小时 |
| [CPU-Optimized Regular · 16 vCPU / 32 GiB / 1000 GiB · c5-16vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 1000GiB SSD · 7000GiB/月 | US$496/月上限；参考US$0.7381/小时 |
| [CPU-Optimized Premium Intel · 16 vCPU / 32 GiB / 1000 GiB · c5-16vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 32GiB内存 · 1000GiB NVMe SSD · 7000GiB/月 | US$645/月上限；参考US$0.95982/小时 |
| [CPU-Optimized Regular · 32 vCPU / 64 GiB / 400 GiB · c-32](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 400GiB SSD · 9000GiB/月 | US$672/月上限；参考US$1/小时 |
| [CPU-Optimized Regular · 32 vCPU / 64 GiB / 800 GiB · c2-32vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 800GiB SSD · 9000GiB/月 | US$752/月上限；参考US$1.11905/小时 |
| [CPU-Optimized Premium Intel · 32 vCPU / 64 GiB / 400 GiB · c-32-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 400GiB NVMe SSD · 9000GiB/月 | US$874/月上限；参考US$1.3006/小时 |
| [CPU-Optimized Premium Intel · 32 vCPU / 64 GiB / 800 GiB · c2-32vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 800GiB NVMe SSD · 9000GiB/月 | US$978/月上限；参考US$1.45536/小时 |
| [CPU-Optimized Regular · 32 vCPU / 64 GiB / 2000 GiB · c5-32vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 2000GiB SSD · 9000GiB/月 | US$992/月上限；参考US$1.47619/小时 |
| [CPU-Optimized Regular · 48 vCPU / 96 GiB / 600 GiB · c-48](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 600GiB SSD · 11000GiB/月 | US$1008/月上限；参考US$1.5/小时 |
| [CPU-Optimized Regular · 48 vCPU / 96 GiB / 1200 GiB · c2-48vcpu-96gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 1200GiB SSD · 11000GiB/月 | US$1128/月上限；参考US$1.67857/小时 |
| [CPU-Optimized Premium Intel · 32 vCPU / 64 GiB / 2000 GiB · c5-32vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 64GiB内存 · 2000GiB NVMe SSD · 9000GiB/月 | US$1290/月上限；参考US$1.91964/小时 |
| [CPU-Optimized Premium Intel · 48 vCPU / 96 GiB / 600 GiB · c-48-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 600GiB NVMe SSD · 11000GiB/月 | US$1310/月上限；参考US$1.9494/小时 |
| [CPU-Optimized Premium Intel · 48 vCPU / 96 GiB / 1200 GiB · c2-48vcpu-96gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 1200GiB NVMe SSD · 11000GiB/月 | US$1466/月上限；参考US$2.18155/小时 |
| [CPU-Optimized Regular · 48 vCPU / 96 GiB / 3000 GiB · c5-48vcpu-96gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 3000GiB SSD · 11000GiB/月 | US$1488/月上限；参考US$2.21429/小时 |
| [CPU-Optimized Premium Intel · 60 vCPU / 120 GiB / 750 GiB · c-60-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 120GiB内存 · 750GiB NVMe SSD · 12000GiB/月 | US$1639/月上限；参考US$2.43899/小时 |
| [CPU-Optimized Premium Intel · 60 vCPU / 120 GiB / 1500 GiB · c2-60vcpu-120gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 120GiB内存 · 1500GiB NVMe SSD · 12000GiB/月 | US$1834/月上限；参考US$2.72917/小时 |
| [CPU-Optimized Premium Intel · 48 vCPU / 96 GiB / 3000 GiB · c5-48vcpu-96gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 96GiB内存 · 3000GiB NVMe SSD · 11000GiB/月 | US$1934/月上限；参考US$2.87798/小时 |
| [CPU-Optimized Premium Intel · 60 vCPU / 120 GiB / 3750 GiB · c5-60vcpu-120gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 120GiB内存 · 3750GiB NVMe SSD · 12000GiB/月 | US$2419/月上限；参考US$3.5997/小时 |

---

### general-purpose

**选型建议**：  
General Purpose 系列采用公认最均衡的 **1:4 核心内存配比**（如 2核8G、4核16G、8核32G），同样提供**专用 vCPU**。

很多技术负责人在规划生产环境时最常犯的错误，是在 Basic 上通过多次盲目升级来获取更多内存。实际上，一旦单机承载了主站生产流量、多个 Docker 容器组合或活跃的企业级后台，Basic 的共享 CPU 会在流量高峰期因为争抢宿主机资源而引发响应延迟飙升。此时，直接切入 General Purpose 是更稳健的选择。

- 推荐起步：[General Purpose Regular · 2 vCPU / 8 GiB / 25 GiB · g-2vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$63/月）。这个配置能非常从容地同时跑一套 Nginx + 多个应用后端服务 + 本地辅存。
- 如果你的项目是中型电商平台或动态多租户 SaaS 系统，需要更稳定的突发读取能力，搭载 NVMe 的 [General Purpose Premium Intel · 4 vCPU / 16 GiB / 60 GiB · g-4vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$151/月）能提供极具弹性的系统吞吐余量。

#### 完整规格表：General Purpose（39款）

| 套餐 / 规格标识 | 配置与流量 | 价格与周期 |
| --- | --- | --- |
| [General Purpose Regular · 2 vCPU / 8 GiB / 25 GiB · g-2vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 25GiB SSD · 4000GiB/月 | US$63/月上限；参考US$0.09375/小时 |
| [General Purpose Regular · 2 vCPU / 8 GiB / 50 GiB · gd-2vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 50GiB SSD · 4000GiB/月 | US$68/月上限；参考US$0.10119/小时 |
| [General Purpose Premium Intel · 2 vCPU / 8 GiB / 30 GiB · g-2vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 30GiB NVMe SSD · 4000GiB/月 | US$76/月上限；参考US$0.1131/小时 |
| [General Purpose Premium Intel · 2 vCPU / 8 GiB / 60 GiB · gd-2vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 60GiB NVMe SSD · 4000GiB/月 | US$79/月上限；参考US$0.11756/小时 |
| [General Purpose Regular · 2 vCPU / 8 GiB / 163 GiB · g6_5-2vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 163GiB SSD · 4000GiB/月 | US$90.6/月上限；参考US$0.13482/小时 |
| [General Purpose Premium Intel · 2 vCPU / 8 GiB / 165 GiB · g5_5-2vcpu-8gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 8GiB内存 · 165GiB NVMe SSD · 4000GiB/月 | US$92.2/月上限；参考US$0.1372/小时 |
| [General Purpose Regular · 4 vCPU / 16 GiB / 50 GiB · g-4vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 50GiB SSD · 5000GiB/月 | US$126/月上限；参考US$0.1875/小时 |
| [General Purpose Regular · 4 vCPU / 16 GiB / 100 GiB · gd-4vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 100GiB SSD · 5000GiB/月 | US$136/月上限；参考US$0.20238/小时 |
| [General Purpose Premium Intel · 4 vCPU / 16 GiB / 60 GiB · g-4vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 60GiB NVMe SSD · 5000GiB/月 | US$151/月上限；参考US$0.2247/小时 |
| [General Purpose Premium Intel · 4 vCPU / 16 GiB / 120 GiB · gd-4vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 120GiB NVMe SSD · 5000GiB/月 | US$158/月上限；参考US$0.23512/小时 |
| [General Purpose Regular · 4 vCPU / 16 GiB / 325 GiB · g6_5-4vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 325GiB SSD · 5000GiB/月 | US$181/月上限；参考US$0.26935/小时 |
| [General Purpose Premium Intel · 4 vCPU / 16 GiB / 330 GiB · g5_5-4vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 16GiB内存 · 330GiB NVMe SSD · 5000GiB/月 | US$183.4/月上限；参考US$0.27292/小时 |
| [General Purpose Regular · 8 vCPU / 32 GiB / 100 GiB · g-8vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 100GiB SSD · 6000GiB/月 | US$252/月上限；参考US$0.375/小时 |
| [General Purpose Regular · 8 vCPU / 32 GiB / 200 GiB · gd-8vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 200GiB SSD · 6000GiB/月 | US$272/月上限；参考US$0.40476/小时 |
| [General Purpose Premium Intel · 8 vCPU / 32 GiB / 120 GiB · g-8vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 120GiB NVMe SSD · 6000GiB/月 | US$302/月上限；参考US$0.4494/小时 |
| [General Purpose Premium Intel · 8 vCPU / 32 GiB / 240 GiB · gd-8vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 240GiB NVMe SSD · 6000GiB/月 | US$317/月上限；参考US$0.47173/小时 |
| [General Purpose Regular · 8 vCPU / 32 GiB / 650 GiB · g6_5-8vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 650GiB SSD · 6000GiB/月 | US$362/月上限；参考US$0.53869/小时 |
| [General Purpose Premium Intel · 8 vCPU / 32 GiB / 660 GiB · g5_5-8vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 32GiB内存 · 660GiB NVMe SSD · 6000GiB/月 | US$366.8/月上限；参考US$0.54583/小时 |
| [General Purpose Regular · 16 vCPU / 64 GiB / 200 GiB · g-16vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 200GiB SSD · 7000GiB/月 | US$504/月上限；参考US$0.75/小时 |
| [General Purpose Regular · 16 vCPU / 64 GiB / 400 GiB · gd-16vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 400GiB SSD · 7000GiB/月 | US$544/月上限；参考US$0.80952/小时 |
| [General Purpose Premium Intel · 16 vCPU / 64 GiB / 240 GiB · g-16vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 240GiB NVMe SSD · 7000GiB/月 | US$605/月上限；参考US$0.9003/小时 |
| [General Purpose Premium Intel · 16 vCPU / 64 GiB / 480 GiB · gd-16vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 480GiB NVMe SSD · 7000GiB/月 | US$634/月上限；参考US$0.94345/小时 |
| [General Purpose Regular · 16 vCPU / 64 GiB / 1300 GiB · g6_5-16vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 1300GiB SSD · 7000GiB/月 | US$724/月上限；参考US$1.07738/小时 |
| [General Purpose Premium Intel · 16 vCPU / 64 GiB / 1320 GiB · g5_5-16vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 64GiB内存 · 1320GiB NVMe SSD · 7000GiB/月 | US$734.6/月上限；参考US$1.09315/小时 |
| [General Purpose Regular · 32 vCPU / 128 GiB / 400 GiB · g-32vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 400GiB SSD · 8000GiB/月 | US$1008/月上限；参考US$1.5/小时 |
| [General Purpose Regular · 32 vCPU / 128 GiB / 800 GiB · gd-32vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 800GiB SSD · 8000GiB/月 | US$1088/月上限；参考US$1.61905/小时 |
| [General Purpose Premium Intel · 32 vCPU / 128 GiB / 480 GiB · g-32vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 480GiB NVMe SSD · 8000GiB/月 | US$1210/月上限；参考US$1.8006/小时 |
| [General Purpose Regular · 40 vCPU / 160 GiB / 500 GiB · g-40vcpu-160gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 40vCPU · 160GiB内存 · 500GiB SSD · 9000GiB/月 | US$1260/月上限；参考US$1.875/小时 |
| [General Purpose Premium Intel · 32 vCPU / 128 GiB / 960 GiB · gd-32vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 960GiB NVMe SSD · 8000GiB/月 | US$1268/月上限；参考US$1.8869/小时 |
| [General Purpose Regular · 40 vCPU / 160 GiB / 1000 GiB · gd-40vcpu-160gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 40vCPU · 160GiB内存 · 1000GiB SSD · 9000GiB/月 | US$1360/月上限；参考US$2.02381/小时 |
| [General Purpose Regular · 32 vCPU / 128 GiB / 2600 GiB · g6_5-32vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 2600GiB SSD · 8000GiB/月 | US$1448/月上限；参考US$2.15476/小时 |
| [General Purpose Premium Intel · 32 vCPU / 128 GiB / 2640 GiB · g5_5-32vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 128GiB内存 · 2640GiB NVMe SSD · 8000GiB/月 | US$1469.2/月上限；参考US$2.18631/小时 |
| [General Purpose Regular · 40 vCPU / 160 GiB / 3250 GiB · g6_5-40vcpu-160gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 40vCPU · 160GiB内存 · 3250GiB SSD · 9000GiB/月 | US$1810/月上限；参考US$2.69345/小时 |
| [General Purpose Premium Intel · 48 vCPU / 192 GiB / 720 GiB · g-48vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 192GiB内存 · 720GiB NVMe SSD · 9000GiB/月 | US$1814/月上限；参考US$2.6994/小时 |
| [General Purpose Premium Intel · 48 vCPU / 192 GiB / 1440 GiB · gd-48vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 192GiB内存 · 1440GiB NVMe SSD · 11000GiB/月 | US$1901/月上限；参考US$2.82887/小时 |
| [General Purpose Premium Intel · 48 vCPU / 192 GiB / 3960 GiB · g5_5-48vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 192GiB内存 · 3960GiB NVMe SSD · 9000GiB/月 | US$2202.8/月上限；参考US$3.27798/小时 |
| [General Purpose Premium Intel · 60 vCPU / 240 GiB / 900 GiB · g-60vcpu-240gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 240GiB内存 · 900GiB NVMe SSD · 10000GiB/月 | US$2269/月上限；参考US$3.37649/小时 |
| [General Purpose Premium Intel · 60 vCPU / 240 GiB / 1800 GiB · gd-60vcpu-240gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 240GiB内存 · 1800GiB NVMe SSD · 10000GiB/月 | US$2378/月上限；参考US$3.53869/小时 |
| [General Purpose Premium Intel · 60 vCPU / 240 GiB / 4950 GiB · g5_5-60vcpu-240gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 60vCPU · 240GiB内存 · 4950GiB NVMe SSD · 10000GiB/月 | US$2755/月上限；参考US$4.0997/小时 |

---

### memory-optimized

**选型建议**：  
Memory-Optimized 系列将核心内存配比直接拉升到 **1:8 的悬殊比例**（1 个专用 vCPU 搭配 8 GiB 内存）。全系标配 NVMe SSD，专门为对内存容量极度饥渴的应用所设计。

典型的适用软件包括：
- **Redis / Memcached**：大规模高速内存键值缓存
- **Elasticsearch / OpenSearch**：需要大量堆内存处理 Lucene 倒排索引的搜索引擎节点
- **Apache Kafka / RabbitMQ**：有大量内存消息缓冲区堆积需求的中间件
- **高负载关系型数据库的只读副本（Read Replica）**：通过大内存最大化 InnoDB Buffer Pool，将更多热点数据常驻内存

对于中小型数据系统，入门款 [Memory-Optimized Regular · 2 vCPU / 16 GiB / 50 GiB · m-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$84/月）性价比极高。如果你的 Elasticsearch 索引很大、或者需要更高的写入吞吐，可以考虑带更大磁盘空间的 [Memory-Optimized Regular · 2 vCPU / 16 GiB / 150 GiB · m3-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$104/月）。

#### 完整规格表：Memory-Optimized（32款）

| 套餐 / 规格标识 | 配置与流量 | 价格与周期 |
| --- | --- | --- |
| [Memory-Optimized Regular · 2 vCPU / 16 GiB / 50 GiB · m-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 50GiB NVMe SSD · 4000GiB/月 | US$84/月上限；参考US$0.125/小时 |
| [Memory-Optimized Premium Intel · 2 vCPU / 16 GiB / 50 GiB · m-2vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 50GiB NVMe SSD · 4000GiB/月 | US$99/月上限；参考US$0.14732/小时 |
| [Memory-Optimized Regular · 2 vCPU / 16 GiB / 150 GiB · m3-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 150GiB NVMe SSD · 4000GiB/月 | US$104/月上限；参考US$0.15476/小时 |
| [Memory-Optimized Premium Intel · 2 vCPU / 16 GiB / 150 GiB · m3-2vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 150GiB NVMe SSD · 4000GiB/月 | US$110/月上限；参考US$0.16369/小时 |
| [Memory-Optimized Regular · 2 vCPU / 16 GiB / 300 GiB · m6-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 300GiB NVMe SSD · 4000GiB/月 | US$131/月上限；参考US$0.19494/小时 |
| [Memory-Optimized Regular · 4 vCPU / 32 GiB / 100 GiB · m-4vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 100GiB NVMe SSD · 6000GiB/月 | US$168/月上限；参考US$0.25/小时 |
| [Memory-Optimized Premium Intel · 4 vCPU / 32 GiB / 100 GiB · m-4vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 100GiB NVMe SSD · 6000GiB/月 | US$198/月上限；参考US$0.29464/小时 |
| [Memory-Optimized Regular · 4 vCPU / 32 GiB / 300 GiB · m3-4vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 300GiB NVMe SSD · 6000GiB/月 | US$208/月上限；参考US$0.30952/小时 |
| [Memory-Optimized Premium Intel · 4 vCPU / 32 GiB / 300 GiB · m3-4vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 300GiB NVMe SSD · 6000GiB/月 | US$220/月上限；参考US$0.32738/小时 |
| [Memory-Optimized Regular · 4 vCPU / 32 GiB / 600 GiB · m6-4vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 600GiB NVMe SSD · 6000GiB/月 | US$262/月上限；参考US$0.38988/小时 |
| [Memory-Optimized Regular · 8 vCPU / 64 GiB / 200 GiB · m-8vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 200GiB NVMe SSD · 7000GiB/月 | US$336/月上限；参考US$0.5/小时 |
| [Memory-Optimized Premium Intel · 8 vCPU / 64 GiB / 200 GiB · m-8vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 200GiB NVMe SSD · 7000GiB/月 | US$396/月上限；参考US$0.58929/小时 |
| [Memory-Optimized Regular · 8 vCPU / 64 GiB / 600 GiB · m3-8vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 600GiB NVMe SSD · 7000GiB/月 | US$416/月上限；参考US$0.61905/小时 |
| [Memory-Optimized Premium Intel · 8 vCPU / 64 GiB / 600 GiB · m3-8vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 600GiB NVMe SSD · 7000GiB/月 | US$440/月上限；参考US$0.65476/小时 |
| [Memory-Optimized Regular · 8 vCPU / 64 GiB / 1170 GiB · m6-8vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 1170GiB NVMe SSD · 7000GiB/月 | US$524/月上限；参考US$0.77976/小时 |
| [Memory-Optimized Regular · 16 vCPU / 128 GiB / 400 GiB · m-16vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 400GiB NVMe SSD · 8000GiB/月 | US$672/月上限；参考US$1/小时 |
| [Memory-Optimized Premium Intel · 16 vCPU / 128 GiB / 400 GiB · m-16vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 400GiB NVMe SSD · 8000GiB/月 | US$792/月上限；参考US$1.17857/小时 |
| [Memory-Optimized Regular · 16 vCPU / 128 GiB / 1170 GiB · m3-16vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 1170GiB NVMe SSD · 8000GiB/月 | US$832/月上限；参考US$1.2381/小时 |
| [Memory-Optimized Premium Intel · 16 vCPU / 128 GiB / 1200 GiB · m3-16vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 1200GiB NVMe SSD · 8000GiB/月 | US$880/月上限；参考US$1.30952/小时 |
| [Memory-Optimized Regular · 24 vCPU / 192 GiB / 600 GiB · m-24vcpu-192gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 600GiB NVMe SSD · 9000GiB/月 | US$1008/月上限；参考US$1.5/小时 |
| [Memory-Optimized Regular · 16 vCPU / 128 GiB / 2340 GiB · m6-16vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 2340GiB NVMe SSD · 8000GiB/月 | US$1048/月上限；参考US$1.55952/小时 |
| [Memory-Optimized Premium Intel · 24 vCPU / 192 GiB / 600 GiB · m-24vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 600GiB NVMe SSD · 9000GiB/月 | US$1188/月上限；参考US$1.76786/小时 |
| [Memory-Optimized Regular · 24 vCPU / 192 GiB / 1760 GiB · m3-24vcpu-192gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 1760GiB NVMe SSD · 9000GiB/月 | US$1248/月上限；参考US$1.85714/小时 |
| [Memory-Optimized Premium Intel · 24 vCPU / 192 GiB / 1800 GiB · m3-24vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 1800GiB NVMe SSD · 9000GiB/月 | US$1320/月上限；参考US$1.96429/小时 |
| [Memory-Optimized Regular · 32 vCPU / 256 GiB / 800 GiB · m-32vcpu-256gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 800GiB NVMe SSD · 10000GiB/月 | US$1344/月上限；参考US$2/小时 |
| [Memory-Optimized Regular · 24 vCPU / 192 GiB / 3520 GiB · m6-24vcpu-192gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 3520GiB NVMe SSD · 9000GiB/月 | US$1572/月上限；参考US$2.33929/小时 |
| [Memory-Optimized Premium Intel · 32 vCPU / 256 GiB / 800 GiB · m-32vcpu-256gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 800GiB NVMe SSD · 10000GiB/月 | US$1584/月上限；参考US$2.35714/小时 |
| [Memory-Optimized Regular · 32 vCPU / 256 GiB / 2340 GiB · m3-32vcpu-256gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 2340GiB NVMe SSD · 10000GiB/月 | US$1664/月上限；参考US$2.47619/小时 |
| [Memory-Optimized Premium Intel · 32 vCPU / 256 GiB / 2400 GiB · m3-32vcpu-256gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 2400GiB NVMe SSD · 10000GiB/月 | US$1760/月上限；参考US$2.61905/小时 |
| [Memory-Optimized Regular · 32 vCPU / 256 GiB / 4690 GiB · m6-32vcpu-256gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 4690GiB NVMe SSD · 10000GiB/月 | US$2096/月上限；参考US$3.11905/小时 |
| [Memory-Optimized Premium Intel · 48 vCPU / 384 GiB / 1200 GiB · m-48vcpu-384gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 384GiB内存 · 1200GiB NVMe SSD · 11000GiB/月 | US$2376/月上限；参考US$3.53571/小时 |
| [Memory-Optimized Premium Intel · 48 vCPU / 384 GiB / 3600 GiB · m3-48vcpu-384gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 384GiB内存 · 3600GiB NVMe SSD · 11000GiB/月 | US$2640/月上限；参考US$3.92857/小时 |

---

### storage-optimized

**选型建议**：  
Storage-Optimized 系列在保持 **1:8 专用 vCPU/内存高配比** 的基础上，最大的杀手锏是**直挂超大容量的宿主机本地 NVMe 磁盘**（从 300 GiB 一直到 7,030 GiB）。

很多用户会问：普通 Droplet 也可以通过后台挂载 Volumes 块存储来扩容硬盘，为什么要专门选 Storage-Optimized？
- **区别在于 I/O 延迟与吞吐路径**：Volumes 走的是内部网络存储协议，即使内网延迟极低，也无法与物理直连在宿主机 PCI-E 通道上的 NVMe 固态硬盘相提并论。
- 如果你的系统承载着每秒几万次随机写入的 MongoDB、Cassandra、PostgreSQL 主库，或是大体量 ClickHouse 列式分析，网络存储很容易成为最大系统瓶颈。Storage-Optimized 正是为了消除这一瓶颈而生。

入门推荐关注 [Storage-Optimized Regular · 2 vCPU / 16 GiB / 300 GiB · so-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$131/月）或带有更强主频表现的 [Storage-Optimized Premium Intel · 2 vCPU / 16 GiB / 300 GiB · so-2vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)（US$131/月），两者月费相同，但后者在单核计算和单盘 I/O 响应上更具优势。

#### 完整规格表：Storage-Optimized（25款）

| 套餐 / 规格标识 | 配置与流量 | 价格与周期 |
| --- | --- | --- |
| [Storage-Optimized Premium Intel · 2 vCPU / 16 GiB / 300 GiB · so-2vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 300GiB NVMe SSD · 4000GiB/月 | US$131/月上限；参考US$0.19494/小时 |
| [Storage-Optimized Regular · 2 vCPU / 16 GiB / 300 GiB · so-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 300GiB NVMe SSD · 4000GiB/月 | US$131/月上限；参考US$0.19494/小时 |
| [Storage-Optimized Premium Intel · 2 vCPU / 16 GiB / 450 GiB · so1_5-2vcpu-16gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 450GiB NVMe SSD · 4000GiB/月 | US$139/月上限；参考US$0.20685/小时 |
| [Storage-Optimized Regular · 2 vCPU / 16 GiB / 450 GiB · so1_5-2vcpu-16gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 2vCPU · 16GiB内存 · 450GiB NVMe SSD · 4000GiB/月 | US$163/月上限；参考US$0.24256/小时 |
| [Storage-Optimized Premium Intel · 4 vCPU / 32 GiB / 600 GiB · so-4vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 600GiB NVMe SSD · 6000GiB/月 | US$262/月上限；参考US$0.38988/小时 |
| [Storage-Optimized Regular · 4 vCPU / 32 GiB / 600 GiB · so-4vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 600GiB NVMe SSD · 6000GiB/月 | US$262/月上限；参考US$0.38988/小时 |
| [Storage-Optimized Premium Intel · 4 vCPU / 32 GiB / 900 GiB · so1_5-4vcpu-32gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 900GiB NVMe SSD · 6000GiB/月 | US$278/月上限；参考US$0.41369/小时 |
| [Storage-Optimized Regular · 4 vCPU / 32 GiB / 900 GiB · so1_5-4vcpu-32gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 4vCPU · 32GiB内存 · 900GiB NVMe SSD · 6000GiB/月 | US$326/月上限；参考US$0.48512/小时 |
| [Storage-Optimized Premium Intel · 8 vCPU / 64 GiB / 1170 GiB · so-8vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 1170GiB NVMe SSD · 7000GiB/月 | US$524/月上限；参考US$0.77976/小时 |
| [Storage-Optimized Regular · 8 vCPU / 64 GiB / 1170 GiB · so-8vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 1170GiB NVMe SSD · 7000GiB/月 | US$524/月上限；参考US$0.77976/小时 |
| [Storage-Optimized Premium Intel · 8 vCPU / 64 GiB / 1760 GiB · so1_5-8vcpu-64gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 1760GiB NVMe SSD · 7000GiB/月 | US$556/月上限；参考US$0.82738/小时 |
| [Storage-Optimized Regular · 8 vCPU / 64 GiB / 1760 GiB · so1_5-8vcpu-64gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 8vCPU · 64GiB内存 · 1760GiB NVMe SSD · 7000GiB/月 | US$652/月上限；参考US$0.97024/小时 |
| [Storage-Optimized Premium Intel · 16 vCPU / 128 GiB / 2340 GiB · so-16vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 2340GiB NVMe SSD · 8000GiB/月 | US$1048/月上限；参考US$1.55952/小时 |
| [Storage-Optimized Regular · 16 vCPU / 128 GiB / 2340 GiB · so-16vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 2340GiB NVMe SSD · 8000GiB/月 | US$1048/月上限；参考US$1.55952/小时 |
| [Storage-Optimized Premium Intel · 16 vCPU / 128 GiB / 3520 GiB · so1_5-16vcpu-128gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 3520GiB NVMe SSD · 8000GiB/月 | US$1112/月上限；参考US$1.65476/小时 |
| [Storage-Optimized Regular · 16 vCPU / 128 GiB / 3520 GiB · so1_5-16vcpu-128gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 16vCPU · 128GiB内存 · 3520GiB NVMe SSD · 8000GiB/月 | US$1304/月上限；参考US$1.94048/小时 |
| [Storage-Optimized Premium Intel · 24 vCPU / 192 GiB / 3520 GiB · so-24vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 3520GiB NVMe SSD · 9000GiB/月 | US$1572/月上限；参考US$2.33929/小时 |
| [Storage-Optimized Regular · 24 vCPU / 192 GiB / 3520 GiB · so-24vcpu-192gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 3520GiB NVMe SSD · 9000GiB/月 | US$1572/月上限；参考US$2.33929/小时 |
| [Storage-Optimized Premium Intel · 24 vCPU / 192 GiB / 5270 GiB · so1_5-24vcpu-192gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 5270GiB NVMe SSD · 9000GiB/月 | US$1668/月上限；参考US$2.48214/小时 |
| [Storage-Optimized Regular · 24 vCPU / 192 GiB / 5270 GiB · so1_5-24vcpu-192gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 24vCPU · 192GiB内存 · 5270GiB NVMe SSD · 9000GiB/月 | US$1956/月上限；参考US$2.91071/小时 |
| [Storage-Optimized Premium Intel · 32 vCPU / 256 GiB / 4690 GiB · so-32vcpu-256gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 4690GiB NVMe SSD · 10000GiB/月 | US$2096/月上限；参考US$3.11905/小时 |
| [Storage-Optimized Regular · 32 vCPU / 256 GiB / 4690 GiB · so-32vcpu-256gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 4690GiB NVMe SSD · 10000GiB/月 | US$2096/月上限；参考US$3.11905/小时 |
| [Storage-Optimized Premium Intel · 32 vCPU / 256 GiB / 7030 GiB · so1_5-32vcpu-256gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 7030GiB NVMe SSD · 10000GiB/月 | US$2224/月上限；参考US$3.30952/小时 |
| [Storage-Optimized Regular · 32 vCPU / 256 GiB / 7030 GiB · so1_5-32vcpu-256gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 32vCPU · 256GiB内存 · 7030GiB NVMe SSD · 10000GiB/月 | US$2608/月上限；参考US$3.88095/小时 |
| [Storage-Optimized Premium Intel · 48 vCPU / 384 GiB / 7000 GiB · so-48vcpu-384gb-intel](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) | 48vCPU · 384GiB内存 · 7000GiB NVMe SSD · 11000GiB/月 | US$3144/月上限；参考US$4.67857/小时 |

---

## 六、写在最后：务实的选型原则

挑选云服务最忌讳的是“为想象中的过剩流量提前买单”。

在 DigitalOcean 的架构体系下，**垂直升级（Resize）是非常平滑的操作**：
- 如果你刚启动一个新项目，从最便宜的 [Basic Regular · 1 vCPU / 1 GiB / 25 GiB · s-1vcpu-1gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409) 开始验证技术栈，成本压力最小；
- 随着访问量攀升，如果只是偶发性卡顿，可以一键平滑变配到带有 NVMe 的 [Basic Premium AMD · 1 vCPU / 2 GiB / 50 GiB · s-1vcpu-2gb-amd](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)；
- 只有当监控指标显示 CPU 核心被持续压满、并发请求开始排队时，再考虑迁往以专用物理核心为基底的 [General Purpose Regular · 2 vCPU / 8 GiB / 25 GiB · g-2vcpu-8gb](https://www.awin1.com/cread.php?awinmid=123996&awinaffid=3131409)。

时刻利用好**快照（Snapshot）**与**彻底销毁（Destroy）**的组合拳来管理临时环境，在后台设置好预算预警，把每一分计算预算都花在刀刃上。

---

[VPSMap · VPS 套餐与价格对比](https://vpsmap.net/)
<!-- GEMINI_ARTICLE_COMPLETE -->
