# Assembly Breakdown

This section contains a cleaned version of the `perf annotate` output.
The profiling percentages, navigation arrows, and TUI decorations have been removed so the assembly can be explained line-by-line.

> **How to use:** Add the meaning of each instruction in the **Explanation** column.

## Visual Example of the Structure 
 <img width="1312" height="1199" alt="image" src="https://github.com/user-attachments/assets/bc6cec6f-c124-4f8e-9e59-518666ecc897" />


## Assembly Explained 
| # | Assembly instruction | Explanation |
|---:|---|---|
| 1 | `push %r15` | save register r15 on stack |
| 2 | `mov %rsi,%r15` | copie the value of rsi to register r15|
| 3 | `push %r14` | save register r14 on stack |
| 4 | `mov %edx,%r14d` | copie value of edx to register 14{d signifies because the destination is the r14d it copies the lower 32 bits and zero-extends them into r14 |
| 5 | `push %r13` | save register r13 |
| 6 | `push %r12` | save register r12 |
| 7 | `push %rbp` | save the old rbp.rbp is callee-saved |
| 8 | `push %rbx` | save the old rbx.Same is callee-saved |
| 9 | `mov %rdi,%rbx` | copy the first function argument into rbx.In the System V AMD64 ABI the first argument is in rdi |
| 10 | `mov %rsi,%rdi` | move the second argument into the first-argument register |
| 11 | `sub $0x78,%rsp` | allocate 120 bytes of stack space 0x78=120 decimal,rsp moves downward by 120 bytes,creating space for local variables/temporary data |
| 12 | `mov %ecx,0x14(%rsp)` | store the 4rth argument on the stack.Ecx contains the fourth integer argument and it is stored at mem address rsp+0x14 |
| 13 | `mov %fs:0x28,%rax` | load the stack-canary value. fs:0x28 is a thread-local location commonly containing the stack protection value. It is loaded into rax |
| 14 | `mov %rax,0x68(%rsp)` | Save the stack canary in this function's stack frame.The value from rax is stored at rsp+0x68 |
| 15 | `xor %eax,%eax` | Set eax to zero.XORing a register with itself produces zero.Eax is the lower 32 bits of rax,writting zero to eax also clears the entire rax register x86-64 |
| 16 | `call llama_synchronize@plt` | calling function.The call instruction saves the return address on the stack and jumps to the function {@plt means the call goes through Procedure Linkage Table,which is used for dynamically linked functions |
| 17 | `lea 0x30(%rsp),%rax` | calculate the address,without loading the memory contents. lea means "load effective address". This puts rsp+0x30 into rax. It is essentially calculating a pointer to a local stack object |
| 18 | `movzbl 0x6d(%rbx),%edx` | Load one byte from memory and zero-extend it to 32 bits.It reads the byte at rbx+0x6d,places it in edx and fills the upper bits with 0s|
| 19 | `lea 0x270(%rbx),%rsi` | calculate the address by adding 0x270 to %rbx and stores the resulting address in %rsi |
| 20 | `mov %rax,%rdi` | copy the value in %rax into %rdi prepareing the first function argument |
| 21 | `mov %rax,0x8(%rsp)` | Store the value in %rax at the stack location 8(%rsp) |
| 22 | `call common_time_meas::common_time_meas(long&, bool)@plt` | call the common_time_meas constructor through the PLT |
| 23 | `mov %r14d,%esi` | copy the lower 32 bits of %r14 into %esi,preparing a function argument |
| 24 | `mov %r15,%rdi` | copy the %r15 into %rdi,preparing the first argument |
| 25 | `call llama_get_sampled_probs_ith@plt` | call the llama_get_sampled_probs_ith retrieving the sampled probabilities for a given token position |
| 26 | `mov %r14d,%esi` | Copy the lower 32 bits of %r14 into %esi for the next function call |
| 27 | `mov %r15,%rdi` | copies %r15 into %rdi as the first argument |
| 28 | `mov %rax,%r13` | save the return value from the previous function call in to %r13 |
| 29 | `call llama_get_sampled_logits_ith@plt` | call llama_get_sampled_logits_ith retrieving the sampled logits for the specified position |
| 30 | `mov %r14d,%esi` | copy the lower 32 bits of %r14 inot %esi for the next call |
| 31 | `mov %r15,%rdi` | copy %r15 into %rdi as the first argument |
| 32 | `mov %r14d,(%rsp)` | store the lower 32 bits of %r14 at the current stack location |
| 33 | `mov %rax,%rbp` | save the return value from llama_get_sampled_logits_ith in %rbp |
| 34 | `call llama_get_sampled_candidates_ith@plt` | call llama_get_sampled_candidates_ith retrievinf the sampled candidate tokens |
| 35 | `mov %r15,%rdi` | copie r15 into rdi preparing the argument for the next function call |
| 36 | `mov %rax,%r12` | save the return value from llama_get_sampled_candidates_ith in r12 |
| 37 | `call llama_get_model@plt` | call llama_model_get_vocab to retrive the vocabulary assiciated with the model |
| 38 | `mov %rax,%rdi` | move the returned model pointer from rax to rdi |
| 39 | `call llama_model_get_vocab@plt` | call the llama_model_get_vocab to retrieve the vocabulary associated with the model |
| 40 | `mov %rax,%rdi` | move the returned vocabulary pointer to rdi,preparing it as the first argument for the following call |
| 41 | `call llama_vocab_n_tokens@plt` | call llama_vocab_n_tokens to obtain the total number of tokens in the model vocabulary |
| 42 | `mov (%rsp),%esi` | load the 32-bit value stored at the top of the stack into esi|
| 43 | `mov %eax,%r14d` | save the return value from llama_vocab_n_tokens(the vocabulary size) in r14d |
| 44 | `mov %r15,%rdi` | copy r15 into rdi preparing the first argument for the next call |
| 45 | `test %r13,%r13` | test whether r13 is zero by performing a bitwise AND of the register with itself.Sets the CPU flags without modifying r13 |
| 46 | `je 370` | jump to address 370 if the Zero flag is set,meaning r13 was zero |
| 47 | `call llama_get_sampled_probs_count_ith@plt` | call llama_get_sampled_probs_count_ith to obtain the number of sampled probability entries for the specified position |
| 48 | `mov 0x240(%rbx),%rcx` | load the 64-bit value stored at memory address rbx+0x240 into rcx |
| 49 | `mov %eax,%r11d` |  save the 32-bit return value from llama_get_sampled_probs_count_ith in r11d |
| 50 | `mov 0x238(%rbx),%rdx` | loads the 64-bit value at memory address rbx + 0x238 |
| 51 | `movabs $0xaaaaaaaaaaaaaaab,%rdi` | load the constant **0xAAAAAAAAAAB** into rdi. This a compiler-generated constant used for efficient division by 3 through multiplication and shifting |
| 52 | `mov %r11,0x18(%rsp)` | save the r11 on the stack at offset 0x18 from rsp |
| 53 | `mov %r11,%r14` | copy the sampled-probability count from r11 into r14 |
| 54 | `mov %rcx,%rax` | copy the rcx into rax as the starting value for an arithmetic calculation |
| 55 | `sub %rdx,%rax` | substract rdx from rax,calculating the difference between the 2 addresses/values |
| 56 | `sar $0x2,%rax` | Arithmetic-shift rax by 2 bits,effectively dividing the value by 4 while preserving the sign |
| 57 | `imul %rdi,%rax` | multiply rax by the constant in rdi |
| 58 | `mov %rax,%rsi` | copy the calculated value from rax into rsi,preparing it for comparison/use as another argument |
| 59 | `cmp %r11,%rax` | compare the calculated value in rax with r11 by computing rax-r11 and setting the CPU flags |
| 60 | `jb 9f8` |  jump to 9f8 if rax is below r11 in an unsigned comparison |
| 61 | `cmp %rsi,%r11` | compare r11 with rsi, setting the CPU flags according to r11-rsi |
| 62 | `jb 9c8` | jump to 9c8 if r11 is below rsi in an unsigned comparison |
| 63 | `f7: test %r14d,%r14d` | Test whether the sampled-probability count in r14 is zero. The f7 label marks a branch target in the disassembly |
| 64 | `je 2a0` | jump to 2a0 if r14d is zero,skipping the following processing when there are no sampled probabilities |
| 65 | `100: cmp $0x4,%r14d` | Compare the sampled-probability count in r14d with 4. The 100: label marks another branch target |
| 66 | `jbe 960` | jump to 960 if r14d is less than or equal to 4 using an unsigned comparison |
| 67 | `mov 0x18(%rsp),%rcx` | reload the previously saved sampled-probability count from the stack into rcx |
| 68 | `lea 0x4(%rdx),%rdi` | calculate rdx+4 and stores the resulting address/value in rdi |
| 69 | `lea 0x0(,%rcx,4),%rax` | calculate rcx *4 using lea,without performing a memory access |
| 70 | `lea (%rcx,%rcx,2),%rcx` | calculates rcx * 3 using lea :rcx + 2* rcx |
| 71 | `shl $0x2,%rcx` | shift rcx left by 2 bits, multiplying its value by 4 |
| 72 | `lea 0x0(%r13,%rax,1),%r8` | calculate the r13 + rax and store the result in r8 |
| 73 | `lea (%rdx,%rcx,1),%r9` | calculate the rdx + rcx and store the result in r9 |
| 74 | `cmp %r8,%rdi` | compare the r8 with rdi by internally computing rdi-r8 and setting the CPU flags  |
| 75 | `setae %r8b` | set the r8b to i if rdi is greater than or equal to r8 in an unsigned comparison,otherwise set it to 0 |
| 76 | `cmp %r9,%r13` | compare r13 with r9 using an unsigned comparison |
| 77 | `setae %r10b` | set r10b to 1 if r13 is greater than or equal to r9 otherwise set it to 0 |
| 78 | `or %r10d,%r8d` | perform a bitwise OR between r10d and r9d,combining the results of the 2 comparison conditions |
| 79 | `lea 0x0(%rbp,%rax,1),%r10` | calculate rbp + rax and store the result in r10 |
| 80 | `cmp %r10,%rdi` | compare r10 with rdi  setting the CPU flags for an unsigned comparison |
| 81 | `setae %dil` | set dil to 1 if rdi is greater than or equal to r10 otherwise set it to 0 |
| 82 | `cmp %r9,%rbp` | compare r9 to rbp  using an unsinged comparison |
| 83 | `setae %r9b` | set r9b to 1 if rbp is greater than or equal to r9b otherwise set it to 0|
| 84 | `or %r9d,%edi` |  perform a bitwise OR op between r9d and edi combining the comparison result |
| 85 | `test %dil,%r8b` | perform a bitwise AND op between dil and r8b to determine whether both condition results are non-zero |
| 86 | `je 960` | jump to 960 if the result of the previous test is zero,meaning required conditions were not satisfied |
| 87 | `lea -0x8(%rdx,%rcx,1),%rcx` | calculate rdx + rcx - 8 and store the result into rcx |
| 88 | `cmp %rcx,%r12` | compare rcx with r12, setting the CPU flags for an usnsigned comparison |
| 89 | `setae %cl` |  set c1 to 1 if  r12 is greater than or equal to rcx,otherwise set c1 to 0|
| 90 | `add %r12,%rax` | add r12 to rax |
| 91 | `cmp %rax,%rdx` | compare rax with rdx setting the CPU flag for an unsigned comparison |
| 92 | `setae %al` | set a1 to 1 if rdx is greater than or equal to rax,otherwise set a1 to 0 |
| 93 | `or %al,%cl` | perform a bitwise OR op between c1 and a1 combining the 2 comparison results |
| 94 | `je 960` | jump to 960 if both comparison conditions resulted in zero |
| 95 | `mov %r14d,%ecx` | copy the sampled-probability count from r14d into ecx |
| 96 | `mov %rdx,0x18(%rsp)` | save the value of rdc to the stack at offset 0x18(%rsp) |
| 97 | `mov %rdx,%rax` | copy rdx into rax  |
| 98 | `shr $0x2,%ecx` | logical shift right on the ecx by 2 bits,effectively dividing the unsugned value by 4 |
| 99 | `shl $0x4,%rcx` | shl rcx left by 4 bits,multiplying its value by 16 |
| 100 | `mov %rcx,%rdi` | copy rcx into rdi |
| 101 | `xor %ecx,%ecx` | clear ecx,setting it to 0.This initializes the loop offset/index |
| 102 | `mov %rdi,%rdx` | copy rdi into rdx,preserving the calculated value for use as a loop boundary or destination address |
| 103 | `nop` | No operation,does nothing |
| 104 | `198: lea (%r12,%rcx,1),%rdi` | calculates r12 + rcx and stores the resulting address in rdi.This forms the address of the current element in r12 data region |
| 105 | `add $0x30,%rax` | add 0x30(48bytes) to rax advancing the destination pointer by 48 |
| 106 | `mov 0xc(%rdi),%r8d` | load a 32-bit value from offset 12 of the current rdi structure/element into r8d |
| 107 | `mov 0x8(%rdi),%r9d` | load a 32-bit value from offset 8 into r9d |
| 108 | `mov 0x4(%rdi),%r10d` | load a 32-bit value from offset 4 into r10d |
| 109 | `mov (%rdi),%r11d` | load the first 32-bit value at rdi into r11d |
| 110 | `lea 0x0(%rbp,%rcx,1),%rdi` | calculates rbp + rcx obtaining the address od the corresponding element in the rbp data region |
| 111 | `movss 0xc(%rdi),%xmm0` | load a single-precision floating-point value (32-bit float) from offset 12 into xmm0 |
| 112 | `movss 0x8(%rdi),%xmm1` | load a single-precision floating-point value (32-bit float) from offset 8 into xmm1 |
| 113 | `movss 0x4(%rdi),%xmm2` | load a single-precision floating-point value (32-bit float) from offset 4 into xmm2 |
| 114 | `movss (%rdi),%xmm3` | load a single-precision floating-point value (32-bit float) from rdi into xmm3  |
| 115 | `lea 0x0(%r13,%rcx,1),%rdi` | calculate r13+rcx,obtaining the address of the corresponding element in the r13 data region |
| 116 | `add $0x10,%rcx` | Advance the loop offset by 16 bytes |
| 117 | `movss 0xc(%rdi),%xmm4` |  load a single-precision floating-point value (32-bit float) from offset 12 into xmm4 |
| 118 | `movss 0x8(%rdi),%xmm5` | load a single-precision floating-point value (32-bit float) from offset 8 into xmm5 |
| 119 | `movss 0x4(%rdi),%xmm6` | load a single-precision floating-point value (32-bit float) from offset 4 into xmm6 |
| 120 | `movss (%rdi),%xmm7` |  load a single-precision floating-point value (32-bit float) from rdi into xmm7 |
| 121 | `mov %r11d,-0x30(%rax)` | store the first 32-bit value loaded from the r12 region into the destination buffer at offset -48 |
| 122 | `unpcklps %xmm5,%xmm1` | interleaves the lower single-precision point values of xmm1 and xmm5,preparing for a packed/SIMD layout |
| 123 | `unpcklps %xmm4,%xmm0` | interleaves the lower single-precision point values of xmm0 and xmm4 |
| 124 | `mov %r10d,-0x24(%rax)` | store the value from r10d into the destination buffer at offset -36 |
| 125 | `unpcklps %xmm7,%xmm3` | interleaves the lower single-precision point values of xmm7 and xmm3 |
| 126 | `unpcklps %xmm6,%xmm2` | interleaves the lower single-precision point values of xmm2 and xmm6 |
| 127 | `mov %r9d,-0x18(%rax)` | store the value from r9d into the destination buffer at offset -18 |
| 128 | `mov %r8d,-0xc(%rax)` | store the value from r8d into the destination buffer at offset -12 |
| 129 | `movlps %xmm3,-0x2c(%rax)` | store the lower 64 bits of xmm3(2 32-bit floating-point values) into the destination buffer at offset -44 |
| 130 | `movlps %xmm2,-0x20(%rax)` | store the lower 64 bits of xmm2(2 32-bit floating-point values) into the destination buffer at offset -20 |
| 131 | `movlps %xmm1,-0x14(%rax)` | store the lower 64 bits of xmm1(2 32-bit floating-point values) into the destination buffer at offset -44 |
| 132 | `movlps %xmm0,-0x8(%rax)` | store the lower 64 bits of xmm0(2 32-bit floating-point values) into the destination buffer at offset -8 |
| 133 | `cmp %rdx,%rcx` | compare the current loop offset rcx against the loop boundary stored in rdx |
| 134 | `jne 198` | jump back to label 198 if rcx and rdx are not equal,continuing the loop |
| 135 | `mov %r14d,%eax` | copy the sampled-probability count from r14d into eax |
| 136 | `mov 0x18(%rsp),%rdx` | reload the saved value from stack offset 0x18 (%rsp) into rdx |
| 137 | `and $0xfffffffc,%eax` | clear the lowest 2 bits of eax,effectively rounding the count down to the nearest multiple of 4| 
| 138 | `test $0x3,%r14b` | test the lowest 2 bits of r14b to determine whether the sampled count has a remainder when divided by 4|
| 139 | `je 2a0` | jump to 2a0 if the lowest 2 bits of r14b are 0,meaning the count is already divisible by 4|
| 140 | `mov %eax,%ecx` |  copy the rounded-down count from eax into ecx |
| 141 | `movss 0x0(%r13,%rcx,4),%xmm1` | load a single-precision floating-point value from r13 + rcx * 4 into %xmm1 |
| 142 | `movss 0x0(%rbp,%rcx,4),%xmm0` | load a single-precision floating-point value from r13 + rcx * 4 into %xmm0 |
| 143 | `lea (%rcx,%rcx,2),%rdi` | calculate rcx * 3 using lea and store the result in rdi |
| 144 | `mov (%r12,%rcx,4),%r8d` | load a 32-bit value from r12 + rcx *4 into r8d |
| 145 | `lea (%rdx,%rdi,4),%rdi` | calculate the rdx + (%rdi * 4) effectively rdx + rcx * 12 and stores the resulting address in rdi |
| 146 | `lea 0x1(%rax),%ecx` | calculates eax +1 and stores the result in ecx advancing the element index by one |
| 147 | `unpcklps %xmm1,%xmm0` | interleaves the lower single-precision values from xmm0 and xmm1 creating a packed paid of floats |
| 148 | `mov %r8d,(%rdi)` | store the 32-bit value from r8d at the calculated sestination address |
| 149 | `movlps %xmm0,0x4(%rdi)` | store the lower 64 bits of xmm0 at offset 4 from the destination address |
| 150 | `cmp %r14d,%ecx` | compare r14 with ecx setting the CPU flag |
| 151 | `jae 2a0` | jump to 2a0 if ecx is greater than or equal to r14d  |
| 152 | `movss 0x0(%r13,%rcx,4),%xmm1` | load a 32-bit float-point value from r13 + rcx *4 into xmm1 |
| 153 | `mov (%r12,%rcx,4),%r8d` |  move a 32-bit float-point value from r12 + rcx * 4 into r8d|
| 154 | `lea (%rcx,%rcx,2),%rdi` | calculate rcx * 3 and stores the result in rdi |
| 155 | `add $0x2,%eax` | add 2 to eax, advancing the processed-element index by 2 |
| 156 | `movss 0x0(%rbp,%rcx,4),%xmm0` | load 32-bit float-value from rbp + rcx * 4 into xmm0 |
| 157 | `lea (%rdx,%rdi,4),%rdi` | calculate the rdix + rdi *12 producing the destination address  |
| 158 | `mov %r8d,(%rdi)` | store the 32-bit value from r8d at the previously calculated address |
| 159 | `unpcklps %xmm1,%xmm0` | interleave the 2 loaded single-prec floating-point values in xmm0 and xmm1 |
| 160 | `movlps %xmm0,0x4(%rdi)` | store the 64 bits of xmm0 at offset 4 from destination |
| 161 | `cmp %r14d,%eax` | compare r14d with eax  |
| 162 | `jae 2a0` | jump to 2a0 if eax greater than or equal to r14d |
| 163 | `mov (%r12,%rax,4),%edi` | load 32 bit value from r12 + rax * 4 into edi |
| 164 | `lea (%rax,%rax,2),%rcx` | calculate rax * 3 and store the result into rcx |
| 165 | `movss 0x0(%r13,%rax,4),%xmm1` | load a 32-bit floating point value from r13 + rax * 4 into xmm1 |
| 166 | `movss 0x0(%rbp,%rax,4),%xmm0` | load a 32 bit floating point value from rbp + rax * 4 into xmm0|
| 167 | `lea (%rdx,%rcx,4),%rcx` | calculate rdx +(rcx * 4) into rcx,producing the destination address |
| 168 | `mov %edi,(%rcx)` | store the value edi into the address calculated previously |
| 169 | `unpcklps %xmm1,%xmm0` | interleave the 2 single-precision floating point values in xmm0 and xmm1 |
| 170 | `movlps %xmm0,0x4(%rcx)` | store the lower 64 bits of xmm0 at offset 4 from the destination address |
| 171 | `nop` | No operation |
| 172 | `2a0: mov %rsi,0x258(%rbx)` | label 2a0 store the value of rs1 into the structure at offset 0x258 from rbx.This is likely updating an internal pointer/counter |
| 173 | `mov (%rsp),%esi` | load into esi the 32bit value stored at the top of the stack |
| 174 | `mov %r15,%rdi` | copy r15 into rdi |
| 175 | `mov %rdx,0x250(%rbx)` | store rbx into the structure at offset 0x250(%rbx) |
| 176 | `movq $0xffffffffffffffff,0x260(%rbx)` | store -1{0xffffffffffffffff} at offset 0x260(rbx), likely marking a value as invalid/uninitialized |
| 177 | `movb $0x0,0x268(%rbx)` | store 0 as a byte at offset 0x268(%rbx) clearing/resetting a flag or state field |
| 178 | `call llama_get_sampled_token_ith@plt` | call llama_get_sampled_token_ith to retrieve the sampled token at the specified position |
| 179 | `mov %eax,%ebp` | save the return token ID in ebp |
| 180 | `cmp $0xffffffff,%eax` | compare the returning id eax with -1  |
| 181 | `je 568` | jump to label 568 if eax equals -1 |
| 182 | `call common_log_get_verbosity_thold()@plt` | call the logging subsystem to obtain the current logging verbosity threshold |
| 183 | `cmp $0x4,%eax` | compare the logging verbosity level with 4 |
| 184 | `jg 998` | jump to label 998 if eax is greater than 4 |
| 185 | `2e4: cmpq $0x0,0x1e8(%rbx)` | label 2e4 checks whether the 64-bit value at offset 0 |
| 186 | `jne 105e` | jump to label 105e if the value checked on the previous line is non zero |
| 187 | `cmpq $0x0,0x1f0(%rbx)` | check whether the 64-bit value at offset 0x1f0 is zero  |
| 188 | `jne 1029` | jump to label 1029 if the previous value checked is non zero |
| 189 | `mov 0x258(%rbx),%rcx` | load the value stored at offset 0x258(%rbx) into rcx |
| 190 | `test %rcx,%rcx` |  test whether rcx is zero by performing a bitwise AND on rxc setting the CPU flags without changing rcx |
| 191 | `je 338` | jump to label 338 if rcx is zero  |
| 192 | `mov 0x250(%rbx),%rdx` | load the value stored at offset 0x250(%rbx) into rdx |
| 193 | `xor %eax,%eax` | clears eax,setting it to zero. This initializes the loop counter |
| 194 | `jmp 32d` | unconditionally jump to label 32d entering the loop's condition check |
| 195 | `nop` | No operation |
| 196 | `320: add $0x1,%rax` | label 320, increment the loop counter rax by 1 |
| 197 | `add $0xc,%rdx` | advances rdx by 12 bytes, moving the next entry in the data structure |
| 198 | `cmp %rax,%rcx` | compare rax with rcx[loop counter with the number of entries stored] |
| 199 | `je 338` | jump to label 338 if rax is equal to rcx |
| 200 | `32d: cmp %ebp,(%rdx)` | label 32d compare ebp to rdx [sampled token ID cmp with the 32-bit value stored at the mem address pointed by rdx]|
| 201 | `jne 320` | jump to label 320 if rdx not equal to ebp |
| 202 | `mov %rax,0x260(%rbx)` | store rax at offset 0x260(%rbx)  |
| 203 | `338: mov 0x8(%rsp),%rdi` | label 338 load the value stored at stack offset 0x8 into rdi, preparing the argument for the destructor call |
| 204 | `call common_time_meas::~common_time_meas()@plt` | call the common_time_meas destructor, cleaning up the timing-measurement object |
| 205 | `mov 0x68(%rsp),%rax` | load the saved stack-canary value from 0x68(%rsp) into rax |
| 206 | `sub %fs:0x28,%rax` | substract the current thread-local stack-canary value from the saved value to check whether the stack has been corrupted |
| 207 | `jne f91` | jump to label f91 if the stack-canary values differ, indicating possible stack corruption |
| 208 | `add $0x78,%rsp` | release 0x78(120) bytes of stack space used by the function |
| 209 | `mov %ebp,%eax` | move the function's result/token ID from ebp into eax, the standard return-value register |
| 210 | `pop %rbx` | restore the saved rbx register from the stack |
| 211 | `pop %rbp` | restore the saved rbp register from the stack |
| 212 | `pop %r12` | restore the saved r12 register from the stack |
| 213 | `pop %r13` | restore the saved r13 register from the stack |
| 214 | `pop %r14` | restore the saved r14 register from the stack |
| 215 | `pop %r15` | restore the saved r15 register from the stack |
| 216 | `← ret` | return from the current function to the caller |
| 217 | `nop` | No operation |
| 218 | `370: test %rbp,%rbp` | label 370 test whether the rbp is zero by performing a bitwise AND [ %rbp & %rbp] setting the CPU flags without modifying rbp |
| 219 | `je a70` | jump tp label a70 if rbp is zero |
| 220 | `call llama_get_sampled_logits_count_ith@plt` | call llama_get_sampled_logits_count_ith to obtain the number of sampled logits for the specified position |
| 221 | `mov 0x240(%rbx),%rcx` |  load the value at offset 0x240(%rbx) into rcx |
| 222 | `mov %eax,%r14d` | save the returned sampled-logit count in r14d |
| 223 | `mov 0x238(%rbx),%rdx` | load the value at offset(%rbx) into rdx|
| 224 | `movabs $0xaaaaaaaaaaaaaaab,%rdi` | load the compiler-generated magic constant used as part of an optimized division-by-3 calculation |
| 225 | `mov %r14,%r13` | copy the sampled-logit count from r14 into r13 |
| 226 | `mov %rcx,%rax` | copy rcx into rax as the starting value of an address/size calculation |
| 227 | `sub %rdx,%rax` | subtract rdx from rax,calculating the difference between the 2 values |
| 228 | `sar $0x2,%rax` | arithmetic right-shift rax by 2 bits,effectively dividing by 4 |
| 229 | `imul %rdi,%rax` | multiply rax by the magic constant as part of the compiler's optimized division calculation |
| 230 | `mov %rax,%rsi` | copy the calculated value into rsi |
| 231 | `cmp %r14,%rax` | compare r14 with rax |
| 232 | `jb b20` | jump to label b20 if rax is bellow r14 in an unsigned comparison |
| 233 | `cmp %rax,%r14` | compare rax with r14 |
| 234 | `jae 3de` | jump to label 3de if r14 is greater than or equal to rax in an unsigned comparison |
| 235 | `lea (%r14,%r14,2),%rax` | calculate r14 + ra14 * 2 and store the result in rax |
| 236 | `shl $0x2,%rax` | shift-left rax by 2 bits,multiplying it by 4 |
| 237 | `lea (%rdx,%rax,1),%r8` | calculate rdx + rax producing an address offset by 12 * r14 |
| 238 | `cmp %r8,%rcx` | conpare rcx with the calculated address r8 |
| 239 | `je 3de` | jump to label 3de if rcx equal to r8 |
| 240 | `sar $0x2,%rax` | divide rax by 4,using an arithmetic right shift |
| 241 | `mov %r8,0x240(%rbx)` | store the newly calculated pointer/counter into the structure at offset 0x240(%rbx) |
| 242 | `imul %rdi,%rax` | multiply rax by the magic constant as part of another optimized division calculation |
| 243 | `mov %rax,%rsi` | copy the result into rsi |
| 244 | `3de: test %r13d,%r13d` | laber 3de test whether the sampled-logit count in r13d is zero |
| 245 | `je 2a0` | jump to label 2a0 if r13d equals 0,skipping the processing loop |
| 246 | `cmp $0x8,%r13d` | compare r13d with 8 |
| 247 | `jbe a38` | jump to label a38 if the counter r13d is less than or equal to 8 using an unsigned comparison |
| 248 | `3f1: lea 0x0(,%r14,4),%rcx` | label 3f1 calculate r14 * 4 and store the result into rcx  |
| 249 | `lea (%r14,%r14,2),%rax` | calculate r14 * 3 and store the result in rax |
| 250 | `shl $0x2,%rax` | shift left rax by 4, multiplying rax by 4 |
| 251 | `lea 0x0(%rbp,%rcx,1),%rdi` | calculate rbp + rcx producing an address based on the sampled-logit index |
| 252 | `lea 0x4(%rdx),%r8` | calculate rdx + 4 producing the address immediately after the first 4-byte field |
| 253 | `cmp %rdi,%r8` | compare r8 with rdi using an unsigned comparison as part of a mem range check |
| 254 | `lea (%rdx,%rax,1),%r8` | calculate the rdx + rax effectively rdx + 12 * r14, producing the end of the relevant mem range |
| 255 | `setae %dil` | set dil to 1 if r8 is greather than or equal to rdi,otherwise set dil to 0 |
| 256 | `cmp %r8,%rbp` | compare r8 with rbp |
| 257 | `setae %r8b` | set r8b to 1 if rbp is greater than or eqaul to r8,otherwise set it to 0 |
| 258 | `or %r8b,%dil` | combine the 2 boolean comparison results using a bitwise OR |
| 259 | `je a38` | jump to label a38 if the combined result is zero,indicating the required mem-range conditions were not satisfied |
| 260 | `lea -0x8(%rdx,%rax,1),%rax` | calculate rdx + rax -8,producing an address near the end of the mem region |
| 261 | `cmp %rax,%r12` | compare rax with r12 |
| 262 | `setae %al` | set al to 1 if r12 is greater than or equal to rax,otherwise set it to 0 |
| 263 | `add %r12,%rcx` | add r12 to rcx |
| 264 | `cmp %rcx,%rdx` | compare rcx with rdx|
| 265 | `setae %cl` | set cl to 1 if rdx is greater than or equal to rcx,otherwise set cl to 0 |
| 266 | `or %cl,%al` | combines the 2 boolean comparison results using a Bitwise OR |
| 267 | `je a38` | jump to laber a38 if the the combined result is zero, failing the pointer/range validation |
| 268 | `mov %r13d,%r11d` | copy the sampled-logit count into r11d |
| 269 | `mov %rdx,%rax` | copy rdx into rax |
| 270 | `xor %ecx,%ecx` | clear ecx, initializing the loop offset to 0|
| 271 | `shr $0x2,%r11d` | logic shift right r11d by 4, dividing the r11d by 4 |
| 272 | `shl $0x4,%r11` | logic shift left r11 by 16 |
| 273 | `nop` | No operation |
| 274 | `458: lea (%r12,%rcx,1),%rdi` | label 458, calculate r12+ rcx obtaining the address of the current input element |
| 275 | `add $0x30,%rax` | advances the destination pointer[rax] by 48 bytes |
| 276 | `mov 0xc(%rdi),%r8d` | load a 32-bit value at offset 12 from the current input element |
| 277 | `mov 0x8(%rdi),%r9d` | load the 32-bit value at offset 8 |
| 278 | `mov 0x4(%rdi),%r10d` | load the 32-bit value at offset 4|
| 279 | `mov (%rdi),%r14d` | load the first 32-bit value from the current input element into r14d |
| 280 | `lea 0x0(%rbp,%rcx,1),%rdi` | calculate rbp + rcx,obtaining the corresponding address in the second input region |
| 281 | `add $0x10,%rcx` | advance the loop offset by 16 bytes,moving to the next group of input data |
| 282 | `movss 0x8(%rdi),%xmm1` | load a 32-bit single-precision floating-point value from offset 8 into xmm1 |
| 283 | `movss 0x4(%rdi),%xmm2` | load a 32-bit floating-point value from offset 4 into xmm2 |
| 284 | `movss (%rdi),%xmm3` | load a 32-bit floating-point value from offset 0 into xmm3 |
| 285 | `movss 0xc(%rdi),%xmm0` | load a 32-bit floating-point value from offset 12 into xmm0 |
| 286 | `mov %r14d,-0x30(%rax)` | store the 32-bit value from r14d into the destination at offset -48 |
| 287 | `mov %r10d,-0x24(%rax)` | store the value from r10d at destination offset -36 |
| 288 | `mov %r9d,-0x18(%rax)` | store the value from r19d at destination offset -24 |
| 289 | `mov %r8d,-0xc(%rax)` |store the value from r8d at destination offset -12 |
| 290 | `movss %xmm3,-0x2c(%rax)` | store the 32-bit floating-point value from xmm3 at destination offset -44 |
| 291 | `movss %xmm2,-0x20(%rax)` | store the 32-bit floating-point value from xmm2 at destination offset -32 |
| 292 | `movss %xmm1,-0x14(%rax)` | store the 32-bit floating-point value from xmm1 at destination offset -20 |
| 293 | `movss %xmm0,-0x8(%rax)` | store the 32-bit floating-point value from xmm0 at destination offset -8 |
| 294 | `movl $0x0,-0x28(%rax)` |  store zero in the 32-bit field at offset -40 |
| 295 | `movl $0x0,-0x1c(%rax)` | store zero at offset -28 |
| 296 | `movl $0x0,-0x10(%rax)` | store zero at offset -16|
| 297 | `movl $0x0,-0x4(%rax)` | store zero at offset -4 |
| 298 | `cmp %r11,%rcx` | compare r11 with rcx [rcx= current loop offset and r11 calculated loop boundary] |
| 299 | `jne 458` | jump to label 458 if the loop offset has not reached the boundary, continuing the loop |
| 300 | `mov %r13d,%eax` | copy the original sampled-logit count from r13d into eax,preparing it for subsequent processing |
| 301 | `and $0xfffffffc,%eax` | clear the lowest 2 bits of eax,rounding the count down to a multiple of 4 |
| 302 | `test $0x3,%r13b` | test whether r13 has any of its lowest 2bit set -effectively checks r13 % 4 |
| 303 | `je 2a0` | jump to label 2a0 if r13b is divisible by 4, meaning no remainder elements need processing |
| 304 | `mov %eax,%ecx` |  copy the rounded-down count into ecx, using it as the starting index for the remainder loop |
| 305 | `mov (%r12,%rcx,4),%r8d` | load a 32-bit value from the r12 array at index rcx ( r12 + rcx *4) |
| 306 | `movss 0x0(%rbp,%rcx,4),%xmm0` | load the corresponding 32-bit floating-point value from the rbp array into xmm0 |
| 307 | `lea (%rcx,%rcx,2),%rdi` | compute 3 * rcx using LEA|
| 308 | `lea 0x1(%rax),%ecx` | compute eax + 1 and stores it into ecx without modifying eax |
| 309 | `lea (%rdx,%rdi,4),%rdi` | compute destination address rdx + (3 * old_rcx * 4) = rdx + 12 * old rcx |
| 310 | `mov %r8d,(%rdi)` | store the loaded 32-bit value into the destination structure |
| 311 | `movl $0x0,0x8(%rdi)` |  store 0 into the destination field at offset +8 |
| 312 | `movss %xmm0,0x4(%rdi)` | storethe floating-point value inot the destination field at offset +4 |
| 313 | `cmp %r13d,%ecx` | compare r13d with ecx |
| 314 | `jae 2a0` | jump to label 2a0 if ecx greater than or equal to r13d,meaning no more elements remaining |
| 315 | `mov (%r12,%rcx,4),%r8d` | load the 32-bit value from r12 array into r8d |
| 316 | `movss 0x0(%rbp,%rcx,4),%xmm0` | load the corresponding floating-point value from rbp into xmm0 |
| 317 | `lea (%rcx,%rcx,2),%rdi` | compute 3 * rcx  |
| 318 | `add $0x2,%eax` | advance eax by 2 |
| 319 | `lea (%rdx,%rdi,4),%rdi` | compute destination address rdx + rdi * 4 |
| 320 | `mov %r8d,(%rdi)` | store the 32-bit value in r8d at the beginning of the destination entry |
| 321 | `movl $0x0,0x8(%rdi)` | store 0 at destination offset +8 |
| 322 | `movss %xmm0,0x4(%rdi)` | store the 32-bit floating-point value from the lower part of xmm0 at destination offset +4 |
| 323 | `cmp %r13d,%eax` | compare eax with r13d |
| 324 | `jae 2a0` | jump to label 2a0 if eax greater than or equal to r13d |
| 325 | `mov (%r12,%rax,4),%edi` | load a 32-bit value from r12 + rax *4 into edi |
| 326 | `movss 0x0(%rbp,%rax,4),%xmm0` | load the corresponding 32-bit float from rbp + rax * 4 inot xmm0 |
| 327 | `lea (%rax,%rax,2),%rcx` | compute 3 * rax and store it in rcx |
| 328 | `lea (%rdx,%rcx,4),%rcx` | compute destination address rdx + rcx * 4 and store it into rcx |
| 329 | `mov %edi,(%rcx)` | sore the loaded 32-bit value at the begging of the destination entry |
| 330 | `movl $0x0,0x8(%rcx)` | store 0 at destination offset +8 |
| 331 | `movss %xmm0,0x4(%rcx)` |  store the floating-point value at the destination offset +4 |
| 332 | `jmp 2a0` | jump to label 2a0 after the remaining elements have been processed |
| 333 | `nop` | No operation  |
| 334 | `568: lea 0x250(%rbx),%r12` | label 568. compute the address rbx + 0x250 and store it in r12 |
| 335 | `mov 0x1f0(%rbx),%rdi` | load the 64-bit value stored at rbx + 0x1f0 into rdi,preparing the first fuction argument |
| 336 | `mov %r12,%rsi` | copy r12 into rsi|
| 337 | `call llama_sampler_apply@plt` | call llama_sampler_apply with arguments prepared in rsi and rdi |
| 338 | `cmpb $0x0,0x14(%rsp)` | compare the byte at stack address rsp + 0x14 with 0,this checks a boolean/flag |
| 339 | `je 5b2` | jump to label 5b2 if that flag is 0 |
| 340 | `mov 0x1e8(%rbx),%rdi` | load the 64-bit value at rbx+ 0x1e8 into rdi |
| 341 | `test %rdi,%rdi` | test rdi against zero by performing rdi & rdi,set the zero flag if it is zero |
| 342 | `je 5b2` | jump to label 5b2 if rdi is equal to 0 |
| 343 | `mov 0x1f0(%rbx),%rax` | load the value rbx + 01f0 into rax |
| 344 | `test %rax,%rax` | test whether rax is 0 by performing rax & rax |
| 345 | `je 5aa` | jump to label 5aa if rax is equal to 0|
| 346 | `cmpb $0x0,0xd0(%rbx)` | compare the byte at rbx + 0xd0 with zero, checking another flag |
| 347 | `jne ba9` | jump to label ba9 if the flag from the previous line is non zero |
| 348 | `5aa: mov %r12,%rsi` | label 5aa, copy r12 into rsi |
| 349 | `call llama_sampler_apply@plt` | call llama_sampler_apply again |
| 350 | `5b2: mov 0x1f8(%rbx),%rdi` | label 5b2, load the value rbx + 0x1f8 into rdi |
| 351 | `mov %r12,%rsi` | copy r12 into rsi |
| 352 | `call llama_sampler_apply@plt` | call llama_sampler_apply again|
| 353 | `mov 0x260(%rbx),%rax` | load the 64-bit value at rbx + 0x260 into rax |
| 354 | `mov 0x250(%rbx),%r8` | load a pointer/value from the object at rbx + 0x250 into r8 |
| 355 | `cmpb $0x0,0x14(%rsp)` |  compare the byte at stack offset rsp+0x14 with 0 |
| 356 | `lea (%rax,%rax,2),%rax` | compute rax = rax + 2*rax = 3*rax |
| 357 | `lea (%r8,%rax,4),%rax` | compute rax = r8 + 4*(3*old_rax) effectively an indexed address calculation. |
| 358 | `mov (%rax),%ebp` | load the 32-bit value at that address into ebp |
| 359 | `jne 338` | if the byte checked at line 355 was not zero, jump back to 338 |
| 360 | `mov 0x1e8(%rbx),%rdi` | load another pointer/value from rbx + 0x1e8 into rdi (first argument register) |
| 361 | `test %rdi,%rdi` | test whether rdi is 0 |
| 362 | `je 338` | if it is 0, jump back to 338 |
| 363 | `mov 0x1f0(%rbx),%rax` | load another pointer/value from rbx + 0x1f0 |
| 364 | `test %rax,%rax` | check whether it is 0 |
| 365 | `je 60d` | if zero, jump to label 60d |
| 366 | `cmpb $0x0,0xd0(%rbx)` | check a byte flag at rbx + 0xd0 |
| 367 | `jne b8c` | if that flag is nonzero, jump to b8c |
| 368 | `60d: lea 0x24(%rsp),%rax` | set rax = rsp + 0x24; this is a pointer to a local stack structure |
| 369 | `lea 0x40(%rsp),%rsi` | set rsi = rsp + 0x40; another local structure/address |
| 370 | `mov %ebp,0x24(%rsp)` | store the token ID/value from ebp into the local structure |
| 371 | `movq $0x3f800000,0x28(%rsp)` | store 0x3f800000 at rsp+0x28. 0x3f800000 is the IEEE-754 representation of 1.0f |
| 372 | `movq $0x0,0x58(%rsp)` | initialize a local field to 0 |
| 373 | `movq $0x1,0x48(%rsp)` | initialize a local field to 1 |
| 374 | `movq $0xffffffffffffffff,0x50(%rsp)` | initialize a field to -1 |
| 375 | `mov %rax,0x40(%rsp)` | store the pointer to the local token structure into the structure at rsp+0x40 |
| 376 | `call llama_sampler_apply@plt` | call the llama.cpp sampler.This is the key operation in this section |
| 377 | `mov 0x40(%rsp),%rax` | reload the pointer/field modified or used by llama_sampler_apply |
| 378 | `movss std::_Sp_make_shared_tag::_S_ti()::__tag+0x750,%xmm0` | load a single-precision floating-point constant into xmm0 |
| 379 | `ucomiss 0x4(%rax),%xmm0` | compare the float at rax+4 with that constant |
| 380 | `jbe 338` | if the comparison is less than or equal,jump back to 338 |
| 381 | `mov (%rsp),%r13d` | load a 32-bit value from the stack into r13d.This is likely a sequence/token index |
| 382 | `mov %r15,%rdi` | place the r15 pointer into the first argument register |
| 383 | `mov %r13d,%esi` | place the index from r13d into the second argument register |
| 384 | `call llama_get_sampled_probs_ith@plt` | get the sampled probabilities for item/index r13d |
| 385 | `mov %r13d,%esi` | prepare the same index as the second argument |
| 386 | `mov %r15,%rdi` | restore the first argument |
| 387 | `mov %rax,%r14` | save the returned probability pointer in r14 |
| 388 | `call llama_get_sampled_logits_ith@plt` | get the sampled logits for the same item/index |
| 389 | `mov %r13d,%esi` | again prepare the index argument |
| 390 | `mov %r15,%rdi` | prepare the context/object pointer |
| 391 | `mov %r13d,(%rsp)` | save the index back onto the stack |
| 392 | `mov %rax,%rbp` | save the returned logits pointer in rbp |
| 393 | `call llama_get_sampled_candidates_ith@plt` | get the candidate tokens for this sampled item |
| 394 | `mov %r15,%rdi` | prepare the llama context/object as first argument |
| 395 | `mov %rax,%r13` | save the returned candidates pointer in r13 |
| 396 | `call llama_get_model@plt` | retriece the underlying llama model |
| 397 | `mov %rax,%rdi` | pass the model as the first argument |
| 398 | `call llama_model_get_vocab@plt` | retrieve the model's vocabulary object |
| 399 | `mov %rax,%rdi` | pass the vocabulary object to the next function |
| 400 | `call llama_vocab_n_tokens@plt` | get the number of tokens in the vocabulary |
| 401 | `mov %eax,0x14(%rsp)` | save the return value from llama_vocab_n_tokens() into a local stack variable.This is the vocabulary size |
| 402 | `mov (%rsp),%esi` | load the previously saved index into esi, 2nd function argument |
| 403 | `mov %r15,%rdi` | place the llama context/object pointer into rdi, 1st argument |
| 404 | `test %r14,%r14` | check whether the probability pointer saved in r14 is 0 |
| 405 | `je bd0` | If the probability pointer is 0, jump to bd0 |
| 406 | `call llama_get_sampled_probs_count_ith@plt` | get the number of sampled probabilities for this index.Return value is in eax |
| 407 | `mov 0x240(%rbx),%rdi` | load a pointer from rbx+0x240 into rdi |
| 408 | `mov %eax,%r10d` | copie the probability count into r10d |
| 409 | `mov 0x238(%rbx),%rdx` | load another pointer from rbx+0x238 into rdx |
| 410 | `movabs $0xaaaaaaaaaaaaaaab,%rsi` | load the constant 0xAAAAAAAAAAAAAAAB into rsi.This is a magic constant used for fast integer division |
| 411 | `mov %r10,(%rsp)` | save the sampled-probability count on the stack |
| 412 | `mov %r10,%r15` | copie the count into r15,r15 now represents the number of elements being processed |
| 413 | `mov %rdi,%rax` | copy the pointer from rdi to rax |
| 414 | `sub %rdx,%rax` | calculate rdi - rdx, i.e the distance between 2 memory pointers |
| 415 | `sar $0x2,%rax` | arithmetic shift right by 2, divides by 4 bytes.This converts a byte distance into a number of 4-byte elements |
| 416 | `imul %rsi,%rax` | multiply by the magic constant.This implements a compiler-optimized integer division |
| 417 | `mov %rax,%rcx` | copy the calculated value into rcx |
| 418 | `cmp %r10,%rax` | compare the calculated capacity/index against the sampled-probability count |
| 419 | `jb dfd` | If rax < r10,jump to dfd.Likely handles an insufficient-capacity/reallocation path |
| 420 | `cmp %rcx,%r10` | compare the count against the calculated value|
| 421 | `jae 726` | if r10 >= rcx,jump to 726 |
| 422 | `lea (%r10,%r10,2),%rax` | calculate rax = 3*r10 |
| 423 | `shl $0x2,%rax` | multiply by 4,rax=12*r20 |
| 424 | `lea (%rdx,%rax,1),%r8` | calculate r8 = rdx + 12*r10.This is an address for the element at index r10 |
| 425 | `cmp %r8,%rdi` | check whether this calculated address equals the current end pointer |
| 426 | `je 726` | if equal,jump to 726 |
| 427 | `sar $0x2,%rax` | divide the byte offset by 4 |
| 428 | `mov %r8,0x240(%rbx)` | update the pointer at rbx+0x240 to the newly calculated address |
| 429 | `imul %rsi,%rax` | again applies the magic multiplication/division optimization |
| 430 | `mov %rax,%rcx` | save the resulting calculated element count |
| 431 | `726: test %r15d,%r15d` | test whether the sampled-probability count is 0|
| 432 | `je 8c6` | if 0,jump to 8c6,nothing to process |
| 433 | `72f: cmp $0x4,%r15d` | compare the count with 4 |
| 434 | `jbe dc2` | if count less than or equal to 4 then jump to dc2 |
| 435 | `mov (%rsp),%rdi` | reload the sampled-probability count |
| 436 | `lea 0x4(%rdx),%rsi` | calculate rsi = rdx + 4, one 32-bit element past the start |
| 437 | `lea 0x0(,%rdi,4),%rax` | calculate rax = count * 4 |
| 438 | `lea (%rdi,%rdi,2),%rdi` | calculate rdi = 3 * count |
| 439 | `shl $0x2,%rdi` | multiply by 4 rdi = 12 * count |
| 440 | `lea 0x0(%rbp,%rax,1),%r8` | calculate r8 = rbp + count×4 |
| 441 | `lea (%rdx,%rdi,1),%r9` | calculate r9 = rdx + count×12 |
| 442 | `cmp %r8,%rsi` |  compare the beginning/next element address against the calculated array boundary |
| 443 | `setae %r8b` | set r8b = 1 if rsi >= r8,otherwise 0 |
| 444 | `cmp %r9,%rbp` | compare another pointer against the calculated candidate-array boundary |
| 445 | `setae %r10b` | set r10b = 1 if rbp >= r9 |
| 446 | `or %r10d,%r8d` | combine the 2 Boolean conditions |
| 447 | `lea (%r14,%rax,1),%r10` | calculate r10 = r14 + count×4.Yhis is likely the end of the probability array |
| 448 | `cmp %r10,%rsi` |  compare the relevant pointer against that probability-array boundary |
| 449 | `setae %sil` | set sil = 1 if rsi >=10 |
| 450 | `cmp %r9,%r14` | compare the probability-array pointer against the candidate-array boundary |
| 451 | `setae %r9b` | set r9b = 1 if r14 >= r9 |
| 452 | `or %r9d,%esi` | combine the 2 Boolean conditions |
| 453 | `test %sil,%r8b` | test the Boolean flags accumulated in sil and r8b |
| 454 | `je dc2` | if the relevant conditions are false,jump to dc2 |
| 455 | `lea -0x8(%rdx,%rdi,1),%rsi` | calculate rsi = rdx + rdi - 8.This establishes another memory boundary |
| 456 | `cmp %rsi,%r13` | compare the candidate pointer in r13 against the boundary |
| 457 | `setae %sil` | sil = 1 if r13 >= rsi,otherwise 0 |
| 458 | `add %r13,%rax` | add the candidaye pointer r13 to rax |
| 459 | `cmp %rax,%rdx` | compare the resulting address against rdx |
| 460 | `setae %al` | al = 1 if rdx >= rax,otherwise 0 |
| 461 | `or %al,%sil` | combines the 2 Boolean conditions |
| 462 | `je dc2` | if neither conditions is true,jump to dc2 |
| 463 | `mov %r15d,%esi` | copy the sampled-element count into esi |
| 464 | `mov %rdx,(%rsp)` | save rdx to the stack |
| 465 | `mov %rdx,%rax` | copy rdx into rax,rax becomes the destination/current pointer |
| 466 | `shr $0x2,%esi` | divide the element count by 4 |
| 467 | `shl $0x4,%rsi` | multiply by 16 |
| 468 | `mov %rsi,%rdi` | copy this calculated byte into rdi |
| 469 | `xor %esi,%esi` | set esi = 0.This becomes the loop offset |
| 470 | `mov %rdi,%rdx` | save the calculated size in rdx |
| 471 | `7c0: lea 0x0(%r13,%rsi,1),%rdi` | calculate rdi = r13 + rsi.This is the source address for the current iteration |
| 472 | `add $0x30,%rax` | advance destination pointer by 48 bytes |
| 473 | `mov 0xc(%rdi),%r8d` | load a 32-bit field at offset +12 |
| 474 | `mov 0x8(%rdi),%r9d` | load a 32-bit field at +8 |
| 475 | `mov 0x4(%rdi),%r10d` | load a 32-bit field at offset +4 |
| 476 | `mov (%rdi),%r11d` | load the first 32-bit field at offset +0 |
| 477 | `lea 0x0(%rbp,%rsi,1),%rdi` | calculate another source address rdi = rbp + offset |
| 478 | `movss 0xc(%rdi),%xmm0` | load a 32-bit float from offset +12 |
| 479 | `movss 0x8(%rdi),%xmm1` | load a float from offset +8 |
| 480 | `movss 0x4(%rdi),%xmm2` | load a float from offset +4 |
| 481 | `movss (%rdi),%xmm3` | load a float from offset +0 |
| 482 | `lea (%r14,%rsi,1),%rdi` | calculate another source address r14 + offset |
| 483 | `add $0x10,%rsi` |  advance the loop offset by 16 |
| 484 | `movss 0xc(%rdi),%xmm4` | load another float  |
| 485 | `movss 0x8(%rdi),%xmm5` | load another float |
| 486 | `movss 0x4(%rdi),%xmm6` | load another float |
| 487 | `movss (%rdi),%xmm7` | load another float |
| 488 | `mov %r11d,-0x30(%rax)` | store the first 32-bit field into the destination |
| 489 | `unpcklps %xmm5,%xmm1` |interleaves the low floats of xmm1 and xmm5 |
| 490 | `unpcklps %xmm4,%xmm0` | interleaves the low floats of xmm0 and xmm4 |
| 491 | `mov %r10d,-0x24(%rax)` | store another 32-bit field |
| 492 | `unpcklps %xmm7,%xmm3` | interleaves floats from xmm7 and xmm3 |
| 493 | `unpcklps %xmm6,%xmm2` | interleaves floats from xmm6 and xmm2 |
| 494 | `mov %r9d,-0x18(%rax)` | store another 32-bit field |
| 495 | `mov %r8d,-0xc(%rax)` | store the 4th 32-bit field |
| 496 | `movlps %xmm3,-0x2c(%rax)` | store 2 32-bit floats from xmm3 into the destination |
| 497 | `movlps %xmm2,-0x20(%rax)` | store 2 floats from xmm2 |
| 498 | `movlps %xmm1,-0x14(%rax)` | store 2 floats from xmm1 |
| 499 | `movlps %xmm0,-0x8(%rax)` | store 2 floats from xmm0 |
| 500 | `cmp %rsi,%rdx` | compare the current loop offset against the total size to determine whether the loop is finished|
| 501 | `jne 7c0` | |
| 502 | `mov %r15d,%eax` | |
| 503 | `mov (%rsp),%rdx` | |
| 504 | `and $0xfffffffc,%eax` | |
| 505 | `test $0x3,%r15b` | |
| 506 | `je 8c6` | |
| 507 | `mov %eax,%esi` | |
| 508 | `movss (%r14,%rsi,4),%xmm1` | |
| 509 | `movss 0x0(%rbp,%rsi,4),%xmm0` | |
| 510 | `lea (%rsi,%rsi,2),%rdi` | |
| 511 | `mov 0x0(%r13,%rsi,4),%r8d` | |
| 512 | `lea (%rdx,%rdi,4),%rdi` | |
| 513 | `lea 0x1(%rax),%esi` | |
| 514 | `unpcklps %xmm1,%xmm0` | |
| 515 | `mov %r8d,(%rdi)` | |
| 516 | `movlps %xmm0,0x4(%rdi)` | |
| 517 | `cmp %r15d,%esi` | |
| 518 | `jae 8c6` | |
| 519 | `movss (%r14,%rsi,4),%xmm1` | |
| 520 | `mov 0x0(%r13,%rsi,4),%r8d` | |
| 521 | `lea (%rsi,%rsi,2),%rdi` | |
| 522 | `add $0x2,%eax` | |
| 523 | `movss 0x0(%rbp,%rsi,4),%xmm0` | |
| 524 | `lea (%rdx,%rdi,4),%rdi` | |
| 525 | `mov %r8d,(%rdi)` | |
| 526 | `unpcklps %xmm1,%xmm0` | |
| 527 | `movlps %xmm0,0x4(%rdi)` | |
| 528 | `cmp %r15d,%eax` | |
| 529 | `jae 8c6` | |
| 530 | `mov 0x0(%r13,%rax,4),%edi` | |
| 531 | `lea (%rax,%rax,2),%rsi` | |
| 532 | `movss (%r14,%rax,4),%xmm1` | |
| 533 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 534 | `lea (%rdx,%rsi,4),%rsi` | |
| 535 | `mov %edi,(%rsi)` | |
| 536 | `unpcklps %xmm1,%xmm0` | |
| 537 | `movlps %xmm0,0x4(%rsi)` | |
| 538 | `8c6: mov %rdx,0x250(%rbx)` | |
| 539 | `mov 0x1f0(%rbx),%rdi` | |
| 540 | `mov %r12,%rsi` | |
| 541 | `mov %rcx,0x258(%rbx)` | |
| 542 | `movq $0xffffffffffffffff,0x260(%rbx)` | |
| 543 | `movb $0x0,0x268(%rbx)` | |
| 544 | `call llama_sampler_apply@plt` | |
| 545 | `mov 0x1e8(%rbx),%rdi` | |
| 546 | `test %rdi,%rdi` | |
| 547 | `je 922` | |
| 548 | `mov 0x1f0(%rbx),%rax` | |
| 549 | `test %rax,%rax` | |
| 550 | `je 91a` | |
| 551 | `cmpb $0x0,0xd0(%rbx)` | |
| 552 | `jne e68` | |
| 553 | `91a: mov %r12,%rsi` | |
| 554 | `call llama_sampler_apply@plt` | |
| 555 | `922: mov 0x1f8(%rbx),%rdi` | |
| 556 | `mov %r12,%rsi` | |
| 557 | `call llama_sampler_apply@plt` | |
| 558 | `mov 0x260(%rbx),%rax` | |
| 559 | `cmp $0xffffffffffffffff,%rax` | |
| 560 | `je fc7` | |
| 561 | `mov 0x250(%rbx),%rdx` | |
| 562 | `lea (%rax,%rax,2),%rax` | |
| 563 | `lea (%rdx,%rax,4),%rax` | |
| 564 | `mov (%rax),%ebp` | |
| 565 | `jmp 338` | |
| 566 | `nop` | |
| 567 | `960: mov %rdx,%rcx` | |
| 568 | `xor %eax,%eax` | |
| 569 | `nop` | |
| 570 | `968: movss 0x0(%r13,%rax,4),%xmm1` | |
| 571 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 572 | `add $0xc,%rcx` | |
| 573 | `mov (%r12,%rax,4),%edi` | |
| 574 | `add $0x1,%rax` | |
| 575 | `unpcklps %xmm1,%xmm0` | |
| 576 | `mov %edi,-0xc(%rcx)` | |
| 577 | `movlps %xmm0,-0x8(%rcx)` | |
| 578 | `cmp %r14d,%eax` | |
| 579 | `jb 968` | |
| 580 | `jmp 2a0` | |
| 581 | `nop` | |
| 582 | `998: call common_log_main()@plt` | |
| 583 | `mov %rax,%rdi` | |
| 584 | `mov %ebp,%r8d` | |
| 585 | `xor %eax,%eax` | |
| 586 | `mov $0x1,%esi` | |
| 587 | `lea _fini+0x6e68,%rcx` | |
| 588 | `lea _fini+0x192d0,%rdx` | |
| 589 | `call common_log_add(common_log*, ggml_log_level, char const*, ...)@plt` | |
| 590 | `jmp 2e4` | |
| 591 | `nop` | |
| 592 | `9c8: lea (%r11,%r11,2),%rax` | |
| 593 | `shl $0x2,%rax` | |
| 594 | `lea (%rdx,%rax,1),%r8` | |
| 595 | `cmp %r8,%rcx` | |
| 596 | `je f7` | |
| 597 | `sar $0x2,%rax` | |
| 598 | `mov %r8,0x240(%rbx)` | |
| 599 | `imul %rdi,%rax` | |
| 600 | `mov %rax,%rsi` | |
| 601 | `jmp f7` | |
| 602 | `nop` | |
| 603 | `9f8: mov %r11,%rsi` | |
| 604 | `lea 0x238(%rbx),%rdi` | |
| 605 | `sub %rax,%rsi` | |
| 606 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 607 | `mov 0x238(%rbx),%rdx` | |
| 608 | `mov 0x240(%rbx),%rax` | |
| 609 | `movabs $0xaaaaaaaaaaaaaaab,%rcx` | |
| 610 | `sub %rdx,%rax` | |
| 611 | `sar $0x2,%rax` | |
| 612 | `imul %rcx,%rax` | |
| 613 | `mov %rax,%rsi` | |
| 614 | `jmp 100` | |
| 615 | `nop` | |
| 616 | `a38: mov %rdx,%rcx` | |
| 617 | `xor %eax,%eax` | |
| 618 | `nop` | |
| 619 | `a40: mov (%r12,%rax,4),%edi` | |
| 620 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 621 | `add $0x1,%rax` | |
| 622 | `movl $0x0,0x8(%rcx)` | |
| 623 | `add $0xc,%rcx` | |
| 624 | `mov %edi,-0xc(%rcx)` | |
| 625 | `movss %xmm0,-0x8(%rcx)` | |
| 626 | `cmp %r13d,%eax` | |
| 627 | `jb a40` | |
| 628 | `jmp 2a0` | |
| 629 | `nop` | |
| 630 | `a70: call llama_get_logits_ith@plt` | |
| 631 | `mov %rax,%rbp` | |
| 632 | `test %rax,%rax` | |
| 633 | `je f96` | |
| 634 | `mov 0x240(%rbx),%rsi` | |
| 635 | `mov 0x238(%rbx),%rdx` | |
| 636 | `movslq %r14d,%r12` | |
| 637 | `movabs $0xaaaaaaaaaaaaaaab,%rcx` | |
| 638 | `mov %rsi,%rax` | |
| 639 | `sub %rdx,%rax` | |
| 640 | `sar $0x2,%rax` | |
| 641 | `imul %rcx,%rax` | |
| 642 | `cmp %r12,%rax` | |
| 643 | `jb b67` | |
| 644 | `cmp %rax,%r12` | |
| 645 | `jae acf` | |
| 646 | `lea (%r12,%r12,2),%rax` | |
| 647 | `lea (%rdx,%rax,4),%rax` | |
| 648 | `cmp %rax,%rsi` | |
| 649 | `je acc` | |
| 650 | `mov %rax,0x240(%rbx)` | |
| 651 | `acc: mov %rax,%rsi` | |
| 652 | `acf: mov %rdx,%rcx` | |
| 653 | `xor %eax,%eax` | |
| 654 | `test %r14d,%r14d` | |
| 655 | `jle b01` | |
| 656 | `nop` | |
| 657 | `ae0: movss 0x0(%rbp,%rax,4),%xmm0` | |
| 658 | `mov %eax,(%rcx)` | |
| 659 | `add $0x1,%rax` | |
| 660 | `add $0xc,%rcx` | |
| 661 | `movl $0x0,-0x4(%rcx)` | |
| 662 | `movss %xmm0,-0x8(%rcx)` | |
| 663 | `cmp %rax,%r12` | |
| 664 | `jne ae0` | |
| 665 | `b01: movabs $0xaaaaaaaaaaaaaaab,%rax` | |
| 666 | `sub %rdx,%rsi` | |
| 667 | `sar $0x2,%rsi` | |
| 668 | `imul %rax,%rsi` | |
| 669 | `jmp 2a0` | |
| 670 | `nop` | |
| 671 | `b20: mov %r14,%rsi` | |
| 672 | `lea 0x238(%rbx),%rdi` | |
| 673 | `sub %rax,%rsi` | |
| 674 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 675 | `mov 0x238(%rbx),%rdx` | |
| 676 | `mov 0x240(%rbx),%rax` | |
| 677 | `movabs $0xaaaaaaaaaaaaaaab,%rcx` | |
| 678 | `sub %rdx,%rax` | |
| 679 | `sar $0x2,%rax` | |
| 680 | `imul %rcx,%rax` | |
| 681 | `mov %rax,%rsi` | |
| 682 | `cmp $0x8,%r13d` | |
| 683 | `ja 3f1` | |
| 684 | `jmp a38` | |
| 685 | `b67: mov %r12,%rsi` | |
| 686 | `lea 0x238(%rbx),%rdi` | |
| 687 | `sub %rax,%rsi` | |
| 688 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 689 | `mov 0x238(%rbx),%rdx` | |
| 690 | `mov 0x240(%rbx),%rsi` | |
| 691 | `jmp acf` | |
| 692 | `b8c: mov %rax,%rdi` | |
| 693 | `call common_reasoning_budget_get_state(llama_sampler const*)@plt` | |
| 694 | `and $0xfffffffb,%eax` | |
| 695 | `jne 338` | |
| 696 | `mov 0x1e8(%rbx),%rdi` | |
| 697 | `jmp 60d` | |
| 698 | `ba9: mov %rax,%rdi` | |
| 699 | `call common_reasoning_budget_get_state(llama_sampler const*)@plt` | |
| 700 | `and $0xfffffffb,%eax` | |
| 701 | `jne 5b2` | |
| 702 | `mov 0x1e8(%rbx),%rdi` | |
| 703 | `jmp 5aa` | |
| 704 | `cs nopw 0x0(%rax,%rax,1)` | |
| 705 | `bd0: test %rbp,%rbp` | |
| 706 | `je e85` | |
| 707 | `call llama_get_sampled_logits_count_ith@plt` | |
| 708 | `mov 0x240(%rbx),%rdi` | |
| 709 | `mov %eax,%r14d` | |
| 710 | `mov 0x238(%rbx),%rdx` | |
| 711 | `movabs $0xaaaaaaaaaaaaaaab,%rsi` | |
| 712 | `mov %r14,%r15` | |
| 713 | `mov %rdi,%rax` | |
| 714 | `sub %rdx,%rax` | |
| 715 | `sar $0x2,%rax` | |
| 716 | `imul %rsi,%rax` | |
| 717 | `mov %rax,%rcx` | |
| 718 | `cmp %r14,%rax` | |
| 719 | `jb f32` | |
| 720 | `cmp %rax,%r14` | |
| 721 | `jae c3e` | |
| 722 | `lea (%r14,%r14,2),%rax` | |
| 723 | `shl $0x2,%rax` | |
| 724 | `lea (%rdx,%rax,1),%r8` | |
| 725 | `cmp %r8,%rdi` | |
| 726 | `je c3e` | |
| 727 | `sar $0x2,%rax` | |
| 728 | `mov %r8,0x240(%rbx)` | |
| 729 | `imul %rsi,%rax` | |
| 730 | `mov %rax,%rcx` | |
| 731 | `c3e: test %r15d,%r15d` | |
| 732 | `je 8c6` | |
| 733 | `c47: cmp $0x8,%r15d` | |
| 734 | `jbe e37` | |
| 735 | `lea 0x0(,%r14,4),%rax` | |
| 736 | `lea (%r14,%r14,2),%rsi` | |
| 737 | `shl $0x2,%rsi` | |
| 738 | `lea 0x4(%rdx),%r8` | |
| 739 | `lea 0x0(%rbp,%rax,1),%rdi` | |
| 740 | `cmp %rdi,%r8` | |
| 741 | `lea (%rdx,%rsi,1),%r8` | |
| 742 | `setae %dil` | |
| 743 | `cmp %r8,%rbp` | |
| 744 | `setae %r8b` | |
| 745 | `or %r8b,%dil` | |
| 746 | `je e37` | |
| 747 | `lea -0x8(%rdx,%rsi,1),%rsi` | |
| 748 | `cmp %rsi,%r13` | |
| 749 | `setae %sil` | |
| 750 | `add %r13,%rax` | |
| 751 | `cmp %rax,%rdx` | |
| 752 | `setae %al` | |
| 753 | `or %al,%sil` | |
| 754 | `je e37` | |
| 755 | `mov %r15d,%esi` | |
| 756 | `mov %rdx,%rax` | |
| 757 | `shr $0x2,%esi` | |
| 758 | `shl $0x4,%rsi` | |
| 759 | `mov %rsi,%r14` | |
| 760 | `xor %esi,%esi` | |
| 761 | `cb5: lea 0x0(%r13,%rsi,1),%rdi` | |
| 762 | `add $0x30,%rax` | |
| 763 | `mov 0xc(%rdi),%r8d` | |
| 764 | `mov 0x8(%rdi),%r9d` | |
| 765 | `mov 0x4(%rdi),%r10d` | |
| 766 | `mov (%rdi),%r11d` | |
| 767 | `lea 0x0(%rbp,%rsi,1),%rdi` | |
| 768 | `add $0x10,%rsi` | |
| 769 | `movss 0x8(%rdi),%xmm1` | |
| 770 | `movss 0x4(%rdi),%xmm2` | |
| 771 | `movss (%rdi),%xmm3` | |
| 772 | `movss 0xc(%rdi),%xmm0` | |
| 773 | `mov %r11d,-0x30(%rax)` | |
| 774 | `mov %r10d,-0x24(%rax)` | |
| 775 | `mov %r9d,-0x18(%rax)` | |
| 776 | `mov %r8d,-0xc(%rax)` | |
| 777 | `movss %xmm3,-0x2c(%rax)` | |
| 778 | `movss %xmm2,-0x20(%rax)` | |
| 779 | `movss %xmm1,-0x14(%rax)` | |
| 780 | `movss %xmm0,-0x8(%rax)` | |
| 781 | `movl $0x0,-0x28(%rax)` | |
| 782 | `movl $0x0,-0x1c(%rax)` | |
| 783 | `movl $0x0,-0x10(%rax)` | |
| 784 | `movl $0x0,-0x4(%rax)` | |
| 785 | `cmp %rsi,%r14` | |
| 786 | `jne cb5` | |
| 787 | `mov %r15d,%eax` | |
| 788 | `and $0xfffffffc,%eax` | |
| 789 | `test $0x3,%r15b` | |
| 790 | `je 8c6` | |
| 791 | `mov %eax,%esi` | |
| 792 | `mov 0x0(%r13,%rsi,4),%r8d` | |
| 793 | `movss 0x0(%rbp,%rsi,4),%xmm0` | |
| 794 | `lea (%rsi,%rsi,2),%rdi` | |
| 795 | `lea 0x1(%rax),%esi` | |
| 796 | `lea (%rdx,%rdi,4),%rdi` | |
| 797 | `mov %r8d,(%rdi)` | |
| 798 | `movl $0x0,0x8(%rdi)` | |
| 799 | `movss %xmm0,0x4(%rdi)` | |
| 800 | `cmp %r15d,%esi` | |
| 801 | `jae 8c6` | |
| 802 | `mov 0x0(%r13,%rsi,4),%r8d` | |
| 803 | `movss 0x0(%rbp,%rsi,4),%xmm0` | |
| 804 | `lea (%rsi,%rsi,2),%rdi` | |
| 805 | `add $0x2,%eax` | |
| 806 | `lea (%rdx,%rdi,4),%rdi` | |
| 807 | `mov %r8d,(%rdi)` | |
| 808 | `movl $0x0,0x8(%rdi)` | |
| 809 | `movss %xmm0,0x4(%rdi)` | |
| 810 | `cmp %r15d,%eax` | |
| 811 | `jae 8c6` | |
| 812 | `mov 0x0(%r13,%rax,4),%edi` | |
| 813 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 814 | `lea (%rax,%rax,2),%rsi` | |
| 815 | `lea (%rdx,%rsi,4),%rsi` | |
| 816 | `mov %edi,(%rsi)` | |
| 817 | `movl $0x0,0x8(%rsi)` | |
| 818 | `movss %xmm0,0x4(%rsi)` | |
| 819 | `jmp 8c6` | |
| 820 | `dc2: mov %rdx,%rsi` | |
| 821 | `xor %eax,%eax` | |
| 822 | `nop` | |
| 823 | `dd0: movss (%r14,%rax,4),%xmm1` | |
| 824 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 825 | `add $0xc,%rsi` | |
| 826 | `mov 0x0(%r13,%rax,4),%edi` | |
| 827 | `add $0x1,%rax` | |
| 828 | `unpcklps %xmm1,%xmm0` | |
| 829 | `mov %edi,-0xc(%rsi)` | |
| 830 | `movlps %xmm0,-0x8(%rsi)` | |
| 831 | `cmp %r15d,%eax` | |
| 832 | `jb dd0` | |
| 833 | `jmp 8c6` | |
| 834 | `dfd: mov %r10,%rsi` | |
| 835 | `lea 0x238(%rbx),%rdi` | |
| 836 | `sub %rax,%rsi` | |
| 837 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 838 | `mov 0x238(%rbx),%rdx` | |
| 839 | `mov 0x240(%rbx),%rcx` | |
| 840 | `movabs $0xaaaaaaaaaaaaaaab,%rax` | |
| 841 | `sub %rdx,%rcx` | |
| 842 | `sar $0x2,%rcx` | |
| 843 | `imul %rax,%rcx` | |
| 844 | `jmp 72f` | |
| 845 | `e37: mov %rdx,%rsi` | |
| 846 | `xor %eax,%eax` | |
| 847 | `e3c: mov 0x0(%r13,%rax,4),%edi` | |
| 848 | `movss 0x0(%rbp,%rax,4),%xmm0` | |
| 849 | `add $0x1,%rax` | |
| 850 | `movl $0x0,0x8(%rsi)` | |
| 851 | `add $0xc,%rsi` | |
| 852 | `mov %edi,-0xc(%rsi)` | |
| 853 | `movss %xmm0,-0x8(%rsi)` | |
| 854 | `cmp %r15d,%eax` | |
| 855 | `jb e3c` | |
| 856 | `jmp 8c6` | |
| 857 | `e68: mov %rax,%rdi` | |
| 858 | `call common_reasoning_budget_get_state(llama_sampler const*)@plt` | |
| 859 | `and $0xfffffffb,%eax` | |
| 860 | `jne 922` | |
| 861 | `mov 0x1e8(%rbx),%rdi` | |
| 862 | `jmp 91a` | |
| 863 | `e85: call llama_get_logits_ith@plt` | |
| 864 | `mov %rax,%r14` | |
| 865 | `test %rax,%rax` | |
| 866 | `je ff8` | |
| 867 | `mov 0x240(%rbx),%rsi` | |
| 868 | `mov 0x238(%rbx),%rdx` | |
| 869 | `movabs $0xaaaaaaaaaaaaaaab,%rcx` | |
| 870 | `movslq 0x14(%rsp),%r13` | |
| 871 | `mov %rsi,%rax` | |
| 872 | `sub %rdx,%rax` | |
| 873 | `sar $0x2,%rax` | |
| 874 | `imul %rcx,%rax` | |
| 875 | `cmp %r13,%rax` | |
| 876 | `jb f6c` | |
| 877 | `cmp %rax,%r13` | |
| 878 | `jae ee7` | |
| 879 | `lea 0x0(%r13,%r13,2),%rax` | |
| 880 | `lea (%rdx,%rax,4),%rax` | |
| 881 | `cmp %rax,%rsi` | |
| 882 | `je ee4` | |
| 883 | `mov %rax,0x240(%rbx)` | |
| 884 | `ee4: mov %rax,%rsi` | |
| 885 | `ee7: mov 0x14(%rsp),%edi` | |
| 886 | `mov %rdx,%rcx` | |
| 887 | `xor %eax,%eax` | |
| 888 | `test %edi,%edi` | |
| 889 | `jle f15` | |
| 890 | `ef4: movss (%r14,%rax,4),%xmm0` | |
| 891 | `mov %eax,(%rcx)` | |
| 892 | `add $0x1,%rax` | |
| 893 | `add $0xc,%rcx` | |
| 894 | `movl $0x0,-0x4(%rcx)` | |
| 895 | `movss %xmm0,-0x8(%rcx)` | |
| 896 | `cmp %rax,%r13` | |
| 897 | `jne ef4` | |
| 898 | `f15: movabs $0xaaaaaaaaaaaaaaab,%rax` | |
| 899 | `sub %rdx,%rsi` | |
| 900 | `mov %rsi,%rcx` | |
| 901 | `sar $0x2,%rcx` | |
| 902 | `imul %rax,%rcx` | |
| 903 | `jmp 8c6` | |
| 904 | `f32: mov %r14,%rsi` | |
| 905 | `lea 0x238(%rbx),%rdi` | |
| 906 | `sub %rax,%rsi` | |
| 907 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 908 | `mov 0x238(%rbx),%rdx` | |
| 909 | `mov 0x240(%rbx),%rcx` | |
| 910 | `movabs $0xaaaaaaaaaaaaaaab,%rax` | |
| 911 | `sub %rdx,%rcx` | |
| 912 | `sar $0x2,%rcx` | |
| 913 | `imul %rax,%rcx` | |
| 914 | `jmp c47` | |
| 915 | `f6c: mov %r13,%rsi` | |
| 916 | `lea 0x238(%rbx),%rdi` | |
| 917 | `sub %rax,%rsi` | |
| 918 | `call std::vector<llama_token_data, std::allocator<llama_token_data> >::_M_default_append(unsigned` | |
| 919 | `mov 0x238(%rbx),%rdx` | |
| 920 | `mov 0x240(%rbx),%rsi` | |
| 921 | `jmp ee7` | |
| 922 | `f91: call __stack_chk_fail@plt` | |
| 923 | `f96: mov 0x68(%rsp),%rax` | |
| 924 | `sub %fs:0x28,%rax` | |
| 925 | `jne f91` | |
| 926 | `lea _fini+0x6e56,%rcx` | |
| 927 | `lea _fini+0x14d1,%rdx` | |
| 928 | `mov $0x9a,%esi` | |
| 929 | `xor %eax,%eax` | |
| 930 | `lea _fini+0x19290,%rdi` | |
| 931 | `call ggml_abort@plt` | |
| 932 | `fc7: mov 0x68(%rsp),%rax` | |
| 933 | `sub %fs:0x28,%rax` | |
| 934 | `jne f91` | |
| 935 | `lea _fini+0x193e0,%rcx` | |
| 936 | `lea _fini+0x14d1,%rdx` | |
| 937 | `mov $0x29f,%esi` | |
| 938 | `xor %eax,%eax` | |
| 939 | `lea _fini+0x19290,%rdi` | |
| 940 | `call ggml_abort@plt` | |
| 941 | `ff8: mov 0x68(%rsp),%rax` | |
| 942 | `sub %fs:0x28,%rax` | |
| 943 | `jne f91` | |
| 944 | `\| lea _fini+0x6e56,%rcx` | |
| 945 | `lea _fini+0x14d1,%rdx` | |
| 946 | `mov $0x9a,%esi` | |
| 947 | `xor %eax,%eax` | |
| 948 | `lea _fini+0x19290,%rdi` | |
| 949 | `call ggml_abort@plt` | |
| 950 | `1029: mov 0x68(%rsp),%rax` | |
| 951 | `sub %fs:0x28,%rax` | |
| 952 | `jne f91` | |
| 953 | `lea _fini+0x19378,%rcx` | |
| 954 | `lea _fini+0x14d1,%rdx` | |
| 955 | `mov $0x26a,%esi` | |
| 956 | `xor %eax,%eax` | |
| 957 | `lea _fini+0x19290,%rdi` | |
| 958 | `call ggml_abort@plt` | |
| 959 | `105e: mov 0x68(%rsp),%rax` | |
| 960 | `sub %fs:0x28,%rax` | |
| 961 | `jne f91` | |
| 962 | `lea _fini+0x19320,%rcx` | |
| 963 | `lea _fini+0x14d1,%rdx` | |
| 964 | `mov $0x269,%esi` | |
| 965 | `xor %eax,%eax` | |
| 966 | `lea _fini+0x19290,%rdi` | |
| 967 | `call ggml_abort@plt` | |
| 968 | `endbr64 \` | |
| 969 | `mov %rax,%rbx` | |
| 970 | `\| jmp e9e0a <common_sampler_sample(common_sampler*, llama_context*, int, bool) [clone .cold]>` | |

## Notes

- `perf annotate` decorations such as `→`, `↓`, `↑`, `▒`, and `◆` were removed.
- Sampling percentages were removed from the assembly listing.
- Labels such as `198:`, `2a0:`, `ae0:`, etc. are preserved because they identify jump/loop targets.
- The assembly syntax is kept in the AT&T format produced by `perf`.

        
        
