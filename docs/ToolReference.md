_Visit the [main page](../README.md)_

# Tool reference

This page provides detailed documentation about the different tools as well as examples. Material for the individual tools is also available via the `--help` option.

* [Common options and behavior](#common-options-and-behavior)
* [csv2tsv](#csv2tsv-reference)
* [keep-header](#keep-header-reference)
* [number-lines](#number-lines-reference)
* [tsv-append](#tsv-append-reference)
* [tsv-filter](#tsv-filter-reference)
* [tsv-join](#tsv-join-reference)
* [tsv-pretty](#tsv-pretty-reference)
* [tsv-sample](#tsv-sample-reference)
* [tsv-select](#tsv-select-reference)
* [tsv-split](#tsv-split-reference)
* [tsv-summarize](#tsv-summarize-reference)
* [tsv-uniq](#tsv-uniq-reference)

___

## Common options and behavior

Information in this section applies to all the tools.

### Specifying options

Multi-letter options are specified with a double dash. Single letter options can be specified with a single dash or double dash. For example:
```
$ tsv-select -f 1,2         # Valid
$ tsv-select --f 1,2        # Valid
$ tsv-select --fields 1,2   # Valid
$ tsv-select -fields 1,2    # Invalid.
```

### Help (`-h`, `--help`, `--help-verbose`)

All tools print help if given the `-h` or `--help` option. Many provide more detail via the `--help-verbose` option.

### Field numbers and field-lists.

Field numbers are one-upped integers, following Unix conventions. Some tools use zero to represent the entire line (`tsv-join`, `tsv-uniq`).

In many cases multiple fields can be entered as a "field-list". A field-list is a comma separated list of field numbers or field ranges. For example:

```
$ tsv-select -f 3          # Field 3
$ tsv-select -f 3,5        # Fields 3 and 5
$ tsv-select -f 3-5        # Fields 3, 4, 5
$ tsv-select -f 1,3-5      # Fields 1, 3, 4, 5
```

Most tools process or output fields in the order listed, and repeated use is usually fine:
```
$ tsv-select -f 5-1       # Fields 5, 4, 3, 2, 1
$ tsv-select -f 1-3,2,1   # Fields 1, 2, 3, 2, 1
```

### UTF-8 input

These tools assume data is utf-8 encoded.

### Line endings

These tools have been tested on Unix platforms, including macOS, but not Windows. On Unix platforms, Unix line endings (`\n`) are expected, with the notable exception of `tsv2csv`. Not all the tools are affected by DOS and Windows line endings (`\r\n`), those that are check the first line and flag an error. `csv2tsv` explicitly handles DOS and Windows line endings, converting to Unix line endings as part of the conversion.

The `dos2unix` tool can be used to convert Windows line endings to Unix format. See [Convert newline format and character encoding with dos2unix and iconv](TipsAndTricks.md#convert-newline-format-and-character-encoding-with-dos2unix-and-iconv)

The tools were written to respect platform line endings. If built on Windows, then Windows line endings. However, given the lack of testing, a Windows build should be expected to have some issues with line endings.

### File format and alternate delimiters (`--delimiter`)

Any character can be used as a delimiter, TAB is the default. However, there is no escaping for including the delimiter character or newlines within a field. This differs from CSV file format which provides an escaping mechanism. In practice the lack of an escaping mechanism is not a meaningful limitation for data oriented files.

Aside from a header line, all lines are expected to have data. There is no comment mechanism and no special handling for blank lines. Tools taking field indices as arguments expect the specified fields to be available on every line.

### Headers (`-H`, `--header`)

Most tools handle the first line of files as a header when given the `-H` or `--header` option. For example, `tsv-filter` passes the header through without filtering it. When `--header` is used, all files and stdin are assumed to have header lines. Only one header line is written to stdout. If multiple files are being processed, header lines from subsequent files are discarded.

### Multiple files and standard input

Tools can read from any number of files and from standard input. As per typical Unix behavior, a single dash represents standard input when included in a list of files. Terminate non-file arguments with a double dash (`--`) when using a single dash in this fashion. Example:
```
$ head -n 1000 file-c.tsv | tsv-filter --eq 2:1000 -- file-a.tsv file-b.tsv - > out.tsv
```

The above passes `file-a.tsv`, `file-b.tsv`, and the first 1000 lines of `file-c.tsv` to `tsv-filter` and write the results to `out.tsv`.

---

## csv2tsv reference

**Synopsis:** csv2tsv [options] [file...]

csv2tsv converts CSV (comma-separated) text to TSV (tab-separated) format. Records are read from files or standard input, converted records are written to standard output.

Both formats represent tabular data, each record on its own line, fields separated by a delimiter character. The key difference is that CSV uses escape sequences to represent newlines and field separators in the data, whereas TSV disallows these characters in the data. The most common field delimiters are comma for CSV and tab for TSV, but any character can be used. See [Comparing TSV and CSV formats](comparing-tsv-and-csv.md) for addition discussion of the formats.

Conversion to TSV is done by removing CSV escape syntax, changing field delimiters, and replacing newlines and tabs in the data. By default, newlines and tabs in the data are replaced by spaces. Most details are customizable.

There is no single spec for CSV, any number of variants can be found. The escape syntax is common enough: fields containing newlines or field delimiters are placed in double quotes. Inside a quoted field, a double quote is represented by a pair of double quotes. As with field separators, the quoting character is customizable.

Behaviors of this program that often vary between CSV implementations:
* Newlines are supported in quoted fields.
* Double quotes are permitted in a non-quoted field. However, a field starting with a quote must follow quoting rules.
* Each record can have a different numbers of fields.
* The three common forms of newlines are supported: CR, CRLF, LF.
* A newline will be added if the file does not end with one.
* No whitespace trimming is done.

This program does not validate CSV correctness, but will terminate with an error upon reaching an inconsistent state. Improperly terminated quoted fields are the primary cause.

UTF-8 input is assumed. Convert other encodings prior to invoking this tool.

**Options:**
* `--h|help` - Print help.
* `--help-verbose` - Print detailed help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header. Only the header of the first file is output.
* `--q|quote CHR` - Quoting character in CSV data. Default: double-quote (")
* `--c|csv-delim CHR` - Field delimiter in CSV data. Default: comma (,).
* `--t|tsv-delim CHR` - Field delimiter in TSV data. Default: TAB
* `--r|replacement STR` - Replacement for newline and TSV field delimiters found in CSV input. Default: Space.

---

## keep-header reference

**Synopsis:** keep-header [file...] \-- program [args]

Execute a command against one or more files in a header-aware fashion. The first line of each file is assumed to be a header. The first header is output unchanged. Remaining lines are sent to the given command via standard input, excluding the header lines of subsequent files. Output from the command is appended to the initial header line. A double dash (\--) delimits the command, similar to how the pipe operator (\|) delimits commands.

The following commands sort files in the usual way, except for retaining a single header line:
```
$ keep-header file1.txt -- sort
$ keep-header file1.txt file2.txt -- sort -k1,1nr
```

Data can also be read from from standard input. For example:
```
$ cat file1.txt | keep-header -- sort
$ keep-header file1.txt -- sort -r | keep-header -- grep red
```

The last example can be simplified using a shell command:
```
$ keep-header file1.txt -- /bin/sh -c '(sort -r | grep red)'
```

`keep-header` is especially useful for commands like `sort` and `shuf` that reorder input lines. It is also useful with filtering commands like `grep`, many `awk` uses, and even `tail`, where the header should be retained without filtering or evaluation.

`keep-header` works on any file where the first line is delimited by a newline character. This includes all TSV files and the majority of CSV files. It won't work on CSV files having embedded newlines in the header.

**Options:**
* `--h|help` - Print help.
* `--V|version` - Print version information and exit.

---

## number-lines reference

**Synopsis:** number-lines [options] [file...]

number-lines reads from files or standard input and writes each line to standard output preceded by a line number. It is a simplified version of the Unix `nl` program. It supports one feature `nl` does not: the ability to treat the first line of files as a header. This is useful when working with tab-separated-value files. If header processing used, a header line is written for the first file, and the header lines are dropped from any subsequent files.

**Options:**
* `--h|help` - Print help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header. The first input file's header is output, subsequent file headers are discarded.
* `--s|header-string STR` - String to use as the header for the line number field. Implies `--header`. Default: 'line'.
* `--n|start-number NUM` - Number to use for the first line. Default: 1.
* `--d|delimiter CHR` - Character appended to line number, preceding the rest of the line. Default: TAB (Single byte UTF-8 characters only.)

**Examples:**
```
$ # Number lines in a file
$ number-lines file.tsv

$ # Number lines from multiple files. Treat the first line of each file
$ # as a header.
$ number-lines --header data*.tsv
```

**See Also:**

* [tsv-uniq](#tsv-uniq-reference) supports numbering lines grouped by key.

---

## tsv-append reference

**Synopsis:** tsv-append [options] [file...]

tsv-append concatenates multiple TSV files, similar to the Unix `cat` utility. Unlike `cat`, it is header-aware (`--H|header`), writing the header from only the first file. It also supports source tracking, adding a column indicating the original file to each row. Results are written to standard output.

Concatenation with header support is useful when preparing data for traditional Unix utilities like `sort` and `sed` or applications that read a single file.

Source tracking is useful when creating long/narrow form tabular data, a format used by many statistics and data mining packages. In this scenario, files have been used to capture related data sets, the difference between data sets being a condition represented by the file. For example, results from different variants of an experiment might each be recorded in their own files. Retaining the source file as an output column preserves the condition represented by the file.

The file-name (without extension) is used as the source value. This can customized using the `--f|file` option.

Example: Header processing:
```
$ tsv-append -H file1.tsv file2.tsv file3.tsv
```

Example: Header processing and source tracking:
```
$ tsv-append -H -t file1.tsv file2.tsv file3.tsv
```

Example: Source tracking with custom source values:
```
$ tsv-append -H -s test_id -f test1=file1.tsv -f test2=file2.tsv
 ```

**Options:**
* `--h|help` - Print help.
* `--help-verbose` - Print detailed help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header.
* `--t|track-source` - Track the source file. Adds an column with the source name.
* `--s|source-header STR` - Use STR as the header for the source column. Implies `--H|header` and `--t|track-source`. Default: 'file'
* `--f|file STR=FILE` - Read file FILE, using STR as the 'source' value. Implies `--t|track-source`.
* `--d|delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)

---

## tsv-filter reference

_Note: See the [tsv-filter](../README.md#tsv-filter) description in the project [README](../README.md) for a tutorial style introduction._

**Synopsis:** tsv-filter [options] [file...]

Filter lines of tab-delimited files via comparison tests against fields. Multiple tests can be specified, by default they are evaluated as AND clause. Lines satisfying the tests are written to standard output.

**General options:**
* `--help` - Print help.
* `--help-verbose` - Print detailed help.
* `--help-options` - Print the options list by itself.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header.
* `--d|delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)
* `--or` - Evaluate tests as an OR rather than an AND. This applies globally.
* `--v|invert` - Invert the filter, printing lines that do not match. This applies globally.

**Tests:**

Empty and blank field tests:
* `--empty <field-list>` - True if field is empty (no characters)
* `--not-empty <field-list>` - True if field is not empty.
* `--blank <field-list>` - True if field is empty or all whitespace.
* `--not-blank <field-list>` - True if field contains a non-whitespace character.

Numeric type tests:
* `--is-numeric <field-list>` - True if the field can be interpreted as a number.
* `--is-finite <field-list>` - True if the field can be interpreted as a number, and it is not NaN or infinity.
* `--is-nan <field-list>` - True if the field is NaN (including: "nan", "NaN", "NAN").
* `--is-infinity <field-list>` - True if the field is infinity (including: "inf", "INF", "-inf", "-INF")

Numeric comparisons:
* `--le <field-list>:NUM` - FIELD <= NUM (numeric).
* `--lt <field-list>:NUM` - FIELD <  NUM (numeric).
* `--ge <field-list>:NUM` - FIELD >= NUM (numeric).
* `--gt <field-list>:NUM` - FIELD >  NUM (numeric).
* `--eq <field-list>:NUM` - FIELD == NUM (numeric).
* `--ne <field-list>:NUM` - FIELD != NUM (numeric).

String comparisons:
* `--str-le <field-list>:STR` - FIELD <= STR (string).
* `--str-lt <field-list>:STR` - FIELD <  STR (string).
* `--str-ge <field-list>:STR` - FIELD >= STR (string).
* `--str-gt <field-list>:STR` - FIELD >  STR (string).
* `--str-eq <field-list>:STR` - FIELD == STR (string).
* `--istr-eq <field-list>:STR` - FIELD == STR (string, case-insensitive).
* `--str-ne <field-list>:STR` - FIELD != STR (string).
* `--istr-ne <field-list>:STR` - FIELD != STR (string, case-insensitive).
* `--str-in-fld <field-list>:STR` - FIELD contains STR (substring search).
* `--istr-in-fld <field-list>:STR` - FIELD contains STR (substring search, case-insensitive).
* `--str-not-in-fld <field-list>:STR` - FIELD does not contain STR (substring search).
* `--istr-not-in-fld <field-list>:STR` - FIELD does not contain STR (substring search, case-insensitive).

Regular expression tests:
* `--regex <field-list>:REGEX` - FIELD matches regular expression.
* `--iregex <field-list>:REGEX` - FIELD matches regular expression, case-insensitive.
* `--not-regex <field-list>:REGEX` - FIELD does not match regular expression.
* `--not-iregex <field-list>:REGEX` - FIELD does not match regular expression, case-insensitive.

Field length tests
* `--char-len-le <field-list>:NUM` - FIELD character length <= NUM.
* `--char-len-lt <field-list>:NUM` - FIELD character length < NUM.
* `--char-len-ge <field-list>:NUM` - FIELD character length >= NUM.
* `--char-len-gt <field-list>:NUM` - FIELD character length > NUM.
* `--char-len-eq <field-list>:NUM` - FIELD character length == NUM.
* `--char-len-ne <field-list>:NUM` - FIELD character length != NUM.
* `--byte-len-le <field-list>:NUM` - FIELD byte length <= NUM.
* `--byte-len-lt <field-list>:NUM` - FIELD byte length < NUM.
* `--byte-len-ge <field-list>:NUM` - FIELD byte length >= NUM.
* `--byte-len-gt <field-list>:NUM` - FIELD byte length > NUM.
* `--byte-len-eq <field-list>:NUM` - FIELD byte length == NUM.
* `--byte-len-ne <field-list>:NUM` - FIELD byte length != NUM.

Field to field comparisons:
* `--ff-le FIELD1:FIELD2` - FIELD1 <= FIELD2 (numeric).
* `--ff-lt FIELD1:FIELD2` - FIELD1 <  FIELD2 (numeric).
* `--ff-ge FIELD1:FIELD2` - FIELD1 >= FIELD2 (numeric).
* `--ff-gt FIELD1:FIELD2` - FIELD1 >  FIELD2 (numeric).
* `--ff-eq FIELD1:FIELD2` - FIELD1 == FIELD2 (numeric).
* `--ff-ne FIELD1:FIELD2` - FIELD1 != FIELD2 (numeric).
* `--ff-str-eq FIELD1:FIELD2` - FIELD1 == FIELD2 (string).
* `--ff-istr-eq FIELD1:FIELD2` - FIELD1 == FIELD2 (string, case-insensitive).
* `--ff-str-ne FIELD1:FIELD2` - FIELD1 != FIELD2 (string).
* `--ff-istr-ne FIELD1:FIELD2` - FIELD1 != FIELD2 (string, case-insensitive).
* `--ff-absdiff-le FIELD1:FIELD2:NUM` - abs(FIELD1 - FIELD2) <= NUM
* `--ff-absdiff-gt FIELD1:FIELD2:NUM` - abs(FIELD1 - FIELD2)  > NUM
* `--ff-reldiff-le FIELD1:FIELD2:NUM` - abs(FIELD1 - FIELD2) / min(abs(FIELD1), abs(FIELD2)) <= NUM
* `--ff-reldiff-gt FIELD1:FIELD2:NUM` - abs(FIELD1 - FIELD2) / min(abs(FIELD1), abs(FIELD2))  > NUM

**Examples:**

Basic comparisons:
```
$ # Field 2 non-zero
$ tsv-filter --ne 2:0 data.tsv

$ # Field 1 == 0 and Field 2 >= 100, first line is a header.
$ tsv-filter --header --eq 1:0 --ge 2:100 data.tsv

$ # Field 1 == -1 or Field 1 > 100
$ tsv-filter --or --eq 1:-1 --gt 1:100

$ # Field 3 is foo, Field 4 contains bar
$ tsv-filter --header --str-eq 3:foo --str-in-fld 4:bar data.tsv

$ # Field 3 == field 4 (numeric test)
$ tsv-filter --header --ff-eq 3:4 data.tsv
```

Field lists:

Field lists can be used to run the same test on multiple fields. For example:
```
$ # Test that fields 1-10 are not blank
$ tsv-filter --not-blank 1-10 data.tsv

$ # Test that fields 1-5 are not zero
$ tsv-filter --ne 1-5:0

$ # Test that fields 1-5, 7, and 10-20 are less than 100
$ tsv-filter --lt 1-5,7,10-20:100
```

Regular expressions:

The regular expression syntax supported is that defined by the [D regex library](<http://dlang.org/phobos/std_regex.html>). The  basic syntax has become quite standard and is used by many tools. It will rarely be necessary to consult the D language documentation. A general reference such as the guide available at [Regular-Expressions.info](http://www.regular-expressions.info/) will suffice in nearly all cases. (Note: Unicode properties are supported.)

```
$ # Field 2 has a sequence with two a's, one or more digits, then 2 a's.
$ tsv-filter --regex '2:aa[0-9]+aa' data.tsv

$ # Same thing, except the field starts and ends with the two a's.
$ tsv-filter --regex '2:^aa[0-9]+aa$' data.tsv

$ # Field 2 is a sequence of "word" characters with two or more embedded
$ # whitespace sequences (match against entire field)
$ tsv-filter --regex '2:^\w+\s+(\w+\s+)+\w+$' data.tsv

$ # Field 2 containing at least one cyrillic character.
$ tsv-filter --regex '2:\p{Cyrillic}' data.tsv
```

Short-circuiting expressions:

Numeric tests like `--gt` (greater-than) assume field values can be interpreted as numbers. An error occurs if the field cannot be parsed as a number, halting the program. This can be avoiding by including a testing ensure the field is recognizable as a number. For example:

```
$ # Ensure field 2 is a number before testing for greater-than 10.
$ tsv-filter --is-numeric 2 --gt 2:10 data.tsv

$ # Ensure field 2 is a number, not NaN or infinity before greater-than test.
$ tsv-filter --is-finite 2 --gt 2:10 data.tsv
```

The above tests work because `tsv-filter` short-circuits evaluation, only running as many tests as necessary to filter each line. Tests are run in the order listed on the command line. In the first example, if `--is-numeric 2` is false, the remaining tests do not get run.

_**Tip:**_ Bash completion is very helpful when using commands like `tsv-filter` that have many options. See [Enable bash-completion](TipsAndTricks.md#enable-bash-completion) for details.

---

## tsv-join reference

**Synopsis:** tsv-join --filter-file file [options] file [file...]

tsv-join matches input lines against lines from a 'filter' file. The match is based on exact match comparison of one or more 'key' fields. Fields are TAB delimited by default. Matching lines are written to standard output, along with any additional fields from the key file that have been specified.

**Options:**
* `--h|help` - Print help.
* `--h|help-verbose` - Print detailed help.
* `--V|version` - Print version information and exit.
* `--f|filter-file FILE` - (Required) File with records to use as a filter.
* `--k|key-fields <field-list>` - Fields to use as join key. Default: 0 (entire line).
* `--d|data-fields <field-list>` - Data record fields to use as join key, if different than `--key-fields`.
* `--a|append-fields <field-list>` - Filter fields to append to matched records.
* `--H|header` - Treat the first line of each file as a header.
* `--p|prefix STR` - String to use as a prefix for `--append-fields` when writing a header line.
* `--w|write-all STR` - Output all data records. STR is the `--append-fields` value when writing unmatched records. This is an outer join.
* `--e|exclude` - Exclude matching records. This is an anti-join.
* `--delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)
* `--z|allow-duplicate-keys` - Allow duplicate keys with different append values (last entry wins). Default behavior is that this is an error.

**Examples:**

Filter one file based on another, using the full line as the key.
```
$ # Output lines in data.txt that appear in filter.txt
$ tsv-join -f filter.txt data.txt

$ # Output lines in data.txt that do not appear in filter.txt
$ tsv-join -f filter.txt --exclude data.txt
```

Filter multiple files, using fields 2 & 3 as the filter key.
```
$ tsv-join -f filter.tsv --key-fields 2,3 data1.tsv data2.tsv data3.tsv
```

Same as previous, except use field 4 & 5 from the data files.
```
$ tsv-join -f filter.tsv --key-fields 2,3 --data-fields 4,5 data1.tsv data2.tsv data3.tsv
```

Append fields from the filter file to matched records.
```
$ tsv-join -f filter.tsv --key-fields 1 --append-fields 2-5 data.tsv
```

Write out all records from the data file, but when there is no match, write the 'append fields' as NULL. This is an outer join.
```
$ tsv-join -f filter.tsv --key-fields 1 --append-fields 2 --write-all NULL data.tsv
```

Managing headers: Often it's useful to join a field from one data file to anther, where the data fields are related and the headers have the same name in both files. They can be kept distinct by adding a prefix to the filter file header. Example:
```
$ tsv-join -f run1.tsv --header --key-fields 1 --append-fields 2 --prefix run1_ run2.tsv
```

---

## tsv-pretty reference

**Synopsis:** tsv-pretty [options] [file...]

`tsv-pretty` outputs TSV data in a format intended to be more human readable when working on the command line. This is done primarily by lining up data into fixed-width columns. Text is left aligned, numbers are right aligned. Floating points numbers are aligned on the decimal point when feasible.

Processing begins by reading the initial set of lines into memory to determine the field widths and data types of each column. This look-ahead buffer is used for header detection as well. Output begins after this processing is complete.

By default, only the alignment is changed, the actual values are not modified. Several of the formatting options do modify the values.

Features:

* Floating point numbers: Floats can be printed in fixed-width precision, using the same precision for all floats in a column. This makes then line up nicely. Precision is determined by values seen during look-ahead processing. The max precision defaults to 9, this can be changed when smaller or larger values are desired. See the `--f|format-floats` and `--p|precision` options.

* Header lines: Headers are detected automatically when possible. This can be overridden when automatic detection doesn't work as desired. Headers can be underlined and repeated at regular intervals.

* Missing values: A substitute value can be used for empty fields. This is often less confusing than spaces. See `--e|replace-empty` and `--E|empty-replacement`.

* Exponential notion: As part of float formatting, `--f|format-floats` re-formats columns where exponential notation is found so all the values in the column are displayed using exponential notation and the same precision.

* Preamble: A number of initial lines can be designated as a preamble and output unchanged. The preamble is before the header, if a header is present. Preamble lines can be auto-detected via the heuristic that they lack field delimiters. This works well when the field delimiter is a TAB.

* Fonts: Fixed-width fonts are assumed. CJK characters are assumed to be double width. This is not always correct, but works well in most cases.

**Options:**

* `--help-verbose` - Print full help.
* `--H|header` - Treat the first line of each file as a header.
* `--x|no-header` -  Assume no header. Turns off automatic header detection.
* `--l|lookahead NUM` - Lines to read to interpret data before generating output. Default: 1000
* `--r|repeat-header NUM` - Lines to print before repeating the header. Default: No repeating header
* `--u|underline-header` - Underline the header.
* `--f|format-floats` - Format floats for better readability. Default: No
* `--p|precision NUM` - Max floating point precision. Implies --format-floats. Default: 9
* `--e|replace-empty` - Replace empty fields with `--`.
* `--E|empty-replacement STR` - Replace empty fields with a string.
* `--d|delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)
* `--s|space-between-fields NUM` - Spaces between each field (Default: 2)
* `--m|max-text-width NUM` - Max reserved field width for variable width text fields. Default: 40
* `--a|auto-preamble` - Treat initial lines in a file as a preamble if the line contains no field delimiters. The preamble is output unchanged.
* `--b|preamble NUM` - Treat the first NUM lines as a preamble and output them unchanged.
* `--V|version` - Print version information and exit.
* `--h|help` - This help information.

**Examples:**

A tab-delimited file printed without any formatting:
```
$ cat sample.tsv
Color   Count   Ht      Wt
Brown   106     202.2   1.5
Canary Yellow   7       106     0.761
Chartreuse	1139	77.02   6.22
Fluorescent Orange	422     1141.7  7.921
Grey	19	140.3	1.03
```
The same file printed with `tsv-pretty`:
```
$ tsv-pretty sample.tsv
Color               Count       Ht     Wt
Brown                 106   202.2   1.5
Canary Yellow           7   106     0.761
Chartreuse           1139    77.02  6.22
Fluorescent Orange    422  1141.7   7.921
Grey                   19   140.3   1.03
```
Printed with float formatting and header underlining:
```
$ tsv-pretty -f -u sample.tsv
Color               Count       Ht     Wt
-----               -----       --     --
Brown                 106   202.20  1.500
Canary Yellow           7   106.00  0.761
Chartreuse           1139    77.02  6.220
Fluorescent Orange    422  1141.70  7.921
Grey                   19   140.30  1.030
```
Printed with setting the precision to one:
```
$ tsv-pretty -u -p 1 sample.tsv
Color               Count      Ht   Wt
-----               -----      --   --
Brown                 106   202.2  1.5
Canary Yellow           7   106.0  0.8
Chartreuse           1139    77.0  6.2
Fluorescent Orange    422  1141.7  7.9
Grey                   19   140.3  1.0
```

---

## tsv-sample reference

**Synopsis:** tsv-sample [options] [file...]

`tsv-sample` subsamples input lines or randomizes their order. Several techniques are available: shuffling, simple random sampling, weighted random sampling, Bernoulli sampling, and distinct sampling. These are provided via several different modes operation:

* **Shuffling** (_default_): All lines are read into memory and output in random order. All orderings are equally likely.
* **Simple random sampling** (`--n|num N`): A random sample of `N` lines is selected and written to standard output. Selected lines are written in random order, similar to shuffling. All sample sets and orderings are equally likely. Use `--i|inorder` to preserve the original input order.
* **Weighted random sampling** (`--n|num N`, `--w|weight-field F`): A weighted sample of N lines is selected using weights from a field on each line. Selected lines are written in weighted selection order. Use `--i|inorder` to preserve the original input order. Omit `--n|num` to shuffle all input lines (weighted shuffling).
* **Sampling with replacement** (`--r|replace`, `--n|num N`): All lines are read into memory, then lines are selected one at a time at random and written out. Lines can be selected multiple times. Output continues until `N` samples have been written. Output continues forever if `--n|num` is zero or not specified.
* **Bernoulli sampling** (`--p|prob P`): Lines are read one-at-a-time in a streaming fashion and a random subset is output based on the inclusion probability. For example, `--prob 0.2` gives each line a 20% chance of being selected. All lines have an equal likelihood of being selected. The order of the lines is unchanged.
* **Distinct sampling** (`--k|key-fields F`, `--p|prob P`): Input lines are sampled based on a key from each line. A key is made up of one or more fields. A subset of the keys are chosen based on the inclusion probability (a "distinct" set of keys). All lines with one of the selected keys are output. This is a streaming operation: a decision is made on each line as it is read. The order of the lines is not changed.

**Sample size**: The `--n|num` option controls the sample size for all sampling methods. In the case of simple and weighted random sampling it also limits the amount of memory required.

**Performance and memory use**: `tsv-sample` is designed for large data sets. Algorithms make one pass over the data, using reservoir sampling and hashing when possible to limit the memory required. Bernoulli sampling and distinct sampling make immediate decisions on each line, with no memory accumulation. They can operate on arbitrary length data streams. Sampling with replacement reads all lines into memory and is limited by available memory. Shuffling also reads all lines into memory and is similarly limited. Simple and weighted random sampling use reservoir sampling algorithms and only need to hold the sample size (`--n|num`) in memory. See [Shuffling large files](TipsAndTricks.md#shuffling-large-files) for ways to use disk when available memory is not sufficient.

**Controlling randomization**: Each run produces a different randomization. Using `--s|static-seed` changes this so multiple runs produce the same randomization. This works by using the same random seed each run. The random seed can be specified using `--v|seed-value`. This takes a non-zero, 32-bit positive integer. A zero value is a no-op and ignored.

**Weighted sampling**: Weighted line order randomization is done using an algorithm for weighted reservoir sampling described by Pavlos Efraimidis and Paul Spirakis. Weights should be positive values representing the relative weight of the entry in the collection. Counts and similar can be used as weights, it is *not* necessary to normalize to a [0,1] interval. Negative values are not meaningful and given the value zero. Input order is not retained, instead lines are output ordered by the randomized weight that was assigned. This means that a smaller valid sample can be produced by taking the first N lines of output. For more information see:
* Wikipedia: https://en.wikipedia.org/wiki/Reservoir_sampling
* "Weighted Random Sampling over Data Streams", Pavlos S. Efraimidis (https://arxiv.org/abs/1012.0256)

**Distinct sampling**: Distinct sampling selects a subset based on a key in data. Consider a query log with records consisting of <user, query, clicked-url> triples. Distinct sampling selects all records matching a subset of values from one of the fields. For example, all events for ten percent of the users. This is important for certain types of analysis. Distinct sampling works by converting the specified probability (`--p|prob`) into a set of buckets and mapping every key into one of the buckets. One bucket is used to select records in the sample. Buckets are equal size and therefore may be a bit larger than the inclusion probability. Since every key is assigned a bucket, this method can also be used to fully divide a set of records into distinct groups. (See *Printing random values* below.) The term "distinct sampling" originates from algorithms estimating the number of distinct elements in extremely large data sets.

**Printing random values**: Most of these algorithms work by generating a random value for each line. (See also "Compatibility mode" below.) The nature of these values depends on the sampling algorithm. They are used for both line selection and output ordering. The `--print-random` option can be used to print these values. The random value is prepended to the line separated by the `--d|delimiter` char (TAB by default). The `--gen-random-inorder` option takes this one step further, generating random values for all input lines without changing the input order. The types of values currently used are specific to the sampling algorithm:
* Shuffling, simple random sampling, Bernoulli sampling: Uniform random value in the interval [0,1].
* Weighted random sampling: Value in the interval [0,1]. Distribution depends on the values in the weight field.
* Distinct sampling: An integer, zero and up, representing a selection group (aka. "bucket"). The inclusion probability determines the number of selection groups.
* Sampling with replacement: Random value printing is not supported.

The specifics behind these random values are subject to change in future releases.

**Compatibility mode**: As described above, many of the sampling algorithms assign a random value to each line. This is useful when printing random values. It has another occasionally useful property: repeated runs with the same static seed but different selection parameters are more compatible with each other, as each line gets assigned the same random value on every run. This property comes at a cost: in some cases there are faster algorithms that don't assign random values to each line. By default, `tsv-sample` will use the fastest algorithm available. The `--compatibility-mode` option changes this, switching to algorithms that assign a random value per line. Printing random values also engages compatibility mode. Compatibility mode is beneficial primarily when using Bernoulli sampling or random sampling:
* Bernoulli sampling - A run with a larger probability will be a superset of a smaller probability. In the example below, all lines selected in the first run are also selected in the second.
  ```
  $ tsv-sample --static-seed --compatibility-mode --prob 0.2 data.tsv
  $ tsv-sample --static-seed --compatibility-mode --prob 0.3 data.tsv
  ```
* Random sampling - A run with a larger sample size will be a superset of a smaller sample size. In the example below, all lines selected in the first run are also selected in the second.
  ```
  $ tsv-sample --static-seed --compatibility-mode -n 1000 data.tsv
  $ tsv-sample --static-seed --compatibility-mode -n 1500 data.tsv
  ```
  This works for weighted sampling as well.

**Options:**

* `--h|help` - This help information.
* `--help-verbose` - Print more detailed help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header.
* `--n|num NUM` - Maximum number of lines to output. All selected lines are output if not provided or zero.
* `--p|prob NUM` - Inclusion probability (0.0 < NUM <= 1.0). For Bernoulli sampling, the probability each line is selected output. For distinct sampling, the probability each unique key is selected for output.
* `--k|key-fields <field-list>` - Fields to use as key for distinct sampling. Use with `--p|prob`. Specify `--k|key-fields 0` to use the entire line as the key.
* `--w|weight-field NUM` - Field containing weights. All lines get equal weight if not provided or zero.
* `--r|replace` - Simple random sampling with replacement. Use `--n|num` to specify the sample size.
* `--s|static-seed` - Use the same random seed every run.
* `--v|seed-value NUM` - Sets the random seed. Use a non-zero, 32 bit positive integer. Zero is a no-op.
* `--print-random` - Output the random values that were assigned.
* `--gen-random-inorder` - Output all lines with assigned random values prepended, no changes to the order of input.
* `--random-value-header` - Header to use with `--print-random` and `--gen-random-inorder`. Default: `random_value`.
* `--compatibility-mode` - Turns on "compatibility mode".
* `--d|delimiter CHR` - Field delimiter.
* `--prefer-skip-sampling` - (Internal) Prefer the skip-sampling algorithm for Bernoulli sampling. Used for testing and diagnostics.
* `--prefer-algorithm-r` - (Internal) Prefer Algorithm R for unweighted line order randomization. Used for testing and diagnostics.

---

## tsv-select reference

**Synopsis:** tsv-select [options] [file...]

tsv-select reads files or standard input and writes specified fields to standard output in the order listed. Similar to Unix `cut` with the ability to reorder fields.

Fields numbers start with one. They are comma separated, and ranges can be used. Fields can be listed more than once, and fields not listed can be selected as a group using the `--rest` option. When working with multiple files, the `--header` option can be used to retain the header from the just the first file.

Fields can be excluded using `--e|exclude`. All fields not excluded are output. `--f|fields` and `--r|rest` can be used with `--e|exclude` to change the order of non-excluded fields.

**Options:**
* `--h|help` - Print help.
* `--help-verbose` -  Print more detailed help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header.
* `--f|fields <field-list>` - Fields to retain. Fields are output in the order listed.
* `--e|--exclude <field-list>` - Fields to exclude.
* `--r|rest first|last` - Output location for fields not included in the `--f|fields` field-list.
* `--d|delimiter CHR` - Character to use as field delimiter. Default: TAB. (Single byte UTF-8 characters only.)

**Notes:**
* One of `--f|fields` or `--e|exclude` is required.
* Fields specified by `--f|fields` and `--e|exclude` cannot overlap.
* When `--f|fields` and `--e|exclude` are used together, the effect is to specify `--rest last`. This can be overridden by specifying `--rest first`.
* Each input line must be long enough to contain all fields specified with `--f|fields`. This is not necessary for `--e|exclude` fields.

**Examples:**
```
$ # Keep the first field from two files
$ tsv-select -f 1 file1.tsv file2.tsv

$ # Keep fields 1 and 2, retain the header from the first file
$ tsv-select -H -f 1,2 file1.tsv file2.tsv
   
$ # Output fields 2 and 1, in that order
$ tsv-select -f 2,1 file.tsv

$ # Output a range of fields
$ tsv-select -f 3-30 file.tsv

$ # Output a range of fields in reverse order
$ tsv-select -f 30-3 file.tsv

$ # Drop the first field, keep everything else
$ # Equivalent to 'cut -f 2- file.tsv'
$ tsv-select --exclude 1 file.tsv
$ tsv-select -e 1 file.tsv

$ # Move field 1 to the end of the line
$ tsv-select -f 1 --rest first file.tsv

$ # Move fields 7 and 3 to the start of the line
$ tsv-select -f 7,3 --rest last file.tsv

# Output with repeating fields
$ tsv-select -f 1,2,1 file.tsv
$ tsv-select -f 1-3,3-1 file.tsv

$ # Read from standard input
$ cat file*.tsv | tsv-select -f 1,4-7,11

$ # Read from a file and standard input. The '--' terminates command
$ # option processing, '-' represents standard input.
$ cat file1.tsv | tsv-select -f 1-3 -- - file2.tsv

$ # Files using comma as the separator ('simple csv')
$ # (Note: Does not handle CSV escapes.)
$ tsv-select -d , --fields 5,1,2 file.csv

$ # Move field 2 to the front and drop fields 10-15
$ tsv-select -f 2 -e 10-15 file.tsv

$ # Move field 2 to the end, dropping fields 10-15
$ tsv-select -f 2 -rest first -e 10-15 file.tsv
```

---

## tsv-split reference

Synopsis: tsv-split [options] [file...]

Split input lines into multiple output files. There are three modes of operation:

* **Fixed number of lines per file** (`--l|lines-per-file NUM`): Each input block of NUM lines is written to a new file. Similar to Unix `split`.

* **Random assignment** (`--n|num-files NUM`): Each input line is written to a randomly selected output file. Random selection is from NUM files.

* **Random assignment by key** (`--n|num-files NUM`, `--k|key-fields FIELDS`): Input lines are written to output files using fields as a key. Each unique key is randomly assigned to one of NUM output files. All lines with the same key are written to the same file.

**Output files**: By default, files are written to the current directory and have names of the form `part_NNN<suffix>`, with `NNN` being a number and `<suffix>` being the extension of the first input file. If the input file is `file.txt`, the names will take the form `part_NNN.txt`. The suffix is empty when reading from standard input. The numeric part defaults to 3 digits for `--l|lines-per-files`. For `--n|num-files` enough digits are used so all filenames are the same length. The output directory and file names are customizable.

**Header lines**: There are two ways to handle input with headers: write a header to all output files (`--H|header`), or exclude headers from all output files (`--I|header-in-only`). The best choice depends on the follow-up processing. All tsv-utils tools support header lines in multiple input files, but many other tools do not. For example, [GNU parallel](https://www.gnu.org/software/parallel/) works best on files without header lines. (See [Faster processing using GNU parallel](TipsAndTricks.md#faster-processing-using-gnu-parallel) for some info on using GNU parallel and tsv-utils together.)

**About Random assignment** (`--n|num-files`): Random distribution of records to a set of files is a common task. When data fits in memory the preferred approach is usually to shuffle the data and split it into fixed sized blocks. Both of the following command lines accomplish this:
```
$ shuf data.tsv | split -l NUM
$ tsv-sample data.tsv | tsv-split -l NUM
```

However, alternate approaches are needed when data is too large for convenient shuffling. tsv-split's random assignment feature can be useful in these cases. Each input line is written to a randomly selected output file. Note that output files will have similar but not identical numbers of records.

**About Random assignment by key** (`--n|num-files NUM`, `--k|key-fields FIELDS`): This splits a data set into multiple files sharded by key. All lines with the same key are written to the same file. This partitioning enables parallel computation based on the key. For example, statistical calculation (`tsv-summarize --group-by`) or duplicate removal (`tsv-uniq --fields`). These operations can be parallelized using tools like GNU parallel, which simplifies concurrent operations on multiple files.

**Random seed**: By default, each tsv-split invocation using random assignment or random assignment by key produces different assignments to the output files. Using `--s|static-seed` changes this so multiple runs produce the same assignments. This works by using the same random seed each run. The seed can be specified using `--v|seed-value`.

**Appending to existing files**: By default, an error is triggered if an output file already exists. `--a|append` changes this so that lines are appended to existing files. (Header lines are not appended to files with data.) This is useful when adding new data to files created by a previous `tsv-split` run. Random assignment should use the same `--n|num-files` value each run, but different random seeds (avoid `--s|static-seed`). Random assignment by key should use the same `--n|num-files`, `--k|key-fields`, and seed (`--s|static-seed` or `--v|seed-value`) each run.

**Max number of open files**: Random assignment and random assignment by key are dramatically faster when all output files are kept open. However, keeping a large numbers of open files can bump into system limits or limit resources available to other processes. By default, `tsv-split` uses up to 4096 open files or the system per-process limit, whichever is smaller. This can be changed using `--max-open-files`, though it cannot be set larger than the system limit. The system limit varies considerably between systems. On many systems it is unlimited. On MacOS it is often set to 256. Use Unix `ulimit` to display and modify the limits:
```
$ ulimit -n       # Show the "soft limit". The per-process maximum.
$ ulimit -Hn      # Show the "hard limit". The max allowed soft limit.
$ ulimit -Sn NUM  # Change the "soft limit" to NUM.
```

**Examples**:
```
$ # Split a 10 million line file into 1000 files, 10,000 lines each.
$ # Output files are part_000.txt, part_001.txt, ... part_999.txt.
$ tsv-split data.txt --lines-per-file 10000

$ # Same as the previous example, but write files to a subdirectory.
$  tsv-split data.txt --dir split_files --lines-per-file 10000

$ # Split a file into 10,000 line files, writing a header line to each
$ tsv-split data.txt -H --lines-per-file 10000

$ # Same as the previous example, but dropping the header line.
$ tsv-split data.txt -I --lines-per-file 10000

$ # Randomly assign lines to 1000 files
$ tsv-split data.txt --num-files 1000

$ # Randomly assign lines to 1000 files while keeping unique keys from
$  # field 3 together.
$  tsv-split data.tsv --num-files 1000 -k 3

$ # Randomly assign lines to 1000 files. Later, randomly assign lines
$ # from a second data file to the same output files.
$ tsv-split data1.tsv -n 1000
$ tsv-split data2.tsv -n 1000 --append

$ # Randomly assign lines to 1000 files using field 3 as a key.
$ # Later, add a second file to the same output files.
$ tsv-split data1.tsv -n 1000 -k 3 --static-seed
$ tsv-split data2.tsv -n 1000 -k 3 --static-seed --append

$ # Change the system per-process open file limit for one command.
$ # The parens create a sub-shell. The current shell is not changed.
$ ( ulimit -Sn 1000 && tsv-split --num-files 1000 data.txt )
```

**Options**:
* `--h|--help` - Print help.
* `--help-verbose` - Print more detailed help.
* `--V|--version` -  Print version information and exit.
* `--H|header` - Input files have a header line. Write the header to each output file.
* `--I|header-in-only` - Input files have a header line. Do not write the header to output files.
* `--l|lines-per-file NUM` - Number of lines to write to each output file (excluding the header line).
* `--n|num-files NUM` - Number of output files to generate.
* `--k|key-fields <field-list>` - Fields to use as key. Lines with the same key are written to the same output file. Use `--k|key-fields 0` to use the entire line as the key.
* `--dir STR` - Directory to write to. Default: Current working directory.
* `--prefix STR` - Filename prefix. Default: `part_`
* `--suffix STR` - Filename suffix. Default: First input file extension. None for standard input.
* `--w|digit-width NUM` - Number of digits in filename numeric portion. Default: `--l|lines-per-file`: 3. `--n|num-files`: Chosen so filenames have the same length. `--w|digit-width 0` uses the default.
* `--a|append` - Append to existing files.
* `--s|static-seed` - Use the same random seed every run.
* `--v|seed-value NUM` - Sets the random seed. Use a non-zero, 32 bit positive integer. Zero is a no-op.
* `--d|delimiter CHR` - Field delimiter.
* `--max-open-files NUM` - Maximum open file handles to use. Min of 5 required.

---

## tsv-summarize reference

Synopsis: tsv-summarize [options] file [file...]

`tsv-summarize` generates summary statistics on fields of a TSV file. A variety of statistics are supported. Calculations can run against the entire data stream or grouped by key. Consider the file data.tsv:
```
make    color   time
ford    blue    131
chevy   green   124
ford    red     128
bmw     black   118
bmw     black   126
ford    blue    122
```

The min and average 'time' values for the 'make' field is generated by the command:
```
$ tsv-summarize --header --group-by 1 --min 3 --mean 3 data.tsv
```

This produces:
```
make   time_min time_mean
ford   122      127
chevy  124      124
bmw    118      122
```

Using `--group-by 1,2` will group by both 'make' and 'color'. Omitting the `--group-by` entirely summarizes fields for full file.

The program tries to generate useful headers, but custom headers can be specified. Example:
```
$ tsv-summarize --header --group-by 1 --min 3:fastest --mean 3:average data.tsv
make	fastest	average
ford	122	127
chevy	124	124
bmw	118	122
```

Most operators take custom headers in a manner shown above, following the syntax:
```
--<operator-name> FIELD[:header]
```

Operators can be specified multiple times. They can also take multiple fields (though not when a custom header is specified). Examples:
```
--median 2,3,4
--median 1,5-8
```

The quantile operator requires one or more probabilities after the fields:
```
--quantile 2:0.25              # Quantile 1 of field 2
--quantile 2-4:0.25,0.5,0.75   # Q1, Median, Q3 of fields 2, 3, 4
```

Summarization operators available are:
```
   count       range        mad            values
   retain      sum          var            unique-values
   first       mean         stddev         unique-count
   last        median       mode           missing-count
   min         quantile     mode-count     not-missing-count
   max
```

Calculated numeric values are printed to 12 significant digits by default. This can be changed using the `--p|float-precision` option. If six or less it sets the number of significant digits after the decimal point. If greater than six it sets the total number of significant digits.

Calculations hold onto the minimum data needed while reading data. A few operations like median keep all data values in memory. These operations will start to encounter performance issues as available memory becomes scarce. The size that can be handled effectively is machine dependent, but often quite large files can be handled.

Operations requiring numeric entries will signal an error and terminate processing if a non-numeric entry is found.

Missing values are not treated specially by default, this can be changed using the `--x|exclude-missing` or `--r|replace-missing` option. The former turns off processing for missing values, the latter uses a replacement value.

**Options:**
* `--h|help` - Print help.
* `--help-verbose` - Print detailed help.
* `--V|version` - Print version information and exit.
* `--g|group-by <field-list>` - Fields to use as key.
* `--H|header` - Treat the first line of each file as a header.
* `--w|write-header` - Write an output header even if there is no input header.
* `--d|delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)
* `--v|values-delimiter CHR` - Values delimiter. Default: vertical bar (\|). (Single byte UTF-8 characters only.)
* `--p|float-precision NUM` - 'Precision' to use printing floating point numbers. Affects the number of digits printed and exponent use. Default: 12
* `--x|exclude-missing` - Exclude missing (empty) fields from calculations.
* `--r|replace-missing STR` - Replace missing (empty) fields with STR in calculations.

**Operators:**
* `--count` - Count occurrences of each unique key (`--g|group-by`), or the total number of records if no key field is specified.
* `--count-header STR` - Count occurrences of each unique key, like `--count`, but use STR as the header.
* `--retain <field-list>` - Retain one copy of the field. The field header is unchanged.
* `--first <field-list>[:STR]` - First value seen.
* `--last <field-list>[:STR]`- Last value seen.
* `--min <field-list>[:STR]` - Min value. (Numeric fields only.)
* `--max <field-list>[:STR]` - Max value. (Numeric fields only.)
* `--range <field-list>[:STR]` - Difference between min and max values. (Numeric fields only.)
* `--sum <field-list>[:STR]` - Sum of the values. (Numeric fields only.)
* `--mean <field-list>[:STR]` - Mean (average). (Numeric fields only.)
* `--median <field-list>[:STR]` - Median value. (Numeric fields only. Reads all values into memory.)
* `--quantile <field-list>:p[,p...][:STR]` - Quantiles. One or more fields, then one or more 0.0-1.0 probabilities. (Numeric fields only. Reads all values into memory.)
* `--mad <field-list>[:STR]` - Median absolute deviation from the median. Raw value, not scaled. (Numeric fields only. Reads all values into memory.)
* `--var <field-list>[:STR]` - Variance. (Sample variance, numeric fields only).
* `--stdev <field-list>[:STR]` - Standard deviation. (Sample st.dev, numeric fields only).
* `--mode <field-list>[:STR]` - Mode. The most frequent value. (Reads all unique values into memory.)
* `--mode-count <field-list>[:STR]` - Count of the most frequent value. (Reads all unique values into memory.)
* `--unique-count <field-list>[:STR]` - Number of unique values. (Reads all unique values into memory).
* `--missing-count <field-list>[:STR]` - Number of missing (empty) fields. Not affected by the `--x|exclude-missing` or `--r|replace-missing` options.
* `--not-missing-count <field-list>[:STR]` - Number of filled (non-empty) fields. Not affected by `--r|replace-missing`.
* `--values <field-list>[:STR]` - All the values, separated by `--v|values-delimiter`. (Reads all values into memory.)
* `--unique-values <field-list>[:STR]` - All the unique values, separated by `--v|values-delimiter`. (Reads all unique values into memory.)

_**Tip:**_ Bash completion is very helpful when using commands like `tsv-summarize` that have many options. See [Enable bash-completion](TipsAndTricks.md#enable-bash-completion) for details.

---

## tsv-uniq reference

`tsv-uniq` identifies equivalent lines in files or standard input. Input is read line by line, recording a key based on one or more of the fields. Two lines are equivalent if they have the same key. When operating in the default 'uniq' mode, the first time a key is seen the line is written to standard output. Subsequent lines having the same key are discarded. This is similar to the Unix `uniq` program, but based on individual fields and without requiring sorted data.

`tsv-uniq` can be run without specifying a key field. In this case the whole line is used as a key, same as the Unix `uniq` program. As with `uniq`, this works on any line-oriented text file, not just TSV files. There is no need to sort the data and the original input order is preserved.

The alternates to the default 'uniq' mode are 'number' mode and 'equiv-class' mode. In 'equiv-class' mode (`--e|equiv`), all lines are written to standard output, but with a field appended marking equivalent entries with an ID. The ID is a one-upped counter.

'Number' mode (`--z|number`) also writes all lines to standard output, but with a field appended numbering the occurrence count for the line's key. The first line with a specific key is assigned the number '1', the second with the key is assigned number '2', etc. 'Number' and 'equiv-class' modes can be used together.

The `--r|repeated` option can be used to print only lines occurring more than once. Specifically, the second occurrence of a key is printed. The `--a|at-least N` option is similar, printing lines occurring at least N times. (Like repeated, the Nth line with the key is printed.)

The `--m|max MAX` option changes the behavior to output the first MAX lines for each key, rather than just the first line for each key.

If both `--a|at-least` and `--m|max` are specified, the occurrences starting with 'at-least' and ending with 'max' are output.

**Synopsis:** tsv-uniq [options] [file...]

**Options:**
* `-h|help` - Print help.
* `--help-verbose` - Print detailed help.
* `--V|version` - Print version information and exit.
* `--H|header` - Treat the first line of each file as a header.
* `--f|fields <field-list>` - Fields to use as the key. Default: 0 (entire line).
* `--i|ignore-case` - Ignore case when comparing keys.
* `--e|equiv` - Output equiv class IDs rather than uniq'ing entries.
* `--equiv-header STR` - Use STR as the equiv-id field header. Applies when using `--header --equiv`. Default: `equiv_id`.
* `--equiv-start INT` - Use INT as the first equiv-id. Default: 1.
* `--z|number` - Output equivalence class occurrence counts rather than uniq'ing entries.
* `--number-header STR` - Use STR as the `--number` field header (when using `-H --number`). Default: `equiv_line`.
* `--r|repeated` - Output only lines that are repeated (based on the key).
* `--a|at-least INT` - Output only lines that are repeated INT times (based on the key). Zero and one are ignored.
* `--m|max INT` - Max number of each unique key to output (zero is ignored).
* `--d|delimiter CHR` - Field delimiter. Default: TAB. (Single byte UTF-8 characters only.)

**Examples:**
```
$ # Uniq a file, using the full line as the key
$ tsv-uniq data.txt

$ # Same as above, but case-insensitive
$ tsv-uniq --ignore-case data.txt

$ # Unique a file based on one field
$ tsv-unique -f 1 data.tsv

$ # Unique a file based on two fields
$ tsv-uniq -f 1,2 data.tsv

$ # Unique a file based on the first four fields
$ tsv-uniq -f 1-4 data.tsv

$ # Output all the lines, generating an ID for each unique entry
$ tsv-uniq -f 1,2 --equiv data.tsv

$ # Generate uniq IDs, but account for headers
$ tsv-uniq -f 1,2 --equiv --header data.tsv

$ # Generate line numbers specific to each key
$ tsv-uniq -f 1,2 --number --header data.tsv

$ # --Examples showing the data--

$ cat data.tsv
field1  field2  field2
ABCD    1234    PQR
efgh    5678    stu
ABCD    1234    PQR
wxyz    1234    stu
efgh    5678    stu
ABCD    1234    PQR

$ # Uniq using the full line as key
$ tsv-uniq -H data.tsv
field1  field2  field2
ABCD    1234    PQR
efgh    5678    stu
wxyz    1234    stu

$ # Uniq using field 2 as key
$ tsv-uniq -H -f 2 data.tsv
field1  field2  field2
ABCD    1234    PQR
efgh    5678    stu

$ # Generate equivalence class IDs
$ tsv-uniq -H --equiv data.tsv
field1  field2  field2  equiv_id
ABCD    1234    PQR     1
efgh    5678    stu     2
ABCD    1234    PQR     1
wxyz    1234    stu     3
efgh    5678    stu     2
ABCD    1234    PQR     1

$ # Generate equivalence class IDs and line numbers
$ tsv-uniq -H --equiv --number data.tsv
field1	field2	field2	equiv_id  equiv_line
ABCD    1234    PQR     1         1
efgh    5678    stu     2         1
ABCD    1234    PQR     1         2
wxyz    1234    stu     3         1
efgh    5678    stu     2         2
ABCD    1234    PQR     1         3
```
