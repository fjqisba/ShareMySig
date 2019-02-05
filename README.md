# ShareMySig
这是一个esig共享库,大家可以在这里上传自己制作的esig....

## esig编写格式

和普通的十六进制特征码不一样,esig根据需求拓展了一些规则.

esig中分Config，SubFunc，Func三个板块.
Name代表在edebug中显示的名称

Description代表在edebug显示的描述符,在易语言的支持库中统一为GUID,且不可修改.

IsAligned代表是否地址对齐,某些库函数地址始终是对齐的,这样可以设置为IsAligned=true,提升扫描速度.

SubFunc为第二层匹配函数,采用的是慢速匹配模式(文本解析,占用时间),函数表达文本和Func没有任何区别,但是函数名不能重复

Func为第一层匹配函数,采用的是快速匹配模式(字典树,占用空间),函数名可以重复

因为通过对函数的一些分析得知,绝大多数函数十六进制相差较大(0x00-0xFF),每个字节或许就能成为函数匹配成功的关键,
通俗地说,就是当通过快速定位,完全匹配了第一层之间的前几个字节时候,此函数便有极大概率是想要匹配的函数.
此时再转而使用慢速匹配模式,进一步精确确认,从而得到想要的匹配结果.

# Func文本规则

<>代表的是普通的函数,即特征码E8????????的扩展,esig理论可以内嵌很多层<>

[]代表jmp到API函数,即特征码FF15????????的扩展,支持IAT与EAT匹配,例如[KERNEL32.EnterCriticalSection||RtlEnterCriticalSection]

<[]>代表Call到API函数,即特征码FF25????????的扩展,支持IAT与EAT匹配,例如<[KERNEL32.EnterCriticalSection||RtlEnterCriticalSection]>

!!代表的是常量的识别,通过一些函数的对比发现,有些函数其常量会是识别的关键,例如

push 0x401000,而0x401000中的"string too long"会将成为识别的关键,此时可以编写特征码

SubFunc添加子函数
string:737472696E6720746F6F206C6F6E67
Func中规则
68!string!

来表示这个重要的常量

-->代表长跳转,即特征码E9????????的扩展,在匹配时会跳转过去继续识别,暂不支持短跳转

