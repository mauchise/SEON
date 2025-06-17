# SEON: S-Expression Object Notation
> Version: Draft v0.1

SEON is a modern, efficient data notation format that is extremely friendly to both humans and machines.  
We believe data expression should return to the essence of being concise, efficient, and readable.  
Drawing inspiration from Lisp’s S-expressions, SEON aims to be a more elegant and powerful alternative to
markup formats such as JSON, XML, and YAML.

## Why Choose SEON?
In the evolution of data-exchange and configuration-file formats, we have made too many compromises
between “machine-friendly” and “human-readable,” introducing a lot of unnecessary syntactic noise.
SEON is designed to break this status quo.

- **Simplicity First**: The core syntax of SEON consists only of parentheses `()` and whitespace.
  No commas, colons, or mandatory double quotes. Data—not syntax—becomes the focus.
- **High Signal-to-Noise Ratio**: Every character serves structure or data.  
  Compared with JSON’s verbose `{"key": "value"}`, SEON’s `(key value)` is far more direct.
- **Naturally Structured**: Nested S-expressions are trees by nature; hierarchy is obvious
  without guessing or relying on indentation.
- **Easy to Parse**: SEON’s rules are extremely simple and consistent, making parsers easy to write
  and very performant.
- **Multi-scenario Adaptability**: SEON is fully compatible with mainstream configuration-file concepts
  and is an excellent choice for configs, front-/back-end communication, DSLs, and more.

### SEON vs. Mainstream Markup Formats

| Feature         | SEON                               | JSON                    | XML                    | YAML                   |
| :-------------- | :--------------------------------- | :---------------------- | :--------------------- | :--------------------- |
| **Basic Structure** | Object `{}` / S-expression `(key …)` | Object `{}` / Array `[]` | Tag `<tag>`            | Indented key-value     |
| **Syntax Noise**    | Very low                         | Medium (quotes, commas, braces) | High (angle brackets, end tags) | Low (but indentation-sensitive) |
| **Data Types**      | Explicit `#` prefix              | Built-in support        | All strings (needs schema) | Auto inference (error-prone) |
| **Comments**        | Supported (`;`)                  | Not supported           | Supported `<!-- -->`   | Supported `#`          |
| **Readability**     | Very high (once accustomed)      | Good                    | Verbose                | Good (but indentation pitfalls) |
| **Extensibility**   | Macros & Schema built-in         | Via external standards  | Powerful (DTD, XSD)    | Weak                   |

## Understanding SEON
Let’s explore SEON’s core syntax through a complete blog post (with metadata and rich-text content).

```racket
; Nested objects are wrapped by {}.
; {} is sugar for (#object ...).
{
  ; Format is (key ...values). A single value is treated as a scalar.
  (id post-123)
  (title `SEON: A New Era for Data`)

  ; Multiple values automatically form a list.
  (tags data-format lisp s-expression seon)

  ; If logically a list but only one element, or empty, use () explicitly.
  (co-authors ({
    (id user-9527)
    (name `Alex Raymond`)
    (age #nil) ; null value
  }))
  (empty-list ())

  (theme-color \#facccc) ; escape with \
  (published #true)

  (reading-time #15.5) ; float
  (views #2.4e3)       ; scientific notation
  (reading-stats
    (2025-01 #1200 #300)
    (2025-02 #1350 #320)
    (2025-03 #1600 #380)
  )

  (content
    (h1 `Welcome to the World of SEON`)
    
    (p 
      `SEON is a data format based on S-expressions.`
      `Its goal is to achieve both ` (b `human readability`) ` and ` (b `machine friendliness`)
      `, completely eliminating the syntactic noise of JSON or XML.`
    )
    
    (h2 `Core Features`)
    
    (ul
      (li `Ultra-minimal syntax`)
      (li `High signal-to-noise ratio`)
      (li `Naturally structured, easy to nest`)
    )
    
    (p 
      `For more information, visit `
      (link {(href `https://github.com/mauchise/SEON`)} `the official repository`)
      `.`
    )
  )
}
```

Its JSON equivalent:

```json
{
  "id": "post-123",
  "title": "SEON: A New Era for Data",
  "tags": ["data-format", "lisp", "s-expression", "seon"],
  "co-authors": [{
    "id": "user-9527",
    "name": "Alex Raymond",
    "age": null
  }],
  "empty-list": [],
  "theme-color": "#facccc",
  "published": true,
  "reading-time": 15.5,
  "views": 2.4e3,
  "reading-stats": [["2025-01", 1200, 300], ["2025-02", 1350, 320], ["2025-03", 1600, 380]],
  "content": [
    ["h1", "Welcome to the World of SEON"],
    ["p",
      "SEON is a data format based on S-expressions.",
      "Its goal is to achieve both ", ["b", "human readability"], " and ", ["b", "machine friendliness"],
      ", completely eliminating the syntactic noise of JSON or XML."
    ],
    ["h2", "Core Features"],
    ["ul",
      ["li", "Ultra-minimal syntax"],
      ["li", "High signal-to-noise ratio"],
      ["li", "Naturally structured, easy to nest"]
    ],
    ["p",
      "For more information, visit ",
      ["link", {"href": "https://github.com/mauchise/SEON"}, "the official repository"],
      "."
    ]
  ]
}
```

## Syntax Essentials
SEON’s syntax revolves around one core idea: **everything is an S-expression**.
An S-expression is a list enclosed by parentheses `()` with elements separated by whitespace.

### 1. Basics & Comments

- **S-expressions**: A SEON document is composed of one or more S-expressions.
- **Separators**: Elements inside an expression (keys, values, etc.) are separated by one or more
  spaces, newlines, or tabs.
- **Comments**: The semicolon `;` marks a single-line comment; everything from `;` to end of line
  is ignored.

```racket
; This is a comment and will be ignored.
{ ; object starts here
  (key value) ; a key-value pair
}
```

### 2. Objects
Objects—collections of key-value pairs—are the core data structure in SEON.

- **Syntax**: Objects are wrapped by `{}` or written as the native form `(#object …)`.  
  Braces are sugar for the latter and are recommended.
- **Key-value Pair**: Inside an object, each S-expression `(key …values)` is treated
  as a key-value pair.
  - `key` is a string or symbol (see below).
  - `…values` are one or more values following the key.

#### Forms of Values
Depending on the number of values after the key, semantics are determined automatically:

- **Single Value (Scalar)**

  ```racket
  (id post-123)         ; string "post-123"
  (published #true)     ; boolean true
  (author { (id u-1) }) ; another object
  ```

- **Multiple Values (List)**

  ```racket
  (tags data-format lisp seon) ; ["data-format", "lisp", "seon"]
  (co-authors {..} {..})       ; [ {..}, {..} ]
  ```

- **Explicit List**

  ```racket
  (tags (data-format)) ; ["data-format"]
  (co-authors ())      ; []
  ```

### 3. Lists

Besides appearing as values within objects, lists can also exist independently,
often representing rich-text content or tuple collections.  
An S-expression outside an object—`(...)`—is usually parsed as a list.

```racket
; reading-stats is a list, each element is also a list
(reading-stats
  (2025-01 #1200 #300) ; ["2025-01", 1200, 300]
  (2025-02 #1350 #320) ; ["2025-02", 1350, 320]
)

; content is a list of various S-expressions
(content
  (h1 `Welcome ...`) ; ["h1", "Welcome ..."]
  (p `SEON is ...`)  ; ["p", "SEON is ..."]
  (ul ...)
)
```

### 4. Data Types
SEON supports rich data types. Strings are written directly, while other explicit types
are marked with the `#` prefix.

#### a. Strings
- **Bare Strings**
  - Used for keys and simple enum-like values without spaces or special chars.
  - May contain letters, digits, and symbols like `-` or `.`, but not spaces, `()`, `{}`, `;`, `#`, or `` ` ``.
  - Examples: `id`, `post-123`, `h1`, `theme-color`

- **Back-quoted Strings**
  - Enclosed by back-quotes `` ` `` and can contain spaces and newlines.
  - Ideal for paragraphs, headings, and other rich-text content.
  - Escape a back-quote inside with `\` as in `\`\``.
  - Example: `` `SEON: A New Era for Data` ``

#### b. `#` Type Prefix
To avoid ambiguous type inference (e.g., YAML’s `no` → `false`),
SEON requires explicit `#` for non-string atomic types.

| Expression            | Meaning       | JSON Equivalent     |
| :-------------------- | :------------ | :------------------ |
| **Numbers**           |               |                     |
| `#123`                | Integer       | `123`               |
| `#-45`                | Negative int  | `-45`               |
| `#3.14`               | Float         | `3.14`              |
| `#6.022e23`           | Scientific    | `6.022e23`          |
| **Boolean**           |               |                     |
| `#true`               | True          | `true`              |
| `#false`              | False         | `false`             |
| **Null**              |               |                     |
| `#nil`                | Null          | `null`              |
| **Constants**         |               |                     |
| `#inf`, `#-inf`       | ±Infinity     | *implementation*    |

### 5. Escaping
Backslash `\` is the escape character in SEON.  
When you need to use a reserved character inside a bare string or symbol,
escape it to remove its special meaning.

- **Scenario**: You want to start a bare value with `#` that isn’t a type mark.

  ```racket
  (theme-color \#facccc) ; "#facccc" instead of a syntax error
  ```

- **Characters to Escape**: `(`, `)`, `{`, `}`, `;`, `#`, `\`, and `` ` `` when necessary.

## Future Plans
SEON’s ambitions go far beyond being “a prettier JSON.”  
We are building a full ecosystem that makes data processing simpler and more powerful than ever.

- **Native Schema System**: Define structure directly inside data files with macros such as
  `#define` and `#list`, enabling self-describing and self-validating data.
- **Type Constraints**: Define strong types (`number`, `text`, `bool`, etc.) for schema fields
  and set default values.
- **Modularity & Imports**: Reuse external schema definitions via `#import`,
  building composable and maintainable data models.
- **Editor/IDE Support**: We aim to support VS Code, JetBrains IDEs, and others with
  - Syntax highlighting
  - Formatting tools
  - Schema-based auto-completion and linting
- **Official Parsers**: High-performance reference implementations for
  Go, Rust, Python, and TypeScript/JavaScript.

## Contributing
We warmly welcome all developers interested in SEON!  
Whether you want to improve the spec, write code, create tools, or author docs,
your help is invaluable.

- **Discussion**: Start a thread in GitHub Discussions for any ideas or suggestions.
- **Report Issues**: Submit Bugs or spec ambiguities via GitHub Issues.
- **Contribute Code**:
  - Fork this repo.
  - Create a feature branch (`git checkout -b feature/your-amazing-idea`).
  - Commit your changes (`git commit -m 'Add some amazing feature'`).
  - Push to your fork (`git push origin feature/your-amazing-idea`).
  - Open a Pull Request.

## License
This project is licensed under the MIT License.
