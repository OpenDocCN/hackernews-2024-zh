<!--yml
category: 未分类
date: 2024-05-27 15:03:28
-->

# Release Notes - Django REST framework

> 来源：[https://www.django-rest-framework.org/community/release-notes/#315x-series](https://www.django-rest-framework.org/community/release-notes/#315x-series)

Minor version numbers (0.0.x) are used for changes that are API compatible. You should be able to upgrade between minor point releases without any other code changes.

Medium version numbers (0.x.0) may include API changes, in line with the [deprecation policy](#deprecation-policy). You should read the release notes carefully before upgrading between medium point releases.

Major version numbers (x.0.0) are reserved for substantial project milestones.

REST framework releases follow a formal deprecation policy, which is in line with [Django's deprecation policy](https://docs.djangoproject.com/en/stable/internals/release-process/#internal-release-deprecation-policy).

The timeline for deprecation of a feature present in version 1.0 would work as follows:

*   Version 1.1 would remain **fully backwards compatible** with 1.0, but would raise `RemovedInDRF13Warning` warnings, subclassing `PendingDeprecationWarning`, if you use the feature that are due to be deprecated. These warnings are **silent by default**, but can be explicitly enabled when you're ready to start migrating any required changes. For example if you start running your tests using `python -Wd manage.py test`, you'll be warned of any API changes you need to make.

*   Version 1.2 would escalate these warnings to subclass `DeprecationWarning`, which is loud by default.

*   Version 1.3 would remove the deprecated bits of API entirely.

Note that in line with Django's policy, any parts of the framework not mentioned in the documentation should generally be considered private API, and may be subject to change.

To upgrade Django REST framework to the latest version, use pip:

```
pip install -U djangorestframework 
```

You can determine your currently installed version using `pip show`:

```
pip show djangorestframework 
```

* * *

Date: 22nd March 2024

*   Fix `SearchFilter` handling of quoted and comma separated strings, when `.get_search_terms` is being called into by a custom class. See [[#9338](https://github.com/encode/django-rest-framework/issues/9338)]
*   Revert number of 3.15.0 issues which included unintended side-effects. See [[#9331](https://github.com/encode/django-rest-framework/issues/9331)]

Date: 15th March 2024

*   Django 5.0 and Python 3.12 support [[#9157](https://github.com/encode/django-rest-framework/pull/9157)]
*   Use POST method instead of GET to perform logout in browsable API [[9208](https://github.com/encode/django-rest-framework/pull/9208)]
*   Added jQuery 3.7.1 support & dropped previous version [[#9094](https://github.com/encode/django-rest-framework/pull/9094)]
*   Use str as default path converter [[#9066](https://github.com/encode/django-rest-framework/pull/9066)]
*   Document support for http.HTTPMethod in the @action decorator added in Python 3.11 [[#9067](https://github.com/encode/django-rest-framework/pull/9067)]
*   Update exceptions.md [[#9071](https://github.com/encode/django-rest-framework/pull/9071)]
*   Partial serializer should not have required fields [[#7563](https://github.com/encode/django-rest-framework/pull/7563)]
*   Propagate 'default' from model field to serializer field. [[#9030](https://github.com/encode/django-rest-framework/pull/9030)]
*   Allow to override child.run_validation call in ListSerializer [[#8035](https://github.com/encode/django-rest-framework/pull/8035)]
*   Align SearchFilter behaviour to django.contrib.admin search [[#9017](https://github.com/encode/django-rest-framework/pull/9017)]
*   Class name added to unknown field error [[#9019](https://github.com/encode/django-rest-framework/pull/9019)]
*   Fix: Pagination response schemas. [[#9049](https://github.com/encode/django-rest-framework/pull/9049)]
*   Fix choices in ChoiceField to support IntEnum [[#8955](https://github.com/encode/django-rest-framework/pull/8955)]
*   Fix `SearchFilter` rendering search field with invalid value [[#9023](https://github.com/encode/django-rest-framework/pull/9023)]
*   Fix OpenAPI Schema yaml rendering for `timedelta` [[#9007](https://github.com/encode/django-rest-framework/pull/9007)]
*   Fix `NamespaceVersioning` ignoring `DEFAULT_VERSION` on non-None namespaces [[#7278](https://github.com/encode/django-rest-framework/pull/7278)]
*   Added Deprecation Warnings for CoreAPI [[#7519](https://github.com/encode/django-rest-framework/pull/7519)]
*   Removed usage of `field.choices` that triggered full table load [[#8950](https://github.com/encode/django-rest-framework/pull/8950)]
*   Permit mixed casing of string values for `BooleanField` validation [[#8970](https://github.com/encode/django-rest-framework/pull/8970)]
*   Fixes `BrowsableAPIRenderer` for usage with `ListSerializer`. [[#7530](https://github.com/encode/django-rest-framework/pull/7530)]
*   Change semantic of `OR` of two permission classes [[#7522](https://github.com/encode/django-rest-framework/pull/7522)]
*   Remove dependency on `pytz` [[#8984](https://github.com/encode/django-rest-framework/pull/8984)]
*   Make set_value a method within `Serializer` [[#8001](https://github.com/encode/django-rest-framework/pull/8001)]
*   Fix URLPathVersioning reverse fallback [[#7247](https://github.com/encode/django-rest-framework/pull/7247)]
*   Warn about Decimal type in min_value and max_value arguments of DecimalField [[#8972](https://github.com/encode/django-rest-framework/pull/8972)]
*   Fix mapping for choice values [[#8968](https://github.com/encode/django-rest-framework/pull/8968)]
*   Refactor read function to use context manager for file handling [[#8967](https://github.com/encode/django-rest-framework/pull/8967)]
*   Fix: fallback on CursorPagination ordering if unset on the view [[#8954](https://github.com/encode/django-rest-framework/pull/8954)]
*   Replaced `OrderedDict` with `dict` [[#8964](https://github.com/encode/django-rest-framework/pull/8964)]
*   Refactor get_field_info method to include max_digits and decimal_places attributes in SimpleMetadata class [[#8943](https://github.com/encode/django-rest-framework/pull/8943)]
*   Implement `__eq__` for validators [[#8925](https://github.com/encode/django-rest-framework/pull/8925)]
*   Ensure CursorPagination respects nulls in the ordering field [[#8912](https://github.com/encode/django-rest-framework/pull/8912)]
*   Use ZoneInfo as primary source of timezone data [[#8924](https://github.com/encode/django-rest-framework/pull/8924)]
*   Add username search field for TokenAdmin (#8927) [[#8934](https://github.com/encode/django-rest-framework/pull/8934)]
*   Handle Nested Relation in SlugRelatedField when many=False [[#8922](https://github.com/encode/django-rest-framework/pull/8922)]
*   Bump version of jQuery to 3.6.4 & updated ref links [[#8909](https://github.com/encode/django-rest-framework/pull/8909)]
*   Support UniqueConstraint [[#7438](https://github.com/encode/django-rest-framework/pull/7438)]
*   Allow Request, Response, Field, and GenericAPIView to be subscriptable. This allows the classes to be made generic for type checking. [[#8825](https://github.com/encode/django-rest-framework/pull/8825)]
*   Feat: Add some changes to ValidationError to support django style validation errors [[#8863](https://github.com/encode/django-rest-framework/pull/8863)]
*   Fix Respect `can_read_model` permission in DjangoModelPermissions [[#8009](https://github.com/encode/django-rest-framework/pull/8009)]
*   Add SimplePathRouter [[#6789](https://github.com/encode/django-rest-framework/pull/6789)]
*   Re-prefetch related objects after updating [[#8043](https://github.com/encode/django-rest-framework/pull/8043)]
*   Fix FilePathField required argument [[#8805](https://github.com/encode/django-rest-framework/pull/8805)]
*   Raise ImproperlyConfigured exception if `basename` is not unique [[#8438](https://github.com/encode/django-rest-framework/pull/8438)]
*   Use PrimaryKeyRelatedField pkfield in openapi [[#8315](https://github.com/encode/django-rest-framework/pull/8315)]
*   replace partition with split in BasicAuthentication [[#8790](https://github.com/encode/django-rest-framework/pull/8790)]
*   Fix BooleanField's allow_null behavior [[#8614](https://github.com/encode/django-rest-framework/pull/8614)]
*   Handle Django's ValidationErrors in ListField [[#6423](https://github.com/encode/django-rest-framework/pull/6423)]
*   Remove a bit of inline CSS. Add CSP nonce where it might be required and is available [[#8783](https://github.com/encode/django-rest-framework/pull/8783)]
*   Use autocomplete widget for user selection in Token admin [[#8534](https://github.com/encode/django-rest-framework/pull/8534)]
*   Make browsable API compatible with strong CSP [[#8784](https://github.com/encode/django-rest-framework/pull/8784)]
*   Avoid inline script execution for injecting CSRF token [[#7016](https://github.com/encode/django-rest-framework/pull/7016)]
*   Mitigate global dependency on inflection [[#8017](https://github.com/encode/django-rest-framework/pull/8017)] [[#8781](https://github.com/encode/django-rest-framework/pull/8781)]
*   Register Django urls [[#8778](https://github.com/encode/django-rest-framework/pull/8778)]
*   Implemented Verbose Name Translation for TokenProxy [[#8713](https://github.com/encode/django-rest-framework/pull/8713)]
*   Properly handle OverflowError in DurationField deserialization [[#8042](https://github.com/encode/django-rest-framework/pull/8042)]
*   Fix OpenAPI operation name plural appropriately [[#8017](https://github.com/encode/django-rest-framework/pull/8017)]
*   Represent SafeString as plain string on schema rendering [[#8429](https://github.com/encode/django-rest-framework/pull/8429)]
*   Fix #8771 - Checking for authentication even if `_ignore_model_permissions = True` [[#8772](https://github.com/encode/django-rest-framework/pull/8772)]
*   Fix 404 when page query parameter is empty string [[#8578](https://github.com/encode/django-rest-framework/pull/8578)]
*   Fixes instance check in ListSerializer.to_representation [[#8726](https://github.com/encode/django-rest-framework/pull/8726)] [[#8727](https://github.com/encode/django-rest-framework/pull/8727)]
*   FloatField will crash if the input is a number that is too big [[#8725](https://github.com/encode/django-rest-framework/pull/8725)]
*   Add missing DurationField to SimpleMetada label_lookup [[#8702](https://github.com/encode/django-rest-framework/pull/8702)]
*   Add support for Python 3.11 [[#8752](https://github.com/encode/django-rest-framework/pull/8752)]
*   Make request consistently available in pagination classes [[#8764](https://github.com/encode/django-rest-framework/pull/9764)]
*   Possibility to remove trailing zeros on DecimalFields representation [[#6514](https://github.com/encode/django-rest-framework/pull/6514)]
*   Add a method for getting serializer field name (OpenAPI) [[#7493](https://github.com/encode/django-rest-framework/pull/7493)]
*   Add `__eq__` method for `OperandHolder` class [[#8710](https://github.com/encode/django-rest-framework/pull/8710)]
*   Avoid importing `django.test` package when not testing [[#8699](https://github.com/encode/django-rest-framework/pull/8699)]
*   Preserve exception messages for wrapped Django exceptions [[#8051](https://github.com/encode/django-rest-framework/pull/8051)]
*   Include `examples` and `format` to OpenAPI schema of CursorPagination [[#8687](https://github.com/encode/django-rest-framework/pull/8687)] [[#8686](https://github.com/encode/django-rest-framework/pull/8686)]
*   Fix infinite recursion with deepcopy on Request [[#8684](https://github.com/encode/django-rest-framework/pull/8684)]
*   Refactor: Replace try/except with contextlib.suppress() [[#8676](https://github.com/encode/django-rest-framework/pull/8676)]
*   Minor fix to SerializeMethodField docstring [[#8629](https://github.com/encode/django-rest-framework/pull/8629)]
*   Minor refactor: Unnecessary use of list() function [[#8672](https://github.com/encode/django-rest-framework/pull/8672)]
*   Unnecessary list comprehension [[#8670](https://github.com/encode/django-rest-framework/pull/8670)]
*   Use correct class to indicate present deprecation [[#8665](https://github.com/encode/django-rest-framework/pull/8665)]

Date: 22nd September 2022

*   Django 2.2 is no longer supported. [[#8662](https://github.com/encode/django-rest-framework/pull/8662)]
*   Django 4.1 compatibility. [[#8591](https://github.com/encode/django-rest-framework/pull/8591)]
*   Add `--api-version` CLI option to `generateschema` management command. [[#8663](https://github.com/encode/django-rest-framework/pull/8663)]
*   Enforce `is_valid(raise_exception=False)` as a keyword-only argument. [[#7952](https://github.com/encode/django-rest-framework/pull/7952)]
*   Stop calling `set_context` on Validators. [[#8589](https://github.com/encode/django-rest-framework/pull/8589)]
*   Return `NotImplemented` from `ErrorDetails.__ne__`. [[#8538](https://github.com/encode/django-rest-framework/pull/8538)]
*   Don't evaluate `DateTimeField.default_timezone` when a custom timezone is set. [[#8531](https://github.com/encode/django-rest-framework/pull/8531)]
*   Make relative URLs clickable in Browsable API. [[#8464](https://github.com/encode/django-rest-framework/pull/8464)]
*   Support `ManyRelatedField` falling back to the default value when the attribute specified by dot notation doesn't exist. Matches `ManyRelatedField.get_attribute` to `Field.get_attribute`. [[#7574](https://github.com/encode/django-rest-framework/pull/7574)]
*   Make `schemas.openapi.get_reference` public. [[#7515](https://github.com/encode/django-rest-framework/pull/7515)]
*   Make `ReturnDict` support `dict` union operators on Python 3.9 and later. [[#8302](https://github.com/encode/django-rest-framework/pull/8302)]
*   Update throttling to check if `request.user` is set before checking if the user is authenticated. [[#8370](https://github.com/encode/django-rest-framework/pull/8370)]

Date: 15th December 2021

*   Revert schema naming changes with function based `@api_view`. [#8297]

Date: 13th December 2021

*   Django 4.0 compatibility. [#8178]
*   Add `max_length` and `min_length` options to `ListSerializer`. [#8165]
*   Add `get_request_serializer` and `get_response_serializer` hooks to `AutoSchema`. [#7424]
*   Fix OpenAPI representation of null-able read only fields. [#8116]
*   Respect `UNICODE_JSON` setting in API schema outputs. [#7991]
*   Fix for `RemoteUserAuthentication`. [#7158]
*   Make Field constructors keyword-only. [#7632]

* * *

Date: 26th March 2021

*   Revert use of `deque` instead of `list` for tracking throttling `.history`. (Due to incompatibility with DjangoRedis cache backend. See #7870) [#7872]

Date: 25th March 2021

*   Properly handle ATOMIC_REQUESTS when multiple database configurations are used. [#7739]
*   Bypass `COUNT` query when `LimitOffsetPagination` is configured but pagination params are not included on the request. [#6098]
*   Respect `allow_null=True` on `DecimalField`. [#7718]
*   Allow title cased `"Yes"`/`"No"` values with `BooleanField`. [#7739]
*   Add `PageNumberPagination.get_page_number()` method for overriding behavior. [#7652]
*   Fixed rendering of timedelta values in OpenAPI schemas, when present as default, min, or max fields. [#7641]
*   Render JSONFields with indentation in browsable API forms. [#6243]
*   Remove unnecessary database query in admin Token views. [#7852]
*   Raise validation errors when bools are passed to `PrimaryKeyRelatedField` fields, instead of casting to ints. [#7597]
*   Don't include model properties as automatically generated ordering fields with `OrderingFilter`. [#7609]
*   Use `deque` instead of `list` for tracking throttling `.history`. [#7849]

Date: 13th October 2020

*   Fix issue if `rest_framework.authtoken.models` is imported, but `rest_framework.authtoken` is not in INSTALLED_APPS. [#7571]
*   Ignore subclasses of BrowsableAPIRenderer in OpenAPI schema. [#7497]
*   Narrower exception catching in serilizer fields, to ensure that any errors in broken `get_queryset()` methods are not masked. [#7480]

Date: 28th September 2020

*   Add `TokenProxy` migration. [#7557]

Date: 28th September 2020

*   Add `--file` option to `generateschema` command. [#7130]
*   Support `tags` for OpenAPI schema generation. See [the schema docs](https://www.django-rest-framework.org/api-guide/schemas/#grouping-operations-with-tags). [#7184]
*   Support customising the operation ID for schema generation. See [the schema docs](https://www.django-rest-framework.org/api-guide/schemas/#operationid). [#7190]
*   Support OpenAPI components for schema generation. See [the schema docs](https://www.django-rest-framework.org/api-guide/schemas/#components). [#7124]
*   The following methods on `AutoSchema` become public API: `get_path_parameters`, `get_pagination_parameters`, `get_filter_parameters`, `get_request_body`, `get_responses`, `get_serializer`, `get_paginator`, `map_serializer`, `map_field`, `map_choice_field`, `map_field_validators`, `allows_filters`. See [the schema docs](https://www.django-rest-framework.org/api-guide/schemas/#autoschema)
*   Add support for Django 3.1's database-agnositic `JSONField`. [#7467]
*   `SearchFilter` now supports nested search on `JSONField` and `HStoreField` model fields. [#7121]
*   `SearchFilter` now supports searching on `annotate()` fields. [#6240]
*   The authtoken model no longer exposes the `pk` in the admin URL. [#7341]
*   Add `__repr__` for Request instances. [#7239]
*   UTF-8 decoding with Latin-1 fallback for basic auth credentials. [#7193]
*   CharField treats surrogate characters as a validation failure. [#7026]
*   Don't include callables as default values in schemas. [#7105]
*   Improve `ListField` schema output to include all available child information. [#7137]
*   Allow `default=False` to be included for `BooleanField` schema outputs. [#7165]
*   Include `"type"` information in `ChoiceField` schema outputs. [#7161]
*   Include `"type": "object"` on schema objects. [#7169]
*   Don't include component in schema output for DELETE requests. [#7229]
*   Fix schema types for `DecimalField`. [#7254]
*   Fix schema generation for `ObtainAuthToken` view. [#7211]
*   Support passing `context=...` to view `.get_serializer()` methods. [#7298]
*   Pass custom code to `PermissionDenied` if permission class has one set. [#7306]
*   Include "example" in schema pagination output. [#7275]
*   Default status code of 201 on schema output for POST requests. [#7206]
*   Use camelCase for operation IDs in schema output. [#7208]
*   Warn if duplicate operation IDs exist in schema output. [#7207]
*   Improve handling of decimal type when mapping `ChoiceField` to a schema output. [#7264]
*   Disable YAML aliases for OpenAPI schema outputs. [#7131]
*   Fix action URL names for APIs included under a namespaced URL. [#7287]
*   Update jQuery version from 3.4 to 3.5\. [#7313]
*   Fix `UniqueTogether` handling when serializer fields use `source=...`. [#7143]
*   HTTP `HEAD` requests now set `self.action` correctly on a ViewSet instance. [#7223]
*   Return a valid OpenAPI schema for the case where no API schema paths exist. [#7125]
*   Include tests in package distribution. [#7145]
*   Allow type checkers to support annotations like `ModelSerializer[Author]`. [#7385]
*   Don't include invalid `charset=None` portion in the request `Content-Type` header when using APIClient. [#7400]
*   Fix `\Z`/`\z` tokens in OpenAPI regexs. [#7389]
*   Fix `PrimaryKeyRelatedField` and `HyperlinkedRelatedField` when source field is actually a property. [#7142]
*   `Token.generate_key` is now a class method. [#7502]
*   `@action` warns if method is wrapped in a decorator that does not preserve information using `@functools.wraps`. [#7098]
*   Deprecate `serializers.NullBooleanField` in favour of `serializers.BooleanField` with `allow_null=True` [#7122]

* * *

**Date**: 30th September 2020

*   **Security**: Drop `urlize_quoted_links` template tag in favour of Django's built-in `urlize`. Removes a XSS vulnerability for some kinds of content in the browsable API.

**Date**: 5th August 2020

*   Fix compat with Django 3.1

**Date**: 12th December 2019

*   Drop `.set_context` API [in favour of a `requires_context` marker](../3.11-announcement/#validator-default-context).
*   Changed default widget for TextField with choices to select box. [#6892](https://github.com/encode/django-rest-framework/issues/6892)
*   Supported nested writes on non-relational fields, such as JSONField. [#6916](https://github.com/encode/django-rest-framework/issues/6916)
*   Include request/response media types in OpenAPI schemas, based on configured parsers/renderers. [#6865](https://github.com/encode/django-rest-framework/issues/6865)
*   Include operation descriptions in OpenAPI schemas, based on the docstring on the view. [#6898](https://github.com/encode/django-rest-framework/issues/6898)
*   Fix representation of serializers with all optional fields in OpenAPI schemas. [#6941](https://github.com/encode/django-rest-framework/issues/6941), [#6944](https://github.com/encode/django-rest-framework/issues/6944)
*   Fix representation of `serializers.HStoreField` in OpenAPI schemas. [#6914](https://github.com/encode/django-rest-framework/issues/6914)
*   Fix OpenAPI generation when title or version is not provided. [#6912](https://github.com/encode/django-rest-framework/issues/6912)
*   Use `int64` representation for large integers in OpenAPI schemas. [#7018](https://github.com/encode/django-rest-framework/issues/7018)
*   Improved error messages if no `.to_representation` implementation is provided on a field subclass. [#6996](https://github.com/encode/django-rest-framework/issues/6996)
*   Fix for serializer classes that use multiple inheritance. [#6980](https://github.com/encode/django-rest-framework/issues/6980)
*   Fix for reversing Hyperlinked URL fields with percent encoded components in the path. [#7059](https://github.com/encode/django-rest-framework/issues/7059)
*   Update bootstrap to 3.4.1\. [#6923](https://github.com/encode/django-rest-framework/issues/6923)

**Date**: 4th September 2019

*   Include API version in OpenAPI schema generation, defaulting to empty string.
*   Add pagination properties to OpenAPI response schemas.
*   Add missing "description" property to OpenAPI response schemas.
*   Only include "required" for non-empty cases in OpenAPI schemas.
*   Fix response schemas for "DELETE" case in OpenAPI schemas.
*   Use an array type for list view response schemas.
*   Use consistent `lowerInitialCamelCase` style in OpenAPI operation IDs.
*   Fix `minLength`/`maxLength`/`minItems`/`maxItems` properties in OpenAPI schemas.
*   Only call `FileField.url` once in serialization, for improved performance.
*   Fix an edge case where throttling calculations could error after a configuration change.

**Date**: 29th July 2019

*   Various `OpenAPI` schema fixes.
*   Ability to specify urlconf in include_docs_urls.

**Date**: 17th July 2019

*   Don't include autocomplete fields on TokenAuth admin, since it forces constraints on custom user models & admin.
*   Require `uritemplate` for OpenAPI schema generation, but not `coreapi`.

**Date**: [15th July 2019](https://github.com/encode/django-rest-framework/milestone/69?closed=1)

*   Switch to OpenAPI schema generation.
*   Drop Python 2 support.
*   Add `generateschema --generator_class` CLI option
*   Updated PyYaml dependency for OpenAPI schema generation to `pyyaml>=5.1` [#6680](https://github.com/encode/django-rest-framework/issues/6680)
*   Resolve DeprecationWarning with markdown. [#6317](https://github.com/encode/django-rest-framework/issues/6317)
*   Use `user.get_username` in templates, in preference to `user.username`.
*   Fix for cursor pagination issue that could occur after object deletions.
*   Fix for nullable fields with `source="*"`
*   Always apply all throttle classes during throttling checks.
*   Updates to jQuery and Markdown dependencies.
*   Don't strict disallow redundant `SerializerMethodField` field name arguments.
*   Don't render extra actions in browable API if not authenticated.
*   Strip null characters from search parameters.
*   Deprecate the `detail_route` decorator in favor of `action`, which accepts a `detail` bool. Use `@action(detail=True)` instead. [gh6687](https://github.com/encode/django-rest-framework/issues/6687)
*   Deprecate the `list_route` decorator in favor of `action`, which accepts a `detail` bool. Use `@action(detail=False)` instead. [gh6687](https://github.com/encode/django-rest-framework/issues/6687)

**Date**: 10th May 2019

This is a maintenance release that fixes an error handling bug under Python 2.

**Date**: 29th April 2019

This is the last Django REST Framework release that will support Python 2. Be sure to upgrade to Python 3 before upgrading to Django REST Framework 3.10.

*   Adjusted the compat check for django-guardian to allow the last guardian version (v1.4.9) compatible with Python 2\. [#6613](https://github.com/encode/django-rest-framework/issues/6613)

**Date**: [3rd March 2019](https://github.com/encode/django-rest-framework/milestone/71?closed=1)

*   Routers: invalidate `_urls` cache on `register()` [#6407](https://github.com/encode/django-rest-framework/issues/6407)
*   Deferred schema renderer creation to avoid requiring pyyaml. [#6416](https://github.com/encode/django-rest-framework/issues/6416)
*   Added 'request_forms' block to base.html [#6340](https://github.com/encode/django-rest-framework/issues/6340)
*   Fixed SchemaView to reset renderer on exception. [#6429](https://github.com/encode/django-rest-framework/issues/6429)
*   Update Django Guardian dependency. [#6430](https://github.com/encode/django-rest-framework/issues/6430)
*   Ensured support for Django 2.2 [#6422](https://github.com/encode/django-rest-framework/issues/6422) & [#6455](https://github.com/encode/django-rest-framework/issues/6455)
*   Made templates compatible with session-based CSRF. [#6207](https://github.com/encode/django-rest-framework/issues/6207)
*   Adjusted field `validators` to accept non-list iterables. [#6282](https://github.com/encode/django-rest-framework/issues/6282)
*   Added SearchFilter.get_search_fields() hook. [#6279](https://github.com/encode/django-rest-framework/issues/6279)
*   Fix DeprecationWarning when accessing collections.abc classes via collections [#6268](https://github.com/encode/django-rest-framework/issues/6268)
*   Allowed Q objects in limit_choices_to introspection. [#6472](https://github.com/encode/django-rest-framework/issues/6472)
*   Added lazy evaluation to composed permissions. [#6463](https://github.com/encode/django-rest-framework/issues/6463)
*   Add negation ~ operator to permissions composition [#6361](https://github.com/encode/django-rest-framework/issues/6361)
*   Avoided calling distinct on annotated fields in SearchFilter. [#6240](https://github.com/encode/django-rest-framework/issues/6240)
*   Introduced `RemovedInDRF…Warning` classes to simplify deprecations. [#6480](https://github.com/encode/django-rest-framework/issues/6480)

**Date**: [16th January 2019](https://github.com/encode/django-rest-framework/milestone/70?closed=1)

*   Resolve XSS issue in browsable API. [#6330](https://github.com/encode/django-rest-framework/issues/6330)
*   Upgrade Bootstrap to 3.4.0 to resolve XSS issue.
*   Resolve issues with composable permissions. [#6299](https://github.com/encode/django-rest-framework/issues/6299)
*   Respect `limit_choices_to` on foreign keys. [#6371](https://github.com/encode/django-rest-framework/issues/6371)

**Date**: [18th October 2018](https://github.com/encode/django-rest-framework/milestone/66?closed=1)

*   Improvements to ViewSet extra actions [#5605](https://github.com/encode/django-rest-framework/issues/5605)
*   Fix `action` support for ViewSet suffixes [#6081](https://github.com/encode/django-rest-framework/issues/6081)
*   Allow `action` docs sections [#6060](https://github.com/encode/django-rest-framework/issues/6060)
*   Deprecate the `Router.register` `base_name` argument in favor of `basename`. [#5990](https://github.com/encode/django-rest-framework/issues/5990)
*   Deprecate the `Router.get_default_base_name` method in favor of `Router.get_default_basename`. [#5990](https://github.com/encode/django-rest-framework/issues/5990)
*   Change `CharField` to disallow null bytes. [#6073](https://github.com/encode/django-rest-framework/issues/6073) To revert to the old behavior, subclass `CharField` and remove `ProhibitNullCharactersValidator` from the validators. `python class NullableCharField(serializers.CharField): def __init__(self, *args, **kwargs): super().__init__(*args, **kwargs) self.validators = [ v for v in self.validators if not isinstance(v, ProhibitNullCharactersValidator) ]`
*   Add `OpenAPIRenderer` and `generate_schema` management command. [#6229](https://github.com/encode/django-rest-framework/issues/6229)
*   Add OpenAPIRenderer by default, and add schema docs. [#6233](https://github.com/encode/django-rest-framework/issues/6233)
*   Allow permissions to be composed [#5753](https://github.com/encode/django-rest-framework/issues/5753)
*   Allow nullable BooleanField in Django 2.1 [#6183](https://github.com/encode/django-rest-framework/issues/6183)
*   Add testing of Python 3.7 support [#6141](https://github.com/encode/django-rest-framework/issues/6141)
*   Test using Django 2.1 final release. [#6109](https://github.com/encode/django-rest-framework/issues/6109)
*   Added djangorestframework-datatables to third-party packages [#5931](https://github.com/encode/django-rest-framework/issues/5931)
*   Change ISO 8601 date format to exclude year/month-only options [#5936](https://github.com/encode/django-rest-framework/issues/5936)
*   Update all pypi.python.org URLs to pypi.org [#5942](https://github.com/encode/django-rest-framework/issues/5942)
*   Ensure that html forms (multipart form data) respect optional fields [#5927](https://github.com/encode/django-rest-framework/issues/5927)
*   Allow hashing of ErrorDetail. [#5932](https://github.com/encode/django-rest-framework/issues/5932)
*   Correct schema parsing for JSONField [#5878](https://github.com/encode/django-rest-framework/issues/5878)
*   Render descriptions (from help_text) using safe [#5869](https://github.com/encode/django-rest-framework/issues/5869)
*   Removed input value from deault_error_message [#5881](https://github.com/encode/django-rest-framework/issues/5881)
*   Added min_value/max_value support in DurationField [#5643](https://github.com/encode/django-rest-framework/issues/5643)
*   Fixed instance being overwritten in pk-only optimization try/except block [#5747](https://github.com/encode/django-rest-framework/issues/5747)
*   Fixed AttributeError from items filter when value is None [#5981](https://github.com/encode/django-rest-framework/issues/5981)
*   Fixed Javascript `e.indexOf` is not a function error [#5982](https://github.com/encode/django-rest-framework/issues/5982)
*   Fix schemas for extra actions [#5992](https://github.com/encode/django-rest-framework/issues/5992)
*   Improved get_error_detail to use error_dict/error_list [#5785](https://github.com/encode/django-rest-framework/issues/5785)
*   Improved URLs in Admin renderer [#5988](https://github.com/encode/django-rest-framework/issues/5988)
*   Add "Community" section to docs, minor cleanup [#5993](https://github.com/encode/django-rest-framework/issues/5993)
*   Moved guardian imports out of compat [#6054](https://github.com/encode/django-rest-framework/issues/6054)
*   Deprecate the `DjangoObjectPermissionsFilter` class, moved to the `djangorestframework-guardian` package. [#6075](https://github.com/encode/django-rest-framework/issues/6075)
*   Drop Django 1.10 support [#5657](https://github.com/encode/django-rest-framework/issues/5657)
*   Only catch TypeError/ValueError for object lookups [#6028](https://github.com/encode/django-rest-framework/issues/6028)
*   Handle models without .objects manager in ModelSerializer. [#6111](https://github.com/encode/django-rest-framework/issues/6111)
*   Improve ModelSerializer.create() error message. [#6112](https://github.com/encode/django-rest-framework/issues/6112)
*   Fix CSRF cookie check failure when using session auth with django 1.11.6+ [#6113](https://github.com/encode/django-rest-framework/issues/6113)
*   Updated JWT docs. [#6138](https://github.com/encode/django-rest-framework/issues/6138)
*   Fix autoescape not getting passed to urlize_quoted_links filter [#6191](https://github.com/encode/django-rest-framework/issues/6191)

**Date**: [6th April 2018](https://github.com/encode/django-rest-framework/milestone/68?closed=1)

*   Fix `read_only` + `default` `unique_together` validation. [#5922](https://github.com/encode/django-rest-framework/issues/5922)
*   authtoken.views import coreapi from rest_framework.compat, not directly. [#5921](https://github.com/encode/django-rest-framework/issues/5921)
*   Docs: Add missing argument 'detail' to Route [#5920](https://github.com/encode/django-rest-framework/issues/5920)

**Date**: [4th April 2018](https://github.com/encode/django-rest-framework/milestone/67?closed=1)

*   Use old `url_name` behavior in route decorators [#5915](https://github.com/encode/django-rest-framework/issues/5915)

    For `list_route` and `detail_route` maintain the old behavior of `url_name`, basing it on the `url_path` instead of the function name.

**Date**: [3rd April 2018](https://github.com/encode/django-rest-framework/milestone/61?closed=1)

*   **Breaking Change**: Alter `read_only` plus `default` behaviour. [#5886](https://github.com/encode/django-rest-framework/issues/5886)

    `read_only` fields will now **always** be excluded from writable fields.

    Previously `read_only` fields with a `default` value would use the `default` for create and update operations.

    In order to maintain the old behaviour you may need to pass the value of `read_only` fields when calling `save()` in the view:

    ```
    def perform_create(self, serializer):
        serializer.save(owner=self.request.user) 
    ```

    Alternatively you may override `save()` or `create()` or `update()` on the serializer as appropriate.

*   Correct allow_null behaviour when required=False [#5888](https://github.com/encode/django-rest-framework/issues/5888)

    Without an explicit `default`, `allow_null` implies a default of `null` for outgoing serialization. Previously such fields were being skipped when read-only or otherwise not required.

    **Possible backwards compatibility break** if you were relying on such fields being excluded from the outgoing representation. In order to restore the old behaviour you can override `data` to exclude the field when `None`.

    For example:

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

*   Refactor dynamic route generation and improve viewset action introspectibility. [#5705](https://github.com/encode/django-rest-framework/issues/5705)

    `ViewSet`s have been provided with new attributes and methods that allow it to introspect its set of actions and the details of the current action.

    *   Merged `list_route` and `detail_route` into a single `action` decorator.
    *   Get all extra actions on a `ViewSet` with `.get_extra_actions()`.
    *   Extra actions now set the `url_name` and `url_path` on the decorated method.
    *   `url_name` is now based on the function name, instead of the `url_path`, as the path is not always suitable (e.g., capturing arguments in the path).
    *   Enable action url reversing through `.reverse_action()` method (added in 3.7.4)
    *   Example reverse call: `self.reverse_action(self.custom_action.url_name)`
    *   Add `detail` initkwarg to indicate if the current action is operating on a collection or a single instance.

    Additional changes:

    *   Deprecated `list_route` & `detail_route` in favor of `action` decorator with `detail` boolean.
    *   Deprecated dynamic list/detail route variants in favor of `DynamicRoute` with `detail` boolean.
    *   Refactored the router's dynamic route generation.
    *   `list_route` and `detail_route` maintain the old behavior of `url_name`, basing it on the `url_path` instead of the function name.
*   Fix formatting of the 3.7.4 release note [#5704](https://github.com/encode/django-rest-framework/issues/5704)

*   Docs: Update DRF Writable Nested Serializers references [#5711](https://github.com/encode/django-rest-framework/issues/5711)
*   Docs: Fixed typo in auth URLs example. [#5713](https://github.com/encode/django-rest-framework/issues/5713)
*   Improve composite field child errors [#5655](https://github.com/encode/django-rest-framework/issues/5655)
*   Disable HTML inputs for dict/list fields [#5702](https://github.com/encode/django-rest-framework/issues/5702)
*   Fix typo in HostNameVersioning doc [#5709](https://github.com/encode/django-rest-framework/issues/5709)
*   Use rsplit to get module and classname for imports [#5712](https://github.com/encode/django-rest-framework/issues/5712)
*   Formalize URLPatternsTestCase [#5703](https://github.com/encode/django-rest-framework/issues/5703)
*   Add exception translation test [#5700](https://github.com/encode/django-rest-framework/issues/5700)
*   Test staticfiles [#5701](https://github.com/encode/django-rest-framework/issues/5701)
*   Add drf-yasg to documentation and schema 3rd party packages [#5720](https://github.com/encode/django-rest-framework/issues/5720)
*   Remove unused `compat._resolve_model()` [#5733](https://github.com/encode/django-rest-framework/issues/5733)
*   Drop compat workaround for unsupported Python 3.2 [#5734](https://github.com/encode/django-rest-framework/issues/5734)
*   Prefer `iter(dict)` over `iter(dict.keys())` [#5736](https://github.com/encode/django-rest-framework/issues/5736)
*   Pass `python_requires` argument to setuptools [#5739](https://github.com/encode/django-rest-framework/issues/5739)
*   Remove unused links from docs [#5735](https://github.com/encode/django-rest-framework/issues/5735)
*   Prefer https protocol for links in docs when available [#5729](https://github.com/encode/django-rest-framework/issues/5729)
*   Add HStoreField, postgres fields tests [#5654](https://github.com/encode/django-rest-framework/issues/5654)
*   Always fully qualify ValidationError in docs [#5751](https://github.com/encode/django-rest-framework/issues/5751)
*   Remove unreachable code from ManualSchema [#5766](https://github.com/encode/django-rest-framework/issues/5766)
*   Allowed customising API documentation code samples [#5752](https://github.com/encode/django-rest-framework/issues/5752)
*   Updated docs to use `pip show` [#5757](https://github.com/encode/django-rest-framework/issues/5757)
*   Load 'static' instead of 'staticfiles' in templates [#5773](https://github.com/encode/django-rest-framework/issues/5773)
*   Fixed a typo in `fields` docs [#5783](https://github.com/encode/django-rest-framework/issues/5783)
*   Refer to "NamespaceVersioning" instead of "NamespacedVersioning" in the documentation [#5754](https://github.com/encode/django-rest-framework/issues/5754)
*   ErrorDetail: add `__eq__`/`__ne__` and `__repr__` [#5787](https://github.com/encode/django-rest-framework/issues/5787)
*   Replace `background-attachment: fixed` in docs [#5777](https://github.com/encode/django-rest-framework/issues/5777)
*   Make 404 & 403 responses consistent with `exceptions.APIException` output [#5763](https://github.com/encode/django-rest-framework/issues/5763)
*   Small fix to API documentation: schemas [#5796](https://github.com/encode/django-rest-framework/issues/5796)
*   Fix schema generation for PrimaryKeyRelatedField [#5764](https://github.com/encode/django-rest-framework/issues/5764)
*   Represent serializer DictField as an Object in schema [#5765](https://github.com/encode/django-rest-framework/issues/5765)
*   Added docs example reimplementing ObtainAuthToken [#5802](https://github.com/encode/django-rest-framework/issues/5802)
*   Add schema to the ObtainAuthToken view [#5676](https://github.com/encode/django-rest-framework/issues/5676)
*   Fix request formdata handling [#5800](https://github.com/encode/django-rest-framework/issues/5800)
*   Fix authtoken views imports [#5818](https://github.com/encode/django-rest-framework/issues/5818)
*   Update pytest, isort [#5815](https://github.com/encode/django-rest-framework/issues/5815) [#5817](https://github.com/encode/django-rest-framework/issues/5817) [#5894](https://github.com/encode/django-rest-framework/issues/5894)
*   Fixed active timezone handling for non ISO8601 datetimes. [#5833](https://github.com/encode/django-rest-framework/issues/5833)
*   Made TemplateHTMLRenderer render IntegerField inputs when value is `0`. [#5834](https://github.com/encode/django-rest-framework/issues/5834)
*   Corrected endpoint in tutorial instructions [#5835](https://github.com/encode/django-rest-framework/issues/5835)
*   Add Django Rest Framework Role Filters to Third party packages [#5809](https://github.com/encode/django-rest-framework/issues/5809)
*   Use single copy of static assets. Update jQuery [#5823](https://github.com/encode/django-rest-framework/issues/5823)
*   Changes ternary conditionals to be PEP308 compliant [#5827](https://github.com/encode/django-rest-framework/issues/5827)
*   Added links to 'A Todo List API with React' and 'Blog API' tutorials [#5837](https://github.com/encode/django-rest-framework/issues/5837)
*   Fix comment typo in ModelSerializer [#5844](https://github.com/encode/django-rest-framework/issues/5844)
*   Add admin to installed apps to avoid test failures. [#5870](https://github.com/encode/django-rest-framework/issues/5870)
*   Fixed schema for UUIDField in SimpleMetadata. [#5872](https://github.com/encode/django-rest-framework/issues/5872)
*   Corrected docs on router include with namespaces. [#5843](https://github.com/encode/django-rest-framework/issues/5843)
*   Test using model objects for dotted source default [#5880](https://github.com/encode/django-rest-framework/issues/5880)
*   Allow traversing nullable related fields [#5849](https://github.com/encode/django-rest-framework/issues/5849)
*   Added: Tutorial: Django REST with React (Django 2.0) [#5891](https://github.com/encode/django-rest-framework/issues/5891)
*   Add `LimitOffsetPagination.get_count` to allow method override [#5846](https://github.com/encode/django-rest-framework/issues/5846)
*   Don't show hidden fields in metadata [#5854](https://github.com/encode/django-rest-framework/issues/5854)
*   Enable OrderingFilter to handle an empty tuple (or list) for the 'ordering' field. [#5899](https://github.com/encode/django-rest-framework/issues/5899)
*   Added generic 500 and 400 JSON error handlers. [#5904](https://github.com/encode/django-rest-framework/issues/5904)

**Date**: [21st December 2017](https://github.com/encode/django-rest-framework/milestone/65?closed=1)

*   Fix typo to include *.mo locale files to packaging. [#5697](https://github.com/encode/django-rest-framework/issues/5697), [#5695](https://github.com/encode/django-rest-framework/issues/5695)

**Date**: [21st December 2017](https://github.com/encode/django-rest-framework/milestone/64?closed=1)

*   Add missing *.ico icon files to packaging.

**Date**: [21st December 2017](https://github.com/encode/django-rest-framework/milestone/63?closed=1)

*   Add missing *.woff2 font files to packaging. [#5692](https://github.com/encode/django-rest-framework/issues/5692)
*   Add missing *.mo locale files to packaging. [#5695](https://github.com/encode/django-rest-framework/issues/5695), [#5696](https://github.com/encode/django-rest-framework/issues/5696)

**Date**: [20th December 2017](https://github.com/encode/django-rest-framework/milestone/62?closed=1)

*   Schema: Extract method for `manual_fields` processing [#5633](https://github.com/encode/django-rest-framework/issues/5633)

    Allows for easier customisation of `manual_fields` processing, for example to provide per-method manual fields. `AutoSchema` adds `get_manual_fields`, as the intended override point, and a utility method `update_fields`, to handle by-name field replacement from a list, which, in general, you are not expected to override.

    Note: `AutoSchema.__init__` now ensures `manual_fields` is a list. Previously may have been stored internally as `None`.

*   Remove ulrparse compatibility shim; use six instead [#5579](https://github.com/encode/django-rest-framework/issues/5579)

*   Drop compat wrapper for `TimeDelta.total_seconds()` [#5577](https://github.com/encode/django-rest-framework/issues/5577)
*   Clean up all whitespace throughout project [#5578](https://github.com/encode/django-rest-framework/issues/5578)
*   Compat cleanup [#5581](https://github.com/encode/django-rest-framework/issues/5581)
*   Add pygments CSS block in browsable API views [#5584](https://github.com/encode/django-rest-framework/issues/5584) [#5587](https://github.com/encode/django-rest-framework/issues/5587)
*   Remove `set_rollback()` from compat [#5591](https://github.com/encode/django-rest-framework/issues/5591)
*   Fix request body/POST access [#5590](https://github.com/encode/django-rest-framework/issues/5590)
*   Rename test to reference correct issue [#5610](https://github.com/encode/django-rest-framework/issues/5610)
*   Documentation Fixes [#5611](https://github.com/encode/django-rest-framework/issues/5611) [#5612](https://github.com/encode/django-rest-framework/issues/5612)
*   Remove references to unsupported Django versions in docs and code [#5602](https://github.com/encode/django-rest-framework/issues/5602)
*   Test Serializer exclude for declared fields [#5599](https://github.com/encode/django-rest-framework/issues/5599)
*   Fixed schema generation for filter backends [#5613](https://github.com/encode/django-rest-framework/issues/5613)
*   Minor cleanup for ModelSerializer tests [#5598](https://github.com/encode/django-rest-framework/issues/5598)
*   Reimplement request attribute access w/ `__getattr__` [#5617](https://github.com/encode/django-rest-framework/issues/5617)
*   Fixed SchemaJSRenderer renders invalid Javascript [#5607](https://github.com/encode/django-rest-framework/issues/5607)
*   Make Django 2.0 support official/explicit [#5619](https://github.com/encode/django-rest-framework/issues/5619)
*   Perform type check on passed request argument [#5618](https://github.com/encode/django-rest-framework/issues/5618)
*   Fix AttributeError hiding on request authenticators [#5600](https://github.com/encode/django-rest-framework/issues/5600)
*   Update test requirements [#5626](https://github.com/encode/django-rest-framework/issues/5626)
*   Docs: `Serializer._declared_fields` enable modifying fields on a serializer [#5629](https://github.com/encode/django-rest-framework/issues/5629)
*   Fix packaging [#5624](https://github.com/encode/django-rest-framework/issues/5624)
*   Fix readme rendering for PyPI, add readme build to CI [#5625](https://github.com/encode/django-rest-framework/issues/5625)
*   Update tutorial [#5622](https://github.com/encode/django-rest-framework/issues/5622)
*   Non-required fields with `allow_null=True` should not imply a default value [#5639](https://github.com/encode/django-rest-framework/issues/5639)
*   Docs: Add `allow_null` serialization output note [#5641](https://github.com/encode/django-rest-framework/issues/5641)
*   Update to use the Django 2.0 release in tox.ini [#5645](https://github.com/encode/django-rest-framework/issues/5645)
*   Fix `Serializer.data` for Browsable API rendering when provided invalid `data` [#5646](https://github.com/encode/django-rest-framework/issues/5646)
*   Docs: Note AutoSchema limitations on bare APIView [#5649](https://github.com/encode/django-rest-framework/issues/5649)
*   Add `.basename` and `.reverse_action()` to ViewSet [#5648](https://github.com/encode/django-rest-framework/issues/5648)
*   Docs: Fix typos in serializers documentation [#5652](https://github.com/encode/django-rest-framework/issues/5652)
*   Fix `override_settings` compat [#5668](https://github.com/encode/django-rest-framework/issues/5668)
*   Add DEFAULT_SCHEMA_CLASS setting [#5658](https://github.com/encode/django-rest-framework/issues/5658)
*   Add docs note re generated BooleanField being `required=False` [#5665](https://github.com/encode/django-rest-framework/issues/5665)
*   Add 'dist' build [#5656](https://github.com/encode/django-rest-framework/issues/5656)
*   Fix typo in docstring [#5678](https://github.com/encode/django-rest-framework/issues/5678)
*   Docs: Add `UNAUTHENTICATED_USER = None` note [#5679](https://github.com/encode/django-rest-framework/issues/5679)
*   Update OPTIONS example from “Documenting Your API” [#5680](https://github.com/encode/django-rest-framework/issues/5680)
*   Docs: Add note on object permissions for FBVs [#5681](https://github.com/encode/django-rest-framework/issues/5681)
*   Docs: Add example to `to_representation` docs [#5682](https://github.com/encode/django-rest-framework/issues/5682)
*   Add link to Classy DRF in docs [#5683](https://github.com/encode/django-rest-framework/issues/5683)
*   Document ViewSet.action [#5685](https://github.com/encode/django-rest-framework/issues/5685)
*   Fix schema docs typo [#5687](https://github.com/encode/django-rest-framework/issues/5687)
*   Fix URL pattern parsing in schema generation [#5689](https://github.com/encode/django-rest-framework/issues/5689)
*   Add example using `source=‘*’` to custom field docs. [#5688](https://github.com/encode/django-rest-framework/issues/5688)
*   Fix format_suffix_patterns behavior with Django 2 path() routes [#5691](https://github.com/encode/django-rest-framework/issues/5691)

**Date**: [6th November 2017](https://github.com/encode/django-rest-framework/milestone/60?closed=1)

*   Fix `AppRegistryNotReady` error from contrib.auth view imports [#5567](https://github.com/encode/django-rest-framework/issues/5567)

**Date**: [6th November 2017](https://github.com/encode/django-rest-framework/milestone/59?closed=1)

*   Fixed Django 2.1 compatibility due to removal of django.contrib.auth.login()/logout() views. [#5510](https://github.com/encode/django-rest-framework/issues/5510)
*   Add missing import for TextLexer. [#5512](https://github.com/encode/django-rest-framework/issues/5512)
*   Adding examples and documentation for caching [#5514](https://github.com/encode/django-rest-framework/issues/5514)
*   Include date and date-time format for schema generation [#5511](https://github.com/encode/django-rest-framework/issues/5511)
*   Use triple backticks for markdown code blocks [#5513](https://github.com/encode/django-rest-framework/issues/5513)
*   Interactive docs - make bottom sidebar items sticky [#5516](https://github.com/encode/django-rest-framework/issues/5516)
*   Clarify pagination system check [#5524](https://github.com/encode/django-rest-framework/issues/5524)
*   Stop JSONBoundField mangling invalid JSON [#5527](https://github.com/encode/django-rest-framework/issues/5527)
*   Have JSONField render as textarea in Browsable API [#5530](https://github.com/encode/django-rest-framework/issues/5530)
*   Schema: Exclude OPTIONS/HEAD for ViewSet actions [#5532](https://github.com/encode/django-rest-framework/issues/5532)
*   Fix ordering for dotted sources [#5533](https://github.com/encode/django-rest-framework/issues/5533)
*   Fix: Fields with `allow_null=True` should imply a default serialization value [#5518](https://github.com/encode/django-rest-framework/issues/5518)
*   Ensure Location header is strictly a 'str', not subclass. [#5544](https://github.com/encode/django-rest-framework/issues/5544)
*   Add import to example in api-guide/parsers [#5547](https://github.com/encode/django-rest-framework/issues/5547)
*   Catch OverflowError for "out of range" datetimes [#5546](https://github.com/encode/django-rest-framework/issues/5546)
*   Add djangorestframework-rapidjson to third party packages [#5549](https://github.com/encode/django-rest-framework/issues/5549)
*   Increase test coverage for `drf_create_token` command [#5550](https://github.com/encode/django-rest-framework/issues/5550)
*   Add trove classifier for Python 3.6 support. [#5555](https://github.com/encode/django-rest-framework/issues/5555)
*   Add pip cache support to the Travis CI configuration [#5556](https://github.com/encode/django-rest-framework/issues/5556)
*   Rename [`wheel`] section to [`bdist_wheel`] as the former is legacy [#5557](https://github.com/encode/django-rest-framework/issues/5557)
*   Fix invalid escape sequence deprecation warnings [#5560](https://github.com/encode/django-rest-framework/issues/5560)
*   Add interactive docs error template [#5548](https://github.com/encode/django-rest-framework/issues/5548)
*   Add rounding parameter to DecimalField [#5562](https://github.com/encode/django-rest-framework/issues/5562)
*   Fix all BytesWarning caught during tests [#5561](https://github.com/encode/django-rest-framework/issues/5561)
*   Use dict and set literals instead of calls to dict() and set() [#5559](https://github.com/encode/django-rest-framework/issues/5559)
*   Change ImageField validation pattern, use validators from DjangoImageField [#5539](https://github.com/encode/django-rest-framework/issues/5539)
*   Fix processing unicode symbols in query_string by Python 2 [#5552](https://github.com/encode/django-rest-framework/issues/5552)

**Date**: [16th October 2017](https://github.com/encode/django-rest-framework/milestone/58?closed=1)

*   Fix Interactive documentation always uses false for boolean fields in requests [#5492](https://github.com/encode/django-rest-framework/issues/5492)
*   Improve compatibility with Django 2.0 alpha. [#5500](https://github.com/encode/django-rest-framework/issues/5500) [#5503](https://github.com/encode/django-rest-framework/issues/5503)
*   Improved handling of schema naming collisions [#5486](https://github.com/encode/django-rest-framework/issues/5486)
*   Added additional docs and tests around providing a default value for dotted `source` fields [#5489](https://github.com/encode/django-rest-framework/issues/5489)

**Date**: [6th October 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.7.0+Release%22)

*   Fix `DjangoModelPermissions` to ensure user authentication before calling the view's `get_queryset()` method. As a side effect, this changes the order of the HTTP method permissions and authentication checks, and 405 responses will only be returned when authenticated. If you want to replicate the old behavior, see the PR for details. [#5376](https://github.com/encode/django-rest-framework/issues/5376)
*   Deprecated `exclude_from_schema` on `APIView` and `api_view` decorator. Set `schema = None` or `@schema(None)` as appropriate. [#5422](https://github.com/encode/django-rest-framework/issues/5422)
*   Timezone-aware `DateTimeField`s now respect active or default `timezone` during serialization, instead of always using UTC. [#5435](https://github.com/encode/django-rest-framework/issues/5435)

    Resolves inconsistency whereby instances were serialized with supplied datetime for `create` but UTC for `retrieve`. [#3732](https://github.com/encode/django-rest-framework/issues/3732)

    **Possible backwards compatibility break** if you were relying on datetime strings being UTC. Have client interpret datetimes or [set default or active timezone (docs)](https://docs.djangoproject.com/en/1.11/topics/i18n/timezones/#default-time-zone-and-current-time-zone) to UTC if needed.

*   Removed DjangoFilterBackend inline with deprecation policy. Use `django_filters.rest_framework.FilterSet` and/or `django_filters.rest_framework.DjangoFilterBackend` instead. [#5273](https://github.com/encode/django-rest-framework/issues/5273)

*   Don't strip microseconds from `time` when encoding. Makes consistent with `datetime`. **BC Change**: Previously only milliseconds were encoded. [#5440](https://github.com/encode/django-rest-framework/issues/5440)
*   Added `STRICT_JSON` setting (default `True`) to raise exception for the extended float values (`nan`, `inf`, `-inf`) accepted by Python's `json` module. **BC Change**: Previously these values would converted to corresponding strings. Set `STRICT_JSON` to `False` to restore the previous behaviour. [#5265](https://github.com/encode/django-rest-framework/issues/5265)
*   Add support for `page_size` parameter in CursorPaginator class [#5250](https://github.com/encode/django-rest-framework/issues/5250)
*   Make `DEFAULT_PAGINATION_CLASS` `None` by default. **BC Change**: If your were **just** setting `PAGE_SIZE` to enable pagination you will need to add `DEFAULT_PAGINATION_CLASS`. The previous default was `rest_framework.pagination.PageNumberPagination`. There is a system check warning to catch this case. You may silence that if you are setting pagination class on a per-view basis. [#5170](https://github.com/encode/django-rest-framework/issues/5170)
*   Catch `APIException` from `get_serializer_fields` in schema generation. [#5443](https://github.com/encode/django-rest-framework/issues/5443)
*   Allow custom authentication and permission classes when using `include_docs_urls` [#5448](https://github.com/encode/django-rest-framework/issues/5448)
*   Defer translated string evaluation on validators. [#5452](https://github.com/encode/django-rest-framework/issues/5452)
*   Added default value for 'detail' param into 'ValidationError' exception [#5342](https://github.com/encode/django-rest-framework/issues/5342)
*   Adjust schema get_filter_fields rules to match framework [#5454](https://github.com/encode/django-rest-framework/issues/5454)
*   Updated test matrix to add Django 2.0 and drop Django 1.8 & 1.9 **BC Change**: This removes Django 1.8 and Django 1.9 from Django REST Framework supported versions. [#5457](https://github.com/encode/django-rest-framework/issues/5457)
*   Fixed a deprecation warning in serializers.ModelField [#5058](https://github.com/encode/django-rest-framework/issues/5058)
*   Added a more explicit error message when `get_queryset` returned `None` [#5348](https://github.com/encode/django-rest-framework/issues/5348)
*   Fix docs for Response `data` description [#5361](https://github.com/encode/django-rest-framework/issues/5361)
*   Fix **pycache**/.pyc excludes when packaging [#5373](https://github.com/encode/django-rest-framework/issues/5373)
*   Fix default value handling for dotted sources [#5375](https://github.com/encode/django-rest-framework/issues/5375)
*   Ensure content_type is set when passing empty body to RequestFactory [#5351](https://github.com/encode/django-rest-framework/issues/5351)
*   Fix ErrorDetail Documentation [#5380](https://github.com/encode/django-rest-framework/issues/5380)
*   Allow optional content in the generic content form [#5372](https://github.com/encode/django-rest-framework/issues/5372)
*   Updated supported values for the NullBooleanField [#5387](https://github.com/encode/django-rest-framework/issues/5387)
*   Fix ModelSerializer custom named fields with source on model [#5388](https://github.com/encode/django-rest-framework/issues/5388)
*   Fixed the MultipleFieldLookupMixin documentation example to properly check for object level permission [#5398](https://github.com/encode/django-rest-framework/issues/5398)
*   Update get_object() example in permissions.md [#5401](https://github.com/encode/django-rest-framework/issues/5401)
*   Fix authtoken management command [#5415](https://github.com/encode/django-rest-framework/issues/5415)
*   Fix schema generation markdown [#5421](https://github.com/encode/django-rest-framework/issues/5421)
*   Allow `ChoiceField.choices` to be set dynamically [#5426](https://github.com/encode/django-rest-framework/issues/5426)
*   Add the project layout to the quickstart [#5434](https://github.com/encode/django-rest-framework/issues/5434)
*   Reuse 'apply_markdown' function in 'render_markdown' templatetag [#5469](https://github.com/encode/django-rest-framework/issues/5469)
*   Added links to `drf-openapi` package in docs [#5470](https://github.com/encode/django-rest-framework/issues/5470)
*   Added docstrings code highlighting with pygments [#5462](https://github.com/encode/django-rest-framework/issues/5462)
*   Fixed documentation rendering for views named `data` [#5472](https://github.com/encode/django-rest-framework/issues/5472)
*   Docs: Clarified 'to_internal_value()' validation behavior [#5466](https://github.com/encode/django-rest-framework/issues/5466)
*   Fix missing six.text_type() call on APIException.**str** [#5476](https://github.com/encode/django-rest-framework/issues/5476)
*   Document documentation.py [#5478](https://github.com/encode/django-rest-framework/issues/5478)
*   Fix naming collisions in Schema Generation [#5464](https://github.com/encode/django-rest-framework/issues/5464)
*   Call Django's authenticate function with the request object [#5295](https://github.com/encode/django-rest-framework/issues/5295)
*   Update coreapi JS to 0.1.1 [#5479](https://github.com/encode/django-rest-framework/issues/5479)
*   Have `is_list_view` recognise RetrieveModel… views [#5480](https://github.com/encode/django-rest-framework/issues/5480)
*   Remove Django 1.8 & 1.9 compatibility code [#5481](https://github.com/encode/django-rest-framework/issues/5481)
*   Remove deprecated schema code from DefaultRouter [#5482](https://github.com/encode/django-rest-framework/issues/5482)
*   Refactor schema generation to allow per-view customisation. **BC Change**: `SchemaGenerator.get_serializer_fields` has been refactored as `AutoSchema.get_serializer_fields` and drops the `view` argument [#5354][gh5354]

**Date**: [21st August 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.4+Release%22)

*   Ignore any invalidly formed query parameters for OrderingFilter. [#5131](https://github.com/encode/django-rest-framework/issues/5131)
*   Improve memory footprint when reading large JSON requests. [#5147](https://github.com/encode/django-rest-framework/issues/5147)
*   Fix schema generation for pagination. [#5161](https://github.com/encode/django-rest-framework/issues/5161)
*   Fix exception when `HTML_CUTOFF` is set to `None`. [#5174](https://github.com/encode/django-rest-framework/issues/5174)
*   Fix browsable API not supporting `multipart/form-data` correctly. [#5176](https://github.com/encode/django-rest-framework/issues/5176)
*   Fixed `test_hyperlinked_related_lookup_url_encoded_exists`. [#5179](https://github.com/encode/django-rest-framework/issues/5179)
*   Make sure max_length is in FileField kwargs. [#5186](https://github.com/encode/django-rest-framework/issues/5186)
*   Fix `list_route` & `detail_route` with kwargs contains curly bracket in `url_path` [#5187](https://github.com/encode/django-rest-framework/issues/5187)
*   Add Django manage command to create a DRF user Token. [#5188](https://github.com/encode/django-rest-framework/issues/5188)
*   Ensure API documentation templates do not check for user authentication [#5162](https://github.com/encode/django-rest-framework/issues/5162)
*   Fix special case where OneToOneField is also primary key. [#5192](https://github.com/encode/django-rest-framework/issues/5192)
*   Added aria-label and a new region for accessibility purposes in base.html [#5196](https://github.com/encode/django-rest-framework/issues/5196)
*   Quote nested API parameters in api.js. [#5214](https://github.com/encode/django-rest-framework/issues/5214)
*   Set ViewSet args/kwargs/request before dispatch. [#5229](https://github.com/encode/django-rest-framework/issues/5229)
*   Added unicode support to SlugField. [#5231](https://github.com/encode/django-rest-framework/issues/5231)
*   Fix HiddenField appears in Raw Data form initial content. [#5259](https://github.com/encode/django-rest-framework/issues/5259)
*   Raise validation error on invalid timezone parsing. [#5261](https://github.com/encode/django-rest-framework/issues/5261)
*   Fix SearchFilter to-many behavior/performance. [#5264](https://github.com/encode/django-rest-framework/issues/5264)
*   Simplified chained comparisons and minor code fixes. [#5276](https://github.com/encode/django-rest-framework/issues/5276)
*   RemoteUserAuthentication, docs, and tests. [#5306](https://github.com/encode/django-rest-framework/issues/5306)
*   Revert "Cached the field's root and context property" [#5313](https://github.com/encode/django-rest-framework/issues/5313)
*   Fix introspection of list field in schema. [#5326](https://github.com/encode/django-rest-framework/issues/5326)
*   Fix interactive docs for multiple nested and extra methods. [#5334](https://github.com/encode/django-rest-framework/issues/5334)
*   Fix/remove undefined template var "schema" [#5346](https://github.com/encode/django-rest-framework/issues/5346)

**Date**: [12th May 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.3+Release%22)

*   Raise 404 if a URL lookup results in ValidationError. ([#5126](https://github.com/encode/django-rest-framework/issues/5126))
*   Honor http_method_names on class based view, when generating API schemas. ([#5085](https://github.com/encode/django-rest-framework/issues/5085))
*   Allow overridden `get_limit` in LimitOffsetPagination to return all records. ([#4437](https://github.com/encode/django-rest-framework/issues/4437))
*   Fix partial update for the ListSerializer. ([#4222](https://github.com/encode/django-rest-framework/issues/4222))
*   Render JSONField control correctly in browsable API. ([#4999](https://github.com/encode/django-rest-framework/issues/4999), [#5042](https://github.com/encode/django-rest-framework/issues/5042))
*   Raise validation errors for invalid datetime in given timezone. ([#4987](https://github.com/encode/django-rest-framework/issues/4987))
*   Support restricting doc & schema shortcuts to a subset of urls. ([#4979](https://github.com/encode/django-rest-framework/issues/4979))
*   Resolve SchemaGenerator error with paginators that have no `page_size` attribute. ([#5086](https://github.com/encode/django-rest-framework/issues/5086), [#3692](https://github.com/encode/django-rest-framework/issues/3692))
*   Resolve HyperlinkedRelatedField exception on string with %20 instead of space. ([#4748](https://github.com/encode/django-rest-framework/issues/4748), [#5078](https://github.com/encode/django-rest-framework/issues/5078))
*   Customizable schema generator classes. ([#5082](https://github.com/encode/django-rest-framework/issues/5082))
*   Update existing vary headers in response instead of overwriting them. ([#5047](https://github.com/encode/django-rest-framework/issues/5047))
*   Support passing `.as_view()` to view instance. ([#5053](https://github.com/encode/django-rest-framework/issues/5053))
*   Use correct exception handler when settings overridden on a view. ([#5055](https://github.com/encode/django-rest-framework/issues/5055), [#5054](https://github.com/encode/django-rest-framework/issues/5054))
*   Update Boolean field to support 'yes' and 'no' values. ([#5038](https://github.com/encode/django-rest-framework/issues/5038))
*   Fix unique validator for ChoiceField. ([#5004](https://github.com/encode/django-rest-framework/issues/5004), [#5026](https://github.com/encode/django-rest-framework/issues/5026), [#5028](https://github.com/encode/django-rest-framework/issues/5028))
*   JavaScript cleanups in API Docs. ([#5001](https://github.com/encode/django-rest-framework/issues/5001))
*   Include URL path regexs in API schemas where valid. ([#5014](https://github.com/encode/django-rest-framework/issues/5014))
*   Correctly set scheme in coreapi TokenAuthentication. ([#5000](https://github.com/encode/django-rest-framework/issues/5000), [#4994](https://github.com/encode/django-rest-framework/issues/4994))
*   HEAD requests on ViewSets should not return 405\. ([#4705](https://github.com/encode/django-rest-framework/issues/4705), [#4973](https://github.com/encode/django-rest-framework/issues/4973), [#4864](https://github.com/encode/django-rest-framework/issues/4864))
*   Support usage of 'source' in `extra_kwargs`. ([#4688](https://github.com/encode/django-rest-framework/issues/4688))
*   Fix invalid content type for schema.js ([#4968](https://github.com/encode/django-rest-framework/issues/4968))
*   Fix DjangoFilterBackend inheritance issues. ([#5089](https://github.com/encode/django-rest-framework/issues/5089), [#5117](https://github.com/encode/django-rest-framework/issues/5117))

**Date**: [10th March 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.2+Release%22)

*   Support for Safari & IE in API docs. ([#4959](https://github.com/encode/django-rest-framework/issues/4959), [#4961](https://github.com/encode/django-rest-framework/issues/4961))
*   Add missing `mark_safe` in API docs template tags. ([#4952](https://github.com/encode/django-rest-framework/issues/4952), [#4953](https://github.com/encode/django-rest-framework/issues/4953))
*   Add missing glyphicon fonts. ([#4950](https://github.com/encode/django-rest-framework/issues/4950), [#4951](https://github.com/encode/django-rest-framework/issues/4951))
*   Fix One-to-one fields in API docs. ([#4955](https://github.com/encode/django-rest-framework/issues/4955), [#4956](https://github.com/encode/django-rest-framework/issues/4956))
*   Test clean ups. ([#4949](https://github.com/encode/django-rest-framework/issues/4949))

**Date**: [9th March 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.1+Release%22)

*   Ensure `markdown` dependency is optional. ([#4947](https://github.com/encode/django-rest-framework/issues/4947))

**Date**: [9th March 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.6.0+Release%22)

See the [release announcement](../3.6-announcement/).

* * *

**Date**: [10th February 2017](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.4+Release%22)

*   Add max_length and min_length arguments for ListField. ([#4877](https://github.com/encode/django-rest-framework/issues/4877))
*   Add per-view custom exception handler support. ([#4753](https://github.com/encode/django-rest-framework/issues/4753))
*   Support disabling of declared fields on serializer subclasses. ([#4764](https://github.com/encode/django-rest-framework/issues/4764))
*   Support custom view names on `@list_route` and `@detail_route` endpoints. ([#4821](https://github.com/encode/django-rest-framework/issues/4821))
*   Correct labels for fields in login template when custom user model is used. ([#4841](https://github.com/encode/django-rest-framework/issues/4841))
*   Whitespace fixes for descriptions generated from docstrings. ([#4759](https://github.com/encode/django-rest-framework/issues/4759), [#4869](https://github.com/encode/django-rest-framework/issues/4869), [#4870](https://github.com/encode/django-rest-framework/issues/4870))
*   Better error reporting when schemas are returned by views without a schema renderer. ([#4790](https://github.com/encode/django-rest-framework/issues/4790))
*   Fix for returned response of `PUT` requests when `prefetch_related` is used. ([#4661](https://github.com/encode/django-rest-framework/issues/4661), [#4668](https://github.com/encode/django-rest-framework/issues/4668))
*   Fix for breadcrumb view names. ([#4750](https://github.com/encode/django-rest-framework/issues/4750))
*   Fix for RequestsClient ensuring fully qualified URLs. ([#4678](https://github.com/encode/django-rest-framework/issues/4678))
*   Fix for incorrect behavior of writable-nested fields check in some cases. ([#4634](https://github.com/encode/django-rest-framework/issues/4634), [#4669](https://github.com/encode/django-rest-framework/issues/4669))
*   Resolve Django deprecation warnings. ([#4712](https://github.com/encode/django-rest-framework/issues/4712))
*   Various cleanup of test cases.

**Date**: [7th November 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.3+Release%22)

*   Don't raise incorrect FilterSet deprecation warnings. ([#4660](https://github.com/encode/django-rest-framework/issues/4660), [#4643](https://github.com/encode/django-rest-framework/issues/4643), [#4644](https://github.com/encode/django-rest-framework/issues/4644))
*   Schema generation should not raise 404 when a view permission class does. ([#4645](https://github.com/encode/django-rest-framework/issues/4645), [#4646](https://github.com/encode/django-rest-framework/issues/4646))
*   Add `autofocus` support for input controls. ([#4650](https://github.com/encode/django-rest-framework/issues/4650))

**Date**: [1st November 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.2+Release%22)

*   Restore exception tracebacks in Python 2.7\. ([#4631](https://github.com/encode/django-rest-framework/issues/4631), [#4638](https://github.com/encode/django-rest-framework/issues/4638))
*   Properly display dicts in the admin console. ([#4532](https://github.com/encode/django-rest-framework/issues/4532), [#4636](https://github.com/encode/django-rest-framework/issues/4636))
*   Fix is_simple_callable with variable args, kwargs. ([#4622](https://github.com/encode/django-rest-framework/issues/4622), [#4602](https://github.com/encode/django-rest-framework/issues/4602))
*   Support 'on'/'off' literals with BooleanField. ([#4640](https://github.com/encode/django-rest-framework/issues/4640), [#4624](https://github.com/encode/django-rest-framework/issues/4624))
*   Enable cursor pagination of value querysets. ([#4569](https://github.com/encode/django-rest-framework/issues/4569))
*   Fix support of get_full_details() for Throttled exceptions. ([#4627](https://github.com/encode/django-rest-framework/issues/4627))
*   Fix FilterSet proxy. ([#4620](https://github.com/encode/django-rest-framework/issues/4620))
*   Make serializer fields import explicit. ([#4628](https://github.com/encode/django-rest-framework/issues/4628))
*   Drop redundant requests adapter. ([#4639](https://github.com/encode/django-rest-framework/issues/4639))

**Date**: [21st October 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.1+Release%22)

*   Make `rest_framework/compat.py` imports. ([#4612](https://github.com/encode/django-rest-framework/issues/4612), [#4608](https://github.com/encode/django-rest-framework/issues/4608), [#4601](https://github.com/encode/django-rest-framework/issues/4601))
*   Fix bug in schema base path generation. ([#4611](https://github.com/encode/django-rest-framework/issues/4611), [#4605](https://github.com/encode/django-rest-framework/issues/4605))
*   Fix broken case of ListSerializer with single item. ([#4609](https://github.com/encode/django-rest-framework/issues/4609), [#4606](https://github.com/encode/django-rest-framework/issues/4606))
*   Remove bare `raise` for Python 3.5 compat. ([#4600](https://github.com/encode/django-rest-framework/issues/4600))

**Date**: [20th October 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.5.0+Release%22)

* * *

**Date**: [21st September 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.7+Release%22)

*   Fallback behavior for request parsing when request.POST already accessed. ([#3951](https://github.com/encode/django-rest-framework/issues/3951), [#4500](https://github.com/encode/django-rest-framework/issues/4500))
*   Fix regression of `RegexField`. ([#4489](https://github.com/encode/django-rest-framework/issues/4489), [#4490](https://github.com/encode/django-rest-framework/issues/4490), [#2617](https://github.com/encode/django-rest-framework/issues/2617))
*   Missing comma in `admin.html` causing CSRF error. ([#4472](https://github.com/encode/django-rest-framework/issues/4472), [#4473](https://github.com/encode/django-rest-framework/issues/4473))
*   Fix response rendering with empty context. ([#4495](https://github.com/encode/django-rest-framework/issues/4495))
*   Fix indentation regression in API listing. ([#4493](https://github.com/encode/django-rest-framework/issues/4493))
*   Fixed an issue where the incorrect value is set to `ResolverMatch.func_name` of api_view decorated view. ([#4465](https://github.com/encode/django-rest-framework/issues/4465), [#4462](https://github.com/encode/django-rest-framework/issues/4462))
*   Fix `APIClient.get()` when path contains unicode arguments ([#4458](https://github.com/encode/django-rest-framework/issues/4458))

**Date**: [23rd August 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.6+Release%22)

*   Fix malformed Javascript in browsable API. ([#4435](https://github.com/encode/django-rest-framework/issues/4435))
*   Skip HiddenField from Schema fields. ([#4425](https://github.com/encode/django-rest-framework/issues/4425), [#4429](https://github.com/encode/django-rest-framework/issues/4429))
*   Improve Create to show the original exception traceback. ([#3508](https://github.com/encode/django-rest-framework/issues/3508))
*   Fix `AdminRenderer` display of PK only related fields. ([#4419](https://github.com/encode/django-rest-framework/issues/4419), [#4423](https://github.com/encode/django-rest-framework/issues/4423))

**Date**: [19th August 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.5+Release%22)

*   Improve debug error handling. ([#4416](https://github.com/encode/django-rest-framework/issues/4416), [#4409](https://github.com/encode/django-rest-framework/issues/4409))
*   Allow custom CSRF_HEADER_NAME setting. ([#4415](https://github.com/encode/django-rest-framework/issues/4415), [#4410](https://github.com/encode/django-rest-framework/issues/4410))
*   Include .action attribute on viewsets when generating schemas. ([#4408](https://github.com/encode/django-rest-framework/issues/4408), [#4398](https://github.com/encode/django-rest-framework/issues/4398))
*   Do not include request.FILES items in request.POST. ([#4407](https://github.com/encode/django-rest-framework/issues/4407))
*   Fix rendering of checkbox multiple. ([#4403](https://github.com/encode/django-rest-framework/issues/4403))
*   Fix docstring of Field.get_default. ([#4404](https://github.com/encode/django-rest-framework/issues/4404))
*   Replace utf8 character with its ascii counterpart in README. ([#4412](https://github.com/encode/django-rest-framework/issues/4412))

**Date**: [12th August 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.4+Release%22)

*   Ensure views are fully initialized when generating schemas. ([#4373](https://github.com/encode/django-rest-framework/issues/4373), [#4382](https://github.com/encode/django-rest-framework/issues/4382), [#4383](https://github.com/encode/django-rest-framework/issues/4383), [#4279](https://github.com/encode/django-rest-framework/issues/4279), [#4278](https://github.com/encode/django-rest-framework/issues/4278))
*   Add form field descriptions to schemas. ([#4387](https://github.com/encode/django-rest-framework/issues/4387))
*   Fix category generation for schema endpoints. ([#4391](https://github.com/encode/django-rest-framework/issues/4391), [#4394](https://github.com/encode/django-rest-framework/issues/4394), [#4390](https://github.com/encode/django-rest-framework/issues/4390), [#4386](https://github.com/encode/django-rest-framework/issues/4386), [#4376](https://github.com/encode/django-rest-framework/issues/4376), [#4329](https://github.com/encode/django-rest-framework/issues/4329))
*   Don't strip empty query params when paginating. ([#4392](https://github.com/encode/django-rest-framework/issues/4392), [#4393](https://github.com/encode/django-rest-framework/issues/4393), [#4260](https://github.com/encode/django-rest-framework/issues/4260))
*   Do not re-run query for empty results with LimitOffsetPagination. ([#4201](https://github.com/encode/django-rest-framework/issues/4201), [#4388](https://github.com/encode/django-rest-framework/issues/4388))
*   Stricter type validation for CharField. ([#4380](https://github.com/encode/django-rest-framework/issues/4380), [#3394](https://github.com/encode/django-rest-framework/issues/3394))
*   RelatedField.choices should preserve non-string values. ([#4111](https://github.com/encode/django-rest-framework/issues/4111), [#4379](https://github.com/encode/django-rest-framework/issues/4379), [#3365](https://github.com/encode/django-rest-framework/issues/3365))
*   Test case for rendering checkboxes in vertical form style. ([#4378](https://github.com/encode/django-rest-framework/issues/4378), [#3868](https://github.com/encode/django-rest-framework/issues/3868), [#3868](https://github.com/encode/django-rest-framework/issues/3868))
*   Show error traceback HTML in browsable API ([#4042](https://github.com/encode/django-rest-framework/issues/4042), [#4172](https://github.com/encode/django-rest-framework/issues/4172))
*   Fix handling of ALLOWED_VERSIONS and no DEFAULT_VERSION. [#4370](https://github.com/encode/django-rest-framework/issues/4370)
*   Allow `max_digits=None` on DecimalField. ([#4377](https://github.com/encode/django-rest-framework/issues/4377), [#4372](https://github.com/encode/django-rest-framework/issues/4372))
*   Limit queryset when rendering relational choices. ([#4375](https://github.com/encode/django-rest-framework/issues/4375), [#4122](https://github.com/encode/django-rest-framework/issues/4122), [#3329](https://github.com/encode/django-rest-framework/issues/3329), [#3330](https://github.com/encode/django-rest-framework/issues/3330), [#3877](https://github.com/encode/django-rest-framework/issues/3877))
*   Resolve form display with ChoiceField, MultipleChoiceField and non-string choices. ([#4374](https://github.com/encode/django-rest-framework/issues/4374), [#4119](https://github.com/encode/django-rest-framework/issues/4119), [#4121](https://github.com/encode/django-rest-framework/issues/4121), [#4137](https://github.com/encode/django-rest-framework/issues/4137), [#4120](https://github.com/encode/django-rest-framework/issues/4120))
*   Fix call to TemplateHTMLRenderer.resolve_context() fallback method. ([#4371](https://github.com/encode/django-rest-framework/issues/4371))

**Date**: [5th August 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.3+Release%22)

*   Include fallback for users of older TemplateHTMLRenderer internal API. ([#4361](https://github.com/encode/django-rest-framework/issues/4361))

**Date**: [5th August 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.2+Release%22)

*   Include kwargs passed to 'as_view' when generating schemas. ([#4359](https://github.com/encode/django-rest-framework/issues/4359), [#4330](https://github.com/encode/django-rest-framework/issues/4330), [#4331](https://github.com/encode/django-rest-framework/issues/4331))
*   Access `request.user.is_authenticated` as property not method, under Django 1.10+ ([#4358](https://github.com/encode/django-rest-framework/issues/4358), [#4354](https://github.com/encode/django-rest-framework/issues/4354))
*   Filter HEAD out from schemas. ([#4357](https://github.com/encode/django-rest-framework/issues/4357))
*   extra_kwargs takes precedence over uniqueness kwargs. ([#4198](https://github.com/encode/django-rest-framework/issues/4198), [#4199](https://github.com/encode/django-rest-framework/issues/4199), [#4349](https://github.com/encode/django-rest-framework/issues/4349))
*   Correct descriptions when tabs are used in code indentation. ([#4345](https://github.com/encode/django-rest-framework/issues/4345), [#4347](https://github.com/encode/django-rest-framework/issues/4347))*
*   Change template context generation in TemplateHTMLRenderer. ([#4236](https://github.com/encode/django-rest-framework/issues/4236))
*   Serializer defaults should not be included in partial updates. ([#4346](https://github.com/encode/django-rest-framework/issues/4346), [#3565](https://github.com/encode/django-rest-framework/issues/3565))
*   Consistent behavior & descriptive error from FileUploadParser when filename not included. ([#4340](https://github.com/encode/django-rest-framework/issues/4340), [#3610](https://github.com/encode/django-rest-framework/issues/3610), [#4292](https://github.com/encode/django-rest-framework/issues/4292), [#4296](https://github.com/encode/django-rest-framework/issues/4296))
*   DecimalField quantizes incoming digitals. ([#4339](https://github.com/encode/django-rest-framework/issues/4339), [#4318](https://github.com/encode/django-rest-framework/issues/4318))
*   Handle non-string input for IP fields. ([#4335](https://github.com/encode/django-rest-framework/issues/4335), [#4336](https://github.com/encode/django-rest-framework/issues/4336), [#4338](https://github.com/encode/django-rest-framework/issues/4338))
*   Fix leading slash handling when Schema generation includes a root URL. ([#4332](https://github.com/encode/django-rest-framework/issues/4332))
*   Test cases for DictField with allow_null options. ([#4348](https://github.com/encode/django-rest-framework/issues/4348))
*   Update tests from Django 1.10 beta to Django 1.10\. ([#4344](https://github.com/encode/django-rest-framework/issues/4344))

**Date**: [28th July 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.1+Release%22)

*   Added `root_renderers` argument to `DefaultRouter`. ([#4323](https://github.com/encode/django-rest-framework/issues/4323), [#4268](https://github.com/encode/django-rest-framework/issues/4268))
*   Added `url` and `schema_url` arguments. ([#4321](https://github.com/encode/django-rest-framework/issues/4321), [#4308](https://github.com/encode/django-rest-framework/issues/4308), [#4305](https://github.com/encode/django-rest-framework/issues/4305))
*   Unique together checks should apply to read-only fields which have a default. ([#4316](https://github.com/encode/django-rest-framework/issues/4316), [#4294](https://github.com/encode/django-rest-framework/issues/4294))
*   Set view.format_kwarg in schema generator. ([#4293](https://github.com/encode/django-rest-framework/issues/4293), [#4315](https://github.com/encode/django-rest-framework/issues/4315))
*   Fix schema generator for views with `pagination_class = None`. ([#4314](https://github.com/encode/django-rest-framework/issues/4314), [#4289](https://github.com/encode/django-rest-framework/issues/4289))
*   Fix schema generator for views with no `get_serializer_class`. ([#4265](https://github.com/encode/django-rest-framework/issues/4265), [#4285](https://github.com/encode/django-rest-framework/issues/4285))
*   Fixes for media type parameters in `Accept` and `Content-Type` headers. ([#4287](https://github.com/encode/django-rest-framework/issues/4287), [#4313](https://github.com/encode/django-rest-framework/issues/4313), [#4281](https://github.com/encode/django-rest-framework/issues/4281))
*   Use verbose_name instead of object_name in error messages. ([#4299](https://github.com/encode/django-rest-framework/issues/4299))
*   Minor version update to Twitter Bootstrap. ([#4307](https://github.com/encode/django-rest-framework/issues/4307))
*   SearchFilter raises error when using with related field. ([#4302](https://github.com/encode/django-rest-framework/issues/4302), [#4303](https://github.com/encode/django-rest-framework/issues/4303), [#4298](https://github.com/encode/django-rest-framework/issues/4298))
*   Adding support for RFC 4918 status codes. ([#4291](https://github.com/encode/django-rest-framework/issues/4291))
*   Add LICENSE.md to the built wheel. ([#4270](https://github.com/encode/django-rest-framework/issues/4270))
*   Serializing "complex" field returns None instead of the value since 3.4 ([#4272](https://github.com/encode/django-rest-framework/issues/4272), [#4273](https://github.com/encode/django-rest-framework/issues/4273), [#4288](https://github.com/encode/django-rest-framework/issues/4288))

**Date**: [14th July 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.4.0+Release%22)

*   Don't strip microseconds in JSON output. ([#4256](https://github.com/encode/django-rest-framework/issues/4256))
*   Two slightly different iso 8601 datetime serialization. ([#4255](https://github.com/encode/django-rest-framework/issues/4255))
*   Resolve incorrect inclusion of media type parameters. ([#4254](https://github.com/encode/django-rest-framework/issues/4254))
*   Response Content-Type potentially malformed. ([#4253](https://github.com/encode/django-rest-framework/issues/4253))
*   Fix setup.py error on some platforms. ([#4246](https://github.com/encode/django-rest-framework/issues/4246))
*   Move alternate formats in coreapi into separate packages. ([#4244](https://github.com/encode/django-rest-framework/issues/4244))
*   Add localize keyword argument to `DecimalField`. ([#4233](https://github.com/encode/django-rest-framework/issues/4233))
*   Fix issues with routers for custom list-route and detail-routes. ([#4229](https://github.com/encode/django-rest-framework/issues/4229))
*   Namespace versioning with nested namespaces. ([#4219](https://github.com/encode/django-rest-framework/issues/4219))
*   Robust uniqueness checks. ([#4217](https://github.com/encode/django-rest-framework/issues/4217))
*   Minor refactoring of `must_call_distinct`. ([#4215](https://github.com/encode/django-rest-framework/issues/4215))
*   Overridable offset cutoff in CursorPagination. ([#4212](https://github.com/encode/django-rest-framework/issues/4212))
*   Pass through strings as-in with date/time fields. ([#4196](https://github.com/encode/django-rest-framework/issues/4196))
*   Add test confirming that required=False is valid on a relational field. ([#4195](https://github.com/encode/django-rest-framework/issues/4195))
*   In LimitOffsetPagination `limit=0` should revert to default limit. ([#4194](https://github.com/encode/django-rest-framework/issues/4194))
*   Exclude read_only=True fields from unique_together validation & add docs. ([#4192](https://github.com/encode/django-rest-framework/issues/4192))
*   Handle bytestrings in JSON. ([#4191](https://github.com/encode/django-rest-framework/issues/4191))
*   JSONField(binary=True) represents using binary strings, which JSONRenderer does not support. ([#4187](https://github.com/encode/django-rest-framework/issues/4187))
*   JSONField(binary=True) represents using binary strings, which JSONRenderer does not support. ([#4185](https://github.com/encode/django-rest-framework/issues/4185))
*   More robust form rendering in the browsable API. ([#4181](https://github.com/encode/django-rest-framework/issues/4181))
*   Empty cases of `.validated_data` and `.errors` as lists not dicts for ListSerializer. ([#4180](https://github.com/encode/django-rest-framework/issues/4180))
*   Schemas & client libraries. ([#4179](https://github.com/encode/django-rest-framework/issues/4179))
*   Removed `AUTH_USER_MODEL` compat property. ([#4176](https://github.com/encode/django-rest-framework/issues/4176))
*   Clean up existing deprecation warnings. ([#4166](https://github.com/encode/django-rest-framework/issues/4166))
*   Django 1.10 support. ([#4158](https://github.com/encode/django-rest-framework/issues/4158))
*   Updated jQuery version to 1.12.4\. ([#4157](https://github.com/encode/django-rest-framework/issues/4157))
*   More robust default behavior on OrderingFilter. ([#4156](https://github.com/encode/django-rest-framework/issues/4156))
*   description.py codes and tests removal. ([#4153](https://github.com/encode/django-rest-framework/issues/4153))
*   Wrap guardian.VERSION in tuple. ([#4149](https://github.com/encode/django-rest-framework/issues/4149))
*   Refine validator for fields with  kwargs. ([#4146](https://github.com/encode/django-rest-framework/issues/4146))
*   Fix None values representation in children of ListField, DictField. ([#4118](https://github.com/encode/django-rest-framework/issues/4118))
*   Resolve TimeField representation for midnight value. ([#4107](https://github.com/encode/django-rest-framework/issues/4107))
*   Set proper status code in AdminRenderer for the redirection after POST/DELETE requests. ([#4106](https://github.com/encode/django-rest-framework/issues/4106))
*   TimeField render returns None instead of 00:00:00\. ([#4105](https://github.com/encode/django-rest-framework/issues/4105))
*   Fix incorrectly named zh-hans and zh-hant locale path. ([#4103](https://github.com/encode/django-rest-framework/issues/4103))
*   Prevent raising exception when limit is 0\. ([#4098](https://github.com/encode/django-rest-framework/issues/4098))
*   TokenAuthentication: Allow custom keyword in the header. ([#4097](https://github.com/encode/django-rest-framework/issues/4097))
*   Handle incorrectly padded HTTP basic auth header. ([#4090](https://github.com/encode/django-rest-framework/issues/4090))
*   LimitOffset pagination crashes Browsable API when limit=0\. ([#4079](https://github.com/encode/django-rest-framework/issues/4079))
*   Fixed DecimalField arbitrary precision support. ([#4075](https://github.com/encode/django-rest-framework/issues/4075))
*   Added support for custom CSRF cookie names. ([#4049](https://github.com/encode/django-rest-framework/issues/4049))
*   Fix regression introduced by #4035\. ([#4041](https://github.com/encode/django-rest-framework/issues/4041))
*   No auth view failing permission should raise 403\. ([#4040](https://github.com/encode/django-rest-framework/issues/4040))
*   Fix string_types / text_types confusion. ([#4025](https://github.com/encode/django-rest-framework/issues/4025))
*   Do not list related field choices in OPTIONS requests. ([#4021](https://github.com/encode/django-rest-framework/issues/4021))
*   Fix typo. ([#4008](https://github.com/encode/django-rest-framework/issues/4008))
*   Reorder initializing the view. ([#4006](https://github.com/encode/django-rest-framework/issues/4006))
*   Type error in DjangoObjectPermissionsFilter on Python 3.4\. ([#4005](https://github.com/encode/django-rest-framework/issues/4005))
*   Fixed use of deprecated Query.aggregates. ([#4003](https://github.com/encode/django-rest-framework/issues/4003))
*   Fix blank lines around docstrings. ([#4002](https://github.com/encode/django-rest-framework/issues/4002))
*   Fixed admin pagination when limit is 0\. ([#3990](https://github.com/encode/django-rest-framework/issues/3990))
*   OrderingFilter adjustments. ([#3983](https://github.com/encode/django-rest-framework/issues/3983))
*   Non-required serializer related fields. ([#3976](https://github.com/encode/django-rest-framework/issues/3976))
*   Using safer calling way of "@api_view" in tutorial. ([#3971](https://github.com/encode/django-rest-framework/issues/3971))
*   ListSerializer doesn't handle unique_together constraints. ([#3970](https://github.com/encode/django-rest-framework/issues/3970))
*   Add missing migration file. ([#3968](https://github.com/encode/django-rest-framework/issues/3968))
*   `OrderingFilter` should call `get_serializer_class()` to determine default fields. ([#3964](https://github.com/encode/django-rest-framework/issues/3964))
*   Remove old Django checks from tests and compat. ([#3953](https://github.com/encode/django-rest-framework/issues/3953))
*   Support callable as the value of `initial` for any `serializer.Field`. ([#3943](https://github.com/encode/django-rest-framework/issues/3943))
*   Prevented unnecessary distinct() call in SearchFilter. ([#3938](https://github.com/encode/django-rest-framework/issues/3938))
*   Fix None UUID ForeignKey serialization. ([#3936](https://github.com/encode/django-rest-framework/issues/3936))
*   Drop EOL Django 1.7\. ([#3933](https://github.com/encode/django-rest-framework/issues/3933))
*   Add missing space in serializer error message. ([#3926](https://github.com/encode/django-rest-framework/issues/3926))
*   Fixed _force_text_recursive typo. ([#3908](https://github.com/encode/django-rest-framework/issues/3908))
*   Attempt to address Django 2.0 deprecate warnings related to `field.rel`. ([#3906](https://github.com/encode/django-rest-framework/issues/3906))
*   Fix parsing multipart data using a nested serializer with list. ([#3820](https://github.com/encode/django-rest-framework/issues/3820))
*   Resolving APIs URL to different namespaces. ([#3816](https://github.com/encode/django-rest-framework/issues/3816))
*   Do not HTML-escape `help_text` in Browsable API forms. ([#3812](https://github.com/encode/django-rest-framework/issues/3812))
*   OPTIONS fetches and shows all possible foreign keys in choices field. ([#3751](https://github.com/encode/django-rest-framework/issues/3751))
*   Django 1.9 deprecation warnings ([#3729](https://github.com/encode/django-rest-framework/issues/3729))
*   Test case for #3598 ([#3710](https://github.com/encode/django-rest-framework/issues/3710))
*   Adding support for multiple values for search filter. ([#3541](https://github.com/encode/django-rest-framework/issues/3541))
*   Use get_serializer_class in ordering filter. ([#3487](https://github.com/encode/django-rest-framework/issues/3487))
*   Serializers with many=True should return empty list rather than empty dict. ([#3476](https://github.com/encode/django-rest-framework/issues/3476))
*   LimitOffsetPagination limit=0 fix. ([#3444](https://github.com/encode/django-rest-framework/issues/3444))
*   Enable Validators to defer string evaluation and handle new string format. ([#3438](https://github.com/encode/django-rest-framework/issues/3438))
*   Unique validator is executed and breaks if field is invalid. ([#3381](https://github.com/encode/django-rest-framework/issues/3381))
*   Do not ignore overridden View.get_view_name() in breadcrumbs. ([#3273](https://github.com/encode/django-rest-framework/issues/3273))
*   Retry form rendering when rendering with serializer fails. ([#3164](https://github.com/encode/django-rest-framework/issues/3164))
*   Unique constraint prevents nested serializers from updating. ([#2996](https://github.com/encode/django-rest-framework/issues/2996))
*   Uniqueness validators should not be run for excluded (read_only) fields. ([#2848](https://github.com/encode/django-rest-framework/issues/2848))
*   UniqueValidator raises exception for nested objects. ([#2403](https://github.com/encode/django-rest-framework/issues/2403))
*   `lookup_type` is deprecated in favor of `lookup_expr`. ([#4259](https://github.com/encode/django-rest-framework/issues/4259))

* * *

**Date**: [14th March 2016](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.3+Release%22).

*   Remove version string from templates. Thanks to @blag for the report and fixes. ([#3878](https://github.com/encode/django-rest-framework/issues/3878), [#3913](https://github.com/encode/django-rest-framework/issues/3913), [#3912](https://github.com/encode/django-rest-framework/issues/3912))
*   Fixes vertical html layout for `BooleanField`. Thanks to Mikalai Radchuk for the fix. ([#3910](https://github.com/encode/django-rest-framework/issues/3910))
*   Silenced deprecation warnings on Django 1.8\. Thanks to Simon Charette for the fix. ([#3903](https://github.com/encode/django-rest-framework/issues/3903))
*   Internationalization for authtoken. Thanks to Michael Nacharov for the fix. ([#3887](https://github.com/encode/django-rest-framework/issues/3887), [#3968](https://github.com/encode/django-rest-framework/issues/3968))
*   Fix `Token` model as `abstract` when the authtoken application isn't declared. Thanks to Adam Thomas for the report. ([#3860](https://github.com/encode/django-rest-framework/issues/3860), [#3858](https://github.com/encode/django-rest-framework/issues/3858))
*   Improve Markdown version compatibility. Thanks to Michael J. Schultz for the fix. ([#3604](https://github.com/encode/django-rest-framework/issues/3604), [#3842](https://github.com/encode/django-rest-framework/issues/3842))
*   `QueryParameterVersioning` does not use `DEFAULT_VERSION` setting. Thanks to Brad Montgomery for the fix. ([#3833](https://github.com/encode/django-rest-framework/issues/3833))
*   Add an explicit `on_delete` on the models. Thanks to Mads Jensen for the fix. ([#3832](https://github.com/encode/django-rest-framework/issues/3832))
*   Fix `DateField.to_representation` to work with Python 2 unicode. Thanks to Mikalai Radchuk for the fix. ([#3819](https://github.com/encode/django-rest-framework/issues/3819))
*   Fixed `TimeField` not handling string times. Thanks to Areski Belaid for the fix. ([#3809](https://github.com/encode/django-rest-framework/issues/3809))
*   Avoid updates of `Meta.extra_kwargs`. Thanks to Kevin Massey for the report and fix. ([#3805](https://github.com/encode/django-rest-framework/issues/3805), [#3804](https://github.com/encode/django-rest-framework/issues/3804))
*   Fix nested validation error being rendered incorrectly. Thanks to Craig de Stigter for the fix. ([#3801](https://github.com/encode/django-rest-framework/issues/3801))
*   Document how to avoid CSRF and missing button issues with `django-crispy-forms`. Thanks to Emmanuelle Delescolle, José Padilla and Luis San Pablo for the report, analysis and fix. ([#3787](https://github.com/encode/django-rest-framework/issues/3787), [#3636](https://github.com/encode/django-rest-framework/issues/3636), [#3637](https://github.com/encode/django-rest-framework/issues/3637))
*   Improve Rest Framework Settings file setup time. Thanks to Miles Hutson for the report and Mads Jensen for the fix. ([#3786](https://github.com/encode/django-rest-framework/issues/3786), [#3815](https://github.com/encode/django-rest-framework/issues/3815))
*   Improve authtoken compatibility with Django 1.9\. Thanks to S. Andrew Sheppard for the fix. ([#3785](https://github.com/encode/django-rest-framework/issues/3785))
*   Fix `Min/MaxValueValidator` transfer from a model's `DecimalField`. Thanks to Kevin Brown for the fix. ([#3774](https://github.com/encode/django-rest-framework/issues/3774))
*   Improve HTML title in the Browsable API. Thanks to Mike Lissner for the report and fix. ([#3769](https://github.com/encode/django-rest-framework/issues/3769))
*   Fix `AutoFilterSet` to inherit from `default_filter_set`. Thanks to Tom Linford for the fix. ([#3753](https://github.com/encode/django-rest-framework/issues/3753))
*   Fix transifex config to handle the new Chinese language codes. Thanks to @nypisces for the report and fix. ([#3739](https://github.com/encode/django-rest-framework/issues/3739))
*   `DateTimeField` does not handle empty values correctly. Thanks to Mick Parker for the report and fix. ([#3731](https://github.com/encode/django-rest-framework/issues/3731), [#3726](https://github.com/encode/django-rest-framework/issues/3726))
*   Raise error when setting a removed rest_framework setting. Thanks to Luis San Pablo for the fix. ([#3715](https://github.com/encode/django-rest-framework/issues/3715))
*   Add missing csrf_token in AdminRenderer post form. Thanks to Piotr Śniegowski for the fix. ([#3703](https://github.com/encode/django-rest-framework/issues/3703))
*   Refactored `_get_reverse_relationships()` to use correct `to_field`. Thanks to Benjamin Phillips for the fix. ([#3696](https://github.com/encode/django-rest-framework/issues/3696))
*   Document the use of `get_queryset` for `RelatedField`. Thanks to Ryan Hiebert for the fix. ([#3605](https://github.com/encode/django-rest-framework/issues/3605))
*   Fix empty pk detection in HyperlinkRelatedField.get_url. Thanks to @jslang for the fix ([#3962](https://github.com/encode/django-rest-framework/issues/3962))

**Date**: [14th December 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.2+Release%22).

*   `ListField` enforces input is a list. ([#3513](https://github.com/encode/django-rest-framework/issues/3513))
*   Fix regression hiding raw data form. ([#3600](https://github.com/encode/django-rest-framework/issues/3600), [#3578](https://github.com/encode/django-rest-framework/issues/3578))
*   Fix Python 3.5 compatibility. ([#3534](https://github.com/encode/django-rest-framework/issues/3534), [#3626](https://github.com/encode/django-rest-framework/issues/3626))
*   Allow setting a custom Django Paginator in `pagination.PageNumberPagination`. ([#3631](https://github.com/encode/django-rest-framework/issues/3631), [#3684](https://github.com/encode/django-rest-framework/issues/3684))
*   Fix relational fields without `to_fields` attribute. ([#3635](https://github.com/encode/django-rest-framework/issues/3635), [#3634](https://github.com/encode/django-rest-framework/issues/3634))
*   Fix `template.render` deprecation warnings for Django 1.9\. ([#3654](https://github.com/encode/django-rest-framework/issues/3654))
*   Sort response headers in browsable API renderer. ([#3655](https://github.com/encode/django-rest-framework/issues/3655))
*   Use related_objects api for Django 1.9+. ([#3656](https://github.com/encode/django-rest-framework/issues/3656), [#3252](https://github.com/encode/django-rest-framework/issues/3252))
*   Add confirm modal when deleting. ([#3228](https://github.com/encode/django-rest-framework/issues/3228), [#3662](https://github.com/encode/django-rest-framework/issues/3662))
*   Reveal previously hidden AttributeErrors and TypeErrors while calling has_[object_]permissions. ([#3668](https://github.com/encode/django-rest-framework/issues/3668))
*   Make DRF compatible with multi template engine in Django 1.8\. ([#3672](https://github.com/encode/django-rest-framework/issues/3672))
*   Update `NestedBoundField` to also handle empty string when rendering its form. ([#3677](https://github.com/encode/django-rest-framework/issues/3677))
*   Fix UUID validation to properly catch invalid input types. ([#3687](https://github.com/encode/django-rest-framework/issues/3687), [#3679](https://github.com/encode/django-rest-framework/issues/3679))
*   Fix caching issues. ([#3628](https://github.com/encode/django-rest-framework/issues/3628), [#3701](https://github.com/encode/django-rest-framework/issues/3701))
*   Fix Admin and API browser for views without a filter_class. ([#3705](https://github.com/encode/django-rest-framework/issues/3705), [#3596](https://github.com/encode/django-rest-framework/issues/3596), [#3597](https://github.com/encode/django-rest-framework/issues/3597))
*   Add app_name to rest_framework.urls. ([#3714](https://github.com/encode/django-rest-framework/issues/3714))
*   Improve authtoken's views to support url versioning. ([#3718](https://github.com/encode/django-rest-framework/issues/3718), [#3723](https://github.com/encode/django-rest-framework/issues/3723))

**Date**: [4th November 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.1+Release%22).

*   Resolve parsing bug when accessing `request.POST` ([#3592](https://github.com/encode/django-rest-framework/issues/3592))
*   Correctly deal with `to_field` referring to primary key. ([#3593](https://github.com/encode/django-rest-framework/issues/3593))
*   Allow filter HTML to render when no `filter_class` is defined. ([#3560](https://github.com/encode/django-rest-framework/issues/3560))
*   Fix admin rendering issues. ([#3564](https://github.com/encode/django-rest-framework/issues/3564), [#3556](https://github.com/encode/django-rest-framework/issues/3556))
*   Fix issue with DecimalValidator. ([#3568](https://github.com/encode/django-rest-framework/issues/3568))

**Date**: [28th October 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.3.0+Release%22).

*   HTML controls for filters. ([#3315](https://github.com/encode/django-rest-framework/issues/3315))
*   Forms API. ([#3475](https://github.com/encode/django-rest-framework/issues/3475))
*   AJAX browsable API. ([#3410](https://github.com/encode/django-rest-framework/issues/3410))
*   Added JSONField. ([#3454](https://github.com/encode/django-rest-framework/issues/3454))
*   Correctly map `to_field` when creating `ModelSerializer` relational fields. ([#3526](https://github.com/encode/django-rest-framework/issues/3526))
*   Include keyword arguments when mapping `FilePathField` to a serializer field. ([#3536](https://github.com/encode/django-rest-framework/issues/3536))
*   Map appropriate model `error_messages` on `ModelSerializer` uniqueness constraints. ([#3435](https://github.com/encode/django-rest-framework/issues/3435))
*   Include `max_length` constraint for `ModelSerializer` fields mapped from TextField. ([#3509](https://github.com/encode/django-rest-framework/issues/3509))
*   Added support for Django 1.9\. ([#3450](https://github.com/encode/django-rest-framework/issues/3450), [#3525](https://github.com/encode/django-rest-framework/issues/3525))
*   Removed support for Django 1.5 & 1.6\. ([#3421](https://github.com/encode/django-rest-framework/issues/3421), [#3429](https://github.com/encode/django-rest-framework/issues/3429))
*   Removed 'south' migrations. ([#3495](https://github.com/encode/django-rest-framework/issues/3495))

* * *

**Date**: [27th October 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.5+Release%22).

*   Escape `username` in optional logout tag. ([#3550](https://github.com/encode/django-rest-framework/issues/3550))

**Date**: [21th September 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.4+Release%22).

*   Don't error on missing `ViewSet.search_fields` attribute. ([#3324](https://github.com/encode/django-rest-framework/issues/3324), [#3323](https://github.com/encode/django-rest-framework/issues/3323))
*   Fix `allow_empty` not working on serializers with `many=True`. ([#3361](https://github.com/encode/django-rest-framework/issues/3361), [#3364](https://github.com/encode/django-rest-framework/issues/3364))
*   Let `DurationField` accepts integers. ([#3359](https://github.com/encode/django-rest-framework/issues/3359))
*   Multi-level dictionaries not supported in multipart requests. ([#3314](https://github.com/encode/django-rest-framework/issues/3314))
*   Fix `ListField` truncation on HTTP PATCH ([#3415](https://github.com/encode/django-rest-framework/issues/3415), [#2761](https://github.com/encode/django-rest-framework/issues/2761))

**Date**: [24th August 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.3+Release%22).

*   Added `html_cutoff` and `html_cutoff_text` for limiting select dropdowns. ([#3313](https://github.com/encode/django-rest-framework/issues/3313))
*   Added regex style to `SearchFilter`. ([#3316](https://github.com/encode/django-rest-framework/issues/3316))
*   Resolve issues with setting blank HTML fields. ([#3318](https://github.com/encode/django-rest-framework/issues/3318)) ([#3321](https://github.com/encode/django-rest-framework/issues/3321))
*   Correctly display existing 'select multiple' values in browsable API forms. ([#3290](https://github.com/encode/django-rest-framework/issues/3290))
*   Resolve duplicated validation message for `IPAddressField`. ([#3249[gh3249](https://github.com/encode/django-rest-framework/issues/3249)) ([#3250](https://github.com/encode/django-rest-framework/issues/3250))
*   Fix to ensure admin renderer continues to work when pagination is disabled. ([#3275](https://github.com/encode/django-rest-framework/issues/3275))
*   Resolve error with `LimitOffsetPagination` when count=0, offset=0\. ([#3303](https://github.com/encode/django-rest-framework/issues/3303))

**Date**: [13th August 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.2+Release%22).

*   Add `display_value()` method for use when displaying relational field select inputs. ([#3254](https://github.com/encode/django-rest-framework/issues/3254))
*   Fix issue with `BooleanField` checkboxes incorrectly displaying as checked. ([#3258](https://github.com/encode/django-rest-framework/issues/3258))
*   Ensure empty checkboxes properly set `BooleanField` to `False` in all cases. ([#2776](https://github.com/encode/django-rest-framework/issues/2776))
*   Allow `WSGIRequest.FILES` property without raising incorrect deprecated error. ([#3261](https://github.com/encode/django-rest-framework/issues/3261))
*   Resolve issue with rendering nested serializers in forms. ([#3260](https://github.com/encode/django-rest-framework/issues/3260))
*   Raise an error if user accidentally pass a serializer instance to a response, rather than data. ([#3241](https://github.com/encode/django-rest-framework/issues/3241))

**Date**: [7th August 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.1+Release%22).

*   Fix for relational select widgets rendering without any choices. ([#3237](https://github.com/encode/django-rest-framework/issues/3237))
*   Fix for `1`, `0` rendering as `true`, `false` in the admin interface. [#3227](https://github.com/encode/django-rest-framework/issues/3227))
*   Fix for ListFields with single value in HTML form input. ([#3238](https://github.com/encode/django-rest-framework/issues/3238))
*   Allow `request.FILES` for compat with Django's `HTTPRequest` class. ([#3239](https://github.com/encode/django-rest-framework/issues/3239))

**Date**: [6th August 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.2.0+Release%22).

*   Add `AdminRenderer`. ([#2926](https://github.com/encode/django-rest-framework/issues/2926))
*   Add `FilePathField`. ([#1854](https://github.com/encode/django-rest-framework/issues/1854))
*   Add `allow_empty` to `ListField`. ([#2250](https://github.com/encode/django-rest-framework/issues/2250))
*   Support django-guardian 1.3\. ([#3165](https://github.com/encode/django-rest-framework/issues/3165))
*   Support grouped choices. ([#3225](https://github.com/encode/django-rest-framework/issues/3225))
*   Support error forms in browsable API. ([#3024](https://github.com/encode/django-rest-framework/issues/3024))
*   Allow permission classes to customize the error message. ([#2539](https://github.com/encode/django-rest-framework/issues/2539))
*   Support `source=<method>` on hyperlinked fields. ([#2690](https://github.com/encode/django-rest-framework/issues/2690))
*   `ListField(allow_null=True)` now allows null as the list value, not null items in the list. ([#2766](https://github.com/encode/django-rest-framework/issues/2766))
*   `ManyToMany()` maps to `allow_empty=False`, `ManyToMany(blank=True)` maps to `allow_empty=True`. ([#2804](https://github.com/encode/django-rest-framework/issues/2804))
*   Support custom serialization styles for primary key fields. ([#2789](https://github.com/encode/django-rest-framework/issues/2789))
*   `OPTIONS` requests support nested representations. ([#2915](https://github.com/encode/django-rest-framework/issues/2915))
*   Set `view.action == "metadata"` for viewsets with `OPTIONS` requests. ([#3115](https://github.com/encode/django-rest-framework/issues/3115))
*   Support `allow_blank` on `UUIDField`. ([#3130][gh#3130])
*   Do not display view docstrings with 401 or 403 response codes. ([#3216](https://github.com/encode/django-rest-framework/issues/3216))
*   Resolve Django 1.8 deprecation warnings. ([#2886](https://github.com/encode/django-rest-framework/issues/2886))
*   Fix for `DecimalField` validation. ([#3139](https://github.com/encode/django-rest-framework/issues/3139))
*   Fix behavior of `allow_blank=False` when used with `trim_whitespace=True`. ([#2712](https://github.com/encode/django-rest-framework/issues/2712))
*   Fix issue with some field combinations incorrectly mapping to an invalid `allow_blank` argument. ([#3011](https://github.com/encode/django-rest-framework/issues/3011))
*   Fix for output representations with prefetches and modified querysets. ([#2704](https://github.com/encode/django-rest-framework/issues/2704), [#2727](https://github.com/encode/django-rest-framework/issues/2727))
*   Fix assertion error when CursorPagination is provided with certain invalid query parameters. (#2920)[gh2920](https://github.com/encode/django-rest-framework/issues/2920).
*   Fix `UnicodeDecodeError` when invalid characters included in header with `TokenAuthentication`. ([#2928](https://github.com/encode/django-rest-framework/issues/2928))
*   Fix transaction rollbacks with `@non_atomic_requests` decorator. ([#3016](https://github.com/encode/django-rest-framework/issues/3016))
*   Fix duplicate results issue with Oracle databases using `SearchFilter`. ([#2935](https://github.com/encode/django-rest-framework/issues/2935))
*   Fix checkbox alignment and rendering in browsable API forms. ([#2783](https://github.com/encode/django-rest-framework/issues/2783))
*   Fix for unsaved file objects which should use `"url": null` in the representation. ([#2759](https://github.com/encode/django-rest-framework/issues/2759))
*   Fix field value rendering in browsable API. ([#2416](https://github.com/encode/django-rest-framework/issues/2416))
*   Fix `HStoreField` to include `allow_blank=True` in `DictField` mapping. ([#2659](https://github.com/encode/django-rest-framework/issues/2659))
*   Numerous other cleanups, improvements to error messaging, private API & minor fixes.

* * *

**Date**: [4th June 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.3+Release%22).

*   Add `DurationField`. ([#2481](https://github.com/encode/django-rest-framework/issues/2481), [#2989](https://github.com/encode/django-rest-framework/issues/2989))
*   Add `format` argument to `UUIDField`. ([#2788](https://github.com/encode/django-rest-framework/issues/2788), [#3000](https://github.com/encode/django-rest-framework/issues/3000))
*   `MultipleChoiceField` empties incorrectly on a partial update using multipart/form-data ([#2993](https://github.com/encode/django-rest-framework/issues/2993), [#2894](https://github.com/encode/django-rest-framework/issues/2894))
*   Fix a bug in options related to read-only `RelatedField`. ([#2981](https://github.com/encode/django-rest-framework/issues/2981), [#2811](https://github.com/encode/django-rest-framework/issues/2811))
*   Fix nested serializers with `unique_together` relations. ([#2975](https://github.com/encode/django-rest-framework/issues/2975))
*   Allow unexpected values for `ChoiceField`/`MultipleChoiceField` representations. ([#2839](https://github.com/encode/django-rest-framework/issues/2839), [#2940](https://github.com/encode/django-rest-framework/issues/2940))
*   Rollback the transaction on error if `ATOMIC_REQUESTS` is set. ([#2887](https://github.com/encode/django-rest-framework/issues/2887), [#2034](https://github.com/encode/django-rest-framework/issues/2034))
*   Set the action on a view when override_method regardless of its None-ness. ([#2933](https://github.com/encode/django-rest-framework/issues/2933))
*   `DecimalField` accepts `2E+2` as 200 and validates decimal place correctly. ([#2948](https://github.com/encode/django-rest-framework/issues/2948), [#2947](https://github.com/encode/django-rest-framework/issues/2947))
*   Support basic authentication with custom `UserModel` that change `username`. ([#2952](https://github.com/encode/django-rest-framework/issues/2952))
*   `IPAddressField` improvements. ([#2747](https://github.com/encode/django-rest-framework/issues/2747), [#2618](https://github.com/encode/django-rest-framework/issues/2618), [#3008](https://github.com/encode/django-rest-framework/issues/3008))
*   Improve `DecimalField` for easier subclassing. ([#2695](https://github.com/encode/django-rest-framework/issues/2695))

**Date**: [13rd May 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.2+Release%22).

*   `DateField.to_representation` can handle str and empty values. ([#2656](https://github.com/encode/django-rest-framework/issues/2656), [#2687](https://github.com/encode/django-rest-framework/issues/2687), [#2869](https://github.com/encode/django-rest-framework/issues/2869))
*   Use default reason phrases from HTTP standard. ([#2764](https://github.com/encode/django-rest-framework/issues/2764), [#2763](https://github.com/encode/django-rest-framework/issues/2763))
*   Raise error when `ModelSerializer` used with abstract model. ([#2757](https://github.com/encode/django-rest-framework/issues/2757), [#2630](https://github.com/encode/django-rest-framework/issues/2630))
*   Handle reversal of non-API view_name in `HyperLinkedRelatedField` ([#2724](https://github.com/encode/django-rest-framework/issues/2724), [#2711](https://github.com/encode/django-rest-framework/issues/2711))
*   Don't require pk strictly for related fields. ([#2745](https://github.com/encode/django-rest-framework/issues/2745), [#2754](https://github.com/encode/django-rest-framework/issues/2754))
*   Metadata detects null boolean field type. ([#2762](https://github.com/encode/django-rest-framework/issues/2762))
*   Proper handling of depth in nested serializers. ([#2798](https://github.com/encode/django-rest-framework/issues/2798))
*   Display viewset without paginator. ([#2807](https://github.com/encode/django-rest-framework/issues/2807))
*   Don't check for deprecated `.model` attribute in permissions ([#2818](https://github.com/encode/django-rest-framework/issues/2818))
*   Restrict integer field to integers and strings. ([#2835](https://github.com/encode/django-rest-framework/issues/2835), [#2836](https://github.com/encode/django-rest-framework/issues/2836))
*   Improve `IntegerField` to use compiled decimal regex. ([#2853](https://github.com/encode/django-rest-framework/issues/2853))
*   Prevent empty `queryset` to raise AssertionError. ([#2862](https://github.com/encode/django-rest-framework/issues/2862))
*   `DjangoModelPermissions` rely on `get_queryset`. ([#2863](https://github.com/encode/django-rest-framework/issues/2863))
*   Check `AcceptHeaderVersioning` with content negotiation in place. ([#2868](https://github.com/encode/django-rest-framework/issues/2868))
*   Allow `DjangoObjectPermissions` to use views that define `get_queryset`. ([#2905](https://github.com/encode/django-rest-framework/issues/2905))

**Date**: [23rd March 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.1+Release%22).

*   **Security fix**: Escape tab switching cookie name in browsable API.
*   Display input forms in browsable API if `serializer_class` is used, even when `get_serializer` method does not exist on the view. ([#2743](https://github.com/encode/django-rest-framework/issues/2643))
*   Use a password input for the AuthTokenSerializer. ([#2741](https://github.com/encode/django-rest-framework/issues/2641))
*   Fix missing anchor closing tag after next button. ([#2691](https://github.com/encode/django-rest-framework/issues/2691))
*   Fix `lookup_url_kwarg` handling in viewsets. ([#2685](https://github.com/encode/django-rest-framework/issues/2685), [#2591](https://github.com/encode/django-rest-framework/issues/2591))
*   Fix problem with importing `rest_framework.views` in `apps.py` ([#2678](https://github.com/encode/django-rest-framework/issues/2678))
*   LimitOffsetPagination raises `TypeError` if PAGE_SIZE not set ([#2667](https://github.com/encode/django-rest-framework/issues/2667), [#2700](https://github.com/encode/django-rest-framework/issues/2700))
*   German translation for `min_value` field error message references `max_value`. ([#2645](https://github.com/encode/django-rest-framework/issues/2645))
*   Remove `MergeDict`. ([#2640](https://github.com/encode/django-rest-framework/issues/2640))
*   Support serializing unsaved models with related fields. ([#2637](https://github.com/encode/django-rest-framework/issues/2637), [#2641](https://github.com/encode/django-rest-framework/issues/2641))
*   Allow blank/null on radio.html choices. ([#2631](https://github.com/encode/django-rest-framework/issues/2631))

**Date**: [5th March 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.1.0+Release%22).

For full details see the [3.1 release announcement](../3.1-announcement/).

* * *

**Date**: [10th February 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.5+Release%22).

*   Fix a bug where `_closable_objects` breaks pickling. ([#1850](https://github.com/encode/django-rest-framework/issues/1850), [#2492](https://github.com/encode/django-rest-framework/issues/2492))
*   Allow non-standard `User` models with `Throttling`. ([#2524](https://github.com/encode/django-rest-framework/issues/2524))
*   Support custom `User.db_table` in TokenAuthentication migration. ([#2479](https://github.com/encode/django-rest-framework/issues/2479))
*   Fix misleading `AttributeError` tracebacks on `Request` objects. ([#2530](https://github.com/encode/django-rest-framework/issues/2530), [#2108](https://github.com/encode/django-rest-framework/issues/2108))
*   `ManyRelatedField.get_value` clearing field on partial update. ([#2475](https://github.com/encode/django-rest-framework/issues/2475))
*   Removed '.model' shortcut from code. ([#2486](https://github.com/encode/django-rest-framework/issues/2486))
*   Fix `detail_route` and `list_route` mutable argument. ([#2518](https://github.com/encode/django-rest-framework/issues/2518))
*   Prefetching the user object when getting the token in `TokenAuthentication`. ([#2519](https://github.com/encode/django-rest-framework/issues/2519))

**Date**: [28th January 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.4+Release%22).

*   Django 1.8a1 support. ([#2425](https://github.com/encode/django-rest-framework/issues/2425), [#2446](https://github.com/encode/django-rest-framework/issues/2446), [#2441](https://github.com/encode/django-rest-framework/issues/2441))
*   Add `DictField` and support Django 1.8 `HStoreField`. ([#2451](https://github.com/encode/django-rest-framework/issues/2451), [#2106](https://github.com/encode/django-rest-framework/issues/2106))
*   Add `UUIDField` and support Django 1.8 `UUIDField`. ([#2448](https://github.com/encode/django-rest-framework/issues/2448), [#2433](https://github.com/encode/django-rest-framework/issues/2433), [#2432](https://github.com/encode/django-rest-framework/issues/2432))
*   `BaseRenderer.render` now raises `NotImplementedError`. ([#2434](https://github.com/encode/django-rest-framework/issues/2434))
*   Fix timedelta JSON serialization on Python 2.6\. ([#2430](https://github.com/encode/django-rest-framework/issues/2430))
*   `ResultDict` and `ResultList` now appear as standard dict/list. ([#2421](https://github.com/encode/django-rest-framework/issues/2421))
*   Fix visible `HiddenField` in the HTML form of the web browsable API page. ([#2410](https://github.com/encode/django-rest-framework/issues/2410))
*   Use `OrderedDict` for `RelatedField.choices`. ([#2408](https://github.com/encode/django-rest-framework/issues/2408))
*   Fix ident format when using `HTTP_X_FORWARDED_FOR`. ([#2401](https://github.com/encode/django-rest-framework/issues/2401))
*   Fix invalid key with memcached while using throttling. ([#2400](https://github.com/encode/django-rest-framework/issues/2400))
*   Fix `FileUploadParser` with version 3.x. ([#2399](https://github.com/encode/django-rest-framework/issues/2399))
*   Fix the serializer inheritance. ([#2388](https://github.com/encode/django-rest-framework/issues/2388))
*   Fix caching issues with `ReturnDict`. ([#2360](https://github.com/encode/django-rest-framework/issues/2360))

**Date**: [8th January 2015](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.3+Release%22).

*   Fix `MinValueValidator` on `models.DateField`. ([#2369](https://github.com/encode/django-rest-framework/issues/2369))
*   Fix serializer missing context when pagination is used. ([#2355](https://github.com/encode/django-rest-framework/issues/2355))
*   Namespaced router URLs are now supported by the `DefaultRouter`. ([#2351](https://github.com/encode/django-rest-framework/issues/2351))
*   `required=False` allows omission of value for output. ([#2342](https://github.com/encode/django-rest-framework/issues/2342))
*   Use textarea input for `models.TextField`. ([#2340](https://github.com/encode/django-rest-framework/issues/2340))
*   Use custom `ListSerializer` for pagination if required. ([#2331](https://github.com/encode/django-rest-framework/issues/2331), [#2327](https://github.com/encode/django-rest-framework/issues/2327))
*   Better behavior with null and '' for blank HTML fields. ([#2330](https://github.com/encode/django-rest-framework/issues/2330))
*   Ensure fields in `exclude` are model fields. ([#2319](https://github.com/encode/django-rest-framework/issues/2319))
*   Fix `IntegerField` and `max_length` argument incompatibility. ([#2317](https://github.com/encode/django-rest-framework/issues/2317))
*   Fix the YAML encoder for 3.0 serializers. ([#2315](https://github.com/encode/django-rest-framework/issues/2315), [#2283](https://github.com/encode/django-rest-framework/issues/2283))
*   Fix the behavior of empty HTML fields. ([#2311](https://github.com/encode/django-rest-framework/issues/2311), [#1101](https://github.com/encode/django-rest-framework/issues/1101))
*   Fix Metaclass attribute depth ignoring fields attribute. ([#2287](https://github.com/encode/django-rest-framework/issues/2287))
*   Fix `format_suffix_patterns` to work with Django's `i18n_patterns`. ([#2278](https://github.com/encode/django-rest-framework/issues/2278))
*   Ability to customize router URLs for custom actions, using `url_path`. ([#2010](https://github.com/encode/django-rest-framework/issues/2010))
*   Don't install Django REST Framework as egg. ([#2386](https://github.com/encode/django-rest-framework/issues/2386))

**Date**: [17th December 2014](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.2+Release%22).

*   Ensure `request.user` is made available to response middleware. ([#2155](https://github.com/encode/django-rest-framework/issues/2155))
*   `Client.logout()` also cancels any existing `force_authenticate`. ([#2218](https://github.com/encode/django-rest-framework/issues/2218), [#2259](https://github.com/encode/django-rest-framework/issues/2259))
*   Extra assertions and better checks to preventing incorrect serializer API use. ([#2228](https://github.com/encode/django-rest-framework/issues/2228), [#2234](https://github.com/encode/django-rest-framework/issues/2234), [#2262](https://github.com/encode/django-rest-framework/issues/2262), [#2263](https://github.com/encode/django-rest-framework/issues/2263), [#2266](https://github.com/encode/django-rest-framework/issues/2266), [#2267](https://github.com/encode/django-rest-framework/issues/2267), [#2289](https://github.com/encode/django-rest-framework/issues/2289), [#2291](https://github.com/encode/django-rest-framework/issues/2291))
*   Fixed `min_length` message for `CharField`. ([#2255](https://github.com/encode/django-rest-framework/issues/2255))
*   Fix `UnicodeDecodeError`, which can occur on serializer `repr`. ([#2270](https://github.com/encode/django-rest-framework/issues/2270), [#2279](https://github.com/encode/django-rest-framework/issues/2279))
*   Fix empty HTML values when a default is provided. ([#2280](https://github.com/encode/django-rest-framework/issues/2280), [#2294](https://github.com/encode/django-rest-framework/issues/2294))
*   Fix `SlugRelatedField` raising `UnicodeEncodeError` when used as a multiple choice input. ([#2290](https://github.com/encode/django-rest-framework/issues/2290))

**Date**: [11th December 2014](https://github.com/encode/django-rest-framework/issues?q=milestone%3A%223.0.1+Release%22).

*   More helpful error message when the default Serializer `create()` fails. ([#2013](https://github.com/encode/django-rest-framework/issues/2013))
*   Raise error when attempting to save serializer if data is not valid. ([#2098](https://github.com/encode/django-rest-framework/issues/2098))
*   Fix `FileUploadParser` breaks with empty file names and multiple upload handlers. ([#2109](https://github.com/encode/django-rest-framework/issues/2109))
*   Improve `BindingDict` to support standard dict-functions. ([#2135](https://github.com/encode/django-rest-framework/issues/2135), [#2163](https://github.com/encode/django-rest-framework/issues/2163))
*   Add `validate()` to `ListSerializer`. ([#2168](https://github.com/encode/django-rest-framework/issues/2168), [#2225](https://github.com/encode/django-rest-framework/issues/2225), [#2232](https://github.com/encode/django-rest-framework/issues/2232))
*   Fix JSONP renderer failing to escape some characters. ([#2169](https://github.com/encode/django-rest-framework/issues/2169), [#2195](https://github.com/encode/django-rest-framework/issues/2195))
*   Add missing default style for `FileField`. ([#2172](https://github.com/encode/django-rest-framework/issues/2172))
*   Actions are required when calling `ViewSet.as_view()`. ([#2175](https://github.com/encode/django-rest-framework/issues/2175))
*   Add `allow_blank` to `ChoiceField`. ([#2184](https://github.com/encode/django-rest-framework/issues/2184), [#2239](https://github.com/encode/django-rest-framework/issues/2239))
*   Cosmetic fixes in the HTML renderer. ([#2187](https://github.com/encode/django-rest-framework/issues/2187))
*   Raise error if `fields` on serializer is not a list of strings. ([#2193](https://github.com/encode/django-rest-framework/issues/2193), [#2213](https://github.com/encode/django-rest-framework/issues/2213))
*   Improve checks for nested creates and updates. ([#2194](https://github.com/encode/django-rest-framework/issues/2194), [#2196](https://github.com/encode/django-rest-framework/issues/2196))
*   `validated_attrs` argument renamed to `validated_data` in `Serializer` `create()`/`update()`. ([#2197](https://github.com/encode/django-rest-framework/issues/2197))
*   Remove deprecated code to reflect the dropped Django versions. ([#2200](https://github.com/encode/django-rest-framework/issues/2200))
*   Better serializer errors for nested writes. ([#2202](https://github.com/encode/django-rest-framework/issues/2202), [#2215](https://github.com/encode/django-rest-framework/issues/2215))
*   Fix pagination and custom permissions incompatibility. ([#2205](https://github.com/encode/django-rest-framework/issues/2205))
*   Raise error if `fields` on serializer is not a list of strings. ([#2213](https://github.com/encode/django-rest-framework/issues/2213))
*   Add missing translation markers for relational fields. ([#2231](https://github.com/encode/django-rest-framework/issues/2231))
*   Improve field lookup behavior for dicts/mappings. ([#2244](https://github.com/encode/django-rest-framework/issues/2244), [#2243](https://github.com/encode/django-rest-framework/issues/2243))
*   Optimized hyperlinked PK. ([#2242](https://github.com/encode/django-rest-framework/issues/2242))

**Date**: 1st December 2014

For full details see the [3.0 release announcement](../3.0-announcement/).

* * *

For older release notes, [please see the version 2.x documentation](https://github.com/encode/django-rest-framework/blob/version-2.4.x/docs/topics/release-notes.md).