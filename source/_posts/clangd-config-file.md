---
title: Clangd 常用文件 (.clang-format 和 .clang-tidy)
date: 2025-09-11 12:21:11
categories:
- 技术
tags:
- C/C++
---

# Clangd 常用文件 (.clang-format 和 .clang-tidy)

## .clang-format
```yaml
BasedOnStyle: LLVM  # 基础风格采用LLVM（与C++ Core Guidelines较接近）
Language: Cpp
ColumnLimit: 100    # 行宽限制（C++ Core Guidelines建议80-100）
IndentWidth: 4      # 缩进4空格
UseTab: Never       # 禁止使用Tab
TabWidth: 4

# 指针与引用对齐（C++ Core Guidelines推荐左对齐）
PointerAlignment: Left
DerivePointerAlignment: false

# 大括号换行风格（类/函数/控制语句统一换行）
BreakBeforeBraces: Custom
BraceWrapping:
  AfterClass: true      # class后换行
  AfterFunction: true   # 函数后换行
  AfterControlStatement: true  # if/for等控制语句后换行
  SplitEmptyRecord: false
  SplitEmptyFunction: false

AccessModifierOffset: -4  # 强制访问修饰符不缩进（左对齐）

# 空格与对齐
AlignAfterOpenBracket: Align
AlignConsecutiveAssignments: true  # 连续赋值对齐
AlignOperands: Align              # 表达式操作符对齐
AlignTrailingComments: true       # 注释对齐

# 函数与参数
AllowAllParametersOfDeclarationOnNextLine: false  # 参数不换行
BinPackParameters: false          # 参数不压缩（每行一个）
AllowShortFunctionsOnASingleLine: Empty  # 仅空函数允许单行

# 模板与命名空间
AlwaysBreakTemplateDeclarations: Yes  # 模板声明换行
NamespaceIndentation: All             # 命名空间缩进

# 现代C++特性支持
SortIncludes: false                   # 头文件排序

```

## .clang-tidy

注意使用下面文件的时候记得把中文注释删了
```yaml
# .clang-tidy 配置文件 (仿 Visual Assist X 风格)
# 核心思想：注重实效、安全第一、易于重构、提升可读性，避免过于挑剔的风格检查。

Checks: >
  # --- 第零步：重置 ---
  -*

  # --- 第一步：VAX 核心优势：错误预防和代码安全 (最高优先级) ---
  # 这些是VAX也会高亮显示的严重问题，必须启用。
  clang-analyzer-*,
  bugprone-*,

  # --- 第二步：VAX 常用重构建议的自动化 ---
  # 这些检查能自动完成VAX常用“建议”操作，是本配置的重点。
  modernize-use-auto,           # 建议使用auto (类似VAX的变量类型简化建议)
  modernize-use-nullptr,        # 使用nullptr替代NULL (VAX会提示)
  modernize-use-override,       # 明确使用override (VAX会在虚函数提示)
  modernize-use-equals-default, # 建议使用=default (VAX生成代码时常用)
  modernize-use-equals-delete,  # 建议使用=delete
  modernize-replace-auto-ptr,   # 替换已废弃的auto_ptr
  modernize-loop-convert,       # 将手写loop转换为范围for循环 (VAX常用重构)
  modernize-make-unique,        # 使用std::make_unique (VAX会提示资源管理)
  modernize-make-shared,        # 使用std::make_shared
  modernize-return-braced-init-list, # 返回大括号初始化列表
  modernize-use-emplace,        # 使用emplace_back替代push_back (性能提升，VAX风格)

  # --- 第三步：性能和资源管理 (VAX非常关注的点) ---
  performance-*,
  -performance-no-int-to-ptr,   # 禁用：在系统级编程中有时需要
  misc-*,
  -misc-non-private-member-variables-in-classes, # 禁用：有时需要public成员

  # --- 第四步：可读性和维护性 (提升代码清晰度，便于VAX解析和导航) ---
  readability-*,
  -readability-identifier-length,       # 禁用：不强制变量名长度 (VAX不强制)
  -readability-magic-numbers,           # 禁用：字面量检查过于烦人 (与VAX理念不符)
  -readability-implicit-bool-conversion, # 谨慎：有时隐式转换是清晰的，可根据项目开启
  -readability-function-cognitive-complexity, # 禁用：过于主观
  -readability-qualified-auto,          # 禁用：有时`auto`比`const auto&`更简洁
  -readability-isolate-declaration,     # 禁用：VAX不强制每行一个声明

  # --- 第五步：禁用一些与VAX实用主义冲突的“学术性”检查 ---
  -modernize-avoid-c-arrays,            # 禁用：与C接口或嵌入式开发时需要
  -modernize-use-trailing-return-type,  # 禁用：尾置返回类型非必需，风格选择
  -modernize-pass-by-value,             # 谨慎：有时传值并非最佳选择，可根据项目开启
  -bugprone-easily-swappable-parameters, # 禁用：对现有代码库警告太多
  -bugprone-branch-clone,               # 禁用：误报较多

  # --- 第六步：C++核心指南检查 (可选，VAX也会涉及一些最佳实践) ---
  hicpp-*,
  -hicpp-no-array-decay,                # 禁用：与“-modernize-avoid-c-arrays”原因相同
  -hicpp-vararg,                        # 禁用：禁用C风格可变参数，但有时必须使用

  # --- 第七步：明确启用一些非常有用的特定检查 ---
  misc-unused-parameters,       # 检测未使用的参数 (VAX会灰显)
  misc-unused-using-decls,      # 检测未使用的using声明
  misc-definitions-in-headers,  # 警惕在头文件中定义非内联函数

# 检查选项微调 (根据VAX的常见行为调整)
CheckOptions:
  - key: modernize-use-nullptr.NullMacros
    value: 'NULL'
  - key: modernize-use-auto.MinTypeNameLength
    value: '8' # 类型名长度大于8个字符时建议使用auto，避免对`int`, `char*`等使用auto
  - key: modernize-loop-convert.MinConfidence
    value: 'reasonable' # 只对“合理安全”的循环转换给出建议
  - key: modernize-use-emplace.InitializerListTypes
    value: 'false' # 对于初始化列表类型，不使用emplace

# 对头文件也进行检查
HeaderFilterRegex: '.*'

# 不将任何警告提升为错误，保持建议性质（VAX的警告也是建议性的）
WarningsAsErrors: ''

# 代码格式使用项目中单独的.clang-format文件管理
# VAX不强制代码格式，格式与逻辑分离是明智之举
FormatStyle: file
```