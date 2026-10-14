# 好物清单 · 双十一全球购

移动端优先的亲友拼单网页：朋友查看商品、选择数量、提交清单；组织者统一核对、收集订单、管理费用并生成可复制的微信账单。

> 本项目不是山姆会员商店官方产品，不会自动登录或绕过限制抓取山姆 App。演示商品、图片和价格均为界面样例。正式使用时，请以有权访问的商品信息为准，人工录入或通过 CSV 导入。实际售价、库存、规格、购买资格和售后以山姆 App 下单页面为准。

## 功能

- 手机与桌面自适应商品广告页，支持搜索、分类筛选、图片、价格和商品介绍。
- 朋友选择商品和数量，填写称呼、备注后提交购物清单。
- 组织者后台管理商品，支持手动录入、图片上传、CSV 批量导入。
- 集中查看订单、调整附加费用/优惠、流转状态、生成账单文字、导出 CSV。
- 未配置后端时为本机演示模式；接入 Supabase 后商品和订单可跨设备共享。
- 不处理微信支付、不代替实际下单；账单由组织者核对实际购买结果后手动发送。

## 当前仓库的组织方式

为通过 GitHub 连接自动提交完整网页包，原始静态站点 ZIP 被切分为 `payload/00.b64` 至 `payload/14.b64`。请不要手动编辑这些编码片段。GitHub Actions 会把片段拼回 ZIP，验证后解压为网页文件并发布到 Pages。

应用原始文件包括 `index.html`、`app.js`、`styles.css`、`config.js`、`database.sql` 和 `sam-products-template.csv`。需要更新应用时，应先修改源文件、重新打包，并同步更新 payload 片段；不要只更改编码片段中的字符。

## 启用 GitHub Pages

1. 打开本仓库的 [Settings → Pages](https://github.com/chenzhe223-tech/sam-global-buy/settings/pages)。
2. 在 **Build and deployment** 下把 **Source** 设为 **GitHub Actions**。
3. 打开 [Actions 工作流](https://github.com/chenzhe223-tech/sam-global-buy/actions)，确认最新的 `Deploy static site to GitHub Pages` 运行成功。
4. 成功后访问：**https://chenzhe223-tech.github.io/sam-global-buy/**

如果 Actions 在 `Configure Pages` 步骤提示 Pages site not found，通常说明还没在 Settings → Pages 启用 GitHub Actions 来源。配置好以后，再向 `main` 提交一次更改即可触发部署。

## 配置 Supabase 云端共享

GitHub Pages 只负责托管静态网页，不能保存跨设备共享订单。要正式收单，还需要：

1. 在 Supabase 创建项目。
2. 在 SQL Editor 执行 `database.sql`。
3. 关闭公开注册，在 Authentication → Users 中创建组织者管理员账号。
4. 获取 Project URL 与 anon / publishable key，填入 `config.js` 的 `supabaseUrl` 和 `supabaseAnonKey`。
5. 再发布并测试：朋友无需登录即可提交；未登录者不能查看订单；管理员登录后能管理商品与订单；用两台设备提交测试单，确认后台都能看到。

**不要**把 `service_role` 或 secret key 放到前端。数据安全依赖数据库 RLS 策略。正式公开使用前应做权限测试，并考虑匿名提交的验证码/限流。

## 批量导入商品

组织者后台 → **商品管理** → **下载模板**，或编辑 `sam-products-template.csv`。支持以下 CSV 列：

`name,price,image_url,category,brand,tag,product_url,description`

`image_url` 填可直接访问的图片 URL。仅能在 App 内查看的图片，可在你有权使用的前提下保存后从后台上传。图片上传会压缩后写入商品记录，适合小规模亲友拼单。

## 重要说明

- 当前没有自动抓取山姆 App 商品数据。App 信息可能要求登录或通过动态接口提供；本项目不绕过登录、验证码或访问限制。
- 商品价格是选购时参考值，订单里的金额只用于预估与对账，必须以实际下单记录为准。
- 本工具不是山姆官方页面，不收集微信支付密码、不处理支付 API，也不会自动发送微信消息。
- 公开链接可能被转发，只向信任的亲友分享。不要填写身份证、银行卡号等完成代购不需要的敏感信息。


## Deployment check

After enabling GitHub Pages with **GitHub Actions** as the source, a new push to `main` triggers deployment automatically. See the latest run at [Actions](https://github.com/chenzhe223-tech/sam-global-buy/actions).


## 从山姆 App 导入真实商品（新）

已新增本地浏览器数据转换页：  
**https://chenzhe223-tech.github.io/sam-global-buy/sam-app-import.html**

此页面可解析你通过正常 App 使用过程合法导出的商品列表 JSON，或带有接口响应正文的 HAR，提取商品名称、图片、价格、ID、可能存在的商品链接，生成选品后台支持的 CSV。它只在浏览器本地处理你选中的文件，不上传 HAR 或登录态。

步骤：
1. 从山姆 App 全球购列表获取你已获授权访问的商品数据响应，并保存成 JSON/HAR。不要分享包含 Cookie、auth-token、手机号、地址或其他个人信息的原始 HAR。
2. 打开导入页，选择文件或粘贴 **响应正文 JSON**，点击解析。
3. 核对商品名称、图片、价格和商品 ID，下载 CSV。
4. 在好物清单后台 → 商品管理 → 导入 CSV。

这一步是“真实数据导入”，不是持续的实时 API 同步。当前未把山姆 App 登录凭据写入网页、GitHub 或 Supabase，也没有使用未知的官方授权接口。若要自动定时刷新，需先获得可持续合法使用的数据接口/数据授权，并在私有后端安全存放凭据。实际售价、库存和规格请在山姆 App 最终确认。
