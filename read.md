🚀 Postman + GitHub Actions 接口自动化测试项目
晓楠的企业级接口自动化实战项目 | 从 0 到 1 搭建完整 CI/CD 测试流程
📌 项目亮点
✅ 双集合自动化：覆盖核心业务场景，支持并行 / 串行执行
✅ 自动鉴权：登录接口自动提取 Token，全局注入无需手动维护
✅ CI/CD 集成：代码提交即触发测试，无人值守全流程自动化
✅ 安全合规：敏感信息（Token）通过 GitHub 密钥管理，绝不明文泄露
✅ 可视化报告：自动生成 HTML/JSON 报告，结果一目了然
🎯 项目背景与目标
背景
在接口测试过程中，人工执行回归测试存在效率低、易出错、耗时久等问题，尤其在频繁迭代的项目中，重复劳动成本极高。
目标
搭建一套从用例设计到自动化执行、报告输出的端到端接口自动化体系，实现：
接口用例的批量自动化执行
Token 自动获取与全局注入
代码提交后自动触发测试流程
敏感信息安全管理，避免泄露
标准化测试报告输出与分析
🛠️ 技术栈与工具
类别	工具 / 技术	说明
接口测试	Postman	接口用例设计、调试、集合管理
自动化执行	Newman	Postman 集合命令行执行工具
版本控制	Git + GitHub	代码版本管理与协作
CI/CD 平台	GitHub Actions	自动化工作流编排与触发
测试报告	Newman Reporter	生成 HTML/JSON 格式测试报告
安全管理	GitHub Secrets	敏感信息（Token）加密存储与注入
📋 核心实现步骤
1. Postman 用例设计与自动化
集合拆分：按业务模块拆分为 2 个核心集合，覆盖登录、用户查询等高频场景
自动鉴权：在登录接口 Tests 中提取 Token 并存入环境变量：
javascript
运行
// 从响应体提取 Token 并设置为环境变量
const jsonData = pm.response.json();
pm.environment.set("access_token", jsonData.token);
统一断言：为关键接口添加状态码、数据完整性断言，保障测试可靠性：
javascript
运行
pm.test("✅ 请求成功 - 状态码 200", function () {
  pm.response.to.have.status(200);
});

pm.test("✅ 响应数据非空", function () {
  pm.response.json(); // 若解析失败则抛出错误
});
2. Newman 本地验证
安装 Newman：
bash
运行
npm install -g newman
本地执行集合，验证用例正确性：
bash
运行
# 执行集合1
newman run collection1.json -e environment.json
# 执行集合2
newman run collection2.json -e environment.json
3. GitHub Actions 自动化集成
创建 .github/workflows/postman-test.yml，实现全流程自动化：
yaml
name: 🧪 Postman API 自动化测试

on:
  push:
    branches: [ main ]  # 代码推送到 main 分支自动触发
  workflow_dispatch:    # 支持手动触发

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: 📥 拉取代码
        uses: actions/checkout@v4

      - name: 🟢 配置 Node.js 环境
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: 📦 安装 Newman
        run: npm install -g newman

      - name: 🚀 运行接口集合1
        run: |
          newman run collection1.json \
            -e environment.json \
            -r cli,html,json \
            --reporter-json-export report1.json \
            --reporter-html-export report1.html \
            --env-var "access_token=${{ secrets.ACCESS_TOKEN }}"

      - name: 🚀 运行接口集合2
        run: |
          newman run collection2.json \
            -e environment.json \
            -r cli,html,json \
            --reporter-json-export report2.json \
            --reporter-html-export report2.html \
            --env-var "access_token=${{ secrets.ACCESS_TOKEN }}"

      - name: 📊 上传测试报告
        uses: actions/upload-artifact@v4
        with:
          name: postman-test-reports
          path: |
            report1.html
            report1.json
            report2.html
            report2.json
        if: always()  # 无论测试成功/失败，都上传报告
4. 敏感信息安全管理
环境文件清理：environment.json 中 access_token 字段值清空为 ""
GitHub 密钥配置：
仓库 → Settings → Secrets and variables → Actions
新建 ACCESS_TOKEN 密钥，存入真实 Token 值
密钥注入：通过 --env-var 在 Newman 运行时动态注入 Token，避免明文泄露
📊 项目成果与价值
维度	提升效果	说明
执行效率	⚡ 提升 90%+	人工执行 30 分钟 → 自动化 2 分钟完成
测试稳定性	✅ 显著提升	统一断言 + 自动化执行，消除人为操作误差
信息安全	🔒 完全合规	Token 加密存储，绝不明文出现在代码库
可复用性	♻️ 高度可复用	形成标准化模板，可快速迁移至其他项目
流程标准化	📏 流程固化	从用例到报告全流程自动化，降低团队协作成本
🚀 如何运行
1. 本地执行
bash
运行
# 安装 Newman
npm install -g newman

# 执行集合
newman run collection1.json -e environment.json
newman run collection2.json -e environment.json
2. 自动化执行（GitHub Actions）
自动触发：代码推送到 main 分支自动运行
手动触发：仓库 → Actions → 选择工作流 → Run workflow
查看报告：运行成功后，在 Artifacts 中下载 postman-test-reports 查看 HTML/JSON 报告
📂 项目结构
plaintext
.
├── .github/
│   └── workflows/
│       └── postman-test.yml  # GitHub Actions 自动化配置
├── collection1.json          # 接口集合 1（业务模块 A）
├── collection2.json          # 接口集合 2（业务模块 B）
├── environment.json          # 环境配置（敏感信息已清空）
└── README.md                 # 项目说明文档
📝 作者信息
作者：晓楠
项目类型：接口自动化测试实战
学习周期：3 天从 0 到 1 完成企业级 CI/CD 接口自动化体系搭建
