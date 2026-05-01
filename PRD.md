# PRD-001 汇率换算工具 | exchange-rate

## 背景

用户需要一个汇率换算网页工具，支持输入金额后实时展示多个币种的换算结果。

## 原始需求

给我实现一个汇率换算的功能，对接实时汇率，能够按输入的金额自动换算出其他币种汇率

## 功能需求

### 核心功能

<lark-table rows="5" cols="4" header-row="true" column-widths="183,183,183,183">

  <lark-tr>
    <lark-td>
      功能ID
    </lark-td>
    <lark-td>
      功能描述
    </lark-td>
    <lark-td>
      优先级
    </lark-td>
    <lark-td>
      备注
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      FUNC-01
    </lark-td>
    <lark-td>
      **汇率 API 对接**
    </lark-td>
    <lark-td>
      必须
    </lark-td>
    <lark-td>
      使用 frankfurter.app（免费、无 key、支持 30+ 币种）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      FUNC-02
    </lark-td>
    <lark-td>
      金额输入
    </lark-td>
    <lark-td>
      必须
    </lark-td>
    <lark-td>
      用户输入待换算金额
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      FUNC-03
    </lark-td>
    <lark-td>
      币种选择
    </lark-td>
    <lark-td>
      必须
    </lark-td>
    <lark-td>
      支持 8–10 个主流币种，默认基准 USD
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      FUNC-04
    </lark-td>
    <lark-td>
      换算结果展示
    </lark-td>
    <lark-td>
      必须
    </lark-td>
    <lark-td>
      实时换算（防抖 300ms）
    </lark-td>
  </lark-tr>
</lark-table>

### 边界情况处理

<lark-table rows="8" cols="3" header-row="true" column-widths="200,200,300">

  <lark-tr>
    <lark-td>
      场景
    </lark-td>
    <lark-td>
      处理方式
    </lark-td>
    <lark-td>
      提示文案
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      空输入
    </lark-td>
    <lark-td>
      不触发请求，显示占位
    </lark-td>
    <lark-td>
      "请输入金额"
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      非法字符输入
    </lark-td>
    <lark-td>
      过滤非数字和小数点字符
    </lark-td>
    <lark-td>
      无提示，静默过滤
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      负数 / 零
    </lark-td>
    <lark-td>
      阻止换算
    </lark-td>
    <lark-td>
      "请输入正数"
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      金额超限（> 999,999,999,999）
    </lark-td>
    <lark-td>
      阻止换算
    </lark-td>
    <lark-td>
      "金额过大，请重新输入"
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      网络异常 / 超时
    </lark-td>
    <lark-td>
      显示错误提示，尝试降级
    </lark-td>
    <lark-td>
      "汇率获取失败，请稍后重试"
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      API 不可用（frankfurter.app 503/429）
    </lark-td>
    <lark-td>
      显示缓存汇率 + 时间戳
    </lark-td>
    <lark-td>
      "当前数据来自缓存（更新时间：XX:XX）"
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      API 请求超时
    </lark-td>
    <lark-td>
      超时 5s 中断请求，提示降级
    </lark-td>
    <lark-td>
      "汇率获取超时，请稍后重试"
    </lark-td>
  </lark-tr>
</lark-table>

### UI 交互说明

#### 输入框
- placeholder="请输入金额"
- 辅助提示文字："支持最多2位小数"
- 最大长度：15位整数 + 2位小数
- 实时数字格式（千分位，如 1,234,567.89）

#### 结果展示
- 每个币种一个卡片
- 卡片内容：【币种符号 + 金额 + 币种名称】
- 例：$1,234.56 USD 美元

#### Loading 状态
- 换算区域显示骨架屏或 loading 动画
- 文案："正在获取汇率..."

#### 基准货币切换
- 下拉选择基准货币（默认 USD）
- 切换后自动重新计算所有币种

#### 精度切换
- 可切换显示 2 位小数 / 4 位小数
- 例：¥1,234.5678 CNY 人民币

#### 响应式设计
- 移动端（<768px）：单列布局
- 桌面端（≥768px）：2-3 列网格

#### 市场休市提示
- 周末/假日（非交易日）显示提示
- 文案："非交易日，数据可能延迟"

### 字段说明

#### 币种符号对照表

<lark-table rows="11" cols="3" header-row="true" column-widths="120,120,200">

  <lark-tr>
    <lark-td>
      币种代码
    </lark-td>
    <lark-td>
      符号
    </lark-td>
    <lark-td>
      币种名称
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      USD
    </lark-td>
    <lark-td>
      $
    </lark-td>
    <lark-td>
      美元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      EUR
    </lark-td>
    <lark-td>
      €
    </lark-td>
    <lark-td>
      欧元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      GBP
    </lark-td>
    <lark-td>
      £
    </lark-td>
    <lark-td>
      英镑
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      JPY
    </lark-td>
    <lark-td>
      ¥
    </lark-td>
    <lark-td>
      日元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CNY
    </lark-td>
    <lark-td>
      ¥
    </lark-td>
    <lark-td>
      人民币
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      HKD
    </lark-td>
    <lark-td>
      $
    </lark-td>
    <lark-td>
      港币
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      KRW
    </lark-td>
    <lark-td>
      ₩
    </lark-td>
    <lark-td>
      韩元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      SGD
    </lark-td>
    <lark-td>
      $
    </lark-td>
    <lark-td>
      新加坡元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AUD
    </lark-td>
    <lark-td>
      $
    </lark-td>
    <lark-td>
      澳元
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      CAD
    </lark-td>
    <lark-td>
      $
    </lark-td>
    <lark-td>
      加元
    </lark-td>
  </lark-tr>
</lark-table>

#### 精度规则
- 统一保留 2 位小数（人民币 JPY 保留 3 位）
- 四舍五入处理

#### 数字格式
- 千分位分隔符（如 1,234,567.89）

### 缓存策略
- 汇率数据缓存 10 分钟
- 避免频繁请求 API，降低限速风险
- 缓存命中时直接使用，不发起网络请求

### 无障碍支持
- 键盘完全可导航（Tab / Shift+Tab）
- 输入框和按钮有清晰的 ARIA 标签
- 屏幕阅读器友好（语义化 HTML）
- 足够的颜色对比度

### API 降级方案
- frankfurter.app 不可用时（HTTP 503/429）
- 显示上次缓存的汇率 + 时间戳
- 提示用户数据可能非实时

### 非功能需求

<lark-table rows="6" cols="2" header-row="true" column-widths="200,500">

  <lark-tr>
    <lark-td>
      需求类型
    </lark-td>
    <lark-td>
      说明
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      页面加载速度
    </lark-td>
    <lark-td>
      首次加载 < 2s（纯前端，无后端）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      Loading 状态
    </lark-td>
    <lark-td>
      API 请求期间显示 loading 动画或骨架屏
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      异常提示
    </lark-td>
    <lark-td>
      网络异常、超时、API 不可用时有友好提示
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      缓存
    </lark-td>
    <lark-td>
      汇率缓存 10 分钟，支持降级展示
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      无障碍
    </lark-td>
    <lark-td>
      键盘导航、ARIA 标签、屏幕阅读器支持
    </lark-td>
  </lark-tr>
</lark-table>

## 用户故事

作为用户，我希望输入一个金额后快速看到其他币种的换算结果，以便日常参考实时汇率。

## 验收标准

<lark-table rows="7" cols="2" header-row="true" column-widths="350,350">

  <lark-tr>
    <lark-td>
      验收项
    </lark-td>
    <lark-td>
      通过条件
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-01
    </lark-td>
    <lark-td>
      输入金额后能展示换算结果
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-02
    </lark-td>
    <lark-td>
      汇率数据来源于实时 API
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-03
    </lark-td>
    <lark-td>
      支持主流币种换算（USD/EUR/GBP/JPY/CNY/HKD/KRW/SGD）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-04
    </lark-td>
    <lark-td>
      异常情况有友好提示
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-05
    </lark-td>
    <lark-td>
      空输入/负数/超限金额有明确提示
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      AC-06
    </lark-td>
    <lark-td>
      基准货币可切换，切换后自动重新计算
    </lark-td>
  </lark-tr>
</lark-table>

## 技术方案

<lark-table rows="8" cols="2" header-row="true" column-widths="200,500">

  <lark-tr>
    <lark-td>
      项目
    </lark-td>
    <lark-td>
      方案
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      汇率 API
    </lark-td>
    <lark-td>
      frankfurter.app（免费，无需 key）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      基准货币
    </lark-td>
    <lark-td>
      USD（用户可切换）
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      交互方式
    </lark-td>
    <lark-td>
      实时换算，300ms 防抖
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      技术栈
    </lark-td>
    <lark-td>
      纯前端 HTML/CSS/JS
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      币种
    </lark-td>
    <lark-td>
      USD, EUR, GBP, JPY, CNY, HKD, KRW, SGD, AUD, CAD
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      API 超时
    </lark-td>
    <lark-td>
      5 秒超时
    </lark-td>
  </lark-tr>
  <lark-tr>
    <lark-td>
      缓存策略
    </lark-td>
    <lark-td>
      LocalStorage / 内存缓存 10 分钟
    </lark-td>
  </lark-tr>
</lark-table>

## 项目信息

- **Git 仓库**: D:\ai-dev-git\exchange-rate
- **GitHub Pages**: https://fallenjie.github.io/exchange-rate/
- **创建时间**: 2026-05-01
