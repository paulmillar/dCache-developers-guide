# dCache coding style

This document describes the coding style of the dCache project.

At the time of writing this document, the dCache project has been around for almost 20 years.  During that period, the Java language has evolved considerably and the code base as matured.  Therefore, the code does not have a truly uniform style.

That said, there are certain styles and rules we have adopted.

## Code-changing patchs do not change style \(and vice versa\)

A patch should have a specific focus and avoid doing extraneous things.  Changing existing code to follow a particular rule as described here is one such focus, but such patches should not also change how dCache behaves by modifying code at the same time.

## Uniformity beats correctness

If a java file has a particular style that is at odds with this document, changes to that file should follow the style in the file, not this guide.

## No tabs

There are no tab character in any dCache source code. Space characters are used instead.

