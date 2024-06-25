<!--yml

category: 未分类

date: 2024-05-27 14:28:58

-->

# Rails 7.1 引入了枚举的验证选项 | Saeloun 博客

> 来源：[`blog.saeloun.com/2024/01/02/rails-enums-validate-option/`](https://blog.saeloun.com/2024/01/02/rails-enums-validate-option/)

枚举是一种数据类型，允许我们定义一组命名常量。Rails 7.1 通过引入`:validate`选项，枚举处理得到了显著的增强。

这将允许在 ActiveRecord 模型中更灵活和强健地验证枚举值。

在这篇博客文章中，我们将探讨这个[变更](https://github.com/rails/rails/pull/49100)如何影响我们的代码，以及在 Rails 的早期版本中如何处理枚举值的验证。

### 之前

在 Rails 的早期版本中，如果我们将一个不正确的值赋给枚举，则会引发`ArgumentError`。

我们可以通过以下示例来说明这一点。

```
class Holiday < ApplicationRecord

  enum type: [:national, :regional]

end

holiday = Holiday.last

holiday.type = :optional
```

```
'optional' is not a valid type (ArgumentError)
```

在上面的片段中，我们有一个带有枚举`type`列的`Holiday`模型，该列可以采用`national`或`regional`作为有效的节日类型。

当我们尝试将`optional`作为一个节日类型传递时，它引发了一个`ArgumentError`。

### 后

Rails 7.1 引入了枚举的`validate`选项。它将允许开发人员在保存枚举值之前强制执行验证检查。

让我们尝试将`validate: true`传递给上面的片段，并看看从这个版本引入的变化。

```
class Holiday < ApplicationRecord

  enum type: [:national, :regional], validate: true

end

holiday = Holiday.last

holiday.type = :optional
holiday.valid? # Output: false

holiday.type = nil
holiday.valid? # Output: false

holiday.type= :national
holiday.valid? # Output: true
```

我们也可以传递其他验证选项

```
class Holiday < ApplicationRecord

  enum type: [:national, :regional], validate: { allow_nil: true }

end

holiday = Holiday.last

holiday.type = :optional
holiday.valid? # Output: false

holiday.type = nil
holiday.valid? # Output: true
```

如果我们不传递`validate`选项，它会像早期版本一样引发`ArgumentError`。

要了解更多关于这个功能的信息，请参考这个[PR](https://github.com/rails/rails/pull/49100)
