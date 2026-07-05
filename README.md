# youtube2bilibili（依赖升级重置版）

本项目已完成一次大版本重置，核心变化：

- 从旧 `biliup-rs` 迁移到 [biliup/biliup](https://github.com/biliup/biliup) 的 **`biliupR`**
- `yt-dlp` 改为 Python API（`ydl_opts`）
- 全部配置改为外部 `config.yaml`

## 快速启动

推荐使用 `uv` 和 Python 3.11+：

```bash
uv venv --python 3.11
cp config.example.yaml config.yaml
uv run python install.py --config config.yaml
uv run python upload.py --config config.yaml
```

系统还需要安装 `ffmpeg`，否则高画质下载时音视频无法合并：

```bash
brew install ffmpeg
```

说明：

- 如果 `config.yaml` 不存在，脚本会自动从 `config.example.yaml` 生成并继续引导。
- 你可以直接跟着引导式流程运行，或提前手动修改 `config.yaml`。

## 启动流程（固定顺序）

每次 `python upload.py` 都会按以下顺序执行：

1. 验证 YouTube 连通性  
   - 优先使用配置代理  
   - 失败时支持终端输入代理（如 `http://127.0.0.1:7890`）  
   - 验证成功后会自动回写到 `config.yaml`
2. 验证 B 站登录状态  
   - 使用 `biliup renew` 校验  
   - 未登录自动执行 `biliup login`（可扫码）
   - 所有 `biliup` 命令不传 `-p` 代理参数
3. 检查并更新依赖  
   - `yt-dlp` / `deno`（PyPI）  
   - `biliupR`（GitHub Release）
4. 检查本地命令依赖
   - 当前要求已安装 `ffmpeg`
5. 进入运行模式

## 四种模式

1. 单视频上传  
输入一个 URL，下载并投稿。

2. 列表/频道批量上传  
输入播放列表或频道 URL，解析后批量上传，状态写入 `url_list.json`。

3. 断点续传  
继续上传 `url_list.json` 里 `status=no` 的条目。

4. 手动多链接  
逐条输入 URL（输入“完毕”结束），然后批量上传。

批量模式结束后会输出成功/失败统计与失败 URL 列表。

## 配置文件

配置模板：`config.example.yaml`  
实际配置：`config.yaml`

常用配置项：

- 代理：`network.proxy`、`youtube.proxy`
- 启动行为：`startup.ask_proxy_on_youtube_check_fail`、`startup.auto_update_python_deps`
- YouTube 鉴权：
  - `youtube.cookies.enabled + youtube.cookies.file`
  - `youtube.cookies_from_browser.enabled + browser/profile/...`
- yt-dlp JS 运行时：
  - 默认仅 `youtube.js_runtime.remote_components: ["ejs:github"]`
  - `youtube.js_runtime.js_runtimes` 为可选项（按需再开）
- 视频画质：
  - 在 `youtube.ydl_opts.format` 中设置，例如 `bestvideo+bestaudio/best`
  - 限制 1080p 可用 `bestvideo[height<=1080]+bestaudio/best[height<=1080]`
- 投稿默认项：`biliup_studio_defaults`
- 标题正则规则：`upload.title_rules.regex_replace`

## 文件说明

- `upload.py`：主脚本
- `install.py`：安装 Python 依赖并初始化/更新 `biliupR`
- `biliupr_installer.py`：`biliupR` 自动下载与更新模块
- `config.example.yaml`：配置模板

## 常用命令

仅更新 `biliupR`（跳过 pip）：

```bash
uv run python install.py --config config.yaml --skip-pip
```

强制重装最新 `biliupR`：

```bash
uv run python install.py --config config.yaml --force-biliupr
```
