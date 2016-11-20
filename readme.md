# Asynchronous Blocks

ECMAScript proposal, specs, tests, and reference implementation for Asynchronous locks

This initial proposal was drafted by @johnhenry with input from @_ and @_.

Designated TC39 reviewers: @_ @_

## <a name="table-of-contents"></a>Table of Contents

- [Table of Contents](#table-of-contents)
- [Status](#status)
- [Prior Art](#prior-art)
- [Rationale and Motivation](#rationale-motivation)
- [Proposed Solution: Asynchronous Blocks](#proposed-solution)
  - [other operators](#proposed-solution:other-operators)
    - [for/for-in/for-of](#proposed-solution:other-operators:for-for-for)
    - [for-await-fo](#proposed-solution:other-operators:for-await)
    - [while](#proposed-solution:other-operators:while)
    - [do](#proposed-solution:other-operators:do)
- [Specification](#specification)
  - [Transformation](#specification:transformation)

## <a name="status"></a>Status

This proposal is currently at stage 0 of the process.

## <a hame="prior-art"></a>Prior Art

Some of the syntaxes in this proposal, namely the "do await" syntax, have been [discussed before](https://github.com/tc39/ecmascript-asyncawait/issues/9)

## <a name="rationale-motivation"></a>Rationale and Motivation

Previously, in order to preform a one-off task within a program that didn't affect the global scope, one had to use an Immediately Invoked Function Expression (IIFE).

```javascript
(function(){
  const a = 1;
  console.log(a);
  //other stuff...
})();
(function(){
  const a = 2;
  console.log(a);
  //other stuff...
})();
//logs "1", "2";
```

With blocks and blocked scoped variables, [introduced for 2015](), the syntax becomes much cleaner.

```javascript
{
  const a = 1;
  console.log(a);
  //other stuff
}
{
  const a = 2;
  console.log(a);
  //other stuff
}
console.log(a);
//logs "1", "2";
```

Currently if we want to invoke as asynchronous expression, set to be [introduced for 2017]() we must resort to using an IIFE once more.

```javascript
(async function(){
  const a = await eventually(1);
  //other stuff...
})();
(async function(){
  const a = await eventually(2);
  //other stuff...
})();
//logs eventually "1", "2" or "2", "1"
```

Alternatively, we can use non-anoymous functions and invoke them later in the program:

```javascript
const main = async function(){
  const a = await eventually(1);
  //other stuff...
};
const main2 = async function(){
  const a = await eventually(2);
  //other stuff...
}
main();
main2();
//logs eventually "1", "2" or "2", "1"
```

but this is still not as elegant as one might hope.

## <a name="proposed-solution"></a>Proposed Solution: Asynchronous Blocks

We can make this cleaner by implementing asynchronous blocks

```javascript
async {
  const a = await eventually(1);
  console.log(a);
  //other stuff
}
async {
  const a = await eventually(2);
  console.log(a);
  //other stuff
}
//logs eventually  "1", "2" or "2", "1"
```

Asynchronous blocks are executed asynchronously with respect to the surrounding context and allow use of the "await" keyword without having to create another functional scope.


### <a name="proposed-solution:other-operators"></a>other operators

This would work in conjunctions with other operators that come before blocks:

####<a name="proposed-solution:other-operators:for-for-for"></a>for/for-in/for-of
```javascript
const for(const i of [1, 2]) async {
  await eventually(i);
}
console.log(0);
//logs "0"
//logs eventually "1", "2"
```

#### <a name="proposed-solution:other-operators:for-await"></a>for-await-of

See [Asynchronous Iterator Proposal ](https://github.com/tc39/proposal-async-iteration)

```javascript
async {
  for await(const i of eventuallyIterator(1, 2)) async{
    console.log(i);
  }
  console.log(0);
}
//logs "0"
//logs eventually "1", "2"
```

#### <a name="proposed-solution:other-operators:while"></a>while
```javascript
while(true) async {
  await eventually(i++);
}
console.log(0);
//logs "0"
//logs eventually "1", "2"...
```

#### <a name="proposed-solution:other-operators:do"></a>do

See [Do-Expression Proposal](http://wiki.ecmascript.org/doku.php?id=strawman:do_expressions)

```javascript
const result = do async{
 const a = 1;
}
result.then(console.log.bind(console));
//logs eventually  "1"
```

## <a name="specification"></a>Asynchronous blocks

When Number.prototype[Symbol.iterator] is called, the following steps are taken:

  - While parsing, if the keyword "async" is encountered directly before a block, execute that block's contents asynchronously with respect to the surrounding program.
  - That block will now have the "await" keyword available at the top level.
  - Declarations using keywords "let" and "const" would not affect the scope outside of the asynchronous block.
  - Keeping in mind backward compatibility, declarations using the keyword "var", along with naked declarations, would affect the scope outside of the asynchronous block.

### <a name="specification:transformation"></a>transformation

The following transformation recursively on a string:

```javascript
const match = /^async\s*{\s*(.*)\s*}/m;
const wrap = (code) => `(async function(){${code}})()`;
export default (code)=>{
  return match.test(code) ? wrap(code.match(match)[1]) : code;
};
```

would have the following result:

```coffeescript
async { ... } -> (async function(){ ... })()
```

Although, it misses the requirement regarding "backward compatibility" above in that it does not respect the use of the "var" keyword.
