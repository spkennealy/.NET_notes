# Razor syntax reference for ASP.NET Core

Razor is a markup syntax for embedding server-based code into webpages. The Razor syntax consists of Razor markup, C#, and HTML. Files containing Razor generally have a .cshtml file extension.

## **Razor syntax**
Razor supports C# and uses the @ symbol to transition from HTML to C#. Razor evaluates C# expressions and renders them in the HTML output.

When an `@` symbol is followed by a Razor reserved keyword, it transitions into Razor-specific markup. Otherwise, it transitions into plain C#.

To escape an `@` symbol in Razor markup, use a second `@` symbol:
```html
<!-- .cshtml file --> 
<p>@@Username</p>
```

The code is rendered in HTML with a single @ symbol:
```html
<!-- .html file --> 
<p>@Username</p>
```

HTML attributes and content containing email addresses don't treat the @ symbol as a transition character. The email addresses in the following example are untouched by Razor parsing:
```html
<!-- .cshtml file --> 
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

## **Implicit Razor expressions**

Implicit Razor expressions start with @ followed by C# code:

```html
<!-- .cshtml file --> 
<p>@DateTime.Now</p>
<p>@DateTime.IsLeapYear(2016)</p>
```

With the exception of the C# await keyword, implicit expressions must not contain spaces. If the C# statement has a clear ending, spaces can be intermingled:
```html
<!-- .cshtml file --> 
<p>@await DoSomething("hello", "world")</p>
```

Implicit expressions cannot contain C# generics, as the characters inside the brackets (<>) are interpreted as an HTML tag. The following code is not valid:
```html
<!-- .cshtml file --> 
<p>@GenericMethod<int>()</p>
```

The preceding code generates a compiler error similar to one of the following:

* The "int" element wasn't closed. All elements must be either self-closing or have a matching end tag.
* Cannot convert method group 'GenericMethod' to non-delegate type 'object'. Did you intend to invoke the method?

Generic method calls must be wrapped in an explicit Razor expression or a Razor code block.

## **Explicit Razor expressions**

