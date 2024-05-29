<!--yml
category: 未分类
date: 2024-05-27 14:28:58
-->

# Rails 7.1 Introduced Validate Option For Enums | Saeloun Blog

> 来源：[https://blog.saeloun.com/2024/01/02/rails-enums-validate-option/](https://blog.saeloun.com/2024/01/02/rails-enums-validate-option/)

An Enum is a data type that allows us to define a set of named constants. Rails 7.1 brings a notable enhancement to enum handling by introducing the `:validate` option.

It will allow more flexible and robust validation of enum values within the ActiveRecord models.

In this blog post, we’ll explore how this [change](https://github.com/rails/rails/pull/49100) affects our code and how the validations for enum values were handled in previous versions of Rails.

### Before

In previous versions of Rails, if we assign an incorrect value to the enum then it used to raise an `ArgumentError`.

We can illustrate this by the following example.

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

In the above snippet, we have a `Holiday` model with an enum `type` column which can take `national` or `regional` as valid holiday types.

When we tried passing `optional` as a holiday type it raised an `ArgumentError`.

### After

Rails 7.1 introduced the `validate` option for enums. It will allow developers to enforce the validation checks before saving enum values.

Let’s try passing the `validate: true` to the above snippet and see the change introduced from this version.

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

We can also pass additional validation options

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

If we don’t pass the `validate` option it will raise the `ArgumentError` as in the earlier versions.

To know more about this feature, please refer to this [PR](https://github.com/rails/rails/pull/49100)