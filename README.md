# NASSCOM-RISC-V
NASSCOM RISC-V Based MYTH Repo [16 April - 25 April 2025]

# Day 1
<details>
<summary>Introductio to RISC-V ISA and GNU compiler toolchain</summary>

Explanation of RV64I (base integer instruction), RV64M (multiply extension), RV64F (floatingpoint extension), RV64D (double precision extension), ABI (application binary interface), memory allocation & stack pointer

create simple C-program sum1ton.c, compile and run.

```c
#include <stdio.h>

int main() {
	int i, sum = 0, n = 5;
	for (i=1; i <= n; ++i) {
		sum += i;
	}
	printf("Sum of numbers form 1 to %d is %d\n", n, sum);
	return 0;
}
```

![image](https://github.com/user-attachments/assets/0e29972f-61cf-4276-8a7f-1918d141a789)

now compile C-program with risc-V C-Compiler using optimizasion level -O1 creating an objectfile called sum1ton.o

```sh
riscv64-unknown-elf-gcc -O1 -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```
![image](https://github.com/user-attachments/assets/e916bd58-69a2-4472-a806-90e134419113)

now look into objectfile sum1ton.o with and check RISC-V asm-code of mail() module using following command:

```sh
riscv64-unknown-elf-objdump -d sum1ton.o | more
```

![image](https://github.com/user-attachments/assets/97e8f20f-36ad-4fc9-acbd-53267d35739b)

Output:
assembly code with optimization level -O1

```s
0000000000010184 <main>:
   10184:	ff010113          	addi	sp,sp,-16
   10188:	00113423          	sd	ra,8(sp)
   1018c:	06400793          	li	a5,100
   10190:	fff7879b          	addiw	a5,a5,-1
   10194:	fe079ee3          	bnez	a5,10190 <main+0xc>
   10198:	00001637          	lui	a2,0x1
   1019c:	3ba60613          	addi	a2,a2,954 # 13ba <register_fini-0xecf6>
   101a0:	06400593          	li	a1,100
   101a4:	00021537          	lui	a0,0x21
   101a8:	19050513          	addi	a0,a0,400 # 21190 <__clzdi2+0x48>
   101ac:	26c000ef          	jal	ra,10418 <printf>
   101b0:	00000513          	li	a0,0
   101b4:	00813083          	ld	ra,8(sp)
   101b8:	01010113          	addi	sp,sp,16
   101bc:	00008067          	ret

```

now change optimization level to -oFast

```sh
riscv64-unknown-elf-gcc -Ofast -mabi=lp64 -march=rv64i -o sum1ton.o sum1ton.c
```
![image](https://github.com/user-attachments/assets/552ce222-bcc6-4fe4-b508-42839725dc29)

and check now asm-Code for difference.

```s
00000000000100b0 <main>:
   100b0:	00001637          	lui	a2,0x1
   100b4:	00021537          	lui	a0,0x21
   100b8:	ff010113          	addi	sp,sp,-16
   100bc:	3ba60613          	addi	a2,a2,954 # 13ba <main-0xecf6>
   100c0:	06400593          	li	a1,100
   100c4:	18050513          	addi	a0,a0,384 # 21180 <__clzdi2+0x44>
   100c8:	00113423          	sd	ra,8(sp)
   100cc:	340000ef          	jal	ra,1040c <printf>
   100d0:	00813083          	ld	ra,8(sp)
   100d4:	00000513          	li	a0,0
   100d8:	01010113          	addi	sp,sp,16
   100dc:	00008067          	ret
```

what we see is that with optimization level -ofast we get a different asm-code with fewer number of code lines!

next lab is to single step through asm-code of main() with "spike" RISC-V debugger.

first run program im RISC-V emulator "spike" and validate output.

```sh
spike pk sum1ton.o
```

![image](https://github.com/user-attachments/assets/b19b54ff-de2e-496d-80ff-1b0b8de90e5b)

to debug main() function in RISC-V world use again spike with folowwing syntax:

```sh
vsduser@vsduser-VirtualBox:~/Day_1$ spike -d pk sum1ton.o
(spike) until pc 0 100b0
bbl loader
(spike) reg 0 a2
0x0000000000000000
(spike) reg 0 a0
0x0000000000000001
(spike) reg 0 sp
0x0000003ffffffb50
(spike) 
core   0: 0x00000000000100b0 (0x00001637) lui     a2, 0x1
(spike) reg 0 a2
0x0000000000001000
(spike) 
core   0: 0x00000000000100b4 (0x00021537) lui     a0, 0x21
(spike) reg 0 a0
0x0000000000021000
(spike) reg 0 sp
0x0000003ffffffb50
(spike) 
core   0: 0x00000000000100b8 (0xff010113) addi    sp, sp, -16
(spike) reg 0 sp
0x0000003ffffffb40
(spike) 
```

we run code unti pc (program counter) "100b0" which is starting point of main() function an single step through the next instructions

![image](https://github.com/user-attachments/assets/a3be1886-0db2-45ad-bd7c-e39ff3da28d1)

first instruction is "lui a2, 0x1" - load immediate register a2 with hex 01.

![image](https://github.com/user-attachments/assets/4d56c66f-df6c-4448-a913-dcd7d1f1eca5)

the instruction load 0x01 into bit [31-12] of reg a2 show in above debug session.
```sh
(spike) 
core   0: 0x00000000000100b0 (0x00001637) lui     a2, 0x1
(spike) reg 0 a2
0x0000000000001000
(spike)
```

next instruction is "lui a0, 0x21" where register a0 is loaded with hex 21 follw same rule for reg a0.

next is "addi sp, sp, -16" wich mean that sp (stack pointer) will be subtracted be 16, hex 10.

![image](https://github.com/user-attachments/assets/fcb68874-2c46-4f30-8bb3-4b2eaf437dde)

</details>

# Day 2

<details>
<summary>Introduction to ABI and basic verification flow</summary>
  
</details>

# Day 3

<details>
<summary>Introductio to TL-Verilog and Makerchip</summary>
  
</details>

# Day 4

<details>
<summary>Basic RISC-V CPU microarchitectur/summary>
  
</details>

# Day 5

<details>
<summary>Compete pipliened RISC-V CPU micro architectur</summary>
  
</details>
