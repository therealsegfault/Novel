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

## Calling Functions

Functions can be invoked three ways:

```novel
shout greet                      ← fire and forget, return data is discarded
ask.(greet)                      ← fire, result stored in ask.LastAns
remember greet as result         ← fire, result stored in named variable
```

**`shout`** — direct invocation, no storage. Loud, immediate, doesn't care about the return.

**`ask`** — universal getter. Fires the function and stores the result in `ask.LastAns`.

**`remember`** — fires and stores in one step (see Memory).

Arguments are passed in brackets:

```novel
ask.(greet ["Aria"])
shout greet ["Aria"]
```

---

## Memory & Storage

`remember` stores a value. Where it goes depends on the syntax:

```novel
remember (ask.LastAns) as userName             ← named variable, in-memory
remember (ask.LastAns) as notes/pizza.md       ← written to file
remember (ask.LastAns)                         ← auto-named shortterm1, shortterm2...
```

### Shortterm slots

When no target is given, Novel auto-assigns `shortterm1`, `shortterm2`, etc. in sequence.
Shortterm slots are flushed when `bye` is called — they exist only for the lifetime of the script.

```novel
ask (shortterm1)                ← retrieve shortterm slot 1
say ask.LastAns
```

### Files

Files persist past `bye`. Extension determines format (`.md`, `.txt`, `.book`, etc.).
H++ writes relative to the app's working directory.

### Memory tiers

| Storage         | Syntax                              | Persists past `bye`? |
|-----------------|-------------------------------------|----------------------|
| Named variable  | `remember (x) as score`            | No                   |
| Shortterm slot  | `remember (x)`                     | No — flushed on bye  |
| File            | `remember (x) as notes/pizza.md`   | Yes                  |

---

## Fetching Data from Disk

Use `fetch:` as a protocol prefix inside `ask` to read from disk.
Supports standard path syntax including `~` for home directory.

```novel
ask (fetch:~/Documents/wholikespizza.md)
say ask.LastAns
```

`fetch` is provided by the platform module (see Modules).

---

## Modules & Imports

Use `open` to import a module. This is Novel's import system.

```novel
open linux.fetch_64bit
open math.advanced
open com.myapp.utils
```

### Builtins — always available, no `open` needed

- Core keywords: `say`, `ask`, `who`, `Really`, `remember`, `shout`, `bye`, `are`
- `math.*` — arithmetic and expressions
- Language parsing internals
- **Platform modules** — H++ automatically pulls in the correct platform namespace based on the host OS:

| OS      | Auto-imported namespace |
|---------|------------------------|
| Linux   | `linux.*`              |
| macOS   | `macos.*`              |
| Windows | `windows.*`            |

Architecture defaults to 64bit unless the H++ installer specifies otherwise.
This means `fetch:` works out of the box on all platforms — no explicit `open` required
on standard installs.

### Modules — require `open`

Anything outside the builtins must be explicitly opened. Examples:

| Module              | Provides                        |
|---------------------|---------------------------------|
| `linux.fetch_64bit` | Disk read (explicit override)   |
| `math.advanced`     | Extended math operations        |
| `io.console`        | Extended console output         |
| `com.myapp.utils`   | App-specific utilities          |

---

## H++ Shell Reference

| Command                     | Description                                        |
|-----------------------------|----------------------------------------------------|
| `run apple/clock`           | Run the `com.apple.clock` script                   |
| `pkg get linux.fetch_64bit` | Install a module from the H++ package registry     |

---

## H++ Environment

### Platform & Architecture

H++ runs natively on Linux, macOS, and Windows. On install, it detects the host OS
and architecture, auto-importing the correct platform namespace. Defaults to 64bit
unless the installer specifies otherwise.

### Package Manager — `pkg get`

Used in the H++ shell to install modules that aren't native to the current platform.

```
pkg get linux.fetch_64bit
```

Once installed, the module is available to any Novel script via `open`.

Cross-platform availability:

| Install | Native namespace | `pkg get` unix modules | Sees Windows FS |
|---------|-----------------|------------------------|-----------------|
| Linux   | `linux.*`       | N/A — already native   | ❌              |
| macOS   | `macos.*`       | ✅ (BSD/Unix layer)    | ❌              |
| Windows | `windows.*`     | ❌ — no Unix layer     | ✅              |
| WSL     | `linux.*`       | N/A — already native   | ❌              |

Windows cannot use `linux.*` or `macos.*` modules — there is no underlying Unix layer
to hook into. No WSL bridge is provided.

### Installing H++ on WSL

H++ can be installed inside a WSL environment using the Linux package manager of
the active distribution:

```
apt install hplusplus
paru -S hplusplus
dnf install hplusplus
```

A WSL install behaves as a standard Linux install. However, H++ communicates only
within the WSL vmdisk — it cannot see the Windows filesystem or communicate with
Windows-native apps. Paths like `~/Documents` resolve to the Linux vmdisk home,
not `C:\Users\...`.

---

## The `!` Modifier — Force Override

`!` is a universal override modifier. It means: *"do it now, no checks, no questions asked."*
It bypasses guards, queues, caches, and conflict detection depending on the keyword it modifies.

`!` appears in two positions depending on the keyword:

- **On the verb** — for direct commands: `remember!`, `shout!`
- **On the argument** — when `ask` is the invoker: `ask (draw!)`

### Behavior by keyword

| Expression | Normal behavior | With `!` |
|---|---|---|
| `remember (x) as file.md` | Fails or prompts if file exists | Overwrites unconditionally |
| `shout runDrawCode` | Queues if conflicting code is running | Forces immediate execution |
| `ask (draw!)` | Normal repaint cycle | Forces full repaint, elevated |
| `ask (fetch!:~/file.md)` | May use cache | Bypasses cache, reads fresh |

### Examples

```novel
remember! (ask.LastAns) as tile.md     ← overwrite tile.md even if it exists
shout! runDrawCode                      ← run draw code even if draw is active
ask (draw!)                             ← force full repaint immediately
ask (fetch!:~/Documents/data.md)        ← bypass cache, force fresh read
```

### Rules

- `!` never introduces new behavior — it only removes safety checks on existing behavior
- Overuse of `!` is a code smell — if you need it everywhere, something is wrong with your logic
- `!` is not valid on `bye`, `declare`, `are`, or `who` — these have no override semantics

---

## Appendix — Reserved Keywords

`appN`, `declare`, `==`, `are`, `who`, `ask`, `Really`, `say`, `shout`, `remember`,
`fetch`, `open`, `func`, `bye`, `yes`, `no`, `Int`, `Str`, `Float`, `Bool`, `List`,
`Map`, `Func`, `shortterm`, `draw`, `drawNew`, `redraw`, `on`

---

## Appendix — Reserved Modifiers

`!` — force override modifier. Bypasses safety checks, queues, caches, and conflict detection.

---

*Novel v0.1 — Subject to change as the language evolves.*
