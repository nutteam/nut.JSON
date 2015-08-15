# nut-JSON
-----
This is a JSON parser to solve Number overflow use [JISON Project](http://zaach.github.io/jison) 

##PROMBLE
if you evaluate 

`JSON.parse('{"id":10446744073709551616}')` 

you will get

`{id:10446744073709552000}`

 To make the id display collectly you need to parse JSON yourself.If the id field is just for display,it is ok to parse it as string. so
 
 `nutParser.parse('{"id":10446744073709551616}').id=='10446744073709551616'`
 
 For more detail see the [design file](./design/readme.md)

##REBUILD
If you want to modify the source and rebuild the parser.
Run node command

````
    $ npm install jison@0.4.13 -g
    $ jison src/grammar.y -o myParser.js

````

