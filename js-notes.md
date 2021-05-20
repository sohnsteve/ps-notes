# JavaScript Notes

## Creating JS Objects

A direct way of creating objects in JS involves object literals, as shown below:

```javascript
let person = {
    firstName: "John",
    lastName: "Doe"
}
```

The object is assigned to the variable named `person` with two properties.

JavaScript is a dynamically-typed language which allows us to create objects 
"on the fly" like above without giving it a type (or class). This also affords 
us to add properties dynamically, as well. Let's add two new properties:

```javascript
person.age = 29;
person.isAdult = function() { return this.age >= 18; }
```

Notice that we assigned a function to a property named `isAdult`. In this case, we
refer to them as methods.

### Object Literal Property Shorthand Syntax

Creating objects and assigning property names is a common task in JS. For example:

```javascript
function registerUser(firstName, lastName) {
    let person = {
        firstName: firstName,
        lastName: lastName
    }
}
```

The above function creates a `person` object with properties `firstName` and `lastName`.
We can quickly assign properties by assuming the property names from the parameters as follows:

```javascript
function registerUser(firstName, lastName) {
    let person = {
        firstName,
        lastName
    }
}
```

While it may look confusing, the above two code blocks are exactly the same!

### Object Literal Declaration Shorthand

A function defined within an object can be shortened from:

```javascript
let person = {
    firstName: firstName,
    lastName: lastName,
    isAdult: function() { return this.age >= 18; }
}
```

to:

```javascript
let person = {
    firstName: firstName,
    lastName: lastName,
    isAdult() { return this.age >= 18; }
}
```

Again, both code blocks above are exactly the same.

## JS Equality

There are three types of equality in JS:
- `==` rarely used
- `===` most common
- `Object.is(item1, item2)` similar to `===` with a few mathematical diffrences

The following examples of `==` are all evaluate to true and highlight cases where it may be useful (but also why
it should be avoided, otherwise):

```javascript
"42" == 42
0 == false
null == undefined
"" == 0
[1,2] == "1,2"
```

As evidenced above, `==` is NOT type-safe. The other operators are both type-safe.

`===` considers `NaN` not equal to `NaN`, whereas `Object.is()` does. Also, `===` 
considers +0 to equal -0 whereas `Object.is()` does not.

When comparing object variables, however, JS will compare the memory addresses of the object variable
names, not the content of the object. Note that JS will compare values for primitive types (and not compare 
the memory addresses).

### Object Assign

`Object.assign(toBeMerged, newProperties)` To avoid mutating original objects, it is normal practice to 
use `Object.assign()` the following way:

`Object.assign({}, toBeMerged, newProperties)` That way, the object denoted `toBeMerged` remains unmutated, but 
rather a new object is created.

### Constructor Functions

To keep consistency in JS, we can use constructor functions with the `this` keyword as well as `new`.

```javascript
function Person() {
    this.firstName = "Jim";
    this.lastName = "Cooper";
}

let person = new Person();
```

In JS, `this` always refers to an object. The object it refers to depends on the context it is used.
In the above code block, `this` is pointed by the new object created and denoted by `new`. The above 
constructor function is very restrictive, however, limiting all new `person` objects to have the name
Jim Cooper. Let's change it slightly to a form that is more commonly seen:

```javascript
function Person(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    this.isAdult = function() { return this.age > 21; }
}

let jim = new Person("Jim", "Cooper", 29);
let sofia = new Person("Sofia", "Cooper", 17);
```

The above code block shows how the constructor function can now be used more widely to create `person` 
objects with certain property values.

## Deeper Dive into Object Properties

### Bracket Notation

We can access object properties using bracket notation such that for some property `person`, the following 
calls are equivalent: `person.firstName`, `person[firstName]`.

### Object Property Attributes

Objects properties have the following attributes:

- value
- writable
- enumerable
- configurable

With `writable` properties, you can further `freeze` them to stop changes for all sub-properties.

Setting the `enumerable` property to false prevents that attribute from being accessed via loop, `Objects.keys()`, or JSON serialized.

The `configurable` property, when set to false, means its `configurable` and `enumerable` from being changed, but it can still be made `writable`. It also cannot be deleted.

### Property Getters/Setters

Example code snippet:

```javascript
let person = {
    name: {
        first: 'Jim',
        last: 'Cooper'
    },
    age: 29
};

Object.defineProperty(person, 'fullName', 
{
    get: function() {
        return this.name.first + ' ' + this.name.last;
    },
    set: function(value) {
        var nameParts = value.split(' ');
        this.name.first = nameParts[0];
        this.name.last = nameParts[1];
    }
});

person.fullName = 'Fred Jones';
```

The call `person.fullName = 'Fred Jones';` will set the `firstName` property to "Fred" and the `lastName` property to "Jones" using the setter method `fullName`.

## JS Prototypes

A **prototype** is a JS property that exists for all functions. Objects also have prototypes but not as an explicity property. Note, however, that an object's prototype and a function's prototypes are used differently.

A function's prototype is the object *instance* that will become the prototype for all objects created using this function as a constructor.

An object's prototype is the object *instance* from which the object is inherited. 

Let's explore this with the following code snippet:

```javascript
// constructor
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
}

let jim = new Person('Jim', 'Cooper');
```

In the above snippet, the prototype of `Person` and `jim` are the same, since `jim` is constructed using `Person`.

Let's explore further:

```javascript
// assume same constructor as above

Person.prototype.age = 29;

let jim = new Person('Jim', 'Cooper');

jim.age = 18;

display(jim.age); // will display 18
display(jim.__proto__.age); // will display 29
```

First we notice that although `age` was not defined in the constructor, we can add it as a prototype explicitly as shown just like properties. Next, we see that though we explicity declare the `age` property for `jim`, its `age` prototype is the same as the one it inherited from. In fact, if we left out the line `jim.age = 18`, what JS will do is look up its prototype and assign the `age` property value from that-- in this case, it will print the value 29 for the line `display(jim.age)`.

### Using Inheritance with Prototypes

Examine the following code snippet:

```javascript
function Person(firstName, lastName, age) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.age = age;
    Object.defineProperty(this, 'fullName', {
        get: function() {
            return this.firstName + ' ' + this.lastName
        },
        enumerable: true
    });
}

function Student (firstName, lastName, age) {
    Person.call(this, firstName, lastName, age);
    this._enrolledCourses = [];

    this.enroll = function(courseId) {
        this._enrolledCourses.push(courseId);
    };

    this.getCourses = function () {
        return this.fullName + "'s enrolled courses are: " + 
        this._enrolledCourses.join(', ');
    };
}

Student.prototype = Object.create(Person.prototype);
Student.prototype.constructor = Student;

let jim = new Student('Jim', 'Cooper', 29);

display(jim.__proto__); // Student prototype
display(jim.__proto__.__proto__); // Person prototype
```

## JS Classes

Should be noted that classes are syntactic sugar for what we've already done so far. That is, they don't add functionality but rather provide a cleaner syntax for everything we've done with objects so far.

Let's rewrite the `Person` object we've been dealing with so far in class syntax:

```javascript
class Person {
    constructor(firstName, lastName, age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }
}

let jim = new Person('Jim', 'Cooper', 29);
```

Note that not all properties have to be assigned a value through the constructor as arguments.

### Getters/Setters in Classes

We can create getters/setters with less syntactic clutter by simply creating get/set functions. For example:

```javascript
// assume inside Person class

get fullName() {
    return this.firstName + ' ' + this.lastName;
}

set fullName(fullName) {
    var nameParts = value.split(' ');
    this.name.first = nameParts[0];
    this.name.last = nameParts[1];
}

// ...

let jim = new Person('Jim', 'Cooper', 29);
display(jim.fullName); // will print "Jim Cooper"
jim.fullName = 'Fred Jones';
display(jim.fullName); // will print "Fred Jones"
```

### Adding Functions

Adding functions to classes are straightfoward. For example:

```javascript
// assume inside Person class
isAdult() {
    return this.age >= 18;
}
```

### Inheritance in Classes

Classes can inherit using the **extends** keyword. To invoke a parent's constructor, use the **super** keyword. For example:

```javascript
class Student extends Person {
    constructor(firstName, lastName, age) {
        super(firstName, lastName, age);
        this._enrolledCourses = [];
    }

    // ...
}
```

### Static Properties and Methods

Methods that are not depedent on instances of a class are called **static** methods.

```javascript
// assume in Student class

static fromPerson(person) {
    return new Student(person.firstName, person.lastName, person.age);

// outside the Student class:

let jim = new Person('Jim', 'Cooper', 29);
let jimStudent = Student.fromPerson(jim);
}
```

Properties that don't change are **static** properties. The Math class, for example, makes use of static methods and properties since there are many constants and formulas that don't change.

## Built-in JS Objects

Math, Date, Regex

There are a few useful built-in JS objects provided by JS. For example, we can tap into common math functions like `Math.pow()`. Similarly, the `Date` object provides a useful and universal way with dealing with time.

Common pitfall of the month property from `Date`: it is zero-based! (i.e., January is the 0th month, Feb is 1, etc.)

### Regex

We can call upon the Regex function as follows, `let regex = new RegExp()`, of course, passing into it a regular expression.

To test a regex, we can use the `test()` method as follows: `regex.test(test);`

A useful regex function is the `exec()` method which is a stateful method and returns an array. Capture groups are used to capture multiple "things" related to what is being caught in the regex. Because it is stateful, subseqent calls will return further regex's. Therefore, it is useful to put it in a loop and capture all instances of the regex until it returns null.


## Questions

In the following block of code:

```javascript
(function() {
    let person = {
        firstName: 'Jim',
        lastName: 'Cooper',
        age: 29
    };

    let propertyName = 'firstName';

    display(person[propertyName]);
})();
```

Why is `propertyName` tied to object `person`?