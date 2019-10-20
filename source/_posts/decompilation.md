---
title: decompilation
tags:
  - 反编译
categories:
  - 文章
toc: false
date: 2019-10-06 21:41:43
---

# 可以执行程序反编译为汇编程序

![image.png](/images/2019/10/06/8f80cc70-e83e-11e9-9212-5951f1d89d17.png)

```c
#include <stdio.h>

int f(int a){
	if(a==100) return 1;
	return 0;
}
 
int main()
{
    printf("Enter a num: ");
 
 	int a;
 	scanf("%d",&a);
 	printf("num is %d\n", a);
 	// int code = 100;
 	if(f(a)){
 		printf("code is true\n");
 	}else{
 		printf("code is false\n");
 	}
 	scanf("%s",&a);
    return 0;
}
```

使用x64dbg加载 `main.exe`

```armasm
push ebp
mov ebp,esp
and esp,FFFFFFF0
sub esp,20
call main.401A30
mov dword ptr ss:[esp],main.405064
call <JMP.&printf>
lea eax,dword ptr ss:[esp+1C]
mov dword ptr ss:[esp+4],eax
mov dword ptr ss:[esp],main.405072
call <JMP.&scanf>
mov eax,dword ptr ss:[esp+1C]
mov dword ptr ss:[esp+4],eax
mov dword ptr ss:[esp],main.405075
call <JMP.&printf>
mov eax,dword ptr ss:[esp+1C]
mov dword ptr ss:[esp],eax
call main.401460
test eax,eax
je main.4014D7
mov dword ptr ss:[esp],main.405080
call <JMP.&puts>
jmp main.4014E3
mov dword ptr ss:[esp],main.40508D
call <JMP.&puts>
lea eax,dword ptr ss:[esp+1C]
mov dword ptr ss:[esp+4],eax
mov dword ptr ss:[esp],main.40509B
call <JMP.&scanf>
mov eax,0
leave 
ret 
nop 
nop 
```