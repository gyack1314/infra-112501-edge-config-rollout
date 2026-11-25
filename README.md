# 跨版本边缘配置发布及不可逆迁移协调

站点发版时使用设备能力上报、配置修订和代理应用回执。所有材料是合成数据。配置的项目字段以站点运维手册中的命名写入记录，版本编号仅表示身份。

## 事件格式

每个 JSONL 文件是一段独立演练。每行含整数逻辑秒点 timestamp、事件名称 type 和对象 data。同一秒的行按文件顺序排列。时间从零起算。文件里的业务标识只用于建立输入之间的引用。所有场景均可在本地用合成角色重新运行。

- **base**　共享配置修订，values 保存键值。后续基线更新不会修改已经引用的编号。
- **override**　站点覆盖修订。set 的值写入站点配置。inherit 解除该键的覆盖，clear 表示该键在站点配置中明确不可用。
- **agent**　代理能力记录。kind 为 collector 或 relay。formats 是代理能服务的队列格式，relay_versions 为采集端支持的中转版本。installed 为上报时真实使用的配置身份，queue 是设备已形成的队列格式。min_online 为事先约定的同站采集可用位数。
- **release**　运维选择 base 和站点覆盖，提交目标版本、站点波次，给出每类代理的目标格式与可使用的中转版本。waves 中内层数组为可并行的代理组，前一波确认可用后才能放行下一波。
- **claim**　指定代理请求在站点领取某次发布的配置并据此改动现场。重复领取使用同一请求号，ack 引用它。
- **ack**　应用回执，outcome 为 applied、failed 或 unknown。记录实际配置与队列格式，只有已知实际值时携带 installed 和 queue。applied 的中转代理不携带 queue。
- **pause**　运维暂停某个 release。
- **recover**　指定发布尝试撤回尚未迁移的配置或应用后续兼容配置。mode 为 revert 或 forward，target 为目标版本。
- **inspect**　查询 release 或 site 的领取资格及实际进展。

## 资料文件

- pause-during-migration.jsonl
- overlapping-releases.jsonl

## 本地资料检查

需 Python 3.11。资料工具使用标准库，无需联网。下列命令读取事件文件并检查封装格式。

```sh
python3 trace_io.py data
python3 -m unittest -v
```

资料工具输出读取的文件、事件数量及首尾秒点。业务状态由使用该事件契约的服务维护。
