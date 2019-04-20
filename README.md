# Opinionated C#
This is an opinionated C# standards/best practices guide. The purpose is to provide a guide to developing C#, showing conventions and why they are used.

## Table of Contents
* [Whitespace and Brackets](#whitespace-and-brackets)
* [Naming](#naming)
* [Properties](#properties)
* [Code Ordering](#code-ordering)
* [Object Initialization](#object-initialization)
* [Using Directive](#using-directive)
* [Methods](#methods)
* [Strings](#strings)
* [Exceptions](#exceptions)
* [Operators](#operators)
* [Comments](#comments)
* [Variable Declarations](#variable-declarations)
* [Generics](#generics)
* [Linq](#linq)
* [Xml](#xml)
* [Logging](#logging)
* [Unit Testing](#unit-testing)
* [Coding Principles](#coding-principles)

## Whitespace and Brackets
### Indention
Indention should always be done via tabs, not spaces. (Your Visual Studio / Tools / Options / Text Editor / C# / Tabs setting should be set to Kee p tabs, not Insert spaces.)

* Why?: Although tab settings are an eternal religious debate among developers, tabs represent in one character what is otherwise represented as 4 or more characters. Using tabs also allows indentation to remain consistent regardless of the tab size. Tab size = 4
* Why?: This helps avoid whitespace conflicts during commits and it keeps the code consistently readable throughout.

You may sometimes see a gray bar at the top of a source file in Visual Studio, stating "You have mixed tabs and spaces. Fix this?" Use the "Tabify" button to convert all leading space indentations to tabs. Note, however, that this will mark all affected lines in the file as changed, when it comes to code review

### Block Statements
Brackets need to be on their own line

* Why?: This makes it completely obvious that the brackets relate to the block scope. Not only does this improve maintainability and readability, this also helps keep code looking consistent thoughtout. Without this standard, code layout could look different and it can be jarring for a reader to have to switch styles.

Bad: The author thought this line of code would run inside the if, but it won't.
```c
if (i == 10) j++;  
  return j - i; 
```
Not much better
```c
if (i == 10) { j++; }  
  return j - i;
```
This now runs inside the if block, when it wasn't expected to.
```c
if (i == 10)     
  // j++; 
return j - i;
```

Good:
```c
if (i == 10) 
{  
  j++;   
  return j - i;
}
```

>Note: Although this isn't a C/C++ guide, the maintainers of GCC are adding a compiler flag in GCC 6 to warn about misleading indentation such as the scenario that resulted in Apple iOS and OSX devices accepting invalid certificates. It has been enough of an issue across the industry - leading in some cases to multi-million dollar bugs - that they see value in warning developers against writing code that fits this pattern.

### Properties
If a property has any logic within it, brackets need to be on their own line.

```c
public string MyPaddedString {  
  get {   
    return GetMyString()    
      .TrimEnd('x')
      .PadRight(10, '0');
 }
}
```

Auto-properties should be on one line.

```c
public string MyString { get; set; }
```

### Parentheses
Remove redundant parentheses on object initializers.

* Why?: They clutter the code without providing any useful information.

```c
var myValue = new Value() {
 NestedValue = "1",
 Code = "123"
};
```

Parens are not necessary here 
```c
var myValue = new Value {
 NestedValue = "1",
 Code = "123"
};
```

## Naming
### Classes
Refrain from using generic words for class names such as helper, utility, object, etc.

* Why?: Class names should indicate the function the class is serving. Prepending or appending these words don't add value in describing what the class is doing.

### Methods
Methods should always be in pascal case i.e. MyMethod

Methods should be appropriately named based on what operation it is performing.

* Why?: Consistent naming conventions improve code readability. Safe assumptions can be made to save time if the method is name appropriately and actually does what it claims to do.
* Why?: If a method name contains multiple verbs, this is a sign that it may be doing too much and should be refactored.

| Operation | Convention | Examples(s) |
| ------ | ------ | ------ |
| Returns an object or objects | GetMyObject | GetUser |
| Populates an object | PopulateMyObject | PopulateUserSession |
| Validates an object | ValidateMyObject | ValidatePassword |
| Performing an action | ActionMyObject | AuthenticateUser |
| Saving an object | SaveMyObject | SaveUser |
| Updating existing object | UpdateMyObject | UpdatePassword |
| Delete an object | DeleteMyObject | DeleteUser |

Methods whose names start with 'Get' should return a value.

* Why?: A method whose name starts with 'Get' implies that it returns a value. A void method does not return a value. Therefore, these methods should not be void

### Properties
Properties should always be in pascal case i.e. MyProperty

### Variables
Private class level variables should always start with an underscore followed by camel case i.e. _myVariable 

Parameters and method level variables should be camel case, but not start with an underscore i.e. myVariable

Do not use abbreviations for your variables unless they are company/industry standard acceptable abbreviations.

* Why?: We don't get points (or faster code) for using shorter variable names. Make sure your variable names are clear as to what they contain. Abbreviations might work for you, but might be confusing for a reader.

***No Hungarian Notation!!!***

### Constants
Constants should be in upper case with underscores, i.e. MY_CONSTANT.
> This is sometimes called "screaming snake case"

### Nameof Expression
C# 6 now has a new expression called nameof which can be used like this:

```c
if (x == null) {
 throw new ArgumentNullException(nameof(x));
}
```

This is a great way to get the actual name of the variable and it is very useful in the example above. The reason this is important is that, if you refactored the parameter name, the string value would not be updated to reflect the change. By using nameof, the refactoring tool will update the reference so it stays in sync.

### Extension Methods
Extension Methods are acceptable, but consider how they will be used.  Obviously, the coding standards dictate full unit testing of the Extension Methods. If you will have a lot of logic inside the extension method, consider how you would test classes that use the extension method.  It might be better to implement an Interface and ConcreteImplementation so that you can mock that complex logic.

### Defining Properties
 
Properties can be implemented in two different ways: backing-field, or auto-property.

Backing field 
```csharp
private string _awesomeThing;  
public string AwesomeThing 
{  
  get  
  {   
    return _awesomeThing;  
  }  
  set  
  {
    _awesomeThing = value;
  }
}
```

Auto property 
```csharp
public string AwesomeThing { get; set; }
```

Unless you have a compelling reason otherwise, prefer the auto-property syntax.
* Why?: Code is less cluttered and easier to read/understand.
Model classes returned by the API or passed between data/business/API layers must use auto-property syntax.
* Why?: These classes are not unit tested directly, so logic may not be fully tested.
* Why?: Logic should be implemented in the business layer or as part of mapping between layers, not in the objects themselves.

### Auto Property Default Values
C# 6 now allows syntax for defining default values to auto properties. This can simplify setting defaults and it will move it out of the constructor. These are fine to use, but we should not abuse these and just set defaults for everything. For example, the .net default value for a string should suffice, you should not be setting your defaults to String.Empty or setting your int defaults to something like -1 unless you really depend on those defaults. Try to rely on the .net defaults as much as possible.

Auto property with a default value 
```csharp
public string AwesomeThing { get; set; } = "It is awesome"; 
public string GetterOnly { get; } = "You're stuck with me";
```

> Note: This actually creates a readonly property behind the scenes, so you can still change this in your constructor, but not outside of it.

* Why?: Using auto property default values can clean up a constructor and possibly even remove the need for it.

## Code Ordering
### Main Outline

usings 
namespace 
class 
consts
fields
properties / backing fields 
delegates
events
constructors 
finalizers (destructors)
indexers
methods

### Access Modifiers Order
public internal 
protected internal 
protected private

### Property / Backing Field Interspersed Example
```csharp
public string MyVar1 
{  
  get 
  { 
    return _myVar1; 
  }  
  set 
  { 
    _myVar1 = value; 
  } 
} 

private string _myVar1 = String.Empty; 

public string MyVar2 
{  
  get 
  { 
    return _myVar2; 
  }  
  set 
  { 
    _myVar2 = value; 
  } 
}

private string _myVar2 = "Tacos";
```

### Example Class

```csharp
// usings 
using System;  

// namespace 
namespace Some.Example.Code 
{  

  // class
  public class LayoutExample  
  {
    // consts
    private const string LAYOUT_EXAMPLE_STRING = "ThisIsAString";    
	
	// fields   
	private ICompanyObject _companyObject;    
	
	// properties   
	public ISomeObject   
	{    
	  get 
	  { 
	    return _someObject; 
	  }   
	}    
	
	public string SomeCode { get; set; }  
	
    // delegates
	
    // events
	
    // constructor(s) - with dependency injection, you shouldn't need more than one.
	public LayoutExample(    
	  ISomeObject someObject
	)
	{
	  _someObject = someObject;
	}
 
    // finalizers
	public ~LayoutExample() {}
 
    // indexers
	
    // methods   
	public void PublicMethod()   
	{
      InternalMethod();
    }    
	
	internal void InternalMethod()   
	{
      ProtectedInternalMethod();
    }  
	
    protected internal void ProtectedInternalMethod()   
	{    
	  ProtectedMethod();
    }  
	
    protected void ProtectedMethod()   
	{    
	  PrivateMethod();
    }    
	
	private void PrivateMethod()   
	{
      _someObject.MakeEverythingAwesome(LAYOUT_EXAMPLE_STRING);
    }
	
  }
}
```

## Object Initialization
### Object Initializers
Object initializers should always be used when newing up objects.

Discouraged
```csharp
Person person = new Person();
person.FirstName = "Jonathan"; person.LastName = "Doe";
```

Preferred
```
var person = new Person {
 FirstName = "Jonathan",
 LastName = "Doe",
 DateOfBirth = new DateTime(1970, 2, 14)
};
```

Objects can be initialized on a single line if only 2-3 properties are being initialized. When initializing any more than that, place the brackets and properties on new lines.
* Why?: Typically it is good to keep lines less than 120 characters long, If a line is too long, it would require the reader to use the horizontal scrollbar which can easily interrupt the flow of reading the code. This also allows the addition, modification, or removal of properties from the initializer with fewer merge conflicts in Git.

### Index Initializers
In C# 6, Microsoft provided an extension to object initializers that allows index initialization. A common use for this would be for creating Json objects declaratively in one expression, rather than having to first create the object and then assign into it in separate statements.

Both ways are acceptable 
```csharp
var oldWay = new Dictionary<string, string> 
{  
  { "some", "thing" },
  { "other", "thing" }
};

var newWay = new Dictionary<string, string> 
{  
  ["some"] = "thing",
  ["other"] = "thing"
};

var numericExample = new Dictionary<int, string> 
{ 
  [2] = "two",
  [42] = "the answer"
}
```

Objects can be initialized on a single line if only 2-3 properties are being initialized. When initializing any more than that, place the brackets and properties on new lines.
* Why?: Typically it is good to keep lines less than 120 characters long, If a line is too long, it would require the reader to use the horizontal scrollbar which can easily interrupt the flow of reading the code. This also allows the addition, modification, or removal of properties from the initializer with fewer merge conflicts in Git.

### Be Mindful of Unit Testing 
Bad 
```csharp
public void DeleteAllEmployees() 
{  
   Repository myRepository = new Repository();
   myRepository.DeleteAllEmployees(); 
}
```

Good 
```csharp
private IRepository _repository; 

public MyClass() : this(
  new Repository()
) {} 

internal MyClass(IRepository repository) 
{
  _repository = repository;
}  

// In this case, a unit test would inject a mock repository. Now when it runs, it will not actually delete all of the employees. 
public void DeleteAllEmployees() 
{
  _repository.DeleteAllEmployees(); 
}
```

Better 
```csharp
private IRepository _repository;  

// At runtime, dependency injection will give us a concrete instance implementing IRepository
public MyClass(IRepository repository)  {
 _repository = repository;
}  

// For unit testing, inject a mock repository and assert that the correct repository method is called.
public void DeleteAllEmployees() {
  _repository.DeleteAllEmployees(); 
}
```

When using dependency injection, your public constructor should declare explicitly the classes and interfaces it needs for proper construction, and the container will figure out how to construct its dependencies. Unit tests will inject these directly with mocks.

## Using Directive
### Standard Usage
```csharp
using Some.Framework;
```
Only use what you are actually using. If you do not need a specific reference, remove it.

### Static Usage
```csharp
using static TylersTacoTruck.TacoGenerator;  

public class Program   
{       
  static void Main()       
  {   
    IList<Taco> tacos = GetTacos();
  }   
} 
```
Avoid this usage at all costs!!!
* Why?: It hides code and it causes problems when unit testing.

### Alias Usage
```csharp
using Project = PC.MyCompany.Project;  
```
Only use this when you would have conflicts in types. An example would be in mapping where you have a Claim object that exists in the Data layer and the Business layer. Creating an Alias helps the readability of the code.

## Methods
### Expression-bodied function members
C# 6 introduces a new way to express class members by allowing them to act more like methods. You can think of these like one line methods. 

Accepted use 
```csharp
public Point Move(int dx, int dy) => new Point(x + dx, y + dy);

public static Complex operator +(Complex a, Complex b) => a.Add(b); 

public static implicit operator string(Person p) => "\{p.First} \{p.Last}";

Unaccepted use
```csharp
public string Username(userId) => _repository.FindOne(
  new SelectUsernameByUserId 
  { 
    UserId = userId 
  }
).Username;
```

These can really clean up code when used correctly, but they should not be very complex.
* Why?: Because of this new syntax, you might want to start making everything one-liners. The problem here is that it can become cumbersome to read. The Unaccepted use example above works, but its doing too much. Think about the single responsibility principle here.

### Methods Signatures
#### Too Many Parameters
Try to keep method signatures under 4 parameters. If you find yourself using 4 or more parameters, consider grouping the parameters into a class.
* Why?: According to the book Clean Code, anything over 4 parameters is mentally tasking to understand. Keeping parameter signatures simple and small helps a reader quickly understand what that method might actually do.

#### Optional Parameters
Optional parameters can be a little difficult to quickly understand when walking through code, so here are some guidelines for usage:
* Avoid using multiple optional parameters in the same signature. If you think you need multiple optional parameters, pass an object with appropriate properties instead.
* Do not use optional parameters on methods which are also overloaded.

#### Out Parameters
Avoid using out parameters unless they are 100% needed. For the most part, you can avoid them altogether.
> Why?: Many usages of out parameters are due to oversights in the usage of the method.
* If your method has a void return type and an out parameter, then the out parameter should be the return value.
* If you are returning an object and using an out parameter, your method is most likely doing too much and should be refactored.
* All objects are passed by reference, so passing an object as an out parameter (or ref for that matter) is unnecessary. Most likely, you just need to be using a PopulateMyObject method.

## Strings
### String Interpolation
String Interpolation should be used in place of String.Format.
Preferred
```csharp
string messageInterpolated = $"Hello!  My name is {person.FirstName} {person.LastName} and I am {person.Age} years old.";
```

Use when it makes sense
```csharp
string message = string.Format("Hello!  My name is {0} {1} and I am {2} years old.", person.FirstName, person.LastName, person.Age);
```
> Why?: The string will be easier to read in code.

## Exceptions
### Exception Filters
In C# 6, exception filters were introduced. These allow you to be more explicit in your exception conditions. Here is an example that would only log exceptions on weekends:
// Old way 
```csharp
try 
{  
  DoStuff(); 
} 
catch (Exception e) 
{  
  if(DateTime.Now.DayOfWeek == DayOfWeek.Saturday 
    || DateTime.Now.DayOfWeek == DayOfWeek.Sunday) 
  {
    Log(e);  
  } 
  else 
  {   
    throw;  
  }
}
```

New way 
```csharp
try 
{  
  DoStuff();
} 
catch (Exception e) when (
  DateTime.Now.DayOfWeek == DayOfWeek.Saturday 
  || DateTime.Now.DayOfWeek == DayOfWeek.Sunday
) 
{
  Log(e);
}
```
> Why?: This can really clean up the actual catch block and it allows you to have targeted catch blocks for the same exception type.

### Rethrowing Exceptions
Always use `throw;` to rethrow an exception, not `throw ex;` (assuming your exception object in the catch block is named `ex`.)
> Why?: `throw ex;` resets the stack trace on your exception to the catch block, not where it was originally thrown. `throw;` preserves the original stack trace.

Bad 
```csharp
try 
{  
  throw new Exception("Stack trace here!"); 
} 
catch (Exception ex) 
{  
  // ... log or whatever  
  throw ex; // Stack trace is now reset to here. 
}
```

Good 
```csharp
try 
{
  throw new Exception("Stack trace here!"); 
} 
catch (Exception ex) 
{  
  // ... log or whatever  
  throw; // Stack trace is preserved
}
```
 
## Operators
### Null-Conditional Operator
C# 6 introduces a great new way to do null checking.
Good
```
if(user?.name?.first === "John") { }
```

Bad 
```csharp
if(NameChecker(user?.name?.first)) { }  
var firstname = some?.massive?.collection?[0]?.users?[0]?.user?.name?.first; 
var lastname = some?.massive?.collection?[0]?.users?[0]?.user?.name?.last;
```
The operator will perform a short-circuit on the logic to check for nulls before checking the sub-property. This can really clean up code and it can protect against null referencing. We do want to avoid passing null checking like this into a method as it pushes behavioral changes down into a method. For example, a method that has a required parameter throws and argument exception when something is null. But null conditionals will always pass nulls so one would be tempted to remove the argument exception. Repeating the same null conditional operator is a code smell and hides bad coding. 

> Note: You should be aware of what the code will actually compile down to. 

## Comments
Only comment if it's imperative in explaining the code. Code should be written with meaningful and appropriate names to indicate what the code is actually doing. 

If you feel the need to comment, ask yourself if the code could be refactored to better explain what it's doing.
Comments should explain why the code works a certain way, not what it is doing. Comments that describe what the code is doing become obsolete and they duplicate information anyway.

If you replace obsolete code, don't leave commented-out code in place as an artifact. This is a problem for multiple reasons:
* Code files are bigger than they need to be, for no benefit
* Code is cluttered and difficult to read
* Searching for method or variable names can find hits inside of commented code, making search results less useful 
*The previous version is already preserved in source control history

## Variable Declarations 
The use of var is acceptable; however, should be limited to instances where it is clear as to what type the variable actually is. You should always use descriptive meaningful variable names, but it is even more important when using var!

Good 
```csharp
var myList = new List(); // object initialization

var myObject = GetMyObject(); // well named methods

var myObject = yourObject as MyObject; // casting 

var expiredOffices = offices.Where(x => x.Expired); // good variable names

var myObject = new { Name = "Joe", Age = 1000 }; // anon Type  

for (var i = 0; i < 10; i++) { } // for loops

foreach (var file in _ioFactory.GetFiles("my/directory") { } // foreach loops 

using (var file = _ioFactory.GetFile("my/path")) { } // using statements  

Bad
```csharp
var item = ProcessItem(); // ProcessItem doesn't clearly indicate what it is returning 

var individuals = GetContacts(ContactTypes.Individuals); // Contacts is actually be returned. Individual is not a type.
```

Remember that code readability is paramount and knowing data types is important when reading code.

Again, naming is key, so always try to code as if the code has to explain everything and the tools cannot always be relied upon.

## Generics
Generics are a powerful way to implement functionality that can be applied to multiple types in a consistent manner. Do not abuse generics. They can make code more difficult to read and understand.

### Use where to limit type parameters
You are strongly encouraged to use where syntax on your generic type parameters to specify what kind of objects your generic expects to handle. This is particularly true if you can restrict it to a base class or interface.
```csharp
public class SelectQueryBase<TFilter, TResult> where TFilter : FilterBase where TResult : IList
```
> Why?: Anyone who calls your generic class should know what it expects and what it will do, based on the generic types it accepts.

### Prefer delegate over Func<> for method pointers
When passing a function pointer to another method, declare a delegate with the appropriate signature rather than defining a parameter of a Fun c<> or Action<> type.
Why?: The parameters of a delegate can be named so that they express intent.
```csharp
public delegate bool MakeItSoMethod(string commandFrom, string commandTo, string command);  

// Anyone maintaining this method will get IntelliSense information about the method parameters.
public bool DoCaptainStuff(MakeItSoMethod myMethod) 
{  
  return myMethod("Picard", "Riker", "take the helm"); 
}  

// Anyone maintaining this method has absolutely no idea what this method will actually do with its parameters.
public bool DoCaptainStuff2(
  Func<string, string, string, bool> myMethod2
) 
{  
  return myMethod2("Riker", "Wesley", "polish my shoes"); 
}

// ....  
public void DoSomeCaptainStuff() 
{  
  bool command1Worked = DoCaptainStuff(MakeItSo); // Method matches delegate signature   
  
  bool command2Worked = DoCaptainStuff2(MakeItSo); // Method matches Func<> signature
  
  Console.WriteLine("Command 1 {0}", command1Worked ? "worked" : "did not work.");  
  
  Console.WriteLine("Command 2 {0}", command2Worked ? "worked" : "did not work."); 
}  
  
private bool MakeItSo(string commandFrom, string commandTo, string command) 
{  
  if (commandFrom == "Wesley")  
  {   
    return false;  
  }   
  
  bool isCaptain = commandFrom == "Picard";   
  
  if (isCaptain)  
  {
    Console.Write("This is the captain speaking. ");
  }
 
  Console.Write("{0}, {1}.", commandTo, command);
 
  Console.WriteLine(isCaptain ? "Now!" : "Please.");
  
  return true; 
}
```

## Linq
Linq is awesome. Use it when it makes sense, but be careful. Linq can turn ugly very quickly. So keep the statements clear and concise.

Push any advanced operations (grouping, sorting, filtering) into the database if possible. Don't retrieve a thousand records to C# code in order to produce 3 output rows.

Be aware of deferred execution: when you write a series of Linq expressions, they are not executed until the result is actually needed. (In Linq terms, this is when the query is materialized.)

## Lambda Expressions
Similar to for-loops, it is ok to use a less descriptive input parameter if there is only a single parameter used. However, input parameters need to have appropriate names when multiple parameters are used. And as mentioned previously, you don't score points for using short names.

Good
```csharp
var listItem = myList.Where(x => x.MyProperty == "a"); 

var add = (x, y) => x + y; 

var users = users.Where(user => 
    user.Name == "John" 
    && user.Offices.Where(office => office.Expired
  )
);  
```

Bad 
```csharp
var users = users.Where(x => 
    x.Name == "John" 
    && x.Offices.Where(y => y.Expired
  )
);
```

**Never, under any circumstance, use _ as the expression parameter name.**
> Why?: Not only is this confusing with our private variable naming, it also is confusing for people switching back and forth from Angular, where _ is the lodash shortcut. It also blocks whatever small semantic value might be found in a single character name. (For example, if looping over a list of offices and you use "o" as the parameter, that at least tells you something. What does "_" mean?)

> Why?: "In some functional languages, it is common to use _ for an unused parameter." Why do you have an unused parameter? There have also been instances committed to our repos where a parameter was named this way, and was used in the lambda expression body.

## Any() vs Count
Use list.Any() instead of list.Count > 0
> Why?: Cleaner to read.

# Use Any(<lambda>), All(<lambda>) methods instead of Where(<lambda>).Any()
Any(), All(), Count(), and several other Linq methods can accept lambda expressions to indicate what you are looking for. Use these instead of calling Where(...).Any(), etc. Why?: Easier to read
> Why?: Any() and All() can shortcut operation as soon as they find one that doesn't match the criteria. Where(...) does not.
Bad
```csharp
if (colorList.Where(c => c.Color == Color.Green).Any()) 
{  
  // ...
}
```

Good
```csharp
if (colorList.Any(c => c.Color == Color.Green)) 
{  
  // ...
}
```

## XML
### XmlDocument vs XDocument
Use XDocument over XmlDocument

## Logging
Logging is an important part of app monitoring and tooling. Remember that other people use logging beyond just developers. When logging, always ask the questions, "Am I using the right category?" and "Is this useful to anyone else?".

### Logging Categories
* DEBUG: Only used when debugging, usually very verbose
* INFO: Used to indicated something started, or something got deleted, etc 
* * This can also be used by monitoring programs such as Splunk. When logging, try to think about what information would be good for a monitoring program.
* WARN: Something happened but we are still going to move on with the program flow
* ERROR: Something bad happened and we probably stop or change program workflow
* FATAL: Oh crap! Oh crap!!! Totally did not expect this to happen! Yeah....we are going to shutdown the program here...

## Unit Testing
Just do it
 
## Coding Principles
### DRY Principle
 
DRY stands for Don't Repeat Yourself aka DRY (Don't Repeat Yourself). The idea is that if you find some code that you have written doing similar stuff, you might be able to simplify it or genericize it. The DRY priniciple basically just leads to better code!
 
Before 
```csharp
public class SomeProgram 
{  
  public static int Main(string[] args)  
  {
    SaySomething();
    SaySomethingElse();
    YellSomethingCrazy();
    WhisperSomethingAwesome();
 
    Console.ReadKey();
  }   
   
  private static SaySomething()  {
    string message = "Hello everyone!";  
    Console.WriteLine(message);
  }   
 
  private static SaySomethingElse()  
  {
    string message = "Hello everybody else!";   Console.WriteLine(message); 
  }   
  
  private static YellSomethingCrazy()  
  {   
    string message = "Coding standards are awesome!";   
    message = message.ToUpper();   
    Console.WriteLine(message); 
  }   
  
  private static WhisperSomethingAwesome() {  
    string message = "Linux is way better than windows.";   
    message = message.ToLower();   
    Console.WriteLine(message);
  }
}
```

Notice how there is a lot of the same code in each method for saying something. We can simplify this...

After 
```csharp
public class SomeProgram 
{  
  public int static Main(string[] args)  
  {
    WriteMessage("Hello everyone!", Tone.Normal);
    WriteMessage("Hello everybody else!", Tone.Normal);
    WriteMessage("Coding standards are awesome!", Tone.Yell);
    WriteMessage("Linux is way better than windows.", Tone.Whisper);  
    
    Console.ReadKey();
  }   
  
  private static WriteMessage(string message, Tone tone) {
    switch(tone) {    
      case Tone.Whisper:
        message = message.ToLower();     
        break;    
      case Tone.Yell:
        message = message.ToUpper();     
        break;   
    }
    Console.WriteLine(message);
  }
  
  private enum Tone 
  {   
    Normal,
    Whisper,
    Yell
  }
}
```

The number of lines might not be much smaller, but the second example is very dry. Not only that, it is reusable! This code could grow. We can now add a ton of new Tones and the messages are generic.

### DAMP Principle (Unit Testing)
DAMP stands for "**D**escriptive **A**nd **M**eaningful **P**hrases". The idea behind the DAMP Principle is to use "descriptive and meaningful" naming throughout your code. The point being to make your code readable and not requiring a lot of deep thinking to understand the logic.
* From StackOverflow: (http://stackoverflow.com/questions/6453235/what-does-damp-not-dry-mean-when-talking-about-unit-tests)
 
#### It's a balance, not a contradiction
DAMP and DRY are not contradictory, rather they balance two different aspects of a code's maintainability. Maintainable code (code that is easy to change) is the ultimate goal here.

#### DAMP (Descriptive And Meaningful Phrases) promotes the readability of the code.
To maintain code, you first need to understand the code. To understand it, you have to read it. Consider for a moment how much time you spend reading code. It's a lot. DAMP increases maintainability by reducing the time necessary to read and understand the code.

#### DRY (Don't repeat yourself) promotes the orthogonality of the code.
Removing duplication ensures that every concept in the system has a single authoritative representation in the code. A change to a single business concept results in a single change to the code. DRY increases maintainability by isolating change (risk) to only those parts of the system that must change.

#### So, why is duplication more acceptable in tests?
Tests often contain inherent duplication because they are testing the same thing over and over again, only with slightly different input values or setup code. However, unlike production code, this duplication is usually isolated only to the scenarios within a single test fixture/file. Because of this, the duplication is minimal and obvious, which means it poses less risk to the project than other types of duplication.

Furthermore, removing this kind of duplication reduces the readability of the tests. The details that were previously duplicated in each test are now hidden away in some new method or class. To get the full picture of the test, you now have to mentally put all these pieces back together.
Therefore, since test code duplication often carries less risk, and promotes readability, its easy to see how it is considered acceptable.

As a principle, favor DRY in production code, favor DAMP in test code. While both are equally important, with a little wisdom you can tip the balance in your favor.
### Campfire Principle
When camping, a general rule of thumb is to make the condition of your campsite better on your way out than it was when you arrived. The same can hold true for coding!

Examples could include:
* Finding some old code that needed to communicate with the database, but it used the old way. If you switched it to use the Repository layer, you would be following the campfire principle!
* When touching old code that does not have unit tests, add new unit tests around the code. This fits the Campfire principle
* Finding some string being written like this:

```csharp
_logger.WriteError("The file, " + fileName + " was only" + fileSize + " bytes, which was below our minimum requirement of " + MIN_FILE_SIZE
+ " bytes!");
```

If you were to be in some code that looked like the line above, you could easily clean this up by changing it to look like the line below:

```csharp
_logger.WriteError($"The file, {fileName} was only {fileSize} bytes, which was below our minimum requirement of {MIN_FILE_SIZE} bytes!");
```

License
----

MIT