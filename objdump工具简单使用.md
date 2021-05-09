objdump是一个在linux下用于分析ELF文件的工具，现在列一下常用的功能。

直接输入objdump输出帮助信息：

```text
用法：objdump <选项> <文件>
 显示来自目标 <文件> 的信息。
 至少必须给出以下选项之一：
  -a, --archive-headers    Display archive header information
  -f, --file-headers       Display the contents of the overall file header
  -p, --private-headers    Display object format specific file header contents
  -P, --private=OPT,OPT... Display object format specific contents
  -h, --[section-]headers  Display the contents of the section headers
  -x, --all-headers        Display the contents of all headers
  -d, --disassemble        Display assembler contents of executable sections
  -D, --disassemble-all    Display assembler contents of all sections
      --disassemble=<sym>  Display assembler contents from <sym>
  -S, --source             Intermix source code with disassembly
  -s, --full-contents      Display the full contents of all sections requested
  -g, --debugging          Display debug information in object file
  -e, --debugging-tags     Display debug information using ctags style
  -G, --stabs              Display (in raw form) any STABS info in the file
  -W[lLiaprmfFsoRtUuTgAckK] or
  --dwarf[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,
          =frames-interp,=str,=loc,=Ranges,=pubtypes,
          =gdb_index,=trace_info,=trace_abbrev,=trace_aranges,
          =addr,=cu_index,=links,=follow-links]
                           Display DWARF info in the file
  -t, --syms               Display the contents of the symbol table(s)
  -T, --dynamic-syms       Display the contents of the dynamic symbol table
  -r, --reloc              Display the relocation entries in the file
  -R, --dynamic-reloc      Display the dynamic relocation entries in the file
  @<file>                  Read options from <file>
  -v, --version            Display this program's version number
  -i, --info               List object formats and architectures supported
  -H, --help               Display this information
```

接下来会使用一个简单的函数作为分析内容：

main.c:

```c
#include <stdio.h>

int add(int a, int b)
{
    return a + b;
}

int main(int argc, char** argv)
{
    printf("hello world %d\n", add(5, 6));
    return 0;
}

```

使用gcc编译：

```bash
gcc -Og -o main main.c
```

## 打印头部信息

```bash
objdump -a main
```

输出：

```text

main：     文件格式 elf64-x86-64
main


```

## 打印文件头

```bash
objdump -f main
```

输出：

```text

main：     文件格式 elf64-x86-64
体系结构：i386:x86-64，标志 0x00000150：
HAS_SYMS, DYNAMIC, D_PAGED
起始地址 0x0000000000001040


```

## 打印反汇编

```bash
objdump -S main
```

输出

```s

main：     文件格式 elf64-x86-64


Disassembly of section .init:

0000000000001000 <_init>:
    1000:	f3 0f 1e fa          	endbr64 
    1004:	48 83 ec 08          	sub    $0x8,%rsp
    1008:	48 8b 05 d9 2f 00 00 	mov    0x2fd9(%rip),%rax        # 3fe8 <__gmon_start__>
    100f:	48 85 c0             	test   %rax,%rax
    1012:	74 02                	je     1016 <_init+0x16>
    1014:	ff d0                	callq  *%rax
    1016:	48 83 c4 08          	add    $0x8,%rsp
    101a:	c3                   	retq   

Disassembly of section .plt:

0000000000001020 <.plt>:
    1020:	ff 35 e2 2f 00 00    	pushq  0x2fe2(%rip)        # 4008 <_GLOBAL_OFFSET_TABLE_+0x8>
    1026:	ff 25 e4 2f 00 00    	jmpq   *0x2fe4(%rip)        # 4010 <_GLOBAL_OFFSET_TABLE_+0x10>
    102c:	0f 1f 40 00          	nopl   0x0(%rax)

0000000000001030 <printf@plt>:
    1030:	ff 25 e2 2f 00 00    	jmpq   *0x2fe2(%rip)        # 4018 <printf@GLIBC_2.2.5>
    1036:	68 00 00 00 00       	pushq  $0x0
    103b:	e9 e0 ff ff ff       	jmpq   1020 <.plt>

Disassembly of section .text:

0000000000001040 <_start>:
    1040:	f3 0f 1e fa          	endbr64 
    1044:	31 ed                	xor    %ebp,%ebp
    1046:	49 89 d1             	mov    %rdx,%r9
    1049:	5e                   	pop    %rsi
    104a:	48 89 e2             	mov    %rsp,%rdx
    104d:	48 83 e4 f0          	and    $0xfffffffffffffff0,%rsp
    1051:	50                   	push   %rax
    1052:	54                   	push   %rsp
    1053:	4c 8d 05 86 01 00 00 	lea    0x186(%rip),%r8        # 11e0 <__libc_csu_fini>
    105a:	48 8d 0d 0f 01 00 00 	lea    0x10f(%rip),%rcx        # 1170 <__libc_csu_init>
    1061:	48 8d 3d d5 00 00 00 	lea    0xd5(%rip),%rdi        # 113d <main>
    1068:	ff 15 72 2f 00 00    	callq  *0x2f72(%rip)        # 3fe0 <__libc_start_main@GLIBC_2.2.5>
    106e:	f4                   	hlt    
    106f:	90                   	nop

0000000000001070 <deregister_tm_clones>:
    1070:	48 8d 3d b9 2f 00 00 	lea    0x2fb9(%rip),%rdi        # 4030 <__TMC_END__>
    1077:	48 8d 05 b2 2f 00 00 	lea    0x2fb2(%rip),%rax        # 4030 <__TMC_END__>
    107e:	48 39 f8             	cmp    %rdi,%rax
    1081:	74 15                	je     1098 <deregister_tm_clones+0x28>
    1083:	48 8b 05 4e 2f 00 00 	mov    0x2f4e(%rip),%rax        # 3fd8 <_ITM_deregisterTMCloneTable>
    108a:	48 85 c0             	test   %rax,%rax
    108d:	74 09                	je     1098 <deregister_tm_clones+0x28>
    108f:	ff e0                	jmpq   *%rax
    1091:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)
    1098:	c3                   	retq   
    1099:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)

00000000000010a0 <register_tm_clones>:
    10a0:	48 8d 3d 89 2f 00 00 	lea    0x2f89(%rip),%rdi        # 4030 <__TMC_END__>
    10a7:	48 8d 35 82 2f 00 00 	lea    0x2f82(%rip),%rsi        # 4030 <__TMC_END__>
    10ae:	48 29 fe             	sub    %rdi,%rsi
    10b1:	48 89 f0             	mov    %rsi,%rax
    10b4:	48 c1 ee 3f          	shr    $0x3f,%rsi
    10b8:	48 c1 f8 03          	sar    $0x3,%rax
    10bc:	48 01 c6             	add    %rax,%rsi
    10bf:	48 d1 fe             	sar    %rsi
    10c2:	74 14                	je     10d8 <register_tm_clones+0x38>
    10c4:	48 8b 05 25 2f 00 00 	mov    0x2f25(%rip),%rax        # 3ff0 <_ITM_registerTMCloneTable>
    10cb:	48 85 c0             	test   %rax,%rax
    10ce:	74 08                	je     10d8 <register_tm_clones+0x38>
    10d0:	ff e0                	jmpq   *%rax
    10d2:	66 0f 1f 44 00 00    	nopw   0x0(%rax,%rax,1)
    10d8:	c3                   	retq   
    10d9:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)

00000000000010e0 <__do_global_dtors_aux>:
    10e0:	f3 0f 1e fa          	endbr64 
    10e4:	80 3d 45 2f 00 00 00 	cmpb   $0x0,0x2f45(%rip)        # 4030 <__TMC_END__>
    10eb:	75 33                	jne    1120 <__do_global_dtors_aux+0x40>
    10ed:	55                   	push   %rbp
    10ee:	48 83 3d 02 2f 00 00 	cmpq   $0x0,0x2f02(%rip)        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
    10f5:	00 
    10f6:	48 89 e5             	mov    %rsp,%rbp
    10f9:	74 0d                	je     1108 <__do_global_dtors_aux+0x28>
    10fb:	48 8b 3d 26 2f 00 00 	mov    0x2f26(%rip),%rdi        # 4028 <__dso_handle>
    1102:	ff 15 f0 2e 00 00    	callq  *0x2ef0(%rip)        # 3ff8 <__cxa_finalize@GLIBC_2.2.5>
    1108:	e8 63 ff ff ff       	callq  1070 <deregister_tm_clones>
    110d:	c6 05 1c 2f 00 00 01 	movb   $0x1,0x2f1c(%rip)        # 4030 <__TMC_END__>
    1114:	5d                   	pop    %rbp
    1115:	c3                   	retq   
    1116:	66 2e 0f 1f 84 00 00 	nopw   %cs:0x0(%rax,%rax,1)
    111d:	00 00 00 
    1120:	c3                   	retq   
    1121:	66 66 2e 0f 1f 84 00 	data16 nopw %cs:0x0(%rax,%rax,1)
    1128:	00 00 00 00 
    112c:	0f 1f 40 00          	nopl   0x0(%rax)

0000000000001130 <frame_dummy>:
    1130:	f3 0f 1e fa          	endbr64 
    1134:	e9 67 ff ff ff       	jmpq   10a0 <register_tm_clones>

0000000000001139 <add>:
    1139:	8d 04 37             	lea    (%rdi,%rsi,1),%eax
    113c:	c3                   	retq   

000000000000113d <main>:
    113d:	48 83 ec 08          	sub    $0x8,%rsp
    1141:	be 06 00 00 00       	mov    $0x6,%esi
    1146:	bf 05 00 00 00       	mov    $0x5,%edi
    114b:	e8 e9 ff ff ff       	callq  1139 <add>
    1150:	89 c6                	mov    %eax,%esi
    1152:	48 8d 3d ab 0e 00 00 	lea    0xeab(%rip),%rdi        # 2004 <_IO_stdin_used+0x4>
    1159:	b8 00 00 00 00       	mov    $0x0,%eax
    115e:	e8 cd fe ff ff       	callq  1030 <printf@plt>
    1163:	b8 00 00 00 00       	mov    $0x0,%eax
    1168:	48 83 c4 08          	add    $0x8,%rsp
    116c:	c3                   	retq   
    116d:	0f 1f 00             	nopl   (%rax)

0000000000001170 <__libc_csu_init>:
    1170:	f3 0f 1e fa          	endbr64 
    1174:	41 57                	push   %r15
    1176:	4c 8d 3d 6b 2c 00 00 	lea    0x2c6b(%rip),%r15        # 3de8 <__frame_dummy_init_array_entry>
    117d:	41 56                	push   %r14
    117f:	49 89 d6             	mov    %rdx,%r14
    1182:	41 55                	push   %r13
    1184:	49 89 f5             	mov    %rsi,%r13
    1187:	41 54                	push   %r12
    1189:	41 89 fc             	mov    %edi,%r12d
    118c:	55                   	push   %rbp
    118d:	48 8d 2d 5c 2c 00 00 	lea    0x2c5c(%rip),%rbp        # 3df0 <__init_array_end>
    1194:	53                   	push   %rbx
    1195:	4c 29 fd             	sub    %r15,%rbp
    1198:	48 83 ec 08          	sub    $0x8,%rsp
    119c:	e8 5f fe ff ff       	callq  1000 <_init>
    11a1:	48 c1 fd 03          	sar    $0x3,%rbp
    11a5:	74 1f                	je     11c6 <__libc_csu_init+0x56>
    11a7:	31 db                	xor    %ebx,%ebx
    11a9:	0f 1f 80 00 00 00 00 	nopl   0x0(%rax)
    11b0:	4c 89 f2             	mov    %r14,%rdx
    11b3:	4c 89 ee             	mov    %r13,%rsi
    11b6:	44 89 e7             	mov    %r12d,%edi
    11b9:	41 ff 14 df          	callq  *(%r15,%rbx,8)
    11bd:	48 83 c3 01          	add    $0x1,%rbx
    11c1:	48 39 dd             	cmp    %rbx,%rbp
    11c4:	75 ea                	jne    11b0 <__libc_csu_init+0x40>
    11c6:	48 83 c4 08          	add    $0x8,%rsp
    11ca:	5b                   	pop    %rbx
    11cb:	5d                   	pop    %rbp
    11cc:	41 5c                	pop    %r12
    11ce:	41 5d                	pop    %r13
    11d0:	41 5e                	pop    %r14
    11d2:	41 5f                	pop    %r15
    11d4:	c3                   	retq   
    11d5:	66 66 2e 0f 1f 84 00 	data16 nopw %cs:0x0(%rax,%rax,1)
    11dc:	00 00 00 00 

00000000000011e0 <__libc_csu_fini>:
    11e0:	f3 0f 1e fa          	endbr64 
    11e4:	c3                   	retq   

Disassembly of section .fini:

00000000000011e8 <_fini>:
    11e8:	f3 0f 1e fa          	endbr64 
    11ec:	48 83 ec 08          	sub    $0x8,%rsp
    11f0:	48 83 c4 08          	add    $0x8,%rsp
    11f4:	c3                   	retq   

```

注意一下add函数：

```s
0000000000001139 <add>:
    1139:	8d 04 37             	lea    (%rdi,%rsi,1),%eax
    113c:	c3                   	retq   
```

用寻址的方式完成了加法计算：
```s
lea    (%rdi,%rsi,1),%eax
```

就相当于 `eax=rdi+rsi*1`，`rdi`和`rsi`是传参的两个寄存器。

这也是gcc优化程序的一种手段。

## 查看节

```bash
objdump -s main
```

输出：

```text

main：     文件格式 elf64-x86-64

Contents of section .interp:
 02a8 2f6c6962 36342f6c 642d6c69 6e75782d  /lib64/ld-linux-
 02b8 7838362d 36342e73 6f2e3200           x86-64.so.2.    
Contents of section .note.gnu.build-id:
 02c4 04000000 14000000 03000000 474e5500  ............GNU.
 02d4 cd22e2f0 7545bdf9 2ae713c1 dba544d6  ."..uE..*.....D.
 02e4 031eb047                             ...G            
Contents of section .note.ABI-tag:
 02e8 04000000 10000000 01000000 474e5500  ............GNU.
 02f8 00000000 03000000 02000000 00000000  ................
Contents of section .gnu.hash:
 0308 01000000 01000000 01000000 00000000  ................
 0318 00000000 00000000 00000000           ............    
Contents of section .dynsym:
 0328 00000000 00000000 00000000 00000000  ................
 0338 00000000 00000000 3f000000 20000000  ........?... ...
 0348 00000000 00000000 00000000 00000000  ................
 0358 0b000000 12000000 00000000 00000000  ................
 0368 00000000 00000000 21000000 12000000  ........!.......
 0378 00000000 00000000 00000000 00000000  ................
 0388 5b000000 20000000 00000000 00000000  [... ...........
 0398 00000000 00000000 6a000000 20000000  ........j... ...
 03a8 00000000 00000000 00000000 00000000  ................
 03b8 12000000 22000000 00000000 00000000  ...."...........
 03c8 00000000 00000000                    ........        
Contents of section .dynstr:
 03d0 006c6962 632e736f 2e360070 72696e74  .libc.so.6.print
 03e0 66005f5f 6378615f 66696e61 6c697a65  f.__cxa_finalize
 03f0 005f5f6c 6962635f 73746172 745f6d61  .__libc_start_ma
 0400 696e0047 4c494243 5f322e32 2e35005f  in.GLIBC_2.2.5._
 0410 49544d5f 64657265 67697374 6572544d  ITM_deregisterTM
 0420 436c6f6e 65546162 6c65005f 5f676d6f  CloneTable.__gmo
 0430 6e5f7374 6172745f 5f005f49 544d5f72  n_start__._ITM_r
 0440 65676973 74657254 4d436c6f 6e655461  egisterTMCloneTa
 0450 626c6500                             ble.            
Contents of section .gnu.version:
 0454 00000000 02000200 00000000 0200      ..............  
Contents of section .gnu.version_r:
 0468 01000100 01000000 10000000 00000000  ................
 0478 751a6909 00000200 33000000 00000000  u.i.....3.......
Contents of section .rela.dyn:
 0488 e83d0000 00000000 08000000 00000000  .=..............
 0498 30110000 00000000 f03d0000 00000000  0........=......
 04a8 08000000 00000000 e0100000 00000000  ................
 04b8 28400000 00000000 08000000 00000000  (@..............
 04c8 28400000 00000000 d83f0000 00000000  (@.......?......
 04d8 06000000 01000000 00000000 00000000  ................
 04e8 e03f0000 00000000 06000000 03000000  .?..............
 04f8 00000000 00000000 e83f0000 00000000  .........?......
 0508 06000000 04000000 00000000 00000000  ................
 0518 f03f0000 00000000 06000000 05000000  .?..............
 0528 00000000 00000000 f83f0000 00000000  .........?......
 0538 06000000 06000000 00000000 00000000  ................
Contents of section .rela.plt:
 0548 18400000 00000000 07000000 02000000  .@..............
 0558 00000000 00000000                    ........        
Contents of section .init:
 1000 f30f1efa 4883ec08 488b05d9 2f000048  ....H...H.../..H
 1010 85c07402 ffd04883 c408c3             ..t...H....     
Contents of section .plt:
 1020 ff35e22f 0000ff25 e42f0000 0f1f4000  .5./...%./....@.
 1030 ff25e22f 00006800 000000e9 e0ffffff  .%./..h.........
Contents of section .text:
 1040 f30f1efa 31ed4989 d15e4889 e24883e4  ....1.I..^H..H..
 1050 f050544c 8d058601 0000488d 0d0f0100  .PTL......H.....
 1060 00488d3d d5000000 ff15722f 0000f490  .H.=......r/....
 1070 488d3db9 2f000048 8d05b22f 00004839  H.=./..H.../..H9
 1080 f8741548 8b054e2f 00004885 c07409ff  .t.H..N/..H..t..
 1090 e00f1f80 00000000 c30f1f80 00000000  ................
 10a0 488d3d89 2f000048 8d35822f 00004829  H.=./..H.5./..H)
 10b0 fe4889f0 48c1ee3f 48c1f803 4801c648  .H..H..?H...H..H
 10c0 d1fe7414 488b0525 2f000048 85c07408  ..t.H..%/..H..t.
 10d0 ffe0660f 1f440000 c30f1f80 00000000  ..f..D..........
 10e0 f30f1efa 803d452f 00000075 33554883  .....=E/...u3UH.
 10f0 3d022f00 00004889 e5740d48 8b3d262f  =./...H..t.H.=&/
 1100 0000ff15 f02e0000 e863ffff ffc6051c  .........c......
 1110 2f000001 5dc3662e 0f1f8400 00000000  /...].f.........
 1120 c366662e 0f1f8400 00000000 0f1f4000  .ff...........@.
 1130 f30f1efa e967ffff ff8d0437 c34883ec  .....g.....7.H..
 1140 08be0600 0000bf05 000000e8 e9ffffff  ................
 1150 89c6488d 3dab0e00 00b80000 0000e8cd  ..H.=...........
 1160 feffffb8 00000000 4883c408 c30f1f00  ........H.......
 1170 f30f1efa 41574c8d 3d6b2c00 00415649  ....AWL.=k,..AVI
 1180 89d64155 4989f541 544189fc 55488d2d  ..AUI..ATA..UH.-
 1190 5c2c0000 534c29fd 4883ec08 e85ffeff  \,..SL).H...._..
 11a0 ff48c1fd 03741f31 db0f1f80 00000000  .H...t.1........
 11b0 4c89f24c 89ee4489 e741ff14 df4883c3  L..L..D..A...H..
 11c0 014839dd 75ea4883 c4085b5d 415c415d  .H9.u.H...[]A\A]
 11d0 415e415f c366662e 0f1f8400 00000000  A^A_.ff.........
 11e0 f30f1efa c3                          .....           
Contents of section .fini:
 11e8 f30f1efa 4883ec08 4883c408 c3        ....H...H....   
Contents of section .rodata:
 2000 01000200 68656c6c 6f20776f 726c6420  ....hello world 
 2010 25640a00                             %d..            
Contents of section .eh_frame_hdr:
 2014 011b033b 38000000 06000000 0cf0ffff  ...;8...........
 2024 6c000000 2cf0ffff 54000000 25f1ffff  l...,...T...%...
 2034 94000000 29f1ffff a8000000 5cf1ffff  ....).......\...
 2044 c4000000 ccf1ffff 0c010000           ............    
Contents of section .eh_frame:
 2050 14000000 00000000 017a5200 01781001  .........zR..x..
 2060 1b0c0708 90010000 14000000 1c000000  ................
 2070 d0efffff 2f000000 00440710 00000000  ..../....D......
 2080 24000000 34000000 98efffff 20000000  $...4....... ...
 2090 000e1046 0e184a0f 0b770880 003f1a3b  ...F..J..w...?.;
 20a0 2a332422 00000000 10000000 5c000000  *3$"........\...
 20b0 89f0ffff 04000000 00000000 18000000  ................
 20c0 70000000 79f0ffff 30000000 00440e10  p...y...0....D..
 20d0 6b0e0800 00000000 44000000 8c000000  k.......D.......
 20e0 90f0ffff 65000000 00460e10 8f02490e  ....e....F....I.
 20f0 188e0345 0e208d04 450e288c 05440e30  ...E. ..E.(..D.0
 2100 8606480e 38830747 0e406e0e 38410e30  ..H.8..G.@n.8A.0
 2110 410e2842 0e20420e 18420e10 420e0800  A.(B. B..B..B...
 2120 10000000 d4000000 b8f0ffff 05000000  ................
 2130 00000000 00000000                    ........        
Contents of section .init_array:
 3de8 30110000 00000000                    0.......        
Contents of section .fini_array:
 3df0 e0100000 00000000                    ........        
Contents of section .dynamic:
 3df8 01000000 00000000 01000000 00000000  ................
 3e08 0c000000 00000000 00100000 00000000  ................
 3e18 0d000000 00000000 e8110000 00000000  ................
 3e28 19000000 00000000 e83d0000 00000000  .........=......
 3e38 1b000000 00000000 08000000 00000000  ................
 3e48 1a000000 00000000 f03d0000 00000000  .........=......
 3e58 1c000000 00000000 08000000 00000000  ................
 3e68 f5feff6f 00000000 08030000 00000000  ...o............
 3e78 05000000 00000000 d0030000 00000000  ................
 3e88 06000000 00000000 28030000 00000000  ........(.......
 3e98 0a000000 00000000 84000000 00000000  ................
 3ea8 0b000000 00000000 18000000 00000000  ................
 3eb8 15000000 00000000 00000000 00000000  ................
 3ec8 03000000 00000000 00400000 00000000  .........@......
 3ed8 02000000 00000000 18000000 00000000  ................
 3ee8 14000000 00000000 07000000 00000000  ................
 3ef8 17000000 00000000 48050000 00000000  ........H.......
 3f08 07000000 00000000 88040000 00000000  ................
 3f18 08000000 00000000 c0000000 00000000  ................
 3f28 09000000 00000000 18000000 00000000  ................
 3f38 fbffff6f 00000000 00000008 00000000  ...o............
 3f48 feffff6f 00000000 68040000 00000000  ...o....h.......
 3f58 ffffff6f 00000000 01000000 00000000  ...o............
 3f68 f0ffff6f 00000000 54040000 00000000  ...o....T.......
 3f78 f9ffff6f 00000000 03000000 00000000  ...o............
 3f88 00000000 00000000 00000000 00000000  ................
 3f98 00000000 00000000 00000000 00000000  ................
 3fa8 00000000 00000000 00000000 00000000  ................
 3fb8 00000000 00000000 00000000 00000000  ................
 3fc8 00000000 00000000 00000000 00000000  ................
Contents of section .got:
 3fd8 00000000 00000000 00000000 00000000  ................
 3fe8 00000000 00000000 00000000 00000000  ................
 3ff8 00000000 00000000                    ........        
Contents of section .got.plt:
 4000 f83d0000 00000000 00000000 00000000  .=..............
 4010 00000000 00000000 36100000 00000000  ........6.......
Contents of section .data:
 4020 00000000 00000000 28400000 00000000  ........(@......
Contents of section .comment:
 0000 4743433a 2028474e 55292039 2e322e30  GCC: (GNU) 9.2.0
 0010 00                                   .               

```

注意到printf中的常量字符串位于.rodata节中。

## 反汇编指定符号

```bash
objdump --disassemble=add main
```

输出：

```s

main：     文件格式 elf64-x86-64


Disassembly of section .init:

Disassembly of section .plt:

Disassembly of section .text:

0000000000001139 <add>:
    1139:	8d 04 37             	lea    (%rdi,%rsi,1),%eax
    113c:	c3                   	retq   

Disassembly of section .fini:

```

## 打印符号表内容

```bash
objdump -t main
```

输出

```text

main：     文件格式 elf64-x86-64

SYMBOL TABLE:
00000000000002a8 l    d  .interp	0000000000000000              .interp
00000000000002c4 l    d  .note.gnu.build-id	0000000000000000              .note.gnu.build-id
00000000000002e8 l    d  .note.ABI-tag	0000000000000000              .note.ABI-tag
0000000000000308 l    d  .gnu.hash	0000000000000000              .gnu.hash
0000000000000328 l    d  .dynsym	0000000000000000              .dynsym
00000000000003d0 l    d  .dynstr	0000000000000000              .dynstr
0000000000000454 l    d  .gnu.version	0000000000000000              .gnu.version
0000000000000468 l    d  .gnu.version_r	0000000000000000              .gnu.version_r
0000000000000488 l    d  .rela.dyn	0000000000000000              .rela.dyn
0000000000000548 l    d  .rela.plt	0000000000000000              .rela.plt
0000000000001000 l    d  .init	0000000000000000              .init
0000000000001020 l    d  .plt	0000000000000000              .plt
0000000000001040 l    d  .text	0000000000000000              .text
00000000000011e8 l    d  .fini	0000000000000000              .fini
0000000000002000 l    d  .rodata	0000000000000000              .rodata
0000000000002014 l    d  .eh_frame_hdr	0000000000000000              .eh_frame_hdr
0000000000002050 l    d  .eh_frame	0000000000000000              .eh_frame
0000000000003de8 l    d  .init_array	0000000000000000              .init_array
0000000000003df0 l    d  .fini_array	0000000000000000              .fini_array
0000000000003df8 l    d  .dynamic	0000000000000000              .dynamic
0000000000003fd8 l    d  .got	0000000000000000              .got
0000000000004000 l    d  .got.plt	0000000000000000              .got.plt
0000000000004020 l    d  .data	0000000000000000              .data
0000000000004030 l    d  .bss	0000000000000000              .bss
0000000000000000 l    d  .comment	0000000000000000              .comment
0000000000000000 l    df *ABS*	0000000000000000              init.c
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
0000000000001070 l     F .text	0000000000000000              deregister_tm_clones
00000000000010a0 l     F .text	0000000000000000              register_tm_clones
00000000000010e0 l     F .text	0000000000000000              __do_global_dtors_aux
0000000000004030 l     O .bss	0000000000000001              completed.7392
0000000000003df0 l     O .fini_array	0000000000000000              __do_global_dtors_aux_fini_array_entry
0000000000001130 l     F .text	0000000000000000              frame_dummy
0000000000003de8 l     O .init_array	0000000000000000              __frame_dummy_init_array_entry
0000000000000000 l    df *ABS*	0000000000000000              main.c
0000000000000000 l    df *ABS*	0000000000000000              crtstuff.c
0000000000002134 l     O .eh_frame	0000000000000000              __FRAME_END__
0000000000000000 l    df *ABS*	0000000000000000              
0000000000003df0 l       .init_array	0000000000000000              __init_array_end
0000000000003df8 l     O .dynamic	0000000000000000              _DYNAMIC
0000000000003de8 l       .init_array	0000000000000000              __init_array_start
0000000000002014 l       .eh_frame_hdr	0000000000000000              __GNU_EH_FRAME_HDR
0000000000004000 l     O .got.plt	0000000000000000              _GLOBAL_OFFSET_TABLE_
0000000000001000 l     F .init	0000000000000000              _init
00000000000011e0 g     F .text	0000000000000005              __libc_csu_fini
0000000000000000  w      *UND*	0000000000000000              _ITM_deregisterTMCloneTable
0000000000004020  w      .data	0000000000000000              data_start
0000000000001139 g     F .text	0000000000000004              add
0000000000004030 g       .data	0000000000000000              _edata
00000000000011e8 g     F .fini	0000000000000000              .hidden _fini
0000000000000000       F *UND*	0000000000000000              printf@@GLIBC_2.2.5
0000000000000000       F *UND*	0000000000000000              __libc_start_main@@GLIBC_2.2.5
0000000000004020 g       .data	0000000000000000              __data_start
0000000000000000  w      *UND*	0000000000000000              __gmon_start__
0000000000004028 g     O .data	0000000000000000              .hidden __dso_handle
0000000000002000 g     O .rodata	0000000000000004              _IO_stdin_used
0000000000001170 g     F .text	0000000000000065              __libc_csu_init
0000000000004038 g       .bss	0000000000000000              _end
0000000000001040 g     F .text	000000000000002f              _start
0000000000004030 g       .bss	0000000000000000              __bss_start
000000000000113d g     F .text	0000000000000030              main
0000000000004030 g     O .data	0000000000000000              .hidden __TMC_END__
0000000000000000  w      *UND*	0000000000000000              _ITM_registerTMCloneTable
0000000000000000  w    F *UND*	0000000000000000              __cxa_finalize@@GLIBC_2.2.5



```

## 打印动态符号表(外部链接)

```bash
objdump -T main
```

输出：

```text

main：     文件格式 elf64-x86-64

DYNAMIC SYMBOL TABLE:
0000000000000000  w   D  *UND*	0000000000000000              _ITM_deregisterTMCloneTable
0000000000000000      DF *UND*	0000000000000000  GLIBC_2.2.5 printf
0000000000000000      DF *UND*	0000000000000000  GLIBC_2.2.5 __libc_start_main
0000000000000000  w   D  *UND*	0000000000000000              __gmon_start__
0000000000000000  w   D  *UND*	0000000000000000              _ITM_registerTMCloneTable
0000000000000000  w   DF *UND*	0000000000000000  GLIBC_2.2.5 __cxa_finalize



```

## 打印重定位信息

```bash
objdump -r main
```

输出

```text

main：     文件格式 elf64-x86-64


```

因为文件是可执行文件，所有没有可重定位信息。

## 打印动态可重定位信息

```bash
objdump -R main
```

```text

main：     文件格式 elf64-x86-64

DYNAMIC RELOCATION RECORDS
OFFSET           TYPE              VALUE 
0000000000003de8 R_X86_64_RELATIVE  *ABS*+0x0000000000001130
0000000000003df0 R_X86_64_RELATIVE  *ABS*+0x00000000000010e0
0000000000004028 R_X86_64_RELATIVE  *ABS*+0x0000000000004028
0000000000003fd8 R_X86_64_GLOB_DAT  _ITM_deregisterTMCloneTable
0000000000003fe0 R_X86_64_GLOB_DAT  __libc_start_main@GLIBC_2.2.5
0000000000003fe8 R_X86_64_GLOB_DAT  __gmon_start__
0000000000003ff0 R_X86_64_GLOB_DAT  _ITM_registerTMCloneTable
0000000000003ff8 R_X86_64_GLOB_DAT  __cxa_finalize@GLIBC_2.2.5
0000000000004018 R_X86_64_JUMP_SLOT  printf@GLIBC_2.2.5



```

动态链接库的可重定位信息
