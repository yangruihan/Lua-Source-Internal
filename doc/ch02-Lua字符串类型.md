# Lua字符串

## 1. Lua 字符串数据结构定义

首先来看Lua中表示字符串的数据结构定义:

```c
/* (lobject.h) */

/*
** Header for string value; string bytes follow the end of this structure
** (aligned according to 'UTString'; see next).
*/
typedef struct TString {
  CommonHeader;
  lu_byte extra;  /* reserved words for short strings; "has hash" for longs */
  lu_byte shrlen;  /* length for short strings */
  unsigned int hash;
  union {
    size_t lnglen;  /* length for long strings */
    struct TString *hnext;  /* linked list for hash table */
  } u;
} TString;

/*
** Ensures that address after this type is always fully aligned.
*/
typedef union UTString {
  L_Umaxalign dummy;  /* ensures maximum alignment for strings */
  TString tsv;
} UTString;
```

可以看见，Lua 这里有两个结构：

1. `TString`：

    其中字段基本都有注释，也比较容易理解

    - `CommonHeader`：gc 对象通用部分
    - `extra`：对于短字符串来说为表示保留字符串，对长字符串 来说表示是否计算过 hash
    - `shrlen`：表示短字符串的长度，对长字符串无意义
    - `hash`：表示该字符串的 hash 值，如果是短字符串，则该 值在创建时就计算好，因为短字符串会被添加到全局的字符串表 中，避免重复创建；而对于长字符串，该值并不会立即计算，而 是在需要它的时候再进行计算，计算函数为 `luaS_hashlongstr (TString *ts) // lstring.c`， 一旦计算过，便会将上面提到的`extra`字段设置为1，避免重复 计算
    - `union { lnglen; hnext; }`：对于短字符串来说，`lnglen`没有意义，由于该串将被加入到全局的字符串表中，因此`hnext`表示表中下一个串，对于长字符串来说，`hnext`没有意义，`lnglen`表示长字符串的长度，这里长字符串和短字符串之所以没有用同一个字段来表示，是因为长字符串长度可能非常长，然后短字符串最长为40

2. `UTString`

    可以看见是一个 union，其目的是为了让`TString`数据类型 按照`L_Umaxalign`类型来进行对齐

    ```c
    /* type to ensure maximum alignment */
    #if defined(LUAI_USER_ALIGNMENT_T)
    typedef LUAI_USER_ALIGNMENT_T L_Umaxalign;
    #else
    typedef union {
       lua_Number n;
       double u;
       void *s;
       lua_Integer i;
       long l;
    } L_Umaxalign;
    #endif
    ```

    C语言中，struct/union 这样的复合数据类型，是按照这个类型中最大对齐量的数据来进行对齐的，所以这里就是按照double 类型的对齐量来进行对齐，一般而言是8字节。

    而在结构体`UTString`中，其最大的对齐单位肯定不会比 double 大，所以整个`UTString` union 是按照 double 的对齐量来进行对齐的。

    可以从`UTString`的注释中得知，这样设计的目的是为了保证后面紧跟着的字符串 bytes 内存始终是满对齐

## 2. 长串和短串

   从上面看出，Lua 内部对字符串的实现分成了短字符串和长字符串，对它们俩的操作也不尽相同。比如创建一个短字符串首先会查询全局字符串表，如果已经存在了，则直接复用，否则再进行创建，而创建一个长字符串，则直接创建，允许冗余。那么为什么要这么做呢，基本上是因为如下理由：

   1. 复用性：显而易见的短字符串的重复度会比长字符串高很多，由于 table 是 Lua 唯一的数据结构，字符串又作为 table 非常重要的键，无论是使用 table 访问其键对应的值，还是在其他地方使用，

在Lua中，所有字符串是一个保存在一个全局的地方，在global_state的strt里面，这是一个hash数组，专门用于存放字符串:

    (lstate.h)
     38 typedef struct stringtable {
     39   GCObject **hash;
     40   lu_int32 nuse;  /* number of elements */
     41   int size;
     42 } stringtable;
 
 一个字符串TString，首先根据hash算法算出hash值，这就是stringtable中hash的索引值，如果这里已经有元素，则使用链表串接起来.
 
 同时，TString中的字段reserved，表示这个字符串是不是保留字符串，比如Lua的关键字，在最开始赋值的时候是这么处理的:
 
     (llex.c)
     64 void luaX_init (lua_State *L) {
     65   int i;
     66   for (i=0; i<NUM_RESERVED; i++) {
     67     TString *ts = luaS_new(L， luaX_tokens[i]);
     68     luaS_fix(ts);  /* reserved words are never collected */
     69     lua_assert(strlen(luaX_tokens[i])+1 <= TOKEN_LEN);
     70     ts->tsv.reserved = cast_byte(i+1);  /* reserved word */
     71   }
     72 }
     
 这里存放的值，是数组luaX_tokens中的索引.这样一方面可以迅速定位到是哪个关键字，另方面如果这个reserved字段不为0，则表示该字符串是不可自动回收的，在GC过程中会略过这个字符串的处理:
 
     (llex.c)
      36 /* ORDER RESERVED */
     37 const char *const luaX_tokens [] = {
     38     "and"， "break"， "do"， "else"， "elseif"，
     39     "end"， "false"， "for"， "function"， "if"，
     40     "in"， "local"， "nil"， "not"， "or"， "repeat"，
     41     "return"， "then"， "true"， "until"， "while"，
     42     ".."， "..."， "=="， ">="， "<="， "~="，
     43     "<number>"， "<name>"， "<string>"， "<eof>"，
     44     NULL
     45 }; 
     
这里的每个字符串都是与某个保留字Token类型一一对应的:

    20 /*
     21 * WARNING: if you change the order of this enumeration，
     22 * grep "ORDER RESERVED"
     23 */
     24 enum RESERVED {
     25   /* terminal symbols denoted by reserved words */
     26   TK_AND = FIRST_RESERVED， TK_BREAK，
     27   TK_DO， TK_ELSE， TK_ELSEIF， TK_END， TK_FALSE， TK_FOR， TK_FUNCTION，
     28   TK_IF， TK_IN， TK_LOCAL， TK_NIL， TK_NOT， TK_OR， TK_REPEAT，
     29   TK_RETURN， TK_THEN， TK_TRUE， TK_UNTIL， TK_WHILE，
     30   /* other terminal symbols */
     31   TK_CONCAT， TK_DOTS， TK_EQ， TK_GE， TK_LE， TK_NE， TK_NUMBER，
     32   TK_NAME， TK_STRING， TK_EOS
     33 };
 
需要说明的是，上面luaX_tokens字符串数组中的"\<number>"， "\<name>"， "\<string>"， "\<eof>"这几个字符串并不真实做为Lua语言中的保留关键字存在，但是因为有相应的保留字Token类型，所以也就干脆这么定义一个对应的字符串了.
