# lab1实验报告

**学号：202308010412**  
**姓名：王宇航**

## 一、实验要求

本次实验需要根据`cminux-f`的词法补全[lexical_analyer.l](../../src/lexer/lexical_analyzer.l)文件，完成词法分析器，能够输出识别出的`token`，`type` ,`line(刚出现的行数)`，`pos_start(该行开始位置)`，`pos_end(结束的位置,不包含)`。如：

文本输入：

```c
 int a;
```

则识别结果应为：

```shell
int     280     1       2       5
a       285     1       6       7
;       270     1       7       8
```
**具体的需识别token参考[lexical_analyzer.h](../../include/lexical_analyzer.h)**

**特别说明对于部分token，我们只需要进行过滤，即只需被识别，但是不应该被输出到分析结果中。因为这些token对程序运行不起到任何作用。**

> 注意，修改的文件应仅有[lexical_analyer.l](../../src/lexer/lexical_analyzer.l)。关于`FLEX`用法上文已经进行简短的介绍，更高阶的用法请参考百度、谷歌和官方说明。

## 二、实验难点
### 1. 环境配置
**实验环境：Ubuntu 20.04**   

**Tips** .安装LLVM可以直接在Linux下命令行输入  

`sudo apt-get install llvm bison flex`

![](../../Documentations/common/figs/llvm-install.png)

安装完成后可以输入

`flex --version`

`bison --version`

查看版本

### 2. FLEX 基础用法
什么是flex？  
* flex是指 fast lexical analyzer generator,快速词法分析器生成器，也就是说，flex用于产生词法分析器；

  - flex的输入是文件或输入设备，这些输入中的信息以正则表达式和C代码的形式组成，这些形式被称为规则（rule）；
  - flex的默认输出是C语言的源码文件：lex.yy.c，也可以重命名；该文件通过编译生成可执行的文件；  
  
* 当可执行文件被执行时，其分析输入中可能存在的符合规则的内容，当找到任何一个正则表达式相匹配内容时，相应的C代码将被执行；

flex的输入文件由3段组成，用一行中只有%%来分隔；  
```bash
定义:definition
%%
规则:rules
%%
用户代码:code
```
### 3.git 基础用法
**1. 获取项目到本地的工作目录**  

```bash
htps://gitee.com/你的gitee用户名/cminus_compiler-2021-fall.git
```

“你的gitee用户名”是打开个人主页，昵称下面有个“@……”，用户名就是@后面的一串字符；或者打开仓库，在最上边的https://gitee.com……也可以看到。

**2.将工作上传至git仓库**

打开本地的工作目录，在命令行中输入

```bash
git add *

git commit -m "注释语句"
```

**3. 然后push到仓库**

`git push`



## 三、实验设计
### 1. token定义
```c
typedef enum cminus_token_type {
    //运算
    ADD = 259,
    SUB = 260,
    MUL = 261,
    DIV = 262,
    LT = 263,
    LTE = 264,
    GT = 265,
    GTE = 266,
    EQ = 267,
    NEQ = 268,
    ASSIN = 269,
    //符号
    SEMICOLON = 270,
    COMMA = 271,
    LPARENTHESE = 272,
    RPARENTHESE = 273,
    LBRACKET = 274,
    RBRACKET = 275,
    LBRACE = 276,
    RBRACE = 277,
    //关键字
    ELSE = 278,
    IF = 279,
    INT = 280,
    FLOAT = 281,
    RETURN = 282,
    VOID = 283,
    WHILE = 284,
    //ID和NUM
    IDENTIFIER = 285,
    INTEGER = 286,
    FLOATPOINT = 287,
    ARRAY = 288,
    LETTER = 289,
    //others
    EOL = 290,
    COMMENT = 291,
    BLANK = 292,
    ERROR = 258

} Token;
```

**这是 C-Minus 编译器的词法分析阶段使用的令牌类型定义，用于：**

- **Flex/Lex 词法分析器返回令牌类型**

- **语法分析器识别不同的语法元素**

- **错误处理和语法检查**

### 2. token对应的正则表达式
```c
 /******** 运算 ********/
\+   {pos_start = pos_end; pos_end++; return ADD;}
\-   {pos_start = pos_end; pos_end++; return SUB;}
\*   {pos_start = pos_end; pos_end++; return MUL;}
\/   {pos_start = pos_end; pos_end++; return DIV;}
\<   {pos_start = pos_end; pos_end++; return LT;}
"<=" {pos_start = pos_end; pos_end+=2; return LTE;}
\>   {pos_start = pos_end; pos_end++; return GT;}
">=" {pos_start = pos_end; pos_end+=2; return GTE;}
"==" {pos_start = pos_end; pos_end+=2; return EQ;}  
"!=" {pos_start = pos_end; pos_end+=2; return NEQ;}
\=   {pos_start = pos_end; pos_end++; return ASSIN;}
​
/******** 符号 ********/
\;   {pos_start = pos_end; pos_end++; return SEMICOLON;}
\,   {pos_start = pos_end; pos_end++; return COMMA;}
\(  {pos_start = pos_end; pos_end++; return LPARENTHESE;}
\)  {pos_start = pos_end; pos_end++; return RPARENTHESE;}
\[  {pos_start = pos_end; pos_end++; return LBRACKET;}
\]  {pos_start = pos_end; pos_end++; return RBRACKET;}
\{  {pos_start = pos_end; pos_end++; return LBRACE;}
\}  {pos_start = pos_end; pos_end++; return RBRACE;}
​
 /******** 关键字 ********/
else {pos_start = pos_end; pos_end+=4; return ELSE;}
if   {pos_start = pos_end; pos_end+=2; return IF;}
int  {pos_start = pos_end; pos_end+=3; return INT;}
float {pos_start = pos_end; pos_end+=5; return FLOAT;}
return {pos_start = pos_end; pos_end+=6; return RETURN;}
void   {pos_start = pos_end; pos_end+=4; return VOID;}
while  {pos_start = pos_end; pos_end+=5; return WHILE;}
​
/******** ID和NUM ********/
[a-zA-Z]+ {pos_start = pos_end; pos_end+=strlen(yytext); return IDENTIFIER;}
[a-zA-Z]  {pos_start = pos_end; pos_end++; return LETTER;}  
[0-9]+    {pos_start = pos_end; pos_end+=strlen(yytext); return INTEGER;}
[0-9]+\.|[0-9]*\.[0-9]+{pos_start=pos_end;pos_end+=strlen(yytext);return FLOATPOINT;}
"[]"      {pos_start = pos_end; pos_end+=2; return ARRAY;}
​
/******** others ********/
\n  {return EOL;} 
"/*"([^*]|\*+[^*/])*\*+"/"  {return COMMENT;}
[" "|\t] {pos_start = pos_end; pos_end+=strlen(yytext);return BLANK;}
. {return ERROR;}
```

### 3. 补全辅助函数
```c
while(token = yylex()){
        switch(token){
            case COMMENT: //注释
        len = strlen(yytext);
        for(int i=0;i<len;i++){//循环判断注释中是否存在换行符
           if(yytext[i]=='\n') {//yytext[i]是换行\n
               lines++;{    //则将lines加1；
               pos_end = 1; //将pos_start和pos_end设置为1
            }
            else pos_end++;  //否则pos_end++;
           }
            break;  //当循环结束则break;
  case BLANK:
         break;
  case EOL:
         lines++;
         pos_end = 1;
         break;
```
**这段代码是词法分析器的词法单元处理循环，主要功能是：**
1. **注释处理 (COMMENT)**    
**功能：统计注释中的换行符数量，更新行号和位置计数器。**
1. **空白符处理 (BLANK)**

2. **行结束符处理 (EOL)**

3. **核心变量作用**
- token：当前读取的词法单元类型

- yytext：当前此法单元对应的文本内容

- lines：行号计数器，遇到换行符时递增

- pos_end：位置计数器，记录当前行内的字符位置
## 四、实验结果验证

### 1. 编译运行

**通过下列指令进行编译运行**
* 编译

  ```shell
  # 进入workspace
  $ cd cminus_compiler-2022-fall
  
  # 创建build文件夹，配置编译环境
  $ mkdir build 
  $ cd build 
  $ cmake ../
  
  # 开始编译
  # 如果你只需要编译lab 1，请使用 make lexer
  $ make
  ```

  编译成功将在`${WORKSPACE}/build/`下生成`lexer`命令

* 运行

  ```shell
  $ cd cminus_compiler-2022-fall
  # 运行lexer命令
  $ ./build/lexer
  usage: lexer input_file output_file
  ```
* 个人编译运行结果：
![编译运行](../../Documentations/common/figs/compile-run.png)


### 2. 助教测试案例
我们提供了`./tests/lab1/test_lexer.py` python脚本用于调用`lexer`批量完成分析任务。

  ```shell
  # test_lexer.py脚本将自动分析./tests/lab1/testcase下所有文件后缀为.cminus的文件，并将输出结果保存在./tests/lab1/token文件下下
  $ python3 ./tests/lab1/test_lexer.py
  	···
  	···
  	···
  #上诉指令将在./tests/lab1/token文件夹下产生对应的分析结果
  $ ls ./tests/lab1/token
  1.tokens  2.tokens  3.tokens  4.tokens  5.tokens  6.tokens
  ```

**运行Python脚本并对比结果：**  
![](../../Documentations/common/figs/python-run.png)

由结果可见，可成功生成token文件，且在对比助教提供的正确版本后发现结果相同

### 3. 个人测试案例


## 五、实验反馈
