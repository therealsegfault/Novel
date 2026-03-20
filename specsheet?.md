# Novel Language Specification
### Version 0.1 — Draft

Novel is a beginner-friendly scripting language designed to read like natural conversation.
It runs inside the H++ shell environment.

---

## File Structure

Every Novel script is a `.book` file. The structure is:

```
appN <identifier>       ← app identity declaration (required, must be first)
declare ...             ← variable declarations (optional, near top)

... body ...

bye                     ← end of file (required, must be last)
```

---

## App Identity

```novel
appN com.apple.clock
```

Declares the identity of this script as an app. Follows reverse-domain naming convention
(like Java packages). Not yet enforced by H++, but reserved for future package/environment
management (similar to Python venvs).

Run from the H++ shell with:

```
run apple/clock
```

---

## Variables

### Declaration

Variables must be declared before use. Declarations belong near the top of the file.
Declaration and assignment are always separate.

```novel
declare.Int score
declare.Str name
declare.Float ratio
declare.Bool boxTicked
declare.List items
declare.Map settings
declare.Func greet
```

### Types

| Keyword   | Type             | Example values         |
|-----------|------------------|------------------------|
| `Int`     | Integer          | `0`, `42`, `-7`        |
| `Str`     | String           | `"hello"`              |
| `Float`   | Decimal number   | `3.14`, `0.5`          |
| `Bool`    | Boolean          | `yes` / `no`           |
| `List`    | Ordered collection | TBD                  |
| `Map`     | Key-value pairs  | TBD                    |
| `Func`    | Function         | (see Functions)        |

Types can also be inferred without declaration:

```novel
score == 10             ← inferred Int
name == "Aria"          ← inferred Str
```

### Assignment

`==` means *"this is"*. It is the assignment operator.

```novel
score == 10
name == "Aria"
boxTicked == yes
```

### Comparison

`are` signals a comparison. `==` still means *"is"* in this context.

```novel
are boxTicked == yes
are score == 10
are name == "Aria"
```

---

## Input & Output

### User Input — `who`

Prompts the user for input. Whatever is written in `()` becomes the prompt label.
H++ renders it with a colon appended.

```novel
who (Birthday?)
```

Displays in H++ as:
```
Birthday?:
```

### System Fetch — `Really`

Fetches a value from the system or OS level.

```novel
Really time.clock           ← fetches current system clock value
Really cpu.temperature      ← fetches CPU temperature (example)
```

`time.clock` specifically queries the program's internal runtime clock.

### Querying — `ask`

Fires a `Really` expression or retrieves a `who` input, storing the result
in `ask.LastAns`.

```novel
ask.(Really time.clock)         ← fetch system clock, store result
ask (who [Sandwich])            ← retrieve what the user typed for "Sandwich"
```

`ask.LastAns` always holds the result of the most recent `ask` call.

### Output — `say`

Prints a value or string to the console.

```novel
say ask.LastAns
say "Hello!"
say "Hi, (ask.LastAns)!"        ← string interpolation using ()
```

---

## Functions

Declared with `func`, closed with `bye func;`. Parameters use `who`.
Semicolons mark the function signature open and close.

```novel
declare.Func greet

func greet;
who (What is your name)
ask (who [What is your name])
say "Hi, (ask.LastAns)!"
bye func;
```

H++ displays the `who` prompt as:
```
What is your name:
```

Functions can use `say`, `ask`, `who`, `Really`, and other Novel constructs internally.

---

## Program Termination

`bye` marks the end of file and signals H++ to cleanly exit the script.
It is required and must be the last line.

```novel
bye
```

Inside functions, `bye func;` closes the function block (not the script).

---

## Full Example

```novel
appN com.apple.clock

declare.Str userName

func greet;
who (What is your name)
ask (who [What is your name])
say "Hi, (ask.LastAns)! The current time is:"
bye func;

ask.(Really time.clock)
say ask.LastAns

bye
```

---

## H++ Shell Reference

| Command          | Description                        |
|------------------|------------------------------------|
| `run apple/clock` | Run the `com.apple.clock` script  |

---

## Appendix — Reserved Keywords

`appN`, `declare`, `==`, `are`, `who`, `ask`, `Really`, `say`, `func`, `bye`,
`yes`, `no`, `Int`, `Str`, `Float`, `Bool`, `List`, `Map`, `Func`

---

*Novel v0.1 — Subject to change as the language evolves.*
