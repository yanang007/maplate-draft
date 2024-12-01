{{Maplate}}
-----------
`Maplate` is a logic-ful templating engine. It is simplified for basic text interpolations (e.g. config rendering, code generation) while providing support for complex control flow (like `if`, `for`, `while` in Python-like style).

`Maplate` comes with a command line interface `mapl` to be able to directly render a template from input without the need of a programming language. `mapl` makes it easy to integrate with other systems for tasks like config rendering or code generation.

Script below shows how to render a template with environment variables:
```bash
$ NAME=Bob mapl -d ENV -s "Hello! {{NAME}}!"
Hello! BOB!
```

Usage
-----
Given input data in JSON format:
```json
// input.json
{
    "name": "Bob",
    "info": {
        "role": "admin",
        "items": ["apple", "banana", "cherry"]
    }
}
```

A typical `.mpl` template file may look like:
```
Hello, {{name | ToUpper}}!

{{if info.role == "admin"}}
You are an admin.
You have following items:
{{for i, item in info.items}}
    {{i + 1}}. {{item}}
{{/for}}
{{else}}
You are not allowed to access this page.
{{/if}}
```

Now it can be compiled with the following command:
```bash
$ mapl template.mpl -d input.json -o output.txt
```

The output will be:
```
Hello, BOB!

You are an admin.
You have the following items:
    1. apple
    2. banana
    3. cherry
```

Syntax
------
### Basics
Every statement should be enclosed in `{{` and `}}`. Anything outside of these delimiters will be treated as plain text and will be copied to the output as is.

```
Hello, {{name}}!  // Output: Hello, Bob!
{{receiver = "Unknown"}}
{{  // Code can be written in multiple lines
    if name == "Bob"
        receiver = "Alice"
    /if
}}
{{receiver}}       // Output: Alice
```

#### Expressions
Expressions are treated as output variables (like an extended version of `{{var}}` in Mustache).

```
{{name}}            // Output: Bob
{{name + "!"}}      // Output: Bob!
{{1 + 1}}           // Output: 2
```

* Filters are functions that can be applied to variables. For example, `ToUpper` and `ToLower` are built-in filters that convert a string to upper case and lower case, respectively. Filters can be chained.

```
{{ToUpper(name)}}   // Output: BOB
{{name | ToUpper}}  // Output: BOB
{{name | ToUpper | ToLower}}  // Output: bob
```

* `undefined` and `null` will be treated as empty string. Null-coalescing operator `??` can be used to provide a default value.
```
{{null}}                               // Output: 
{{someUndefinedVariable}}              // Output: 
{{someUndefinedVariable or "default"}} // Output: default
```

#### Variables
Variables can be declared, assigned and referenced in the template file. Variables are local to the template file and is not visible in the output if not explicitly printed with [expressions](#expressions).

```
{{a, b = 1, 2}}
{{a}}           // Output: 1
{{b}}           // Output: 2
{{b += a}}
{{b}}           // Output: 3
```

#### Sections
Contents capured by statements like `{{if}}` or `{{for}}` becomes a section. A section can be rendered multiple times or not at all, depending on the condition.

##### If-Else
Conditional statements can be used to control the flow of the template (like `{{#flag}}` in Mustache). The syntax is similar to Python.

```
{{if name == "Bob" or name == "Alice"}}
    You are Bob.
{{elif name == "Alice"}}
    You are Alice.
{{else}}
    I don't know you.
{{/if}}
```

##### For
Loop statements can be used to iterate over a list or a range of numbers (like `{{#items}}` in Mustache).

```
{{for i in Range(3)}}
    {{i + 1}}. Hello!
{{/for}}

// Output:
    1. Hello!
    2. Hello!
    3. Hello!
```

- `Foreach` loop with index:
```
{{for i, item in Enumerate(info.items)}}
    {{i + 1}}. {{item}}
{{/for}}

// Output:
    1. apple
    2. banana
    3. cherry
```

##### While
While loop statement can be used to repeat a block of code while a condition is true.

```
{{
    items = ["apple", "banana", "cherry"]
    left, right = 0, Len(items) - 1
}}
{{while left <= right}}
    You got {{items[left]}} and {{items[right]}}!
    {{left, right = left + 1, right - 1}}
{{/while}}

// Output:
    You got apple and cherry!
    You got banana and banana!
```

#### Include
Templates can be included into current template (like `{{> file}}`/partial in Mustache or `#include` in C/C++).

```
{{if authorized}}
    {{include "content.mpl"}}
{{/if}}
```

##### Slot [TODO]
Slot like Vue [TODO]

### Operators
Aside from basic arithmetic operators (`+`, `-`, `*`, `/`, `%`) and logic operators (`==`, `!=`, `>`, `<`, `>=`, `<=`, `and`, `or`, `not`, `in`), `Maplate` also supports some special operators:

-  `?.`, `??`: Null coalescing operator, e.g. `{{a?.b ?? "default"}}` will return `a.b` if both `a` and `a.b` are not `null`, otherwise `"default"`.

Extension
---------
`Maplate` is designed to be extensible and easy to integrate with other systems. You can define your own filters, functions, and even operators. 

### Functions and filters 
Filters are functions that can be applied to variables. You can define your own filters by creating a Python file with the following structure:

```go
# my_filters.go
func Emphasize(s string) string {
    return "*" + s + "*"
}

func main() {
    engine := mapl.NewEngine()
    engine.RegisterFilter("Emphasize", Emphasize)

    fmt.Println(engine.Render("{{name | Emphasize}}")) // Output: *Bob*
}
```

### Operators [TODO]
[TODO]

Ahead-of-Time Compilation [TODO]
-------------------------
`Maplate` is designed to be a static text generator, which means it can compile a `.mpl` template into a standalone executable file with the power of Golang. This is especially useful when you want to distribute the template file without the need to install `Maplate` and any runtime on target machine.

Similar Project
---------------
This project is heavily inspired by the following projects, thanks to their awesome work!

- [Inja](https://github.com/pantor/inja): A template engine for modern C++. `Maplate` is designed around the same concept as `Inja`, but with a simplified Python-like syntax.
- [Mustache](https://mustache.github.io/): A logic-less template syntax.
- [Handlebars](https://handlebarsjs.com/): A superset of Mustache with logic.
- [moban](https://github.com/moremoban/moban): A general purpose static text generator.
