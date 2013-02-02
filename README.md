# sqlbits

```javascript
var bits = require('sqlbits'), $=bits.$,SELECT=bits.SELECT,BETWEEN=bits.BETWEEN,AND=bits.AND;

var query =
  SELECT('id,first,last,age')
		.FROM('peeps')
		.WHERE("first=",$("Tom"), AND, "age",BETWEEN(18, 25))
		.ORDERBY('last')

console.log(query+";");
//SELECT id,first,last,age FROM peeps WHERE(first=$1 AND age BETWEEN($2 AND $3)) ORDER BY last;
console.log(query.params);
//[ 'Tom', 18, 25 ]
```


### The "bits"
The project is named `Sqlbits` because it aims to only provide _**bits of help**_ with building SQL query strings. You can use a `bit` to generate pieces of a query **or** in conjunction with other `bits` to form a complete query... if that's what raises your flag.

There are 3 types of `bits`: [Contexts](#contexts), [Statements](#statements), [Params](#params)

### undefined values
By design, most statements **will not** output if all associated parameters are undefined. This example omits the first `AND` statement because the `IN` [Param](#params) only has `undefined` values:
```javascript
var bits = require('sqlbits'),
	x, y, z;//intentionally undefined

with(bits)//GASP! OMG! ..<rant>The Crock can bite me</rant>

console.log(
	SQL(    AND("id", IN(x,y,z), OR("1=1")    ))+""
);
//AND(1=1)
```


### Syntactic Sugar
> In computer science, syntactic sugar is syntax within a programming 
> language that is designed to make things easier to read or to express.
-[Wikipedia](http://en.wikipedia.org/wiki/Syntactic_sugar)

That's all `Sqlbits` is. No ORM. No database connectivity. Just sugar.

We all know about SQL injection (hopefully). Concatenating strings with parameters isn't hard- but 
things can get ugly fast. Consider queries that return data based on different conditions. This is 
very common for reports. For example: "transactions for product $1 between $2 & $3 for user $4" but 
you want these conditions to be optional. Concatenating a query like that (AND keeping track of the 
params) gets ugly. This is why `Sqlbits` omits things when their params are [undefined](#undefinedvalues).
```javascript
```


### Chaining
`Bits` are chainable with other `bits`, but no effort is made to ensure that the chained `bits` form logical output. For instance:
```javascript
var bits = require('sqlbits')
    SELECT = bits.SELECT;

var query = SELECT("%@!!").ORDERBY('seawead').ON('top of old smoky');

console.log(query+";");
//SELECT %@!! ORDER BY seawead ON top of old smoky;
```

Perhaps in some future world that output will be desired.


### Installation
    $ npm install sqlbits



## Contexts
A `Context` object manages the `sql` output and the `params` Array for what you're doing. [Statements](#statements) 
and [Params](#params) need a `Context` to operate.

This project comes with a PostgreSQL context and output generator, [PGContext](#pgcontext), which inherits from the `Context` base Object. It was abstracted to allow for the possibility of other output formaters.

All Context objects will have `sql` and `params` getters.
`Context.toString()` proxies to the `sql` getter.

There are several ways to start a context. You can use the [SQL](#sql), [SELECT](#select), [INSERT.INTO](#insertinto), 
[DELETE.FROM](#deletefrom), and [UPDATE](#update) functions... or just create one with `new PGContext()`.



### SQL([arg][,arg]...)
As mentioned, `sqlbits` is here to just provide help. The `SQL` function frees you from _*needing*_ other bits. Observe this example that uses MSSQL's `TOP N` statement which Postgres does not have...

```javascript
var bits = require('sqlbits'), SQL=bits.SQL, $=bits.$, AND=bits.AND, BETWEEN=bits.BETWEEN
n = 100;

var ctx = new bits.PGContext();
ctx.createParam = function(idx){ return '@p'+idx; };

var query = ctx.SQL("SELECT TOP ", $(n), "FROM books WHERE author LIKE 'Tom%'", AND, "rating", BETWEEN(4.4, 5));

console.log(query+";");
//SELECT TOP @p1 FROM books WHERE author LIKE 'Tom%' AND rating BETWEEN(@p2 AND @p3);
console.log(query.params);
//[ 100, 4.4, 5 ]
```



### SELECT([columns])
Creates a [Context](#context) that starts with a `SELECT` statement. If the columns argument is `undefined` the `SELECT` statement will not output.
```javascript
var bits = require('sqlbits'), SELECT=bits.SELECT;

console.log(    SELECT("foo,bar,baz")+""   );
//SELECT foo,bar,baz

console.log(    SELECT().FROM("foo")+""   );
//FROM foo
```



### UPDATE([table])
Creates a [Context](#context) that starts with a `UPDATE` statement. If the table argument is `undefined` the `UPDATE` statement will not output.
```javascript
var bits = require('sqlbits'), UPDATE=bits.UPDATE,
	users;//intentionally undefined

console.log(    UPDATE("passwords")+""   );
//UPDATE passwords

console.log(    UPDATE(users).SET({first:"Nic",last:"Wicks"})+""   );
//SET first=$1,last=$2
```

### DELETE.FROM(table)
### INSERT.INTO(table, values)

## PGContext
```javascript
var bits = require('sqlbits'), $=bits.$,
    ctx = new bits.PGContext();

//You can call toString() on any chain started
//off of ctx to get that portion of the sql output
console.log(  ctx.AND("1=", $(1)).toString()  );
//AND 1=$1

console.log(  ctx.AND("2=", $(33)).OR("3=", $(99)).toString()  );
// AND 2=$2 OR 3=$3

console.log(  ctx.AND("4=", $(1)).toString()  );
// AND 4=$1

//Calling toString() or ctx.sql will return the entire sql string.
console.log(ctx.sql, ctx.params);
//AND 1=$1 AND 2=$2 AND 3=$1 [ 1, 33 ]
```


## Statements

### AND/OR/WHERE/ON([expression[,param]][,bit][,bit]...)
(These statements all work the same)

### SET(data)
### FROM(table)
### 

## Params
### $(value)
### IN(array)
### IN([value][,value]...)

### BETWEEN(min,max)


