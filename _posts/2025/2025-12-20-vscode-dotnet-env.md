---
layout:     post
title:      "VSCode开源版本的C#开发环境"
subtitle:   "Disable Realtime Monitoring"
date:       2025-12-06 12:15:00
language:   zh-CN
author:     "Gumc"
header-img: "assets/img/2015/post-bg-2015.jpg"
catalog:    true
tags:
    - Game Development
---
- VSCode是微软开发的编辑器，微软官方的C# Dev Kit扩展只能在微软官方的VSCode上使用，如果你安装了就会提示：The C# Dev Kit extension may be used only with Microsoft Visual Studio Code, vscode.dev, GitHub Codespaces from GitHub, Inc., and successor Microsoft, GitHub, and other Microsoft affiliates' products and services. 而微软官方的开源C#环境插件，没有缓存每次打开都很慢，各种支持也比较差。
- 现在市面上AI IDE很多都是基于VSCode开源代码，但开源版本不能使用C# Dev Kit。C# by Resharper插件基本可以替代C# Dev Kit，但它不是开源的，且公开预览结束后肯定会收费。
- 目前最好用的是DotRush，轻量而代码分析功能较弱。代码逻辑分析，格式规范，解决方案预览插件，合并作为替代C# Dev Kit的开发环境，方便使用VSCode的同时，快速迁移到其他基于VSCode的AI IDE，同时需要保持格式规范和代码规范。

## 配置插件链接

VSCode开源版本如需要使用VSCode插件，都需要修改插件商店链接。Antigravity为以下配置：

```json
"antigravity.marketplaceGalleryItemURL": "https://marketplace.visualstudio.com/items",
"antigravity.marketplaceExtensionGalleryServiceURL": "https://marketplace.visualstudio.com/_apis/public/gallery",
```

## 插件或代码包

### DotRush

VSCode插件，提供C#开发环境的功能。基于 Roslyn，代码补全和分析。

当前Antigravity（1.107.1）使用DotRush插件，都会提示QuickFix的问题，似乎if加括号的情况无法正常QuickFix。所以目前不考虑在Antigravity上进行QuickFix。

`[warning]nromanov.dotrush - Code actions of kind 'quickfix' requested but returned code action is of kind 'refactor'. Code action will be dropped. Please check 'CodeActionContext.only' to only return requested code actions.`

这个问题目前无解，用C#(Base language support for C#)也会有相同问题。

### Error Lens

VSCode插件，用于将信息提示在行末，方便提醒信息。

需要配置只有error和warning才显示在行末：

```json
"errorLens.enabledDiagnosticLevels": [
        "error",
        "warning"
        // "info",  <-- 注释掉或删除这两行
        // "hint"   <-- C# 的 Suggestion 通常属于这一级
    ],
```

### **Roslynator**

代码分析器，诊断与修复命名规范、冗余代码、现代语法建议、逻辑简化。提供QuickFix用于快速修复。

使用方式，在.sln或.slnx解决方案相同位置下新建Directory.Build.targets文件，让自动所有csproj都引入：

```xml
<Project>
    <!--在构建初期介入-->
    <ItemGroup>
        <!-- Roslynator.Analyzers (NuGet 包) -->
        <!-- 分析代码逻辑 -->
        <PackageReference Include="Roslynator.Analyzers" Version="*" PrivateAssets="all" />
        <!-- 提供灯泡修复功能 -->
        <PackageReference Include="Roslynator.CodeFixes" Version="*" PrivateAssets="all"/>
        <!-- 可选：格式化分析 -->
        <PackageReference Include="Roslynator.Formatting.Analyzers" Version="*" PrivateAssets="all" />
    </ItemGroup>
</Project>

```

### **CSharpier**

VSCode插件，强制处理空格、换行、缩进、大括号位置等物理布局格式。保存时强制修复。

需要开启IDE的Format On Save

## 配置

### .editorConfig

.editorConfig在配置（不一定其效果，会被CSharpier覆盖）

```
# .editorconfig - 团队 C# 代码规范标准文件（最终合并优化版）
# 适用环境: VS Code

# 【核心】阻止从上级目录继承配置，确保此文件为项目根配置
root = true

# 1. 基础文件格式 (Global Settings)
# 注意：使用 CSharpier 后，大部分格式项由 CSharpier 接管，此处保留作为 IDE 默认回退
# ==========================================

[*]
# 字符集：使用 utf-8 编码
charset = utf-8
# 缩进风格：使用空格 (space) - CSharpier 强制
indent_style = space
# 缩进大小：4 个空格 - CSharpier 强制
indent_size = 4
# 制表符宽度：4 - CSharpier 强制
tab_width = 4
# 换行符：使用 Unix 风格 (LF) - CSharpier 自动处理
end_of_line = crlf
# 插入最后一行：文件末尾强制空行 - CSharpier 强制
insert_final_newline = true
# 去除行尾空格：删除所有行末多余空格 - CSharpier 强制
trim_trailing_whitespace = true
# CSharpier 默认建议 100，团队倾向 120-180，将在 .csharpierrc 中定义
max_line_length = 180
dotnet_style_qualification_for_field = false:silent
dotnet_style_qualification_for_property = false:silent
dotnet_style_qualification_for_method = false:silent
dotnet_style_qualification_for_event = false:silent
dotnet_style_operator_placement_when_wrapping = beginning_of_line


# 针对特定文件格式的缩进优化
[*.{json,travis,yml}]
indent_size = 2

[*.md]
insert_final_newline = false
trim_trailing_whitespace = false

# ==============================================================================
# 2. C# 核心语法规范 (C# Coding Conventions)
# ==============================================================================
[*.cs]

# dotnet_analyzer_diagnostic.severity = default
# ------------------------------------------------------------------------------
# 2.1 访问修饰符与修饰符 (Accessibility & Modifiers)
# ------------------------------------------------------------------------------
dotnet_style_require_accessibility_modifiers = for_non_interface_members:warning # 强制要求显式声明访问修饰符 (如 private, public)
dotnet_style_readonly_field = true:suggestion                               # 建议将未在构造函数外修改的字段标记为只读
dotnet_diagnostic.IDE0044.severity = suggestion                             # [IDE0044] 建议将字段标记为只读

# ------------------------------------------------------------------------------
# 2.2 类型偏好与变量 (Types & Variables)
# ------------------------------------------------------------------------------
dotnet_style_predefined_type_for_locals_parameters_members = true:suggestion # 使用语言关键字 (如 int) 而非框架类型 (如 Int32)
dotnet_style_predefined_type_for_member_access = true:suggestion             # 成员访问时使用语言关键字而非框架类型
csharp_style_var_elsewhere = false:silent                                    # 团队规范：不推荐在其他地方使用 'var'
csharp_style_var_for_built_in_types = false:silent                           # 不推荐对内置类型使用 'var'
csharp_style_var_when_type_is_apparent = false:silent                        # 即便类型显式也不推荐使用 'var' (偏好显式类型声明)

# ------------------------------------------------------------------------------
# 2.3 成员访问与限定 (Member Access & Qualification)
# ------------------------------------------------------------------------------
dotnet_style_qualification_for_event = false:silent                          # 不推荐使用 'this.' 限定事件访问
dotnet_style_qualification_for_field = false:silent                          # 不推荐使用 'this.' 限定字段访问
dotnet_style_qualification_for_method = false:silent                         # 不推荐使用 'this.' 限定方法访问
dotnet_style_qualification_for_property = false:silent                       # 不推荐使用 'this.' 限定属性访问

# ------------------------------------------------------------------------------
# 2.4 代码块与控制流 (Blocks & Control Flow)
# ------------------------------------------------------------------------------
csharp_prefer_braces = true:warning                                          # 强制 if/for/while 必须使用大括号
dotnet_diagnostic.IDE0011.severity = warning                                 # [IDE0011] 强制要求代码块使用大括号
dotnet_diagnostic.RCS1007.severity = warning                                 # [RCS1007] (Roslynator) 强制要求代码块使用大括号

dotnet_style_parentheses_in_arithmetic_binary_operators = always_for_clarity:suggestion # 算术运算建议保留括号以提高清晰度
dotnet_style_parentheses_in_relational_binary_operators = always_for_clarity:suggestion # 关系运算建议保留括号以提高清晰度
dotnet_style_parentheses_in_other_binary_operators = always_for_clarity:suggestion     # 其他二元运算建议保留括号
dotnet_style_parentheses_in_other_operators = never_if_unnecessary:suggestion         # 其他不必要的情况不建议使用括号
dotnet_diagnostic.RCS1002.severity = none                                    # [RCS1002] (Roslynator) 移除冗余括号 (由 style 配置控制)

# ------------------------------------------------------------------------------
# 2.5 表达式偏好与语法糖 (Expressions & Syntax Sugar)
# ------------------------------------------------------------------------------
dotnet_style_object_creation_when_type_is_apparent = true:warning            # 类型明显时简化 'new' 表达式 (C# 9.0+)
csharp_style_implicit_object_creation_when_type_is_apparent = true:warning    # 类型明显时简化 'new' 表达式
dotnet_diagnostic.IDE0090.severity = warning                                 # [IDE0090] 使用简化的 'new' 表达式

dotnet_style_coalesce_expression = true:suggestion                           # 偏好使用空合并表达式 (??)
dotnet_style_collection_initializer = true:suggestion                        # 偏好使用集合初始化器
dotnet_style_explicit_tuple_names = true:suggestion                          # 偏好使用显式元组名称
dotnet_style_null_propagation = true:suggestion                              # 偏好使用空传播操作符 (?.)
dotnet_style_object_initializer = true:suggestion                            # 偏好使用对象初始化器
dotnet_style_prefer_auto_properties = true:silent                             # 偏好使用自动属性
dotnet_style_prefer_compound_assignment = true:suggestion                    # 偏好使用复合赋值 (+=, -=)
dotnet_style_prefer_conditional_expression_over_assignment = true:suggestion  # 偏好条件表达式替代赋值
dotnet_style_prefer_conditional_expression_over_return = true:suggestion      # 偏好条件表达式替代返回
dotnet_style_prefer_inferred_anonymous_type_member_names = true:suggestion    # 偏好推断的匿名类型成员名
dotnet_style_prefer_is_null_check_over_reference_equality_method = true:suggestion # 偏好 'is null' 检查
dotnet_style_prefer_simplified_boolean_expressions = true:suggestion          # 偏好简化的布尔表达式
dotnet_style_prefer_simplified_interpolation = true:suggestion               # 偏好简化的字符串内插
dotnet_style_prefer_collection_expression = when_types_loosely_match:suggestion # C# 12+ 偏好集合表达式
dotnet_style_prefer_foreach_explicit_cast_in_source = when_strongly_typed:suggestion # foreach 显式转换偏好
csharp_style_throw_expression = true:suggestion                              # 偏好使用 throw 表达式
csharp_style_prefer_primary_constructors = true:suggestion                    # 偏好使用主构造函数
csharp_style_prefer_not_pattern = true:suggestion                            # 偏好使用 'not' 模式匹配
csharp_style_prefer_pattern_matching = true:suggestion                       # 偏好开启模式匹配
csharp_style_prefer_switch_expression = true:warning                          # 偏好使用 switch 表达式
csharp_style_pattern_matching_over_as_with_null_check = true:suggestion      # 偏好模式匹配替代 'as' 结合 null 检查
csharp_style_pattern_matching_over_is_with_cast_check = true:suggestion      # 偏好模式匹配替代 'is' 结合类型转换

# ------------------------------------------------------------------------------
# 2.6 表达式体成员 (Expression-Bodied Members)
# ------------------------------------------------------------------------------
csharp_style_expression_bodied_accessors = true:silent                   # 访问器：偏好表达式体成员
csharp_style_expression_bodied_constructors = false:silent                   # 构造函数：不推荐表达式体成员
csharp_style_expression_bodied_indexers = true:silent                    # 索引器：偏好表达式体成员
csharp_style_expression_bodied_lambdas = true:silent                     # Lambda：偏好表达式体成员
csharp_style_expression_bodied_local_functions = false:silent                # 局部函数：不推荐表达式体成员
csharp_style_expression_bodied_methods = false:silent                        # 方法：不推荐表达式体成员
csharp_style_expression_bodied_operators = false:silent                      # 运算符：不推荐表达式体成员
csharp_style_expression_bodied_properties = true:silent                  # 属性：偏好表达式体成员

# ------------------------------------------------------------------------------
# 2.7 命名空间与 Using 指令 (Namespaces & Usings)
# ------------------------------------------------------------------------------
csharp_style_namespace_declarations = block_scoped:silent                    # 偏好块作用域命名空间声明 (Unity 兼容性)
csharp_using_directive_placement = outside_namespace:silent              # Using 指令建议放在命名空间声明外部
dotnet_diagnostic.IDE0005.severity = warning                                 # [IDE0005] 移除不必要的 Using 指令
dotnet_diagnostic.IDE0130.severity = none                                    # [IDE0130] 命名空间不需要按照文件夹结构

csharp_prefer_simple_using_statement = true:suggestion                          # 不推荐简化 using 语句 (保留传统大括号范围)
dotnet_diagnostic.IDE0063.severity = none                                    # [IDE0063] 禁用简化 using 语句提示
csharp_style_prefer_using_declaration = false:none                           # 明确禁用 using 声明建议

# ------------------------------------------------------------------------------
# 2.8 代码质量与清理 (Quality & Cleanup)
# ------------------------------------------------------------------------------
dotnet_diagnostic.CS0219.severity = error                                    # [CS0219] 变量已赋值但其值从未被使用 (视为错误)
dotnet_diagnostic.CS0168.severity = error                                    # [CS0168] 声明了变量但未被使用 (视为错误)
dotnet_diagnostic.CS4014.severity = error                                    # [CS4014] 未等待异步调用 (必须 await 或显式处理 Task)

dotnet_diagnostic.IDE0051.severity = none                                    # [IDE0051] 移除未使用的私有成员
dotnet_diagnostic.IDE0059.severity = suggestion                              # [IDE0059] 移除不必要的赋值
csharp_style_unused_value_assignment_preference = discard_variable:suggestion # 未使用的赋值偏好使用弃元 (_)

dotnet_diagnostic.IDE0060.severity = suggestion                              # [IDE0060] 移除未使用的参数
dotnet_code_quality_unused_parameters = all:silent                           # 标记所有类型的未使用参数

# ------------------------------------------------------------------------------
# 2.9 格式化与排版 (Formatting - Fallback)
# ------------------------------------------------------------------------------
dotnet_style_operator_placement_when_wrapping = beginning_of_line            # 换行时运算符放在新行开头
csharp_indent_labels = one_less_than_current                                 # 标签缩进比当前级别少一级
dotnet_diagnostic.IDE0055.severity = none                                    # [IDE0055] 禁用原生格式化诊断（避免与 CSharpier 冲突）

# ==============================================================================
# 3. 命名样式 (Naming Styles)
# ==============================================================================

# --- 3.1 符号组定义 (Symbol Groups) ---
dotnet_naming_symbols.interface_symbols.applicable_kinds = interface         # 接口符号
dotnet_naming_symbols.interface_symbols.applicable_accessibilities = *       # 任意访问级别

dotnet_naming_symbols.private_internal_fields.applicable_kinds = field       # 私有/内部字段符号
dotnet_naming_symbols.private_internal_fields.applicable_accessibilities = private, internal

dotnet_naming_symbols.public_protected_symbols.applicable_kinds = field, property # 公共/受保护 成员符号
dotnet_naming_symbols.public_protected_symbols.applicable_accessibilities = public, protected

dotnet_naming_symbols.types_symbols.applicable_kinds = class, struct, interface, enum # 类型符号
dotnet_naming_symbols.types_symbols.applicable_accessibilities = *

dotnet_naming_symbols.method_symbols.applicable_kinds = method               # 方法符号
dotnet_naming_symbols.method_symbols.applicable_accessibilities = *

dotnet_naming_symbols.local_var_param_symbols.applicable_kinds = local, parameter # 局部变量与参数符号
dotnet_naming_symbols.local_var_param_symbols.applicable_accessibilities = *

dotnet_naming_symbols.private_static_field_symbols.applicable_kinds = field  # 私有静态字段符号
dotnet_naming_symbols.private_static_field_symbols.applicable_accessibilities = private
dotnet_naming_symbols.private_static_field_symbols.required_modifiers = static

dotnet_naming_symbols.private_const_field_symbols.applicable_kinds = field   # 私有常量符号
dotnet_naming_symbols.private_const_field_symbols.applicable_accessibilities = private
dotnet_naming_symbols.private_const_field_symbols.required_modifiers = const

dotnet_naming_symbols.private_static_property_symbols.applicable_kinds = property # 私有静态属性符号
dotnet_naming_symbols.private_static_property_symbols.applicable_accessibilities = private
dotnet_naming_symbols.private_static_property_symbols.required_modifiers = static

# --- 3.2 样式定义 (Naming Styles) ---
dotnet_naming_style.begins_with_i.required_prefix = I                        # 必须以 I 开头
dotnet_naming_style.begins_with_i.capitalization = pascal_case               # PascalCase 风格

dotnet_naming_style._camelCase_style.required_prefix = _                     # 必须以 _ 开头
dotnet_naming_style._camelCase_style.capitalization = camel_case             # camelCase 风格

dotnet_naming_style.pascal_case_style.capitalization = pascal_case           # PascalCase (首字母大写)
dotnet_naming_style.camel_case_style.capitalization = camel_case             # camelCase (首字母小写)

# --- 3.3 规则绑定 (Naming Rules) ---
dotnet_naming_rule.interface_rule.symbols = interface_symbols                # 接口必须 I 开头
dotnet_naming_rule.interface_rule.style = begins_with_i
dotnet_naming_rule.interface_rule.severity = warning

dotnet_naming_rule.private_field_rule.symbols = private_internal_fields      # 私有字段必须 _camelCase
dotnet_naming_rule.private_field_rule.style = _camelCase_style
dotnet_naming_rule.private_field_rule.severity = warning

dotnet_naming_rule.public_member_rule.symbols = public_protected_symbols     # 公共成员必须 PascalCase
dotnet_naming_rule.public_member_rule.style = pascal_case_style
dotnet_naming_rule.public_member_rule.severity = warning

dotnet_naming_rule.types_rule.symbols = types_symbols                        # 类型名必须 PascalCase
dotnet_naming_rule.types_rule.style = pascal_case_style
dotnet_naming_rule.types_rule.severity = warning

dotnet_naming_rule.methods_rule.symbols = method_symbols                     # 方法名必须 PascalCase
dotnet_naming_rule.methods_rule.style = pascal_case_style
dotnet_naming_rule.methods_rule.severity = warning

dotnet_naming_rule.local_rule.symbols = local_var_param_symbols              # 局部变量必须 camelCase
dotnet_naming_rule.local_rule.style = camel_case_style
dotnet_naming_rule.local_rule.severity = warning

dotnet_naming_rule.private_static_field_rule.symbols = private_static_field_symbols # 私有静态字段必须 _camelCase
dotnet_naming_rule.private_static_field_rule.style = _camelCase_style
dotnet_naming_rule.private_static_field_rule.severity = warning

dotnet_naming_rule.private_const_field_rule.symbols = private_const_field_symbols # 私有常量符号必须 PascalCase
dotnet_naming_rule.private_const_field_rule.style = pascal_case_style
dotnet_naming_rule.private_const_field_rule.severity = warning

dotnet_naming_rule.private_static_property_rule.symbols = private_static_property_symbols # 私有静态属性必须 PascalCase
dotnet_naming_rule.private_static_property_rule.style = pascal_case_style
dotnet_naming_rule.private_static_property_rule.severity = warning

# ==============================================================================
# 4. 静态分析与性能规则 (Analyzers & Performance)
# ==============================================================================

# ------------------------------------------------------------------------------
# 4.1 性能与内存优化 (Performance & Memory)
# ------------------------------------------------------------------------------
# dotnet_diagnostic.CA1822.severity = suggestion                               # [CA1822] 成员可标记为 static
# dotnet_diagnostic.CA1806.severity = warning                                  # [CA1806] 不要忽略方法返回值
# dotnet_diagnostic.CA1825.severity = warning                                  # [CA1825] 使用 Array.Empty<T>() 替代长度为 0 的数组分配

# ------------------------------------------------------------------------------
# 4.2 逻辑安全性 (Safety & Null Checks)
# ------------------------------------------------------------------------------
dotnet_diagnostic.CS8600.severity = warning                                  # [CS8600] 潜在的 Null 分配
dotnet_diagnostic.CS8602.severity = warning                                  # [CS8602] 潜在的 Null 引用解引用
# dotnet_diagnostic.CA2211.severity = warning                                  # [CA2211] 非常量字段不应可见

# ------------------------------------------------------------------------------
# 4.3 Roslynator 分析器规则 (Roslynator Specifics)
# ------------------------------------------------------------------------------
# roslynator_analyzer_released_plugins = all                                 # 启用所有已发布的 Roslynator 插件
# dotnet_analyzer_diagnostic.severity = none                                 # 全局覆盖,也没生效
dotnet_diagnostic.RCS1007.severity = suggestion                                 # [RCS1007] 强制要求在某些逻辑（如 if/for）中使用大括号，即使是单行
dotnet_diagnostic.RCS1032.severity = none                                    # [RCS1032] 冗余括号 (由 style 配置接管)
dotnet_diagnostic.RCS1036.severity = none                                    # [RCS1036] 冗余空行 (CSharpier 已处理)
dotnet_diagnostic.RCS1046.severity = warning                                 # [RCS1046] 异步方法必须以 "Async" 结尾
dotnet_diagnostic.RCS1057.severity = none                                    # [RCS1057] 声明间增加空行 (CSharpier 强制)
dotnet_diagnostic.RCS1059.severity = none                                    # [RCS1059] 在私有字段上加锁(很正常)
dotnet_diagnostic.RCS1016.severity = suggestion                              # [RCS1016] 简化逻辑：建议将 if (x == null) return false; return true; 简化为 return x != null;
dotnet_diagnostic.RCS1102.severity = warning                                 # [RCS1102] 增强枚举检查：确保 switch 覆盖了所有枚举值
dotnet_diagnostic.RCS1123.severity = suggestion
dotnet_diagnostic.RCS1194.severity = none                                    # [RCS1194] 异常实现标准构造函数
dotnet_diagnostic.RCS1207.severity = suggestion                              # [RCS1207] 建议使用 switch 表达式（C# 8.0+）
dotnet_diagnostic.RCS1242.severity = none                                    # [RCS1242] 不要通过只读引用传递非只读结构体
csharp_style_conditional_delegate_call = true:suggestion
csharp_space_around_binary_operators = before_and_after


# ==============================================================================
# 1. 模拟 AnalysisMode=All：开启所有类别的全量诊断
# ==============================================================================
# dotnet_analyzer_diagnostic.category-Design.severity = warning
# dotnet_analyzer_diagnostic.category-Documentation.severity = warning
# dotnet_analyzer_diagnostic.category-Globalization.severity = warning
# dotnet_analyzer_diagnostic.category-Interoperability.severity = warning
# dotnet_analyzer_diagnostic.category-Maintainability.severity = warning
# dotnet_analyzer_diagnostic.category-Naming.severity = warning
# dotnet_analyzer_diagnostic.category-Performance.severity = warning
# dotnet_analyzer_diagnostic.category-Reliability.severity = warning
# dotnet_analyzer_diagnostic.category-Security.severity = warning
# dotnet_analyzer_diagnostic.category-Usage.severity = warning
# dotnet_analyzer_diagnostic.category-Style.severity = none
# dotnet_analyzer_diagnostic.category-CodeQuality.severity = warning

# dotnet_diagnostic.CA1062.severity = none
# dotnet_diagnostic.CA1085.severity = none
# ==============================================================================
# 5. 特定包与第三方屏蔽 (Ignores & ThirdParty)
# ==============================================================================

# 保持对第三方脚本的兼容性：视为生成代码以禁用严格规则

# # 更加精准的匹配模式（涵盖不同层级的 Packages 目录）
# [**/cn.etetet.*/**/*.cs]
# generated_code = true

# dotnet_analyzer_diagnostic.category-Design.severity = none
# dotnet_analyzer_diagnostic.category-Documentation.severity = none
# dotnet_analyzer_diagnostic.category-Globalization.severity = none
# dotnet_analyzer_diagnostic.category-Interoperability.severity = none
# dotnet_analyzer_diagnostic.category-Maintainability.severity = none
# dotnet_analyzer_diagnostic.category-Naming.severity = none
# dotnet_analyzer_diagnostic.category-Performance.severity = none
# dotnet_analyzer_diagnostic.category-Reliability.severity = none
# dotnet_analyzer_diagnostic.category-Security.severity = none
# dotnet_analyzer_diagnostic.category-Usage.severity = none
# dotnet_analyzer_diagnostic.category-Style.severity = none
# dotnet_analyzer_diagnostic.category-CodeQuality.severity = none

# dotnet_diagnostic.CS1263.severity = none
# dotnet_diagnostic.CS1037.severity = none
# # IDE
# dotnet_diagnostic.IDE0011.severity = none
# dotnet_diagnostic.IDE0017.severity = none
# dotnet_diagnostic.IDE0018.severity = none
# dotnet_diagnostic.IDE0019.severity = none
# dotnet_diagnostic.IDE0028.severity = none
# dotnet_diagnostic.IDE0031.severity = none
# dotnet_diagnostic.IDE0059.severity = none
# dotnet_diagnostic.IDE0060.severity = none
# dotnet_diagnostic.IDE0090.severity = none
# dotnet_diagnostic.IDE1006.severity = none
# dotnet_diagnostic.IDE0220.severity = none
# dotnet_diagnostic.IDE0251.severity = none
# dotnet_diagnostic.IDE0290.severity = none
# dotnet_diagnostic.IDE0301.severity = none
# dotnet_diagnostic.IDE0305.severity = none
# dotnet_diagnostic.IDE1006.severity = none



# dotnet_diagnostic.IDE0017.severity = none
# dotnet_diagnostic.IDE0029.severity = none
# dotnet_diagnostic.IDE0034.severity = none
# dotnet_diagnostic.IDE0040.severity = none
# dotnet_diagnostic.IDE0045.severity = none
# dotnet_diagnostic.IDE0046.severity = none
# dotnet_diagnostic.IDE0047.severity = none
# dotnet_diagnostic.IDE0048.severity = none
# dotnet_diagnostic.IDE0057.severity = none
# dotnet_diagnostic.IDE1090.severity = none

# [**/cn.etetet.*.extension/**/*.cs]
# generated_code = false

[**/{cn.etetet.packagemanager,cn.etetet.actorlocation,cn.etetet.core,cn.etetet.loader,cn.etetet.mathematics,cn.etetet.router,cn.etetet.console,cn.etetet.http,cn.etetet.watcher,cn.etetet.referencecollector,cn.etetet.login,cn.etetet.netinner,cn.etetet.excel,cn.etetet.proto,cn.etetet.sourcegenerator,cn.etetet.startconfig,com.alex.tengine,com.michaelo.oxgframe,UniFramework,UnityGameFramework,com.etetet.init,com.halodi.halodi-unity-package-registry-manager,PackageCache,Plugins,ThirdParty,External,Library,CodeMode,Generate}/**]
generated_code = true

dotnet_analyzer_diagnostic.category-Design.severity = none
dotnet_analyzer_diagnostic.category-Documentation.severity = none
dotnet_analyzer_diagnostic.category-Globalization.severity = none
dotnet_analyzer_diagnostic.category-Interoperability.severity = none
dotnet_analyzer_diagnostic.category-Maintainability.severity = none
dotnet_analyzer_diagnostic.category-Naming.severity = none
dotnet_analyzer_diagnostic.category-Performance.severity = none
dotnet_analyzer_diagnostic.category-Reliability.severity = none
dotnet_analyzer_diagnostic.category-Security.severity = none
dotnet_analyzer_diagnostic.category-Usage.severity = none
dotnet_analyzer_diagnostic.category-Style.severity = none
dotnet_analyzer_diagnostic.category-CodeQuality.severity = none

dotnet_diagnostic.CS0246.severity = none
dotnet_diagnostic.CS0618.severity = none
dotnet_diagnostic.CS0649.severity = none
dotnet_diagnostic.CS1037.severity = none
dotnet_diagnostic.CS1123.severity = none
dotnet_diagnostic.CS1263.severity = none
dotnet_diagnostic.CS8669.severity = none
# IDE
dotnet_diagnostic.IDE0011.severity = none
dotnet_diagnostic.IDE1006.severity = none
dotnet_diagnostic.IDE0011.severity = none
dotnet_diagnostic.IDE0017.severity = none
dotnet_diagnostic.IDE0029.severity = none
dotnet_diagnostic.IDE0034.severity = none
dotnet_diagnostic.IDE0040.severity = none
dotnet_diagnostic.IDE0045.severity = none
dotnet_diagnostic.IDE0046.severity = none
dotnet_diagnostic.IDE0047.severity = none
dotnet_diagnostic.IDE0048.severity = none
dotnet_diagnostic.IDE0057.severity = none
dotnet_diagnostic.IDE1090.severity = none
dotnet_diagnostic.IDE1006.severity = none

# dotnet_diagnostic.CS0246.severity = error
# # 显式禁用下划线警告 (CA1707)
# dotnet_diagnostic.CA1707.severity = none
# # 显式禁用命名规范检查 (IDE1006)，防止自定义命名规则触发
# dotnet_diagnostic.IDE1006.severity = none

# ==============================================================================
# 6. 执行约定 (Conventions)
# ==============================================================================
# 务必安装 CSharpier 插件并开启 "Format on Save"
# 务必遵循此 EditorConfig 声明的命名规范

```

### VSCode  setting.json

```json
{
    // ==============================================================================
    // 1. 界面与显示 (Workbench & Appearance)
    // ==============================================================================
    "workbench.editor.wrapTabs": true,                                          // 标签页过多时自动换行显示
    "workbench.startupEditor": "none",                                          // 启动时不打开欢迎页
    "workbench.secondarySideBar.defaultVisibility": "hidden",                   // 默认隐藏辅助侧边栏
    "workbench.editor.enablePreview": false,                                    // 禁用预览模式，单击即持久打开
    "editor.minimap.enabled": false,                                            // 禁用代码小地图
    "terminal.integrated.gpuAcceleration": "off",                               // 禁用终端 GPU 加速
    "telemetry.telemetryLevel": "off",                                          // 禁用遥测数据采集

    // ==============================================================================
    // 2. 文件与性能 (Files & Performance)
    // ==============================================================================
    "explorer.excludeGitIgnore": true,                                          // 资源管理器隐藏 .gitignore 排除项
    "files.trimTrailingWhitespace": true,                                       // 保存时自动清理行尾空格
    "files.insertFinalNewline": true,                                           // 保存时在文件末尾强制插入空行
    "git.enabled": false,                                                       // 禁用编辑器内置 Git 插件
    "files.watcherExclude": {                                                   // 排除高频变动目录的文件监视
        "**/bin/**": true,
        "**/obj/**": true,
        "**/Library/**": true,
        "**/Temp/**": true
    },

    // ==============================================================================
    // 3. 编辑器核心行为 (Editor Core)
    // ==============================================================================
    "editor.formatOnSave": true,                                                // 保存文件时自动格式化
    "editor.defaultFormatter": "nromanov.dotrush",                              // 全局默认格式化器：DotRush
    // "editor.defaultFormatter": "ms-dotnettools.csdevkit",                    // 全局默认格式化器：C# DevKit
    "editor.tabSize": 4,                                                        // 设置缩进为 4 格
    "editor.codeActionsOnSave": {                                               // 定义保存时执行的动作
        "source.fixAll.dotnet": "explicit",                                     // 显式执行所有 .NET 自动修复
        "source.fixAll": "explicit",                                            // 显式执行所有快速修复
        "source.organizeImports": "explicit",                                   // 自动整理并移除未使用的 Using
    },

    // ==============================================================================
    // 4. 扩展插件配置 (Extensions / Plugins)
    // ==============================================================================

    // --- 4.1 DotRush (C# 语言服务) ---
    // "dotrush.enable": true,                                                  // 启用 DotRush 语言服务
    "dotrush.roslyn.diagnosticsFormat":"InfosAsHints",                          // 让信息类建议也以提示形式显示，便于通过灯泡图标快速修复
    "dotrush.roslyn.showItemsFromUnimportedNamespaces":true,                    // 补全能提示并自动添加using
    "dotrush.roslynAnalyzers": true,                                            // 开启 Roslyn 分析器支持
    // "dotrush.backgroundAnalysis": false,                                     // 禁用后台持续分析
    "dotrush.roslyn.loadMetadataForReferencedProjects": false,                  // 禁用引用项目元数据预加载提升速度
    "dotrush.roslyn.analyzerDiagnosticsScope": "Project",                       // 对当前解决方案进行分析

    // --- 4.2 CSharpier (代码强制格式化) ---
    "csharpier.enableDiagnostics": true,                                        // 启用 CSharpier 诊断输出

    // --- 4.3 Error Lens (行内错误诊断) ---
    "errorLens.enabledDiagnosticLevels": ["error", "warning"],                  // Error Lens 仅显示错误和警告

    // --- 4.4 Office Viewer (文档查看) ---
    "vscode-office.editorTheme": "Auto",                                        // Office 查看器主题随系统切换
    "vscode-office.viewAbsoluteLocal": true,                                    // 允许通过绝对路径查看本地文件
    "vscode-office.openOutline": true,                                          // 打开文档时默认显示大纲

    // --- 4.5 英语辞典 (Dictionary & Translation) ---
    "EnglishChineseDictionary.enableHover": true,                               // 启用英汉翻译悬停提示

    // --- 4.6 Antigravity (核心助手设置) ---
    "antigravity.marketplaceGalleryItemURL": "https://marketplace.visualstudio.com/items",
    "antigravity.marketplaceExtensionGalleryServiceURL": "https://marketplace.visualstudio.com/_apis/public/gallery",

    // --- 4.7 C# DevKit (C# 开发工具) ---
    "dotnet.solution.autoOpen": "framework.sln",                                // 自动打开指定的解决方案
    "dotnet.automaticallySyncWithActiveItem": true,                             // 自动同步当前活动项

    // --- 4.8 Trae (AI辅助工具) ---
    "trae.enableCodelens": {
        "enableInlineDocumentation": false,                                     // 禁用内联文档
        "enableInlineExplain": true                                             // 启用内联解释
    },
    "trae.chatLanguage": "cn",                                                  // AI 聊天语言设为中文
    "trae.privacy.mode": true,                                                  // 开启隐私模式
    "trae.tab.enableAutoImport": true,                                          // Tab 补全时自动导入
    "trae.tab.cue": true,                                                       // 开启 Tab 提示

    // ==============================================================================
    // 5. 语言特定细化配置 (Language Specific)
    // ==============================================================================
    "[csharp]": {
        "editor.defaultFormatter": "nromanov.dotrush",                        // C# 首选 DotRush
        // "editor.defaultFormatter": "ms-dotnettools.csdevkit",                   // C# 脚本首选 C# DevKit
        "editor.codeActionsOnSave": {
            "source.fixAll.dotnet": "explicit",                                 // 自动修复诊断规则
            "source.fixAll": "explicit",                                        // 核心：自动应用 QuickFix
            "source.organizeImports": "explicit",                               // 排序 Using 指令
        }
    },

    "json.schemaDownload.enable": true,                                         // 开启 JSON 架构自动下载

    // "markdownFormatter.codeAreaToBlock": "",
    // "markdownFormatter.formatCodes": true,
    // "[markdown]": {
    //     "editor.defaultFormatter": "mervin.markdown-formatter",                 // Markdown 格式化器
    //     "editor.quickSuggestions": {                                            // 开启 Markdown 快速提示
    //         "other": true,
    //         "comments": true,
    //         "strings": true
    //     },
    // },

    // // 设置 JSON 使用 Prettier
    // "[json]": {
    //     "editor.defaultFormatter": "esbenp.prettier-vscode"                  // JSON 使用 Prettier
    // },


    // ==============================================================================
    // 6. 已停用或备份的扩展配置 (Legacy / Backups)
    // ==============================================================================
    // --- OmniSharp (Legacy C# Support) ---
    // "omnisharp.useModernNet": false,                                         // (禁用) 强制旧版 .NET 运行时
    // "dotnetAcquisitionExtension.enableTelemetry": false,                     // (禁用) 禁止遥测数据采集
    // "dotnet.server.useOmnisharp": false,                                     // (禁用) 明确停用 OmniSharp
    // "omnisharp.enableRoslynAnalyzers": true,                                 // 启用分析器
    // "omnisharp.enableEditorConfigSupport": true,                             // 启用 EditorConfig

    // --- Run On Save (备份命令) ---
    // "emeraldwalk.runonsave": { "commands": [ { "match": "\\.cs$", "cmd": "dotnet format..." } ] }
}

```

### Keyboard Shortcuts

| Command   | Keybinding |
| --------- | ---------- |
| Quick Fix | Alt + .    |
