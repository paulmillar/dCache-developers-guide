# dCache coding style

This document describes the coding style of the dCache project.

At the time of writing this document, the dCache project has been around for almost 20 years.  During that period, the Java language has evolved considerably and the code base as matured.  Therefore, the code does not have a truly uniform style.

That said, there are certain styles and rules we have adopted.

## Code-changing patchs do not change style \(and vice versa\)

A patch should have a specific focus and avoid doing extraneous things.  Changing existing code to follow a particular rule as described here is one such focus, but such patches should not also change how dCache behaves by modifying code at the same time.

## Uniformity beats correctness

If a java file has a particular style that is at odds with this document, changes to that file should follow the style in the file, not this guide.

## General comments

Some comments apply generally, not just to Java code.

### No tabs

There are no tab character in any dCache source code. Space characters are used to indent text instead.

### No trailing white space

All lines should either be empty or end with a non-whitespace character.

## Java specific comments

These are some comments specific to Java.

### Indentation

A single logical indentation is four spaces.

A logical line that is less than 80 characters should not be split over multiple lines.  Lines longer than 80 characters should be split over multiple lines.

If a logical line is split over multiple physical lines, the second and subsequent lines should have two logical indentations \(8 spaces\)

```
String combined = foo.buildData(item1, item2) + bar.buildData(item1, item2)
        + baz.buildData(item1, item2);
```

When breaking a line, it is preferable that the indented lines begin with an element linking them together.  For example:

```
Foo value = storage.isValueAvailable
        ? storage.getValue()
        : null;
```

or

```
Foo value = storage.getContainer(item).getLogicalSubItemType()
        .getCompatibleTypeList();
```

### Imports

Import lines are located under the package declaration with one empty line separating the two.  They are organised into upto eight blocks:

1. &lt;all other non-static imports&gt;
2. javax
3. java
4. diskCacheV111
5. dmg
6. gov.fnal
7. org.dcache
8. &lt;all static imports&gt;

Each non-empty block is seperated from other non-empty block by a blank line.  Imports within a block are sorted alphabetically.

Static star imports are allowed if there would otherwise be three or more static imports from that class.

### Class declaration

There are currently two acceptable forms for class declarations.  Either with symmetic braces

```
public class Foo extends Bar implements Baz
{
    // class details go here
}
```

or "K&R" style with the open-brace appearing on the last line of the class' signature, separated by a space:

```
public class Foo extends Bar implements Baz {
    // class details go here
}
```

In either case, the close brace has the same indentation as the first character in the class declaration.

All content within the class definition has at least one logical indentation.

```
public class Foo
{
    private final String name;
    private int counter;

    public Foo(String name)
    {
        this.name = name;
    }
    
    public String getName()
    {
        counter++;
        return name;
    }
    
    public int getCount()
    {
        return count;
    }
}
```

Classes should have JavaDoc comment describing the intended purpose of the class.

Class names should be capitalised camal-case \(e.g., `ScriptNearlineStorage`\).  Although not a requirement, it is common that a class that implements some interface or extends some exist class adds a qualifier to the start of the name \(e.g., `ScriptNearlineStorage` implements `NearlineStorage`\).

## Field members

All field members are declared before any methods.

Static field members are declare before any non-static field member.  Static field members should have all-caps \(e.g., `MINIMUM_RELEASE_VERSION`\).

Non-static field member names should have camal-caps \(e.g., `minimumReleaseVersion`\).  They should be declared `private`, unless there's a very good reason.  No initialiser is written if the Java default value is intended \(e.g., write `private boolean enableFeature;` and not `private boolean enableFeature = false;`\)

Field members should be declared `final` when they are set during the object's construction and not modified subsequently.

## Method declaration

Method names should follow camal-caps \(e.g., myMethod\).

There should be no space between:

* the method name and the open-parenthsis,
* the open-parenthsis and the first character of any argument
* the last character of any argument and the close-parenthesis.

There should be a single space between any qualifiers, the return type, the method name-and-arguments and the open-brace \(if adopting the K&R style\).

The @Override annotation should be included if a method overrides some existing method or provides the implementation of an abstract method, or the method declaration in an interface.

There are two acceptable placements of braces.  Either symmetric:

```
public void acceptVisitor(Visitor visitor)
{
    // process visitor.
}
```

or K&R style:

```
public void acceptVisitor(Visitor visitor) {
    // process visitor
}
```



