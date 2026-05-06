# clash_global_config

一份用于 [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev) 的 **全局覆写脚本（Global Extend Script）**，把任意机场订阅自动加工成一份带分类策略组、按地区自动分组、内置内网直连白名单、并配好 DNS 与域名嗅探的统一配置。

只需导入订阅 + 启用本脚本，即可获得开箱即用的分流体验，无需手动维护 YAML。

## 功能特性

- **自动分类策略组**：Google / Apple / 微软 / Github / Onedrive / AI / YouTube / Netflix / TikTok / Bilibili / Spotify / Adobe / Pornhub / 游戏 / Telegram / 网速测试 / 漏网之鱼。
- **自动按地区分组**：自动识别节点名中的国家/地区关键词（中英文 + 国旗 emoji + ISO 代码），生成 30+ 地区的 `url-test` 自动选优组（美/日/韩/新/港/台/英/法/德/荷/俄/澳/印…）。
- **多种节点选择策略**：延迟选优（url-test）、手动选择、故障转移（fallback）、负载均衡（轮询 / 一致性散列）。
- **内网直连白名单**：可自定义一批域名强制走 `DIRECT`，避免内网/办公服务被代理影响。
- **完善的规则集**：使用 [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) 与 [DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata) 的 `.mrs` 格式规则集，每天自动更新一次。
- **DNS / Sniffer 默认配置**：`fake-ip` 模式 + DoH（阿里 DoH 国内 / 自定义 DoH 国外）+ TLS/HTTP/QUIC 嗅探。
- **同时支持两种订阅形态**：明确节点列表（`proxies`）和订阅提供器（`proxy-providers`）。

## 使用方法（Clash Verge Rev）

1. 打开 Clash Verge Rev → 左侧菜单「**脚本**」（Scripts）。
2. 点击右上角「**新建**」→ 类型选择 **JavaScript**，名称随意（例如 `global_config`）。
3. 把仓库根目录的 [`Script.js`](./Script.js) 内容整段复制粘贴进编辑器，保存。
4. 回到「**订阅**」（Profiles）页面，找到要使用的订阅卡片，右键 → 「**编辑覆写**」（Edit Override） → 在 **Scripts** 一栏勾选刚才创建的脚本。
5. 应用配置即可。

> 也可以把 `Script.js` 放到任意 HTTPS 可访问的位置（例如 GitHub Raw / jsDelivr），在 Clash Verge Rev 的脚本里通过远程链接的形式导入，方便多设备同步。

## 自定义

所有可调项集中在 `Script.js` 文件顶部，按需修改后保存生效。

### 1. 内网/公司直连白名单

修改 `cyberxDirectDomain`（第 1–18 行）。命中其中任一域名（支持通配符）的请求都会强制走 `DIRECT`：

```js
const cyberxDirectDomain = [
    "*.your-company.com",
    "*.your-internal-domain.cc",
    // 在此追加你自己的内网/公司域名
];
```

### 2. 额外自定义代理规则

修改 `customRules`（第 19–25 行），按 Clash 规则语法添加：

```js
const customRules = [
    "DOMAIN-SUFFIX,example.com,节点选择",
    "DOMAIN-KEYWORD,foo,全局直连",
    // ...
];
```

### 3. 测试 URL / 间隔 / 容差

```js
const test_url = "http://www.google.com/generate_204"; // 测速 URL
const test_interval = 240; // 测速间隔（秒）
const test_tolerance = 80; // 切换容差（毫秒），越小切换越频繁
```

### 4. DNS 服务器

```js
const domesticNameservers = [
    'https://223.5.5.5/dns-query', // 阿里云公共 DNS（国内）
];
const foreignNameservers = [
    'https://doh.apad.pro/dns-query', // 国外 DoH（按需替换为 Cloudflare / Google 等）
];
```

> 注意：每项只填一个最快的 DoH 即可，写多了会拖慢解析并增加内核内存占用。

### 5. 地区分组

修改 `regionConfig`（第 256 行起）。`matcher` 是用 `|` 分隔的关键字正则，匹配节点名（中英文 / 国旗 emoji / ISO 代码均可）。新增地区只需追加一项即可。

## 工作原理

Clash Verge Rev 在加载订阅时会调用脚本中的 `main(config)` 函数：

```js
function main(config) {
    // 1. 校验订阅里至少存在 proxies 或 proxy-providers
    // 2. 注入 profile / geodata / 客户端指纹 / DNS / Sniffer
    // 3. 写入 rule-providers 与 rules
    // 4. 写入预定义的 proxy-groups
    // 5. 调用 addRegions(config) 自动生成地区分组
    return config;
}
```

`addRegions` 会同时兼容两种订阅形态：

- 当订阅以 `proxies`（明确节点）形式提供时，按节点名关键字分桶。
- 当订阅以 `proxy-providers`（订阅提供器）形式提供时，对每个地区生成 `url-test + filter` 组。

## 致谢

- [Clash Verge Rev](https://github.com/clash-verge-rev/clash-verge-rev) — GUI 客户端
- [MetaCubeX/mihomo](https://github.com/MetaCubeX/mihomo) — 内核
- [MetaCubeX/meta-rules-dat](https://github.com/MetaCubeX/meta-rules-dat) — 规则集
- [DustinWin/ruleset_geodata](https://github.com/DustinWin/ruleset_geodata) — 规则集
- [Koolson/Qure](https://github.com/Koolson/Qure) — 图标资源

## License

仓库未声明许可证，默认保留所有权利。如需以 MIT / Apache-2.0 等许可证开源，请追加 `LICENSE` 文件。
