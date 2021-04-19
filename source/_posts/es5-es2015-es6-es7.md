---
title: es5-es6-es7
date: 2021-04-14 21:14:48
tags:
---

# es6

## Object.defineProperty

[MDN doc](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) 

> taken from stackoverflow

Property is an OOP feature designed for clean separation of client code. For example, in some e-shop you might have objects like this:
```js
function Product(name,price) {
  this.name = name;
  this.price = price;
  this.discount = 0;
}

var sneakers = new Product("Sneakers",20); // {name:"Sneakers",price:20,discount:0}
var tshirt = new Product("T-shirt",10);  // {name:"T-shirt",price:10,discount:0}
```
Then in your client code (the e-shop), you can add discounts to your products:
```js
function badProduct(obj) { obj.discount+= 20; ... }
function generalDiscount(obj) { obj.discount+= 10; ... }
function distributorDiscount(obj) { obj.discount+= 15; ... }
```
Later, the e-shop owner might realize that the discount can't be greater than say 80%. Now you need to find EVERY occurrence of the discount modification in the client code and add a line
```js
if(obj.discount>80) obj.discount = 80;
```
Then the e-shop owner may further change his strategy, like "if the customer is reseller, the maximal discount can be 90%". And you need to do the change on multiple places again plus you need to remember to alter these lines anytime the strategy is changed. This is a bad design. That's why encapsulation is the basic principle of OOP. If the constructor was like this:
```js
function Product(name,price) {
  var _name=name, _price=price, _discount=0;
  this.getName = function() { return _name; }
  this.setName = function(value) { _name = value; }
  this.getPrice = function() { return _price; }
  this.setPrice = function(value) { _price = value; }
  this.getDiscount = function() { return _discount; }
  this.setDiscount = function(value) { _discount = value; } 
}
```
Then you can just alter the getDiscount (accessor) and setDiscount (mutator) methods. The problem is that most of the members behave like common variables, just the discount needs special care here. But good design requires encapsulation of every data member to keep the code extensible. So you need to add lots of code that does nothing. This is also a bad design, a boilerplate antipattern. Sometimes you can't just refactor the fields to methods later (the eshop code may grow large or some third-party code may depend on the old version), so the boilerplate is lesser evil here. But still, it is evil. That's why properties were introduced into many languages. You could keep the original code, just transform the discount member into a property with get and set blocks:
```js
function Product(name,price) {
  this.name = name;
  this.price = price;
//this.discount = 0; // <- remove this line and refactor with the code below
  var _discount; // private member
  Object.defineProperty(this,"discount",{
    get: function() { return _discount; },
    set: function(value) { _discount = value; if(_discount>80) _discount = 80; }
  });
}

// the client code
var sneakers = new Product("Sneakers",20);
sneakers.discount = 50; // 50, setter is called
sneakers.discount+= 20; // 70, setter is called
sneakers.discount+= 20; // 80, not 90!
alert(sneakers.discount); // getter is called
```
Note the last but one line: the responsibility for correct discount value was moved from the client code (e-shop definition) to the product definition. The product is responsible for keeping its data members consistent. Good design is (roughly said) if the code works the same way as our thoughts.

So much about properties. But javascript is different from pure Object-oriented languages like C# and codes the features differently:

In C#, transforming fields into properties is a breaking change, so public fields should be coded as Auto-Implemented Properties if your code might be used in the separately compiled client.

In Javascript, the standard properties (data member with getter and setter described above) are defined by accessor descriptor (in the link you have in your question). Exclusively, you can use data descriptor (so you can't use i.e. value and set on the same property):

- accessor descriptor = get + set (see the example above)
    - get must be a function; its return value is used in reading the property; if not specified, the default is undefined, which behaves like a function that returns undefined
    - set must be a function; its parameter is filled with RHS in assigning a value to property; if not specified, the default is undefined, which behaves like an empty function
- data descriptor = value + writable (see the example below)
    - value default undefined; if writable, configurable and enumerable (see below) are true, the property behaves like an ordinary data field
    - writable - default false; if not true, the property is read only; attempt to write is ignored without error*!
Both descriptors can have these members:

- configurable - default false; if not true, the property can't be deleted; attempt to delete is ignored without error*!
- enumerable - default false; if true, it will be iterated in for(var i in theObject); if false, it will not be iterated, but it is still accessible as public
* unless in strict mode - in that case JS stops execution with TypeError unless it is caught in try-catch block

To read these settings, use Object.getOwnPropertyDescriptor().

Learn by example:
```js
var o = {};
Object.defineProperty(o,"test",{
  value: "a",
  configurable: true
});
console.log(Object.getOwnPropertyDescriptor(o,"test")); // check the settings    

for(var i in o) console.log(o[i]); // nothing, o.test is not enumerable
console.log(o.test); // "a"
o.test = "b"; // o.test is still "a", (is not writable, no error)
delete(o.test); // bye bye, o.test (was configurable)
o.test = "b"; // o.test is "b"
for(var i in o) console.log(o[i]); // "b", default fields are enumerable
```
If you don't wish to allow the client code such cheats, you can restrict the object by three levels of confinement:

- Object.preventExtensions(yourObject) prevents new properties to be added to yourObject. Use Object.isExtensible(<yourObject>) to check if the method was used on the object. The prevention is shallow (read below).
- Object.seal(yourObject) same as above and properties can not be removed (effectively sets configurable: false to all properties). Use Object.isSealed(<yourObject>) to detect this feature on the object. The seal is shallow (read below).
- Object.freeze(yourObject) same as above and properties can not be changed (effectively sets writable: false to all properties with data descriptor). Setter's writable property is not affected (since it doesn't have one). The freeze is shallow: it means that if the property is Object, its properties ARE NOT frozen (if you wish to, you should perform something like "deep freeze", similar to deep copy - cloning). Use Object.isFrozen(<yourObject>) to detect it.

You don't need to bother with this if you write just a few lines fun. But if you want to code a game (as you mentioned in the linked question), you should care about good design. Try to google something about antipatterns and code smell. It will help you to avoid situations like "Oh, I need to completely rewrite my code again!", it can save you months of despair if you want to code a lot.