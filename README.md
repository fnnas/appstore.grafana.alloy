# appstore.grafana.alloy

## 简介

[Grafana Alloy](https://grafana.com/docs/alloy/latest/) 是 Grafana 官方基于 OpenTelemetry 的可观测性数据采集器，支持 Prometheus、Loki、Tempo、Pyroscope 等后端。  
本项目将其打包为飞牛 NAS 应用商店格式，可在飞牛 NAS 上一键安装。

## 目录结构

```
.
├── .github/workflows/
│   ├── main.yml              # 主编排工作流（触发构建 + 发布）
│   ├── pack_amd64.yml        # amd64 打包工作流
│   └── pack_arm64.yml        # arm64 打包工作流
├── app/
│   ├── config.alloy.conf.template  # Alloy 默认配置模板
│   └── ui/                   # UI 资源目录
├── package/                  # 应用包预制菜
│   ├── manifest              # 应用包元数据
│   ├── ICON.PNG              # 应用图标
│   ├── cmd/                  # 生命周期管理脚本
│   │   ├── main              # 服务启动/停止/状态
│   │   ├── install_init      # 安装前钩子
│   │   ├── install_callback  # 安装后钩子
│   │   ├── uninstall_init    # 卸载前钩子
│   │   ├── uninstall_callback# 卸载后钩子
│   │   ├── upgrade_init      # 升级前钩子
│   │   └── upgrade_callback  # 升级后钩子
│   └── config/
│       ├── resource          # 共享存储及 systemd 配置
│       └── privilege         # 权限配置（以 root 运行）
└── README.md                 # AI润色的readme
```

## 配置说明

安装后，配置文件位于共享目录 `/var/apps/grafana.loki/shares/grafana.alloy/` 下：

- `config.alloy.conf`：Alloy 主配置文件（首次启动时由模板自动生成）

默认配置提供以下功能：

- 自动发现并收集 Docker 容器日志
- 收集系统及应用日志文件（`/var/log/**/*.log`、`/var/apps/**/*.log`、`/usr/trim/**/*.log` 等）
- 推送至本地 Loki（`http://127.0.0.1:3100`），你要是想推到远端记得自己改


修改配置后，重启应用即可生效。

**日志路径**：`/tmp/grafana.log/grafana.alloy.log`

## 相关链接

- [Grafana Alloy 官方文档](https://grafana.com/docs/alloy/latest/)
- [Grafana Alloy GitHub](https://github.com/grafana/alloy)
- [飞牛 NAS](https://www.fnnas.com/)