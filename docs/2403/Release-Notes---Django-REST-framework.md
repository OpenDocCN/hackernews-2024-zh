<!--yml

category: 未分类

日期：2024年5月27日 15:03:28

-->

# 发布说明 - Django REST框架

> 来源：[https://www.django-rest-framework.org/community/release-notes/#315x-series](https://www.django-rest-framework.org/community/release-notes/#315x-series)

次要版本号（0.0.x）用于与API兼容的更改。你应该能够在次要点发布之间进行升级而无需任何其他代码更改。

中等版本号（0.x.0）可能包含API更改，符合[弃用政策](#deprecation-policy)。在升级中间版本之前，请仔细阅读发布说明。

主要版本号（x.0.0）保留用于重大项目里程碑。

REST框架发布遵循正式的弃用政策，与[Django的弃用政策](https://docs.djangoproject.com/en/stable/internals/release-process/#internal-release-deprecation-policy)一致。

版本1.0中存在的功能弃用时间表如下：

+   Version 1.1会保持与1.0 **完全向后兼容**，但会引发`RemovedInDRF13Warning`警告，该警告属于`PendingDeprecationWarning`的子类，如果你使用即将被弃用的功能。这些警告**默认情况下是静默的**，但在你准备好开始迁移任何必要的更改时可以显式启用。例如，如果你开始使用`python -Wd manage.py test`运行测试，你会被警告需要进行的任何API更改。

+   Version 1.2将这些警告升级为`DeprecationWarning`的子类，默认情况下会很响亮。

+   Version 1.3将完全移除API中已弃用的部分。

请注意，根据Django的政策，文档中未提及的框架部分通常应视为私有API，并可能会发生更改。

要将Django REST框架升级到最新版本，请使用pip：

```
pip install -U djangorestframework 
```

你可以使用`pip show`确定当前安装的版本：

```
pip show djangorestframework 
```

* * *

日期：2024年3月22日

+   修复`SearchFilter`在调用`.get_search_terms`时对引号和逗号分隔字符串的处理问题，当被自定义类调用。详见[[#9338](https://github.com/encode/django-rest-framework/issues/9338)]

+   恢复了3.15.0版本中包含的意外副作用的数量。详见[[#9331](https://github.com/encode/django-rest-framework/issues/9331)]

日期：2024年3月15日

+   Django 5.0和Python 3.12支持 [[#9157](https://github.com/encode/django-rest-framework/pull/9157)]

+   使用POST方法而不是GET来执行浏览器API中的注销 [[9208](https://github.com/encode/django-rest-framework/pull/9208)]

+   添加了对jQuery 3.7.1的支持并放弃了先前版本 [[#9094](https://github.com/encode/django-rest-framework/pull/9094)]

+   使用str作为默认路径转换器 [[#9066](https://github.com/encode/django-rest-framework/pull/9066)]

+   在Python 3.11中添加对@action装饰器中的http.HTTPMethod的文档支持 [[#9067](https://github.com/encode/django-rest-framework/pull/9067)]

+   更新 exceptions.md。 [[#9071](https://github.com/encode/django-rest-framework/pull/9071)]

+   部分序列化器不应具有必需字段。 [[#7563](https://github.com/encode/django-rest-framework/pull/7563)]

+   将模型字段的 'default' 传播到序列化器字段。 [[#9030](https://github.com/encode/django-rest-framework/pull/9030)]

+   允许在 `ListSerializer` 中覆盖 `child.run_validation` 调用。 [[#8035](https://github.com/encode/django-rest-framework/pull/8035)]

+   将 `SearchFilter` 行为对齐到 `django.contrib.admin` 搜索。 [[#9017](https://github.com/encode/django-rest-framework/pull/9017)]

+   添加类名到未知字段错误。 [[#9019](https://github.com/encode/django-rest-framework/pull/9019)]

+   修复：分页响应模式。 [[#9049](https://github.com/encode/django-rest-framework/pull/9049)]

+   修复 `ChoiceField` 中支持 `IntEnum` 的选项。 [[#8955](https://github.com/encode/django-rest-framework/pull/8955)]

+   修复 `SearchFilter` 渲染带有无效值的搜索字段。 [[#9023](https://github.com/encode/django-rest-framework/pull/9023)]

+   修复 `OpenAPI` `Schema` yaml 渲染 `timedelta`。 [[#9007](https://github.com/encode/django-rest-framework/pull/9007)]

+   修复 `NamespaceVersioning` 在非 `None` 命名空间上忽略 `DEFAULT_VERSION`。 [[#7278](https://github.com/encode/django-rest-framework/pull/7278)]

+   为 CoreAPI 添加弃用警告。 [[#7519](https://github.com/encode/django-rest-framework/pull/7519)]

+   移除触发完整表加载的 `field.choices` 的使用。 [[#8950](https://github.com/encode/django-rest-framework/pull/8950)]

+   允许混合大小写的字符串值进行 `BooleanField` 验证。 [[#8970](https://github.com/encode/django-rest-framework/pull/8970)]

+   修复 `BrowsableAPIRenderer` 与 `ListSerializer` 的使用。 [[#7530](https://github.com/encode/django-rest-framework/pull/7530)]

+   更改两个权限类的 `OR` 语义。 [[#7522](https://github.com/encode/django-rest-framework/pull/7522)]

+   移除对 `pytz` 的依赖。 [[#8984](https://github.com/encode/django-rest-framework/pull/8984)]

+   将 `set_value` 作为 `Serializer` 的方法。 [[#8001](https://github.com/encode/django-rest-framework/pull/8001)]

+   修复 `URLPathVersioning` 反向回退。 [[#7247](https://github.com/encode/django-rest-framework/pull/7247)]

+   在 `DecimalField` 的 `min_value` 和 `max_value` 参数中发出十进制类型警告。 [[#8972](https://github.com/encode/django-rest-framework/pull/8972)]

+   修复选择值映射 [[#8968](https://github.com/encode/django-rest-framework/pull/8968)]

+   重构读取函数以使用文件处理的上下文管理器。 [[#8967](https://github.com/encode/django-rest-framework/pull/8967)]

+   修复：在视图上未设置时，回退到 `CursorPagination` 排序。 [[#8954](https://github.com/encode/django-rest-framework/pull/8954)]

+   使用 `dict` 替换 `OrderedDict`。 [[#8964](https://github.com/encode/django-rest-framework/pull/8964)]

+   重构get_field_info方法以在SimpleMetadata类中包含max_digits和decimal_places属性 [[#8943](https://github.com/encode/django-rest-framework/pull/8943)]

+   为验证器实现`__eq__`方法 [[#8925](https://github.com/encode/django-rest-framework/pull/8925)]

+   确保CursorPagination在排序字段中尊重null值 [[#8912](https://github.com/encode/django-rest-framework/pull/8912)]

+   将ZoneInfo用作时区数据的主要来源 [[#8924](https://github.com/encode/django-rest-framework/pull/8924)]

+   为TokenAdmin添加用户名搜索字段 (#8927) [[#8934](https://github.com/encode/django-rest-framework/pull/8934)]

+   在many=False时，在SlugRelatedField中处理嵌套关系 [[#8922](https://github.com/encode/django-rest-framework/pull/8922)]

+   将jQuery版本升级到3.6.4并更新引用链接 [[#8909](https://github.com/encode/django-rest-framework/pull/8909)]

+   支持UniqueConstraint [[#7438](https://github.com/encode/django-rest-framework/pull/7438)]

+   允许Request、Response、Field和GenericAPIView进行下标访问。这允许对这些类进行泛型类型检查 [[#8825](https://github.com/encode/django-rest-framework/pull/8825)]

+   Feat: 添加一些更改到ValidationError，以支持django风格的验证错误 [[#8863](https://github.com/encode/django-rest-framework/pull/8863)]

+   修复在DjangoModelPermissions中尊重`can_read_model`权限 [[#8009](https://github.com/encode/django-rest-framework/pull/8009)]

+   添加SimplePathRouter [[#6789](https://github.com/encode/django-rest-framework/pull/6789)]

+   更新后重新预获取相关对象 [[#8043](https://github.com/encode/django-rest-framework/pull/8043)]

+   修复FilePathField的required参数 [[#8805](https://github.com/encode/django-rest-framework/pull/8805)]

+   如果`basename`不唯一，则引发ImproperlyConfigured异常 [[#8438](https://github.com/encode/django-rest-framework/pull/8438)]

+   在openapi中使用PrimaryKeyRelatedField pkfield [[#8315](https://github.com/encode/django-rest-framework/pull/8315)]

+   在BasicAuthentication中用split替换partition [[#8790](https://github.com/encode/django-rest-framework/pull/8790)]

+   修复BooleanField的allow_null行为 [[#8614](https://github.com/encode/django-rest-framework/pull/8614)]

+   在ListField中处理Django的ValidationErrors [[#6423](https://github.com/encode/django-rest-framework/pull/6423)]

+   移除一些内联CSS。在可能且可用时添加CSP nonce [[#8783](https://github.com/encode/django-rest-framework/pull/8783)]

+   在Token admin中为用户选择使用自动完成小部件 [[#8534](https://github.com/encode/django-rest-framework/pull/8534)]

+   使可浏览的API与强CSP兼容 [[#8784](https://github.com/encode/django-rest-framework/pull/8784)]

+   避免内联脚本执行以注入CSRF令牌 [[#7016](https://github.com/encode/django-rest-framework/pull/7016)]

+   减少对 `inflection` 的全局依赖 [[#8017](https://github.com/encode/django-rest-framework/pull/8017)] [[#8781](https://github.com/encode/django-rest-framework/pull/8781)]

+   注册 Django URL [[#8778](https://github.com/encode/django-rest-framework/pull/8778)]

+   为 `TokenProxy` 实现了详细名称翻译 [[#8713](https://github.com/encode/django-rest-framework/pull/8713)]

+   在 `DurationField` 反序列化中正确处理 `OverflowError` [[#8042](https://github.com/encode/django-rest-framework/pull/8042)]

+   适当地修复 OpenAPI 操作名称的复数形式 [[#8017](https://github.com/encode/django-rest-framework/pull/8017)]

+   在模式渲染中将 `SafeString` 表示为普通字符串 [[#8429](https://github.com/encode/django-rest-framework/pull/8429)]

+   修复 #8771 - 即使 `_ignore_model_permissions = True`，也要检查认证 [[#8772](https://github.com/encode/django-rest-framework/pull/8772)]

+   修复 `page` 查询参数为空字符串时的 404 错误 [[#8578](https://github.com/encode/django-rest-framework/pull/8578)]

+   修复在 `ListSerializer.to_representation` 中的实例检查的无限递归 [[#8726](https://github.com/encode/django-rest-framework/pull/8726)] [[#8727](https://github.com/encode/django-rest-framework/pull/8727)]

+   如果输入的数字过大，`FloatField` 将崩溃 [[#8725](https://github.com/encode/django-rest-framework/pull/8725)]

+   添加缺失的 `DurationField` 到 `SimpleMetada` 的 `label_lookup` [[#8702](https://github.com/encode/django-rest-framework/pull/8702)]

+   增加对 Python 3.11 的支持 [[#8752](https://github.com/encode/django-rest-framework/pull/8752)]

+   使分页类中的请求保持一致可用 [[#8764](https://github.com/encode/django-rest-framework/pull/9764)]

+   可能在 `DecimalFields` 表示中移除尾随零 [[#6514](https://github.com/encode/django-rest-framework/pull/6514)]

+   添加一个获取序列化器字段名称（OpenAPI）的方法 [[#7493](https://github.com/encode/django-rest-framework/pull/7493)]

+   为 `OperandHolder` 类添加 `__eq__` 方法 [[#8710](https://github.com/encode/django-rest-framework/pull/8710)]

+   当不进行测试时，避免导入 `django.test` 包 [[#8699](https://github.com/encode/django-rest-framework/pull/8699)]

+   保留包装的 Django 异常的异常消息 [[#8051](https://github.com/encode/django-rest-framework/pull/8051)]

+   将 `examples` 和 `format` 包含到 `CursorPagination` 的 OpenAPI 模式中 [[#8687](https://github.com/encode/django-rest-framework/pull/8687)] [[#8686](https://github.com/encode/django-rest-framework/pull/8686)]

+   修复 `Request` 上的 `deepcopy` 导致的无限递归 [[#8684](https://github.com/encode/django-rest-framework/pull/8684)]

+   重构：用 `contextlib.suppress()` 替换 `try/except` [[#8676](https://github.com/encode/django-rest-framework/pull/8676)]

+   对 `SerializeMethodField` 的文档字符串进行了小修正 [[#8629](https://github.com/encode/django-rest-framework/pull/8629)]

+   小型重构：不必要使用`list()`函数。[[#8672](https://github.com/encode/django-rest-framework/pull/8672)]

+   不必要的列表推导。[[#8670](https://github.com/encode/django-rest-framework/pull/8670)]

+   使用正确的类指示当前弃用。[[#8665](https://github.com/encode/django-rest-framework/pull/8665)]

日期：2022年9月22日

+   不再支持Django 2.2。[[#8662](https://github.com/encode/django-rest-framework/pull/8662)]

+   Django 4.1兼容性。[[#8591](https://github.com/encode/django-rest-framework/pull/8591)]

+   向`generateschema`管理命令添加`--api-version` CLI选项。[[#8663](https://github.com/encode/django-rest-framework/pull/8663)]

+   将`is_valid(raise_exception=False)`强制作为关键字参数。[[#7952](https://github.com/encode/django-rest-framework/pull/7952)]

+   停止在验证器上调用`set_context`。[[#8589](https://github.com/encode/django-rest-framework/pull/8589)]

+   从`ErrorDetails.__ne__`返回`NotImplemented`。[[#8538](https://github.com/encode/django-rest-framework/pull/8538)]

+   在设置自定义时区时，不要评估`DateTimeField.default_timezone`。[[#8531](https://github.com/encode/django-rest-framework/pull/8531)]

+   使得Browsable API中的相对URL可点击。[[#8464](https://github.com/encode/django-rest-framework/pull/8464)]

+   支持`ManyRelatedField`在指定的点符号属性不存在时回退到默认值。匹配`ManyRelatedField.get_attribute`到`Field.get_attribute`。[[#7574](https://github.com/encode/django-rest-framework/pull/7574)]

+   使`schemas.openapi.get_reference`公开。[[#7515](https://github.com/encode/django-rest-framework/pull/7515)]

+   使`ReturnDict`支持Python 3.9及更高版本的`dict`联合操作符。[[#8302](https://github.com/encode/django-rest-framework/pull/8302)]

+   更新限流以检查是否设置了`request.user`再检查用户是否已认证。[[#8370](https://github.com/encode/django-rest-framework/pull/8370)]

日期：2021年12月15日

+   使用基于函数的`@api_view`反转模式命名更改。[#8297]

日期：2021年12月13日

+   Django 4.0兼容性。[#8178]

+   向`ListSerializer`添加`max_length`和`min_length`选项。[#8165]

+   向`AutoSchema`添加`get_request_serializer`和`get_response_serializer`挂钩。[#7424]

+   修复可空只读字段的OpenAPI表示。[#8116]

+   在API模式输出中尊重`UNICODE_JSON`设置。[#7991]

+   修复`RemoteUserAuthentication`。[#7158]

+   使字段构造函数仅限关键字。[#7632]

* * *

日期：2021年3月26日

+   反转使用`deque`而非`list`来跟踪限流历史的变更。（由于与DjangoRedis缓存后端不兼容。参见＃7870）[#7872]

日期：2021年3月25日

+   当使用多个数据库配置时正确处理`ATOMIC_REQUESTS`。[#7739]

+   当配置了`LimitOffsetPagination`但请求中未包含分页参数时，绕过`COUNT`查询。[#6098]

+   尊重 `DecimalField` 上的 `allow_null=True`。[#7718]

+   使用 `BooleanField` 允许标题大小写为 `"Yes"` / `"No"`。[#7739]

+   添加 `PageNumberPagination.get_page_number()` 方法以覆盖行为。[#7652]

+   修复在 OpenAPI 模式中呈现时间差值时的问题，当存在默认、最小或最大字段时。[#7641]

+   在可浏览的 API 表单中使用缩进呈现 `JSONFields`。[#6243]

+   在管理员 Token 视图中删除不必要的数据库查询。[#7852]

+   在传递布尔值到 `PrimaryKeyRelatedField` 字段时引发验证错误，而不是将其转换为整数。[#7597]

+   不要将模型属性作为自动生成的排序字段与 `OrderingFilter` 一起包括。[#7609]

+   使用 `deque` 替代 `list` 来跟踪节流 `.history`。[#7849]

日期：2020年10月13日

+   修复如果导入了 `rest_framework.authtoken.models` 但 `rest_framework.authtoken` 不在 `INSTALLED_APPS` 中的问题。[#7571]

+   在 OpenAPI 模式中忽略 `BrowsableAPIRenderer` 的子类。[#7497]

+   在序列化器字段中进行更窄的异常捕获，以确保不掩盖任何损坏的 `get_queryset()` 方法中的错误。[#7480]

日期：2020年9月28日

+   添加 `TokenProxy` 迁移。[#7557]

日期：2020年9月28日

+   向 `generateschema` 命令添加 `--file` 选项。[#7130]

+   支持用于 OpenAPI 模式生成的 `tags`。参见 [模式文档](https://www.django-rest-framework.org/api-guide/schemas/#grouping-operations-with-tags)。[#7184]

+   支持自定义操作 ID 以进行模式生成。参见 [模式文档](https://www.django-rest-framework.org/api-guide/schemas/#operationid)。[#7190]

+   支持 OpenAPI 组件进行模式生成。参见 [模式文档](https://www.django-rest-framework.org/api-guide/schemas/#components)。[#7124]

+   `AutoSchema` 上的以下方法成为公共 API：`get_path_parameters`、`get_pagination_parameters`、`get_filter_parameters`、`get_request_body`、`get_responses`、`get_serializer`、`get_paginator`、`map_serializer`、`map_field`、`map_choice_field`、`map_field_validators`、`allows_filters`。参见 [模式文档](https://www.django-rest-framework.org/api-guide/schemas/#autoschema)。

+   支持 Django 3.1 的数据库不可知的 `JSONField`。[#7467]

+   `SearchFilter` 现在支持在 `JSONField` 和 `HStoreField` 模型字段上进行嵌套搜索。[#7121]

+   `SearchFilter` 现在支持在 `annotate()` 字段上进行搜索。[#6240]

+   在管理员 URL 中，`authtoken` 模型不再公开 `pk`。[#7341]

+   为请求实例添加 `__repr__`。[#7239]

+   对基本身份验证凭据进行 UTF-8 解码，并以 Latin-1 回退。[#7193]

+   `CharField` 将代理字符视为验证失败。[#7026]

+   不要将可调用对象作为模式中的默认值。[#7105]

+   改进 `ListField` 的模式输出以包括所有可用的子信息。[#7137]

+   允许在 `BooleanField` 的模式输出中包括 `default=False`。[#7165]

+   在 `ChoiceField` 的模式输出中包括 `"type"` 信息。[#7161]

+   在模式对象上包括 `"type": "object"`。[#7169]

+   不在DELETE请求的模式输出中包括组件。[#7229]

+   修复`DecimalField`的模式类型。[#7254]

+   修复`ObtainAuthToken`视图的模式生成问题。[#7211]

+   支持将`context=...`传递给视图的`.get_serializer()`方法。[#7298]

+   如果权限类已设置，则将自定义代码传递给`PermissionDenied`。[#7306]

+   在模式分页输出中包括“example”。[#7275]

+   POST请求的模式输出的默认状态码为201。[#7206]

+   在模式输出中使用camelCase作为操作ID。[#7208]

+   如果在模式输出中存在重复的操作ID，则发出警告。[#7207]

+   在将ChoiceField映射到模式输出时，改进decimal类型的处理。[#7264]

+   禁用OpenAPI模式输出的YAML别名。[#7131]

+   修复包含在命名空间URL下的API的动作URL名称。[#7287]

+   将jQuery版本从3.4更新到3.5。[#7313]

+   修复当序列化器字段使用`source=...`时的UniqueTogether处理。[#7143]

+   HTTP `HEAD`请求现在在ViewSet实例上正确设置`self.action`。[#7223]

+   对于不存在API模式路径的情况，返回有效的OpenAPI模式。[#7125]

+   在软件包分发中包括测试。[#7145]

+   允许类型检查器支持像`ModelSerializer[Author]`这样的注解。[#7385]

+   在使用APIClient时，请求的`Content-Type`标头不包括无效的`charset=None`部分。[#7400]

+   修复OpenAPI正则表达式中的`\Z`/`\z`标记。[#7389]

+   修复当源字段实际上是属性时，PrimaryKeyRelatedField和HyperlinkedRelatedField的问题。[#7142]

+   `Token.generate_key`现在是一个类方法。[#7502]

+   `@action`在包装在不保留信息的装饰器中的方法时发出警告，使用`@functools.wraps`。[#7098]

+   弃用`serializers.NullBooleanField`，改用`serializers.BooleanField`并设置`allow_null=True`。[#7122]

* * *

**日期**：2020年9月30日

+   **安全性**：弃用`urlize_quoted_links`模板标签，改用Django内置的`urlize`，移除了浏览器API中某些内容的XSS漏洞。

**日期**：2020年8月5日

+   与Django 3.1兼容性修复。

**日期**：2019年12月12日

+   弃用`.set_context` API，采用`requires_context`标记。[在这里](../3.11-announcement/#validator-default-context)

+   将TextField与选择框的默认小部件更改为下拉框。[#6892](https://github.com/encode/django-rest-framework/issues/6892)

+   支持在非关系字段（如JSONField）上进行嵌套写操作。[#6916](https://github.com/encode/django-rest-framework/issues/6916)

+   根据配置的解析器/渲染器，包括OpenAPI模式中的请求/响应媒体类型。[#6865](https://github.com/encode/django-rest-framework/issues/6865)

+   根据视图上的文档字符串，包括OpenAPI模式中的操作描述。[#6898](https://github.com/encode/django-rest-framework/issues/6898)

+   修复在OpenAPI模式中，序列化器包含所有可选字段的表示问题。[#6941](https://github.com/encode/django-rest-framework/issues/6941)，[#6944](https://github.com/encode/django-rest-framework/issues/6944)

+   修复在 OpenAPI 模式中 `serializers.HStoreField` 的表示。[#6914](https://github.com/encode/django-rest-framework/issues/6914)

+   在未提供标题或版本时修复 OpenAPI 生成问题。[#6912](https://github.com/encode/django-rest-framework/issues/6912)

+   在 OpenAPI 模式中对大整数使用 `int64` 表示。[#7018](https://github.com/encode/django-rest-framework/issues/7018)

+   如果字段子类未提供 `.to_representation` 实现，则改进错误消息。[#6996](https://github.com/encode/django-rest-framework/issues/6996)

+   修复使用多重继承的序列化器类的问题。[#6980](https://github.com/encode/django-rest-framework/issues/6980)

+   修复反转具有路径中百分比编码组件的超链接 URL 字段的问题。[#7059](https://github.com/encode/django-rest-framework/issues/7059)

+   更新 Bootstrap 到 3.4.1。[#6923](https://github.com/encode/django-rest-framework/issues/6923)

**日期**：2019年9月4日

+   在 OpenAPI 模式生成中包含 API 版本，默认为空字符串。

+   在 OpenAPI 响应模式中添加分页属性。

+   在 OpenAPI 响应模式中添加缺失的 "description" 属性。

+   在 OpenAPI 模式中仅在非空情况下包含 "required"。

+   修复 OpenAPI 模式中 "DELETE" 案例的响应模式。

+   在列表视图响应模式中使用数组类型。

+   在 OpenAPI 操作 ID 中使用一致的 `lowerInitialCamelCase` 风格。

+   修复 OpenAPI 模式中的 `minLength`/`maxLength`/`minItems`/`maxItems` 属性。

+   在序列化中仅调用一次 `FileField.url`，以提升性能。

+   修复配置更改后可能导致节流计算错误的边缘情况。

**日期**：2019年7月29日

+   各种 `OpenAPI` 模式修复。

+   在 `include_docs_urls` 中指定 `urlconf` 的能力。

**日期**：2019年7月17日

+   不要在 `TokenAuth` 管理界面上包含自动完成字段，因为这会对自定义用户模型和管理界面强加限制。

+   要求对于 OpenAPI 模式生成使用 `uritemplate`，但不要求 `coreapi`。

**日期**：[2019年7月15日](https://github.com/encode/django-rest-framework/milestone/69?closed=1)

+   切换到 OpenAPI 模式生成。

+   放弃对 Python 2 的支持。

+   添加 `generateschema --generator_class` CLI 选项。

+   更新 PyYaml 依赖项以用于 OpenAPI 模式生成，版本为 `pyyaml>=5.1`。[#6680](https://github.com/encode/django-rest-framework/issues/6680)

+   解决使用 markdown 时的 DeprecationWarning。[#6317](https://github.com/encode/django-rest-framework/issues/6317)

+   在模板中使用 `user.get_username`，而不是 `user.username`。

+   修复对象删除后可能发生的游标分页问题。

+   修复具有 `source="*"` 的可空字段。

+   在节流检查期间始终应用所有节流类。

+   更新 jQuery 和 Markdown 依赖项。

+   不要在 `SerializerMethodField` 字段名参数上严格禁止冗余。

+   如果未经身份验证，则不要在浏览器 API 中渲染额外操作。

+   从搜索参数中剥离空字符。

+   废弃`detail_route`装饰器，推荐使用`action`，其接受一个`detail`布尔值。请使用`@action(detail=True)`代替。[gh6687](https://github.com/encode/django-rest-framework/issues/6687)

+   废弃`list_route`装饰器，推荐使用`action`，其接受一个`detail`布尔值。请使用`@action(detail=False)`代替。[gh6687](https://github.com/encode/django-rest-framework/issues/6687)

**日期**: 2019年5月10日

这是一个维护版本，修复了Python 2下的错误处理错误。

**日期**: 2019年4月29日

这是最后一个支持Python 2的Django REST Framework发布版。请务必在升级到Django REST Framework 3.10之前升级到Python 3。

+   调整了对django-guardian的兼容性检查，允许最后一个与Python 2兼容的guardian版本（v1.4.9）。[#6613](https://github.com/encode/django-rest-framework/issues/6613)

**日期**: [2019年3月3日](https://github.com/encode/django-rest-framework/milestone/71?closed=1)

+   路由器：在`register()`上使`_urls`缓存失效。[#6407](https://github.com/encode/django-rest-framework/issues/6407)

+   推迟了模式渲染器的创建，避免需要pyyaml。[#6416](https://github.com/encode/django-rest-framework/issues/6416)

+   在base.html中添加了'request_forms'块。[#6340](https://github.com/encode/django-rest-framework/issues/6340)

+   修复SchemaView在异常时重置渲染器的问题。[#6429](https://github.com/encode/django-rest-framework/issues/6429)

+   更新了Django Guardian依赖项。[#6430](https://github.com/encode/django-rest-framework/issues/6430)

+   确保对Django 2.2的支持。[#6422](https://github.com/encode/django-rest-framework/issues/6422) & [#6455](https://github.com/encode/django-rest-framework/issues/6455)

+   使模板与基于会话的CSRF兼容。[#6207](https://github.com/encode/django-rest-framework/issues/6207)

+   调整了字段`validators`以接受非列表可迭代对象。[#6282](https://github.com/encode/django-rest-framework/issues/6282)

+   添加了SearchFilter.get_search_fields()挂钩。[#6279](https://github.com/encode/django-rest-framework/issues/6279)

+   修复访问collections.abc类时的DeprecationWarning问题。[#6268](https://github.com/encode/django-rest-framework/issues/6268)

+   允许在`limit_choices_to`内省中使用Q对象。[#6472](https://github.com/encode/django-rest-framework/issues/6472)

+   添加了对组合权限的惰性评估。[#6463](https://github.com/encode/django-rest-framework/issues/6463)

+   添加否定~操作符到权限组合中。[#6361](https://github.com/encode/django-rest-framework/issues/6361)

+   避免在SearchFilter中对注释字段调用distinct。[#6240](https://github.com/encode/django-rest-framework/issues/6240)

+   引入`RemovedInDRF…Warning`类以简化弃用操作。[#6480](https://github.com/encode/django-rest-framework/issues/6480)

**日期**: [2019年1月16日](https://github.com/encode/django-rest-framework/milestone/70?closed=1)

+   解决可浏览API中的XSS问题。[#6330](https://github.com/encode/django-rest-framework/issues/6330)

+   升级Bootstrap到3.4.0以解决XSS问题。

+   解决可组合权限的问题。[#6299](https://github.com/encode/django-rest-framework/issues/6299)

+   尊重外键上的`limit_choices_to`。[#6371](https://github.com/encode/django-rest-framework/issues/6371)

**日期**: [2018年10月18日](https://github.com/encode/django-rest-framework/milestone/66?closed=1)

+   ViewSet额外操作的改进 [#5605](https://github.com/encode/django-rest-framework/issues/5605)

+   修复ViewSet后缀的`action`支持问题 [#6081](https://github.com/encode/django-rest-framework/issues/6081)

+   允许`action`文档部分 [#6060](https://github.com/encode/django-rest-framework/issues/6060)

+   弃用`Router.register`的`base_name`参数，改用`basename`。[#5990](https://github.com/encode/django-rest-framework/issues/5990)

+   弃用`Router.get_default_base_name`方法，改用`Router.get_default_basename`。[#5990](https://github.com/encode/django-rest-framework/issues/5990)

+   将`CharField`更改为不允许空字节。[#6073](https://github.com/encode/django-rest-framework/issues/6073) 要恢复旧行为，请子类化`CharField`并从验证器中删除`ProhibitNullCharactersValidator`。

+   添加`OpenAPIRenderer`和`generate_schema`管理命令。[#6229](https://github.com/encode/django-rest-framework/issues/6229)

+   默认添加OpenAPIRenderer，并添加模式文档。[#6233](https://github.com/encode/django-rest-framework/issues/6233)

+   允许权限进行组合 [#5753](https://github.com/encode/django-rest-framework/issues/5753)

+   允许在Django 2.1中使用可空BooleanField。[#6183](https://github.com/encode/django-rest-framework/issues/6183)

+   添加Python 3.7支持测试。[#6141](https://github.com/encode/django-rest-framework/issues/6141)

+   使用Django 2.1正式版本进行测试。[#6109](https://github.com/encode/django-rest-framework/issues/6109)

+   添加djangorestframework-datatables到第三方包列表 [#5931](https://github.com/encode/django-rest-framework/issues/5931)

+   更改ISO 8601日期格式以排除年/仅限月份选项 [#5936](https://github.com/encode/django-rest-framework/issues/5936)

+   将所有pypi.python.org的URL更新为pypi.org [#5942](https://github.com/encode/django-rest-framework/issues/5942)

+   确保html表单（多部分表单数据）尊重可选字段 [#5927](https://github.com/encode/django-rest-framework/issues/5927)

+   允许ErrorDetail的哈希化。[#5932](https://github.com/encode/django-rest-framework/issues/5932)

+   修正JSONField的模式解析问题 [#5878](https://github.com/encode/django-rest-framework/issues/5878)

+   使用安全的 [#5869](https://github.com/encode/django-rest-framework/issues/5869) 渲染帮助文本中的描述。

+   从 `default_error_message` 中删除输入值。[#5881](https://github.com/encode/django-rest-framework/issues/5881)。

+   在 DurationField 中添加 `min_value/max_value` 支持。[#5643](https://github.com/encode/django-rest-framework/issues/5643)。

+   修复在仅有 pk 优化尝试/异常块时实例被覆盖的 AttributeError。[#5747](https://github.com/encode/django-rest-framework/issues/5747)。

+   当值为 None 时修复 items 过滤器中的 AttributeError。[#5981](https://github.com/encode/django-rest-framework/issues/5981)。

+   修复 Javascript `e.indexOf` 不是函数的错误。[#5982](https://github.com/encode/django-rest-framework/issues/5982)。

+   修复额外动作的模式。[#5992](https://github.com/encode/django-rest-framework/issues/5992)。

+   改进 `get_error_detail` 以使用 `error_dict/error_list` [#5785](https://github.com/encode/django-rest-framework/issues/5785)。

+   在管理渲染器中改进 URL。[#5988](https://github.com/encode/django-rest-framework/issues/5988)。

+   在文档中添加 "Community" 部分，进行小的清理。[#5993](https://github.com/encode/django-rest-framework/issues/5993)。

+   将 Guardian 的导入移出兼容性模块 [#6054](https://github.com/encode/django-rest-framework/issues/6054)。

+   废弃 `DjangoObjectPermissionsFilter` 类，已移至 `djangorestframework-guardian` 包中。[#6075](https://github.com/encode/django-rest-framework/issues/6075)。

+   放弃对 Django 1.10 的支持 [#5657](https://github.com/encode/django-rest-framework/issues/5657)。

+   仅捕获对象查找时的 TypeError/ValueError [#6028](https://github.com/encode/django-rest-framework/issues/6028)。

+   处理在 ModelSerializer 中没有 `.objects` 管理器的模型。[#6111](https://github.com/encode/django-rest-framework/issues/6111)。

+   改进 ModelSerializer.create() 错误消息。[#6112](https://github.com/encode/django-rest-framework/issues/6112)。

+   使用 Django 1.11.6+ 的会话认证时修复 CSRF cookie 检查失败。[#6113](https://github.com/encode/django-rest-framework/issues/6113)。

+   更新 JWT 文档。[#6138](https://github.com/encode/django-rest-framework/issues/6138)。

+   修复未传递到 `urlize_quoted_links` 过滤器的自动转义问题。[#6191](https://github.com/encode/django-rest-framework/issues/6191)。

**日期**: [2018年4月6日](https://github.com/encode/django-rest-framework/milestone/68?closed=1)。

+   修复 `read_only` + `default` `unique_together` 验证问题。[#5922](https://github.com/encode/django-rest-framework/issues/5922)。

+   从 `rest_framework.compat` 中导入 `coreapi` 而不是直接导入。[#5921](https://github.com/encode/django-rest-framework/issues/5921)。

+   文档: 在 Route 中添加丢失的参数 'detail'。[#5920](https://github.com/encode/django-rest-framework/issues/5920)。

**日期**: [2018年4月4日](https://github.com/encode/django-rest-framework/milestone/67?closed=1)。

+   在路由装饰器中使用旧的 `url_name` 行为。[#5915](https://github.com/encode/django-rest-framework/issues/5915)。

    对于`list_route`和`detail_route`，保持了基于`url_path`而不是函数名的旧行为。

**日期**: [2018年4月3日](https://github.com/encode/django-rest-framework/milestone/61?closed=1)

+   **重大变更**：修改了`read_only`加`default`的行为。[#5886](https://github.com/encode/django-rest-framework/issues/5886)

    `read_only`字段现在**总是**会从可写字段中排除。

    以前具有`default`值的`read_only`字段将在创建和更新操作中使用`default`。

    为了保持旧行为，在调用视图中的`save()`时，可能需要传递`read_only`字段的值：

    ```
    def perform_create(self, serializer):
        serializer.save(owner=self.request.user) 
    ```

    或者你可以根据需要重写`save()`、`create()`或`update()`。

+   在`required=False`时，修正了`allow_null`行为。[#5888](https://github.com/encode/django-rest-framework/issues/5888)

    没有显式`default`的情况下，`allow_null`意味着对出站序列化的默认值为`null`。以前这些字段在只读或其他情况下不需要时被跳过。

    **可能存在向后兼容性问题**，如果您依赖于这些字段被排除在出站表示中。为了恢复旧行为，您可以在`data`中排除字段为`None`时。

    例如：

    ```
    @property
    def data(self):
        """
        Drop `maybe_none` field if None.
        """
        data = super().data
        if 'maybe_none' in data and data['maybe_none'] is None:
            del data['maybe_none']
        return data 
    ```

+   重构动态路由生成并改进视图集操作的可审视性。[#5705](https://github.com/encode/django-rest-framework/issues/5705)

    `ViewSet`已提供新属性和方法，允许其审视其动作集和当前动作的详细信息。

    +   将`list_route`和`detail_route`合并为单个`action`装饰器。

    +   使用`.get_extra_actions()`获取`ViewSet`上的所有额外动作。

    +   额外的操作现在会在装饰的方法上设置`url_name`和`url_path`。

    +   `url_name`现在基于函数名，而不是`url_path`，因为路径并不总是合适（例如，在路径中捕获参数）。

    +   通过`.reverse_action()`方法启用动作URL反向。

    +   示例反向调用：`self.reverse_action(self.custom_action.url_name)`

    +   添加`detail`初始化关键字参数，指示当前操作是在集合还是单个实例上操作。

    其他更改：

    +   弃用`list_route`和`detail_route`，改用带有`detail`布尔值的`action`装饰器。

    +   弃用动态列表/详细路由变体，改用带有`detail`布尔值的`DynamicRoute`。

    +   重构了路由器的动态路由生成。

    +   对于`list_route`和`detail_route`，保持了基于`url_path`而不是函数名的旧行为。

+   修复了3.7.4版本说明的格式错误。[#5704](https://github.com/encode/django-rest-framework/issues/5704)

+   文档：更新了DRF可写嵌套序列化器的引用。[#5711](https://github.com/encode/django-rest-framework/issues/5711)

+   文档：修复了授权URL示例中的拼写错误。[#5713](https://github.com/encode/django-rest-framework/issues/5713)

+   改进复合字段子错误 [#5655](https://github.com/encode/django-rest-framework/issues/5655)

+   禁用字典/列表字段的 HTML 输入 [#5702](https://github.com/encode/django-rest-framework/issues/5702)

+   在 HostNameVersioning 文档中修正拼写错误 [#5709](https://github.com/encode/django-rest-framework/issues/5709)

+   使用 rsplit 获取导入的模块和类名 [#5712](https://github.com/encode/django-rest-framework/issues/5712)

+   正式化 URLPatternsTestCase [#5703](https://github.com/encode/django-rest-framework/issues/5703)

+   添加异常翻译测试 [#5700](https://github.com/encode/django-rest-framework/issues/5700)

+   测试静态文件 [#5701](https://github.com/encode/django-rest-framework/issues/5701)

+   在文档和模式的第三方包中添加 drf-yasg [#5720](https://github.com/encode/django-rest-framework/issues/5720)

+   删除未使用的 `compat._resolve_model()` [#5733](https://github.com/encode/django-rest-framework/issues/5733)

+   放弃对不支持的 Python 3.2 的 compat 兼容性解决方案 [#5734](https://github.com/encode/django-rest-framework/issues/5734)

+   更喜欢使用 `iter(dict)` 而不是 `iter(dict.keys())` [#5736](https://github.com/encode/django-rest-framework/issues/5736)

+   向 setuptools 传递 `python_requires` 参数 [#5739](https://github.com/encode/django-rest-framework/issues/5739)

+   从文档中删除未使用的链接 [#5735](https://github.com/encode/django-rest-framework/issues/5735)

+   在文档中尽可能使用 https 协议的链接 [#5729](https://github.com/encode/django-rest-framework/issues/5729)

+   添加 HStoreField，postgres 字段测试 [#5654](https://github.com/encode/django-rest-framework/issues/5654)

+   文档中始终完全限定 ValidationError [#5751](https://github.com/encode/django-rest-framework/issues/5751)

+   从 ManualSchema 中删除不可达代码 [#5766](https://github.com/encode/django-rest-framework/issues/5766)

+   允许定制 API 文档中的代码示例 [#5752](https://github.com/encode/django-rest-framework/issues/5752)

+   更新文档以使用 `pip show` [#5757](https://github.com/encode/django-rest-framework/issues/5757)

+   在模板中加载 'static' 而不是 'staticfiles' [#5773](https://github.com/encode/django-rest-framework/issues/5773)

+   修正 `fields` 文档中的拼写错误 [#5783](https://github.com/encode/django-rest-framework/issues/5783)

+   在文档中引用 "NamespaceVersioning" 而不是 "NamespacedVersioning" [#5754](https://github.com/encode/django-rest-framework/issues/5754)

+   ErrorDetail：添加 `__eq__`/`__ne__` 和 `__repr__` [#5787](https://github.com/encode/django-rest-framework/issues/5787)

+   在文档中替换 `background-attachment: fixed` [#5777](https://github.com/encode/django-rest-framework/issues/5777)

+   使 `exceptions.APIException` 输出的 404 和 403 响应一致 [#5763](https://github.com/encode/django-rest-framework/issues/5763)

+   API 文档的小修复：schemas [#5796](https://github.com/encode/django-rest-framework/issues/5796)

+   修复了 PrimaryKeyRelatedField 的模式生成。[#5764](https://github.com/encode/django-rest-framework/issues/5764)

+   将序列化器 DictField 表示为模式中的对象。[#5765](https://github.com/encode/django-rest-framework/issues/5765)

+   添加了重新实现 ObtainAuthToken 的文档示例。[#5802](https://github.com/encode/django-rest-framework/issues/5802)

+   向 ObtainAuthToken 视图添加模式。[#5676](https://github.com/encode/django-rest-framework/issues/5676)

+   修复了请求表单数据处理。[#5800](https://github.com/encode/django-rest-framework/issues/5800)

+   修复了 authtoken 视图导入。[#5818](https://github.com/encode/django-rest-framework/issues/5818)

+   更新了 pytest、isort。[#5815](https://github.com/encode/django-rest-framework/issues/5815) [#5817](https://github.com/encode/django-rest-framework/issues/5817) [#5894](https://github.com/encode/django-rest-framework/issues/5894)

+   修复了非 ISO8601 日期时间的活动时区处理。[#5833](https://github.com/encode/django-rest-framework/issues/5833)

+   当值为 `0` 时，使 TemplateHTMLRenderer 渲染 IntegerField 输入。[#5834](https://github.com/encode/django-rest-framework/issues/5834)

+   修正了教程说明中的端点。[#5835](https://github.com/encode/django-rest-framework/issues/5835)

+   将 Django Rest Framework Role Filters 添加到第三方包中。[#5809](https://github.com/encode/django-rest-framework/issues/5809)

+   使用单个静态资产副本。更新 jQuery。[#5823](https://github.com/encode/django-rest-framework/issues/5823)

+   将三元条件改为符合 PEP308 标准。[#5827](https://github.com/encode/django-rest-framework/issues/5827)

+   添加了到 '使用 React 的待办事项列表 API' 和 '博客 API' 教程的链接。[#5837](https://github.com/encode/django-rest-framework/issues/5837)

+   修复了 ModelSerializer 中的注释拼写错误。[#5844](https://github.com/encode/django-rest-framework/issues/5844)

+   将 admin 添加到已安装应用程序中，以避免测试失败。[#5870](https://github.com/encode/django-rest-framework/issues/5870)

+   修复了 SimpleMetadata 中 UUIDField 的模式。[#5872](https://github.com/encode/django-rest-framework/issues/5872)

+   修正了关于使用命名空间包含路由器的文档。[#5843](https://github.com/encode/django-rest-framework/issues/5843)

+   测试使用模型对象作为点源默认值。[#5880](https://github.com/encode/django-rest-framework/issues/5880)

+   允许遍历可空关联字段。[#5849](https://github.com/encode/django-rest-framework/issues/5849)

+   添加：教程：Django REST 与 React（Django 2.0）[#5891](https://github.com/encode/django-rest-framework/issues/5891)

+   添加 `LimitOffsetPagination.get_count` 以允许方法覆盖。[#5846](https://github.com/encode/django-rest-framework/issues/5846)

+   在元数据中不显示隐藏字段。[#5854](https://github.com/encode/django-rest-framework/issues/5854)

+   启用 OrderingFilter 处理 'ordering' 字段的空元组（或列表）。[#5899](https://github.com/encode/django-rest-framework/issues/5899)

+   添加了通用的 500 和 400 JSON 错误处理程序。 [#5904](https://github.com/encode/django-rest-framework/issues/5904)

**日期**：[2017年12月21日](https://github.com/encode/django-rest-framework/milestone/65?closed=1)

+   修复拼写错误以包含 *.mo 语言环境文件到打包中。 [#5697](https://github.com/encode/django-rest-framework/issues/5697), [#5695](https://github.com/encode/django-rest-framework/issues/5695)

**日期**：[2017年12月21日](https://github.com/encode/django-rest-framework/milestone/64?closed=1)

+   在打包中添加缺失的 *.ico 图标文件。

**日期**：[2017年12月21日](https://github.com/encode/django-rest-framework/milestone/63?closed=1)

+   在打包中添加缺失的 *.woff2 字体文件 [#5692](https://github.com/encode/django-rest-framework/issues/5692)

+   在打包中添加缺失的 *.mo 语言环境文件 [#5695](https://github.com/encode/django-rest-framework/issues/5695), [#5696](https://github.com/encode/django-rest-framework/issues/5696)

**日期**：[2017年12月20日](https://github.com/encode/django-rest-framework/milestone/62?closed=1)

+   Schema：提取 `manual_fields` 处理的方法 [#5633](https://github.com/encode/django-rest-framework/issues/5633)

    允许更轻松地定制 `manual_fields` 处理，例如提供每个方法的手动字段。 `AutoSchema` 添加了 `get_manual_fields` 作为预期的重写点，并添加了一个实用方法 `update_fields`，用于处理来自列表的按名称字段替换，通常不希望重写此方法。

    注意：`AutoSchema.__init__` 现在确保 `manual_fields` 是一个列表。之前可能内部存储为 `None`。

+   移除 ulrparse 兼容性接口；改用 six 替代 [#5579](https://github.com/encode/django-rest-framework/issues/5579)

+   删除对 `TimeDelta.total_seconds()` 的兼容包装 [#5577](https://github.com/encode/django-rest-framework/issues/5577)

+   整理项目中的所有空白字符 [#5578](https://github.com/encode/django-rest-framework/issues/5578)

+   兼容性清理 [#5581](https://github.com/encode/django-rest-framework/issues/5581)

+   在浏览 API 视图中添加 pygments CSS 块 [#5584](https://github.com/encode/django-rest-framework/issues/5584) [#5587](https://github.com/encode/django-rest-framework/issues/5587)

+   从兼容性中移除 `set_rollback()` 方法 [#5591](https://github.com/encode/django-rest-framework/issues/5591)

+   修复请求体/POST 访问问题 [#5590](https://github.com/encode/django-rest-framework/issues/5590)

+   重命名测试以引用正确的问题 [#5610](https://github.com/encode/django-rest-framework/issues/5610)

+   文档修复 [#5611](https://github.com/encode/django-rest-framework/issues/5611) [#5612](https://github.com/encode/django-rest-framework/issues/5612)

+   删除文档和代码中对不支持的 Django 版本的引用 [#5602](https://github.com/encode/django-rest-framework/issues/5602)

+   测试 Serializer 排除已声明字段 [#5599](https://github.com/encode/django-rest-framework/issues/5599)

+   修复筛选后端的模式生成问题 [#5613](https://github.com/encode/django-rest-framework/issues/5613)

+   ModelSerializer 测试的轻微清理 [#5598](https://github.com/encode/django-rest-framework/issues/5598)

+   使用 `__getattr__` 重新实现请求属性访问 [#5617](https://github.com/encode/django-rest-framework/issues/5617)

+   修复 SchemaJSRenderer 渲染无效的 Javascript [#5607](https://github.com/encode/django-rest-framework/issues/5607)

+   正式/明确支持 Django 2.0 [#5619](https://github.com/encode/django-rest-framework/issues/5619)

+   对传入的请求参数进行类型检查 [#5618](https://github.com/encode/django-rest-framework/issues/5618)

+   修复请求认证器上的 AttributeError 隐藏问题 [#5600](https://github.com/encode/django-rest-framework/issues/5600)

+   更新测试要求 [#5626](https://github.com/encode/django-rest-framework/issues/5626)

+   文档：`Serializer._declared_fields` 允许修改序列化器上的字段 [#5629](https://github.com/encode/django-rest-framework/issues/5629)

+   修复打包问题 [#5624](https://github.com/encode/django-rest-framework/issues/5624)

+   修复 PyPI 的 readme 渲染问题，将 readme 构建加入 CI [#5625](https://github.com/encode/django-rest-framework/issues/5625)

+   更新教程内容 [#5622](https://github.com/encode/django-rest-framework/issues/5622)

+   使用 `allow_null=True` 的非必需字段不应暗示默认值 [#5639](https://github.com/encode/django-rest-framework/issues/5639)

+   文档：添加 `allow_null` 序列化输出说明 [#5641](https://github.com/encode/django-rest-framework/issues/5641)

+   在 tox.ini 中更新使用 Django 2.0 版本 [#5645](https://github.com/encode/django-rest-framework/issues/5645)

+   修复在提供无效 `data` 时，Browsable API 渲染 `Serializer.data` 的问题 [#5646](https://github.com/encode/django-rest-framework/issues/5646)

+   文档：注意 AutoSchema 在裸 APIView 上的限制 [#5649](https://github.com/encode/django-rest-framework/issues/5649)

+   将 `.basename` 和 `.reverse_action()` 添加到 ViewSet 中 [#5648](https://github.com/encode/django-rest-framework/issues/5648)

+   文档：修正序列化器文档中的拼写错误 [#5652](https://github.com/encode/django-rest-framework/issues/5652)

+   修复 `override_settings` 的兼容性问题 [#5668](https://github.com/encode/django-rest-framework/issues/5668)

+   添加 DEFAULT_SCHEMA_CLASS 设置项 [#5658](https://github.com/encode/django-rest-framework/issues/5658)

+   添加文档说明，生成的 BooleanField 为 `required=False` [#5665](https://github.com/encode/django-rest-framework/issues/5665)

+   添加 'dist' 构建 [#5656](https://github.com/encode/django-rest-framework/issues/5656)

+   修复文档字符串中的拼写错误 [#5678](https://github.com/encode/django-rest-framework/issues/5678)

+   文档：添加 `UNAUTHENTICATED_USER = None` 注释 [#5679](https://github.com/encode/django-rest-framework/issues/5679)

+   更新“文档您的 API”中的 OPTIONS 示例 [#5680](https://github.com/encode/django-rest-framework/issues/5680)

+   在 FBV 中添加对象权限说明。[#5681](https://github.com/encode/django-rest-framework/issues/5681)

+   在 `to_representation` 文档中添加示例。[#5682](https://github.com/encode/django-rest-framework/issues/5682)

+   在文档中添加 Classy DRF 的链接。[#5683](https://github.com/encode/django-rest-framework/issues/5683)

+   文档化 ViewSet.action。[#5685](https://github.com/encode/django-rest-framework/issues/5685)

+   修复模式文档中的拼写错误。[#5687](https://github.com/encode/django-rest-framework/issues/5687)

+   修复模式生成中的 URL 模式解析问题。[#5689](https://github.com/encode/django-rest-framework/issues/5689)

+   在自定义字段文档中添加使用 `source='*'` 的示例。[#5688](https://github.com/encode/django-rest-framework/issues/5688)

+   修复 Django 2 中 `path()` 路由的 `format_suffix_patterns` 行为。[#5691](https://github.com/encode/django-rest-framework/issues/5691)

**日期**：[2017年11月6日](https://github.com/encode/django-rest-framework/milestone/60?closed=1)

+   修复 contrib.auth 视图导入中的 `AppRegistryNotReady` 错误。[#5567](https://github.com/encode/django-rest-framework/issues/5567)

**日期**：[2017年11月6日](https://github.com/encode/django-rest-framework/milestone/59?closed=1)

+   由于移除了 django.contrib.auth.login()/logout() 视图，修复 Django 2.1 兼容性问题。[#5510](https://github.com/encode/django-rest-framework/issues/5510)

+   添加缺失的 TextLexer 导入。[#5512](https://github.com/encode/django-rest-framework/issues/5512)

+   添加缓存示例和文档。[#5514](https://github.com/encode/django-rest-framework/issues/5514)

+   包含日期和日期时间格式以进行模式生成。[#5511](https://github.com/encode/django-rest-framework/issues/5511)

+   使用三个反引号表示 Markdown 代码块。[#5513](https://github.com/encode/django-rest-framework/issues/5513)

+   交互式文档 - 使底部侧边栏项目固定。[#5516](https://github.com/encode/django-rest-framework/issues/5516)

+   澄清分页系统检查。[#5524](https://github.com/encode/django-rest-framework/issues/5524)

+   修复对无效 JSON 的 JSONBoundField 混淆。[#5527](https://github.com/encode/django-rest-framework/issues/5527)

+   在浏览 API 中使 JSONField 渲染为文本区域。[#5530](https://github.com/encode/django-rest-framework/issues/5530)

+   模式：排除 ViewSet 操作的 OPTIONS/HEAD。[#5532](https://github.com/encode/django-rest-framework/issues/5532)

+   修复点源的排序问题。[#5533](https://github.com/encode/django-rest-framework/issues/5533)

+   修复带有 `allow_null=True` 的字段应该暗示默认序列化值。[#5518](https://github.com/encode/django-rest-framework/issues/5518)

+   确保 Location 头部严格为 'str'，而不是子类。[#5544](https://github.com/encode/django-rest-framework/issues/5544)

+   在 api-guide/parsers 示例中添加导入。[#5547](https://github.com/encode/django-rest-framework/issues/5547)

+   捕获"超出范围"日期时间的OverflowError异常 [#5546](https://github.com/encode/django-rest-framework/issues/5546)

+   将djangorestframework-rapidjson添加到第三方包中 [#5549](https://github.com/encode/django-rest-framework/issues/5549)

+   为`drf_create_token`命令增加测试覆盖率 [#5550](https://github.com/encode/django-rest-framework/issues/5550)

+   添加Python 3.6支持的Trove分类器。 [#5555](https://github.com/encode/django-rest-framework/issues/5555)

+   在Travis CI配置中添加pip缓存支持 [#5556](https://github.com/encode/django-rest-framework/issues/5556)

+   将[`wheel`]部分重命名为[`bdist_wheel`]，因为前者已经过时 [#5557](https://github.com/encode/django-rest-framework/issues/5557)

+   修复无效转义序列弃用警告 [#5560](https://github.com/encode/django-rest-framework/issues/5560)

+   添加交互式文档错误模板 [#5548](https://github.com/encode/django-rest-framework/issues/5548)

+   在DecimalField中添加四舍五入参数 [#5562](https://github.com/encode/django-rest-framework/issues/5562)

+   修复测试中捕获的所有BytesWarning警告 [#5561](https://github.com/encode/django-rest-framework/issues/5561)

+   使用字典和集合文字而不是调用dict()和set() [#5559](https://github.com/encode/django-rest-framework/issues/5559)

+   更改ImageField验证模式，使用来自DjangoImageField的验证器 [#5539](https://github.com/encode/django-rest-framework/issues/5539)

+   修复Python 2中在query_string中处理Unicode符号的问题 [#5552](https://github.com/encode/django-rest-framework/issues/5552)

**日期**: [2017年10月16日](https://github.com/encode/django-rest-framework/milestone/58?closed=1)

+   修复交互式文档始终在请求中对布尔字段使用false的问题 [#5492](https://github.com/encode/django-rest-framework/issues/5492)

+   改善与Django 2.0 alpha的兼容性。 [#5500](https://github.com/encode/django-rest-framework/issues/5500) [#5503](https://github.com/encode/django-rest-framework/issues/5503)

+   改进模式命名冲突的处理 [#5486](https://github.com/encode/django-rest-framework/issues/5486)

+   添加有关为点分`source`字段提供默认值的额外文档和测试 [#5489](https://github.com/encode/django-rest-framework/issues/5489)

**日期**: [2017年10月6日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.7.0+Release%22)

+   修复`DjangoModelPermissions`确保在调用视图的`get_queryset()`方法之前对用户进行身份验证的问题。作为副作用，这改变了HTTP方法权限和身份验证检查的顺序，只有在经过身份验证时才会返回405响应。如果要复制旧行为，请参阅有关详细信息的PR。 [#5376](https://github.com/encode/django-rest-framework/issues/5376)

+   弃用了在`APIView`和`api_view`装饰器上的`exclude_from_schema`。根据需要设置`schema = None`或`@schema(None)`。[#5422](https://github.com/encode/django-rest-framework/issues/5422)

+   时区感知的`DateTimeField`在序列化期间现在尊重活动或默认的`timezone`，而不是总是使用UTC。[#5435](https://github.com/encode/django-rest-framework/issues/5435)

    解决了实例在`create`时使用提供的日期时间但在`retrieve`时使用UTC时的不一致性。[#3732](https://github.com/encode/django-rest-framework/issues/3732)

    **可能会破坏向后兼容性**，如果您依赖于日期时间字符串为UTC。如果需要，客户端解释日期时间或[设置默认或活动时区（文档）](https://docs.djangoproject.com/en/1.11/topics/i18n/timezones/#default-time-zone-and-current-time-zone)为UTC。

+   根据弃用策略删除了`DjangoFilterBackend`。改用`django_filters.rest_framework.FilterSet`和/或`django_filters.rest_framework.DjangoFilterBackend`。[#5273](https://github.com/encode/django-rest-framework/issues/5273)

+   在编码时不要从`time`中去除微秒。使其与`datetime`保持一致。 **BC变更**：先前只编码毫秒。[#5440](https://github.com/encode/django-rest-framework/issues/5440)

+   添加了`STRICT_JSON`设置（默认为`True`），以引发Python的`json`模块接受的扩展浮点值（`nan`、`inf`、`-inf`）的异常。 **BC变更**：以前这些值会被转换为相应的字符串。将`STRICT_JSON`设置为`False`以恢复先前的行为。[#5265](https://github.com/encode/django-rest-framework/issues/5265)

+   在CursorPaginator类中添加对`page_size`参数的支持。[#5250](https://github.com/encode/django-rest-framework/issues/5250)

+   将`DEFAULT_PAGINATION_CLASS`默认为`None`。 **BC变更**：如果您仅仅是设置`PAGE_SIZE`以启用分页，您需要添加`DEFAULT_PAGINATION_CLASS`。先前的默认值为`rest_framework.pagination.PageNumberPagination`。有一个系统检查警告来捕获此情况。如果您是按视图设置分页类，可以静音该警告。[#5170](https://github.com/encode/django-rest-framework/issues/5170)

+   在模式生成中捕获来自`get_serializer_fields`的`APIException`。[#5443](https://github.com/encode/django-rest-framework/issues/5443)

+   在使用`include_docs_urls`时允许自定义身份验证和权限类。[#5448](https://github.com/encode/django-rest-framework/issues/5448)

+   在验证器上推迟翻译字符串的评估。[#5452](https://github.com/encode/django-rest-framework/issues/5452)

+   在`ValidationError`异常中为'detail'参数添加了默认值。[#5342](https://github.com/encode/django-rest-framework/issues/5342)

+   调整模式获取过滤字段规则以匹配框架。[#5454](https://github.com/encode/django-rest-framework/issues/5454)

+   更新测试矩阵以添加 Django 2.0 并移除 Django 1.8 和 1.9 **BC Change**: 这移除了 Django 1.8 和 Django 1.9 从 Django REST Framework 支持的版本中。[#5457](https://github.com/encode/django-rest-framework/issues/5457)

+   修复 serializers.ModelField 中的弃用警告 [#5058](https://github.com/encode/django-rest-framework/issues/5058)

+   当 `get_queryset` 返回 `None` 时添加更明确的错误消息 [#5348](https://github.com/encode/django-rest-framework/issues/5348)

+   修复响应 `data` 描述的文档 [#5361](https://github.com/encode/django-rest-framework/issues/5361)

+   修复打包时的 **pycache**/.pyc 排除 [#5373](https://github.com/encode/django-rest-framework/issues/5373)

+   修复点源的默认值处理 [#5375](https://github.com/encode/django-rest-framework/issues/5375)

+   在将空主体传递给 RequestFactory 时确保设置 content_type [#5351](https://github.com/encode/django-rest-framework/issues/5351)

+   修复 ErrorDetail 文档 [#5380](https://github.com/encode/django-rest-framework/issues/5380)

+   允许在通用内容表单中包含可选内容 [#5372](https://github.com/encode/django-rest-framework/issues/5372)

+   更新 NullBooleanField 的支持值 [#5387](https://github.com/encode/django-rest-framework/issues/5387)

+   修复 ModelSerializer 上带有 source 的自定义命名字段 [#5388](https://github.com/encode/django-rest-framework/issues/5388)

+   修复 MultipleFieldLookupMixin 文档示例以正确检查对象级权限 [#5398](https://github.com/encode/django-rest-framework/issues/5398)

+   在 permissions.md 中更新 get_object() 示例 [#5401](https://github.com/encode/django-rest-framework/issues/5401)

+   修复 authtoken 管理命令 [#5415](https://github.com/encode/django-rest-framework/issues/5415)

+   修复模式生成的 markdown [#5421](https://github.com/encode/django-rest-framework/issues/5421)

+   允许动态设置 `ChoiceField.choices` [#5426](https://github.com/encode/django-rest-framework/issues/5426)

+   将项目布局添加到快速入门指南 [#5434](https://github.com/encode/django-rest-framework/issues/5434)

+   在 'render_markdown' 模板标签中重用 'apply_markdown' 函数 [#5469](https://github.com/encode/django-rest-framework/issues/5469)

+   在文档中添加到 `drf-openapi` 包的链接 [#5470](https://github.com/encode/django-rest-framework/issues/5470)

+   使用 pygments 添加代码高亮的文档字符串 [#5462](https://github.com/encode/django-rest-framework/issues/5462)

+   修复名称为 `data` 的视图的文档渲染问题 [#5472](https://github.com/encode/django-rest-framework/issues/5472)

+   文档：澄清了 'to_internal_value()' 的验证行为 [#5466](https://github.com/encode/django-rest-framework/issues/5466)

+   修复 APIException 中缺少的 six.text_type() 调用的 **str** [#5476](https://github.com/encode/django-rest-framework/issues/5476)

+   文档化 documentation.py [#5478](https://github.com/encode/django-rest-framework/issues/5478)

+   修复模式生成中的命名冲突。 [#5464](https://github.com/encode/django-rest-framework/issues/5464)

+   使用请求对象调用 Django 的 authenticate 函数。 [#5295](https://github.com/encode/django-rest-framework/issues/5295)

+   更新 coreapi JS 到 0.1.1 版本。 [#5479](https://github.com/encode/django-rest-framework/issues/5479)

+   使 `is_list_view` 可以识别 RetrieveModel… 视图。 [#5480](https://github.com/encode/django-rest-framework/issues/5480)

+   移除 Django 1.8 和 1.9 兼容性代码。 [#5481](https://github.com/encode/django-rest-framework/issues/5481)

+   移除默认路由器中已弃用的模式代码 [#5482](https://github.com/encode/django-rest-framework/issues/5482)

+   重构模式生成以允许每个视图的定制。 **BC 变更**：`SchemaGenerator.get_serializer_fields` 已重构为 `AutoSchema.get_serializer_fields` 并删除 `view` 参数 [#5354][gh5354]

**日期**：[2017年8月21日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.4+Release%22)

+   忽略任何无效格式的查询参数以供 OrderingFilter 使用。 [#5131](https://github.com/encode/django-rest-framework/issues/5131)

+   改善读取大型 JSON 请求时的内存占用。 [#5147](https://github.com/encode/django-rest-framework/issues/5147)

+   修复分页的模式生成。 [#5161](https://github.com/encode/django-rest-framework/issues/5161)

+   修复当 `HTML_CUTOFF` 设置为 `None` 时的异常。 [#5174](https://github.com/encode/django-rest-framework/issues/5174)

+   修复可浏览 API 未正确支持 `multipart/form-data`。 [#5176](https://github.com/encode/django-rest-framework/issues/5176)

+   修复 `test_hyperlinked_related_lookup_url_encoded_exists`。 [#5179](https://github.com/encode/django-rest-framework/issues/5179)

+   确保 `max_length` 在 FileField kwargs 中。 [#5186](https://github.com/encode/django-rest-framework/issues/5186)

+   修复 `url_path` 中包含大括号的 `list_route` 和 `detail_route` 特例。 [#5187](https://github.com/encode/django-rest-framework/issues/5187)

+   添加 Django 管理命令以创建 DRF 用户 Token。 [#5188](https://github.com/encode/django-rest-framework/issues/5188)

+   确保 API 文档模板不检查用户身份验证。 [#5162](https://github.com/encode/django-rest-framework/issues/5162)

+   修复 OneToOneField 也是主键的特殊情况。 [#5192](https://github.com/encode/django-rest-framework/issues/5192)

+   在 base.html 中为可访问性目的添加 aria-label 和新区域。 [#5196](https://github.com/encode/django-rest-framework/issues/5196)

+   在 api.js 中引用嵌套的 API 参数。 [#5214](https://github.com/encode/django-rest-framework/issues/5214)

+   在调度之前设置 ViewSet 的 args/kwargs/request。 [#5229](https://github.com/encode/django-rest-framework/issues/5229)

+   为 SlugField 添加 Unicode 支持。 [#5231](https://github.com/encode/django-rest-framework/issues/5231)

+   修复`HiddenField`在原始数据表单初始内容中的显示。[#5259](https://github.com/encode/django-rest-framework/issues/5259)

+   在无效时区解析上引发验证错误。[#5261](https://github.com/encode/django-rest-framework/issues/5261)

+   修复`SearchFilter`的多对多行为/性能问题。[#5264](https://github.com/encode/django-rest-framework/issues/5264)

+   简化链式比较和小代码修复。[#5276](https://github.com/encode/django-rest-framework/issues/5276)

+   `RemoteUserAuthentication`，文档和测试。[#5306](https://github.com/encode/django-rest-framework/issues/5306)

+   撤销"缓存字段的根和上下文属性" [#5313](https://github.com/encode/django-rest-framework/issues/5313)

+   修复模式中列表字段的反射。[#5326](https://github.com/encode/django-rest-framework/issues/5326)

+   修复多个嵌套和额外方法的交互式文档。[#5334](https://github.com/encode/django-rest-framework/issues/5334)

+   修复/删除未定义的模板变量"schema" [#5346](https://github.com/encode/django-rest-framework/issues/5346)

**日期**: [2017年5月12日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.3+Release%22)

+   如果URL查找导致`ValidationError`，则引发404错误。([#5126](https://github.com/encode/django-rest-framework/issues/5126))

+   在生成API模式时，根据类视图中的`http_method_names`进行处理。([#5085](https://github.com/encode/django-rest-framework/issues/5085))

+   允许在`LimitOffsetPagination`中重写`get_limit`方法以返回所有记录。([#4437](https://github.com/encode/django-rest-framework/issues/4437))

+   修复`ListSerializer`的部分更新。([#4222](https://github.com/encode/django-rest-framework/issues/4222))

+   在可浏览API中正确渲染`JSONField`控件。([#4999](https://github.com/encode/django-rest-framework/issues/4999), [#5042](https://github.com/encode/django-rest-framework/issues/5042))

+   在给定时区中，对无效的日期时间解析引发验证错误。([#4987](https://github.com/encode/django-rest-framework/issues/4987))

+   支持将文档和模式快捷方式限制为部分URL。([#4979](https://github.com/encode/django-rest-framework/issues/4979))

+   解决`SchemaGenerator`与没有`page_size`属性的分页器错误。([#5086](https://github.com/encode/django-rest-framework/issues/5086), [#3692](https://github.com/encode/django-rest-framework/issues/3692))

+   解决在字符串中使用%20而不是空格引起的`HyperlinkedRelatedField`异常。([#4748](https://github.com/encode/django-rest-framework/issues/4748), [#5078](https://github.com/encode/django-rest-framework/issues/5078))

+   可定制的模式生成器类。([#5082](https://github.com/encode/django-rest-framework/issues/5082))

+   更新现有的vary头部以响应而不是覆盖它们。([#5047](https://github.com/encode/django-rest-framework/issues/5047))

+   支持将`.as_view()`传递给视图实例。([#5053](https://github.com/encode/django-rest-framework/issues/5053))

+   当视图上的设置被覆盖时，使用正确的异常处理程序。 ([#5055](https://github.com/encode/django-rest-framework/issues/5055), [#5054](https://github.com/encode/django-rest-framework/issues/5054))

+   更新布尔字段以支持 'yes' 和 'no' 值。 ([#5038](https://github.com/encode/django-rest-framework/issues/5038))

+   修复 ChoiceField 的唯一验证器。 ([#5004](https://github.com/encode/django-rest-framework/issues/5004), [#5026](https://github.com/encode/django-rest-framework/issues/5026), [#5028](https://github.com/encode/django-rest-framework/issues/5028))

+   API 文档中的 JavaScript 清理。 ([#5001](https://github.com/encode/django-rest-framework/issues/5001))

+   在 API 架构中包括 URL 路径正则表达式（如果有效）。 ([#5014](https://github.com/encode/django-rest-framework/issues/5014))

+   在 coreapi TokenAuthentication 中正确设置方案。 ([#5000](https://github.com/encode/django-rest-framework/issues/5000), [#4994](https://github.com/encode/django-rest-framework/issues/4994))

+   ViewSets 上的 HEAD 请求不应返回 405\. ([#4705](https://github.com/encode/django-rest-framework/issues/4705), [#4973](https://github.com/encode/django-rest-framework/issues/4973), [#4864](https://github.com/encode/django-rest-framework/issues/4864))

+   支持在 `extra_kwargs` 中使用 'source'。 ([#4688](https://github.com/encode/django-rest-framework/issues/4688))

+   修复 schema.js 的无效内容类型。 ([#4968](https://github.com/encode/django-rest-framework/issues/4968))

+   修复 DjangoFilterBackend 的继承问题。 ([#5089](https://github.com/encode/django-rest-framework/issues/5089), [#5117](https://github.com/encode/django-rest-framework/issues/5117))

**日期**: [2017年3月10日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.2+Release%22)

+   API 文档中对 Safari 和 IE 的支持。 ([#4959](https://github.com/encode/django-rest-framework/issues/4959), [#4961](https://github.com/encode/django-rest-framework/issues/4961))

+   在 API 文档模板标签中添加丢失的 `mark_safe`。 ([#4952](https://github.com/encode/django-rest-framework/issues/4952), [#4953](https://github.com/encode/django-rest-framework/issues/4953))

+   添加丢失的 glyphicon 字体。 ([#4950](https://github.com/encode/django-rest-framework/issues/4950), [#4951](https://github.com/encode/django-rest-framework/issues/4951))

+   修复 API 文档中的一对一字段。 ([#4955](https://github.com/encode/django-rest-framework/issues/4955), [#4956](https://github.com/encode/django-rest-framework/issues/4956))

+   清理测试。 ([#4949](https://github.com/encode/django-rest-framework/issues/4949))

**日期**: [2017年3月9日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.1+Release%22)

+   确保 `markdown` 依赖是可选的。 ([#4947](https://github.com/encode/django-rest-framework/issues/4947))

**日期**: [2017年3月9日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.0+Release%22)

查看 [发布公告](../3.6-announcement/)。

* * *

**日期**: [2017年2月10日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.4+Release%22)

+   为ListField添加max_length和min_length参数。([#4877](https://github.com/encode/django-rest-framework/issues/4877))

+   添加每个视图自定义异常处理程序支持。([#4753](https://github.com/encode/django-rest-framework/issues/4753))

+   支持在序列化器子类上禁用声明字段。([#4764](https://github.com/encode/django-rest-framework/issues/4764))

+   在`@list_route`和`@detail_route`端点上支持自定义视图名称。([#4821](https://github.com/encode/django-rest-framework/issues/4821))

+   当使用自定义用户模型时，修正登录模板中字段的标签。([#4841](https://github.com/encode/django-rest-framework/issues/4841))

+   来自文档字符串生成的描述的空白修复。([#4759](https://github.com/encode/django-rest-framework/issues/4759), [#4869](https://github.com/encode/django-rest-framework/issues/4869), [#4870](https://github.com/encode/django-rest-framework/issues/4870))

+   当视图返回模式但没有模式渲染器时，提供更好的错误报告。([#4790](https://github.com/encode/django-rest-framework/issues/4790))

+   当使用`prefetch_related`时修复`PUT`请求的返回响应。([#4661](https://github.com/encode/django-rest-framework/issues/4661), [#4668](https://github.com/encode/django-rest-framework/issues/4668))

+   修复面包屑视图名称。([#4750](https://github.com/encode/django-rest-framework/issues/4750))

+   确保RequestsClient使用完全限定的URL。([#4678](https://github.com/encode/django-rest-framework/issues/4678))

+   修复某些情况下可写嵌套字段检查的不正确行为。([#4634](https://github.com/encode/django-rest-framework/issues/4634), [#4669](https://github.com/encode/django-rest-framework/issues/4669))

+   解决Django弃用警告。([#4712](https://github.com/encode/django-rest-framework/issues/4712))

+   各种测试用例的清理。

**日期**: [2016年11月7日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.3+Release%22)

+   不要引发不正确的FilterSet弃用警告。([#4660](https://github.com/encode/django-rest-framework/issues/4660), [#4643](https://github.com/encode/django-rest-framework/issues/4643), [#4644](https://github.com/encode/django-rest-framework/issues/4644))

+   当视图权限类不存在时，模式生成不应引发404错误。([#4645](https://github.com/encode/django-rest-framework/issues/4645), [#4646](https://github.com/encode/django-rest-framework/issues/4646))

+   为输入控件添加`autofocus`支持。([#4650](https://github.com/encode/django-rest-framework/issues/4650))

**日期**: [2016年11月1日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.2+Release%22)

+   恢复 Python 2.7 中的异常回溯。([#4631](https://github.com/encode/django-rest-framework/issues/4631), [#4638](https://github.com/encode/django-rest-framework/issues/4638))

+   在管理控制台中正确显示字典。([#4532](https://github.com/encode/django-rest-framework/issues/4532), [#4636](https://github.com/encode/django-rest-framework/issues/4636))

+   修复具有可变参数和关键字参数的 is_simple_callable。([#4622](https://github.com/encode/django-rest-framework/issues/4622), [#4602](https://github.com/encode/django-rest-framework/issues/4602))

+   支持 BooleanField 中的 'on'/'off' 字面值。([#4640](https://github.com/encode/django-rest-framework/issues/4640), [#4624](https://github.com/encode/django-rest-framework/issues/4624))

+   启用值查询集的游标分页。([#4569](https://github.com/encode/django-rest-framework/issues/4569))

+   修复对 Throttled 异常的 get_full_details() 支持。([#4627](https://github.com/encode/django-rest-framework/issues/4627))

+   修复 FilterSet 代理。([#4620](https://github.com/encode/django-rest-framework/issues/4620))

+   使序列化器字段导入显式化。([#4628](https://github.com/encode/django-rest-framework/issues/4628))

+   删除多余的请求适配器。([#4639](https://github.com/encode/django-rest-framework/issues/4639))

**日期**: [2016年10月21日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.1+Release%22)

+   使 `rest_framework/compat.py` 导入。([#4612](https://github.com/encode/django-rest-framework/issues/4612), [#4608](https://github.com/encode/django-rest-framework/issues/4608), [#4601](https://github.com/encode/django-rest-framework/issues/4601))

+   修复模式基路径生成中的错误。([#4611](https://github.com/encode/django-rest-framework/issues/4611), [#4605](https://github.com/encode/django-rest-framework/issues/4605))

+   修复单项列表序列化器的问题。([#4609](https://github.com/encode/django-rest-framework/issues/4609), [#4606](https://github.com/encode/django-rest-framework/issues/4606))

+   移除 Python 3.5 兼容性的裸 raise 语句。([#4600](https://github.com/encode/django-rest-framework/issues/4600))

**日期**: [2016年10月20日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.0+Release%22)

* * *

**日期**: [2016年9月21日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.7+Release%22)

+   当已经访问了 request.POST 时，请求解析的回退行为。([#3951](https://github.com/encode/django-rest-framework/issues/3951), [#4500](https://github.com/encode/django-rest-framework/issues/4500))

+   修复 RegexField 的回归问题。([#4489](https://github.com/encode/django-rest-framework/issues/4489), [#4490](https://github.com/encode/django-rest-framework/issues/4490), [#2617](https://github.com/encode/django-rest-framework/issues/2617))

+   在`admin.html`中缺少逗号导致CSRF错误。 ([#4472](https://github.com/encode/django-rest-framework/issues/4472), [#4473](https://github.com/encode/django-rest-framework/issues/4473))

+   修复空上下文响应渲染问题。 ([#4495](https://github.com/encode/django-rest-framework/issues/4495))

+   修复API列表中的缩进回归。 ([#4493](https://github.com/encode/django-rest-framework/issues/4493))

+   修复装饰了`api_view`的视图中`ResolverMatch.func_name`设置错误值的问题。 ([#4465](https://github.com/encode/django-rest-framework/issues/4465), [#4462](https://github.com/encode/django-rest-framework/issues/4462))

+   在路径包含Unicode参数时修复`APIClient.get()`。 ([#4458](https://github.com/encode/django-rest-framework/issues/4458))

**日期**：[2016年8月23日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.6+Release%22)

+   修复浏览器可浏览API中的错误的Javascript。 ([#4435](https://github.com/encode/django-rest-framework/issues/4435))

+   跳过Schema字段中的HiddenField。 ([#4425](https://github.com/encode/django-rest-framework/issues/4425), [#4429](https://github.com/encode/django-rest-framework/issues/4429))

+   改进创建以显示原始异常的回溯。 ([#3508](https://github.com/encode/django-rest-framework/issues/3508))

+   修复`AdminRenderer`显示仅相关字段的问题。 ([#4419](https://github.com/encode/django-rest-framework/issues/4419), [#4423](https://github.com/encode/django-rest-framework/issues/4423))

**日期**：[2016年8月19日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.5+Release%22)

+   改进调试错误处理。 ([#4416](https://github.com/encode/django-rest-framework/issues/4416), [#4409](https://github.com/encode/django-rest-framework/issues/4409))

+   允许自定义`CSRF_HEADER_NAME`设置。 ([#4415](https://github.com/encode/django-rest-framework/issues/4415), [#4410](https://github.com/encode/django-rest-framework/issues/4410))

+   在生成模式时，在视图集上包含`.action`属性。 ([#4408](https://github.com/encode/django-rest-framework/issues/4408), [#4398](https://github.com/encode/django-rest-framework/issues/4398))

+   不要将`request.FILES`项目包含在`request.POST`中。 ([#4407](https://github.com/encode/django-rest-framework/issues/4407))

+   修复复选框多重渲染问题。 ([#4403](https://github.com/encode/django-rest-framework/issues/4403))

+   修复`Field.get_default`的文档字符串。 ([#4404](https://github.com/encode/django-rest-framework/issues/4404))

+   用ASCII对应项替换README中的utf8字符。 ([#4412](https://github.com/encode/django-rest-framework/issues/4412))

**日期**：[2016年8月12日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.4+Release%22)

+   确保生成模式时视图完全初始化。([#4373](https://github.com/encode/django-rest-framework/issues/4373), [#4382](https://github.com/encode/django-rest-framework/issues/4382), [#4383](https://github.com/encode/django-rest-framework/issues/4383), [#4279](https://github.com/encode/django-rest-framework/issues/4279), [#4278](https://github.com/encode/django-rest-framework/issues/4278))

+   将表单字段描述添加到模式中。([#4387](https://github.com/encode/django-rest-framework/issues/4387))

+   修复模式端点的类别生成。([#4391](https://github.com/encode/django-rest-framework/issues/4391), [#4394](https://github.com/encode/django-rest-framework/issues/4394), [#4390](https://github.com/encode/django-rest-framework/issues/4390), [#4386](https://github.com/encode/django-rest-framework/issues/4386), [#4376](https://github.com/encode/django-rest-framework/issues/4376), [#4329](https://github.com/encode/django-rest-framework/issues/4329))

+   分页时不要删除空查询参数。([#4392](https://github.com/encode/django-rest-framework/issues/4392), [#4393](https://github.com/encode/django-rest-framework/issues/4393), [#4260](https://github.com/encode/django-rest-framework/issues/4260))

+   使用 LimitOffsetPagination 时不要重新运行空结果的查询。([#4201](https://github.com/encode/django-rest-framework/issues/4201), [#4388](https://github.com/encode/django-rest-framework/issues/4388))

+   对于 CharField 进行更严格的类型验证。([#4380](https://github.com/encode/django-rest-framework/issues/4380), [#3394](https://github.com/encode/django-rest-framework/issues/3394))

+   RelatedField.choices 应保留非字符串值。([#4111](https://github.com/encode/django-rest-framework/issues/4111), [#4379](https://github.com/encode/django-rest-framework/issues/4379), [#3365](https://github.com/encode/django-rest-framework/issues/3365))

+   在垂直表单样式中测试复选框的呈现。([#4378](https://github.com/encode/django-rest-framework/issues/4378), [#3868](https://github.com/encode/django-rest-framework/issues/3868), [#3868](https://github.com/encode/django-rest-framework/issues/3868))

+   在可浏览的 API 中显示错误回溯的 HTML。([#4042](https://github.com/encode/django-rest-framework/issues/4042), [#4172](https://github.com/encode/django-rest-framework/issues/4172))

+   修复对 ALLOWED_VERSIONS 和没有 DEFAULT_VERSION 的处理。[#4370](https://github.com/encode/django-rest-framework/issues/4370)

+   允许在 DecimalField 上使用 `max_digits=None`。([#4377](https://github.com/encode/django-rest-framework/issues/4377), [#4372](https://github.com/encode/django-rest-framework/issues/4372))

+   渲染关联选择时限制查询集。([#4375](https://github.com/encode/django-rest-framework/issues/4375), [#4122](https://github.com/encode/django-rest-framework/issues/4122), [#3329](https://github.com/encode/django-rest-framework/issues/3329), [#3330](https://github.com/encode/django-rest-framework/issues/3330), [#3877](https://github.com/encode/django-rest-framework/issues/3877))

+   解决 ChoiceField、MultipleChoiceField 和非字符串选择的表单显示问题。([#4374](https://github.com/encode/django-rest-framework/issues/4374), [#4119](https://github.com/encode/django-rest-framework/issues/4119), [#4121](https://github.com/encode/django-rest-framework/issues/4121), [#4137](https://github.com/encode/django-rest-framework/issues/4137), [#4120](https://github.com/encode/django-rest-framework/issues/4120))

+   修复对 TemplateHTMLRenderer.resolve_context() 回退方法的调用。([#4371](https://github.com/encode/django-rest-framework/issues/4371))

**日期**: [2016年8月5日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.3+Release%22)

+   对于使用旧的 TemplateHTMLRenderer 内部 API 的用户提供回退支持。([#4361](https://github.com/encode/django-rest-framework/issues/4361))

**日期**: [2016年8月5日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.2+Release%22)

+   在生成模式时包含传递给 'as_view' 的 kwargs。([#4359](https://github.com/encode/django-rest-framework/issues/4359), [#4330](https://github.com/encode/django-rest-framework/issues/4330), [#4331](https://github.com/encode/django-rest-framework/issues/4331))

+   在 Django 1.10+ 下，将 `request.user.is_authenticated` 作为属性而不是方法访问。([#4358](https://github.com/encode/django-rest-framework/issues/4358), [#4354](https://github.com/encode/django-rest-framework/issues/4354))

+   从模式中过滤掉 HEAD。([#4357](https://github.com/encode/django-rest-framework/issues/4357))

+   extra_kwargs 优先于唯一性 kwargs。([#4198](https://github.com/encode/django-rest-framework/issues/4198), [#4199](https://github.com/encode/django-rest-framework/issues/4199), [#4349](https://github.com/encode/django-rest-framework/issues/4349))

+   在代码缩进中使用制表符时修正描述。([#4345](https://github.com/encode/django-rest-framework/issues/4345), [#4347](https://github.com/encode/django-rest-framework/issues/4347))*

+   更改 TemplateHTMLRenderer 中的模板上下文生成。([#4236](https://github.com/encode/django-rest-framework/issues/4236))

+   序列化器的默认值不应包含在部分更新中。([#4346](https://github.com/encode/django-rest-framework/issues/4346), [#3565](https://github.com/encode/django-rest-framework/issues/3565))

+   文件上传解析器在未包含文件名时保持一致行为和描述性错误。([#4340](https://github.com/encode/django-rest-framework/issues/4340), [#3610](https://github.com/encode/django-rest-framework/issues/3610), [#4292](https://github.com/encode/django-rest-framework/issues/4292), [#4296](https://github.com/encode/django-rest-framework/issues/4296))

+   DecimalField 对传入的数字进行量化。([#4339](https://github.com/encode/django-rest-framework/issues/4339), [#4318](https://github.com/encode/django-rest-framework/issues/4318))

+   处理 IP 字段的非字符串输入。([#4335](https://github.com/encode/django-rest-framework/issues/4335), [#4336](https://github.com/encode/django-rest-framework/issues/4336), [#4338](https://github.com/encode/django-rest-framework/issues/4338))

+   修复当模式生成包含根 URL 时的前导斜杠处理。([#4332](https://github.com/encode/django-rest-framework/issues/4332))

+   带有 allow_null 选项的 DictField 的测试用例。([#4348](https://github.com/encode/django-rest-framework/issues/4348))

+   将测试从 Django 1.10 beta 更新到 Django 1.10\. ([#4344](https://github.com/encode/django-rest-framework/issues/4344))

**日期**: [2016年7月28日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.1+Release%22)

+   添加了 `root_renderers` 参数到 `DefaultRouter`。([#4323](https://github.com/encode/django-rest-framework/issues/4323), [#4268](https://github.com/encode/django-rest-framework/issues/4268))

+   添加了 `url` 和 `schema_url` 参数。([#4321](https://github.com/encode/django-rest-framework/issues/4321), [#4308](https://github.com/encode/django-rest-framework/issues/4308), [#4305](https://github.com/encode/django-rest-framework/issues/4305))

+   唯一的一起检查应该适用于具有默认值的只读字段。([#4316](https://github.com/encode/django-rest-framework/issues/4316), [#4294](https://github.com/encode/django-rest-framework/issues/4294))

+   在模式生成器中设置 view.format_kwarg。([#4293](https://github.com/encode/django-rest-framework/issues/4293), [#4315](https://github.com/encode/django-rest-framework/issues/4315))

+   修复对具有 `pagination_class = None` 的视图的模式生成器。([#4314](https://github.com/encode/django-rest-framework/issues/4314), [#4289](https://github.com/encode/django-rest-framework/issues/4289))

+   修复对没有 `get_serializer_class` 的视图的模式生成器。([#4265](https://github.com/encode/django-rest-framework/issues/4265), [#4285](https://github.com/encode/django-rest-framework/issues/4285))

+   修复在 `Accept` 和 `Content-Type` 标头中媒体类型参数的问题。([#4287](https://github.com/encode/django-rest-framework/issues/4287), [#4313](https://github.com/encode/django-rest-framework/issues/4313), [#4281](https://github.com/encode/django-rest-framework/issues/4281))

+   在错误消息中使用 verbose_name 而不是 object_name。([#4299](https://github.com/encode/django-rest-framework/issues/4299))

+   Twitter Bootstrap的次要版本更新。([#4307](https://github.com/encode/django-rest-framework/issues/4307))

+   使用相关字段时，SearchFilter引发错误。([#4302](https://github.com/encode/django-rest-framework/issues/4302), [#4303](https://github.com/encode/django-rest-framework/issues/4303), [#4298](https://github.com/encode/django-rest-framework/issues/4298))

+   添加对RFC 4918状态码的支持。([#4291](https://github.com/encode/django-rest-framework/issues/4291))

+   将LICENSE.md添加到构建的wheel中。([#4270](https://github.com/encode/django-rest-framework/issues/4270))

+   序列化“复杂”字段自3.4版本以来返回None而不是值。([#4272](https://github.com/encode/django-rest-framework/issues/4272), [#4273](https://github.com/encode/django-rest-framework/issues/4273), [#4288](https://github.com/encode/django-rest-framework/issues/4288))

**日期**: [2016年7月14日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.0+Release%22)

+   在JSON输出中不要去掉微秒。([#4256](https://github.com/encode/django-rest-framework/issues/4256))

+   两种稍有不同的ISO 8601日期时间序列化。([#4255](https://github.com/encode/django-rest-framework/issues/4255))

+   解决媒体类型参数的错误包含。([#4254](https://github.com/encode/django-rest-framework/issues/4254))

+   响应Content-Type可能格式错误。([#4253](https://github.com/encode/django-rest-framework/issues/4253))

+   修复某些平台上setup.py的错误。([#4246](https://github.com/encode/django-rest-framework/issues/4246))

+   将coreapi中的备用格式移入单独的包中。([#4244](https://github.com/encode/django-rest-framework/issues/4244))

+   向`DecimalField`添加本地化关键字参数。([#4233](https://github.com/encode/django-rest-framework/issues/4233))

+   修复自定义列表路由和详细路由的路由器问题。([#4229](https://github.com/encode/django-rest-framework/issues/4229))

+   嵌套命名空间的命名空间版本控制。([#4219](https://github.com/encode/django-rest-framework/issues/4219))

+   强大的唯一性检查。([#4217](https://github.com/encode/django-rest-framework/issues/4217))

+   对`must_call_distinct`进行轻微重构。([#4215](https://github.com/encode/django-rest-framework/issues/4215))

+   CursorPagination中可重写的偏移截止。([#4212](https://github.com/encode/django-rest-framework/issues/4212))

+   在日期/时间字段中按原样传递字符串。([#4196](https://github.com/encode/django-rest-framework/issues/4196))

+   添加测试以确认在关系字段上required=False有效。([#4195](https://github.com/encode/django-rest-framework/issues/4195))

+   在LimitOffsetPagination中，`limit=0`应恢复为默认限制。([#4194](https://github.com/encode/django-rest-framework/issues/4194))

+   从unique_together验证中排除read_only=True字段并添加文档。([#4192](https://github.com/encode/django-rest-framework/issues/4192))

+   处理JSON中的字节串。([#4191](https://github.com/encode/django-rest-framework/issues/4191))

+   JSONField(binary=True)表示使用二进制字符串，JSONRenderer不支持。([#4187](https://github.com/encode/django-rest-framework/issues/4187))

+   JSONField(binary=True)表示使用二进制字符串，JSONRenderer不支持。([#4185](https://github.com/encode/django-rest-framework/issues/4185))

+   在可浏览API中进行更健壮的表单渲染。([#4181](https://github.com/encode/django-rest-framework/issues/4181))

+   ListSerializer中`.validated_data`和`.errors`为空列表而不是字典的空情况。([#4180](https://github.com/encode/django-rest-framework/issues/4180))

+   模式和客户端库。([#4179](https://github.com/encode/django-rest-framework/issues/4179))

+   移除`AUTH_USER_MODEL`兼容性属性。([#4176](https://github.com/encode/django-rest-framework/issues/4176))

+   清理现有的弃用警告。([#4166](https://github.com/encode/django-rest-framework/issues/4166))

+   Django 1.10支持。([#4158](https://github.com/encode/django-rest-framework/issues/4158))

+   更新jQuery版本至1.12.4。([#4157](https://github.com/encode/django-rest-framework/issues/4157))

+   在OrderingFilter上实现更健壮的默认行为。([#4156](https://github.com/encode/django-rest-framework/issues/4156))

+   清理description.py中的代码和测试。([#4153](https://github.com/encode/django-rest-framework/issues/4153))

+   将guardian.VERSION包装在元组中。([#4149](https://github.com/encode/django-rest-framework/issues/4149))

+   为带有kwargs的字段优化验证器。([#4146](https://github.com/encode/django-rest-framework/issues/4146))

+   修复ListField、DictField子项中None值的表示问题。([#4118](https://github.com/encode/django-rest-framework/issues/4118))

+   解决TimeField在午夜值的表示问题。([#4107](https://github.com/encode/django-rest-framework/issues/4107))

+   在POST/DELETE请求后的重定向中为AdminRenderer设置适当的状态码。([#4106](https://github.com/encode/django-rest-framework/issues/4106))

+   TimeField渲染返回None而不是00:00:00。([#4105](https://github.com/encode/django-rest-framework/issues/4105))

+   修正错误命名的zh-hans和zh-hant语言环境路径。([#4103](https://github.com/encode/django-rest-framework/issues/4103))

+   避免在限制为0时引发异常。([#4098](https://github.com/encode/django-rest-framework/issues/4098))

+   允许在头部使用自定义关键词的TokenAuthentication。([#4097](https://github.com/encode/django-rest-framework/issues/4097))

+   处理不正确填充的HTTP基本认证头。([#4090](https://github.com/encode/django-rest-framework/issues/4090))

+   当limit=0时，LimitOffset分页导致Browsable API崩溃。([#4079](https://github.com/encode/django-rest-framework/issues/4079))

+   修复DecimalField的任意精度支持。([#4075](https://github.com/encode/django-rest-framework/issues/4075))

+   增加对自定义 CSRF Cookie 名称的支持。([#4049](https://github.com/encode/django-rest-framework/issues/4049))

+   修复由 #4035 引入的回归问题。([#4041](https://github.com/encode/django-rest-framework/issues/4041))

+   未授权视图故障应引发 403 错误。([#4040](https://github.com/encode/django-rest-framework/issues/4040))

+   修复 string_types / text_types 混淆。([#4025](https://github.com/encode/django-rest-framework/issues/4025))

+   不要在 OPTIONS 请求中列出相关字段的选择项。([#4021](https://github.com/encode/django-rest-framework/issues/4021))

+   修复拼写错误。([#4008](https://github.com/encode/django-rest-framework/issues/4008))

+   重新排序视图初始化过程。([#4006](https://github.com/encode/django-rest-framework/issues/4006))

+   DjangoObjectPermissionsFilter 在 Python 3.4 上的类型错误。([#4005](https://github.com/encode/django-rest-framework/issues/4005))

+   修复使用已弃用的 Query.aggregates。([#4003](https://github.com/encode/django-rest-framework/issues/4003))

+   修复文档字符串周围的空行。([#4002](https://github.com/encode/django-rest-framework/issues/4002))

+   修复当 limit 为 0 时的管理员分页。([#3990](https://github.com/encode/django-rest-framework/issues/3990))

+   OrderingFilter 调整。([#3983](https://github.com/encode/django-rest-framework/issues/3983))

+   非必需的序列化器相关字段。([#3976](https://github.com/encode/django-rest-framework/issues/3976))

+   在教程中使用更安全的 "@api_view" 调用方式。([#3971](https://github.com/encode/django-rest-framework/issues/3971))

+   ListSerializer 未处理 unique_together 约束。([#3970](https://github.com/encode/django-rest-framework/issues/3970))

+   添加丢失的迁移文件。([#3968](https://github.com/encode/django-rest-framework/issues/3968))

+   `OrderingFilter` 应调用 `get_serializer_class()` 来确定默认字段。([#3964](https://github.com/encode/django-rest-framework/issues/3964))

+   从测试和兼容性中删除旧的 Django 检查项。([#3953](https://github.com/encode/django-rest-framework/issues/3953))

+   支持任何 `serializer.Field` 的 `initial` 值作为可调用对象。([#3943](https://github.com/encode/django-rest-framework/issues/3943))

+   防止在 SearchFilter 中不必要地调用 distinct()。([#3938](https://github.com/encode/django-rest-framework/issues/3938))

+   修复 None UUID ForeignKey 的序列化。([#3936](https://github.com/encode/django-rest-framework/issues/3936))

+   放弃 EOL Django 1.7。([#3933](https://github.com/encode/django-rest-framework/issues/3933))

+   在序列化器错误消息中添加缺失的空格。([#3926](https://github.com/encode/django-rest-framework/issues/3926))

+   修复 _force_text_recursive 的拼写错误。([#3908](https://github.com/encode/django-rest-framework/issues/3908))

+   尝试解决与 `field.rel` 相关的 Django 2.0 废弃警告问题。([#3906](https://github.com/encode/django-rest-framework/issues/3906))

+   修复使用嵌套序列化器与列表解析多部分数据的解析问题。([#3820](https://github.com/encode/django-rest-framework/issues/3820))

+   解决API的URL解析到不同的命名空间。([#3816](https://github.com/encode/django-rest-framework/issues/3816))

+   在可浏览API表单中不要对`help_text`进行HTML转义。([#3812](https://github.com/encode/django-rest-framework/issues/3812))

+   OPTIONS获取并显示所有可能的外键选择字段。([#3751](https://github.com/encode/django-rest-framework/issues/3751))

+   Django 1.9废弃警告 ([#3729](https://github.com/encode/django-rest-framework/issues/3729))

+   对#3598的测试用例。([#3710](https://github.com/encode/django-rest-framework/issues/3710))

+   添加对搜索过滤器多值的支持。([#3541](https://github.com/encode/django-rest-framework/issues/3541))

+   在排序过滤器中使用get_serializer_class。([#3487](https://github.com/encode/django-rest-framework/issues/3487))

+   `many=True`的序列化器应返回空列表而不是空字典。([#3476](https://github.com/encode/django-rest-framework/issues/3476))

+   LimitOffsetPagination修复limit=0。([#3444](https://github.com/encode/django-rest-framework/issues/3444))

+   允许验证器延迟字符串评估和处理新的字符串格式。([#3438](https://github.com/encode/django-rest-framework/issues/3438))

+   如果字段无效，则执行唯一性验证器并中断。([#3381](https://github.com/encode/django-rest-framework/issues/3381))

+   不要忽略在面包屑中重写的View.get_view_name()。([#3273](https://github.com/encode/django-rest-framework/issues/3273))

+   渲染表单序列化失败时，重试表单渲染。([#3164](https://github.com/encode/django-rest-framework/issues/3164))

+   唯一约束阻止嵌套序列化器更新。([#2996](https://github.com/encode/django-rest-framework/issues/2996))

+   唯一性验证器不应对排除（只读）字段运行。([#2848](https://github.com/encode/django-rest-framework/issues/2848))

+   UniqueValidator针对嵌套对象引发异常。([#2403](https://github.com/encode/django-rest-framework/issues/2403))

+   `lookup_type`已弃用，应使用`lookup_expr`。([#4259](https://github.com/encode/django-rest-framework/issues/4259))

* * *

**日期**: [2016年3月14日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.3+Release%22)。

+   从模板中移除版本字符串。感谢@blag的报告和修复。([#3878](https://github.com/encode/django-rest-framework/issues/3878), [#3913](https://github.com/encode/django-rest-framework/issues/3913), [#3912](https://github.com/encode/django-rest-framework/issues/3912))

+   修复`BooleanField`的垂直HTML布局。感谢Mikalai Radchuk的修复。([#3910](https://github.com/encode/django-rest-framework/issues/3910))

+   在Django 1.8上静默废弃警告。感谢Simon Charette的修复。([#3903](https://github.com/encode/django-rest-framework/issues/3903))

+   authtoken的国际化。感谢Michael Nacharov的修复。([#3887](https://github.com/encode/django-rest-framework/issues/3887), [#3968](https://github.com/encode/django-rest-framework/issues/3968))

+   修复当未声明authtoken应用程序时`Token`模型作为`abstract`的问题。感谢Adam Thomas的报告。([#3860](https://github.com/encode/django-rest-framework/issues/3860), [#3858](https://github.com/encode/django-rest-framework/issues/3858))

+   改进Markdown版本兼容性。感谢Michael J. Schultz的修复。([#3604](https://github.com/encode/django-rest-framework/issues/3604), [#3842](https://github.com/encode/django-rest-framework/issues/3842))

+   `QueryParameterVersioning`不使用`DEFAULT_VERSION`设置。感谢Brad Montgomery的修复。([#3833](https://github.com/encode/django-rest-framework/issues/3833))

+   在模型上添加显式的`on_delete`。感谢Mads Jensen的修复。([#3832](https://github.com/encode/django-rest-framework/issues/3832))

+   修复`DateField.to_representation`以兼容Python 2的unicode。感谢Mikalai Radchuk的修复。([#3819](https://github.com/encode/django-rest-framework/issues/3819))

+   修复`TimeField`无法处理字符串时间的问题。感谢Areski Belaid的修复。([#3809](https://github.com/encode/django-rest-framework/issues/3809))

+   避免更新`Meta.extra_kwargs`。感谢Kevin Massey的报告和修复。([#3805](https://github.com/encode/django-rest-framework/issues/3805), [#3804](https://github.com/encode/django-rest-framework/issues/3804))

+   修复渲染嵌套验证错误的问题。感谢Craig de Stigter的修复。([#3801](https://github.com/encode/django-rest-framework/issues/3801))

+   记录如何避免与`django-crispy-forms`的CSRF和缺少按钮问题。感谢Emmanuelle Delescolle，José Padilla和Luis San Pablo的报告、分析和修复。([#3787](https://github.com/encode/django-rest-framework/issues/3787), [#3636](https://github.com/encode/django-rest-framework/issues/3636), [#3637](https://github.com/encode/django-rest-framework/issues/3637))

+   改进Rest Framework设置文件的设置时间。感谢Miles Hutson的报告和Mads Jensen的修复。([#3786](https://github.com/encode/django-rest-framework/issues/3786), [#3815](https://github.com/encode/django-rest-framework/issues/3815))

+   改进与Django 1.9兼容的authtoken。感谢S. Andrew Sheppard的修复。([#3785](https://github.com/encode/django-rest-framework/issues/3785))

+   修复从模型的`DecimalField`传输到`Min/MaxValueValidator`的问题。感谢Kevin Brown的修复。([#3774](https://github.com/encode/django-rest-framework/issues/3774))

+   改进Browsable API中的HTML标题。感谢Mike Lissner的报告和修复。([#3769](https://github.com/encode/django-rest-framework/issues/3769))

+   修复 `AutoFilterSet` 以继承 `default_filter_set`。感谢 Tom Linford 的修复。([#3753](https://github.com/encode/django-rest-framework/issues/3753))

+   修复 transifex 配置以处理新的中文语言代码。感谢 @nypisces 的报告和修复。([#3739](https://github.com/encode/django-rest-framework/issues/3739))

+   `DateTimeField` 不能正确处理空值。感谢 Mick Parker 的报告和修复。([#3731](https://github.com/encode/django-rest-framework/issues/3731), [#3726](https://github.com/encode/django-rest-framework/issues/3726))

+   在设置已移除的 rest_framework 设置时引发错误。感谢 Luis San Pablo 的修复。([#3715](https://github.com/encode/django-rest-framework/issues/3715))

+   在 AdminRenderer 的提交表单中添加丢失的 csrf_token。感谢 Piotr Śniegowski 的修复。([#3703](https://github.com/encode/django-rest-framework/issues/3703))

+   重构 `_get_reverse_relationships()` 以使用正确的 `to_field`。感谢 Benjamin Phillips 的修复。([#3696](https://github.com/encode/django-rest-framework/issues/3696))

+   文档化 `RelatedField` 的 `get_queryset` 使用方法。感谢 Ryan Hiebert 的修复。([#3605](https://github.com/encode/django-rest-framework/issues/3605))

+   修复在 HyperlinkRelatedField.get_url 中检测空 pk 的问题。感谢 @jslang 的修复。([#3962](https://github.com/encode/django-rest-framework/issues/3962))

**日期**：[2015年12月14日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.2+Release%22)。

+   `ListField` 强制输入为列表。([#3513](https://github.com/encode/django-rest-framework/issues/3513))

+   修复隐藏原始数据表单的回归问题。([#3600](https://github.com/encode/django-rest-framework/issues/3600), [#3578](https://github.com/encode/django-rest-framework/issues/3578))

+   修复 Python 3.5 兼容性问题。([#3534](https://github.com/encode/django-rest-framework/issues/3534), [#3626](https://github.com/encode/django-rest-framework/issues/3626))

+   允许在 `pagination.PageNumberPagination` 中设置自定义的 Django 分页器。([#3631](https://github.com/encode/django-rest-framework/issues/3631), [#3684](https://github.com/encode/django-rest-framework/issues/3684))

+   修复没有 `to_fields` 属性的关系字段。([#3635](https://github.com/encode/django-rest-framework/issues/3635), [#3634](https://github.com/encode/django-rest-framework/issues/3634))

+   修复 `template.render` 在 Django 1.9 中的弃用警告。([#3654](https://github.com/encode/django-rest-framework/issues/3654))

+   在可浏览 API 渲染器中排序响应头。([#3655](https://github.com/encode/django-rest-framework/issues/3655))

+   使用相关对象的 API 来处理 Django 1.9+。([#3656](https://github.com/encode/django-rest-framework/issues/3656), [#3252](https://github.com/encode/django-rest-framework/issues/3252))

+   添加删除时的确认模态框。([#3228](https://github.com/encode/django-rest-framework/issues/3228), [#3662](https://github.com/encode/django-rest-framework/issues/3662))

+   在调用 `has_[object_]permissions` 时揭示先前隐藏的 AttributeError 和 TypeError。([#3668](https://github.com/encode/django-rest-framework/issues/3668))

+   使 DRF 兼容 Django 1.8 中的多模板引擎。([#3672](https://github.com/encode/django-rest-framework/issues/3672))

+   更新 `NestedBoundField` 在渲染表单时也处理空字符串。([#3677](https://github.com/encode/django-rest-framework/issues/3677))

+   修复 UUID 验证以正确捕获无效输入类型。([#3687](https://github.com/encode/django-rest-framework/issues/3687), [#3679](https://github.com/encode/django-rest-framework/issues/3679))

+   修复缓存问题。([#3628](https://github.com/encode/django-rest-framework/issues/3628), [#3701](https://github.com/encode/django-rest-framework/issues/3701))

+   修复 Admin 和 API 浏览器，以适应没有 filter_class 的视图。([#3705](https://github.com/encode/django-rest-framework/issues/3705), [#3596](https://github.com/encode/django-rest-framework/issues/3596), [#3597](https://github.com/encode/django-rest-framework/issues/3597))

+   添加 app_name 到 rest_framework.urls。([#3714](https://github.com/encode/django-rest-framework/issues/3714))

+   改进 authtoken 的视图以支持 URL 版本控制。([#3718](https://github.com/encode/django-rest-framework/issues/3718), [#3723](https://github.com/encode/django-rest-framework/issues/3723))

**日期**：[2015年11月4日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.1+Release%22)。

+   解决访问 `request.POST` 时的解析 bug。([#3592](https://github.com/encode/django-rest-framework/issues/3592))

+   正确处理 `to_field` 引用主键。([#3593](https://github.com/encode/django-rest-framework/issues/3593))

+   允许在未定义 `filter_class` 的情况下渲染 HTML 过滤器。([#3560](https://github.com/encode/django-rest-framework/issues/3560))

+   修复管理员渲染问题。([#3564](https://github.com/encode/django-rest-framework/issues/3564), [#3556](https://github.com/encode/django-rest-framework/issues/3556))

+   修复 DecimalValidator 问题。([#3568](https://github.com/encode/django-rest-framework/issues/3568))

**日期**：[2015年10月28日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.0+Release%22)。

+   HTML 控件用于过滤器。([#3315](https://github.com/encode/django-rest-framework/issues/3315))

+   表单 API。([#3475](https://github.com/encode/django-rest-framework/issues/3475))

+   AJAX 可浏览 API。([#3410](https://github.com/encode/django-rest-framework/issues/3410))

+   添加 JSONField。([#3454](https://github.com/encode/django-rest-framework/issues/3454))

+   在创建 `ModelSerializer` 关联字段时正确映射 `to_field`。([#3526](https://github.com/encode/django-rest-framework/issues/3526))

+   在将`FilePathField`映射到序列化器字段时包含关键字参数。([#3536](https://github.com/encode/django-rest-framework/issues/3536))

+   在`ModelSerializer`的唯一性约束上映射适当的模型`error_messages`。([#3435](https://github.com/encode/django-rest-framework/issues/3435))

+   为从`TextField`映射的`ModelSerializer`字段包含`max_length`约束。([#3509](https://github.com/encode/django-rest-framework/issues/3509))

+   添加了对Django 1.9的支持。([#3450](https://github.com/encode/django-rest-framework/issues/3450), [#3525](https://github.com/encode/django-rest-framework/issues/3525))

+   移除了对Django 1.5和1.6的支持。([#3421](https://github.com/encode/django-rest-framework/issues/3421), [#3429](https://github.com/encode/django-rest-framework/issues/3429))

+   移除了'south'迁移。([#3495](https://github.com/encode/django-rest-framework/issues/3495))

* * *

**日期**：[2015年10月27日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.5+Release%22)。

+   在可选注销标签中转义`username`。([#3550](https://github.com/encode/django-rest-framework/issues/3550))

**日期**：[2015年9月21日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.4+Release%22)。

+   在缺少`ViewSet.search_fields`属性时不报错。([#3324](https://github.com/encode/django-rest-framework/issues/3324), [#3323](https://github.com/encode/django-rest-framework/issues/3323))

+   修复在`many=True`的序列化器上`allow_empty`不起作用的问题。([#3361](https://github.com/encode/django-rest-framework/issues/3361), [#3364](https://github.com/encode/django-rest-framework/issues/3364))

+   让`DurationField`接受整数。([#3359](https://github.com/encode/django-rest-framework/issues/3359))

+   不支持多级字典在多部分请求中。([#3314](https://github.com/encode/django-rest-framework/issues/3314))

+   修复在HTTP PATCH上`ListField`截断的问题。([#3415](https://github.com/encode/django-rest-framework/issues/3415), [#2761](https://github.com/encode/django-rest-framework/issues/2761))

**日期**：[2015年8月24日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.3+Release%22)。

+   添加了`html_cutoff`和`html_cutoff_text`来限制选择下拉框。([#3313](https://github.com/encode/django-rest-framework/issues/3313))

+   在`SearchFilter`中添加了正则表达式风格。([#3316](https://github.com/encode/django-rest-framework/issues/3316))

+   解决设置空白HTML字段的问题。([#3318](https://github.com/encode/django-rest-framework/issues/3318)) ([#3321](https://github.com/encode/django-rest-framework/issues/3321))

+   在浏览API表单中正确显示现有的“选择多个”值。([#3290](https://github.com/encode/django-rest-framework/issues/3290))

+   解决 `IPAddressField` 的重复验证消息。([#3249](https://github.com/encode/django-rest-framework/issues/3249)) ([#3250](https://github.com/encode/django-rest-framework/issues/3250))

+   修复在禁用分页时管理员渲染器继续工作的问题。([#3275](https://github.com/encode/django-rest-framework/issues/3275))

+   解决 `LimitOffsetPagination` 在 count=0、offset=0 时的错误。([#3303](https://github.com/encode/django-rest-framework/issues/3303))

**日期**：[2015年8月13日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.2+Release%22)。

+   添加 `display_value()` 方法以在显示关联字段选择输入时使用。([#3254](https://github.com/encode/django-rest-framework/issues/3254))

+   修复 `BooleanField` 复选框显示错误的问题。([#3258](https://github.com/encode/django-rest-framework/issues/3258))

+   确保空复选框在所有情况下正确设置 `BooleanField` 为 `False`。([#2776](https://github.com/encode/django-rest-framework/issues/2776))

+   允许 `WSGIRequest.FILES` 属性，避免引发不正确的已弃用错误。([#3261](https://github.com/encode/django-rest-framework/issues/3261))

+   解决在表单中渲染嵌套序列化器的问题。([#3260](https://github.com/encode/django-rest-framework/issues/3260))

+   如果用户意外将序列化器实例传递给响应而不是数据，则引发错误。([#3241](https://github.com/encode/django-rest-framework/issues/3241))

**日期**：[2015年8月7日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.1+Release%22)。

+   修复关联选择部件在没有任何选择时的渲染问题。([#3237](https://github.com/encode/django-rest-framework/issues/3237))

+   修复在管理界面中将 `1`、`0` 渲染为 `true`、`false` 的问题。[#3227](https://github.com/encode/django-rest-framework/issues/3227))

+   修复 HTML 表单输入中 ListFields 单个值的问题。([#3238](https://github.com/encode/django-rest-framework/issues/3238))

+   兼容 Django 的 `HTTPRequest` 类，允许 `request.FILES`。([#3239](https://github.com/encode/django-rest-framework/issues/3239))

**日期**：[2015年8月6日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.0+Release%22)。

+   添加 `AdminRenderer`。([#2926](https://github.com/encode/django-rest-framework/issues/2926))

+   添加 `FilePathField`。([#1854](https://github.com/encode/django-rest-framework/issues/1854))

+   添加 `allow_empty` 到 `ListField`。([#2250](https://github.com/encode/django-rest-framework/issues/2250))

+   支持 django-guardian 1.3。([#3165](https://github.com/encode/django-rest-framework/issues/3165))

+   支持分组选择。([#3225](https://github.com/encode/django-rest-framework/issues/3225))

+   支持可浏览 API 中的错误表单。([#3024](https://github.com/encode/django-rest-framework/issues/3024))

+   允许权限类自定义错误消息。（[#2539](https://github.com/encode/django-rest-framework/issues/2539)）

+   在超链接字段上支持 `source=<method>`。（[#2690](https://github.com/encode/django-rest-framework/issues/2690)）

+   `ListField(allow_null=True)` 现在允许空作为列表值，而不是列表中的空项。（[#2766](https://github.com/encode/django-rest-framework/issues/2766)）

+   `ManyToMany()` 映射到 `allow_empty=False`，`ManyToMany(blank=True)` 映射到 `allow_empty=True`。（[#2804](https://github.com/encode/django-rest-framework/issues/2804)）

+   支持主键字段的自定义序列化样式。（[#2789](https://github.com/encode/django-rest-framework/issues/2789)）

+   `OPTIONS` 请求支持嵌套表示。（[#2915](https://github.com/encode/django-rest-framework/issues/2915)）

+   对于带有 `OPTIONS` 请求的视图集，设置 `view.action == "metadata"`。（[#3115](https://github.com/encode/django-rest-framework/issues/3115)）

+   支持 `UUIDField` 上的 `allow_blank`。（[#3130][gh#3130]）

+   在 401 或 403 响应代码时不显示视图文档字符串。（[#3216](https://github.com/encode/django-rest-framework/issues/3216)）

+   解决 Django 1.8 弃用警告问题。（[#2886](https://github.com/encode/django-rest-framework/issues/2886)）

+   修复 `DecimalField` 验证问题。（[#3139](https://github.com/encode/django-rest-framework/issues/3139)）

+   在与 `trim_whitespace=True` 结合使用时，修复 `allow_blank=False` 的行为问题。（[#2712](https://github.com/encode/django-rest-framework/issues/2712)）

+   修复某些字段组合不正确映射到无效的 `allow_blank` 参数问题。（[#3011](https://github.com/encode/django-rest-framework/issues/3011)）

+   修复带有预获取和修改查询集的输出表示。（[#2704](https://github.com/encode/django-rest-framework/issues/2704), [#2727](https://github.com/encode/django-rest-framework/issues/2727)）

+   在某些无效查询参数情况下，当 `CursorPagination` 被提供时修复断言错误。（#2920）[gh2920](https://github.com/encode/django-rest-framework/issues/2920)。

+   使用 `TokenAuthentication` 时，修复头部包含无效字符导致的 `UnicodeDecodeError`。（[#2928](https://github.com/encode/django-rest-framework/issues/2928)）

+   修复 `@non_atomic_requests` 装饰器事务回滚问题。（[#3016](https://github.com/encode/django-rest-framework/issues/3016)）

+   修复 Oracle 数据库使用 `SearchFilter` 导致重复结果问题。（[#2935](https://github.com/encode/django-rest-framework/issues/2935)）

+   修复浏览 API 表单中复选框对齐和渲染问题。（[#2783](https://github.com/encode/django-rest-framework/issues/2783)）

+   修复未保存文件对象，应在表示中使用 `"url": null`。（[#2759](https://github.com/encode/django-rest-framework/issues/2759)）

+   修复浏览 API 中字段值渲染问题。（[#2416](https://github.com/encode/django-rest-framework/issues/2416)）

+   修复`HStoreField`，在`DictField`映射中包含`allow_blank=True`。（[#2659](https://github.com/encode/django-rest-framework/issues/2659)）

+   大量的其他清理工作，改进错误消息，私有 API 和次要修复。

* * *

**日期**：[2015年6月4日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.3+Release%22)。

+   添加`DurationField`。（[#2481](https://github.com/encode/django-rest-framework/issues/2481), [#2989](https://github.com/encode/django-rest-framework/issues/2989)）

+   添加`UUIDField` 的 `format` 参数。（[#2788](https://github.com/encode/django-rest-framework/issues/2788), [#3000](https://github.com/encode/django-rest-framework/issues/3000)）

+   使用`multipart/form-data`进行部分更新时，`MultipleChoiceField` 在空输入时出现错误。（[#2993](https://github.com/encode/django-rest-framework/issues/2993), [#2894](https://github.com/encode/django-rest-framework/issues/2894)）

+   修复与只读`RelatedField`相关的选项中的错误。（[#2981](https://github.com/encode/django-rest-framework/issues/2981), [#2811](https://github.com/encode/django-rest-framework/issues/2811)）

+   修复带有`unique_together`关系的嵌套序列化器。（[#2975](https://github.com/encode/django-rest-framework/issues/2975)）

+   允许`ChoiceField`/`MultipleChoiceField`表示中出现意外值。（[#2839](https://github.com/encode/django-rest-framework/issues/2839), [#2940](https://github.com/encode/django-rest-framework/issues/2940)）

+   如果设置了`ATOMIC_REQUESTS`，则在错误时回滚事务。（[#2887](https://github.com/encode/django-rest-framework/issues/2887), [#2034](https://github.com/encode/django-rest-framework/issues/2034)）

+   将`override_method`的操作设置在视图上，无论其是否为`None`。（[#2933](https://github.com/encode/django-rest-framework/issues/2933)）

+   `DecimalField` 接受`2E+2`作为200并正确验证小数位。（[#2948](https://github.com/encode/django-rest-framework/issues/2948), [#2947](https://github.com/encode/django-rest-framework/issues/2947)）

+   支持具有自定义`UserModel`且更改`username`的基本身份验证。（[#2952](https://github.com/encode/django-rest-framework/issues/2952)）

+   `IPAddressField` 改进。（[#2747](https://github.com/encode/django-rest-framework/issues/2747), [#2618](https://github.com/encode/django-rest-framework/issues/2618), [#3008](https://github.com/encode/django-rest-framework/issues/3008)）

+   改进`DecimalField` 以便更容易子类化。（[#2695](https://github.com/encode/django-rest-framework/issues/2695)）

**日期**：[2015年5月13日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.2+Release%22)。

+   `DateField.to_representation` 能够处理字符串和空值。（[#2656](https://github.com/encode/django-rest-framework/issues/2656), [#2687](https://github.com/encode/django-rest-framework/issues/2687), [#2869](https://github.com/encode/django-rest-framework/issues/2869)）

+   使用 HTTP 标准的默认原因短语。（[#2764](https://github.com/encode/django-rest-framework/issues/2764)，[#2763](https://github.com/encode/django-rest-framework/issues/2763)）

+   当 `ModelSerializer` 与抽象模型一起使用时引发错误。（[#2757](https://github.com/encode/django-rest-framework/issues/2757)，[#2630](https://github.com/encode/django-rest-framework/issues/2630)）

+   处理 `HyperLinkedRelatedField` 中非 API 视图名的反向问题。（[#2724](https://github.com/encode/django-rest-framework/issues/2724)，[#2711](https://github.com/encode/django-rest-framework/issues/2711)）

+   不再严格要求关联字段必须有主键。（[#2745](https://github.com/encode/django-rest-framework/issues/2745)，[#2754](https://github.com/encode/django-rest-framework/issues/2754)）

+   元数据检测空布尔字段类型。（[#2762](https://github.com/encode/django-rest-framework/issues/2762)）

+   正确处理嵌套序列化器中的深度。（[#2798](https://github.com/encode/django-rest-framework/issues/2798)）

+   显示没有分页器的视图集。（[#2807](https://github.com/encode/django-rest-framework/issues/2807)）

+   不再检查已弃用的 `.model` 属性以用于权限。（[#2818](https://github.com/encode/django-rest-framework/issues/2818)）

+   限制整数字段只能为整数和字符串。（[#2835](https://github.com/encode/django-rest-framework/issues/2835)，[#2836](https://github.com/encode/django-rest-framework/issues/2836)）

+   改进 `IntegerField` 以使用编译后的十进制正则表达式。（[#2853](https://github.com/encode/django-rest-framework/issues/2853)）

+   防止空 `queryset` 引发断言错误。（[#2862](https://github.com/encode/django-rest-framework/issues/2862)）

+   `DjangoModelPermissions` 依赖于 `get_queryset`。（[#2863](https://github.com/encode/django-rest-framework/issues/2863)）

+   在内容协商中检查 `AcceptHeaderVersioning`。（[#2868](https://github.com/encode/django-rest-framework/issues/2868)）

+   允许 `DjangoObjectPermissions` 使用定义了 `get_queryset` 的视图。（[#2905](https://github.com/encode/django-rest-framework/issues/2905)）

**日期**：[2015年3月23日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.1+Release%22)。

+   **安全修复**：在可浏览 API 中转义切换标签的 cookie 名称。

+   如果使用 `serializer_class`，即使视图上没有 `get_serializer` 方法，也在可浏览 API 中显示输入表单。（[#2743](https://github.com/encode/django-rest-framework/issues/2643)）

+   使用密码输入用于 `AuthTokenSerializer`。（[#2741](https://github.com/encode/django-rest-framework/issues/2641)）

+   修复下一个按钮后缺少锚点闭合标签的问题。（[#2691](https://github.com/encode/django-rest-framework/issues/2691)）

+   修复视图集中 `lookup_url_kwarg` 的处理。（[#2685](https://github.com/encode/django-rest-framework/issues/2685)，[#2591](https://github.com/encode/django-rest-framework/issues/2591)）

+   修复在 `apps.py` 中导入 `rest_framework.views` 时的问题。([#2678](https://github.com/encode/django-rest-framework/issues/2678))

+   如果未设置 PAGE_SIZE，则 LimitOffsetPagination 会引发 `TypeError`。([#2667](https://github.com/encode/django-rest-framework/issues/2667), [#2700](https://github.com/encode/django-rest-framework/issues/2700))

+   德语翻译中 `min_value` 字段错误消息引用了 `max_value`。([#2645](https://github.com/encode/django-rest-framework/issues/2645))

+   移除 `MergeDict`。([#2640](https://github.com/encode/django-rest-framework/issues/2640))

+   支持序列化未保存的具有相关字段的模型。([#2637](https://github.com/encode/django-rest-framework/issues/2637), [#2641](https://github.com/encode/django-rest-framework/issues/2641))

+   允许在 `radio.html` 选择中使用空白/空值。([#2631](https://github.com/encode/django-rest-framework/issues/2631))

**日期**: [2015年3月5日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.0+Release%22).

详细信息请参阅 [3.1 发布公告](../3.1-announcement/).

* * *

**日期**: [2015年2月10日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.5+Release%22).

+   修复 `_closable_objects` 破坏序列化的 bug。([#1850](https://github.com/encode/django-rest-framework/issues/1850), [#2492](https://github.com/encode/django-rest-framework/issues/2492))

+   允许使用 `Throttling` 处理非标准的 `User` 模型。([#2524](https://github.com/encode/django-rest-framework/issues/2524))

+   支持在 TokenAuthentication 迁移中使用自定义 `User.db_table`。([#2479](https://github.com/encode/django-rest-framework/issues/2479))

+   修复 `Request` 对象上误导性的 `AttributeError` 追溯问题。([#2530](https://github.com/encode/django-rest-framework/issues/2530), [#2108](https://github.com/encode/django-rest-framework/issues/2108))

+   `ManyRelatedField.get_value` 在部分更新时清除字段。([#2475](https://github.com/encode/django-rest-framework/issues/2475))

+   从代码中移除 '.model' 快捷方式。([#2486](https://github.com/encode/django-rest-framework/issues/2486))

+   修复 `detail_route` 和 `list_route` 可变参数的问题。([#2518](https://github.com/encode/django-rest-framework/issues/2518))

+   在 `TokenAuthentication` 中获取令牌时预获取用户对象。([#2519](https://github.com/encode/django-rest-framework/issues/2519))

**日期**: [2015年1月28日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.4+Release%22).

+   支持 Django 1.8a1。([#2425](https://github.com/encode/django-rest-framework/issues/2425), [#2446](https://github.com/encode/django-rest-framework/issues/2446), [#2441](https://github.com/encode/django-rest-framework/issues/2441))

+   添加 `DictField` 并支持 Django 1.8 的 `HStoreField`。([#2451](https://github.com/encode/django-rest-framework/issues/2451), [#2106](https://github.com/encode/django-rest-framework/issues/2106))

+   添加`UUIDField`并支持Django 1.8的`UUIDField`。([#2448](https://github.com/encode/django-rest-framework/issues/2448), [#2433](https://github.com/encode/django-rest-framework/issues/2433), [#2432](https://github.com/encode/django-rest-framework/issues/2432))

+   `BaseRenderer.render`现在会抛出`NotImplementedError`。([#2434](https://github.com/encode/django-rest-framework/issues/2434))

+   修复Python 2.6上的timedelta JSON序列化问题。([#2430](https://github.com/encode/django-rest-framework/issues/2430))

+   `ResultDict`和`ResultList`现在显示为标准的dict/list。([#2421](https://github.com/encode/django-rest-framework/issues/2421))

+   修复在Web可浏览API页面的HTML表单中可见的`HiddenField`。([#2410](https://github.com/encode/django-rest-framework/issues/2410))

+   对于`RelatedField.choices`，现在使用`OrderedDict`。([#2408](https://github.com/encode/django-rest-framework/issues/2408))

+   修复使用`HTTP_X_FORWARDED_FOR`时的标识格式。([#2401](https://github.com/encode/django-rest-framework/issues/2401))

+   修复在使用节流时与`memcached`不合法的键。([#2400](https://github.com/encode/django-rest-framework/issues/2400))

+   修复版本3.x中的`FileUploadParser`。([#2399](https://github.com/encode/django-rest-framework/issues/2399))

+   修复序列化器继承问题。([#2388](https://github.com/encode/django-rest-framework/issues/2388))

+   修复与`ReturnDict`相关的缓存问题。([#2360](https://github.com/encode/django-rest-framework/issues/2360))

**日期**：[2015年1月8日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.3+Release%22)。

+   修复`models.DateField`上`MinValueValidator`的问题。([#2369](https://github.com/encode/django-rest-framework/issues/2369))

+   修复使用分页时序列化器缺少上下文的问题。([#2355](https://github.com/encode/django-rest-framework/issues/2355))

+   `DefaultRouter`现在支持命名空间路由URL。([#2351](https://github.com/encode/django-rest-framework/issues/2351))

+   `required=False`允许输出时省略值。([#2342](https://github.com/encode/django-rest-framework/issues/2342))

+   使用文本框输入来处理`models.TextField`。([#2340](https://github.com/encode/django-rest-framework/issues/2340))

+   如果需要，使用自定义的`ListSerializer`来进行分页。([#2331](https://github.com/encode/django-rest-framework/issues/2331), [#2327](https://github.com/encode/django-rest-framework/issues/2327))

+   更好地处理空白HTML字段的null和''情况。([#2330](https://github.com/encode/django-rest-framework/issues/2330))

+   确保`exclude`中的字段是模型字段。([#2319](https://github.com/encode/django-rest-framework/issues/2319))

+   修复`IntegerField`和`max_length`参数的不兼容性。([#2317](https://github.com/encode/django-rest-framework/issues/2317))

+   修复3.0版本序列化器的YAML编码器。([#2315](https://github.com/encode/django-rest-framework/issues/2315), [#2283](https://github.com/encode/django-rest-framework/issues/2283))

+   修复空HTML字段的行为。 ([#2311](https://github.com/encode/django-rest-framework/issues/2311), [#1101](https://github.com/encode/django-rest-framework/issues/1101))

+   修复元类属性深度忽略字段属性。 ([#2287](https://github.com/encode/django-rest-framework/issues/2287))

+   修复 `format_suffix_patterns` 与 Django 的 `i18n_patterns` 协作的问题。 ([#2278](https://github.com/encode/django-rest-framework/issues/2278))

+   能够自定义路由器 URL 以进行自定义操作，使用 `url_path`。 ([#2010](https://github.com/encode/django-rest-framework/issues/2010))

+   不要将 Django REST Framework 安装为egg包。 ([#2386](https://github.com/encode/django-rest-framework/issues/2386))

**日期**：[2014年12月17日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.2+Release%22)。

+   确保将 `request.user` 提供给响应中间件。 ([#2155](https://github.com/encode/django-rest-framework/issues/2155))

+   `Client.logout()` 同时取消任何现有的 `force_authenticate`。 ([#2218](https://github.com/encode/django-rest-framework/issues/2218), [#2259](https://github.com/encode/django-rest-framework/issues/2259))

+   额外的断言和更好的检查以防止错误的序列化器 API 使用。 ([#2228](https://github.com/encode/django-rest-framework/issues/2228), [#2234](https://github.com/encode/django-rest-framework/issues/2234), [#2262](https://github.com/encode/django-rest-framework/issues/2262), [#2263](https://github.com/encode/django-rest-framework/issues/2263), [#2266](https://github.com/encode/django-rest-framework/issues/2266), [#2267](https://github.com/encode/django-rest-framework/issues/2267), [#2289](https://github.com/encode/django-rest-framework/issues/2289), [#2291](https://github.com/encode/django-rest-framework/issues/2291))

+   修复 `CharField` 的 `min_length` 消息。 ([#2255](https://github.com/encode/django-rest-framework/issues/2255))

+   修复可以在序列化器 `repr` 上发生的 `UnicodeDecodeError`。 ([#2270](https://github.com/encode/django-rest-framework/issues/2270), [#2279](https://github.com/encode/django-rest-framework/issues/2279))

+   修复当提供默认值时空HTML值的问题。 ([#2280](https://github.com/encode/django-rest-framework/issues/2280), [#2294](https://github.com/encode/django-rest-framework/issues/2294))

+   修复当作为多选输入时 `SlugRelatedField` 引发 `UnicodeEncodeError` 的行为。 ([#2290](https://github.com/encode/django-rest-framework/issues/2290))

**日期**：[2014年12月11日](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.1+Release%22)。

+   当默认序列化器 `create()` 失败时，提供更有用的错误消息。 ([#2013](https://github.com/encode/django-rest-framework/issues/2013))

+   在尝试保存序列化器时，如果数据无效，则引发错误。 ([#2098](https://github.com/encode/django-rest-framework/issues/2098))

+   修复`FileUploadParser`在空文件名和多个上传处理程序时的问题。([#2109](https://github.com/encode/django-rest-framework/issues/2109))

+   改进`BindingDict`以支持标准字典功能。([#2135](https://github.com/encode/django-rest-framework/issues/2135), [#2163](https://github.com/encode/django-rest-framework/issues/2163))

+   在`ListSerializer`中添加`validate()`。([#2168](https://github.com/encode/django-rest-framework/issues/2168), [#2225](https://github.com/encode/django-rest-framework/issues/2225), [#2232](https://github.com/encode/django-rest-framework/issues/2232))

+   修复JSONP渲染器未能转义某些字符的问题。([#2169](https://github.com/encode/django-rest-framework/issues/2169), [#2195](https://github.com/encode/django-rest-framework/issues/2195))

+   为`FileField`添加缺失的默认样式。([#2172](https://github.com/encode/django-rest-framework/issues/2172))

+   调用`ViewSet.as_view()`时需要执行操作。([#2175](https://github.com/encode/django-rest-framework/issues/2175))

+   在`ChoiceField`中添加`allow_blank`选项。([#2184](https://github.com/encode/django-rest-framework/issues/2184), [#2239](https://github.com/encode/django-rest-framework/issues/2239))

+   HTML渲染器中的美观修复。([#2187](https://github.com/encode/django-rest-framework/issues/2187))

+   如果序列化器上的`fields`不是字符串列表，则引发错误。([#2193](https://github.com/encode/django-rest-framework/issues/2193), [#2213](https://github.com/encode/django-rest-framework/issues/2213))

+   改进嵌套创建和更新的检查。([#2194](https://github.com/encode/django-rest-framework/issues/2194), [#2196](https://github.com/encode/django-rest-framework/issues/2196))

+   `Serializer`的`create()`/`update()`中的`validated_attrs`参数已重命名为`validated_data`。([#2197](https://github.com/encode/django-rest-framework/issues/2197))

+   删除已弃用代码以反映删除的Django版本。([#2200](https://github.com/encode/django-rest-framework/issues/2200))

+   对于嵌套写入，改进序列化器错误处理。([#2202](https://github.com/encode/django-rest-framework/issues/2202), [#2215](https://github.com/encode/django-rest-framework/issues/2215))

+   修复分页和自定义权限不兼容性问题。([#2205](https://github.com/encode/django-rest-framework/issues/2205))

+   如果序列化器上的`fields`不是字符串列表，则引发错误。([#2213](https://github.com/encode/django-rest-framework/issues/2213))

+   为关系字段添加缺失的翻译标记。([#2231](https://github.com/encode/django-rest-framework/issues/2231))

+   改进字典/映射的字段查找行为。([#2244](https://github.com/encode/django-rest-framework/issues/2244), [#2243](https://github.com/encode/django-rest-framework/issues/2243))

+   优化超链接的PK。([#2242](https://github.com/encode/django-rest-framework/issues/2242))

**日期**: 2014年12月1日

查看[3.0版本发布公告](../3.0-announcement/)以获取详细信息。

* * *

对于旧版本的发布说明，请查看[版本 2.x 文档](https://github.com/encode/django-rest-framework/blob/version-2.4.x/docs/topics/release-notes.md)。
