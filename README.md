## Preface

The aim of these coding tips is to make our life easier as developers. Here I highly recommend the book "The Art of Readable Code". 

## General

### Do Not Reinvent the Wheel

Before starting your implementation, browse through the current code base to see if anything can be reused. Use libraries and frameworks whenever possible.

### Keep the Updated Information in One Place

Some developers believe in **documentation** and **comments**. In reality, however, most of time these just become reading overhead to developers. One reason is that the documentation or comment is probably not updated, and even it is updated, it is possible that the implementation is buggy. As a result, developers need to, or rather prefer looking at the code to figure out what is going on. The mismatched information makes the documentation not as useful as you think. Here are some tips to avoid lengthy documentation that nobody actually wants to read.

* Create intuitive user interface, use tooltip to explain the functionality in-place.
* If tooltip is not enough, create a help menu and place the documentation in there. It's more helpful there because both developers and users can read it directly. As a result, the information could be more updated as well.
* Write clean and readable code, so that you can write less comments and documentation to explain what is going on.

### Do Not Write Long Methods

* It can be inefficient and time consuming to look through several hundreds / thousands lines of code to understand how method works while debugging. Keep your method below 100 lines.
* In practice, the method might be alright when it is first written, but numerous modifications and additional logic later turns it into a giant monster. We can eliminate this by wrapping the additional logic into a new method if possible, and make a call from the existing function.
* In addition, please use line breaks to separate the code blocks if they are logically independent, make it easy to read.

### Do Not Repeat Yourself

Do not simply copy and paste code. There are plenty of ways to reuse the existing code, such as inheritance, polymorphism, composition, overloading, etc.

**Task:** Reuse the logic in this function with a new parameter c

```JavaScript
function foo(a, b) {
    // logic here
}
```

_Recommended:_ Move logic to the new function, make a reference from existing function
```JavaScript
function foo(a, b) {
    foo(a, b, null);
}

function foo(a, b, c) {
    // some logic here
    if (c != null) {
        // deal with c
    }
    // other logic here
}
```

_Avoid:_ Copy and paste the logic into the new function. Later if others need to modify the logic, they have to do it twice.
```JavaScript
function foo(a, b) {
    // logic here
}

function foo(a, b, c) {
    // same logic duplicated here
    // deal with c
}
```

### Do Not Hard Code Constants

_Recommended:_ Make use of enums. In case you need to make changes, you only need to change in one place.
```Java
public enum SwitchStatusEnum {
    OFF(0),
    LOW(1),
    MID(2),
    HIGH(3);

    private int val;

    SwitchStatusEnum(int val) {
        this.val = val;
    }

    public int getVal() {
        return val;
    }
}

// Usage
int switchStatusVal = SwitchStatusEnum.MID.getVal();

// Another Usage
SwitchStatusEnum switchStatus = SwitchStatusEnum.MID;

// More Usage
public void foo(SwitchStatusEnum switchStatus) {

    switch(switchStatus) {
        case LOW:
            // logic here
            break;
        case HIGH:
            // logic here
            break;
        ...
        default:
            break;
    }
}
```

_Avoid:_ Hard coding of the constants. The reader needs to dig out what the number means by looking through every piece of information like code, comment, documentation, database... It's just simply a waste of time.
```
// What is 2?
int switchStatus = 2;

// What are these numbers?
public void foo(int switchStatus) {

    if (switchStatus == 1) {
        // logic here
    } else if (switchStatus == 3) {
        // logic here
    } else {
        // logic here
    }
}
```

### Use Consistent Variable Name
Some developer may think that assigning different variable names in different context adds certain value to the system. Well it may be the case, but on the other hand it makes the program error prone and difficult to trace while reading and debugging. If the benefit is not significant, please keep the variable name consistent across DB, Java and JavaScript.

_Recommended:_ Keep the naming consistent.
```Java
// Java Entity
@Column(name = "CST_NM")
private String cstNm;

// JavaScript
var cstNm;
```

_Avoid:_ Use different naming here and there. You will soon fall into a big trouble which you need to refer back and forth for the difference in naming.
```Java
// Java Entity
@Column(name = "CST_NM")
private String customerName;

// JavaScript
var cstName;
```

### Use Correct Abbreviations
Try to use abbreviations which are commonly accepted. If you are uncertain, search for its usage [online](http://www.abbreviations.com).
While it may be acceptable if it is rarely used, it is certainly misleading if it actually means another thing.

## Work Related

### Commit Code
* Test it at least once before committing.
* **DO NOT** commit code just before you leave for long holidays, unless it's really urgent. Your possible compile errors and bugs will last and affect all developers until you return and fix them.

### Tab VS Space
* There is always a ongoing debate on which to use. Personally, I prefer Tab because you just need to press once to delete, but I cannot stop Space supporters from using it.
* However, the point is, no matter which one you prefer, **DO NOT** modify the current format of the file if it's already in repository, e.g. converting all tabs to spaces in a file or vice versa. If you really really want to do so, please do it in a single commit, and the commit should not contain other modifications. Otherwise, you can imagine the anger when someone starts debugging by comparing the histories of the file.

## Java

### Do Not Keep State in Stateless Bean
The annotation @Stateless implies that you have to ensure the bean is stateless. If you use states (class scoped variables), it's possible that their value are shared by different threads, and occasionally the methods using them will produce unexpected result. Please search for how stateless beans work if you would like to know more details.

_Recommended:_
```Java
@Stateless
public class CstAddressAcsBean {
    public void checkAddrId(int addrId) {
        // do things with addrId
    }
}
```

_Avoid:_
```Java
@Stateless
public class CstAddressAcsBean {
    int addrId = 0;    // Class scoped variable should not be used here

    public void setAddrId(int addrId) {
        this.addrId = addrId;
    }

    public void processAddrId() {
        // do things with this.addrId
        // Unexpected addrId occasionally
    }
}
```

### Locks in Java
Locks in Java, e.g. synchronized blocks, exist in application level. In the production environment, there can be a server cluster consists of multiple servers, each running one instance of application. In that case, application level lock is insufficient because it does not apply to all instances. Try to use locks at database level instead. Be aware to lock the row only, not the entire table, and always release the lock.

### Write to ServletResponse Correctly
According to [ServletResponse API](http://docs.oracle.com/javaee/6/api/javax/servlet/ServletResponse.html), either getWriter() or getOutputStream() may be called to write the body, **not both**. It is commonly overlooked especially when the response goes through the request filter chain. Please bear this in mind when you write into response.

### Pass By Value
Java is always **pass-by-value**, NOT reference. If the argument is an object, what is passed is essentially its address value. Please do not get confused.

```Java
public void foo() {
    User user = new User("Alice");

    changeNameFail(user);
    System.out.println(user.getName());  // Alice

    changeNameSuccess(user);
    System.out.println(user.getName());  // Jane
}

public void changeNameFail(User user) {
    user = new User("Bob");  // user points to a new object, the original object is detached.
    System.out.println(user.getName());  // Bob
}

public void changeNameSuccess(User user) {
    user.setName("Jane");
    System.out.println(user.getName()); // Jane
}
```

### Avoid using clone()
Some people may like using clone() to copy objects. However, if the cloned object A contains another object B, what's being cloned for B is its address value. Essentially, A and its clone contain the same B. Please take a look at following example. For more information, please refer to [Effective Java](http://books.google.com/books?id=ka2VUBqHiWkC&pg=PA55&lpg=PA55&dq=effective%2Bjava%2Bclone&source=bl&ots=yXGhLnv4O4&sig=zvEip5tp5KGgwqO1sCWgtGyJ1Ns&hl=en&ei=CYANSqygK8jktgfM-JGcCA&sa=X&oi=book%5Fresult&ct=result&resnum=3#PPA54,M1).

```Java
public class TestClone implements Cloneable {

    private Foo foo;

    @Override
    public TestClone clone() throws CloneNotSupportedException {
        return (TestClone) super.clone();
    }

    public static void main(String[] args) throws CloneNotSupportedException {
        TestClone t1 = new TestClone();
        t1.foo = new Foo(1,5);

        TestClone t2 = t1.clone();
        t2.foo.x = 4;

        System.out.println(t1.foo.x + "-" + t1.foo.y); // prints 4-5, NOT 1-5!!!
        System.out.println(t2.foo.x + "-" + t2.foo.y); // prints 4-5
    }
}

class Foo {
    int x;
    int y;
    Foo(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```

### String replace() and replaceAll()
- The method replaceAll() is poorly named. Both methods replace ALL occurrence, the difference is that replaceAll() searches by regular expression.
- Please stick with replace() unless regex is used. 
- Exception may occur if you misused replaceAll() and the replacement string contains special characters like "$".

```Java
String s = "some amount";

System.out.println(s.replace("amount", "$ 100")); // prints "some $ 100"

System.out.println(s.replaceAll("amount", "$ 100")); // java.lang.IllegalArgumentException
```

### Avoid Writing Java in JSP
There are lots of pages in Front Site containing logic written in Java. However, it is not only difficult to read and maintain because you simply cannot put break points to debug, but also error prone because you will not have Java compile errors in JSP. Try to avoid that as much as possible.

## JavaScript

### Use Module Pattern
Please refer to [JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html) and have a better understanding of JavaScript module patterns.
The rule of thumb is, **DO NOT** pollute global namespace.

### Modify a Library###
Generally, it is not recommended because you may encounter problems when upgrading the library. However, sometimes we have to customize it to suit our needs. In that case, we should keep the impact minimum.

_Recommended:_ Write an extension file and keep it in the project folder.
Sometimes the library has build path specified explicitly inside the file and you are using a build tool like CommonJS or AMD, you may not be able to do so without modifying the file in the build path. In that case, you may duplicate the file, rename the original as xxx-original and keep it in the same folder. When you upgrade the library, just look for files named \*-original, compare their differences with the new ones, and apply the differences in the new version.

_Avoid:_ Modify the library directly and leaves no clues. Other developers may use the same library later, and they probably will encounter unexpected behaviors. It could take them days to figure out it is all because you have modified the library.

### Variable definition
_Recommended:_
```JavaScript
// Comma separated
var a = 1,
    b = 1,
    c = 1;
```

_Avoid:_
```JavaScript
// Here b and c become global variables
var a = b = c = 1;
```

### Loops
When writing for..in loop, always remember to check hasOwnProperty().

_Recommended:_
```JavaScript
for (var key in obj) {
    if (obj.hasOwnProperty(key)) {
        // logic here
    }
}
```

_Avoid:_
```JavaScript
for (var key in obj) {
    // logic here
}
```

### Floating Number Computation
You may do it by "multiply and divide" technique, or rather do it in Java using BigDecimal.

_Recommended:_
```
var x = (0.2 * 10 + 0.1 * 10) / 10; // x will be 0.3
```

_Avoid:_
```
var x = 0.2 + 0.1; // x will be 0.30000000000000004
```

### Useful Patterns
Please refer to [The Essentials of Writing High Quality JavaScript](http://code.tutsplus.com/tutorials/the-essentials-of-writing-high-quality-javascript--net-15145).

### File Naming Convention
Use lower case (a-z) words separated by hyphen (-).

_Recommended:_ **text-msg.js**
```
var TextMsg = (function () {
    // Code here
})();
```

### Debugging
Use console.log() to debug, but remember to remove them when you commit the source code.

## JQuery

### Selector
When using a selected element multiple times, it's better to cache it to avoid multiple selection process, or use chaining if possible.

_Recommended:_
```JavaScript
// cache selector
var $a = $('#a');
$a.css("color", "red");
$a.fadeOut();

// or chaining
$('#a').css("color", "red").fadeOut();
```

_Avoid:_
```JavaScript
$('#a').css("color", "red");
$('#a').fadeOut();
```

### Event Listener
When it's possible, try to bind the event listener to the most precise element, to avoid unecessary event handling.

_Recommended:_
```JavaScript
$('#a').on('click', '.b', function(){
    // handle event
});
```

_Avoid:_
```JavaScript
$(document).on('click', '#a .b', function(){
    // handle event
});
```

## AngularJS

Check out [AngularJS StyleGuide](https://github.com/johnpapa/angular-styleguide) before coding.

### Constructor

If you have read through the style guide, you may notice that the author recommend this way of writing a constructor:

```JavaScript
angular
    .module('app')
    .controller('DashboardController', DashboardController);

DashboardController.$inject = ['$location', '$routeParams', 'common', 'dataservice'];

function DashboardController($location, $routeParams, common, dataservice) {
}
```

However, in the above we have to write each service name twice, one inside $inject array and one inside controller arguments. It becomes troublesome to modify because you have to do it twice. As mentioned, we should keep info in one place as possible, so we come up with the following helper function. If you think the word "param" is redundant, feel free to remove it.

```JavaScript
// DashboardController.js
(function() {
    var dependency = ['$location', '$routeParams', 'common', 'dataservice', DashboardController];

    angular
        .module('app')
        .controller('DashboardController', dependency);

    function DashboardController() {
        var vm = this;
        Util.storeParam(vm, dependency, arguments);

        // access to $location by vm.param.$location, etc.
    }
})();

// Util.js
var Util = (function() {
    var util = {};
    util.storeParam = function(vm, dependency, args){
        vm.param = vm.param || {};
        for(var i = 0; i < args.length; i++){
            vm.param[dependency[i]] = args[i];
        }
    };
    return util;
})();
```

### Promise
AngularJS uses $q service to handle promises. It is to mitigate the Pyramid of Doom, which occurs when you nest multiple callback functions. Following are some examples.

Use $q.when() to wrap value
```JavaScript
$q.when(1).then(function(value) {
    // value is 1 here
});
```

Use $q.when() to return promise
```JavaScript
function foo() {
    return $q.when(2);
}
function bar(value) {
    return value + 1;
}

foo().then(bar).then(function(value) {
    // value is 3 here
});
```

Use $timeout to wrap function
```JavaScript
$timeout(function() {
    return 15;
}).then(function(value) {
    // value is 15 here
});
```

Use $http to load resource
```JavaScript
$http.get(url).then(function(response) {
    // access response.data here
})
```

### One-time Binding
If the value is a constant, e.g. the menu text from dictionary, use one-time binding
```JavaScript
{{::value}}
```
instead of
```JavaScript
{{value}}
```
Reducing the number of expressions being watched makes the digest loop faster.

## Database

### Choose a Good Primary Key
When your table design consists of a **composite primary key**, think twice before making it as the primary key. It is sometimes more convenient to introduce an extra ID/Seq column as the primary key, and create an unique index onto the compound columns.

### JPA: getResultList() vs getSingleResult()
* Always use **getResultList()** when you retrieves data.
* Use **getSingleResult()** only when you are retrieving the result of SQL aggregate function, e.g. MIN, MAX, SUM, COUNT, etc.
* Be aware that getSingleResult() will throw **NoResultException** when there is no data, and **NonUniqueResultException** where there are multiple rows of data. The above approach helps you eliminate these exceptions.

### Do Not Extract Column Names into Constants
_Recommended:_ Write one SQL simple and clean.
```Java
String sql = " select p.name, p.gender" +
                " from person p" +
                " where p.name = :name" +
                " and p.gender = :gender" +
                " and p.dob = :dob" +
                " and p.address = :address";
```

_Avoid:_ Extract the column names. The code can often get messy and hard to read. The reader has to type the whole thing again if he wants to test it in DB tools.
```Java
private static final String PERSON= "person";
private static final String NAME= "name";
private static final String GENDER = "gender";
private static final String DOB = "dob";
private static final String ADDRESS = "address";
...
String sql = "select p." + NAME + ", p." + GENDER +
                " from " + PERSON + "p" +
                " where p." + NAME + " = :name" +
                " and p." + GENDER + " = :gender" +
                " and p." + DOB + "  = :dob" +
                " and p." + ADDRESS + "  = :address";
```

### Do Not Loop Database Access
Database access can be costly. Instead of making multiple round trips to retrieve data one by one, group them and retrieve all data in one go. Otherwise it can hurt the performance.

_Recommended:_

personService.java
```Java
List<Long> ids = new ArrayList<Long>();
// some logic to retrieve ids

List<Person> persons = personDao.fetchPersons(ids);
```

personDao.java
```Java
public List<Person> fetchPersons(List<Long> ids) {
    String sql =  " select p " +
                " from person p" +
                " where p.id in (:ids)";
		...
}
```

_Avoid:_

personService.java
```Java
List<Long> ids = new ArrayList<Long>();
// some logic to retrieve ids

List<Person> persons= new ArrayList<Person>();
for (Long id : ids) {
    Person p = personDao.fetchPerson(id);
    persons.add(p);
}
```

personDao.java
```Java
public Person fetchPerson(Long id) {
    String sql =  " select p " +
                " from person p" +
                " where p.id = :id";
    ...
}
```
