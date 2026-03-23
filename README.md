# ast-grep 教程：检查并补全 `el-input` 的 `maxlength`

这份教程演示如何使用 `ast-grep`：

- 找出没有设置 `maxlength` 的 `el-input`
- 自动给这些组件补上 `maxlength="20"`
- 同时支持自闭合和非自闭合两种写法
- 直接在 `.vue` 文件里应用这条规则

## 相关文件

- 项目配置：`sgconfig.yml`
- 工具规则：`utils/has-maxlength-in-bind-object.yml`
- 规则文件：`rules/add-missing-maxlength.yml`
- 规则文件：`rules/add-missing-maxlength-paired.yml`
- 规则文件：`rules/add-missing-maxlength-pascal.yml`
- 规则文件：`rules/add-missing-maxlength-pascal-paired.yml`
- 测试文件：`rule-tests/add-missing-maxlength-test.yml`
- 测试文件：`rule-tests/add-missing-maxlength-paired-test.yml`
- 测试文件：`rule-tests/add-missing-maxlength-pascal-test.yml`
- 测试文件：`rule-tests/add-missing-maxlength-pascal-paired-test.yml`
- 示例文件：`app.vue`

## 1. 安装或直接使用 ast-grep

如果本机没全局安装，可以直接用 `npx`：

```bash
npx -p @ast-grep/cli sg --help
```

如果你想全局安装，也可以自己安装 `@ast-grep/cli`。

## 2. 配置 ast-grep 识别 Vue 文件

因为 `ast-grep` 默认不把 `.vue` 当成 `html` 处理，所以需要在 `sgconfig.yml` 里加映射：

```yaml
ruleDirs:
  - rules
testConfigs:
  - testDir: rule-tests
languageGlobs:
  html:
    - '*.vue'
```

这里最关键的是：

```yaml
languageGlobs:
  html:
    - '*.vue'
```

它的作用是让 `ast-grep` 用 `html` 语法解析 `.vue` 文件中的模板部分。

## 3. 编写规则

自闭合标签规则 `rules/add-missing-maxlength.yml`：

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

这条规则的含义是：

- 先匹配所有自闭合的 `<el-input />`
- 再排除已经有 `maxlength=...` 的情况
- 再排除 `maxlength` 无值属性写法
- 再排除已经有 `:maxlength=...` 的情况
- 再排除 `:maxlength` shorthand 写法
- 再排除 `:maxlength.prop=...` 这类带修饰符写法
- 再排除已经有 `v-bind:maxlength=...` 的情况
- 再排除 `v-bind:maxlength` shorthand 写法
- 再排除 `v-bind:maxlength.camel=...` 这类带修饰符写法
- 再排除 `v-bind="{ maxlength: ... }"` 这类对象展开写法
- 再排除 `v-bind="{ 'maxlength': ... }"` 这类带引号 key 的对象展开写法
- 也排除 `v-bind="{ maxLength: ... }"` 这类驼峰键写法
- 也排除 `v-bind='{ "maxLength": ... }'` 这类双引号 key 写法
- 最后给剩余的节点自动补上 `maxlength="20"`

非自闭合标签规则 `rules/add-missing-maxlength-paired.yml`：

```yaml
id: add-missing-maxlength-paired
language: html
message: el-input is missing maxlength
severity: warning
rule:
  all:
    - pattern: <el-input $$$ATTRS>$$$CHILDREN</el-input>
    - not:
        matches: has-maxlength-attr
    - not:
        matches: has-maxlength-in-bind-object
    - not:
        any:
          - pattern: <el-input $$$BEFORE maxlength $$$AFTER>$$$CHILDREN</el-input>
          - pattern: <el-input $$$BEFORE maxlength=$VALUE $$$AFTER>$$$CHILDREN</el-input>
          - pattern: <el-input $$$BEFORE :maxlength $$$AFTER>$$$CHILDREN</el-input>
          - pattern: <el-input $$$BEFORE :maxlength=$VALUE $$$AFTER>$$$CHILDREN</el-input>
          - pattern: <el-input $$$BEFORE v-bind:maxlength $$$AFTER>$$$CHILDREN</el-input>
          - pattern: <el-input $$$BEFORE v-bind:maxlength=$VALUE $$$AFTER>$$$CHILDREN</el-input>
fix: <el-input $$$ATTRS maxlength="20">$$$CHILDREN</el-input>
```

工具规则 `utils/has-maxlength-attr.yml`：

```yaml
id: has-maxlength-attr
language: html
rule:
  regex: '<(?:el-input|ElInput)\b[^>]*\s(?::maxlength|v-bind:maxlength)\.[^\s=/>]+(?:\s*=\s*(?:"[^"]*"|''[^'']*''|[^\s>]+))?[^>]*>'
```

工具规则 `utils/has-maxlength-in-bind-object.yml`：

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

这条规则的含义是：

- 匹配成对出现的 `<el-input>...</el-input>`
- 排除已经有 `maxlength=...` 的情况
- 排除 `maxlength` 无值属性写法
- 排除已经有 `:maxlength=...` 的情况
- 排除 `:maxlength` shorthand 写法
- 排除 `:maxlength.prop=...` 这类带修饰符写法
- 排除已经有 `v-bind:maxlength=...` 的情况
- 排除 `v-bind:maxlength` shorthand 写法
- 排除 `v-bind:maxlength.camel=...` 这类带修饰符写法
- 排除 `v-bind="{ maxlength: ... }"` 这类对象展开写法
- 排除 `v-bind="{ 'maxlength': ... }"` 这类带引号 key 的对象展开写法
- 也排除 `v-bind="{ maxLength: ... }"` 这类驼峰键写法
- 也排除 `v-bind='{ "maxLength": ... }'` 这类双引号 key 写法
- 保留原有子节点内容，只补上 `maxlength="20"`

## 4. 写测试用例

测试文件 `rule-tests/add-missing-maxlength-test.yml`：

```yaml
id: add-missing-maxlength
valid:
  - <el-input v-model="form.username" maxlength="20" />
  - <el-input v-model="form.email" :maxlength="maxEmailLength" />
  - <el-input v-model="form.boundUndefined" :maxlength="undefined" />
  - <el-input v-model="form.alias" v-bind:maxlength="maxAliasLength" />
  - <el-input v-model="form.profile" v-bind="{ maxlength: 10 }" />
  - '<el-input v-model="form.profile" v-bind="{ ''maxlength'': 10 }" />'
  - "<el-input v-model=\"form.profile\" v-bind='{ \"maxLength\": 10 }' />"
invalid:
  - <el-input v-model="form.address" placeholder="未设置 maxlength" />
  - |
    <el-input
      v-model="form.phone"
      clearable
      placeholder="带其他属性但不含 maxlength"
    />
```

含义：

- `valid` 里的代码不应该被匹配
- `invalid` 里的代码应该被匹配并提示修复

测试文件 `rule-tests/add-missing-maxlength-paired-test.yml`：

```yaml
id: add-missing-maxlength-paired
valid:
  - <el-input v-model="form.username" maxlength="20"></el-input>
  - <el-input v-model="form.email" :maxlength="maxEmailLength"></el-input>
  - <el-input v-model="form.alias" v-bind:maxlength="maxAliasLength"></el-input>
  - <el-input v-model="form.profile" v-bind="{ maxlength: 10 }"></el-input>
  - |
    <el-input
      v-model="form.note"
      clearable
      maxlength="50"
    ></el-input>
invalid:
  - <el-input v-model="form.address"></el-input>
  - |
    <el-input
      v-model="form.phone"
      clearable
      placeholder="非 self-close 且不含 maxlength"
    ></el-input>
```

## 5. 先验证规则是否正确

运行测试：

```bash
npx -p @ast-grep/cli sg test --skip-snapshot-tests
```

通过后，说明规则逻辑没问题。

## 6. 扫描项目中的问题

扫描当前项目：

```bash
npx -p @ast-grep/cli sg scan --json=pretty .
```

这会列出所有命中的 `el-input`，并显示建议替换内容，包括：

- `<el-input ... />`
- `<el-input ...></el-input>`
- `<ElInput ... />`
- `<ElInput ...></ElInput>`

如果你只想看普通输出，也可以：

```bash
npx -p @ast-grep/cli sg scan .
```

## 7. 自动应用规则

直接把规则改写到文件里：

```bash
npx -p @ast-grep/cli sg scan -U .
```

`-U` 的意思是自动应用所有修复。

执行后，原来这样的代码：

```html
<el-input v-model="form.address" placeholder="未设置 maxlength" />
```

会变成：

```html
<el-input v-model="form.address" placeholder="未设置 maxlength" maxlength="20" />
```

如果原来是非自闭合写法：

```html
<el-input v-model="form.address"></el-input>
```

会变成：

```html
<el-input v-model="form.address" maxlength="20"></el-input>
```

## 8. 如何只处理某个文件

如果你只想处理 `app.vue`：

```bash
npx -p @ast-grep/cli sg scan -U app.vue
```

如果只想查看命中但不改：

```bash
npx -p @ast-grep/cli sg scan app.vue
```

## 9. 这条规则当前能处理什么

它已经验证能正确处理 `app.vue` 里的这些情况：

- 已有 `maxlength="20"`：不会匹配
- 已有 `maxlength`：不会匹配
- 已有 `:maxlength="10"`：不会匹配
- 已有 `:maxlength`：不会匹配
- 已有 `:maxlength="maxEmailLength"`：不会匹配
- 已有 `:maxlength.prop="maxEmailLength"`：不会匹配
- 已有 `:maxlength="undefined"`：不会匹配
- 已有 `v-bind:maxlength="maxAliasLength"`：不会匹配
- 已有 `v-bind:maxlength`：不会匹配
- 已有 `v-bind:maxlength.camel="maxAliasLength"`：不会匹配
- 已有 `v-bind="{ maxlength: 10 }"`：不会匹配
- 已有 `v-bind="{ 'maxlength': 10 }"`：不会匹配
- 已有 `v-bind='{ maxlength: 10 }'`：不会匹配
- 已有 `v-bind="{ maxlength }"`：不会匹配
- 已有 `v-bind="{ maxLength: 10 }"`：不会匹配
- 已有 `v-bind='{ "maxLength": 10 }'`：不会匹配
- 已有 `v-bind='{ maxLength: 10 }'`：不会匹配
- 已有 `v-bind="{ maxLength }"`：不会匹配
- 自闭合且没写 `maxlength`：会匹配并补上
- 非自闭合且没写 `maxlength`：会匹配并补上
- PascalCase 的 `<ElInput>`：会按同样逻辑处理

当前仍然有一个已知限制：

- `v-bind="attrs"` 这类变量展开写法，规则暂时无法静态判断其中是否包含 `maxlength`

## 10. 常见使用方式

日常可以按这个流程来：

```bash
npx -p @ast-grep/cli sg test --skip-snapshot-tests
npx -p @ast-grep/cli sg scan .
npx -p @ast-grep/cli sg scan -U .
```

含义分别是：

- 先验证规则
- 再查看命中结果
- 最后自动修复

## 11. 这次 review 后补上的能力

这次会话里，基于现有规则又补了几类之前遗漏的场景。

- 已支持排除 `v-bind:maxlength="..."`
- 已支持排除 `maxlength` 无值属性和 shorthand 写法
- 已支持排除 `:maxlength.prop="..."` / `v-bind:maxlength.camel="..."` 这类带修饰符写法
- 已支持处理 PascalCase 组件：`<ElInput />` 和 `<ElInput></ElInput>`
- 已支持排除对象展开写法：`v-bind="{ maxlength: 10 }"`
- 已支持排除带单引号 key 的对象展开：`v-bind="{ 'maxlength': 10 }"`
- 已支持排除单引号对象展开写法：`v-bind='{ maxlength: 10 }'`
- 已支持排除对象属性简写：`v-bind="{ maxlength }"`
- 已支持排除对象展开里的驼峰键：`v-bind="{ maxLength: 10 }"` / `v-bind="{ maxLength }"`
- 已支持排除带双引号 key 的驼峰对象写法：`v-bind='{ "maxLength": 10 }'`

为此新增了这些文件：

- 工具规则：`utils/has-maxlength-attr.yml`
- 工具规则：`utils/has-maxlength-in-bind-object.yml`
- 规则文件：`rules/add-missing-maxlength-pascal.yml`
- 规则文件：`rules/add-missing-maxlength-pascal-paired.yml`
- 测试文件：`rule-tests/add-missing-maxlength-pascal-test.yml`
- 测试文件：`rule-tests/add-missing-maxlength-pascal-paired-test.yml`

同时，原有规则也增加了对 shorthand / 修饰符属性 / 对象展开 `v-bind` 的排除逻辑。

这次继续补充后，又覆盖了几类之前容易漏掉的写法：

- 无值属性：`maxlength`
- shorthand：`:maxlength`、`v-bind:maxlength`
- 带修饰符属性：`:maxlength.prop="..."`、`v-bind:maxlength.camel="..."`
- 单引号包裹属性值：`v-bind='{ maxlength: 10 }'`
- 带引号对象 key：`v-bind="{ 'maxlength': 10 }"`、`v-bind='{ "maxLength": 10 }'`
- 对象属性简写：`v-bind="{ maxlength }"`
- 驼峰键对象属性：`v-bind="{ maxLength: 10 }"`、`v-bind="{ maxLength }"`

同时也补了反向用例，确保下面这种不包含 `maxlength` 的对象展开仍然会被命中：

- `v-bind="{ foo: 1 }"`

另外，成对标签规则也补充验证了一个细节：

- `<el-input>...</el-input>` / `<ElInput>...</ElInput>` 在自动补全 `maxlength` 时会保留原有子节点内容

## 12. 当前仍然存在的限制

下面这类写法，规则仍然无法可靠静态判断：

```html
<el-input v-model="form.name" v-bind="attrs" />
```

原因是 `attrs` 的真实内容要到运行时或更大范围的数据流里才能知道，单靠模板层 AST 很难确认其中是否已经包含 `maxlength`。

所以当前策略是：

- `v-bind="{ maxlength: 10 }"`：不会匹配
- `v-bind="{ 'maxlength': 10 }"`：不会匹配
- `v-bind='{ maxlength: 10 }'`：不会匹配
- `v-bind="{ maxlength }"`：不会匹配
- `v-bind="{ maxLength: 10 }"`：不会匹配
- `v-bind='{ "maxLength": 10 }'`：不会匹配
- `v-bind="{ maxLength }"`：不会匹配
- `v-bind="{ foo: 1 }"`：会匹配
- `v-bind="{ maxLengthFoo: 1 }"`：会匹配
- `v-bind="attrs"`：仍可能匹配

## 13. 本次验证结果

这次会话中实际做过两类验证：

- 规则测试：`npx -p @ast-grep/cli sg test --skip-snapshot-tests`
- 项目扫描：`npx -p @ast-grep/cli sg scan --json=pretty .`

测试结果：

- `4 passed; 0 failed`

这次补充测试后，额外验证通过的场景包括：

- `maxlength` 无值属性
- `:maxlength` / `v-bind:maxlength` shorthand
- `:maxlength.prop="..."` / `v-bind:maxlength.camel="..."`
- `v-bind='{ maxlength: 10 }'`
- `v-bind="{ 'maxlength': 10 }"`
- `v-bind="{ maxlength }"`
- `v-bind="{ maxLength: 10 }"`
- `v-bind='{ "maxLength": 10 }'`
- `v-bind="{ maxLength }"`
- `v-bind="{ foo: 1 }"` 仍会命中
- `v-bind="{ maxLengthFoo: 1 }"` 仍会命中
- 成对标签保留原有子节点内容

当前仓库扫描命中 5 处，全部位于 `app.vue`，而且都是预期中“缺少 maxlength”的示例：

- `app.vue:42`
- `app.vue:44`
- `app.vue:50`
- `app.vue:57`
- `app.vue:59`

对应的建议修复分别是补上 `maxlength="20"`。

## 可直接给用户的结论

你可以把这套规则理解成一个“批量治理 Vue 模板”的工具：

- 用 `sgconfig.yml` 告诉 `ast-grep`：`.vue` 按 `html` 解析
- 用 `rules/*.yml` 定义“什么代码是问题”
- 可以按标签写法拆成多条规则，分别处理自闭合和非自闭合场景
- 用 `fix` 定义“怎么自动改”
- 用 `sg test` 验证规则
- 用 `sg scan -U` 把规则真正应用到项目里
