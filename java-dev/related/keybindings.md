---
aliases:
  - IntelliJ
  - IDEA
  - Keybindings
---
>IntelliJ IDEA Keybindings
## good to know bindings

| keybinding               | feature                           |     |
| ------------------------ | :-------------------------------- | :-: |
|                          | ==**Code**==                      |     |
| `ctrl`+`shift`+`f8`      | View breakpoints                  |  ⭐  |
| `ctrl`+`shift`+`↑/↓`     | Move Statement up/down            |  ⭐  |
| `ctrl`+`alt`+`h`         | Call hierarchy                    |  ⭐  |
| `ctrl`+`shift`+`i`       | Open quick definition lookup      |  ⭐  |
|                          | ==**Navigation**==                |     |
| `alt`+`f7`               | Find usages                       |  ⭐  |
| `alt`+`ctrl`+`f7`        | Show usages                       |  ⭐  |
| `ctrl`+`f7`              | Find usages in file               |  ⭐  |
| `ctrl`+`shift`+`f7`      | Highlight usages in file          |  ⭐  |
| `ctrl`+`b`               | Go to declaration                 |  ⭐  |
| `ctrl`+`alt`+`b`         | Go to implementation(s)           |  ⭐  |
| `ctrl`+`shift`+`b`       | Go to type declaration            |  ⭐  |
| `alt`+`↑/↓`              | Go to next/previous method        |  ⭐  |
| `f11`                    | Toggle bookmark                   |  ⭐  |
| `shift`+`f11`            | Show bookmarks                    |  ⭐  |
| `f2`                     | Next highlighted error            |  ⭐  |
| `f5`                     | Copy class                        |  ⭐  |
|                          | ==**Files**==                     |     |
| `ctrl`+`d`               | Compare Files                     |  ⭐  |
| `ctrl`+`shift`+`c`       | Copy paths                        |  ⭐  |
|                          | ==**Appearence**==                |     |
| **`ctrl`+`shift`+`f12`** | Toggle maximizing editor          |  ⭐  |
| `alt`+`shift`+`.`        | Increase Font Size in All Editors |  ⭐  |
| `alt`+`shift`+`,`        | Decrease Font Size in All Editors |  ⭐  |
## most of bindings
### Editing

| Linux, Windows           | Feature                                                                            |     |
| ------------------------ | ---------------------------------------------------------------------------------- | :-: |
| `alt`+`shift`+`,`        | Decrease Font Size in All Editors                                                  |  ⭐  |
| `alt`+`shift`+`.`        | Increase Font Size in All Editors                                                  |  ⭐  |
| `alt`+`shift`+`insert`   | Column Selection Mode                                                              |     |
| `alt`+`shift`+`↑/↓`      | Move Line Up/Down                                                                  |     |
| `alt`+`shift`+`j`        | Unselect Occurrence                                                                |     |
| `alt`+`j`                | Add selection for Next Occurrence                                                  |     |
| `ctrl`+`f4`              | Close active editor tab                                                            |     |
| `ctrl`+`shift`+`-/=`     | Collapse/Expand all                                                                |  ◾  |
| `ctrl`+`alt`+`-/=`       | Collapse/Expand code block recursively                                             |     |
| ``ctrl``+`-/=`           | Collapse/Expand code block                                                         |     |
| ``ctrl``+`.`             | Fold selection                                                                     |     |
| `ctrl`+`backspace`       | Delete to word start                                                               |     |
| `ctrl`+`delete`          | Delete to hump end                                                                 |     |
| `ctrl`+`shift`+`←/→`     | Select to word start/end                                                           |     |
| ``ctrl``+``shift``+`[/]` | Select till code block start                                                       |     |
| `ctrl`+`shift`+`]`       | Select till code block end                                                         |     |
| `ctrl`+`shift`+`u`       | Toggle case for word at caret or selected block                                    |     |
| `shift`+`enter`          | Start new line                                                                     |     |
| `ctrl`+`enter`           | Smart line split                                                                   |     |
| `ctrl`+`shift`+`j`       | Smart line join                                                                    |     |
| `ctrl`+`y`               | Delete line at caret                                                               |     |
| `ctrl`+`d`               | Duplicate Selection/Line                                                           |     |
| `ctrl`+`shift`+`v`       | Paste from recent buffers...                                                       |     |
| `ctrl`+`v`               | Paste from clipboard                                                               |     |
| `ctrl`+`c`               | Copy current line or selected block to clipboard                                   |     |
| `shift`+`delete`         | Cut current line or selected block to clipboard                                    |     |
| `ctrl`+`x`               | Cut current line or selected block to clipboard                                    |     |
| `shift`+`tab`            | Unindent selected lines                                                            |     |
| `tab`                    | Indent selected lines                                                              |     |
| `ctrl`+`alt`+`i`         | Auto-indent line(s)                                                                |     |
| `ctrl`+`alt`+`o`         | Optimize imports                                                                   |  ◾  |
| `ctrl`+`alt`+`l`         | Reformat code / selected code                                                      |  ◾  |
| `alt`+`enter`            | Show intention actions and quick-fixes                                             |     |
| `alt`+`q`                | Context info                                                                       |     |
| `ctrl`+`shift`+`w`       | Decrease current selection to previous state                                       |     |
| `ctrl`+`w`               | Select successively increasing code blocks                                         |     |
| `ctrl`+`shift`+`/`       | Comment/uncomment with block comment                                               |     |
| `ctrl`+`/`               | Comment/uncomment with line comment                                                |  ◾  |
| `ctrl`+`alt`+`t`         | Surround with... (if..else, try..catch, for, synchronized, etc.)                   |     |
| `ctrl`+`i`               | Implement methods                                                                  |     |
| `ctrl`+`o`               | Override methods                                                                   |     |
| `alt`+`insert`           | Generate code... (Getters, Setters, Constructors, hashCode/equals, toString)       |  ◾  |
| `ctrl`+`f1`              | Show descriptions of error or warning at caret                                     |     |
| `ctrl`+`🖱️`         | Brief Info                                                                         |     |
| `ctrl`+`f1`              | External Doc                                                                       |     |
| `ctrl`+`q`               | Quick documentation lookup                                                         |     |
| `ctrl`+`p`               | Parameter info (within method call arguments)                                      |     |
| `ctrl`+`shift`+`enter`   | Complete Current Statement                                                         |     |
| `tab`                    | Choose Lookup Item Replace                                                         |     |
| `enter`                  | Choose Lookup Item                                                                 |     |
| `ctrl`+`shift`+`space`   | Smart code completion (filters the list of methods and variables by expected type) |     |
| `ctrl`+`space`           | Basic code completion (the name of any class, method or variable)                  |  ◾  |

### Search/Replace

| Linux, Windows     | Feature                                      |     |
| ------------------ | -------------------------------------------- | --- |
| `shift` `shift`    | Search everywhere                            | ◾   |
| `ctrl`+`f`         | Find                                         |     |
| `f3`               | Find next                                    |     |
| `shift`+`f3`       | Find previous                                |     |
| `ctrl`+`r`         | Replace                                      |     |
| `ctrl`+`shift`+`f` | Find in path                                 | ◾   |
| `ctrl`+`shift`+`r` | Replace in path                              |     |
| `ctrl`+`shift`+`s` | Search structurally (Ultimate Edition only)  |     |
| `ctrl`+`shift`+`m` | Replace structurally (Ultimate Edition only) |     |

### Usage Search

| Linux, Windows      | Feature                  |     |
| ------------------- | ------------------------ | :-: |
| `alt`+`f7`          | Find usages              |  ⭐  |
| `alt`+`ctrl`+`f7`   | Show usages              |  ⭐  |
| `ctrl`+`f7`         | Find usages in file      |  ⭐  |
| `ctrl`+`shift`+`f7` | Highlight usages in file |  ⭐  |

### Compile and Run

| Linux, Windows       | Feature                                      |     |
| -------------------- | -------------------------------------------- | :-: |
| `ctrl`+`f9`          | Make project (compile modifed and dependent) |     |
| `ctrl`+`shift`+`f9`  | Compile selected file, package or module     |     |
| `alt`+`shift`+`f10`  | Select configuration and run                 |     |
| `alt`+`shift`+`f9`   | Select configuration and debug               |     |
| `ctrl` `ctrl`        | Run Anything                                 |     |
| `shift`+`f10`        | Run                                          |  ◾  |
| `shift`+`f9`         | Debug                                        |     |
| `ctrl`+`shift`+`f10` | Run context configuration from editor        |     |
| `ctrl`+`shift`+`f10` | Debug context configuration from editor      |     |

### Debugging

| Linux, Windows      | Feature                         |     |
| ------------------- | ------------------------------- | :-: |
| `ctrl`+`f2`         | Stop                            |     |
| `f8`                | Step over                       |     |
| `f7`                | Step into                       |     |
| `shift`+`f7`        | Smart step into                 |     |
| `shift`+`f8`        | Step out                        |     |
| `alt`+`f9`          | Run to cursor                   |     |
| `alt`+`f8`          | Evaluate expression (selection) |     |
| `f9`                | Resume program                  |  ◾  |
| `ctrl`+`f8`         | Toggle breakpoint               |     |
| `ctrl`+`shift`+`f8` | View breakpoints                |  ⭐  |

### Navigation

| Linux, Windows             | Feature                                                          |     |
| -------------------------- | ---------------------------------------------------------------- | :-: |
| `ctrl`+`n`                 | Go to class                                                      |     |
| `ctrl`+`shift`+`n`         | Go to file                                                       |     |
| `ctrl`+`alt`+`shift`+`n`   | Go to symbol                                                     |     |
| `alt`+`←/→`                | Go to previous/next editor tab                                   |     |
| `f12`                      | Go back to previous tool window                                  |     |
| `escape`                   | Go to editor (from tool window)                                  |     |
| `shift`+`escape`           | Hide Active Tool Window                                          |     |
| `ctrl`+`shift`+`f4`        | Close active run/messages/find/... tab                           |     |
| `ctrl`+`shift`+`'`         | Maximize Tool Window (Problems, Output, Debug Console, Terminal) |     |
| `ctrl`+`g`                 | Go to lines                                                      |  ◾  |
| `ctrl`+`e`                 | Recent files popup                                               |  ◾  |
| `ctrl`+`alt`+`←/→`         | Navigate back/forward                                            |     |
| `ctrl`+`shift`+`backspace` | Navigate to last edit location                                   |     |
| `alt`+`f1`                 | Select current file or symbol in any view                        |     |
| `ctrl`+`b`                 | Go to declaration                                                |  ⭐  |
| `ctrl`+`alt`+`b`           | Go to implementation(s)                                          |  ⭐  |
| `ctrl`+`u`                 | Go to super implementation(s)                                    |     |
| `ctrl`+`shift`+`i`         | Open quick definition lookup                                     |  ⭐  |
| `ctrl`+`shift`+`b`         | Go to type declaration                                           |  ⭐  |
| `ctrl`+`u`                 | Go to super-method/super-class                                   |     |
| `alt`+`↑/↓`                | Go to next/previous method                                       |  ⭐  |
| `ctrl`+`[/]`               | Move to code block start/end                                     |     |
| `ctrl`+`f12`               | File structure popup                                             |  ◾  |
| `ctrl`+`h`                 | Type hierarchy                                                   |     |
| `ctrl`+`shift`+`h`         | Method hierarchy                                                 |     |
| `ctrl`+`alt`+`h`           | Call hierarchy                                                   |  ⭐  |
| `f2`                       | Next highlighted error                                           |  ⭐  |
| `shift`+`f2`               | Previous highlighted error                                       |     |
| `f4`                       | Edit source                                                      |     |
| `ctrl`+`enter`             | View source                                                      |     |
| `ctrl`+`shift`+`←/→`       | Move Statement left/right                                        |  ⭐  |
| `alt`+`home`               | Show navigation bar                                              |     |
| `f11`                      | Toggle bookmark                                                  |  ⭐  |
| `ctrl`+`f11`               | Toggle bookmark with mnemonic                                    |     |
| `ctrl`+`0`                 | Go to numbered bookmark                                          |     |
| `shift`+`f11`              | Show bookmarks                                                   |  ⭐  |
| `ctrl`+`alt`+`shift`+`↑/↓` | Next/Previous Change                                             |     |
| `ctrl`+`home`              | Move Caret to Text Start                                         |     |
| `ctrl`+`end`               | Move Caret to Text End                                           |     |
| `ctrl`+`shift`+`m`         | Move Caret to Matching Brace                                     |     |
| `ctrl`+`shift`+`t`         | Go to Test                                                       |     |

### Refactoring

| Linux, Windows           | Feature             |     |
| ------------------------ | ------------------- | :-: |
| `f5`                     | Copy                |     |
| `ctrl`+`alt`+`shift`+`t` | Refactor This...    |     |
| `f6`                     | Move                |  ◾  |
| `alt`+`delete`           | Safe delete         |     |
| `shift`+`f6`             | Rename              |  ◾  |
| `ctrl`+`f6`              | Change Signature    |     |
| `ctrl`+`alt`+`n`         | Inline              |     |
| `ctrl`+`alt`+`m`         | Extract Method      |     |
| `ctrl`+`alt`+`v`         | Extract Variable    |     |
| `ctrl`+`alt`+`f`         | Extract Field       |     |
| `ctrl`+`alt`+`c`         | Extract Constant    |     |
| `ctrl`+`alt`+`p`         | Introduce Parameter |     |

### VCS/Local History

| Linux, Windows     | Feature                 |     |
| ------------------ | ----------------------- | :-: |
| `ctrl`+`alt`+`k`   | Commit project to VCS   |     |
| `ctrl`+`shift`+`k` | Push commits to VCS     |     |
| `ctrl`+`t`         | Update project from VCS |     |
| `ctrl`+`alt`+`z`   | Rollback Lines          |     |
| `f4`               | Jump to Source          |  ◾  |
| `alt`+`shift`+`c`  | View recent changes     |     |

### General

| Linux, Windows           | Feature                                   |     |
| ------------------------ | ----------------------------------------- | :-: |
| `alt`+`0..9`             | Open corresponding tool                   |     |
| `ctrl`+`s`               | Save all                                  |     |
| `ctrl`+`alt`+`y`         | Synchronize                               |     |
| `ctrl`+`shift`+`f12`     | Toggle maximizing editor                  |  ⭐  |
| `alt`+`shift`+`f`        | Add to Favorites                          |     |
| `alt`+`shift`+`i`        | Inspect current file with current profile |     |
| `ctrl`+ `´`              | Quick switch current scheme               |  ◾  |
| `ctrl`+`alt`+`s`         | Open Settings dialog                      |  ◾  |
| `ctrl`+`alt`+`shift`+`s` | Open Project Structure dialog             |  ◾  |
| `ctrl`+`shift`+`a`       | Find Action                               |     |
| `ctrl`+`tab`             | Switch between tabs and tool window       |     |
| `shift`+`f12`            | Restore Default layout                    |     |

### Custom

| Linux, Windows           | Feature                                                 |     |
| ------------------------ | ------------------------------------------------------- | :-: |
| `ctrl`+`d`               | Compare Files                                           |  ⭐  |
| `ctrl`+`shift`+`tab`     | Select Opposite Diff Pane                               |     |
| `f7`                     | Next difference                                         |  ◾  |
| `shift`+`f7`             | Previous difference                                     |     |
| `alt`+`ctrl`+`enter`     | Start new line before current                           |     |
| `ctrl`+`shift`+`enter`   | Start new line                                          |     |
| `alt`+`f12`              | Toggle and focuses corresponding tool window (Terminal) |  ◾  |
| `ctrl`+`alt`+`shift`+`j` | Sublime Text style multiple selections                  |     |
| `alt`+`←/→`              | Select previous/next tab (Terminal)                     |     |
| `alt`+`tab`              | Goto next splitter                                      |     |
| `alt`+`shift`+`tab`      | Goto previous splitter                                  |     |
| `enter`                  | Open Highlighted File (Explorer)                        |     |
| `f4`                     | Open Highlighted File (Explorer)                        |     |
| `alt`+`home`             | Jump to Navigation Bar                                  |     |
| `ctrl`+`shift`+`c`       | Copy paths                                              |  ⭐  |

