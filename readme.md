# 简单图床 · Picora 插件

![插件图标](assets/icon.svg)

为 [Picora](https://github.com/Alnitak44/Picora) 添加 [EasyImages2.0 简单图床](https://github.com/icret/EasyImages2.0) 支持的原生插件。\
用站点后台生成的 **API Token** 上传，返回的链接、缩略图和删除链接随上传记录保存，可在图片详情里删除云端图片。

## 功能

- 一个插件类型可建多组配置（不同站点、不同 Token 互不干扰）
- 自动解析 `url` / `thumb` / `del`，缩略图与删除链接一并保存
- 支持单张图片云端删除（复用站点返回的一次性删除链接）
- 业务失败按站点 `code` / `message` 返回可读错误，不误判成功
- 网络权限绑定用户填写的图床地址，不使用通配主机

## 安装

1. 在 Picora「插件中心 → 模块仓库」安装「简单图床」；
2. 或下载 Release 里的 `easyimages2-<版本>.picora-plugin.zip`，在「安装插件」中导入；
3. 打开「仓库 → 添加配置」，选择「简单图床」，填写图床地址与 API Token。

## 站点准备

1. **开启 API 上传**：后台打开「API 上传」开关（`apiStatus`），关闭时接口返回 `API Closed`。
2. **生成 API Token**：后台 API Token 区域添加，或写入 `config/api_key.php` 的 `$tokenList`。Token 只允许字母和数字，注意有效期。
3. **关闭分片上传**：后台「分片上传」设为 0。API 走单次 multipart，分片模式下接口会输出非 JSON 的 `Invalid input`。
4. 后台的扩展名白名单、最大尺寸、水印、格式转换等设置照常生效。

## 配置项

| 配置项 | 说明 |
| :--- | :--- |
| 图床地址 | 图床访问地址，例如 `https://img.example.com`；装在子目录填 `https://example.com/pic`。不要带 `/api/index.php`，不要带查询参数或 `#` |
| API Token | 后台创建的上传 Token，只含字母数字，不是后台登录密码 |

## 上传流程

- 请求：`POST {图床地址}/api/index.php`，multipart，文件字段 `image`，附加字段 `token`
- 判定：HTTP 200 且响应 `result` 为 `success`（简单图床业务失败也返回 200，只看状态码会误判）
- 记录：`url` 图片链接、`thumb` 缩略图、`del` 删除链接
- 相对链接会按上传接口地址补全

## 云端删除

1. 在「插件中心 → 简单图床 → 插件详情」开启**同步删除云端图片**（安装或更新后默认关闭）。
2. 之后在图片详情点「删除云端图片」，或从相册删除记录。
3. 只能删除同一配置组上传、且配置未修改过的图片。

限制：站点需 2.8.0 起（删除接口返回 JSON）；后台「用户删除链接」（`show_user_hash_del`）需开启；删除链接域名必须与图床地址同源。

## 常见问题

| 现象 | 处理 |
| :--- | :--- |
| 404 / 返回 HTML | 地址少了子目录，或多了 `/api/index.php`；浏览器直接访问 `https://地址/api/index.php` 确认 |
| Token Error (202) | Token 填错、被删除或含非字母数字字符 |
| Token Expired (203) | Token 过期，重新生成 |
| API Closed (201) | 后台未开启「API 上传」 |
| 205 | 命中 IP 名单、扩展名白名单或 MD5 黑名单 |
| 413 | 超过 Nginx `client_max_body_size` 或 PHP `upload_max_filesize` |
| 删除提示「插件生成了无效的请求地址」 | 站点关闭了用户删除链接，或记录来自老版本站点 |

更多细节见「设置 → 诊断日志」，日志不会写入 Token 与响应正文。

## 权限与安全

- 网络权限绑定「图床地址」配置项的协议、主机与端口，不使用通配主机
- 允许明文 HTTP，便于内网 / 测试环境；公网建议 HTTPS
- 仅读取用户主动选择的上传文件，不枚举本地文件、不执行脚本
- 云端删除需用户单独授权，只能删除本插件、本配置组上传的单张图片
- Token 属于上传凭据，请使用仅上传用途的 Token，不要填后台管理员密码

## 开发

```text
.
├── plugin.json       插件元信息
├── uploader.json     程序入口：配置字段、权限、上传与删除请求
├── readme.md         本文档（同时打进插件包）
├── LICENSE
├── assets/icon.svg   图标
├── tools/pack.ps1    打包脚本
└── docs/开发说明.md   接口细节、设计取舍与发布流程
```

打包：

```powershell
powershell -NoProfile -ExecutionPolicy Bypass -File .\tools\pack.ps1 -Force
```

输出 `releases/easyimages2-<版本>.picora-plugin.zip` 并打印 SHA256。插件协议见 [Picora 插件开发文档](https://github.com/Alnitak44/Picora/blob/main/docs/PLUGINS.md)。

## 致谢与许可

- 图床程序：[icret/EasyImages2.0](https://github.com/icret/EasyImages2.0)（MIT）
- 接口文档：[docs/API.md](https://github.com/icret/EasyImages2.0/blob/master/docs/API.md)

本仓库代码以 MIT 许可发布，详见 [LICENSE](LICENSE)。

> 非官方插件，与 EasyImages2.0 项目无隶属关系，仅调用其公开 API。

