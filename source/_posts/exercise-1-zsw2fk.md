---
title: Fuzzing101 - Exercise 1
date: '2024-03-13 20:01:56'
updated: '2024-03-13 22:59:41'
permalink: /post/exercise-1-zsw2fk.html
comments: true
toc: true
---

# Fuzzing101 - Exercise 1

目标：<span style="font-weight: bold;" data-type="strong">[CVE-2019-13288](https://www.cvedetails.com/cve/CVE-2019-13288/)</span> in XPDF 3.02

工具：AFL（虽然文中要求是AFL++）

‍

‍

# XPDF下载与编译

官网：[https://www.xpdfreader.com/old-versions.html](https://www.xpdfreader.com/old-versions.html)

## 编译

```sh
./configure --prefix="/work/Fuzzing101/Exercise_1/install/"
make && make install
```

## 验证

```sh
root@TruthClient:/work/Fuzzing101/Exercise_1/xpdf-3.02# /work/Fuzzing101/Exercise_1/install/bin/pdfinfo /work/Fuzzing101/Exercise_1/pdf_example/helloworld.pdf 
Tagged:         no
Pages:          1
Encrypted:      no
Page size:      200 x 200 pts
File size:      678 bytes
Optimized:      no
PDF version:    1.7
```

‍

## 使用AFL编译

```sh
CC=/code/AFL/afl-clang CXX=/code/AFL/afl-clang++ ./configure --prefix="/work/Fuzzing101/Exercise_1/install/"
make
make install
```

# RUN

```sh
afl-fuzz -i /work/Fuzzing101/Exercise_1/pdf_example/ -o /work/Fuzzing101/Exercise_1/out/ -- /work/Fuzzing101/Exercise_1/install/bin/pdftotext @@ /work/Fuzzing101/Exercise_1/output
```

​`-s 123`​是AFL++中设置随机种子功能，AFL没有该功能

‍

# 结果

## crash

跑出一个crash后就把fuzz停住了，毕竟是拿来练手的程序，应该不会有其他洞了

```sh

                      american fuzzy lop 2.57b (pdftotext)

┌─ process timing ─────────────────────────────────────┬─ overall results ─────┐
│        run time : 0 days, 0 hrs, 22 min, 27 sec      │  cycles done : 0      │
│   last new path : 0 days, 0 hrs, 2 min, 34 sec       │  total paths : 1962   │
│ last uniq crash : 0 days, 0 hrs, 18 min, 30 sec      │ uniq crashes : 1      │
│  last uniq hang : none seen yet                      │   uniq hangs : 0      │
├─ cycle progress ────────────────────┬─ map coverage ─┴───────────────────────┤
│  now processing : 75* (3.82%)       │    map density : 3.52% / 8.18%         │
│ paths timed out : 0 (0.00%)         │ count coverage : 3.87 bits/tuple       │
├─ stage progress ────────────────────┼─ findings in depth ────────────────────┤
│  now trying : arith 8/8             │ favored paths : 322 (16.41%)           │
│ stage execs : 16.1k/41.9k (38.58%)  │  new edges on : 494 (25.18%)           │
│ total execs : 1.49M                 │ total crashes : 1 (1 unique)           │
│  exec speed : 1144/sec              │  total tmouts : 1 (1 unique)           │
├─ fuzzing strategy yields ───────────┴───────────────┬─ path geometry ────────┤
│   bit flips : 630/90.3k, 141/90.3k, 113/90.3k       │    levels : 3          │
│  byte flips : 5/11.3k, 18/6366, 18/6386             │   pending : 1951       │
│ arithmetics : 260/316k, 21/33.0k, 0/0               │  pend fav : 314        │
│  known ints : 22/28.9k, 69/157k, 83/252k            │ own finds : 1960       │
│  dictionary : 0/0, 0/0, 149/273k                    │  imported : n/a        │
│       havoc : 432/92.3k, 0/0                        │ stability : 100.00%    │
│        trim : 0.64%/5522, 43.57%                    ├────────────────────────┘
└─────────────────────────────────────────────────────┘          [cpu000: 79%]

```

## out目录分析

在存放fuzz结果的目录`/work/Fuzzing101/Exercise_1/out`​结构如下：

```sh
root@TruthClient:/work/Fuzzing101/Exercise_1/out# tree
.
├── crashes			# 存放使程序crash的样例
│   ├── id:000000,sig:11,src:000001,op:flip4,pos:799
│   └── README.txt	# fuzz的命令和一些tips
├── fuzz_bitmap
├── fuzzer_stats	# fuzzer的状态
├── hangs
├── plot_data		# 用于图像显示
└── queue			# fuzz的样例队列
    ├── id:000000,orig:helloworld.pdf
    ├── id:000001,orig:small-example-pdf-file.pdf
    ├── id:000002,src:000000,op:flip1,pos:0,+cov
    ├── id:000003,src:000000,op:flip1,pos:5,+cov
	├──...
```

## crash复现分析

```sh
root@TruthClient:/work/Fuzzing101/Exercise_1# /work/Fuzzing101/Exercise_1/install/bin/pdftotext ./out/crashes/id\:000000\,sig\:11\,src\:000001\,op\:flip4\,pos\:799
Segmentation fault
```

虽然也是崩溃了，但是报错信息不如文中那么多，不知道问题出在哪。。。

需要GDB调试确定。

### 重新编译以获取符号堆栈跟踪

```sh
cd /work/Fuzzing101/Exercise_1/xpdf-3.02
make clean
CFLAGS="-g -O0" CXXFLAGS="-g -O0" ./configure --prefix="/work/Fuzzing101/Exercise_1/install/"
make && make install
```

### GDB调试

```sh
root@TruthClient:/work/Fuzzing101/Exercise_1# gdb ./install/bin/pdftotext -q
pwndbg: loaded 154 pwndbg commands and 46 shell commands. Type pwndbg [--shell | --all] [filter] for a list.
pwndbg: created $rebase, $ida GDB functions (can be used with print/break)
Reading symbols from ./install/bin/pdftotext...
------- tip of the day (disable with set show-tips off) -------
Pwndbg context displays where the program branches to thanks to emulating few instructions into the future. You can disable this with set emulate off which may also speed up debugging
pwndbg> set args /work/Fuzzing101/Exercise_1/out/crashes/id:000000,sig:11,src:000001,op:flip4,pos:799 /work/Fuzzing101/Exercise_1/output 
pwndbg> r
Starting program: /work/Fuzzing101/Exercise_1/install/bin/pdftotext /work/Fuzzing101/Exercise_1/out/crashes/id:000000,sig:11,src:000001,op:flip4,pos:799 /work/Fuzzing101/Exercise_1/output 

Program received signal SIGSEGV, Segmentation fault.
0x00007f561fe49eb1 in _int_malloc (av=av@entry=0x7f561ff9eb80 <main_arena>, bytes=bytes@entry=344) at malloc.c:3718
3718    malloc.c: No such file or directory.
LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA
────────────────────────────────────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]────────────────────────────────────────────────────────────────────────────────
 RAX  0x0
*RBX  0x7f561ff9eb80 (main_arena) ◂— 0x0
*RCX  0x7f561ff9ed30 (main_arena+432) —▸ 0x7f561ff9ed20 (main_arena+416) —▸ 0x7f561ff9ed10 (main_arena+400) —▸ 0x7f561ff9ed00 (main_arena+384) —▸ 0x7f561ff9ecf0 (main_arena+368) ◂— ...
 RDX  0x0
*RDI  0x5b
*RSI  0x158
*R8   0x7f561ff9ed30 (main_arena+432) —▸ 0x7f561ff9ed20 (main_arena+416) —▸ 0x7f561ff9ed10 (main_arena+400) —▸ 0x7f561ff9ed00 (main_arena+384) —▸ 0x7f561ff9ecf0 (main_arena+368) ◂— ...
*R9   0x562e57072ca2 (FileStream::makeSubStream(unsigned int, int, unsigned int, Object*)) ◂— endbr64 
*R10  0x562e59b99000 —▸ 0x562e59b98f70 ◂— 0x6874676e654c /* 'Length' */
*R11  0x7f561ff9ebe0 (main_arena+96) —▸ 0x562e59bb4ca0 ◂— 0x0
*R12  0xffffffffffffff90
*R13  0x160
*R14  0x16
*R15  0x14
*RBP  0x158
*RSP  0x7ffffce01fe0
*RIP  0x7f561fe49eb1 (_int_malloc+1089) ◂— mov qword ptr [rsp + 8], rax
─────────────────────────────────────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]─────────────────────────────────────────────────────────────────────────────────────────
 ► 0x7f561fe49eb1 <_int_malloc+1089>    mov    qword ptr [rsp + 8], rax
   0x7f561fe49eb6 <_int_malloc+1094>    cmp    qword ptr fs:[r12], 0
   0x7f561fe49ebc <_int_malloc+1100>    je     _int_malloc+1118                <_int_malloc+1118>
    ↓
   0x7f561fe49ece <_int_malloc+1118>    mov    qword ptr [rsp], 0
   0x7f561fe49ed6 <_int_malloc+1126>    lea    r11, [rbx + 0x60]
   0x7f561fe49eda <_int_malloc+1130>    mov    qword ptr [rsp + 0x20], rbp
   0x7f561fe49edf <_int_malloc+1135>    mov    dword ptr [rsp + 0x48], r14d
   0x7f561fe49ee4 <_int_malloc+1140>    mov    r14, r12
   0x7f561fe49ee7 <_int_malloc+1143>    mov    rsi, qword ptr [rbx + 0x78]
   0x7f561fe49eeb <_int_malloc+1147>    cmp    rsi, r11
   0x7f561fe49eee <_int_malloc+1150>    je     _int_malloc+1992                <_int_malloc+1992>
──────────────────────────────────────────────────────────────────────────────────────────────────────[ STACK ]───────────────────────────────────────────────────────────────────────────────────────────────────────
<Could not read memory at 0x7ffffce01fe0>
────────────────────────────────────────────────────────────────────────────────────────────────────[ BACKTRACE ]─────────────────────────────────────────────────────────────────────────────────────────────────────
 ► 0   0x7f561fe49eb1 _int_malloc+1089
   1   0x7f561fe4c154 malloc+116
   2   0x7f56201b8b29 operator new(unsigned long)+25
   3   0x562e57072cca FileStream::makeSubStream(unsigned int, int, unsigned int, Object*)+40
   4   0x562e57091606 XRef::fetch(int, int, Object*)+260
   5   0x562e57068b8e Object::fetch(XRef*, Object*)+72
   6   0x562e5700c4f6 Dict::lookup(char*, Object*)+84
   7   0x562e57069843 Object::dictLookup(char*, Object*)+51
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> 
```

与文中的图比对了一下调用链，确实不是一条链路。。

bt看一眼，刷了几万条类似如下的内容：

```sh
#12301 0x0000562e57069843 in Object::dictLookup (this=0x7ffffcf42790, key=0x562e570baa64 "Length", obj=0x7ffffcf42510) at Object.h:253
#12302 0x0000562e5706de39 in Parser::makeStream (this=0x562e599445c0, dict=0x7ffffcf42790, fileKey=0x0, encAlgorithm=cryptRC4, keyLength=0, objNum=7, objGen=0) at Parser.cc:156
#12303 0x0000562e5706da69 in Parser::getObj (this=0x562e599445c0, obj=0x7ffffcf42790, fileKey=0x0, encAlgorithm=cryptRC4, keyLength=0, objNum=7, objGen=0) at Parser.cc:94
#12304 0x0000562e570917dc in XRef::fetch (this=0x562e58c0a230, num=7, gen=0, obj=0x7ffffcf42790) at XRef.cc:823
#12305 0x0000562e57068b8e in Object::fetch (this=0x562e599441e8, xref=0x562e58c0a230, obj=0x7ffffcf42790) at Object.cc:106
#12306 0x0000562e5700c4f6 in Dict::lookup (this=0x562e59944190, key=0x562e570baa64 "Length", obj=0x7ffffcf42790) at Dict.cc:76
```

没事了，还是无限递归dos。。

‍

‍
