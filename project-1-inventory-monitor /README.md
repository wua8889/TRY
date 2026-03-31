
n8n 自动化项目：库存监控与补货处理
项目简介
本项目基于 n8n（自托管）设计并实现两个完全离线运行的自动化工作流，模拟企业级数据同步与异常监控场景。由于环境无法访问外网，所有操作均基于本地文件（CSV）和 n8n 内置节点完成，体现了流程自动化、数据解析、条件判断、异常记录等核心能力。

工作流一：库存监控
目标：定时检查库存表，当商品库存低于阈值时，自动记录告警日志。

节点构成：

Schedule Trigger：每 5 分钟触发一次（可配置）

Read/Write File from Disk：读取 inventory.csv 库存文件

Extract from File：将二进制数据转换为文本

Code：解析 CSV 为 JSON 数组

IF：判断 库存数量 < 10

Code：构造告警日志文本（时间、商品ID、名称、库存）

Read/Write File from Disk：将日志追加写入 alert_log.csv

效果展示
![Coze 工作流截图](库存监控流程图.png)

运行前后对比
运行前库存表：


商品ID,商品名称,库存数量
P001,笔记本,5
P002,钢笔,12
P003,橡皮,3
运行后告警日志：


2026-03-30T10:00:00.000Z,P001,笔记本,5
2026-03-30T10:00:00.000Z,P003,橡皮,3
工作流二：补货处理
目标：读取补货请求文件，提取商品名称和数量，自动更新库存表。

节点构成：

Schedule Trigger：每 5 分钟触发一次

Read/Write File from Disk ×2：分别读取库存表和补货请求文件

Extract from File ×2：将二进制数据转换为文本

Merge：合并两个文件内容为两个独立 item

Code：解析库存 CSV，用正则表达式从补货请求中提取商品名和数量，更新库存映射，生成新 CSV

Code：将新 CSV 转换为二进制格式

Read/Write File from Disk：写回 inventory.csv

效果展示
![Coze 工作流截图](补货处理.png)
图：补货处理工作流节点连线图


技术栈
n8n：工作流编排引擎（v2.12.3）

Docker：容器化部署 n8n 及文件存储

JavaScript：自定义逻辑（CSV 解析、正则提取）

CSV 文件：作为数据存储和日志记录

Git：版本控制与项目托管

如何使用
安装 Docker 并拉取 n8n 镜像：docker pull n8nio/n8n

启动容器并挂载数据目录（参考项目中的启动命令）

将 inventory.csv 和 replenish_requests.txt 放入挂载目录

在 n8n 界面导入两个工作流的 JSON 文件

根据需要调整定时触发间隔

手动触发或等待定时执行，查看文件变化


后续扩展
若环境允许，可将告警和日志接入企业微信、Slack 或邮件

可用 Claude API 替代正则表达式，实现更智能的文本解析

可增加 Webhook 节点，对外提供 API 接口
