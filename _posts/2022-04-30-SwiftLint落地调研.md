---
layout: post
title: 'SwiftLint落地调研'
subtitle: '在项目中加入lint来保证代码质量'
categories: 技术
tags: swift lint
---

## HomeBrew安装

```shell
brew install swiftlint
```

### XCode脚本

```shell
alias swiftlint="/opt/homebrew/bin/swiftlint"

if swiftlint >/dev/null; then
  swiftlint
else
  echo "warning: SwiftLint not installed, download from https://github.com/realm/SwiftLint"
fi
```

## Cocoapods安装

```shell
pod 'SwiftLint'
```

### XCode脚本

```shell
# 启动编译时自动lint
${PODS_ROOT}/SwiftLint/swiftlint

# 启用自动修复功能
${PODS_ROOT}/SwiftLint/swiftlint autocorrect && ${PODS_ROOT}/SwiftLint/swiftlint

```

## Lint配置（.swiftlint.yml）

```yaml
# 规则详情可以查阅 https://realm.github.io/SwiftLint/rule-directory.html

# 从默认启用的集合中禁用规则。
disabled_rules: # rule identifiers turned on by default to exclude from running

  # 需要被修复
  - line_length
  - file_length
  - opening_brace         # {}约束
  - orphaned_doc_comment  # 文档注释约束
  - unused_enumerated     # enumerated约束
  - force_cast
  - force_try
  - cyclomatic_complexity #函数圈复杂度, 10,20
  - type_name

  # 可以考虑自动修复
  - colon                                   # 冒号间距违规
  - trailing_whitespace                     # 行尾部有空格
  - trailing_newline                        # 文件尾部有空白行
  - vertical_parameter_alignment            # 函数参数约束
  - comma                                   # 逗号与空格的约束
  - control_statement                       # 控制语句括号约束
  - return_arrow_whitespace                 # 返回值的空格约束
  - private_over_fileprivate                # private替换filepreivate
  - unneeded_break_in_switch                # switch case 不需要 break
  - trailing_semicolon                      # 行不应该有尾随分号
  - empty_parentheses_with_trailing_closure # 闭包作为最后一个参数时的函数括号约束
  - void_return                             # 无返回值约束
  - syntactic_sugar                         # 语法糖违规 [Int] instead of Array<Int>
  - operator_whitespace                     # 操作符空格约束
  - statement_position                      # 状态条件的空格约束
  - leading_whitespace                      # 文件头部空格
  - vertical_whitespace                     # 垂直空行约束
  - closing_brace                           # 闭包空格约束

  # 不启用
  - class_delegate_protocol                 # Delegate约束
  - no_fallthrough_only                     # fallthrough约束（case中至少有一句才使用fallthrough）
  - comma_inheritance                       # 协议实现逗号约束
  - duplicate_imports                       # 重复引用
  - unavailable_condition                   # #available else 可以用 unavailable 替换
  - implicit_getter                         # 计算的只读属性应避免使用get
  - unused_setter_value                     # 未使用的setter
  - empty_enum_arguments                    # 空enum参数约束
  - nesting                                 # 类型嵌套声明
  - switch_case_alignment                   # switch case 对齐约束
  - multiple_closures_with_trailing_closure # 带有尾随闭包冲突的多个闭包:当传递多个闭包参数时，不应该使用尾随闭包语法
  - legacy_random                           # Prefer using type.random(in:)
  - legacy_constructor                      # 过时方法
  - function_parameter_count                # 函数参数个数控制
  - shorthand_operator                      # += 等符号建议
  - type_body_length                        # 类型长度
  - todo                                    # 不显示TODO
  - mark                                    # mark约束
  - block_based_kvo                         # kvo api 约束
  - identifier_name                         # 变量名约束
  - comment_spacing                         # 注释空格约束
  - trailing_comma                          # 尾随逗号约束
  - computed_accessors_order                # Get Set的顺序问题
  - protocol_property_accessors_order       # Get Set的顺序问题
  - unused_optional_binding                 # 绑定但未使用的可选变量
  - unused_closure_parameter                # 未使用的闭包参数
  - for_where                               # for循环中if语句建议
  - redundant_discardable_let               # let _ = 的初始化检测
  - redundant_optional_initialization       # 可选类型的冗余初始化检测
  - redundant_objc_attribute                # objc相关 冗余检测
  - redundant_string_enum_value             # enum case key与value 为string且相同时可以忽略
  - redundant_void_return                   # 冗余的Void返回值


  # - control_statement

# 启用不来自默认集的规则，一些规则仅仅是可选的，
# 可以通过执行如下指令来查找所有可用的规则:
opt_in_rules: # some rules are turned off by default, so you need to opt-in
  # - accessibility_label_for_image
  # - anonymous_argument_in_multiline_closure
  # - anyobject_protocol
  # - array_init
  # - attributes
  # - balanced_xctest_lifecycle
  - capture_variable                          # 不是常量不应列在闭包的捕获列表中
  # - closure_body_length                     # 闭包长度
  - closure_end_indentation                   # 闭包的{}格式匹配
  - closure_spacing                           # 闭包空格约束
  # - collection_alignment
  # - conditional_returns_on_newline
  - contains_over_filter_count                # filter建议
  - contains_over_filter_is_empty             # filter建议
  - contains_over_first_not_nil               # first建议
  - contains_over_range_nil_comparison        # range建议
  # - convenience_type
  - discarded_notification_center_observer    # 使用block注册通知时，应存储返回的观察者，以便稍后将其删除。
  - discouraged_assert                        # assert约束
  - discouraged_none_name                     # none约束
  # - discouraged_object_literal
  - discouraged_optional_boolean              # 禁用Boolean可选类型
  - discouraged_optional_collection           # 禁用Collection可选类型
  - empty_collection_literal                  # 数组判空约束
  # - empty_count                               # 使用isEmpty替代count直接比较
  - empty_string                              # 字符串判空约束
  # - empty_xctest_method
  - enum_case_associated_values_count         # 枚举的关联值数量
  # - expiring_todo
  # - explicit_acl
  - explicit_enum_raw_value                   # enum 初始raw值
  - explicit_init                             # 避免直接调用.init()
  - explicit_self                             # 实例变量和函数应该使用' self. '显式访问。
  # - explicit_top_level_acl                    # 顶级声明应该显式地指定访问控制级别关键字。
  # - explicit_type_interface                   # 变量的类型声明
  # - extension_access_modifier
  - fallthrough                               # 不可用空的fallthrough
  # - fatal_error_message
  # - file_header
  # - file_name
  # - file_name_no_space
  # - file_types_order
  # - first_where
  # - flatmap_over_map_reduce
  - force_unwrapping                          # 避免强制解包
  # - function_default_parameter_at_end
  # - ibinspectable_in_extension
  - identical_operands                        # 避免两个相同变量的比较
  # - implicit_return
  # - implicitly_unwrapped_optional
  - indentation_width                         # 缩进约束
  # - joined_default_parameter                  # 不建议显示传入函数参数的默认值
  # - last_where
  # - legacy_multiple
  # - legacy_objc_type
  # - let_var_whitespace
  # - literal_expression_end_indentation
  - lower_acl_than_parent                    # 确保内部定义的访问控制级别低于其父级
  # - missing_docs                             # 缺少文档
  # - modifier_order
  # - multiline_arguments
  # - multiline_arguments_brackets
  # - multiline_function_chains
  # - multiline_literal_brackets
  # - multiline_parameters
  # - multiline_parameters_brackets
  # - nimble_operator
  # - no_extension_access_modifier             # 不对extension使用权限修饰符
  # - no_grouping_extension
  - nslocalizedstring_key                     # 对字符串国际化的约束
  # - nslocalizedstring_require_bundle          # 字符串国际化应该指定bundle
  - number_separator
  # - object_literal
  - operator_usage_whitespace                 # 操作符空格约束
  # - optional_enum_case_matching               # 枚举case可选值约束
  - overridden_super_call                     # overridden 方法必须调用super
  # - override_in_extension                     #overrridden 方法不能写在extension中
  # - pattern_matching_keywords                 # 通过将关键字移出元组来组合多个模式匹配绑定。
  # - prefer_nimble
  - prefer_self_in_static_references          # 静态引用的前缀应该是Self而不是类名。
  - prefer_self_type_over_type_of_self        # 调方法/访问属性时，用Self替换type(of:self)
  # - prefer_zero_over_explicit_init
  # - prefixed_toplevel_constant                # 常量需要用k开头
  - private_action                            # IBActions 需要 private
  - private_outlet                            # IBOutlets 需要 private
  # - private_subject
  - prohibited_interface_builder              # 避免使用Interface Builder创建视图。
  - prohibited_super_call                     # 部分API不需要调用父类方法
  # - raw_value_for_camel_cased_codable_enum
  # - reduce_into
  - redundant_nil_coalescing                # nil
  - redundant_type_annotation               # 冗余的类型注释
  # - required_deinit                         # class都必须有deinit
  - required_enum_case                      # 符合指定协议的枚举必须实现特定的case。
  - return_value_from_void_function         # 无返回值类型的函数，不能return返回值

  # - sorted_first_last
  - sorted_imports                          # import排序
  - static_operator                         # 操作符重载应该声明为Static
  # - strict_fileprivate                      # 避免使用fileprivate
  - strong_iboutlet                         # IBOutlet约束
  # - switch_case_on_newline
  # - toggle_bool
  # - trailing_closure
  # - type_contents_order
  # - typesafe_array_init
  # - unavailable_function
  # - unneeded_parentheses_in_closure_argument
  # - unowned_variable_capture
  - untyped_error_in_catch                  # Catch语句不应该声明没有类型转换的Error变量。
  # - unused_declaration
  - unused_import                           # import应当被使用
  # - vertical_parameter_alignment_on_call
  - vertical_whitespace_between_cases
  # - vertical_whitespace_closing_braces
  # - vertical_whitespace_opening_braces
  - weak_delegate                           # delegate应该用weak，以避免引用循环。
  # - yoda_condition

  # 单元测试相关
  # - quick_discouraged_call
  # - quick_discouraged_focused_test
  # - quick_discouraged_pending_test
  # - single_test_class
  # - test_case_accessibility
  #- xct_specific_matcher

# Alternatively, specify all rules explicitly by uncommenting this option:
# only_rules: # delete `disabled_rules` & `opt_in_rules` if using this
#   - empty_parameters
#   - vertical_whitespace

# 执行linting时包含的路径。如果出现这个 `--path` 会被忽略
# included: # paths to include during linting. `--path` is ignored if present.
  # - Source

# 执行linting时，忽略的路径。 优先级比 `included` 更高。
# excluded: # paths to ignore during linting. Takes precedence over `included`.
#   - Pods

# 执行analyze时生效的规则
analyzer_rules: # Rules run by `swiftlint analyze`
  - explicit_self

# 可配置的规则可以通过这个配置文件来自定义
# 二进制规则可以设置他们的严格程度
# configurable rules can be customized from this configuration file
# binary rules can set their severity level

# 类型强制推测约束
force_cast: error # implicitly

force_try:
  severity: error # explicitly

cyclomatic_complexity:
  ignores_case_statements: true

# 元祖长度约束
large_tuple:
  warning: 4
  error: 6

# 行数约束
line_length:
  warning: 160
  error: 1200
  ignores_function_declarations: true
  ignores_comments: true
  ignores_interpolated_strings: true
  ignores_urls: true

# 可以通过一个数组同时进行隐式设置
# they can set both implicitly with an array
# type_body_length:
#   - 300 # warning
#   - 400 # error

# 或者也可以同时进行显式设置
# or they can set both explicitly
file_length:
  warning: 800
  error: 1200

function_body_length:
  warning: 300
  error: 400

# 命名规则可以设置最小长度和最大程度的警告/错误
# 此外它们也可以设置排除在外的名字
# naming rules can set warnings/errors for min_length and max_length
# additionally they can set excluded names
type_name:
  min_length: 4 # 变量名最小长度
  max_length: # warning and error
    warning: 40
    error: 50
  excluded: iPhone # excluded via string
  allowed_symbols: ["_"] # these are allowed in type names
  validates_start_with_lowercase: true

reporter: "xcode" # reporter type (xcode, json, csv, checkstyle, codeclimate, junit, html, emoji, sonarqube, markdown, github-actions-logging)
```

## 自动修复配置（.swiftlint.auto.yml）

[传送门](https://github.com/realm/SwiftLint/issues/1451)

```yaml
whitelist_rules:
  - comma
  - empty_parameters
  - trailing_comma
  - vertical_whitespace

trailing_comma:
  mandatory_comma: true

vertical_whitespace:
  max_empty_lines: 2
```

## 版本提示

```shell
Showing Recent Messages
The `swiftlint autocorrect` command is no longer available.

Please use `swiftlint --fix` instead.

'whitelist_rules' has been renamed to 'only_rules' and will be completely removed in a future release.
```

## 临时生效

```swift
// 文件中临时关闭某个规则
// swiftlint:disable <rule>

// 启用某个规则，与上个注释配对使用
// swiftlint:enable <rule>

```

## Pods配置

```yaml
excluded:
  - Pods

included:
  - ../DevelopmentPodName
```

## 规则修复

[规则查询与修复建议](https://realm.github.io/SwiftLint/rule-directory.html)

## 参考资料

[SwiftLint](https://github.com/realm/SwiftLint/blob/master/README_CN.md)

[How to fix the "SwiftLint not installed" warning on M1 Macs](https://thisdevbrain.com/how-to-fix-the-swiftlint-not-installed-warning-on-m1-macs/)

[[No lintable files found at paths SwiftLint](https://stackoverflow.com/questions/55732527/no-lintable-files-found-at-paths-swiftlint)](https://stackoverflow.com/questions/55732527/no-lintable-files-found-at-paths-swiftlint)