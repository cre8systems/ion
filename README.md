# ION
stands for *Illusionary Object Notation*. It is a compact format invented to transport big amounts of data in some cases and then very explicit key/value information, so that it can be easily read by humans.
While developing the first files, we are using the excellent Sublime Edit and the table extension, which makes the compact tabular data human readible with a push of the TAB button.

ION itself is not concerned with the semantics.

## Objectives
- be as concise (short) as possible
- allow dictionary structured data with key and value
- allow big amounts of data in tabular format
- allow writing documentation within an ION file, using special comments

## Example
```
[META]
#! Information applicable to the all following sections!
source="EG"  # a string
timestamp=2015-06-04T06:47 # a date object, when the file was generated
    time_taken_ms=1234 # a numeric (u64)
    
# indentation before a key will be ignored

[ACCOMODATION.MD]
#! Accommodation master data
| name  | city  | stars | # all lines must start with a |
|-------|-------|-------| # optional
| Ibis  | Dubai | 2.0   |
|Ibis Mall of the Emirates|Dubai|2.0| # leading and trailing whitespace will be ignored
|Kempinkski|" D U B A I "|5.0| # unless you put quotes right after and before |
```

## General Rules

- UTF-8 encoding, always. No exceptions. All data *must* be UTF-8 encoded. Everything else should panic the parser.
- newlines `\r` `\r\n` and `\n` are equivalent.
- empty lines or lines with whitespace are ignored
- lines beginning with `# ` are comments and contain no data. Comments may also be placed at the end of each line
- lines with `@key: value` contain optional meta data for the currently processed section **[TBD]***
- lines beginning with `#!` are section comments and are meant to automatically generate documentation. Those will be placed in the documentation 
- lines beginning with `##` are inline comments

### Strings

The following characters must be escaped
```
\b         - backspace       (U+0008)
\t         - tab             (U+0009)
\n         - linefeed        (U+000A)
\f         - form feed       (U+000C)
\r         - carriage return (U+000D)
\"         - quote           (U+0022)
\\         - backslash       (U+005C)
\uXXXX     - unicode         (U+XXXX)
\UXXXXXXXX - unicode         (U+XXXXXXXX)
```

### Datetime
Datetimes are RFC 3339 dates.

```
datetime = 1979-05-27T07:32:00Z
```

## Sections

A section is started by a section name in square brackets `[NAME]`. 

There are two types of sections - dictionary and tabular.

### Dictionary

This is borrowed from TOML. A dictionary sections *may not* contain a line beginning with `|`. This would indicate a tabular section. 
Parsers must report this as an error.

Whitespace before and after `=` are optional

Example:

```toml
[MY_DICTIONARY]
name="Some name"
city="Dubai"
country_code = "AE"
```
### Tabular

This holds table like data, must have each line beginning with `|`.
The first line always holds the header with the field names!

An optional line with a separator row may be included optionally. In this case the pipe characted is immediately followed by a minus sign, e.g. `|--|--|`

Leading and trailing whitespace is optional and must be removed by the parser. In order to retain whitespace, the value must be enclosed with double quotes`"`

Example: 

```
[MY_TABLE]
| name | city  | country_code | description |
|------|------ |--------------|-------------| # this will be ignored
| Some | Dubai | AE           | none        |          
| Other| Dubai | PL           |" wh ite sp "| # retain whitespace in the description
```

It is recommended that the parser allows to fetch values from a table by name but especially for performance reasons via a numeric index, or as an iterator.
Consider zero-copy 

Rust example:

```rust
let line = "|foo|bar|";
let row : TableRow = line.parse(); // no context, so no headers
let foo : &'str = row[0];          // returns a ref

let raw = "[FOOBAR]\n|name|value|\n|foo|bar|\n";
let section : Section = raw.parse();

match section {
    Table(table)     => { println!("{}={}", table[0].get("name").unwrap(), table[0][0]) },
    Dictionary(dict) => { println!("name={}", dict.get("name")) }
}
```
