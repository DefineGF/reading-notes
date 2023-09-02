#### 局部变量类型



##### 计算64个字的数据包的校验和

64个字即表示64次循环中，每次读取一个int值即可！

```c
int checksum_v1(int *data) 
{
    char i;
    int sum = 0;
    for (i = 0; i < 64; ++i) 
    {
        sum += data[i];
    }
    return sum;
}
```

编译成汇编：

```assembly
checksum_v1
	mov r2, r0  ; r2 = data
	mov r0, #0  ; sum = 0
	mov r1,	#0	; i = 0
checksum_v1_loop
	LDR r3, [r2, r1, LSL #2]  ; r3 = data[i]
	ADD r1, r1, #1			  ; r1 = i + 1
	AND r1, r1, #0xff		  ; i = (char)r1
	CMP r1, 0x40			  ; compare i, 64
	ADD r0, r3, r0			  ; sum += r3
	BCC checksum_v1_loop      ; if (i < 64) loop
	MOV pc, r14				  ; return sum
```

第8行中，增加`AND r1, r1, #0xff` 保证i的范围为0~255，当将i声明为 unsigned int 类型时，该指令就可以省略了！

> 注意第6行中，左移两位（*4），即相当于将4个字节（32b ）的内容加载在寄存器中



##### 计算64个 半字 的数据包的校验和

```c
short checksum_v3(short *data) 
{
    unsigned int i;
    short num = 0;
    for (i = 0; i < 64; ++i)
    {
        sum += (short)(sum + data[i]);
    }
    return sum;
}
```

第 7 行中，sum + data[i]  为整数类型，因此需要转换成 short 类型！编译成汇编：

```assembly
checksum_v3
	mov r2, r0  ; r2 = data
	mov r0, #0  ; sum = 0
	mov r1,	#0	; i = 0
checksum_v3_loop
	ADD r3, r2, r1, LSL #1    ; r3 = &data[i]
	LDRH r3, [r3, #0]		  ; r3 = data[i] 由于 data[i] 为 short 类型，因此移入半字
	ADD r1, r1, #1			  ; i += 1
	CMP r1, #0x40	          ; compare i, 64
	ADD r0, r3, r0			  ; r0 = sum(r0) + r3
	MOV r0, r0, LSL #16		  ; r0 作为 short 类型，逻辑左移16b，
	MOV r0, r0, ASR #16		  ; 算术右移16， 和上一条指令用以完成  sum = (short)r0
	BCC checksum_v3_loop	  ; if (i < 64) goto loop
	mov pc, r14				  ; return sum
```



最终写法：

```c
short chechsum_v(short *data)
{
    unsigned int i;
    int sum = 0;
    for (i = 0; i < 64; ++i)
    {
        sum += *(data++);
    }
    return (short)sum;
}
```

编译成汇编：

```assembly
checksum_v4
	mov r2, #0	;sum = 0
	mov r1, #0 	;i = 0
check_v4_loop
	LDRSH r3, [r0], #2	; r3 = *(data++) 这里的汇编表示 先将 r0(data) 内存中的值（半字，2个字节）加载到 r3 寄存器中，然后 r0 += 2
	ADD  r1, r1, #1		; i++
	...
	mov r0, r2, LSL #16
	mov r0, r0, ASR #16 ; r0 = (short)sum
	mov pc, r14
```





