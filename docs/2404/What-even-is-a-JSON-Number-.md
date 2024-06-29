<!--yml

category: 未分类

date: 2024-05-27 12:47:10

-->

# JSON 数字是什么？

> 来源：[https://blog.trl.sn/blog/what-is-a-json-number/](https://blog.trl.sn/blog/what-is-a-json-number/)

<main id="skip">

# JSON 数字是什么？

并不是人们通常考虑的问题，看起来相当直接。这是一个数字，*显而易见*！但事实证明，这个问题有些难以回答，尤其对于 API 设计者来说，答案非常重要！因此，让我们通过深入研究各种 JSON 规范和实现来探索一下。研究结果在文末总结，如果只想知道答案而不想深入探讨，可以直接跳到底部。

## 目录

## 权威来源

JSON 由两个主要标准定义：[Ecma-404](https://ecma-international.org/wp-content/uploads/ECMA-404_2nd_edition_december_2017.pdf) 和 [RFC 8259](https://datatracker.ietf.org/doc/html/rfc8259)。这两个标准在语义上是相同的，但 RFC 8259 提供了一些额外的建议，用于良好的互操作性。一个相关的标准，[RFC 7493](https://datatracker.ietf.org/doc/html/rfc7493)，描述了紧密相关的 Internet JSON 格式，这是 JSON 的一个限制配置文件，它在 RFC 8259 中的建议中添加了更多的约束。此外，在 API 描述的背景下，[JSON Schema](https://json-schema.org/draft/2020-12/json-schema-core) 定义了一个数字数据类型，这也是 [OpenAPI](https://spec.openapis.org/oas/v3.1.0) 正式引用的。

### ECMA-404

> 数字是一系列十进制数字，不含不必要的前导零。它可能有一个前置减号（U+002D）。它可能有一个由小数点（U+002E）前缀的小数部分。它可能有一个指数，由 e（U+0065）或 E（U+0045）前缀，并可选地有 +（U+002B）或 -（U+002D）。

因此，JSON 数字是一个带有可选符号、小数部分和指数的数字序列。这个描述纯粹是句法的。

### RFC 8259

这个规范提供了一个与 ECMA-404 铁路图相等的 ABNF 语法。它还明确允许实现对接受的数字的范围和精度设置限制。它继续指出：

> 由于广泛使用实现 IEEE 754 二进制 64 位（双精度）数的软件，可以通过期望不超过其提供的精度或范围的实现来实现良好的互操作性。

这表明，一些 JSON 实现使用双精度数来存储 JSON 数值。在浏览器中找到的并被全球数十亿人使用的实现就是这样一个实现。因此，如果 JSON 数字的范围和精度符合双精度数的要求，则可以实现互操作性。

### RFC 7493

此规范使 RFC 8259 的信息性注释成为规范上的 SHOULD NOT：

> I-JSON消息不应包含表达比IEEE 754双精度数提供的更大幅度或精度的数字

它继续建议，如果需要更大的范围或精度，则应将数字编码为字符串。

### JSON Schema和OpenAPI

JSON Schema将数字描述为：

> 从JSON“number”值开始的任意精度的十进制数值

JSON Schema和OpenAPI还定义了整数的概念。 JSON schema根据其值定义整数，即零分数部分的数字。它还指出，整数值不应以分数部分编码。 OpenAPI根据其语法定义整数，作为没有小数部分或指数部分的JSON数字。

## 实践中的JSON数字

RFC 8259提出了一个重要观点，即最终实现决定了JSON数字是什么。在实践中，确实存在范围和精度的限制，但是它们是什么？我们知道至少有一个广泛部署的实现受到双精度限制。还有其他互操作性问题需要考虑吗？让我们通过两条并行轨道进行调查：跨一些常见语言的JSON解析器和序列化器，以及OpenAPI生态系统中的代码生成器。

### 语言实现

语言实现最终决定了JSON数字是什么，因此让我们看几个例子，并检查是否存在常见模式。对于具有可配置序列化/反序列化功能的语言，只涵盖默认行为。

#### JavaScript

JavaScript的内置JSON实现仅适用于内置的`Number`类型，因此所有值都限制在double的范围和精度内。默认情况下不支持BigInt的序列化。JavaScript还使得在往返过程中无法精确保留数值文字，例如将整数作为小数（如`1.0`）放回时，会将其作为`1`发送到网络。有一个[语言提案](https://github.com/tc39/proposal-json-parse-with-source)可以解决这些问题，而不用替换整个解析器。

#### Python 3.8

整数、小数和指数处理方式不同。整数可以作为JSON数字在-10^(4299)到10^(4299)范围内往返，而小数和指数使用double，因此受到double范围和精度的限制。超出int范围的整数在序列化时会导致`ValueError`。超出double范围的指数和小数结果为`inf`。在解析和序列化指数时，指数格式会丢失。

#### C#（.NET 8，System.Text.JSON）

C#的`System.Text.JSON`库是处理JSON数据的推荐方式，尽管`Newtonsoft.JSON`也常用。我们检查前者的行为，后者可能有所不同。

C# 支持通过 `TryGet*` API 将数据反序列化为适当的数据类型。使用此 API，可以无损地反序列化整数类型，最多到 `int64`，并将稍大的整数反序列化为 `decimal`。`decimal` 也可以用于表示小数值，可能会有精度损失。如果事先知道模式，可以添加对其他数据类型（如 `BigInteger`）的反序列化支持。

C# 还支持获取字面量的原始文本，允许自定义处理和往返任意字面量而不会丢失精度。

#### Java（JDK 11+，Jackson）

Java 通常使用 `Jackson` 库来处理 JSON 的序列化和反序列化。`Jackson` 允许序列化和反序列化到任何 Java 数字类型，包括 `BigDecimal`，从而可以表示任何范围和精度的数字字面量而不会丢失精度。

#### Rust（serde）

Rust 的 `serde_json` 库通常用于 JSON 的序列化和反序列化。它支持反序列化符合 `i64`/`u64` 范围内的整数值。它还支持反序列化符合 `f64` 范围内的整数和小数值，尽管这样做可能会导致精度损失。超出 `f64` 范围的整数和小数将导致错误。指数始终反序列化为 double 类型。但是，`serde` 具有一个 `arbitrary_precision` 配置标志，可以用于往返任意数字值而不会丢失精度，假设它们没有以丢失精度的方式反序列化。可以通过一些额外的代码将支持添加到其他数据类型中，但需要知道数据的模式。

#### Go

Go 的 `encoding/json` 库能够动态地将 JSON 数字字面量解组为 `float64` 类型。如果事先知道模式，可以将已知整数解析为适当的整数类型，最多到 `int64`，并根据需要将小数解析为 `float32` 类型。可以通过一些额外的代码添加对例如小数类型或大整数类型的解组支持，但同样需要知道数据的模式。

Go 可以指示将数字字面量反序列化为字符串，允许自定义处理和往返任意字面量而不会丢失精度。

#### 摘要

总结各种实现的行为，我们可以检查它们对以下值的行为：

| 数字字面量 | 描述 |
| --- | --- |
| 10 | 小整数 |
| 1000000000 | 中等整数：超出 int32 范围，但在 int64 范围内 |
| 10000000000000001 | 大整数：超出 double 范围，但在 int64 范围内 |
| 100000000000000000001 | 巨大整数：超出 int64 范围 |
| 1[309 个0] | 荒谬整数：超出 decimal128 范围 |
| 10.0 | 低精度小数 |
| 10000000000000001.1 | 高精度小数：精度 > double |
| 1.[34 个1] | 荒谬精度小数：精度 > decimal128 |
| 1E2 | 小指数形式 |
| 1E309 | 大指数：超出 float 范围 |

下表展示了每种语言中用于表示文本字面值的数据类型。灰色单元格表示错误，红色单元格表示有精度损失但非错误。这里仅涵盖默认的序列化行为。可以通过配置序列化器或其他机制来防御此类错误。此外，我使用的测试代码尝试在可用时使用动态/无模式解析路径。对于某些语言，提前知道模式可以导致更好的行为。测试代码可以在附录中找到。

| 字面值 | JavaScript | C# | Python | Java | Go | Rust |
| --- | --- | --- | --- | --- | --- | --- |
| 小整数 | 数字 | int16 | int | int | float64 | i8 |
| 中等整数 | 数字 | int64 | int | long | float64 | i64 |
| 大整数 | 数字 | int64 | int | long | float64 | i64 |
| 巨大整数 | 数字 | 小数 | int | BigInteger | float64 | f64 |
| 极度荒谬的整数 | 数字 | error | int | BigInteger | error | error |
| 低精度的小数 | 数字 | 小数 | double | float | float64 | f64 |
| 高精度的小数 | 数字 | 小数 | double | BigDecimal | float64 | f64 |
| 极度精确的小数 | 数字 | 小数 | double | BigDecimal | float64 | f64 |
| 小指数 | 数字 | 小数 | double | float | float64 | f64 |
| 大指数 | 数字 | error | double | BigDecimal | error | error |

Notes:

+   **C#**: 测试代码尝试反序列化为最小支持的 int `int16`，然后是 `int64`，然后是 `decimal`。此时不尝试反序列化为 `double`，因为它只会具有`+/- Infinity`的值。还请注意，`decimal`不是 IEEE `decimal128` - 前者具有 28-29 位有效数字，而 `decimal128` 则有 34 位。

+   **Rust**: 此列中的类型表示将值反序列化为的最小可能类型，而不会出错。如果您事先知道模式，则可以将巨大整数情况轻松处理为错误，以避免精度损失。

+   **Go**: 如果您知道数据的模式并且可以要求解码器给您一个 int64，则可以将其序列化为整数类型。该表格表示如果您事先不知道模式，则可以做的最好的处理方式。如果您知道模式，则可以处理大整数情况而不会损失精度，并且还可以将巨大整数情况视为错误。

### OpenAPI 代码生成器

在 JSON API 的上下文中，可以提出这样的论点：OpenAPI 及其代码生成器的生态系统与各种实现中的解析器同样重要。即使语言的 JSON 解析器能够解析特定大小的数字文本字面值，OpenAPI 的签名也可能更或更不受限制，特别是对于强类型语言。

为了了解各种语言的行为，我们将使用 OpenAPI 3 规范本身定义的数字类型和格式以及[OpenAPI 格式注册表](https://spec.openapis.org/registry/format/)中定义的数字格式进行测试：

| 类型 | 格式 | 描述 |
| --- | --- | --- |
| number |  | Arbitrary-precision, base-10 decimal number value |
| integer |  | JSON 中没有小数或指数部分的数字 |
| number | float | 单精度浮点数 |
| number | double | 双精度浮点数 |
| number | decimal | 未指定精度和范围的固定点小数 |
| number | decimal128 | 有34位有效十进制数字的十进制浮点数 |
| integer | int8 | Signed 8-bit integer |
| integer | uint8 | Unsigned 8-bit integer |
| integer | int16 | Signed 16-bit integer |
| integer | int32 | Signed 32-bit integer |
| integer | int64 | Signed 64-bit integer |

除了 `uint8` 之外的无符号整数在 OpenAPI 或格式注册表中都没有定义。`double-int` 最近添加，不太可能在任何地方得到支持。

下表总结了使用[OpenAPI-Generator](https://openapi-generator.tech/)代码生成器时每种语言的输出。红色高亮显示的单元格显示在提供 OpenAPI 规范定义的范围和精度中提供某些值时可能会导致精度损失或错误的情况。

| OpenAPI | JavaScript | C# | Python | Java | Go | Rust |
| --- | --- | --- | --- | --- | --- | --- |
| number | number | decimal128 | int, float | BigDecimal | float32 | f32 |
| integer | number | int32 | int | Integer | int32 | i32 |
| int8 | number | int32 | int | Integer | int32 | i32 |
| uint8 | number | int32 | int | Integer | int32 | i32 |
| int16 | number | int32 | int | Integer | int32 | i32 |
| int32 | number | int32 | int | Integer | int32 | i32 |
| double-int | number | int32 | int | Integer | int32 | i32 |
| int64 | number | int64 | int | Long | int64 | i64 |
| single | number | float | int, float | Float | float32 | f32 |
| double | number | double | int, float | Double | float64 | f64 |
| decimal | number | decimal128 | int, float | BigDecimal | float32 | f32 |
| decimal128 | number | decimal128 | int, float | BigDecimal | float32 | f32 |

从这里我们可以看出，OpenAPI 生成器套件认为一个`integer`是一个int32，尽管规范建议它具有任意范围。因此，在使用这些工具时，似乎没有办法定义一个在所有具有相应数据类型的语言中都具有任意长度的整数。此外，尽管规范建议`number`具有任意范围和精度，但通常被理解为32位的`float`。

## 发现总结

让我们来回答这个问题：JSON 数字是什么？

+   根据权威规范，任意长度和精度的数字字面量。

+   根据JSON的可互操作性概要，数字文字具有双精度的长度和精度。

+   根据各种JSON实现，数字文字具有因实现而异的约束。

+   根据OpenAPI，要么：

    +   任意长度的整数

    +   任意长度和精度的十进制值

+   根据OpenAPI代码生成器，

    +   如果没有格式，则：

        +   对于`number`，一些浮点表示形式，如float32那样小

        +   对于`integer`，一个int32

    +   如果有格式，则：

        +   在该语言中，最接近指定格式的数据类型近似值

        +   如果格式不受支持，则使用`int32`或其最接近的近似值。

我们还确认了双倍范围外数字的互操作性。所有经过测试的实现都可以安全地传输双精度范围内的数字。除JavaScript外的所有实现都可以传输int64范围内的整数字面量（尽管Go需要事先知道模式）。

对于那些使用OpenAPI的人，我们可以推断出一些关于定义使用数字的API的最佳实践：

+   在OpenAPI中始终指定格式。`integer`或`number`都不太可能符合您的要求。

+   避免使用`double-int`。它不受支持，并且在许多语言中可能导致错误或数据丢失。

+   避免使用`decimal`和`decimal128`。它们也不被广泛支持。

+   如果有JavaScript消费者，请避免使用带有`number`类型的`int64`。请改用`string`类型。

如果您使用TypeSpec，此建议可以总结为：

+   避免使用`decimal`、`decimal128`、`integer`和`numeric`类型。

+   如果使用`int64`并且有JavaScript消费者，请通过`@encode("int64", string)`将其编码为字符串。

由于某些语言处理数字文字与指数和小数部分的方式不同，实现时应保留往返格式（例如`10.0`应还原为`10.0`）。

## 附录：测试代码

此代码是垃圾，LLMs在其创建中大量参与。输出需要一些解释。欢迎在[GitHub上提出改进建议](https://github.com/bterlson/blog/blob/main/content/blog/what-is-a-json-number.md)。

### JavaScript

```
const jsonValues = [
 "10",
 "1000000000",
 "10000000000000001",
 "100000000000000000001",
 "1" + "0".repeat(309),
 "10.0",
 "10000000000000001.1",
 "1.1111111111111111111111111111111111",
 "1E2",
 "1E309",
];
 for (const jsonValue of jsonValues) {
 console.log(`Testing JSON value: ${jsonValue}`);
 try {
 // Deserialize the JSON value
 const deserialized = JSON.parse(jsonValue);
 if (String(deserialized) !== jsonValue) {
 console.log("precision loss detected", jsonValue, deserialized);
 }
 const serialized = JSON.stringify(deserialized);
 if (jsonValue !== serialized) {
 console.log("round-trip error detected", jsonValue, serialized);
 }
 } catch (error) {
 console.log(`Deserialization error: ${error.message}`);
 }
 console.log();
}
```

### C#

```
using System;
using System.Text.Json;
 class Program
{
 static void Main()
 {
 string[] jsonValues = {
 "10",
 "1000000000",
 "10000000000000001",
 "100000000000000000001",
 "1" + new string('0', 309),
 "10.0",
 "10000000000000001.1",
 "1.1111111111111111111111111111111111",
 "1E2",
 "1E309",
 };
 foreach (string jsonValue in jsonValues)
 {
 Console.WriteLine($"Testing JSON value: {jsonValue}");
 try
 {
 // Deserialize the JSON value
 JsonElement deserialized = JsonSerializer.Deserialize<JsonElement>(jsonValue);
 // Check the deserialized type and precision loss
 switch (deserialized.ValueKind)
 {
 case JsonValueKind.Number:
 if (deserialized.TryGetInt16(out short smallValue))
 {
 Console.WriteLine("Deserialized as: short");
 }
 else if (deserialized.TryGetInt64(out long longValue))
 {
 Console.WriteLine("Deserialized as: long");
 }
 else if (deserialized.TryGetDecimal(out decimal decimalValue))
 {
 Console.WriteLine("Deserialized as: decimal");
 string deserializedString = decimalValue.ToString("G29");
 if (deserializedString != jsonValue)
 {
 Console.WriteLine("Precision loss detected!");
 Console.WriteLine($"Original value: {jsonValue}");
 Console.WriteLine($"Deserialized value: {deserializedString}");
 }
 }
 else
 {
 Console.WriteLine("Deserialized as: unknown number");
 }
 break;
 default:
 Console.WriteLine($"Deserialized as: {deserialized.ValueKind}");
 break;
 }
 // Serialize the value back to JSON
 string serialized = JsonSerializer.Serialize(deserialized);
 // Check if the serialized value matches the original JSON value
 if (serialized != jsonValue)
 {
 Console.WriteLine("Round-tripping error detected!");
 Console.WriteLine($"Original: {jsonValue}");
 Console.WriteLine($"Serialized: {serialized}");
 }
 }
 catch (JsonException ex)
 {
 Console.WriteLine($"Deserialization error: {ex.Message}");
 }
 Console.WriteLine();
 }
 }
}
```

### Python（3.8）

```
import json
import decimal
 def test_json_number(number_literal):
 print(f"Testing number literal: {number_literal}")
 # Deserialize the JSON number
 deserialized = json.loads(number_literal)
 # Check for precision loss during deserialization
 if str(deserialized) != number_literal:
 print("  Precision loss during deserialization")
 else:
 print("  No precision loss during deserialization")
 # Serialize the deserialized number back to JSON
 serialized = json.dumps(deserialized)
 # Check for round-tripping errors
 if serialized != number_literal:
 print("  Round-tripping error")
 else:
 print("  No round-tripping error")
 print()
 # Test the JSON number literals
test_json_number("10")
test_json_number("1000000000")
test_json_number("10000000000000001")
test_json_number("100000000000000000001")
test_json_number("1" + "0" * 4301)
test_json_number("10.0")
test_json_number("10000000000000001.1")
test_json_number("1." + "1" * 34)
test_json_number("1E2")
test_json_number("1E309")
```

### Java（JDK 21，Jackson）

```
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;
 public class NumberTest {
 private static final String[] testCases = {
 "10",
 "1000000000",
 "10000000000000001",
 "100000000000000000000",
 "1" + "0".repeat(309),
 "10.0",
 "10000000000000001.1",
 "1.1111111111111111111111111111111111",
 "1E2",
 "1E309"
 };
 public static void main(String[] args) {
 ObjectMapper objectMapper = new ObjectMapper();
 for (String testCase : testCases) {
 System.out.println("Testing JSON value: " + testCase);
 try {
 // Parse the JSON value
 JsonNode jsonNode = objectMapper.readTree(testCase);
 // Check the deserialized type and precision loss
 if (jsonNode.isInt()) {
 System.out.println("Deserialized as: int");
 } else if (jsonNode.isLong()) {
 System.out.println("Deserialized as: long");
 } else if (jsonNode.isBigInteger()) {
 System.out.println("Deserialized as: BigInteger");
 String deserializedString = jsonNode.bigIntegerValue().toString();
 if (!deserializedString.equals(testCase)) {
 System.out.println("Precision loss detected!");
 System.out.println("Original value: " + testCase);
 System.out.println("Deserialized value: " + deserializedString);
 }
 } else if (jsonNode.isDouble()) {
 System.out.println("Deserialized as: double");
 String deserializedString = jsonNode.doubleValue() + "";
 if (!deserializedString.equals(testCase)) {
 System.out.println("Precision loss detected!");
 System.out.println("Original value: " + testCase);
 System.out.println("Deserialized value: " + deserializedString);
 }
 } else if (jsonNode.isDecimal()) {
 System.out.println("Deserialized as: BigDecimal");
 String deserializedString = jsonNode.decimalValue().toString();
 if (!deserializedString.equals(testCase)) {
 System.out.println("Precision loss detected!");
 System.out.println("Original value: " + testCase);
 System.out.println("Deserialized value: " + deserializedString);
 }
 } else {
 System.out.println("Deserialized as: " + jsonNode.getNodeType());
 }
 // Serialize the value back to JSON
 String serialized = objectMapper.writeValueAsString(jsonNode);
 // Check if the serialized value matches the original JSON value
 if (!serialized.equals(testCase)) {
 System.out.println("Round-tripping error detected!");
 System.out.println("Original: " + testCase);
 System.out.println("Serialized: " + serialized);
 }
 } catch (Exception e) {
 System.out.println("Deserialization error: " + e.getMessage());
 }
 System.out.println();
 }
 }
}
```

### Rust

```
use serde_json::Value;
 fn main() {
 let json_values = vec![
 "10",
 "1000000000",
 "10000000000000001",
 "100000000000000000001",
 "1000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
 "10.0",
 "10000000000000001.1",
 "1.1111111111111111111111111111111111",
 "1E2",
 "1E309",
 ];
 for json_value in json_values {
 println!("Testing JSON value: {}", json_value);
 // Deserialize the JSON value
 let deserialized: Result<Value, _> = serde_json::from_str(json_value);
 match deserialized {
 Ok(value) => {
 // Check the deserialized type and precision loss
 match &value {
 Value::Number(num) => {
 if num.is_i64() {
 println!("Deserialized as: i64");
 } else if num.is_u64() {
 println!("Deserialized as: u64");
 } else if num.is_f64() {
 println!("Deserialized as: f64");
 let deserialized_value = num.as_f64().unwrap().to_string();
 if deserialized_value != json_value {
 println!("Precision loss detected!");
 println!("Original value: {}", json_value);
 println!("Deserialized value: {}", deserialized_value);
 }
 }
 }
 _ => {
 println!("Deserialized as: {:?}", value);
 }
 }
 // Serialize the value back to JSON
 let serialized = serde_json::to_string(&value).unwrap();
 // Check if the serialized value matches the original JSON value
 if serialized != json_value {
 println!("Round-tripping error detected!");
 println!("Original: {}", json_value);
 println!("Serialized: {}", serialized);
 }
 }
 Err(e) => {
 println!("Deserialization error: {}", e);
 }
 }
 println!();
 }
}
```

### Go

此代码演示了默认行为：

```
package main
 import (
 "encoding/json"
 "fmt"
 "strconv"
 "strings"
)
 func main() {
 testCases := []string{
 "10",
 "1000000000",
 "10000000000000001",
 "100000000000000000000",
 "1" + strings.Repeat("0", 309),
 "10.0",
 "10000000000000001.1",
 "1.1111111111111111111111111111111111",
 "1E2",
 "1E309",
 }
 for _, testCase := range testCases {
 fmt.Printf("Testing JSON value: %s\n", testCase)
 // Unmarshal the JSON value into a float64
 var value float64
 err := json.Unmarshal([]byte(testCase), &value)
 if err != nil {
 fmt.Printf("Deserialization error: %v\n", err)
 fmt.Println()
 continue
 }
 fmt.Println("Deserialized as: float64")
 // Check for precision loss
 deserializedString := strconv.FormatFloat(value, 'g', -1, 64)
 if deserializedString != testCase {
 fmt.Println("Precision loss detected!")
 fmt.Printf("Original value: %s\n", testCase)
 fmt.Printf("Deserialized value: %s\n", deserializedString)
 }
 // Serialize the value back to JSON
 serialized, err := json.Marshal(value)
 if err != nil {
 fmt.Printf("Serialization error: %v\n", err)
 fmt.Println()
 continue
 }
 // Check if the serialized value matches the original JSON value
 if string(serialized) != testCase {
 fmt.Println("Round-tripping error detected!")
 fmt.Printf("Original: %s\n", testCase)
 fmt.Printf("Serialized: %s\n", string(serialized))
 }
 fmt.Println()
 }
}
```

此代码演示了将反序列化为已知形状的结构体：

```
package main
 import (
 "encoding/json"
 "fmt"
 "math/big"
 "strconv"
 "strings"
)
 type TestCase struct {
 Name string `json:"name"`
 Int8 int8 `json:"int8,omitempty"`
 Int16 int16 `json:"int16,omitempty"`
 Int32 int32 `json:"int32,omitempty"`
 Int64 int64 `json:"int64,omitempty"`
 Float float64 `json:"float,omitempty"`
}
 func main() {
 testCases := []string{
 `{"name": "Small integer", "int8": 10}`,
 `{"name": "Medium integer", "int32": 1000000000}`,
 `{"name": "Large integer", "int64": 10000000000000001}`,
 `{"name": "Huge integer", "int64": 100000000000000000001}`,
 `{"name": "Ridonculous integer", "int64": 1` + strings.Repeat("0", 309) + `}`,
 `{"name": "Low-precision decimal", "float": 10.0}`,
 `{"name": "High-precision decimal", "float": 10000000000000001.1}`,
 `{"name": "Ridonculous-precision decimal", "float": 1.1111111111111111111111111111111111}`,
 `{"name": "Small exponential", "float": 1E2}`,
 `{"name": "Large exponential", "float": 1E309}`,
 }
 for _, testCase := range testCases {
 var tc TestCase
 err := json.Unmarshal([]byte(testCase), &tc)
 if err != nil {
 fmt.Printf("Deserialization error: %v\n", err)
 fmt.Println()
 continue
 }
 fmt.Printf("Testing: %s\n", tc.Name)
 // Check the deserialized type and precision loss
 switch {
 case tc.Int8 != 0:
 fmt.Println("Deserialized as: int8")
 case tc.Int16 != 0:
 fmt.Println("Deserialized as: int16")
 case tc.Int32 != 0:
 fmt.Println("Deserialized as: int32")
 case tc.Int64 != 0:
 fmt.Println("Deserialized as: int64")
 if tc.Name == "Ridonculous integer" {
 bigInt := new(big.Int)
 bigInt.SetString(strconv.FormatInt(tc.Int64, 10), 10)
 if bigInt.String() != strconv.FormatInt(tc.Int64, 10) {
 fmt.Println("Precision loss detected!")
 fmt.Printf("Original value: %s\n", strconv.FormatInt(tc.Int64, 10))
 fmt.Printf("Deserialized value: %s\n", bigInt.String())
 }
 }
 default:
 fmt.Println("Deserialized as: float64")
 deserializedString := strconv.FormatFloat(tc.Float, 'g', -1, 64)
 if deserializedString != strconv.FormatFloat(tc.Float, 'f', -1, 64) {
 fmt.Println("Precision loss detected!")
 fmt.Printf("Original value: %s\n", strconv.FormatFloat(tc.Float, 'f', -1, 64))
 fmt.Printf("Deserialized value: %s\n", deserializedString)
 }
 }
 // Serialize the value back to JSON
 serialized, err := json.Marshal(tc)
 if err != nil {
 fmt.Printf("Serialization error: %v\n", err)
 fmt.Println()
 continue
 }
 // Check if the serialized value matches the original JSON value
 var originalTC TestCase
 json.Unmarshal([]byte(testCase), &originalTC)
 var serializedTC TestCase
 json.Unmarshal(serialized, &serializedTC)
 if originalTC != serializedTC {
 fmt.Println("Round-tripping error detected!")
 fmt.Printf("Original: %+v\n", originalTC)
 fmt.Printf("Serialized: %+v\n", serializedTC)
 }
 fmt.Println()
 }
}
```

</main>
