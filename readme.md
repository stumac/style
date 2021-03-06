# The Elements of Programmatic Style

Forked from the infallible [Jason Brennan][original].

## Introduction

The purpose of this guide is to ensure existing and future iOS and Mac products conform to a consistent and well-designed convention. This will make applications easier to build and debug for both newcomers and existing members alike. In this document, we'll be using Magnetic Bear Studios as an organization name, but the same rules can apply to any organization, even if for solo developers.

Many of the conventions in this guide are common to existing Cocoa style guides (see the References section below), and try to combine the best of those worlds.

This document aims for excruciating clarity, attempting to document all possible aspects of iOS project development. If an aspect of coding style is not addressed here, please inform the authors and have the document updated accordingly.

This document also strives to include the best and most modern features of the Objective-C language, Cocoa frameworks, and the iOS SDK in general, so that excessive baggage does not creep in. As the environment evolves, so too must this guide.

### Terminology

First, some quick terminology.

* `(` and `)` are *parentheses*
* `{` and `}` are *braces*.
* `[` and `]` are *brackets*.
* `<` and `>` are *angle brackets*.
* `*` is an *asterisk* or *star* (or *pointer thingy* `:)`).
* `_` is an *underscore*.

### About performance

Some of these guidelines may seem counter-perfomance (that is, these conventions may appear to hurt performance), but in fact, they are in place to *help* performance: Developer performance. Most runtime performance hits as a result to this guide will be negligeable, but the gains to the developers will be substantial.

If any of the guidelines causes a substantial performance hit, it can be dealt with on a case basis, and only after it has been thoroughly profiled.

## Naming

### In general

When giving a name to any kind of symbol, you should strive to make it descriptive over terse. We have text editors capable of auto-completing symbol names, and we expect to make great usage of this. This extends even to iterator variables: these should be named accordingly (even if it's just `index`, but especially in the case of nested loops, `i`, `j`, and `k` will have your head spinning in no time).

    NSInteger currentColumn;

The Cocoa naming conventions call for camel-casing all symbol names (with preprocessor bits being the only exclusion):

    #define X_OFFSET 25.0f
    NSInteger index = [SomeClassName classMethod];

Don't abbreviate symbols and don't use acronyms unless they are extremely common (in which case use uppercase for each letter of the acronym) like `URL` or `PNG`. See the [Apple reference document] [apple2] for a list of acceptable acronyms.

### Files

Files should be given descriptive, camel-cased names, with the initial letter being uppercased. When creating classes in Xcode, this is the default behavior, as the file names match the class names they contain.

Files should be prefixed with the project prefix which should be the 3 initials of the organization (in our case, this is `MBS` for Magnetic Bear Studios).

    MBSClassName.h
    MBSClassName.m

**Note**: A file name should **never** be named with the same prefix as an Apple provided class, with the exception of *Categories*, whose file naming conventions are described below.

### Classes

Class names must be indicative of their class hierarchy. That is, from the class name, you should be able to correctly guess exactly what "kind of" class it is. Don't rely on the Xcode folder/group structure to indicate this, as the text editor itself ignores in which folder the class appears.

    MBSProductsTableViewController

The above name is descriptive, and immediately indicates to the developer this class is a TableViewController which displays products.

Controller classes should contain `Controller` in the name, View classes should contain `View` in the name (e.g. `MBSProductsTableViewCell`), and Model classes should generally contain neither, but still indicate what they are (avoid the use of `Model` in the name as it is redundant).

**Note**: A class should **never** be named with the same prefix as an Apple provided class.

### Protocols

Protocols should be named like the classes they pertain to, additionally appending the protocol role. If there is no distinct role, appending `Protocol` is sufficient. The goal is to be able to tell at a glance what the symbol represents.

    MBSCollectionViewDelegate
    MBSReaperProtocol

### Categories

Category file names should indicate the name of the class being extended, followed by a `+`, followed by the name of the category. I believe this is the default Xcode behaviour as of Xcode 4.2.

When naming the category, descriptive names (i.e. "UIImage+Resizing" or "NSDate+Natural") are preferred over generic ones. If the category methods are a grab bag, then "Utilities" or "Additions" are preferred. The category name should also be prefixed with the project's suffix if the category is hyper-specific to the project (i.e. "UILabel+MBSAdditions" for labels styled for that project's UI).

    NSString+MBSAdditions.h/m

Categories are used to extend existing classes at runtime without subclassing, and we can make liberal use of these. But categories **must not** be used to override existing methods.

### Variables

Variables are to be named in camel-case, initially lowercase. This applies to instance, static, and local variables. Block variables are treated the same as local scope variables in terms of naming conventions.

When declaring objects or any other pointer type, the star belongs with the variable name.

Keep variable declarations on their own line even if they are of the same type.

    NSString *localString;
    NSString *password;

#### Instance variables

Instance variables should be named like normal variables, except with a leading underscore.

Instance variables are not to be declared in the public `@interface` of a class, as this leaks irrelevant implementation details. Instead, declare them as properties or declare them in the `@implementation` block. See the below section for more information on interfaces and implementation.

### Other symbols

Other symbols follow similar rules as already established.

#### Constants

Should be declared with the `const` keyword and should be named with an initial lowercase `k`, followed by the most closely related type or class name, followed by the value.

    const NSString *kMBSProductsDataKey; // defined in an implementation file.

Constants should go in their related class header file if appropriate, or if they are used by potentially many classes, should be placed in a project-wide `Constants.h/m` header file which should be imported in the `Project.pch` prefix file.

#### Enumerations

Enums should be used whenever a variety of integer values could be used (usually as some kind of descriminating grouping). Enumerations should be `typedef`'d and given a name in a similar style to their related class. The enum values should be named similarly, including the name of the type, with the actual type appended.

    typedef enum {
        MBSProductPlacementTypeTop,
        MBSProductPlacementTypeCenter,
        MBSProductPlacementTypeBottom
    } MBSProductPlacementType;

Even though `enum` values are really just integers, they should always be treated as their own type (e.g. `MBSProductPlacementType`, and never just `int`). This gives the symbol extra context instead of just some random, unrelated type. This also means you should never use a plain integer when a enum value is needed.

    cell.placementType = 1; // BAD!
    cell.placementType = MBSProductPlacementTypeCenter; // A++!

If the number the enum value resolves to ever changes, you get the new value for free!

####Define

`#define` values should only be used for small, internal uses where one of the above symbol types seems like overkill. Define is the poor-man's constant.

Defines should be all uppercase, with underscores used as a delimiter. 

    #define CELL_X_OFFSET 25.0f

### Methods

Methods should be given descriptive names, where clarity is the most important feature. Names should be camel-cased with the initial letter lowercased.

Make method names as long as necessary, naming with hints towards the condition, cause, and return value of the method. Methods should read like sentences, and the developer should grasp exactly the purpose of the method and invocation simply by reading its name.

Selector pieces (i.e. the bits between the pieces, which are all really part of the selector) should be named without the use of `and`, `with`, etc. These conjunctions are not necessary and become excessive after only a few instances. Exceptions can be made for methods taking only one parameter.

    - (id)initWithFrame:(CGRect)frame; // OK
    - (id)initWithFrame:(CGRect)frame backgroundColor:(UIColor *)color; // OK
    - (id)initWithFrame:(CGRect)frame andBackgroundColor:(UIColor *)color; // bad

When creating new methods which provide more options to existing methods, list all the original parameters in your new method *first*, before adding additional parameters (as in the `-initWithFrame:backgroundColor:` example above). The only exception to this being when the final parameter of the existing method accepts a Block object, in which case your additional method must keep the Block object paramter as the final parameter.

    - (void)haveAPartyWithFriends:(NSArray *)friends cleanupHandler:(MBSCleanupBlock)handler;
    - (void)haveAPartyWithFriends:(NSArray *)friends byob:(BOOL)byob cleanupHandler:(MBSCleanupBlock)handler;

Selector signatures (in both interface and implementations) must follow the standard Cocoa whitespace pattern as demonstrated throughout this guide.

1. The method scope (`-` and `+` for instance or class method, respectively), followed by a space.
2. The return type of the method, with a space between the type and the star (this is really just like a cast, and all casts must follow this guideline). If there is no star, then there is no space. Double-star casts (e.g. `(NSError **)` should not have a second space.
3. The start of the selector should directly follow the cast *with no space*.
4. If the method takes parameters, they should follow a colon, their cast (with no space between the colon and the cast, and no space between the cast and the argument name).
5. After all parameters, there should be a semi-colon in the declaration.
6. After all parameters, there should be a space and then an opening brace on the same line in the implementation.


        - (NSString *)stringByMashingFirstName:(NSString *)firstName lastName:(NSString *)lastName;

Each method declaration should appear on its own line in an interface, below declared `@property` entries. Group related methods together, and add a line of whitespace between groups.

Finally, don't be abusive with the compiler:

* A return type must be provided, even though this is not required by the compiler.
* All parameters must be named, even though this is not required by the compiler.
* All parameters must be typed, even though this is not required by the compiler.

        - validButStupidlyNamedSelector:::; // This kills the developer.
        - (NSString *)stringByConstructingURLFromHost:(NSString *)host path:(NSString *)path;

#### Functions

Always name all arguments even in the header declaration. This assists with code completion and makes it easier for the developer to know which parameters do what.

    void someFunction(NSString *); // compiles, but so dirty
    void someBetterFunction(NSString *name); // delightful and satisfying.

## Whitespace

Avoid using extraneuous whitespace when unnecessary. The rule of thumb should be a single blank line between methods and chunks of code. Prefer using `#pragma mark`s between groups of methods to achieve rhythm/pacing and logical code separation.

### General

Use tabs instead of spaces (the default setting, which uses a tab size of 4). Xcode will try to assist you in keeping things properly indented. Follow its suggestions (pro tip: if Xcode is being funky about how it indents things, you've probably got a syntax error somewhere).

There is typically no reason to hard-wrap lines, even though Objective-C tends to be a verbose language, we also have wide displays, so there is rarely a need to wrap the lines manually. Turning on word-wrap helps for smaller displays.

If you feel the need to wrap a method line, either the signature or an invocation, you may do so but make sure the parameter colons line up properly (Xcode will try its best to do this for you), but generally this isn't necessary.

###Closing Brackets

When there's a closing bracket, do not include an extra line between the last line of code and the bracket.
    
	if (goodExample) {
		//do something
		return 0;
	} // good

	if (badExample) {
		//do something
		return -1;

	} //bad!

### Control Structures

Control structures (i.e. `if`, `for`, `while`, etc.) all require one space between the keyword and the open paren, and one space between the closing paren and the opening brace. There should be no padding spaces inside the parens before the actual condition.

    if (someCondition == otherThing) {
        NSLog(@"Great success!", nil);
    }

Assignments inside conditionals should be avoided, but in the case of `-init`, should be wrapped in double parentheses, as recommended by the Clang Fixit system.

Using control structures without braces should be avoided, as this can lead to hard to detect errors if additional lines are later added. If you decide to go without braces, prefer to keep your code all on one line:

    if (!condition) return; // bailing early.

#### Comparing against `nil` and buddies

When comparing a variable for equality against `nil` or some constant, always be explicit and put the constant value second. This helps with code readability.

    if (nil == hopefullyExistingObject) { ... // oops! bad.
    if (!hopefullyExistingObject) {       ... // also bad.
    if (hopefullyExistingObject == nil) { ... // more readable.

## Project Conventions

###Keep public API Simple, but testable.
The Public API is what's exposed in your header files. These header files should be split up into two seperate files. Public methods should be included in the .h file of the class. Internal methods which won't either be invoked by outside classes, or overidden by subclasses, should be included in a seperate .h file, and then be imported into the .m file. This may seem odd, but it allows testers to access the methods to verify the correctness of the methods, while keeping the private methods private from the public. This includes `@property` declarations. Public properties belong in the headers, but private ones should be declared in the private .h file.

This also means header files should not contain instance variable declarations, as they are really an implementation detail.

### Keep implementations ordered

This is an easy one. Try to keep methods in the implementation files grouped together logically, instead of strewn about the file. If there are a bunch of delegate methods implemented, put those together, and use a `#pragma mark - SomeDelegateProtocol methods` to mark the beginning of a section of methods.

For subclasses, try to keep your overrides grouped as well, in order of the hierarchy. For example, keep `NSObject` override methods first, then say, `UIViewController` overrides, then the subclass's methods.

### Class extensions

Class extensions are how internal methods and properties should be hidden out of the public interface and header file. They are kind of like anonymous categories. They are to be placed in the `.m` file before the main `@implementation` block. For a class named `ShopTableViewController`, the class extension might look like this:

```
@interface ShopTableViewController ()

@property (nonatomic, strong) NSString *internalString;

- (void)doSomethingInternal;

@end
```

The methods and properties declared in this extension become synthesized and implemented just like any other method or property in a public interface. There is no need to create a second, named implementation block.

### Comments

Comments are good in the public interface to describe what the public APIs do and what the methods' parameters and return values need to be (e.g. "This parameter must not be nil!").

For general code comments, they are often not as necessary. Rather than lots of comments, the methods should be descriptively named, as mentioned above. Comments in implementations can quickly become out of date, and are often reduntant.

Comments which explain *why* a chunk of code exists are much better than comments which explain *what* a chunk of code does. Most programmers can follow along a block of code fairly easily without comments, but the real benefit is explaining something like "We're shifting the bits here to set a flag, which is needed so networking turns on", or something to that effect.

When necessary, comments should be of the form `// Comment here` directly above the code block it details:

    // Only fetch messages from server if we're not already refreshing
    if (!self.refreshControl.isRefreshing) {
        [[APIClient sharedClient] getMessagesWithSuccess:nil failure:nil];
    }

As a general rule, if you believe your method requires comments to explain the steps of the process, it's probably doing too much, and should be refactored.

### Being a good Cocoa citizen

Being a good Cocoa citizen means following the conventions of the environment. We can get lots accomplished with the provided libraries. External frameworks like Three20 should be considered kind of harmful because they impose conventions from a different programming environment.

#### Blocks

When implementing a callback mechanism, use Block objects instead of delegation wherever possible. These cut down on the verbiage of having to declare a delegate protocol, then marking your class as conforming to the protocol, then implementing the delegate methods.

With Block objects, they are typically `#typedef`'d in a class header, and then implemented inline as they are needed. Much simpler.

Use caution with Block objects, as they automatically retain objects referenced inside them. Usually this isn't too much of a problem as they are also released when the Block is destroyed, but it can lead to retain cycles. Consider using `__weak SomeClass weakObject = someStrongObject` and referencing the weak version instead in the Block object to avoid such a cycle (if `someStrongObject` retains the Block object).

#### Delegates

Delegates are useful when fine granularity is needed in callbacks, or when multiple objects might be using the same callbacks, although they should generally be avoided in favour of Block object handlers.

#### Use Cocoa "primitives"

Try to use the declared Cocoa "primitives" instead of their plain C counterparts. This means using `NSInteger` and `NSUInteger` instead of `int` and `unsigned int`, or using `CGFloat` instead of `float`.

We do this because it gives better flexibility if Apple ever changes what they decide an `NSInteger` should refer to. The Cocoa types are really just typedefs which evaluate to different primative types depending on the archictecture they are compiled for. If Apple ever move iOS to a 64 bit platform, all uses of `NSInteger` will automatically be updated on the next compile to use the proper types. We won't have to modify our code nearly as much for any possible future processor architecture changes.

It also means we should use types like `NSTimeInterval`, which really evaluates to a `double`, but using the Apple provided types are better because they give more context as to what the variable represents.

#### Exceptions

Throwing exceptions in Cocoa is reserved for programmer error. If there is a possibility of something going wrong at runtime, your API should require an `NSError` pointer pointer and the method should return `NO` or `nil` on error condition, filling the error pointer too. Exceptions should not be used.

If your code crashes on an `Uncaught Exception`, this means there is something incorrect in your code (i.e. programmer error) and you need to fix this. A typical example would be an `ArrayOutOfBoundsException`. This is a mistake by the programmer, not a runtime error condition.

### ARC

All new projects should use Automatic Reference Counting. Existing projects should be upgraded as soon as possible.

For projects supporting iOS 5+, we should be using `weak` and `strong` properties. Though `retain` and `strong` are synonyms in ARC, `strong` should be used instead as it gives a better and more consistent indication of what the property is doing in terms of the object graph. It's also more consistent with the storage types (like `__weak`, `__strong`, etc.).

### Warnings and Errors

Warnings should be treated as errors, both by you and by the compiler. A warning is just a bug that maybe hasn't happened yet.

All Xcode projects should have the `-Werror` compiler flag turned on and left on. If we turn this flag on from the beginning, then fixing warnings should not be a problem. If we turn it on later in the project, we'll have more work to do.

Fix all the warnings as soon as they happen so they don't get out of control. 

### Xibs, Interface Builder and Storyboards

Interface Builder documents can be really useful and allow us to quickly prototype our interfaces, but they can usually only get us 90% of the way, with the other 10% being impossible without using code.

Xibs and Storyboard files are very difficult to merge if two or more developers have worked on them on different branches, causing massive headaches. Even `diff`ing xibs/storyboards can quickly get out of hand.

Due to these issues, using Interface Builder documents should be avoided for everything except quick prototyping (which will be later thrown out).

### Base SDK and Deployment Target

The project's `Base SDK` should always be set to `Latest iOS Version`, so we'll always be linking against the newest SDK code and symbols.

Our Deployment Target depends on the oldest possible OS version we'd like to support, e.g. only supporting back to iOS 5.0.

## Being a good boyscout

Finally, do your best to always be improving the code. Leave it in a better state than how you found it. If you notice a class is messy, then clean it up and make it adhere to this style guide.

## References

* [Jason Brennan's Original Guide][original]
* [Google][google]
* [Marcus Zarra][zarra]
* [Apple][apple]
* [Code Commandments][commandments]

[google]: http://google-styleguide.googlecode.com/svn/trunk/objcguide.xml
[zarra]: http://www.cimgf.com/zds-code-style-guide/
[commandments]: http://ironwolf.dangerousgames.com/blog/archives/913
[apple]: http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html
[apple2]: http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/APIAbbreviations.html#//apple_ref/doc/uid/20001285-BCIHCGAE
[original]: https://github.com/jbrennan/style
