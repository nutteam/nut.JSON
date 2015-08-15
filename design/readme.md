#JSON 溢出
这是一个实际生产中的问题，数据的id字段类型为long，通过JSON传到前端后，可能会发生显示不正常的情况,产生原因是JS的Number溢出。为了重现这个问题，可以在网页的JS控制台尝试如下的诡异语句。
````
9007199254740993==9007199254740992;//true
````

在其他语言中,long类型可以达到的最大值为 2^64

`18446744073709551616`

而在JS中,整形的最大的值为 Number.MAX_SAFE_INTEGER=2^53-1

`9007199254740991`

因此当含有long类型字段的JSON在被显示前需要进行处理，方案如下

* 服务器修改数据类型，将long转化为string传入
* 前端重写JSON.parse方法，避免Number溢出

最初我们希望服务器的开发人员能够将id字段转化为string，但是那边的开发说了一句话 *“你们怎么连显示一个long都搞不定”...*(此处省略3000字)所以我们聊聊第二种方法
###实现JSON转化
很少有人会把JSON当成一门语言，但是JSON确实是一门语言，这门语言的语法很简单 

"key":value... ，value的类型很间单:string,number,object,array,boolean,null
即使这个语言很简单，但是要自己实现语言的转化还是不容易，不过前人们已经为我们铺好了路。GitHub上有个JS实现的JSON parser项目－－[jisonlint](https://github.com/zaach/jsonlint)。我们需要在他的基础上面做一些修改
原文件 src/jsonlint.y

````
JSONNumber
    : NUMBER   {$$ = Number(yytext);}
    ;

````
修改为

````
JSONNumber
    : NUMBER  {$$ = yytext== String(Number(yytext)) ? yytext:Number(yytext);}
    ;

````

假设我们翻译`{"id":123}` 时，`：`和`｝`之间的 `123` 会被以字符串的形式抽出，这个时候我们需要对它进行转化，请对比一下句子（请仔细思考）

````
//true
Number('18446744073709551616')=='18446744073709551616' 

//true
String(18446744073709551616)=='18446744073709552000' 

//true
18446744073709551616=='18446744073709552000' 

//false
String(Number('18446744073709551616'))=='18446744073709551616' 

//true
String(Number('123'))=='123'
````
验证是否溢出的思路如下，将 'number' 转化为Number再转化为String，如果发生溢出，它在第二次转化为string的时候，字面量会发生变化。试验中只进行一次转化似乎无法达到检测溢出的效果。

###输出JS文件
项目中的语法文件还需要经过JISON 转换成js可执行文件，用node命令行

````
$ npm install jison -g
$ jison src/grammar.y -o myParser.js

````










