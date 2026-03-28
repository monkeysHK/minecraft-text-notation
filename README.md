# Minecraft Text Notation

## Introduction

The Minecraft Text Notation is an XML-like notation that represents displayed texts in Minecraft. It aims to be extensible while being easy for humans to read and write.

This notation format is inspired by the [MiniMessage format](https://docs.papermc.io/adventure/minimessage/format/) by [PaperMC](https://papermc.io/software/paper/).

> **Short Example**
> 
> ```
> <yellow><b><u>Welcome to Minecraft Text Notation</u></b></yellow>
> 
> <hover:show_text:"<gold>The spec is here!</gold>">[What's new?]</hover>
> ```

## General Features

The Minecraft Text Notation defines two modes: strict and non-strict. In strict mode, all validation rules apply. Violation of any rules should result in a parsing failure. In non-strict mode, the rule-violating sections should be coerced into plain texts.

A Minecraft Text is a sequence of of two types of components: plain texts and tags.

```
<minecraft-text> ::= <text-components>

<text-components> ::= <text-component>
                    | <text-component> <text-components>

<text-component> ::= <tag>
                   | <plain-text>
```

### Tags

There are two types of tags: void and normal. A void tag is a tag that does not have any child components. A void tag only has a start tag. A normal tag is a tag that can have child components. A normal tag has a start tag and an end tag.

```
<tag> ::= "<" <void-tag-content> ">"
        | "<" <normal-tag-content> ">"
        | "</" <normal-tag-content> ">"
```

A tag consists of a tag name followed by zero or more tag arguments. The tag name and each of the tag arguments are separated by colons. A tag argument can be a double-quoted string, single-quoted string, or an unquoted string.

> **Example**
> 
> The following tags are equivallent:
> ```
> <hover:show_text:"Hello">
> <hover:"show_text":'Hello'>
> <hover:show_text:Hello>
> ```


Each recognized tag has a specific syntax that the tag name and arguments should follow. A valid tag must follow the syntax of a recognized tag. Additionally, an valid end tag must have a matching tag of any<sup>(non-strict mode)</sup>/the closest<sup>(strict mode)</sup> previous unclosed start tag.

An end tag is a matching tag of a start tag if the tag name and all specified tag arguments in the end tag match those of the start tag by string content. Additional tag arguments can be present on the start tag but unspecified on the matching end tag.

> **Example**
> 
> ```
> <hover:show_text:"Hello">These two tags are matching</hover:show_text:"Hello">
> <hover:show_text:'Hello'>These two tags are matching too</hover:"show_text">
> <hover:show_text:Hello>These two tags are matching as well</hover>
> The unrecognized <unknown> tag is invalid.
> Without a matching start tag, the </yellow> tag is invalid.
> ```

### Plain Texts and Tag Argument Strings

Tag names and unquoted strings should contain only alphanumerical characters and underscores, except that tag names for hex colors should start with `#`.

All Unicode characters may be placed within plain texts, double-quoted strings, and single-quoted strings.

```
<plain-text> ::= (any Unicode character except the unescaped "<")

<quoted-or-unquoted-string> ::= '"' <double-quoted-string-content> '"'
                              | "'" <single-quoted-string-content> "'"
                              | <unquoted-string-content>

<unquoted-string-content> ::= (sequence of alphanumerical character or underscore)

<single-quoted-string-content> ::= (sequence of Unicode character except the unescaped "'")

<double-quoted-string-content> ::= (sequence of Unicode character except the unescaped '"')
```

#### White Spaces

White spaces should not be stripped from tag names, plain texts, double-quoted strings, or single-quoted strings.

The newline character may be used within plain texts, double-quoted strings, and single-quoted strings.

The newline character and the `<br>` tag should cause a line break in plain texts.

> **Example**
> 
> ```
> <hover:show_text:"A new line
> can be used here">A new line can also
> be used here. <br> also works.</hover>
>    \<-- These spaces are not stripped.
> ```

#### Escapes

In plain text, double-quoted strings, and single-quoted strings, `\\` should be a valid escape for the literal `\`.

In plain text, `\<` should be a valid escape for the literal `<`.

In double-quoted strings, `\"` should be a valid escape for the literal `"`.

In single-quoted strings, `\'` should be a valid escape for the literal `'`.

All uses of the backslash unspecified in this section should create invalid escapes.

> **Example**
> 
> ```
> <hover:show_text:"Escaping the double quote: \""></hover>
> <hover:show_text:'Escaping the single quote: \''></hover>
> Escaping the left angle bracket in plain text: \<
> Escaping the backslash: \\
> ```

### Strict Mode

In strict mode, all validation rules apply:

- All tags must be valid.
- An end tag must be a matching tag of the closest previous unclosed start tag. All normal tags must be closed.
- All escapes must be valid.

Violation of any rules should result in a parsing failure.

### Non-strict Mode

In non-strict mode, the rule-violating sections should be coerced into plain texts:

- An invalid tag should be rendered as text literals.
- An invalid escape should be rendered as text literals.
- An end tag should close the closest previous matching unclosed start tag and all unclosed start tags between itself and the found tag, if it can find one.
- A reset tag should close all unclosed start tags between itself and the beginning of the sequence.

> **Example**
> 
> The following notations are equivallent in non-strict mode.
> ```
> <bold><italic><underlined>Hello,</italic> world!
> ```
> is equivallent to
> ```
> <bold><italic><underlined>Hello,</underlined></italic> world!</bold>
> ```

> **Example**
> 
> The following notations are equivallent in non-strict mode.
> ```
> <yellow>Hello, <bold>world<reset><italic>!
> ```
> is equivallent to
> ```
> <yellow>Hello, <bold>world</bold></yellow><italic>!</italic>
> ```

## Recognized Tags

A tag must be a recognized void tag or normal tag.

```
<void-tag-content> ::= "br"
                     | "reset"  ; for non-strict mode only

<normal-tag-content> ::= <named-color-tag-content>
                       | <hex-color-tag-content>
                       | <color-tag-content>
                       | <decoration-tag-content>
                       | <hover-tag-content>
```

### Named Color Tags

A named color tag must have a tag name that is a color name in Minecraft Java Edition and no tag argument.

```
<named-color-tag-content> ::= <named-color>

<named-color> ::= "black" | "dark_blue" | "dark_green" | "dark_aqua" | "dark_red" | " dark_purple" | "gold" | "gray" | "dark_gray" | "blue" | "green" | "aqua" | "red" | "light_purple" | "yellow" | "white"
```

> **Example**
> 
> ```
> Player: <yellow>Hello, <blue>World!
> ```

### Hex Color Tags

A hex color tag must have a tag name that is a hex color with the format `#RRGGBB` and no tag argument.

```
<hex-color-tag-content> ::= <hex-color>

<hex-color> ::= "#" <hex-digit> <hex-digit> <hex-digit> <hex-digit> <hex-digit> <hex-digit>

<hex-digit> ::= (any hexadecimal symbol: 0-9, a-f, A-F)
```

> **Example**
> 
> ```
> World: <#abcdef>Hello, Player!
> ```

### Color Tags

A color tag must have the one tag argument that is one of the following:

- A color name in Minecraft Java Edition
- A hex color with the format `#RRGGBB`

```
; remark: each argument (<named-color>, <hex-color>) can be a <quoted-or-unquoted-string> with that content
<color-tag-content> ::= "color"  ; for end tag only
                      | "color" ":" <named-color>
                      | "color" ":" <hex-color>
```

> **Example**
> 
> ```
> Player: <color:yellow>How is it going, <color:#abcdef>World?
> ```

### Decoration Tags

A decoration tag must have a tag name that is one of the following:

- `bold`(alias: `b`)
- `italic` (alias: `i`)
- `underlined` (alias: `u`)
- `strikethrough` (alias: `st`)
- `obfuscated` (alias: `obf`)

A decoration tag must have no tag argument.

```
<decoration-tag-content> ::= "bold" | "b" | "italic" | "i" | "underlined" | "u" | "strikethrough" | "st" | "obfuscated" | "obf"
```

> **Example**
> 
> ```
> World: <b><i>Pretty good.
> ```

### Hover Tags

A hover tag must have a first argument that is one of the following actions:

- `show_text`

```
<hover-tag-content> ::= "hover"  ; for end tag only
                      | "hover" ":" <show-text-argument>
```

#### Show Text Tags

A show text tag is a hover tag with the `show_text` action. The tag must have a second argument that is a string representing a valid Minecraft Text.

```
; remark: each argument ("show_text", <minecraft-text>) can be a <quoted-or-unquoted-string> with that content
<show-text-argument> ::= "show_text"  ; for end tag only
                       | "show_text" ":" <minecraft-text>
```

> **Example**
> 
> ```
> <hover:show_text:"Player: <yellow>Hello, <blue>World!<br>World: <#abcdef>Hello, Player!">[show conversation]</hover>
> ```

## Grammar

```
<minecraft-text> ::= <text-components>

<text-components> ::= <text-component>
                    | <text-component> <text-components>

<text-component> ::= <tag>
                   | <plain-text>

<tag> ::= "<" <void-tag-content> ">"
        | "<" <normal-tag-content> ">"
        | "</" <normal-tag-content> ">"

<plain-text> ::= (any Unicode character except the unescaped "<")

<quoted-or-unquoted-string> ::= '"' <double-quoted-string-content> '"'
                              | "'" <single-quoted-string-content> "'"
                              | <unquoted-string-content>

<unquoted-string-content> ::= (sequence of alphanumerical character or underscore)

<single-quoted-string-content> ::= (sequence of Unicode character except the unescaped "'")

<double-quoted-string-content> ::= (sequence of Unicode character except the unescaped '"')

<void-tag-content> ::= "br"
                     | "reset"  ; for non-strict mode only

<normal-tag-content> ::= <named-color-tag-content>
                       | <hex-color-tag-content>
                       | <color-tag-content>
                       | <decoration-tag-content>
                       | <hover-tag-content>

<named-color-tag-content> ::= <named-color>

<named-color> ::= "black" | "dark_blue" | "dark_green" | "dark_aqua" | "dark_red" | " dark_purple" | "gold" | "gray" | "dark_gray" | "blue" | "green" | "aqua" | "red" | "light_purple" | "yellow" | "white"

<hex-color-tag-content> ::= <hex-color>

<hex-color> ::= "#" <hex-digit> <hex-digit> <hex-digit> <hex-digit> <hex-digit> <hex-digit>

<hex-digit> ::= (any hexadecimal symbol: 0-9, a-f, A-F)

; remark: each argument (<named-color>, <hex-color>) can be a <quoted-or-unquoted-string> with that content
<color-tag-content> ::= "color"  ; for end tag only
                      | "color" ":" <named-color>
                      | "color" ":" <hex-color>

<decoration-tag-content> ::= "bold" | "b" | "italic" | "i" | "underlined" | "u" | "strikethrough" | "st" | "obfuscated" | "obf"

<hover-tag-content> ::= "hover"  ; for end tag only
                      | "hover" ":" <show-text-argument>

; remark: each argument ("show_text", <minecraft-text>) can be a <quoted-or-unquoted-string> with that content
<show-text-argument> ::= "show_text"  ; for end tag only
                       | "show_text" ":" <minecraft-text>
```

