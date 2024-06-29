<!--yml

类别：未分类

日期：2024-05-29 12:31:57

-->

# 使用奇怪的测试捕捉隐含知识 · 应用地图制作

> 来源：[https://jmduke.com/posts/essays/weird-tests-tacit-knowledge/](https://jmduke.com/posts/essays/weird-tests-tacit-knowledge/)

<main class="max-w-prose prose prose-neutral mx-auto flex flex-col items-center [&amp;>*]:w-full">

在 [Buttondown](https://buttondown.email) 上工作 — 或者任何成熟且复杂的代码库 — 需要大量的隐含知识，我在增加工作在这个代码库上的人数时，越来越快地意识到这一点，这是我之前工作做得不好的地方。

*文档*在字面上是一个很好的第一步和最后一步，当然，但是当一个代码库处于“正在被文档化的过程中”时，写下“这是如何做 X”的方法通常并不能真正*解决*确保每个人都能安全快速地做 X 的问题。

我发现一件有用的事情，在将流程转移到左侧的精神中，是在测试中捕捉步骤。这里有一个简单（但是真实的！）例子：向代码库中添加新的 Django 模块。每当你运行 `python manage.py startapp`，你*也*需要将新应用程序添加到许多不同的地方：

+   `pytest.ini`，以便运行测试；

+   `pyproject.toml`，以便进行代码检查；

+   `modules.txt`，以便导出指标。

这个问题的*完美*解决方案是创建一个脚本，自动将新应用程序添加到所有相关位置，并将其添加到 Justfile 中，但这需要思考和错误处理以及一整套其他工作。相比之下，仅仅在测试中捕捉这些约束相对容易：

```
 from django.conf import settings

RELEVANT_FILES = ["./pyproject.toml", "./pytest.ini", "./modules.txt"]

def pytest_generate_tests(metafunc):
    parameters = []
    for filename in RELEVANT_FILES:
        for module in settings.BUTTONDOWN_APPS:
            parameters.append((module, filename))
    metafunc.parametrize("module,filename", parameters)

def test_module_is_present_in_pytest(module: str, filename: str) -> None:
    assert module in open(filename).read()
```

这种方法在你试图强制执行所有*新*代码的规范或不变量时也很有效。（在 Stripe，我们称之为“棘轮测试”方法，尽管初步的搜索似乎表明这个比喻并没有像野火一样传播开来。）

另一个例子：Buttondown 使用 Django-Ninja 从实时 API 生成 OpenAPI 规范。OpenAPI 很棒，但是它[遗憾地缺乏一种符合人体工程学的能力来记录枚举的每个值](https://github.com/OAI/OpenAPI-Specification/issues/348)，因此我们维护一个单独的 `enums.json` 文件，当相关枚举有新添加时需要更新 — 尽管有些枚举值是未记录的！

在这里也可以采用类似的方法：

```
import json

ENUMS_FILENAME = "../shared/enums.json"
OPENAPI_SPEC_FILENAME = "assets/autogen/openapi.json"

RAW_ENUMS = json.load(open(ENUMS_FILENAME))
RAW_OPENAPI_SPEC = json.load(open(OPENAPI_SPEC_FILENAME))

def pytest_generate_tests(metafunc):
    parameters = []
    enums = RAW_ENUMS.keys()
    for enum_name in enums:
        extant_enum_values = RAW_OPENAPI_SPEC["components"]["schemas"][enum_name][
            "enum"
        ]
        for enum_value in extant_enum_values:
            parameters.append((enum_name, enum_value))
    metafunc.parametrize("enum_name,enum_value", parameters)

KNOWN_MISSING_PAIRS = [
    ("CreateSubscriberErrorCode", "metadata_invalid"),
    ("ExternalFeedAutomationCadence", "daily"),
    ("UpdateSubscriberErrorCode", "email_already_exists"),

]

def test_enum_is_exhaustively_documented(enum_name: str, enum_value: str) -> None:
    assert (
        enum_name in RAW_ENUMS
    ), f"Enum {enum_name} is not documented in {ENUMS_FILENAME}"
    if (enum_name, enum_value) in KNOWN_MISSING_PAIRS:
        return
    assert (
        enum_value in RAW_ENUMS[enum_name]
    ), f"Potential value {enum_value} of enum {enum_name} is not documented in {ENUMS_FILENAME}"
```

我觉得*最*可爱的是这种方法的自我记录特性。像“向现有枚举添加新值”这样的任务显然不应该需要搜索内部知识库，但包含有关它的测试可以包含代码指针、技术解释，*还有*修复方式。

总的来说，每当你审查 PR 时一个好的心理锻炼是“一个测试是否能够捕捉到这个问题？”，然后提醒自己，测试定义应该更少地被定义为“执行业务逻辑的东西”，而更多地被定义为“执行你的代码库的脚本”。

</主要部分>
