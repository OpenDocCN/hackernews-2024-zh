<!--yml
category: 未分类
date: 2024-05-27 14:55:55
-->

# Linux/ELF .eh_frame from the bottom up

> 来源：[https://www.corsix.org/content/elf-eh-frame](https://www.corsix.org/content/elf-eh-frame)

**Problem statement**

Given the current register state for a thread, and read-only access to memory, what would the register state hypothetically become if the current function was to immediately return and execution was to resume in its caller?

Notably, the register state includes (amongst other things) an instruction pointer and a stack pointer, so this operation can be iterated to generate a backtrace. With some extensions to support calling destructors and catching exceptions, this operation can also be the basis of throwing exceptions.

This operation is typically called *unwinding* (as in "unwinding a call frame" or "unwinding the stack").

If certain registers are always undefined immediately after a function return (because they are volatile in the ABI), then we needn't care about their state.

**Portability**

The size and contents of the register state depends on the architecture being used. We'll paper over this slightly by refering to registers by ordinal rather than by name, and rely on DWARF register number mappings to go between ordinal and name, for example:

| Ordinal | x86 name | x86_64 name | aarch64 name |
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
| 30 |  |  | X30 (LR) |
| 31 |  |  | SP |

Because everyone loves special cases, the instruction pointer register (e.g. eip / rip / PC) isn't given an ordinal. On aarch64, it doesn't need one: the return address is always in *some* register (usually X30, aka. LR), and so we can use the return address register to represent the instruction pointer. In contrast, x86 and x86_64 put the return address on the stack rather than in a register. For these, we'll invent a fake register (typically ordinal 8 on x86, ordinal 16 on x86_64), *pretend* that it contains the return address, and use it to represent the instruction pointer.

Floating-point and vector registers are generally volatile in Linux ABIs, so we needn't worry about them too much.

**Start small: unwinding just the stack pointer**

In most cases, the hypothetical stack pointer after function return can be easily calculated by adding some value to some register. This can be expressed as two DWARF-style [ULEB128](https://en.wikipedia.org/wiki/LEB128) values: one ULEB128 to specify which register to start with (often the current stack pointer), then one ULEB128 to specify the amount to add. As foreshadowing, we'll call this `DW_CFA_def_cfa`.

**Unwinding other registers**

In most cases, if a function modifies a non-volatile register, it'll save it to somewhere on the stack before modifying it. Upon function return, it'll re-load the value from said stack slot. The locations of these stack slots can be expressed as being relative to the hypothetical stack pointer after function return. This can again be expressed as two DWARF-style ULEB128 values: one ULEB to specify which register we're talking about, then one ULEB to specify the offset of its stack slot. There's only one problem: because the offset is against the hypothetical stack pointer *after* function return, the required offset is going to be negative, which a ULEB can't represent. To get around this, we'll scale the 2^(nd) ULEB by a so-called `data_align` value (typically -4 or -8), which also has the convenient side-effect of making the ULEB smaller. As foreshadowing, we'll call this `DW_CFA_offset_extended`. When the 1^(st) ULEB is less than 64, we might also call this `DW_CFA_offset`.

**More ways to unwind other registers**

So far we've described forming the address of a stack slot, and then loading from it. There's a natural variation that skips the 2^(nd) part: form the address of a stack slot, then set the value of a register to be that address.

There's another common variant: unwind register X by taking the value from register Y (when X equals Y, this means that said register was unchanged).

For when these common variants aren't sufficient, we'll [later describe](#dw_op_) a little expression language and associated bytecode VM capable of describing all sorts of complicated mechanisms.

This leads to the following definitions to describe how to unwind registers:

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

With the associated unwind logic being:

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

A little bytecode VM is defined to populate an instance of `UnwindActions`:

| Opcode | Name and operands | Semantics assuming `UnwindActions* ua` |
| `0x0c` | DW_CFA_def_cfa(uleb128 Reg, uleb128 Off) | `ua->sp = RegOffset(Reg, Off)` |
| `0x12` | DW_CFA_def_cfa_sf(uleb128 Reg, sleb128 Off) | `ua->sp = RegOffset(Reg, Off * data_align)` |
| `0x0f` | DW_CFA_def_cfa_expression(uleb128 Len, uint8_t DwOpBytecode[Len]) | `ua->sp = ExprResult(DwOpBytecode)` |
| `0x80 + Reg` | DW_CFA_offset(uleb128 Off) | `ua->reg[Reg] = LoadFromStackSlot(Off * data_align)` |
| `0x05` | DW_CFA_offset_extended(uleb128 Reg, uleb128 Off) | `ua->reg[Reg] = LoadFromStackSlot(Off * data_align)` |
| `0x11` | DW_CFA_offset_extended_sf(uleb128 Reg, sleb128 Off) | `ua->reg[Reg] = LoadFromStackSlot(Off * data_align)` |
| `0x2f` | DW_CFA_GNU_negative_offset_extended(uleb128 Reg, uleb128 Off) | `ua->reg[Reg] = LoadFromStackSlot(-(Off * data_align))` |
| `0x14` | DW_CFA_val_offset(uleb128 Reg, uleb128 Off) | `ua->reg[Reg] = AddressOfStackSlot(Off * data_align)` |
| `0x15` | DW_CFA_val_offset_sf(uleb128 Reg, sleb128 Off) | `ua->reg[Reg] = AddressOfStackSlot(Off * data_align)` |
| `0xc0 + Reg` | DW_CFA_restore | `ua->reg[Reg] = CopyFromRegister(Reg)` † |
| `0x06` | DW_CFA_restore_extended(uleb128 Reg) | `ua->reg[Reg] = CopyFromRegister(Reg)` † |
| `0x08` | DW_CFA_same_value(uleb128 Reg) | `ua->reg[Reg] = CopyFromRegister(Reg)` |
| `0x09` | DW_CFA_register(uleb128 Reg, uleb128 Src) | `ua->reg[Reg] = CopyFromRegister(Src)` |
| `0x07` | DW_CFA_undefined(uleb128 Reg) | `ua->reg[Reg] = Undefined` |
| `0x10` | DW_CFA_expression(uleb128 Reg, uleb128 Len, uint8_t DwOpBytecode[Len]) | `ua->reg[Reg] = LoadFromExprResult(DwOpBytecode)` |
| `0x16` | DW_CFA_val_expression(uleb128 Reg, uleb128 Len, uint8_t DwOpBytecode[Len]) | `ua->reg[Reg] = ExprResult(DwOpBytecode)` |
| `0x00` | DW_CFA_nop | No-op |

† This differs from the usual DWARF semantics. To avoid confusion, use DW_CFA_same_value instead of DW_CFA_restore.

If the bytecode does not initialise `sp`, its value is `Undefined`. If the bytecode does not initialise `reg[i]`, its value is `CopyFromRegister(i)`.

**Putting it together in a CIE**

The aforementioned bytecode gets wrapped up in something called a CIE, which also includes a bunch of other fields:

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

The `aug_string` field is a NUL-terminated ASCII string. Conceptually it is a bitmask of flags, except that each flag is represented by one or two characters rather than by a single bit. If the flag has associated operand(s), they are read out of the `aug_operands` field, in the same order as characters appear in `aug_string`. The recognised flags are:

| Char(s) | Operand(s) | Notes |
| `"eh"` | `void* eh_ptr` | Operand not part of `aug_operands` |
| `"z"` | `uleb128 length` | In bytes, of subsequent `aug_operands` |
| `"R"` | `uint8_t fde_ptr_encoding` | [Foreshadowing](#fde) |
| `"P"` | `uint8_t ptr_encoding` `uint8_t personality_ptr[]` | [Foreshadowing](#lsda) |
| `"L"` | `uint8_t lsda_ptr_encoding` | [Foreshadowing](#lsda) |
| `"S"` | None | Signal handler frame |
| `"B"` | None | aarch64 ptrauth uses B key |
| `"\0"` | None | End of `aug_string` |

`"eh"`, if it appears, must be first. In practice, it is never present, except in code compiled by very old versions of g++. `"z"`, if it appears, must be next, and in practice is always present. The remaining characters (except NUL) can appear in any order, though if `"R"` is present, and `"S"` and/or `"B"` are also present, then `"R"` must appear before `"S"` and before `"B"` (to work around bugs in [`get_cie_encoding`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L335-L392), as compared to the correct [`extract_cie_info`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2.c#L412-L517)).

The whole `struct cie` must have a length which is a multiple of `sizeof(uintptr_t)` bytes. The `zero_padding` field at the end ensures this. Helpfully, `DW_CFA_nop` is zero, so `zero_padding` can be seen as an extension of the `dw_cfa_bytecode` field. The length of the whole `struct cie` is `4 + length`. To get the length of the combined `dw_cfa_bytecode` / `zero_padding`, the lengths of all the other fields need to be subtracted off. This is straightforward (albeit fiddly) for most fields, with the only hard case being `aug_operands`: its length depends on the contents of `aug_string`, and there may be characters in `aug_string` which we do not recognise. This is where `"z"` comes in: once we've decoded its operand, it tells us the remaining length of `aug_operands`.

**From one location to many**

Obtaining the `UnwindActions` applicable to one instruction pointer value is all well and good, but this needs to scale to every (*) possible instruction pointer value across an entire program. One relevant observation is that the `UnwindActions` applicable to an instruction pointer X are *usually* very similar (or even identical to) those applicable to an instruction pointer value of X+1.

(*) Assuming `-fasynchronous-unwind-tables`. Less coverage is required if only `-fnon-call-exceptions` is used, and less still is required if neither is used. Note that `-fasynchronous-unwind-tables` is enabled by default on some architectures.

With this observation in mind, we can borrow a trick from video compression: use a keyframe to describe the `UnwindActions` applicable at the *start* of every function, and then use delta frames to describe the differences as the instruction pointer progresses through the function.

This motivates a bunch of new VM opcodes, along with two pieces of VM state: set `target` to the instruction pointer that you want `UnwindActions` for, set `fpc` to the instruction pointer of the start of the function, and execute VM instructions either until you run out of VM instructions, or until `fpc >= target`:

| Opcode | Name and operands | Semantics |
| `0x40 + Off` | DW_CFA_advance_loc | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x02` | DW_CFA_advance_loc1(uint8_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x03` | DW_CFA_advance_loc2(uint16_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x04` | DW_CFA_advance_loc4(uint32_t Off) | `fpc += Off * code_align; if (fpc >= target) break;` |
| `0x01` | DW_CFA_set_loc(uint8_t Ptr[]) | `fpc = DecodePtr(fde_encoding from CIE, Ptr); if (fpc >= target) break;` |

It also motivates two pieces of VM state called `spReg` and `spOff`, along with revisements to DW_CFA_def_cfa / DW_CFA_def_cfa_sf, and new opcodes which perform only one half of DW_CFA_def_cfa / DW_CFA_def_cfa_sf:

| Opcode | Name and operands | Semantics assuming `UnwindActions* ua` |
| `0x0c` | DW_CFA_def_cfa(uleb128 Reg, uleb128 Off) | `ua->sp = RegOffset(spReg = Reg, spOff = Off)` |
| `0x12` | DW_CFA_def_cfa_sf(uleb128 Reg, sleb128 Off) | `ua->sp = RegOffset(spReg = Reg, spOff = Off * data_align)` |
| `0x0d` | DW_CFA_def_cfa_register(uleb128 Reg) | `spReg = Reg; ua->sp = RegOffset(spReg, spOff)` |
| `0x0e` | DW_CFA_def_cfa_offset(uleb128 Off) | `spOff = Off; if (ua->sp is RegOffset) ua->sp = RegOffset(spReg, spOff)` |
| `0x13` | DW_CFA_def_cfa_offset_sf(sleb128 Off) | `spOff = Off * data_align; if (ua->sp is RegOffset) ua->sp = RegOffset(spReg, spOff)` |

Finally, it motivates adding a `Stack<UnwindActions>` to the VM, along with some opcodes to manipulate it:

| Opcode | Name and operands | Semantics assuming `UnwindActions* ua` |
| `0x0a` | DW_CFA_remember_state | `Push(*ua)` (note that `ua` not modified) |
| `0x0b` | DW_CFA_restore_state | `*ua = Pop()` |

**From one function to many**

We could happily have one CIE per function, but there would be quite a lot of repetition between CIEs. To resolve this, we introduce the concept of an FDE, which is essentially a stripped down CIE:

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

The `cie_offset` field points to a CIE, and the FDE inherits everything from the pointed-to CIE. For inheritance of `aug_string`, each flag therein sometimes has operand(s) in the CIE's `aug_operands`, sometimes in the FDE's `aug_operands`, and sometimes in both. The FDE `aug_operands` has:

| Char(s) | Operand(s) | Notes |
| `"eh"` | None |  |
| `"z"` | `uleb128 length` | In bytes, of subsequent `aug_operands` |
| `"R"` | None |  |
| `"P"` | None |  |
| `"L"` | `uint8_t lsda_ptr[]` | Encoded using `lsda_ptr_encoding` from CIE |
| `"S"` | None |  |
| `"B"` | None |  |
| `"\0"` | None | End of `aug_string` |

The `dw_cfa_bytecode` field is inherited by concatenation: executing the bytecode for an FDE involves first executing the bytecode of the associated CIE. All VM state is carried over from the end of the CIE bytecode to the start of the FDE bytecode, thoough if using DW_CFA_remember_state / DW_CFA_restore_state then the stack is emptied as part of switching from the CIE to the FDE.

That leaves `func_start`, which is a variable length pointer, and `func_length`, which is a variable length integer. If `aug_string` does *not* contain `"R"`, then these fields are `void*` and `uintptr_t` respectively. If `aug_string` *does* contain `"R"`, then the operand of `"R"` (`fde_ptr_encoding`) describes the size and interpretation of `func_start`, meanwhile `fde_ptr_encoding & 0xF` describes the size and interpretation of `func_length`. That's the segue to talking about pointer encodings.

**Pointer encodings**

Depending on the context, it can be desirable to encode a pointer in different ways. For example, sometimes a `uintptr_t` is desirable, whereas other times an `int32_t` offset from the start of the current function is desirable. To allow flexibility, various places allow a `uint8_t` to describe the pointer encoding being used. These places are:

| Encoding field | Associated pointer field |
| `cie::fde_ptr_encoding` | `fde::func_start` |
| `cie::fde_ptr_encoding & 0xF` | `fde::func_length` |
| `cie::fde_ptr_encoding` | `DW_CFA_set_loc` operand |
| `cie::aug_string` `"P"` `ptr_encoding` | `cie::aug_string` `"P"` `personality_ptr` |
| `cie::aug_string` `"L"` | `fde::aug_string` `"L"` (LSDA pointer) |
| `eh_frame_hdr::eh_frame_ptr_enc` | `eh_frame_hdr::eh_frame_ptr` † |
| `eh_frame_hdr::fde_count_enc` | `eh_frame_hdr::fde_count` † |
| `eh_frame_hdr::table_enc` | `eh_frame_hdr::sorted_table` † |
| DW_OP_GNU_encoded_addr | DW_OP_GNU_encoded_addr ‡ |

† [Foreshadowing](#eh_frame_hdr).
‡ [More foreshadowing](#dw_op_).

There are two special cases for this `uint8_t`:

| Encoding | Meaning |
| `0xff` | Encoded value is absent/empty, decode as NULL. |
| `0x50` | Encode a `uintptr_t`, but precede with padding to align it. |

Once the two special cases are excluded, the remaining cases split the byte into a four bit field, a three bit field, and a one bit field. The low four bits denote the data type:

| Encoding & 0xF | Data type | Size in bytes |
| `0x0` | `uintptr_t` | Typically either 4 or 8 |
| `0x1` ‡ | `uleb128` | Varies (≥ 1) |
| `0x2` | `uint16_t` | 2 |
| `0x3` | `uint32_t` | 4 |
| `0x4` | `uint64_t` | 8 |
| `0x9` ‡ | `sleb128` | Varies (≥ 1) |
| `0xA` | `int16_t` | 2 |
| `0xB` | `int32_t` | 4 |
| `0xC` | `int64_t` | 8 |

‡ Cannot be used for `fde_ptr_encoding`.

The next three bits denote a base value to be added to non-NULL values:

| Encoding & 0x70 | Base value |
| `0x00` | NULL |
| `0x10` (pcrel) | Address of first byte of encoded pointer |
| `0x20` (textrel) | Start of .text (in theory), but usually NULL in practice |
| `0x30` (datarel) | Start of .got (x86) or NULL (most other architectures) † |
| `0x40` (funcrel) ‡ | `fde::func_start` (after decoding) |

† Except in `eh_frame_hdr::table_enc`, where it means start of .eh_frame_hdr.
‡ Cannot be used for `fde_ptr_encoding`, or in `eh_frame_hdr`.

The final top bit controls an optional dereference:

| Encoding & 0x80 | Semantics |
| `0x00` | Treat value as `void*` |
| `0x80` | Treat value as `void**`, dereference it to get `void*` |

If an integer is desired rather than a pointer (as is the case for `func_length` and `fde_count_enc`), then the `void*` is reinterpreted as `uintptr_t`.

**Finding all the FDEs**

The various CIE and FDE structures are concatenated together, and then placed in the ELF `.eh_frame` section.

Traditionally, the compiler would insert a call to [`__register_frame`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L151-L162) somewhere during startup, passing along the address of the `.eh_frame` section. If non-NULL values were desired for textrel / datarel pointer encodings, it would instead insert a call to [`__register_frame_info_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L108-L143). At shutdown, a matching call to [`__deregister_frame`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L301-L307) or [`__deregister_frame_info_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L224-L293) would be made. If there are multiple `.eh_frame` sections, and the linker didn't merge them, then [`__register_frame_table`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L205-L210) / [`__register_frame_info_table_bases`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde.c#L168-L197) can be used instead, which take a pointer to a NULL-terminated list of pointers to `.eh_frame` sections.

A more modern approach is to add an ELF `.eh_frame_hdr` section, and use either [`_dl_find_object`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde-dip.c#L551) or [`dl_iterate_phdr`](https://github.com/gcc-mirror/gcc/blob/cc136a0bdcf096ca7d38b080a52fc9c041aa36db/libgcc/unwind-dw2-fde-dip.c#L570) to find the appropriate `.eh_frame_hdr` section. Once found, the section contains a sorted table of FDE pointers:

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

The combined size of the `eh_frame_ptr` and `fde_count` fields must be a multiple of four (which happens naturally if `eh_frame_ptr_enc` and `fde_count_enc` both use 4 or 8 byte types). The `eh_frame_ptr` field contains a pointer to the `.eh_frame` section, which is used as fallback if the `.eh_frame_hdr` section is unparseable for some reason. The `table_enc` field contains the encoding used by the contents of `sorted_table`, albeit datarel means relative to the start of `.eh_frame_hdr`, and the only supported encoding is datarel int32_t. The table is sorted in ascending `func_start` order, and the `func_start` value therein overrides the `func_start` from the referenced FDE (though in practice they should be identical).

**Personality pointers and LSDA pointers**

To support calling destructors during unwinding, and to support catching exceptions (and thereby stopping unwinding), a CIE can specify a pointer to a personality function. Said function contains the destructing and/or catching logic, and will get called as part of unwinding (unless only generating a backtrace). I won't get into the specifics of the interface, though the personality function can query various parts of the unwind state:

*   `_Unwind_GetLanguageSpecificData` returns `lsda_ptr` from the FDE.
*   `_Unwind_GetRegionStart` returns `func_start` from the FDE.

Very old versions of g++ use `eh_ptr` instead of personality functions and LSDA pointers. Unless you are implementing [`__frame_state_for`](https://github.com/gcc-mirror/gcc/blob/39d989022dd0eacf1a7b95b7b20621acbe717d70/libgcc/unwind-dw2.c#L1072-L1120), you shouldn't need to worry about `eh_ptr`.

**Expression bytecode**

The `UnwindOneFrame` function from earlier included calls to `EvalExpr` to handle complex cases. The outline of `EvalExpr` is:

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

With the various supported opcodes being:

| Opcode | Name and operands | Semantics (assuming stack for `Push`, `Pop`, `At`) |
| `0x03` | DW_OP_addr(uintptr_t Lit) | `Push(Lit)` |
| `0x08` | DW_OP_const1u(uint8_t Lit) | `Push(Lit)` |
| `0x09` | DW_OP_const1s(int8_t Lit) | `Push(Lit)` |
| `0x0A` | DW_OP_const2u(uint16_t Lit) | `Push(Lit)` |
| `0x0B` | DW_OP_const2s(int16_t Lit) | `Push(Lit)` |
| `0x0C` | DW_OP_const4u(uint32_t Lit) | `Push(Lit)` |
| `0x0D` | DW_OP_const4s(int32_t Lit) | `Push(Lit)` |
| `0x0E` | DW_OP_const8u(uint64_t Lit) | `Push(Lit)` |
| `0x0F` | DW_OP_const8s(int64_t Lit) | `Push(Lit)` |
| `0x10` | DW_OP_constu(uleb128 Lit) | `Push(Lit)` |
| `0x11` | DW_OP_consts(sleb128 Lit) | `Push(Lit)` |
| `0x12` | DW_OP_dup | `Push(At(-1))` (duplicate top element) |
| `0x14` | DW_OP_over | `Push(At(-2))` (duplicate penultimate element) |
| `0x15` | DW_OP_pick(uint8_t Idx) | `Push(At(-1-Idx))` |
| `0x13` | DW_OP_drop | `Pop()` |
| `0x16` | DW_OP_swap | `a = Pop(); b = Pop(); Push(a); Push(b)` |
| `0x17` | DW_OP_rot | `a = Pop(); b = Pop(); c = Pop(); Push(a); Push(c); Push(b)` |
| `0x06` | DW_OP_deref | `Push(*(uintptr_t*)Pop())` |
| `0x94` `0x01` | DW_OP_deref_size | `Push(*(uint8_t*)Pop())` |
| `0x94` `0x02` | DW_OP_deref_size | `Push(*(uint16_t*)Pop())` |
| `0x94` `0x04` | DW_OP_deref_size | `Push(*(uint32_t*)Pop())` |
| `0x94` `0x08` | DW_OP_deref_size | `Push(*(uint64_t*)Pop())` |
| `0x19` | DW_OP_abs | `a = Pop(); Push((intptr_t)a < 0 ? -a : a)` |
| `0x1f` | DW_OP_neg | `a = Pop(); Push(-a)` |
| `0x20` | DW_OP_not | `a = Pop(); Push(~a)` |
| `0x23` | DW_OP_plus_uconst(uleb128 Lit) | `a = Pop(); Push(a + Lit)` |
| `0x1a` | DW_OP_and | `rhs = Pop(); lhs = Pop(); Push(lhs & rhs)` |
| `0x1b` | DW_OP_div | `rhs = Pop(); lhs = Pop(); Push((intptr_t)lhs / (intptr_t)rhs)` |
| `0x1c` | DW_OP_minus | `rhs = Pop(); lhs = Pop(); Push(lhs - rhs)` |
| `0x1d` | DW_OP_mod | `rhs = Pop(); lhs = Pop(); Push(lhs % rhs)` |
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

The gcc implementation of `EvalExpr` assumes that the stack never holds more than 64 elements. Bad things will happen if the stack exceeds this size.

By far the most common usage of this expression language is to allow a single FDE to succinctly describe an arbitrary number of PLT entries. The serialised bytecode in this case will be something like:

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

Which encodes the expression:

```
rsp + 8 + ((((rip & 15) >= 11) ? 1 : 0) << 3) 
```

In other words, each 16-byte PLT entry is split into two pieces: the 1^(st) being 11 bytes long, and the 2^(nd) being 5 bytes long. If in the 1^(st) piece, the expression gives `rsp + 8`, whereas in the 2^(nd) piece it gives `rsp + 16`.

**References**

The [DWARF Standard](https://dwarfstd.org/doc/DWARF5.pdf) is documented, though `.eh_frame` diverges from it in a few areas. It also requires architecture-specific extensions (e.g. [x86](https://www.uclibc.org/docs/psABI-i386.pdf), [x86_64](http://refspecs.linux-foundation.org/elf/x86_64-abi-0.99.pdf), [aarch64](https://github.com/ARM-software/abi-aa/blob/main/aadwarf64/aadwarf64.rst)). The [LSB has a pointer encoding specification](https://refspecs.linuxbase.org/LSB_5.0.0/LSB-Core-generic/LSB-Core-generic/dwarfext.html), though it is incomplete. The only true reference is the gcc source code, some relevant entry points into which are:

**See also**

For a completely different take on the same problem, consider [ARM64 unwinding on Windows](/content/windows-arm64-unwind-codes). An `.xdata` record therein is roughly equivalent to an FDE, and it has a bytecode interpreter similar in purpose (but very different in style) to the DW_CFA_ bytecode.

Some examples of complete `.eh_frame` sections are available in LuaJIT: