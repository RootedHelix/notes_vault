Related to [[Embedded]], [[uint16 and uint8 in Embedded]]
#embedded #uint16vsuint8vsuint32 #uint8 

> uint8 is the same as unsigned char in machine code

uint8 is often typedefed to be the same as unsigned char under the hood

`typedef unsigned char uint8;`

- No performance difference 
- No size Difference 


#uint8vsunsigned_char
uint vs unsigned char
# 🔥 Final Comparison

|Type|Standard?|Always 8 bits?|Signedness guaranteed?|Modern style?|
|---|---|---|---|---|
|`char`|Yes|Yes (1 byte)|❌ No|❌|
|`unsigned char`|Yes|Yes (1 byte)|✔ Yes|✔|
|`u_char`|❌ No|Yes|✔ Yes|❌ (legacy style)|
|`uint8_t`|Yes (C99)|✔ Exactly 8 bits|✔ Yes|⭐ Best practice|
