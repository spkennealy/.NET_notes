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

Explicit Razor expressions consist of an @ symbol with balanced parenthesis. To render last week's time, the following Razor markup is used:
```html
<!-- .cshtml file --> 
<p>Last week this time: @(DateTime.Now - TimeSpan.FromDays(7))</p>
```

Any content within the `@()` parenthesis is evaluated and rendered to the output. Implicit expressions, described in the previous section, generally can't contain spaces.

Explicit expressions can be used to concatenate text with an expression result:
```html
<!-- .cshtml file --> 
@{
    var joe = new Person("Joe", 33);
}

<p>Age@(joe.Age)</p>
```

Without the explicit expression, <p>Age@joe.Age</p> is treated as an email address, and <p>Age@joe.Age</p> is rendered. When written as an explicit expression, <p>Age33</p> is rendered.

## **Expression encoding**

C# expressions that evaluate to a string are HTML encoded. C# expressions that evaluate to IHtmlContent are rendered directly through IHtmlContent.WriteTo. C# expressions that don't evaluate to IHtmlContent are converted to a string by ToString and encoded before they're rendered.

HtmlHelper.Raw output isn't encoded but rendered as HTML markup.
* Using HtmlHelper.Raw on unsanitized user input is a security risk. User input might contain malicious JavaScript or other exploits. Sanitizing user input is difficult. Avoid using HtmlHelper.Raw with user input.

## **Razor code blocks**

Razor code blocks start with @ and are enclosed by {}. Unlike expressions, C# code inside code blocks isn't rendered. Code blocks and expressions in a view share the same scope and are defined in order:
```html
<!-- .cshtml file --> 
@{
    var quote = "The future depends on what you do today. - Mahatma Gandhi";
}

<p>@quote</p>

@{
    quote = "Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.";
}

<p>@quote</p>
```

The code renders the following HTML:
```html
<p>The future depends on what you do today. - Mahatma Gandhi</p>
<p>Hate cannot drive out hate, only love can do that. - Martin Luther King, Jr.</p>
```

### **Implicit transitions**

The default language in a code block is C#, but the Razor Page can transition back to HTML:
```html
<!-- .cshtml file --> 
@{
    var inCSharp = true;
    <p>Now in HTML, was in C# @inCSharp</p>
}
```

### **Explicit delimited transition**

To define a subsection of a code block that should render HTML, surround the characters for rendering with the Razor `<text>` tag:
```html
<!-- .cshtml file --> 
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    <text>Name: @person.Name</text>
}
```

Use this approach to render HTML that isn't surrounded by an HTML tag. Without an HTML or Razor tag, a Razor runtime error occurs.

The `<text>` tag is useful to control whitespace when rendering content:

Only the content between the `<text>` tag is rendered.
No whitespace before or after the `<text>` tag appears in the HTML output.

### **Explicit Line Transition with @:**

To render the rest of an entire line as HTML inside a code block, use the @: syntax:
```html
<!-- .cshtml file --> 
@for (var i = 0; i < people.Length; i++)
{
    var person = people[i];
    @:Name: @person.Name
}
```

Without the @: in the code, a Razor runtime error is generated.

Warning: Extra @ characters in a Razor file can cause compiler errors at statements later in the block. These compiler errors can be difficult to understand because the actual error occurs before the reported error. This error is common after combining multiple implicit/explicit expressions into a single code block.

## **Control structures**

Control structures are an extension of code blocks. All aspects of code blocks (transitioning to markup, inline C#) also apply to the following structures:

### **Conditionals @if, else if, else, and @switch**

`@if` controls when code runs:
```html
<!-- .cshtml file --> 
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
```

else and else if don't require the @ symbol:
```html
<!-- .cshtml file --> 
@if (value % 2 == 0)
{
    <p>The value was even.</p>
}
else if (value >= 1337)
{
    <p>The value is large.</p>
}
else
{
    <p>The value is odd and small.</p>
}
```

The following markup shows how to use a switch statement:
```html
<!-- .cshtml file --> 
@switch (value)
{
    case 1:
        <p>The value is 1!</p>
        break;
    case 1337:
        <p>Your number is 1337!</p>
        break;
    default:
        <p>Your number wasn't 1 or 1337.</p>
        break;
}
```

### **Looping @for, @foreach, @while, and @do while**

Templated HTML can be rendered with looping control statements. To render a list of people:
```html
<!-- .cshtml file --> 
@{
    var people = new Person[]
    {
          new Person("Weston", 33),
          new Person("Johnathon", 41),
          ...
    };
}
```

The following looping statements are supported: `@for`, `@foreach`, `@while`, `@do while`

### **Compound @using**

In C#, a using statement is used to ensure an object is disposed. In Razor, the same mechanism is used to create HTML Helpers that contain additional content. In the following code, HTML Helpers render a form tag with the @using statement:
```html
<!-- .cshtml file --> 
@using (Html.BeginForm())
{
    <div>
        email:
        <input type="email" id="Email" value="">
        <button>Register</button>
    </div>
}
```

### **@try, catch, finally**

Exception handling is similar to C#:
```html
<!-- .cshtml file --> 
@try
{
    throw new InvalidOperationException("You did something invalid.");
}
catch (Exception ex)
{
    <p>The exception message: @ex.Message</p>
}
finally
{
    <p>The finally statement.</p>
}
```

### **@lock**

Razor has the capability to protect critical sections with lock statements:
```html
<!-- .cshtml file --> 
@lock (SomeLock)
{
    // Do critical section work
}
```

### **Comments**

Razor supports C# and HTML comments:
```html
<!-- .cshtml file --> 
@{
    /* C# comment */
    // Another C# comment
}
<!-- HTML comment -->
```

Razor comments are removed by the server before the webpage is rendered. Razor uses @* *@ to delimit comments. 

## **Directives**

