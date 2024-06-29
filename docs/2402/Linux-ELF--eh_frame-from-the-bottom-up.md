<!--yml

分类：未分类

日期：2024-05-27 14:55:55

-->

# Linux/ELF .eh_frame 从底向上

> 来源：[https://www.corsix.org/content/elf-eh-frame](https://www.corsix.org/content/elf-eh-frame)

**问题陈述**

针对线程的当前寄存器状态，并且只读访问内存，如果当前函数立即返回并且执行在其调用者中继续，寄存器状态假设会变成什么？

值得注意的是，寄存器状态包括（除其他外）指令指针和堆栈指针，因此可以迭代此操作以生成回溯。通过一些扩展以支持调用析构函数和捕获异常，此操作还可以作为抛出异常的基础。

此操作通常称为*展开*（如“展开调用帧”或“展开堆栈”）。

如果某些寄存器在函数返回后总是未定义（因为它们在 ABI 中是易失性的），那么我们无需关心它们的状态。

**可移植性**

寄存器状态的大小和内容取决于所使用的体系结构。我们将稍微简化此过程，通过按序号而非名称引用寄存器，并依赖于 DWARF 寄存器号映射来在序号和名称之间进行转换，例如：

| 序号 | x86 名称 | x86_64 名称 | aarch64 名称 |
| --- | --- | --- | --- |
| 0 | eax | rax | X0 |
| 1 | ecx | rdx | X1 |
| 2 | edx | rcx | X2 |
| 3 | ebx | rbx | X3 |
| 4 | esp | rsi | X4 |
| 5 | ebp | rdi | X5 |
| 6 | esi | rbp | X6 |
| 7 | edi | rsp | X7 |
| 8 |  | r8 | X8 |
| 9 |  | r9 | X9 |
| 10 |  | r10 | X10 |
| 11 |  | r11 | X11 |
| 12 |  | r12 | X12 |
| 13 |  | r13 | X13 |
| 14 |  | r14 | X14 |
| 15 |  | r15 | X15 |
| 16 |  |  | X16 |
| ⋮ |  |  | ⋮ |
| 30 |  |  | X30（LR） |
| 31 |  |  | SP |

因为每个人都喜欢特殊情况，指令指针寄存器（例如 eip / rip / PC）没有被赋予序号。在 aarch64 上，它不需要一个：返回地址总是在*某个*寄存器中（通常是 X30，也称为 LR），因此我们可以使用返回地址寄存器来表示指令指针。相反，在 x86 和 x86_64 上，返回地址放在堆栈而不是寄存器中。对于这些情况，我们将发明一个虚拟寄存器（通常是 x86 上的序号 8，x86_64 上的序号 16），*假装*它包含返回地址，并用它来表示指令指针。

浮点和矢量寄存器在 Linux ABI 中通常是易失性的，因此我们不必太担心它们。

**从小开始：仅展开堆栈指针**

在大多数情况下，函数返回后的假设堆栈指针可以通过将某个寄存器的某个值相加来轻松计算。这可以表示为两个 DWARF 风格的 [ULEB128](https://en.wikipedia.org/wiki/LEB128) 值：一个 ULEB128 用于指定从哪个寄存器开始（通常是当前堆栈指针），然后一个 ULEB128 用于指定要添加的数量。作为前奏，我们将称之为 `DW_CFA_def_cfa`。

**展开其他寄存器**

在大多数情况下，如果函数修改了一个非易失性寄存器，它会在修改之前将其保存到堆栈的某个位置。函数返回时，它会重新加载来自该堆栈插槽的值。这些堆栈插槽的位置可以表示为相对于函数返回后的假设堆栈指针。这可以再次表示为两个 DWARF 风格的 ULEB128 值：一个 ULEB 用于指定我们讨论的寄存器，然后一个 ULEB 用于指定其堆栈插槽的偏移量。只有一个问题：由于偏移量针对的是函数返回后的假设堆栈指针，所需的偏移量将是负数，而 ULEB 无法表示负数。为了解决这个问题，我们将所谓的 `data_align` 值（通常为 -4 或 -8）乘以第二个 ULEB，这也有利于使 ULEB 变小。预示着，我们将其称为 `DW_CFA_offset_extended`。当第一个 ULEB 小于 64 时，我们也可以称之为 `DW_CFA_offset`。

**更多展开其他寄存器的方法**

到目前为止，我们已经描述了如何形成堆栈插槽的地址，然后从中加载。有一种自然的变体跳过第二部分：形成堆栈插槽的地址，然后设置寄存器的值为该地址。

还有另一种常见的变体：通过从寄存器 Y 获取值来取消寄存器 X 的展开（当 X 等于 Y 时，这意味着该寄存器未更改）。

当这些常见的变体不足以满足需求时，我们稍后将描述一个小型表达式语言及其相关的字节码虚拟机，可以描述各种复杂的机制。

这导致以下定义来描述如何展开寄存器：

```
enum StackPointerUnwindMechanism {
  Undefined,
  RegOffset(unsigned RegOrdinal, unsigned Offset),
  ExprResult(uint8_t DwOpBytecode[unsigned Len]),
};

enum OtherRegUnwindMechanism {
  Undefined,
  LoadFromStackSlot(int StackSlotOffset),
  AddressOfStackSlot(int StackSlotOffset),
  CopyFromRegister(unsigned RegOrdinal),
  LoadFromExprResult(uint8_t DwOpBytecode[unsigned Len]),
  ExprResult(uint8_t DwOpBytecode[unsigned Len]),
};

struct UnwindActions {
  StackPointerUnwindMechanism sp;
  OtherRegUnwindMechanism reg[NUM_REG];
}; 
```

随附的展开逻辑如下：

```
struct RegState {
  uintptr_t value[NUM_REG];
};

RegState UnwindOneFrame(RegState* input, UnwindActions* actions) {
  RegState output;
  uintptr_t NewSP;
  switch (actions->sp) {
  case RegOffset(unsigned RegOrdinal, unsigned Offset):
    NewSP = input->value[RegOrdinal] + Offset;
    break;
  case ExprResult(uint8_t DwOpBytecode[unsigned Len]):
    NewSP = EvalExpr(DwOpBytecode, 0, input);
    break;
  }
  output->value[SP] = NewSP;
  for (unsigned i = 0; i < NUM_REG; ++i) {
    uintptr_t NewVal;
    switch (actions->reg[i]) {
    case LoadFromStackSlot(int StackSlotOffset):
      NewVal = *(uintptr_t*)(NewSP + StackSlotOffset);
      break;
    case AddressOfStackSlot(int StackSlotOffset):
      NewVal = NewSP + StackSlotOffset;
      break;
    case CopyFromRegister(unsigned RegOrdinal):
      NewVal = input->value[RegOrdinal];
      break;
    case LoadFromExprResult(uint8_t DwOpBytecode[unsigned Len]):
      NewVal = *(uintptr_t*)EvalExpr(DwOpBytecode, NewSP, input);
      break;
    case ExprResult(uint8_t DwOpBytecode[unsigned Len]):
      NewVal = EvalExpr(DwOpBytecode, NewSP, input);
      break;
    default:
      continue;
    }
    output->value[i] = NewVal;
  }
  return output;
} 
```

定义了一个小型字节码虚拟机来填充 `UnwindActions` 的实例：

| 操作码 | 名称和操作数 | 假设 `UnwindActions* ua` 的语义 |
| --- | --- | --- |
| `0x0c` | DW_CFA_def_cfa(uleb128 寄存器, uleb128 偏移量) | `ua->sp = RegOffset(寄存器, 偏移量)` |
| `0x12` | DW_CFA_def_cfa_sf(uleb128 寄存器, sleb128 偏移量) | `ua->sp = RegOffset(寄存器, 偏移量 * data_align)` |
| `0x0f` | DW_CFA_def_cfa_expression(uleb128 长度, uint8_t DwOpBytecode[长度]) | `ua->sp = ExprResult(DwOpBytecode)` |
| `0x80 + 寄存器` | DW_CFA_offset(uleb128 偏移量) | `ua->reg[寄存器] = LoadFromStackSlot(偏移量 * data_align)` |
| `0x05` | DW_CFA_offset_extended(uleb128 寄存器, uleb128 偏移量) | `ua->reg[寄存器] = LoadFromStackSlot(偏移量 * data_align)` |
| `0x11` | DW_CFA_offset_extended_sf(uleb128 寄存器, sleb128 偏移量) | `ua->reg[寄存器] = LoadFromStackSlot(偏移量 * data_align)` |
| `0x2f` | DW_CFA_GNU_negative_offset_extended(uleb128 寄存器, uleb128 偏移量) | `ua->reg[寄存器] = LoadFromStackSlot(-(偏移量 * data_align))` |
| `0x14` | DW_CFA_val_offset(uleb128 寄存器, uleb128 偏移量) | `ua->reg[寄存器] = AddressOfStackSlot(偏移量 * data_align)` |
| `0x15` | DW_CFA_val_offset_sf(uleb128 寄存器, sleb128 偏移量) | `ua->reg[寄存器] = AddressOfStackSlot(偏移量 * data_align)` |
| `0xc0 + 寄存器` | DW_CFA_restore | `ua->reg[寄存器] = CopyFromRegister(寄存器)` † |
| `0x06` | DW_CFA_restore_extended(uleb128 寄存器) | `ua->reg[寄存器] = CopyFromRegister(寄存器)` † |
| `0x08` | DW_CFA_same_value(uleb128 寄存器) | `ua->reg[寄存器] = CopyFromRegister(寄存器)` |
| `0x09` | DW_CFA_register(uleb128 寄存器, uleb128 源) | `ua->reg[寄存器] = CopyFromRegister(源)` |
| `0x07` | DW_CFA_undefined(uleb128 寄存器) | `ua->reg[寄存器] = Undefined` |
| `0x10` | DW_CFA_expression(uleb128 寄存器, uleb128 长度, uint8_t DwOpBytecode[长度]) | `ua->reg[寄存器] = LoadFromExprResult(DwOpBytecode)` |
| `0x16` | DW_CFA_val_expression(uleb128 寄存器, uleb128 长度, uint8_t DwOpBytecode[长度]) | `ua->reg[寄存器] = ExprResult(DwOpBytecode)` |
| `0x00` | DW_CFA_nop | 空操作 |

† 这与通常的DWARF语义不同。为避免混淆，应使用DW_CFA_same_value代替DW_CFA_restore。

如果字节码没有初始化`sp`，其值为`Undefined`。如果字节码没有初始化`reg[i]`，其值为`CopyFromRegister(i)`。

**在CIE中整合这些内容**

上述的字节码被封装在一个称为CIE的东西中，其中还包括一堆其他字段：

```
struct cie {
  uint32_t length;           
  int32_t  zero;             
  uint8_t  version;          
  char     aug_string[];     
  if (aug_string[0] == 'e' && aug_string[1] == 'h') {
    void* eh_ptr;            
  }
  if (version >= 4) {
    uint8_t addr_size;       
    uint8_t segment_size;    
  }
  uleb128 code_align;        
  sleb128 data_align;        
  if (version == 1) {
    uint8_t return_address_ordinal;
  } else {
    uleb128 return_address_ordinal;
  }
  uint8_t aug_operands[];    
  uint8_t dw_cfa_bytecode[];
  uint8_t zero_padding[];    
}; 
```

`aug_string`字段是一个以NUL结尾的ASCII字符串。在概念上，它是一个标志位掩码，不过每个标志由一个或两个字符表示，而不是单个位。如果标志有关联的操作数，则它们按照`aug_string`中字符的顺序从`aug_operands`字段中读取。已识别的标志包括：

| 字符串 | 操作数 | 备注 |
| --- | --- | --- |
| `"eh"` | `void* eh_ptr` | 操作数不是`aug_operands`的一部分 |
| `"z"` | `uleb128 长度` | `aug_operands`后续字节的长度（以字节为单位） |
| `"R"` | `uint8_t fde_ptr_encoding` | [预示](#fde) |
| `"P"` | `uint8_t ptr_encoding` `uint8_t personality_ptr[]` | [预示](#lsda) |
| `"L"` | `uint8_t lsda_ptr_encoding` | [预示](#lsda) |
| `"S"` | 无 | 信号处理程序帧 |
| `"B"` | 无 | aarch64 ptrauth使用B键 |
| `"\0"` | 无 | `aug_string`的结束 |

`"eh"`如果出现，必须放在第一位。实际上，除了由非常旧版本的g++编译的代码，它从未出现过。如果出现`"z"`，必须紧跟其后，而且实际上总是存在。剩余的字符（除了NUL）可以以任何顺序出现，尽管如果`"R"`存在，并且`"S"`和/或`"B"`也存在，则`"R"`必须在`"S"`和`"B"`之前出现（以解决[`get_cie_encoding`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L335-L392)中的错误，与正确的[`extract_cie_info`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2.c#L412-L517)相比）。

整个`struct cie`的长度必须是`sizeof(uintptr_t)`字节的倍数。末尾的`zero_padding`字段确保了这一点。方便的是，`DW_CFA_nop`为零，因此`zero_padding`可以看作是`dw_cfa_bytecode`字段的扩展。整个`struct cie`的长度为`4 + length`。要获得合并的`dw_cfa_bytecode` / `zero_padding`的长度，需要减去所有其他字段的长度。对于大多数字段来说，这很直接（尽管有点麻烦），唯一的难点是`aug_operands`：它的长度取决于`aug_string`的内容，并且`aug_string`中可能有我们无法识别的字符。这就是`"z"`的用处：一旦我们解码了它的操作数，它就告诉我们`aug_operands`的剩余长度。

**从一个位置到多个位置**

获取适用于一个指令指针值的`UnwindActions`是可以的，但这需要扩展到整个程序中的每一个(*)可能的指令指针值。一项相关的观察是，适用于指令指针X的`UnwindActions`*通常*非常相似（甚至相同）于适用于指令指针值X+1的那些。

(*) 假设使用`-fasynchronous-unwind-tables`。如果仅使用`-fnon-call-exceptions`，所需覆盖范围较少，如果两者都不使用，所需覆盖范围更少。请注意，某些架构默认启用`-fasynchronous-unwind-tables`。

鉴于这一观察，我们可以借用视频压缩的一个技巧：使用关键帧描述每个函数开头的`UnwindActions`，然后使用增量帧描述指令指针在函数中逐步推进的差异。

这激励了一系列新的虚拟机操作码，以及两个虚拟机状态：将`target`设置为你想要`UnwindActions`的指令指针，将`fpc`设置为函数开始的指令指针，并执行虚拟机指令，直到虚拟机指令用尽，或直到`fpc >= target`：

| 操作码 | 名称及操作数 | 语义 |
| --- | --- | --- |
| `0x40 + Off` | DW_CFA_advance_loc | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x02` | DW_CFA_advance_loc1(uint8_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x03` | DW_CFA_advance_loc2(uint16_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x04` | DW_CFA_advance_loc4(uint32_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x01` | DW_CFA_set_loc(uint8_t Ptr[]) | `fpc = DecodePtr(fde_encoding from CIE, Ptr); if (fpc >= target) break;` |

它还激励了两个虚拟机状态，分别为`spReg`和`spOff`，以及对DW_CFA_def_cfa / DW_CFA_def_cfa_sf的修订和执行只占DW_CFA_def_cfa / DW_CFA_def_cfa_sf的一半的新操作码：

| 操作码 | 名称及操作数 | 假设`UnwindActions* ua`的语义 |
| --- | --- | --- |
| `0x0c` | DW_CFA_def_cfa(uleb128 Reg, uleb128 Off) | `ua->sp = RegOffset(spReg = Reg, spOff = Off)` |
| `0x12` | DW_CFA_def_cfa_sf(uleb128 Reg, sleb128 Off) | `ua->sp = RegOffset(spReg = Reg, spOff = Off * data_align)` |
| `0x0d` | DW_CFA_def_cfa_register(uleb128 Reg) | `spReg = Reg; ua->sp = RegOffset(spReg, spOff)` |
| `0x0e` | DW_CFA_def_cfa_offset(uleb128 Off) | `spOff = Off; if (ua->sp is RegOffset) ua->sp = RegOffset(spReg, spOff)` |
| `0x13` | DW_CFA_def_cfa_offset_sf(sleb128 Off) | `spOff = Off * data_align; if (ua->sp is RegOffset) ua->sp = RegOffset(spReg, spOff)` |

最后，这激励我们在VM中添加 `Stack<UnwindActions>`，以及一些操作码来操作它：

| 操作码 | 名称和操作数 | 假设 `UnwindActions* ua` 的语义 |
| --- | --- | --- |
| `0x0a` | DW_CFA_remember_state | `Push(*ua)` (请注意，`ua` 没有被修改) |
| `0x0b` | DW_CFA_restore_state | `*ua = Pop()` |

**从一个函数到多个函数**

我们可以很高兴地为每个函数拥有一个CIE，但是在CIE之间会有相当多的重复。为了解决这个问题，我们引入了FDE的概念，它本质上是一个精简版的CIE：

```
struct fde {
  uint32_t length;            
  int32_t  cie_offset;        
  uint8_t  func_start[];      
  uint8_t  func_length[];     
  uint8_t  aug_operands[];    
  uint8_t  dw_cfa_bytecode[]; 
  uint8_t  zero_padding[];    
}; 
```

`cie_offset` 字段指向一个CIE，并且FDE从指向的CIE继承所有内容。对于 `aug_string` 的继承，其中的每个标志有时在CIE的 `aug_operands` 中有操作数，有时在FDE的 `aug_operands` 中有操作数，有时两者都有。FDE 的 `aug_operands` 包括：

| 字符串和操作数 |
| --- |
| `"eh"` | None |  |
| `"z"` | `uleb128 length` | 字节，后续 `aug_operands` 的长度 |
| `"R"` | None |  |
| `"P"` | None |  |
| `"L"` | `uint8_t lsda_ptr[]` | 使用来自CIE的 `lsda_ptr_encoding` 进行编码 |
| `"S"` | None |  |
| `"B"` | None |  |
| `"\0"` | None | `aug_string` 的结尾 |

`dw_cfa_bytecode` 字段通过连接方式继承：执行FDE的字节码首先需要执行关联CIE的字节码。所有VM状态从CIE字节码结束到FDE字节码开始都被保留，尽管使用 DW_CFA_remember_state / DW_CFA_restore_state 时，堆栈在从CIE切换到FDE时会被清空。

那留给 `func_start`，它是一个变长指针，和 `func_length`，它是一个变长整数。如果 `aug_string` 不包含 `"R"`，那么这些字段分别是 `void*` 和 `uintptr_t`。如果 `aug_string` 包含 `"R"`，那么 `"R"` 的操作数 (`fde_ptr_encoding`) 描述了 `func_start` 的大小和解释，同时 `fde_ptr_encoding & 0xF` 描述了 `func_length` 的大小和解释。这是讨论指针编码的过渡。

**指针编码**

根据上下文，以不同方式编码指针可能更合适。例如，有时希望使用 `uintptr_t`，而其他时候希望使用从当前函数开始的 `int32_t` 偏移量。为了允许灵活性，各个位置允许使用 `uint8_t` 描述正在使用的指针编码方式。这些位置包括：

| 编码字段 | 相关的指针字段 |
| --- | --- |
| `cie::fde_ptr_encoding` | `fde::func_start` |
| `cie::fde_ptr_encoding & 0xF` | `fde::func_length` |
| `cie::fde_ptr_encoding` | `DW_CFA_set_loc` 操作数 |
| `cie::aug_string` `"P"` `ptr_encoding` | `cie::aug_string` `"P"` `personality_ptr` |
| `cie::aug_string` `"L"` | `fde::aug_string` `"L"`（LSDA指针） |
| `eh_frame_hdr::eh_frame_ptr_enc` | `eh_frame_hdr::eh_frame_ptr` † |
| `eh_frame_hdr::fde_count_enc` | `eh_frame_hdr::fde_count` † |
| `eh_frame_hdr::table_enc` | `eh_frame_hdr::sorted_table` † |
| DW_OP_GNU_encoded_addr | DW_OP_GNU_encoded_addr ‡ |

† [预示](#eh_frame_hdr)。

‡ [更多预示](#dw_op_)。

有两个`uint8_t`的特殊情况：

| 编码 | 含义 |
| --- | --- |
| `0xff` | 编码值缺失/空，解码为NULL。 |
| `0x50` | 编码一个`uintptr_t`，但是在前面加上填充以对齐。 |

排除了两个特殊情况后，剩下的情况将字节分割为四位字段、三位字段和一位字段。低四位表示数据类型：

| 编码 & 0xF | 数据类型 | 字节大小 |
| --- | --- | --- |
| `0x0` | `uintptr_t` | 通常为4或8 |
| `0x1` ‡ | `uleb128` | 变化（≥ 1） |
| `0x2` | `uint16_t` | 2 |
| `0x3` | `uint32_t` | 4 |
| `0x4` | `uint64_t` | 8 |
| `0x9` ‡ | `sleb128` | 变化（≥ 1） |
| `0xA` | `int16_t` | 2 |
| `0xB` | `int32_t` | 4 |
| `0xC` | `int64_t` | 8 |

‡ 不能用于`fde_ptr_encoding`。

接下来的三位数表示要添加到非NULL值的基础值：

| 编码 & 0x70 | 基础值 |
| --- | --- |
| `0x00` | NULL |
| `0x10`（pcrel） | 编码指针的第一个字节地址 |
| `0x20`（textrel） | .text的起始位置（理论上），但在实践中通常为NULL |
| `0x30`（datarel） | .got的起始位置（x86），或NULL（大多数其他架构）† |
| `0x40`（funcrel） ‡ | `fde::func_start`（解码后） |

† 除了在`eh_frame_hdr::table_enc`中，它指的是.eh_frame_hdr的起始位置。

‡ 不能用于`fde_ptr_encoding`或`eh_frame_hdr`。

最后的最高位控制可选的解引用：

| 编码 & 0x80 | 语义 |
| --- | --- |
| `0x00` | 将值视为`void*` |
| `0x80` | 将值视为`void**`，解引用以获取`void*` |

如果需要整数而不是指针（比如`func_length`和`fde_count_enc`），那么`void*`会被重新解释为`uintptr_t`。

**寻找所有的FDEs**

各种CIE和FDE结构被串联在一起，然后放置在ELF的`.eh_frame`段中。

传统上，在启动期间编译器会插入一个调用[`__register_frame`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L151-L162)，传递`.eh_frame`节的地址。如果需要非NULL值用于textrel / datarel指针编码，则会插入一个调用[`__register_frame_info_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L108-L143)。在关闭期间，会进行匹配的调用[`__deregister_frame`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L301-L307)或[`__deregister_frame_info_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L224-L293)。如果存在多个`.eh_frame`节且链接器未将它们合并，则可以改用[`__register_frame_table`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L205-L210) / [`__register_frame_info_table_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L168-L197)，它们接受指向NULL结尾的`.eh_frame`节指针列表的指针。

更现代的方法是添加一个ELF`.eh_frame_hdr`节，并使用[`_dl_find_object`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde-dip.c#L551)或[`dl_iterate_phdr`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde-dip.c#L570)来查找适当的`.eh_frame_hdr`节。一旦找到，该节包含一个按FDE指针排序的表：

```
struct eh_frame_hdr {
  uint8_t version;          
  uint8_t eh_frame_ptr_enc; 
  uint8_t fde_count_enc;    
  uint8_t table_enc;        
  uint8_t eh_frame_ptr[];   
  uint8_t fde_count[];      
  struct {
    int32_t func_start;     
    int32_t fde_offset;     
  } sorted_table[fde_count];
}; 
```

`eh_frame_ptr`和`fde_count`字段的组合大小必须是四的倍数（如果`eh_frame_ptr_enc`和`fde_count_enc`都使用4或8字节类型，则自然发生）。`eh_frame_ptr`字段包含指向`.eh_frame`节的指针，如果某些原因无法解析`.eh_frame_hdr`节，则用作后备。`table_enc`字段包含`sorted_table`内容使用的编码，虽然datarel表示相对于`.eh_frame_hdr`的起始位置，而且唯一支持的编码是datarel int32_t。表按升序的`func_start`顺序排序，并且其中的`func_start`值会覆盖所引用FDE的`func_start`（尽管实际上它们应该是相同的）。

**人物指针和LSDA指针**

为了支持在展开期间调用析构函数，并支持捕获异常（从而停止展开），CIE可以指定指向人格函数的指针。该函数包含析构和/或捕获逻辑，并将作为展开的一部分调用（除非仅生成回溯）。我不会详述接口的具体细节，不过人格函数可以查询展开状态的各个部分：

+   `_Unwind_GetLanguageSpecificData`从FDE返回`lsda_ptr`。

+   `_Unwind_GetRegionStart`从FDE返回`func_start`。

旧版本的g++使用`eh_ptr`而不是人格函数和LSDA指针。除非您正在实现[`__frame_state_for`](https://github.com/gcc-mirror/gcc/blob/39d989022dd0eacf1a7b95b7b20621acbe717d70/libgcc/unwind-dw2.c#L1072-L1120)，否则您无需担心`eh_ptr`。

**表达式字节码**

之前的`UnwindOneFrame`函数包含对`EvalExpr`的调用，用于处理复杂情况。`EvalExpr`的概述如下：

```
uintptr_t EvalExpr(uint8_t DwOpBytecode[unsigned Len],
                   uintptr_t StackInitial,
                   RegState* original) {
  const uint8_t* bpc = DwOpBytecode;
  Stack<uintptr_t> stk;
  stk.Push(StackInitial);
  while (bpc < DwOpBytecode + Len) {
    uint8_t opcode = *bpc++;

    ...

    ...
  }
  return stk.Pop();
} 
```

以下是支持的各种操作码：

| 操作码 | 名称和操作数 | 语义（假设用于`Push`、`Pop`、`At`的堆栈） |
| --- | --- | --- |
| `0x03` | DW_OP_addr(uintptr_t Lit) | `Push(Lit)`（推送Lit） |
| `0x08` | DW_OP_const1u(uint8_t Lit) | `Push(Lit)`（推送Lit） |
| `0x09` | DW_OP_const1s(int8_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0A` | DW_OP_const2u(uint16_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0B` | DW_OP_const2s(int16_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0C` | DW_OP_const4u(uint32_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0D` | DW_OP_const4s(int32_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0E` | DW_OP_const8u(uint64_t Lit) | `Push(Lit)`（推送Lit） |
| `0x0F` | DW_OP_const8s(int64_t Lit) | `Push(Lit)`（推送Lit） |
| `0x10` | DW_OP_constu(uleb128 Lit) | `Push(Lit)`（推送Lit） |
| `0x11` | DW_OP_consts(sleb128 Lit) | `Push(Lit)`（推送Lit） |
| `0x12` | DW_OP_dup | `Push(At(-1))`（复制顶部元素） |
| `0x14` | DW_OP_over | `Push(At(-2))`（复制倒数第二个元素） |
| `0x15` | DW_OP_pick(uint8_t Idx) | `Push(At(-1-Idx))`（推送At(-1-Idx)） |
| `0x13` | DW_OP_drop | `Pop()`（弹出） |
| `0x16` | DW_OP_swap | `a = Pop(); b = Pop(); Push(a); Push(b)`（a = 弹出(); b = 弹出(); 推送(a); 推送(b)） |
| `0x17` | DW_OP_rot | `a = Pop(); b = Pop(); c = Pop(); Push(a); Push(c); Push(b)`（a = 弹出(); b = 弹出(); c = 弹出(); 推送(a); 推送(c); 推送(b)） |
| `0x06` | DW_OP_deref | `Push(*(uintptr_t*)Pop())`（推送*(uintptr_t*)Pop()） |
| `0x94` `0x01` | DW_OP_deref_size | `Push(*(uint8_t*)Pop())`（推送*(uint8_t*)Pop()） |
| `0x94` `0x02` | DW_OP_deref_size | `Push(*(uint16_t*)Pop())`（推送*(uint16_t*)Pop()） |
| `0x94` `0x04` | DW_OP_deref_size | `Push(*(uint32_t*)Pop())`（推送*(uint32_t*)Pop()） |
| `0x94` `0x08` | DW_OP_deref_size | `Push(*(uint64_t*)Pop())`（推送*(uint64_t*)Pop()） |
| `0x19` | DW_OP_abs | `a = Pop(); Push((intptr_t)a < 0 ? -a : a)`（a = 弹出(); 推送((intptr_t)a < 0 ? -a : a)） |
| `0x1f` | DW_OP_neg | `a = Pop(); Push(-a)`（a = 弹出(); 推送(-a)） |
| `0x20` | DW_OP_not | `a = Pop(); Push(~a)`（a = 弹出(); 推送(~a)） |
| `0x23` | DW_OP_plus_uconst(uleb128 Lit) | `a = Pop(); Push(a + Lit)`（a = 弹出(); 推送(a + Lit)） |
| `0x1a` | DW_OP_and | `rhs = Pop(); lhs = Pop(); Push(lhs & rhs)`（rhs = 弹出(); lhs = 弹出(); 推送(lhs & rhs)） |
| `0x1b` | DW_OP_div | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs / (intptr_t)rhs)`（rhs = 弹出(); lhs = 弹出(); 推送((intptr_t)lhs / (intptr_t)rhs)） |
| `0x1c` | DW_OP_minus | `rhs = Pop(); lhs = Pop(); Push(lhs - rhs)`（rhs = 弹出(); lhs = 弹出(); 推送(lhs - rhs)） |
| `0x1d` | DW_OP_mod | `rhs = Pop(); lhs = Pop(); Push(lhs % rhs)`（rhs = 弹出(); lhs = 弹出(); 推送(lhs % rhs)） |
| `0x1e` | DW_OP_mul | `rhs = Pop(); lhs = Pop(); Push(lhs * rhs)` |
| `0x21` | DW_OP_or | `rhs = Pop(); lhs = Pop(); Push(lhs &#124; rhs)` |
| `0x22` | DW_OP_plus | `rhs = Pop(); lhs = Pop(); Push(lhs + rhs)` |
| `0x24` | DW_OP_shl | `rhs = Pop(); lhs = Pop(); Push(lhs << rhs)` |
| `0x25` | DW_OP_shr | `rhs = Pop(); lhs = Pop(); Push(lhs >> rhs)` |
| `0x26` | DW_OP_shra | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs >> rhs)` |
| `0x27` | DW_OP_xor | `rhs = Pop(); lhs = Pop(); Push(lhs ^ rhs)` |
| `0x29` | DW_OP_eq | `rhs = Pop(); lhs = Pop(); Push(lhs == rhs)` |
| `0x2e` | DW_OP_ne | `rhs = Pop(); lhs = Pop(); Push(lhs != rhs)` |
| `0x2a` | DW_OP_ge | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs >= (intptr_t)rhs)` |
| `0x2b` | DW_OP_gt | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs > (intptr_t)rhs)` |
| `0x2c` | DW_OP_le | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs <= (intptr_t)rhs)` |
| `0x2d` | DW_OP_lt | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs < (intptr_t)rhs)` |
| `0x2f` | DW_OP_skip(int16_t Delta) | `bpc += Delta` |
| `0x28` | DW_OP_bra(int16_t Delta) | `if (Pop() != 0) bpc += Delta` |
| `0x30 + Lit` | DW_OP_lit | `Push(Lit)` |
| `0x50 + Reg` | DW_OP_reg | `Push(original->value[Reg])` |
| `0x70 + Reg` | DW_OP_breg(sleb128 Lit) | `Push(original->value[Reg] + Lit)` |
| `0x90` | DW_OP_regx(uleb128 Reg) | `Push(original->value[Reg])` |
| `0x92` | DW_OP_bregx(uleb128 Reg, sleb128 Lit) | `Push(original->value[Reg] + Lit)` |
| `0x96` | DW_OP_nop | No-op |
| `0xf1` | DW_OP_GNU_encoded_addr(uint8_t PtrEncoding, uint8_t Ptr[]) | `Push(DecodePtr(PtrEncoding, Ptr))` |

`EvalExpr`的gcc实现假设堆栈永远不会超过64个元素。如果堆栈超出此大小，将会发生严重问题。

目前这种表达语言最常见的用法是允许单个FDE简洁地描述任意数量的PLT条目。在这种情况下的序列化字节码可能如下所示：

```
0x92 0x07 0x08  DW_OP_bregx(7 /* rsp */, 8)
0x90 0x10       DW_OP_regx(16 /* rip */)
0x08 0x0f       DW_OP_const1u(15)
0x1a            DW_OP_and
0x08 0x0b       DW_OP_const1u(11)
0x2a            DW_OP_ge
0x08 0x03       DW_OP_const1u(3)
0x24            DW_OP_shl
0x22            DW_OP_plus 
```

用以编码表达式：

```
rsp + 8 + ((((rip & 15) >= 11) ? 1 : 0) << 3) 
```

换句话说，每个16字节的PLT条目分为两部分：第1部分长度为11字节，第2部分长度为5字节。如果在第1部分中，表达式为`rsp + 8`，而在第2部分中为`rsp + 16`。

**参考文献**

[DWARF标准](https://dwarfstd.org/doc/DWARF5.pdf)有详细记录，尽管`.eh_frame`在几个方面与其有所偏差。此外，还需要特定于体系结构的扩展（例如[x86](https://www.uclibc.org/docs/psABI-i386.pdf)，[x86_64](http://refspecs.linux-foundation.org/elf/x86_64-abi-0.99.pdf)，[aarch64](https://github.com/ARM-software/abi-aa/blob/main/aadwarf64/aadwarf64.rst)）。唯一真正的参考是gcc源代码，其一些相关入口点包括：

**参见**

对于同一个问题的完全不同解决方法，请考虑在 Windows 上的[ARM64解包](/content/windows-arm64-unwind-codes)。其中的`.xdata`记录大致相当于FDE，并且它具有一个与DW_CFA_ bytecode在目的上类似（但风格非常不同）的字节码解释器。

LuaJIT 中有一些完整的`.eh_frame`部分的示例：
