# 功能一览

- 应答机制
    - HTTP 回调
    - Stream 回调
    - OutGoing..? (过时，不再专门支持)
- 使用方式
    - 脚本运行
        - 机器人阻塞模式
        - 单 API 模式 (需要额外配置)
    - 控制台模式
- 群聊功能
    - 发送消息
        - 文字
        - Markdown
        - ActionCard
        - FeedCard
    - 发送文件
        - 普通文件
        - 图片
        - 音频
      - 视频
    - 撤回消息 (钉钉限制仅通过API发送的文件才可以撤回)
    - 发送互动卡片 (自行构造JSON数据)
        - 预设 Markdown 卡片
    - 改变卡片内容 (自行构造JSON数据)
    - 发送 AI 卡片，支持流式更新 (需要配置)
    - 创建群
    - 获取群消息
    - 获取部门消息
    - 获取用户详细信息
    - 删除用户 (组织)
    - 创建用户 (组织)
    - 更新用户信息 (组织)
    - 复制群 **(未来可能移除)**
    - 更新群信息
        - 同源功能
            - 更新群标题
            - 改变群主
            - 全体禁言/解禁
    - 解散群 (伪)
    - 踢出用户 (群)
    - 添加用户 (群)
    - 设置/取消管理员 (群)
    - 禁言用户/解除禁言用户 (群)
    - 设置机器人离线卡片 (Stream 独占)
    - 手动上传文件
    - 下载机器人聊天文件
- 特别功能
    - HTTP & Stream 同时运行 (不建议，可能会导致需要API请求的功能无法正常使用)
    - 运行周期内上传文件缓存
    - 即时重载 Debug 模式
    - **临时**切换全局**默认** `AccessToken`
    - 用户/群组信息缓存
    - ~~即时载入/卸载模块~~ (有Bug，仅可载入，无法卸载)