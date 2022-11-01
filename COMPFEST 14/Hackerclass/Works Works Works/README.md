## Overview
We're given a stripped binary. We can open it in a tool like ghidra, but the challenge here is to find the main function. I'm not going to explain how to find the main function here but here it is (this is straight from ghidra so its a little bit convoluted and messed up)
```c
undefined8 FUN_001015d6(void){
bool bVar1;
undefined8 uVar2;
char *__src;
long in_FS_OFFSET;
ulong local_f0;
undefined local_e8 [112];
char local_78 [104];
long local_10;

local_10 = *(long *)(in_FS_OFFSET + 0x28);
memset(local_e8,0,100);
memset(local_78,0,100);
printf("Password: ");
__isoc99_scanf(&DAT_0010200f,local_e8);
bVar1 = true;
uVar2 = FUN_00101229(local_e8);
uVar2 = FUN_00101280(uVar2);
__src = (char *)FUN_0010157f(uVar2);
strcpy(local_78,__src);
for (local_f0 = 0; local_f0 < (ulong)(long)DAT_001040d0; local_f0 = local_f0 + 1){
    if ((int)local_78[local_f0] != *(int *)(&DAT_00104020 + local_f0 * 4)){
        bVar1 = false;
    }
}
if (bVar1) {
    puts("Benar");
}
else {
    puts("Salah");
}
if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                /* WARNING: Subroutine does not return */
__stack_chk_fail();
}
return 0;
}
```
It's a little bit intimidating to read but here is the important part
```c
printf("Password: ");
__isoc99_scanf(&DAT_0010200f,local_e8);
bVar1 = true;
uVar2 = FUN_00101229(local_e8);
uVar2 = FUN_00101280(uVar2);
__src = (char *)FUN_0010157f(uVar2);
strcpy(local_78,__src);
for (local_f0 = 0; local_f0 < (ulong)(long)DAT_001040d0; local_f0 = local_f0 + 1){
    if ((int)local_78[local_f0] != *(int *)(&DAT_00104020 + local_f0 * 4)){
        bVar1 = false;
    }
}
```
The program takes an input (the flag which acts as a password) and modifies it three times using three function, the purpose of the first and third function is pretty easy to tell, the second one not so much as we'll see in a bit. The program then compares the return value of the third function which has a length n stored in 0x001040d0 offset to n numbers stored in 0x00104020 offset.
```c
char * FUN_00101229(char *param_1){
    size_t sVar1;
    ulong local_10;

    local_10 = 0;
    while( true ){
        sVar1 = strlen(param_1);
        if (sVar1 <= local_10) break;
        param_1[local_10] = param_1[local_10] ^ 0x20;
        local_10 = local_10 + 1;
    }
    return param_1;
}
```
the first function xors every character value in the flag with 0x20 (32) this is easy to reverse
```c
char * FUN_0010157f(char *param_1){
    size_t sVar1;
    ulong local_10;

    local_10 = 0;
    while( true ){
        sVar1 = strlen(param_1);
        if (sVar1 <= local_10) break;
        param_1[local_10] = param_1[local_10] + -0x80;
        local_10 = local_10 + 1;
    }
    return param_1;
}
```
the third one substracts 0x80 (128) to every character that is returned by the mysterious second function. 
## Solution
To reverse engineer the flag, we need to start from the very end. Using gdb, we can set breakpoints for every modification the program does. to shorten this solution, it should be known that the second function is a base64 encoding function, which can be decoded easily. The number stored thet is in the 0x001040d0 memory offset is 0x2c (44) and the 44 numbers stored in the 0x00104020 memory offset can also be found using gdb.
```python
import base64

hexes = [0xffffffd9, 0xffffffb2, 0xffffffb9, 0xfffffff4, 0xffffffe3, 0xffffffc7, 0xffffffda, 0xffffffec, 0xffffffe3, 0xffffffb3, 0xffffffd1, 0xffffffd2,0xffffffc6, 0xffffffc6, 0xfffffff4, 0xffffffd9, 0xffffffd4, 0xffffffb1, 0xffffffc9, 0xffffffd4, 0xffffffc5, 0xffffffee, 0xffffffb9, 0xffffffc3, 0xffffffd1, 0xffffffd6, 0xffffffce, 0xffffffc6, 0xffffffc6, 0xffffffe8, 0xffffffd2, 0xffffffaf, 0xffffffd5, 0xffffffb0, 0xffffffe8, 0xffffffca, 0xffffffd2, 0xffffffec, 0xffffffd1, 0xffffffd2,0xffffffc5, 0xffffffe8, 0xffffffe8, 0xffffffe4]

lenh = 0x2c

for i in range(lenh): #every number in hexes is a negative number in c but python handles negatives differently so we need to convert them manually
    hexes[i] = chr(((-1*(4294967295 - hexes[i] + 1)) + 0x80)) 

final_array = [base64.b64decode(''.join(hexes))[i]^0x20 for i in range(len(base64.b64decode(''.join(hexes))))]
decrypted = ''.join([chr(x) for x in final_array])

print(decrypted)
```
##### Solved by Cheesewaffle
