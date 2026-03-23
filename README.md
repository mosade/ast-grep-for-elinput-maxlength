# ast-grep：检查并补全 `el-input` 的 `maxlength`

使用 `ast-grep` 静态分析 Vue 模板，找出缺少 `maxlength` 属性的 `el-input` / `ElInput` 组件，并自动补上 `maxlength="20"`。

## 文件结构

```
sgconfig.yml                                     # ast-grep 项目配置
utils/
  has-maxlength-attr.yml                         # 工具规则：带修饰符的 maxlength 属性
  has-maxlength-in-bind-object.yml               # 工具规则：v-bind 对象展开中含 maxlength
rules/
  add-missing-maxlength.yml                      # el-input 自闭合
  add-missing-maxlength-paired.yml               # el-input 成对标签
  add-missing-maxlength-pascal.yml               # ElInput 自闭合
  add-missing-maxlength-pascal-paired.yml        # ElInput 成对标签
rule-tests/
  add-missing-maxlength-test.yml
  add-missing-maxlength-paired-test.yml
  add-missing-maxlength-pascal-test.yml
  add-missing-maxlength-pascal-paired-test.yml
app.vue                                          # 示例文件
```

## 快速上手

```bash
# 验证规则
npx -p @ast-grep/cli sg test --skip-snapshot-tests

# 扫描项目，查看命中
npx -p @ast-grep/cli sg scan .

# 自动修复
npx -p @ast-grep/cli sg scan -U .

# 只处理某个文件
npx -p @ast-grep/cli sg scan -U app.vue
```

## 配置说明

`sgconfig.yml` 将 `.vue` 文件映射为 `html` 语言，让 ast-grep 能正确解析 Vue 模板：

```yaml
ruleDirs:
  - rules
testConfigs:
  - testDir: rule-tests
languageGlobs:
  html:
    - '*.vue'
```

## 规则设计

### 主规则（以 `add-missing-maxlength.yml` 为例）

```yaml
id: add-missing-maxlength
language: html
message: el-input is missing maxlength
severity: warning
rule:
  all:
    - pattern: <el-input $$$ATTRS />
    - not:
        matches: has-maxlength-attr
    - not:
        matches: has-maxlength-in-bind-object
    - not:
        any:
          - pattern: <el-input $$$BEFORE maxlength $$$AFTER />
          - pattern: <el-input $$$BEFORE maxlength=$VALUE $$$AFTER />
          - pattern: <el-input $$$BEFORE :maxlength $$$AFTER />
          - pattern: <el-input $$$BEFORE :maxlength=$VALUE $$$AFTER />
          - pattern: <el-input $$$BEFORE v-bind:maxlength $$$AFTER />
          - pattern: <el-input $$$BEFORE v-bind:maxlength=$VALUE $$$AFTER />
fix: <el-input $$$ATTRS maxlength="20" />
```

四条主规则结构相同，分别对应：`el-input` 自闭合、`el-input` 成对、`ElInput` 自闭合、`ElInput` 成对。

### 工具规则：`has-maxlength-attr`

处理带修饰符的属性写法（`:maxlength.prop`、`v-bind:maxlength.camel` 等），用 regex 匹配：

```yaml
id: has-maxlength-attr
language: html
rule:
  regex: '<(?:el-input|ElInput)\b[^>]*\s(?::maxlength|v-bind:maxlength)\.[^\s=/>]+(?:\s*=\s*(?:"[^"]*"|''[^'']*''|[^\s>]+))?[^>]*>'
```

### 工具规则：`has-maxlength-in-bind-object`

处理 `v-bind="{ ... }"` 对象展开写法，通过 pattern 捕获 `$OBJ`，再用 regex 约束其内容：

```yaml
id: has-maxlength-in-bind-object
language: html
rule:
  any:
    - pattern: <el-input $$$BEFORE v-bind=$OBJ $$$AFTER />
    - pattern: <el-input $$$BEFORE v-bind=$OBJ $$$AFTER>$$$CHILDREN</el-input>
    - pattern: <ElInput $$$BEFORE v-bind=$OBJ $$$AFTER />
    - pattern: <ElInput $$$BEFORE v-bind=$OBJ $$$AFTER>$$$CHILDREN</ElInput>
constraints:
  OBJ:
    regex: "^['\"]\\{[\\s\\S]*(?:(?:['\"](?:maxlength|maxLength)['\"])|\\bmaxlength\\b|\\bmaxLength\\b)(?:\\s*:|[\\s,}])[\\s\\S]*\\}['\"]$"
```

## 已覆盖的排除场景

| 写法 | 说明 |
|---|---|
| `maxlength="20"` | 普通属性 |
| `maxlength` | 无值布尔属性 |
| `:maxlength="..."` | 动态绑定 |
| `:maxlength` | 动态绑定 shorthand |
| `v-bind:maxlength="..."` | 长写法动态绑定 |
| `v-bind:maxlength` | 长写法 shorthand |
| `:maxlength.prop="..."` | 带修饰符 |
| `v-bind:maxlength.camel="..."` | 带修饰符 |
| `v-bind="{ maxlength: 10 }"` | 对象展开 |
| `v-bind="{ maxlength }"` | 对象展开属性简写 |
| `v-bind="{ 'maxlength': 10 }"` | 对象展开，单引号 key |
| `v-bind='{ "maxlength": 10 }'` | 对象展开，双引号 key，单引号外层 |
| `v-bind='{ maxlength: 10 }'` | 对象展开，单引号外层 |
| `v-bind="{ maxLength: 10 }"` | 对象展开，驼峰 key |
| `v-bind="{ maxLength }"` | 对象展开，驼峰 shorthand |
| `v-bind='{ "maxLength": 10 }'` | 对象展开，驼峰双引号 key |
| `v-bind="{ 'maxLength': 10 }"` | 对象展开，驼峰单引号 key |
| `v-bind='{ maxLength: 10 }'` | 对象展开，驼峰单引号外层 |

以下写法**仍会被命中**（视为缺少 maxlength）：

- `v-bind="{ foo: 1 }"` — 对象中无 maxlength 相关 key
- `v-bind="attrs"` — 变量展开，无法静态判断内容

## 已知限制

`v-bind="attrs"` 这类变量展开写法，规则无法静态判断其中是否包含 `maxlength`，可能产生误报。这是 AST 静态分析的固有限制，需人工确认。
