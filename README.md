# perl-notes

Some tips / things of notes for myself while I'm learning Perl

## Perl Variables and Data Types

### Variable Sigils: $, @, and %

Perl uses different sigils (symbols) to indicate variable types and context:

#### **$ (Scalar Variables)**

- **Single values**: strings, numbers, references
- **Context**: Scalar context
- **Examples**:

```perl
my $name = "John";
my $age = 30;
my $price = 19.99;
my $reference = \@array;  # Reference to array
```

#### **@ (Array Variables)**

- **Lists of values**: indexed collections
- **Context**: List context
- **Examples**:

```perl
my @fruits = ("apple", "banana", "cherry");
my @numbers = (1, 2, 3, 4, 5);
my @mixed = ("text", 42, 3.14);

# Array operations
push @fruits, "date";
pop @fruits;  # Remove last element
shift @fruits;  # Remove first element
unshift @fruits, "grape";  # Add to beginning
```

#### **% (Hash Variables)**

- **Key-value pairs**: associative arrays
- **Context**: List context (when flattened)
- **Examples**:

```perl
my %person = (
    name => "John",
    age => 30,
    city => "New York"
);

# Hash operations
$person{email} = "john@example.com";  # Add new key
delete $person{age};  # Remove key
my @keys = keys %person;  # Get all keys
my @values = values %person;  # Get all values
```

#### **Context Matters**

```perl
my @array = (1, 2, 3, 4, 5);
my $scalar = @array;  # $scalar = 5 (array length)
my @list = @array;    # @list = (1, 2, 3, 4, 5) (copy)

my %hash = (a => 1, b => 2);
my $scalar = %hash;   # $scalar = "2/8" (fractional representation)
my @list = %hash;     # @list = ("a", 1, "b", 2) (flattened)
```

## Functions and Subroutines

### Function Declaration and Usage

#### **Basic Subroutine Syntax**

```perl
sub greet {
    my ($name, $greeting) = @_;
    return "$greeting, $name!";
}

# Calling the function
my $message = greet("Alice", "Hello");
print $message;  # Output: Hello, Alice!
```

#### **Parameter Handling**

```perl
sub process_data {
    my ($required, $optional, $rest) = @_;

    # Default values
    $optional //= "default";

    # Process rest of arguments
    my @remaining = @_[3..$#_];

    return "Processed: $required, $optional";
}
```

#### **Return Values**

```perl
sub get_coordinates {
    my ($x, $y) = @_;
    return ($x, $y);  # Returns a list
}

sub get_person {
    my ($name, $age) = @_;
    return {           # Returns a hash reference
        name => $name,
        age => $age
    };
}

# Using return values
my ($lat, $lon) = get_coordinates(40.7128, -74.0060);
my $person = get_person("John", 30);
print $person->{name};  # Output: John
```

#### **Prototypes (Advanced)**

```perl
sub add($$) {  # Expects exactly 2 scalars
    my ($a, $b) = @_;
    return $a + $b;
}

sub process_array(\@) {  # Expects array reference
    my ($array_ref) = @_;
    # Process array
}
```


## Perl Special Variables

Perl has many special variables that provide access to system information, process details, and runtime context. These variables are essential for effective Perl programming and script automation.

### Table of Contents

- [Default and Pattern Variables](#default-and-pattern-variables)
- [Error and Status Variables](#error-and-status-variables)
- [Process and System Variables](#process-and-system-variables)
- [Input/Output Variables](#inputoutput-variables)
- [Environment and Context Variables](#environment-and-context-variables)
- [Regular Expression Variables](#regular-expression-variables)
- [Array and Hash Variables](#array-and-hash-variables)

### Default and Pattern Variables

| Variable        | Description                         | Example Usage                          |
| --------------- | ----------------------------------- | -------------------------------------- |
| `$_`            | Default variable for many functions | `print if /pattern/;`                  |
| `@_`            | Arguments passed to a subroutine    | `sub example { print $_[0]; }`         |
| `$1, $2, $3...` | Capture groups from regex matches   | `=~/(\\w+)\\s+(\\w+)/; print "$1 $2";` |
| `$&`            | The entire matched string           | `print "Matched: $&" if /pattern/;`    |
| `` $` ``        | String before the match             | ``print "Before: $`" if /pattern/;``   |
| `$'`            | String after the match              | `print "After: $'" if /pattern/;`      |

#### Examples:

```perl
# Using $_ (default variable)
for my $line (<DATA>) {
    print if /ERROR/;  # $_ is implicit
}

# Using subroutine arguments (@_)
sub greet {
    my ($name, $greeting) = @_;
    print "$greeting, $name!\n";
}
greet("Alice", "Hello");

# Using capture groups
my $text = "John Doe";
if ($text =~ /(\w+)\s+(\w+)/) {
    print "First: $1, Last: $2\n";
}
```

### Error and Status Variables

| Variable | Description                                | Example Usage                            |
| -------- | ------------------------------------------ | ---------------------------------------- |
| `$!`     | Error message from system calls            | `open my $fh, "<", "file" or die $!;`    |
| `$@`     | Error message from `eval`                  | `eval { ... }; print $@ if $@;`          |
| `$?`     | Status of last system command or backticks | `system("ls"); print "Exit code: $?\n";` |
| `$^E`    | Extended OS error (Windows)                | `print "OS Error: $^E\n";`               |

#### Examples:

```perl
# File operation error handling
open my $fh, "<", "nonexistent.txt" or die "Cannot open file: $!";

# eval error handling
eval {
    my $result = 10 / 0;
};
if ($@) {
    print "Caught error: $@\n";
}

# System command status
system("false");  # Command that returns non-zero
print "Command failed with exit code: " . ($? >> 8) . "\n";
```

### Process and System Variables

| Variable | Description                    | Example Usage             |
| -------- | ------------------------------ | ------------------------- |
| `$$`     | Process ID of current script   | `print "PID: $$\n";`      |
| `$0`     | Name of the current script     | `print "Script: $0\n";`   |
| `$^O`    | Operating system name          | `print "OS: $^O\n";`      |
| `$^X`    | Path to Perl interpreter       | `print "Perl: $^X\n";`    |
| `$]`     | Perl version number            | `print "Version: $]\n";`  |
| `$^V`    | Perl version as version object | `print "Version: $^V\n";` |

#### Examples:

```perl
# Process information
print "Running script '$0' with PID $$ on $^O\n";
print "Using Perl $^V at $^X\n";

# Creating lock files with PID
my $lockfile = "/tmp/script.lock.$$";
open my $lock, ">", $lockfile or die "Cannot create lock: $!";
print $lock "$$\n";
close $lock;
```

### Input/Output Variables

| Variable | Description                            | Example Usage                      |
| -------- | -------------------------------------- | ---------------------------------- | --- | ------------------------- |
| `$/`     | Input record separator (default `\n`)  | `local $/ = undef;`                |
| `$\`     | Output record separator                | `$\ = "\n"; print "auto newline";` |
| `$,`     | Output field separator                 | `$, = ", "; print @array;`         |
| `$"`     | List separator for array interpolation | `$" = ":"; print "@array";`        |
| `$.`     | Current line number                    | `print "Line $.: $_" while <>;`    |
| `$       | `                                      | Auto-flush output buffer           | `$  | = 1; # Enable auto-flush` |

#### Examples:

```perl
# Reading entire file at once
{
    local $/ = undef;  # Slurp mode
    my $content = <$fh>;
    print "File size: " . length($content) . " characters\n";
}

# Custom field separators
{
    local $, = " | ";  # Set field separator
    print "one", "two", "three";  # Outputs: one | two | three
}

# Line numbering
while (<DATA>) {
    chomp;
    print "Line $.: $_\n";
}
```

### Environment and Context Variables

| Variable | Description                    | Example Usage                         |
| -------- | ------------------------------ | ------------------------------------- |
| `%ENV`   | Environment variables hash     | `print $ENV{'PATH'};`                 |
| `@ARGV`  | Command line arguments         | `print "Args: @ARGV\n";`              |
| `$#ARGV` | Index of last element in @ARGV | `print "Arg count: " . ($#ARGV + 1);` |
| `@INC`   | Module search paths            | `print "Paths: @INC\n";`              |
| `%INC`   | Loaded modules hash            | `print "Loaded: " . keys(%INC);`      |

#### Examples:

```perl
# Environment variables
print "User: $ENV{USER}\n" if exists $ENV{USER};
print "Home: $ENV{HOME}\n" if exists $ENV{HOME};

# Command line processing
if (@ARGV) {
    print "Processing " . scalar(@ARGV) . " arguments:\n";
    for my $i (0..$#ARGV) {
        print "  Arg $i: $ARGV[$i]\n";
    }
} else {
    print "No arguments provided\n";
}

# Module information
use Data::Dumper;
print "Data::Dumper loaded from: $INC{'Data/Dumper.pm'}\n";
```

### Regular Expression Variables

| Variable | Description                       | Example Usage                        |
| -------- | --------------------------------- | ------------------------------------ |
| `$+`     | Last matched capture group        | `/(a+)(b+)/; print "Last: $+";`      |
| `$^N`    | Most recent capture group         | Used in regex callbacks              |
| `@+`     | End positions of capture groups   | `print "Ends at: $+[1]";`            |
| `@-`     | Start positions of capture groups | `print "Starts at: $-[1]";`          |
| `%+`     | Named capture groups              | `/(?\<name\>\\w+)/; print $+{name};` |

#### Examples:

```perl
# Capture group positions
my $text = "Hello World";
if ($text =~ /(H\w+)\s+(W\w+)/) {
    print "Match 1: '$1' at positions $-[1]-$+[1]\n";
    print "Match 2: '$2' at positions $-[2]-$+[2]\n";
    print "Last capture: '$+'\n";
}

# Named captures
if ($text =~ /(?<greeting>H\w+)\s+(?<target>W\w+)/) {
    print "Greeting: $+{greeting}\n";
    print "Target: $+{target}\n";
}
```

### Array and Hash Variables

| Variable  | Description                    | Example Usage                    |
| --------- | ------------------------------ | -------------------------------- |
| `$#array` | Index of last element in array | `print "Last index: $#array";`   |
| `$[`      | Array base index (deprecated)  | Not recommended for use          |
| `@ISA`    | Inheritance array for packages | `@MyClass::ISA = qw(BaseClass);` |

#### Examples:

```perl
# Array operations
my @fruits = qw(apple banana cherry);
print "Array has " . ($#fruits + 1) . " elements\n";
print "Last element: $fruits[$#fruits]\n";

# Adding elements
push @fruits, "date";
print "Now has " . scalar(@fruits) . " elements\n";
```

### Practical Examples

#### Error Handling Pattern

```perl
sub safe_operation {
    my ($filename) = @_;

    open my $fh, "<", $filename or do {
        warn "Cannot open '$filename': $!";
        return;
    };

    my $content = do { local $/; <$fh> };
    close $fh;

    return $content;
}
```

#### Process Management

```perl
# Create a unique temporary file
my $temp_file = "/tmp/process_$$.tmp";
print "Creating temporary file: $temp_file\n";

# Signal handler using process ID
$SIG{INT} = sub {
    print "\nReceived interrupt signal in process $$\n";
    unlink $temp_file if -e $temp_file;
    exit 1;
};
```

#### Configuration Based on Environment

```perl
# Different behavior based on OS
my $config = do {
    if ($^O eq 'MSWin32') {
        { path_sep => '\\', line_end => "\r\n" }
    } elsif ($^O =~ /darwin|linux/) {
        { path_sep => '/', line_end => "\n" }
    } else {
        { path_sep => '/', line_end => "\n" }  # Default Unix-like
    }
};

print "Running on $^O with path separator '$config->{path_sep}'\n";
```

### Best Practices

1. **Use `local` for temporary changes** to special variables:

   ```perl
   {
       local $/ = "";  # Paragraph mode
       while (<DATA>) {
           # Process paragraphs
       }
   }  # $/ automatically restored
   ```

2. **Always check error variables** after system operations:

   ```perl
   system("important_command") == 0 or die "Command failed: $?";
   ```

3. **Use meaningful variable names** instead of special variables when possible:

   ```perl
   my ($name, $age) = @_;  # Better than using $_[0], $_[1]
   ```

4. **Be careful with global special variables** - they can affect other parts of your program.

### Common Pitfalls

- `$/` affects all file reading operations globally
- `$|` affects all output, not just the current filehandle
- `$_` can be modified by many functions, causing unexpected behavior
- `@_` is aliased to actual arguments, modifications affect the original values

This reference covers the most commonly used Perl special variables. Understanding these variables is crucial for writing effective Perl scripts and understanding existing Perl code.

## Perl Regular Expressions - Comprehensive Notes

### Overview

Perl is renowned for its powerful regular expression capabilities. Regular expressions (regex) in Perl are patterns used to match character combinations in strings. They are fundamental to text processing and pattern matching in Perl.

More information on Regexes can be found in the [Perl Documentation](https://perldoc.perl.org/perlre) and regexes can be tested [Here](https://regex101.com/), selecting PCRE (PHP <7.3) to conform to Perl.

### Basic Syntax

#### Pattern Matching Operator

```perl
if ($string =~ /pattern/) { ... }      # Matches pattern in $string
if ($string !~ /pattern/) { ... }     # Does NOT match pattern in $string
```

#### Basic Pattern Delimiters

```perl
/pattern/         # Standard delimiter
m/pattern/        # Explicit match operator
m!pattern!        # Alternative delimiter
m{pattern}        # Balanced delimiters
m(pattern)        # Parentheses as delimiter
```

### Core Operations

#### 1. Match

```perl
if ($string =~ /pattern/) {
    print "Match found!\n";
}
```

#### 2. Match with Capture

```perl
if ($string =~ /(\d+)-(\d+)-(\d+)/) {
    my ($year, $month, $day) = ($1, $2, $3);
    print "Date: $month/$day/$year\n";
}
```

#### 3. Substitute

```perl
$string =~ s/pattern/replacement/;      # Replace first occurrence
$string =~ s/pattern/replacement/g;     # Replace all occurrences (global)
```

#### 4. Global Substitute

```perl
$string =~ s/old/new/g;                 # Replace all occurrences
$string =~ s/pattern/replacement/gi;    # Case-insensitive global replace
```

#### 5. Substitute with Modifiers

```perl
$string =~ s/pattern/replacement/gix;   # Global, case-insensitive, extended
```

#### 6. Match and Modify

```perl
$string =~ s/(pattern)/$1 modified/;    # Capture and modify
```

#### 7. Global Match

```perl
my @matches = ($string =~ /pattern/g);  # Find all matches
while ($string =~ /pattern/g) {         # Process each match
    print "Found: $&\n";
}
```

#### 8. Split

```perl
my @array = split /pattern/, $string;   # Split string using pattern as delimiter
```

#### 9. Positive Lookahead

```perl
if ($string =~ /pattern(?=lookahead)/) { ... }  # Matches pattern only if followed by lookahead
```

#### 10. Negative Lookahead

```perl
if ($string =~ /pattern(?!lookahead)/) { ... }  # Matches pattern only if NOT followed by lookahead
```

#### 11. Positive Lookbehind

```perl
if ($string =~ /(?<=lookbehind)pattern/) { ... } # Matches pattern only if preceded by lookbehind
```

#### 12. Negative Lookbehind

```perl
if ($string =~ /(?<!lookbehind)pattern/) { ... } # Matches pattern only if NOT preceded by lookbehind
```

#### 13. Find and Print

```perl
while ($string =~ /pattern/g) {
    print "Match: $&\n";               # $& contains the matched text
}
```

#### 14. Replace in File

```perl
perl -pe 's/pattern/replacement/g' file.txt    # One-liner to replace in file
```

### Regular Expression Modifiers

#### Common Modifiers

- `i` - Case insensitive
- `g` - Global (find/replace all occurrences)
- `m` - Multiline mode (^ and $ match line boundaries)
- `s` - Single line mode (. matches newlines)
- `x` - Extended mode (ignore whitespace, allow comments)
- `o` - Compile once (for efficiency with variables)

#### Examples with Modifiers

```perl
$string =~ /pattern/i;          # Case insensitive
$string =~ /pattern/g;          # Global match
$string =~ /pattern/gi;         # Case insensitive + global
$string =~ /pattern/gix;        # Case insensitive + global + extended
```

### Character Classes and Metacharacters

#### Basic Character Classes

```perl
.           # Any character except newline
\d          # Digit [0-9]
\D          # Non-digit [^0-9]
\w          # Word character [a-zA-Z0-9_]
\W          # Non-word character [^a-zA-Z0-9_]
\s          # Whitespace [ \t\n\r\f]
\S          # Non-whitespace [^ \t\n\r\f]
```

#### Custom Character Classes

```perl
[abc]       # Match a, b, or c
[a-z]       # Match any lowercase letter
[A-Z]       # Match any uppercase letter
[0-9]       # Match any digit
[^abc]      # Match anything except a, b, or c
[a-zA-Z0-9] # Match alphanumeric characters
```

#### Quantifiers

```perl
*           # Zero or more
+           # One or more
?           # Zero or one (optional)
{n}         # Exactly n times
{n,}        # n or more times
{n,m}       # Between n and m times
*?          # Non-greedy zero or more
+?          # Non-greedy one or more
??          # Non-greedy zero or one
```

#### Anchors

```perl
^           # Beginning of string/line
$           # End of string/line
\b          # Word boundary
\B          # Non-word boundary
\A          # Beginning of string only
\Z          # End of string only
\z          # Absolute end of string
```

### Capture Groups

#### Basic Capturing

```perl
if ($string =~ /(\d+)-(\d+)-(\d+)/) {
    print "Year: $1, Month: $2, Day: $3\n";
}
```

#### Named Captures (Perl 5.10+)

```perl
if ($string =~ /(?<year>\d+)-(?<month>\d+)-(?<day>\d+)/) {
    print "Year: $+{year}, Month: $+{month}, Day: $+{day}\n";
}
```

#### Non-capturing Groups

```perl
/(?:pattern)/       # Non-capturing group
```

### Advanced Features

#### Backreferences

```perl
/(\w+)\s+\1/        # Match repeated words (e.g., "the the")
```

#### Recursive Patterns (Perl 5.10+)

```perl
/\( (?: [^()]+ | (?R) )* \)/x    # Match balanced parentheses
```

#### Conditional Patterns

```perl
/(condition)(?(1)then|else)/     # If condition matches, use "then", else "else"
```

### Practical Examples

#### Email Validation

```perl
if ($email =~ /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {
    print "Valid email\n";
}
```

#### Phone Number Extraction

```perl
my @phones = ($text =~ /\b\d{3}-\d{3}-\d{4}\b/g);
```

#### URL Parsing

```perl
if ($url =~ m{^(https?)://([^/]+)(/.*)$}) {
    my ($protocol, $host, $path) = ($1, $2, $3);
    print "Protocol: $protocol, Host: $host, Path: $path\n";
}
```

#### Log File Processing

```perl
while (<LOG>) {
    if (/^(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)$/) {
        my ($date, $time, $level, $message) = ($1, $2, $3, $4);
        print "[$level] $date $time: $message\n" if $level eq 'ERROR';
    }
}
```

#### Text Cleaning

```perl
$text =~ s/^\s+|\s+$//g;        # Trim whitespace
$text =~ s/\s+/ /g;             # Normalize whitespace
$text =~ s/[^\w\s]//g;          # Remove non-word characters
```

### Special Variables

#### Match Variables

```perl
$&          # The matched string
$`          # String before the match
$'          # String after the match
$1, $2, $3  # Captured groups
$+          # Last captured group
@+          # Array of match end positions
@-          # Array of match start positions
%+          # Named capture hash
```

#### Example Usage

```perl
if ($string =~ /(perl) (regex)/) {
    print "Before: '$`'\n";     # Text before match
    print "Match: '$&'\n";      # The full match
    print "After: '$''\n";      # Text after match
    print "Group 1: '$1'\n";    # First capture group
    print "Group 2: '$2'\n";    # Second capture group
}
```

### Performance Tips

1. **Compile Once**: Use the `o` modifier for patterns with variables

   ```perl
   /$variable/o
   ```

2. **Anchor Patterns**: Use `^` and `$` when appropriate

   ```perl
   /^pattern$/     # Much faster for exact matches
   ```

3. **Avoid Backtracking**: Use atomic groups and possessive quantifiers

   ```perl
   /(?>\d+)/       # Atomic group
   /\d++/          # Possessive quantifier
   ```

4. **Use `\Q...\E` for Literal Strings**
   ```perl
   /\Q$literal_string\E/
   ```

### Common Pitfalls

1. **Greedy vs Non-greedy Matching**

   ```perl
   # Greedy (matches as much as possible)
   $html =~ /<.*>/;        # Bad: matches entire string

   # Non-greedy (matches as little as possible)
   $html =~ /<.*?>/;       # Good: matches individual tags
   ```

2. **Escaping Special Characters**

   ```perl
   /\./            # Literal dot
   /\$/            # Literal dollar sign
   /\(/            # Literal parenthesis
   ```

3. **Case Sensitivity**
   ```perl
   /perl/          # Case sensitive
   /perl/i         # Case insensitive
   ```

### Best Practices

1. Use meaningful variable names for captures
2. Comment complex regex patterns using the `x` modifier
3. Test regex patterns thoroughly with edge cases
4. Consider using `qr//` for reusable patterns
5. Use appropriate anchors to avoid unintended matches
6. Validate user input with regex carefully to avoid security issues

### Example: Complex Pattern with Comments

```perl
my $email_regex = qr{
    ^                           # Start of string
    [a-zA-Z0-9._%+-]+          # Username part
    @                          # @ symbol
    [a-zA-Z0-9.-]+             # Domain name
    \.                         # Literal dot
    [a-zA-Z]{2,}               # Top-level domain
    $                          # End of string
}x;

if ($email =~ /$email_regex/) {
    print "Valid email address\n";
}
```

Regular expressions in Perl are incredibly powerful and flexible, making them essential for text processing, data validation, and string manipulation tasks.

# Input/Output (IO) in Perl

Perl provides a rich set of IO capabilities for handling different types of input and output operations. Understanding the different kinds of IO is crucial for effective Perl programming. This section covers the major IO mechanisms available in Perl, their characteristics, and when to use each type.

#### Types of IO in Perl

Perl supports several categories of IO operations:

1. **File-based IO** - Reading from and writing to files on disk
2. **Stream IO** - Working with standard input/output streams (STDIN, STDOUT, STDERR)
3. **Network IO** - Socket-based communication over networks
4. **Process IO** - Communication between processes via pipes
5. **Memory IO** - Treating strings and memory as file handles
6. **Database IO** - Database connectivity and operations
7. **Tied IO** - Custom IO behavior through tied variables
8. **Specialized IO** - Domain-specific IO mechanisms

### Standard File Handles (Stream IO)

#### STDIN, STDOUT, and STDERR

Perl provides three predefined file handles for standard input, output, and error streams. These are fundamental to Unix/Linux philosophy and enable command-line tools to work together through pipes and redirection.

**STDIN (Standard Input)**

- Default input stream (file descriptor 0)
- Used for reading data from keyboard, files, or pipes
- Can be redirected from files or other programs
- Blocking by default (waits for input)

**STDOUT (Standard Output)**

- Default output stream (file descriptor 1)
- Used for normal program output
- Can be redirected to files or piped to other programs
- Buffered by default for performance

**STDERR (Standard Error)**

- Error output stream (file descriptor 2)
- Used for error messages and diagnostic output
- Separate from STDOUT to allow error handling
- Often unbuffered for immediate error reporting

```perl
# Reading from STDIN
my $input = <STDIN>;
chomp $input;

# Writing to STDOUT (default for print)
print "Hello, World!\n";
print STDOUT "Explicit STDOUT output\n";

# Writing to STDERR
print STDERR "Error message\n";
warn "Warning message";  # Automatically goes to STDERR
```

#### File Handle Operations (File-based IO)

File handles provide the primary mechanism for file-based IO operations. They can be opened in different modes and support various reading/writing patterns.

**File Open Modes:**

- `<` - Read-only (file must exist)
- `>` - Write-only (creates new file, truncates existing)
- `>>` - Append (creates new file, appends to existing)
- `+<` - Read-write (file must exist)
- `+>` - Read-write (creates new file, truncates existing)
- `+>>` - Read-append (creates new file, appends to existing)

**Reading Patterns:**

- **Line-by-line** - Most common for text files
- **Slurp mode** - Read entire file at once
- **Fixed-length** - Read specific number of bytes
- **Binary mode** - Read raw bytes without text processing

**Writing Patterns:**

- **Line-by-line** - Write individual lines
- **Block writing** - Write large chunks of data
- **Buffered writing** - Let Perl handle buffering
- **Unbuffered writing** - Immediate output for real-time applications

```perl
# Opening files with different modes
open(my $fh, '<', 'input.txt') or die "Cannot open file: $!";
open(my $out, '>', 'output.txt') or die "Cannot create file: $!";
open(my $append, '>>', 'log.txt') or die "Cannot append to file: $!";
open(my $rw, '+<', 'data.txt') or die "Cannot open for read-write: $!";

# Different reading patterns
# Line-by-line reading (most common)
while (my $line = <$fh>) {
    chomp $line;
    print "Read: $line\n";
}

# Slurp mode - read entire file
{
    local $/ = undef;  # Disable line separator
    my $content = <$fh>;
    print "File size: " . length($content) . " bytes\n";
}

# Fixed-length reading
my $buffer;
while (read($fh, $buffer, 1024)) {
    print "Read " . length($buffer) . " bytes\n";
}

# Binary mode reading
binmode($fh);
my $binary_data;
read($fh, $binary_data, 1024);

# Writing patterns
print $out "Some data\n";  # Line-by-line

# Block writing
my $large_data = "x" x 10000;
print $out $large_data;

# Unbuffered writing for real-time output
select((select(STDOUT), $| = 1)[0]);
print $out "Immediate output\n";

# Always close file handles
close($fh);
close($out);
close($append);
close($rw);
```

### Sockets (Network IO)

Sockets provide the foundation for network communication in Perl. They enable programs to communicate over networks using various protocols and connection types.

#### Socket Types and Characteristics

**TCP Sockets (Stream Sockets)**

- **Connection-oriented** - Establishes persistent connection
- **Reliable** - Guarantees data delivery and order
- **Flow control** - Prevents overwhelming receiver
- **Error detection** - Built-in error checking and retransmission
- **Best for** - Web servers, file transfers, applications requiring data integrity

**UDP Sockets (Datagram Sockets)**

- **Connectionless** - No persistent connection
- **Unreliable** - No guarantee of delivery or order
- **Fast** - Lower overhead than TCP
- **No flow control** - Can overwhelm receiver
- **Best for** - Real-time applications, broadcasting, simple request-response

**Unix Domain Sockets**

- **Local communication** - Between processes on same machine
- **Fast** - No network overhead
- **Secure** - Cannot be accessed from network
- **File-based** - Appear as files in filesystem

#### TCP Sockets

```perl
use IO::Socket::INET;

# TCP Server
my $server = IO::Socket::INET->new(
    LocalHost => 'localhost',
    LocalPort => 8080,
    Proto     => 'tcp',
    Listen    => 5,
    Reuse     => 1
) or die "Cannot create server: $!";

print "Server listening on port 8080\n";

while (my $client = $server->accept()) {
    print $client "Hello from server!\n";
    my $data = <$client>;
    print "Received: $data";
    close($client);
}

# TCP Client
my $client = IO::Socket::INET->new(
    PeerHost => 'localhost',
    PeerPort => 8080,
    Proto    => 'tcp'
) or die "Cannot connect: $!";

print $client "Hello from client!\n";
my $response = <$client>;
print "Server says: $response";
close($client);
```

#### UDP Sockets

```perl
use IO::Socket::INET;

# UDP Server
my $udp_server = IO::Socket::INET->new(
    LocalPort => 8081,
    Proto     => 'udp'
) or die "Cannot create UDP server: $!";

my $data;
my $peer_address = recv($udp_server, $data, 1024, 0);
my ($port, $ip) = sockaddr_in($peer_address);
print "Received from " . inet_ntoa($ip) . ":$port - $data\n";

# UDP Client
my $udp_client = IO::Socket::INET->new(
    PeerHost => 'localhost',
    PeerPort => 8081,
    Proto    => 'udp'
) or die "Cannot create UDP client: $!";

send($udp_client, "UDP message", 0);
```

### Pipes (Process IO)

Pipes enable inter-process communication (IPC) and allow Perl programs to interact with external commands and other processes. They provide a way to pass data between processes without using temporary files.

#### Pipe Types and Characteristics

**Named Pipes (FIFOs)**

- **Persistent** - Exist as files in filesystem
- **Blocking** - Reader blocks until writer writes data
- **Bidirectional** - Can be opened for reading and writing
- **Process-independent** - Survive process termination
- **Best for** - Long-running communication, multiple readers/writers

**Anonymous Pipes**

- **Temporary** - Created in memory, destroyed when processes end
- **Unidirectional** - Data flows in one direction only
- **Process-dependent** - Tied to specific processes
- **Fast** - No filesystem overhead
- **Best for** - Parent-child communication, command execution

**Bidirectional Pipes**

- **Two-way communication** - Both reading and writing
- **Complex** - Requires careful synchronization
- **Process-dependent** - Tied to specific processes
- **Best for** - Interactive communication with external programs

#### Named Pipes (FIFOs)

```perl
# Create a named pipe
system("mkfifo /tmp/mypipe");

# Writer process
open(my $pipe_out, '>', '/tmp/mypipe') or die "Cannot open pipe for writing: $!";
print $pipe_out "Data through pipe\n";
close($pipe_out);

# Reader process
open(my $pipe_in, '<', '/tmp/mypipe') or die "Cannot open pipe for reading: $!";
my $data = <$pipe_in>;
print "Received: $data";
close($pipe_in);
```

#### Anonymous Pipes

```perl
# Execute command and capture output
open(my $cmd, '-|', 'ls -la') or die "Cannot execute command: $!";
while (my $line = <$cmd>) {
    print "Output: $line";
}
close($cmd);

# Send data to command
open(my $sort, '|-', 'sort') or die "Cannot pipe to sort: $!";
print $sort "zebra\n";
print $sort "apple\n";
print $sort "banana\n";
close($sort);
```

#### Bidirectional Pipes

```perl
use IPC::Open2;

# Open bidirectional pipe to external command
my ($in, $out);
my $pid = open2($in, $out, 'cat');

print $out "Hello\n";
close($out);

my $response = <$in>;
print "Response: $response";
close($in);
waitpid($pid, 0);
```

### String IO (Memory IO)

String IO allows you to treat strings and memory as file handles, enabling you to use familiar file operations on in-memory data. This is useful for parsing, data transformation, and testing without creating temporary files.

#### String IO Characteristics

**Advantages:**

- **No disk I/O** - All operations happen in memory
- **Fast** - No filesystem overhead
- **Temporary** - Data exists only during program execution
- **Flexible** - Can treat any string as a file
- **Testing** - Easy to test file operations with known data

**Use Cases:**

- **Data parsing** - Parse structured data from strings
- **Template processing** - Process templates stored in strings
- **Testing** - Test file operations with controlled input
- **Data transformation** - Transform data between formats
- **Configuration** - Process configuration data from strings

**Limitations:**

- **Memory usage** - Large strings consume memory
- **Temporary** - Data lost when program ends
- **No persistence** - Cannot save data between runs

```perl
use IO::String;

# Writing to a string
my $string_data = "";
my $string_fh = IO::String->new(\$string_data);

print $string_fh "Line 1\n";
print $string_fh "Line 2\n";
print $string_fh "Line 3\n";

print "String contains:\n$string_data";

# Reading from a string
my $input_string = "apple\nbanana\ncherry\n";
my $read_fh = IO::String->new(\$input_string);

while (my $line = <$read_fh>) {
    chomp $line;
    print "Read from string: $line\n";
}

# Alternative: Using open with scalar reference (Perl 5.8+)
my $scalar_string = "";
open(my $scalar_fh, '>', \$scalar_string) or die "Cannot open scalar: $!";
print $scalar_fh "Data written to scalar\n";
close($scalar_fh);

print "Scalar contains: $scalar_string";
```

### Ties (Custom IO)

Ties allow you to associate built-in Perl variables with custom classes, enabling special behavior for IO operations. This provides a way to create custom IO mechanisms that look and behave like standard Perl variables.

#### Tie Types and Characteristics

**Tied File Handles**

- **Custom print behavior** - Intercept print operations
- **Logging** - Automatically log all output
- **Filtering** - Transform data during I/O
- **Validation** - Check data before processing
- **Transparent** - Code using tied handles doesn't need to change

**Tied Scalars**

- **Lazy loading** - Load data only when accessed
- **Caching** - Cache expensive operations
- **Validation** - Validate data on assignment
- **Transformation** - Transform data automatically
- **Persistence** - Automatically save to files/databases

**Tied Arrays and Hashes**

- **Virtual data structures** - Data doesn't exist in memory
- **Database integration** - Automatic database operations
- **Network storage** - Store data across network
- **Compression** - Automatically compress/decompress data
- **Encryption** - Automatically encrypt/decrypt data

**Advantages:**

- **Transparent** - Existing code works without changes
- **Flexible** - Can implement any custom behavior
- **Powerful** - Full control over variable operations
- **Reusable** - Tie classes can be used in multiple contexts

**Disadvantages:**

- **Performance overhead** - Method calls for every operation
- **Complexity** - Requires understanding of tie interface
- **Debugging** - Can be difficult to debug tied behavior

#### Tied File Handles

```perl
package MyTiedHandle;

sub TIEHANDLE {
    my $class = shift;
    return bless {}, $class;
}

sub PRINT {
    my $self = shift;
    my $data = join('', @_);
    # Custom print behavior - prepend timestamp
    my $timestamp = localtime();
    print STDOUT "[$timestamp] $data";
}

sub PRINTF {
    my $self = shift;
    my $format = shift;
    $self->PRINT(sprintf($format, @_));
}

package main;

# Tie a file handle to custom class
tie *MYHANDLE, 'MyTiedHandle';

print MYHANDLE "This will be timestamped\n";
printf MYHANDLE "Number: %d\n", 42;
```

#### Tied Scalars for IO

```perl
package TiedScalar;

sub TIESCALAR {
    my $class = shift;
    my $filename = shift;
    return bless { file => $filename, data => "" }, $class;
}

sub FETCH {
    my $self = shift;
    open(my $fh, '<', $self->{file}) or return "";
    local $/;  # Slurp mode
    my $content = <$fh>;
    close($fh);
    return $content;
}

sub STORE {
    my ($self, $value) = @_;
    open(my $fh, '>', $self->{file}) or die "Cannot write: $!";
    print $fh $value;
    close($fh);
}

package main;

tie my $file_content, 'TiedScalar', 'data.txt';
$file_content = "New content";  # Writes to file
print $file_content;            # Reads from file
```

### Memory Files (Advanced Memory IO)

Memory files provide sophisticated in-memory file-like operations, allowing you to work with data in memory using familiar file operations. They're particularly useful for temporary data processing and testing.

#### Memory File Types and Characteristics

**IO::Scalar**

- **Scalar-based** - Treats a single scalar as a file
- **Read/write** - Supports both reading and writing
- **Position-aware** - Maintains file position
- **Flexible** - Can work with any scalar reference
- **Best for** - Simple string processing, temporary data

**IO::ScalarArray**

- **Array-based** - Treats an array as a file
- **Line-oriented** - Each array element becomes a line
- **Efficient** - No string concatenation overhead
- **Memory-friendly** - Good for large datasets
- **Best for** - Line-by-line processing, large text data

**IO::String**

- **Modern alternative** - Newer than IO::Scalar
- **Standard interface** - Follows standard IO interface
- **Compatible** - Works with most IO operations
- **Flexible** - Can be used in most contexts
- **Best for** - Modern Perl applications, compatibility

**Anonymous Temporary Files**

- **File-like** - Behaves exactly like real files
- **Auto-cleanup** - Automatically deleted when closed
- **Memory-backed** - Stored in memory when possible
- **Fallback** - Falls back to disk if memory insufficient
- **Best for** - Applications requiring true file semantics

```perl
use IO::Scalar;
use IO::ScalarArray;

# IO::Scalar - treat scalar as file
my $data = "Line 1\nLine 2\nLine 3\n";
my $scalar_fh = IO::Scalar->new(\$data);

while (my $line = <$scalar_fh>) {
    print "From scalar: $line";
}

# IO::ScalarArray - treat array as file
my @lines = ("First line\n", "Second line\n", "Third line\n");
my $array_fh = IO::ScalarArray->new(\@lines);

while (my $line = <$array_fh>) {
    print "From array: $line";
}

# Writing to memory
my $mem_data = "";
my $mem_fh = IO::Scalar->new(\$mem_data);
print $mem_fh "Written to memory\n";
print $mem_fh "Another line\n";

print "Memory contains:\n$mem_data";
```

#### Using Anonymous Temporary Files

```perl
use File::Temp;

# Create temporary file in memory (if supported by OS)
my $temp_fh = File::Temp->new(
    TEMPLATE => 'tempXXXXX',
    TMPDIR   => 1,
    UNLINK   => 1  # Auto-delete when done
);

print $temp_fh "Temporary data\n";
seek($temp_fh, 0, 0);  # Rewind to beginning

while (my $line = <$temp_fh>) {
    print "From temp file: $line";
}
```

### DBI Handles (Database IO)

Database handles (DBI) provide IO operations for database connectivity and data manipulation. DBI is Perl's standard database interface, providing a consistent API for working with various database systems.

#### Database IO Characteristics

**Connection Types:**

- **Direct connections** - Direct connection to database server
- **Connection pooling** - Reuse connections for efficiency
- **Persistent connections** - Long-lived connections
- **Transaction-aware** - Support for database transactions

**Data Access Patterns:**

- **Query execution** - Execute SQL statements
- **Result set processing** - Iterate through query results
- **Prepared statements** - Pre-compiled queries for efficiency
- **Batch operations** - Process multiple records efficiently
- **Stored procedures** - Execute database procedures

**Supported Database Types:**

- **Relational databases** - MySQL, PostgreSQL, Oracle, SQLite
- **NoSQL databases** - MongoDB, Redis (via specific drivers)
- **Cloud databases** - Amazon RDS, Google Cloud SQL
- **Embedded databases** - SQLite, Berkeley DB

**Advantages:**

- **Database independence** - Same code works with different databases
- **Performance** - Optimized for database operations
- **Security** - Built-in protection against SQL injection
- **Transaction support** - ACID compliance for data integrity
- **Error handling** - Comprehensive error reporting

**Common Use Cases:**

- **Web applications** - Store and retrieve application data
- **Data processing** - Transform and analyze large datasets
- **Reporting** - Generate reports from database data
- **Data migration** - Move data between systems
- **Backup and recovery** - Database maintenance operations

```perl
use DBI;

# Connect to database
my $dbh = DBI->connect(
    "DBI:SQLite:dbname=example.db",
    "",
    "",
    {
        RaiseError => 1,
        AutoCommit => 1,
        PrintError => 0
    }
) or die "Cannot connect to database: $DBI::errstr";

# Create table
$dbh->do(q{
    CREATE TABLE IF NOT EXISTS users (
        id INTEGER PRIMARY KEY,
        name VARCHAR(50),
        email VARCHAR(100)
    )
});

# Prepare and execute statements
my $sth = $dbh->prepare("INSERT INTO users (name, email) VALUES (?, ?)");
$sth->execute("John Doe", "john@example.com");
$sth->execute("Jane Smith", "jane@example.com");

# Query data
my $select_sth = $dbh->prepare("SELECT id, name, email FROM users");
$select_sth->execute();

while (my $row = $select_sth->fetchrow_hashref()) {
    print "ID: $row->{id}, Name: $row->{name}, Email: $row->{email}\n";
}

# Bind columns for efficient fetching
my ($id, $name, $email);
$select_sth->bind_columns(\$id, \$name, \$email);

$select_sth->execute();
while ($select_sth->fetch()) {
    print "Bound: $id, $name, $email\n";
}

# Transaction handling
$dbh->begin_work();
eval {
    $dbh->do("UPDATE users SET email = ? WHERE id = ?", undef, "newemail@example.com", 1);
    $dbh->do("DELETE FROM users WHERE id = ?", undef, 2);
    $dbh->commit();
};
if ($@) {
    $dbh->rollback();
    die "Transaction failed: $@";
}

# Clean up
$sth->finish();
$select_sth->finish();
$dbh->disconnect();
```

#### Advanced DBI Features

```perl
# Using placeholders and bind parameters
my $insert_sth = $dbh->prepare_cached(q{
    INSERT INTO users (name, email) VALUES (?, ?)
});

# Batch processing
my @users = (
    ["Alice Johnson", "alice@example.com"],
    ["Bob Wilson", "bob@example.com"],
    ["Carol Brown", "carol@example.com"]
);

for my $user (@users) {
    $insert_sth->execute(@$user);
}

# Fetch data in different ways
my $all_users = $dbh->selectall_arrayref(
    "SELECT * FROM users",
    { Slice => {} }  # Return array of hashrefs
);

for my $user (@$all_users) {
    print "User: $user->{name} ($user->{email})\n";
}

# Single row/value fetching
my $user_count = $dbh->selectrow_array("SELECT COUNT(*) FROM users");
print "Total users: $user_count\n";

my $first_user = $dbh->selectrow_hashref("SELECT * FROM users LIMIT 1");
print "First user: $first_user->{name}\n" if $first_user;
```

### Best Practices and Error Handling

#### Error Handling

```perl
# Always check return values
open(my $fh, '<', 'file.txt') or die "Cannot open file: $!";

# Use autodie for automatic error checking
use autodie;
open(my $fh, '<', 'file.txt');  # Dies automatically on failure

# Proper error handling for network operations
use Try::Tiny;

try {
    my $socket = IO::Socket::INET->new(
        PeerHost => 'example.com',
        PeerPort => 80,
        Timeout  => 10
    ) or die "Connection failed: $!";

    # Use the socket
    close($socket);
} catch {
    warn "Network error: $_";
};
```

#### Resource Management

```perl
# Use lexical file handles for automatic cleanup
{
    open(my $fh, '<', 'file.txt') or die "Cannot open: $!";
    # File handle automatically closed when $fh goes out of scope
}

# Explicit cleanup for critical resources
my $dbh = DBI->connect(...);
END { $dbh->disconnect() if $dbh; }
```

#### Performance Considerations

```perl
# Use appropriate buffering
select((select(STDOUT), $| = 1)[0]);  # Unbuffer STDOUT

# Read large files efficiently
while (my $line = <$fh>) {
    # Process line by line instead of slurping entire file
}

# Use prepared statements for repeated database operations
my $sth = $dbh->prepare("SELECT * FROM table WHERE id = ?");
for my $id (@ids) {
    $sth->execute($id);
    # Process results
}
```

### Specialized IO Types

Beyond the core IO mechanisms, Perl supports various specialized IO types for specific domains and use cases.

#### Serial/Device IO

```perl
use Device::SerialPort;

# Serial port communication
my $port = Device::SerialPort->new("/dev/ttyUSB0");
$port->baudrate(9600);
$port->databits(8);
$port->parity("none");
$port->stopbits(1);

# Read from serial device
my $data = $port->read(1024);
print "Received: $data\n";

# Write to serial device
$port->write("ATZ\n");
```

#### HTTP/Web IO

```perl
use LWP::UserAgent;
use HTTP::Request;

# HTTP client operations
my $ua = LWP::UserAgent->new;
my $request = HTTP::Request->new(GET => 'http://example.com');
my $response = $ua->request($request);

if ($response->is_success) {
    print "Content: " . $response->content . "\n";
} else {
    print "Error: " . $response->status_line . "\n";
}
```

#### XML/JSON IO

```perl
use XML::Simple;
use JSON;

# XML processing
my $xml_data = '<root><item>value</item></root>';
my $xml_ref = XMLin($xml_data);
print "XML item: $xml_ref->{item}\n";

# JSON processing
my $json_data = '{"name": "John", "age": 30}';
my $json_ref = decode_json($json_data);
print "JSON name: $json_ref->{name}\n";
```

#### Compressed File IO

```perl
use IO::Compress::Gzip;
use IO::Uncompress::Gunzip;

# Writing compressed data
my $compressed_data;
open my $gz, '>', \$compressed_data;
my $gzip = IO::Compress::Gzip->new($gz);
print $gzip "Data to compress\n";
$gzip->close();

# Reading compressed data
my $gunzip = IO::Uncompress::Gunzip->new(\$compressed_data);
while (my $line = <$gunzip>) {
    print "Decompressed: $line";
}
```

### IO Selection Guidelines

#### When to Use Each IO Type

**File-based IO:**

- ✅ Reading/writing files on disk
- ✅ Logging and data persistence
- ✅ Configuration files
- ❌ Temporary data processing
- ❌ High-frequency operations

**Stream IO:**

- ✅ Command-line tools and scripts
- ✅ Interactive programs
- ✅ Piping between programs
- ❌ Binary data processing
- ❌ Large file operations

**Network IO:**

- ✅ Client-server applications
- ✅ Web services and APIs
- ✅ Distributed systems
- ❌ Local-only applications
- ❌ Simple file operations

**Process IO:**

- ✅ Executing external commands
- ✅ Inter-process communication
- ✅ Command-line tool integration
- ❌ High-performance applications
- ❌ Real-time data processing

**Memory IO:**

- ✅ Temporary data processing
- ✅ Testing and debugging
- ✅ Data transformation
- ❌ Large datasets
- ❌ Persistent storage

**Database IO:**

- ✅ Structured data storage
- ✅ Multi-user applications
- ✅ Complex queries and relationships
- ❌ Simple file operations
- ❌ Temporary data

**Tied IO:**

- ✅ Custom data processing
- ✅ Transparent data transformation
- ✅ Logging and monitoring
- ❌ High-performance applications
- ❌ Simple operations

### Conclusion

Perl's IO system is comprehensive and flexible, supporting everything from simple file operations to complex network programming and database interactions. Understanding the different types of IO and their characteristics is crucial for choosing the right tool for each task.

#### Key Principles for Effective IO:

1. **Choose the right IO type** for your specific use case
2. **Understand the characteristics** of each IO mechanism
3. **Implement proper error handling** and resource management
4. **Leverage Perl's powerful features** like ties for custom behavior
5. **Use modern modules** and best practices for robust applications
6. **Consider performance implications** of your IO choices
7. **Plan for scalability** when designing IO-intensive applications

#### Best Practices Summary:

- **File operations**: Use lexical file handles and proper error checking
- **Network operations**: Implement timeouts and connection pooling
- **Database operations**: Use prepared statements and transactions
- **Memory operations**: Be mindful of memory usage for large datasets
- **Custom IO**: Use ties sparingly and document behavior clearly

Whether you're working with files, networks, databases, or custom IO mechanisms, Perl provides the tools needed for efficient and reliable input/output operations. The key is understanding the strengths and limitations of each IO type and applying them appropriately to your specific requirements.

## Command Line Perl: One-liners

### Perl as sed Equivalent

Perl's `-ple` flags make it a powerful command-line text processor, often replacing `sed` and `awk`:

#### **Basic One-liner Structure**

```bash
perl -ple 's/old/new/g' file.txt
```

#### **Flag Explanations**

- **`-p`**: Creates a loop that reads each line, processes it, and prints it
- **`-l`**: Automatically chomps newlines and adds them back on print
- **`-e`**: Executes the following string as Perl code

#### **Common One-liner Patterns**

**Text Replacement (sed equivalent):**

```bash
# Replace 'old' with 'new' globally
perl -ple 's/old/new/g' file.txt

# Case-insensitive replacement
perl -ple 's/old/new/gi' file.txt

# Replace only on lines containing 'pattern'
perl -ple 's/old/new/g if /pattern/' file.txt
```

**Line Processing:**

```bash
# Print only lines containing 'error'
perl -ple 'print if /error/i' file.txt

# Print line numbers
perl -ple 'print "$.: $_"' file.txt

# Remove empty lines
perl -ple 'print unless /^\s*$/' file.txt
```

**Field Processing (awk equivalent):**

```bash
# Print first field of each line
perl -ple 'print (split /\s+/)[0]' file.txt

# Print lines where field 3 > 100
perl -ple 'print if (split /\s+/)[2] > 100' file.txt

# Sum field 2
perl -lane '$sum += $F[1]; END { print $sum }' file.txt
```

**Advanced Examples:**

```bash
# Convert Windows line endings to Unix
perl -ple 's/\r\n/\n/g' file.txt

# Extract email addresses
perl -ple 'print $& while /\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b/g' file.txt

# JSON pretty print
perl -ple 'print JSON::PP->new->pretty->encode(decode_json($_))' file.json
```

## Best Practices: use strict and use warnings

### Why Use These Pragmas?

These are essential for writing robust, maintainable Perl code:

#### **use strict;**

- **Catches common errors** at compile time
- **Requires variable declaration** with `my`, `our`, or `state`
- **Prevents typos** in variable names
- **Enforces better coding practices**

```perl
use strict;

# This will cause an error:
$undeclared_var = "value";  # Global symbol "$undeclared_var" requires explicit package name

# This is correct:
my $declared_var = "value";
```

#### **use warnings;**

- **Catches runtime warnings** about potential problems
- **Helps identify bugs** before they cause serious issues
- **Warns about deprecated features**

```perl
use warnings;

# These will generate warnings:
my $result = "5" + "3";     # Argument "3" isn't numeric in addition
my $undefined;              # Use of uninitialized value
print $undefined;           # Use of uninitialized value in print
```

#### **Complete Example**

```perl
#!/usr/bin/perl
use strict;
use warnings;

# All variables must be declared
my $name = "John";
my @items = qw(apple banana cherry);
my %config = (debug => 1, verbose => 0);

# Function with proper parameter handling
sub process_item {
    my ($item, $count) = @_;

    # Warnings will catch undefined values
    return "$count x $item";
}

# This would cause errors without strict/warnings
print process_item($items[0], 5), "\n";
```

## Web and HTTP Programming

### LWP::Simple - Lightweight HTTP Client

`LWP::Simple` provides easy HTTP operations for simple web requests:

#### **Basic Usage**

```perl
use LWP::Simple;

# Simple GET request
my $content = get("http://example.com");
if (defined $content) {
    print "Content length: " . length($content) . "\n";
} else {
    print "Failed to get content\n";
}

# Download a file
my $status = getstore("http://example.com/file.txt", "local_file.txt");
if (is_success($status)) {
    print "Download successful\n";
} else {
    print "Download failed: $status\n";
}
```

#### **Common Functions**

```perl
use LWP::Simple;

# Check if URL exists
if (head("http://example.com")) {
    print "URL exists\n";
}

# Get content with timeout
my $content = get("http://example.com");
die "Couldn't get content" unless defined $content;

# Mirror a file (download if newer)
my $status = mirror("http://example.com/file.txt", "local_file.txt");
```

### HTTP::Tiny - Modern HTTP Client (Perl 5.10+)

`HTTP::Tiny` is included with Perl 5.10 and later, providing a lightweight HTTP client:

#### **Basic Usage**

```perl
use HTTP::Tiny;

my $http = HTTP::Tiny->new;

# GET request
my $response = $http->get('http://example.com');
if ($response->{success}) {
    print "Status: $response->{status}\n";
    print "Content: $response->{content}\n";
} else {
    print "Error: $response->{reason}\n";
}
```

#### **Advanced Features**

```perl
use HTTP::Tiny;

my $http = HTTP::Tiny->new(
    timeout => 10,
    max_redirect => 5,
    default_headers => {
        'User-Agent' => 'MyApp/1.0'
    }
);

# POST request
my $response = $http->post_form(
    'http://example.com/submit',
    { name => 'John', age => 30 }
);

# Custom headers
my $response = $http->get(
    'http://example.com/api/data',
    { headers => { 'Authorization' => 'Bearer token123' } }
);
```

### CGI Programming

Perl scripts are excellent for CGI (Common Gateway Interface) web applications:

#### **Basic CGI Script**

```perl
#!/usr/bin/perl
use strict;
use warnings;
use CGI;

my $cgi = CGI->new;

# Send HTTP headers
print $cgi->header('text/html');

# Get form parameters
my $name = $cgi->param('name') || 'Guest';
my $email = $cgi->param('email');

# Generate HTML
print $cgi->start_html('My CGI Script');
print $cgi->h1("Hello, $name!");
print $cgi->start_form(-method => 'POST');
print "Name: ", $cgi->textfield('name'), $cgi->br;
print "Email: ", $cgi->textfield('email'), $cgi->br;
print $cgi->submit('Submit');
print $cgi->end_form;
print $cgi->end_html;
```

#### **CGI Best Practices**

```perl
#!/usr/bin/perl
use strict;
use warnings;
use CGI qw(:standard);
use CGI::Carp qw(fatalsToBrowser);

# Error handling
eval {
    # Your CGI code here
    my $name = param('name');
    print header, start_html;
    print h1("Hello, $name!");
    print end_html;
};
if ($@) {
    print header(-status => '500 Internal Server Error');
    print "An error occurred: $@";
}
```

#### **CGI Security Considerations**

```perl
use CGI;
use HTML::Entities;

my $cgi = CGI->new;

# Sanitize user input
my $user_input = $cgi->param('user_input');
$user_input = encode_entities($user_input);  # Prevent XSS

# Validate file uploads
my $upload = $cgi->upload('file');
if ($upload) {
    my $filename = $cgi->param('file');
    # Validate file type and size
    die "Invalid file type" unless $filename =~ /\.(txt|pdf)$/i;
}
```

## Command Line Argument Processing

### Getopt::Long - Advanced Command Line Options

`Getopt::Long` provides sophisticated command-line argument parsing:

#### **Basic Usage**

```perl
use Getopt::Long;
use strict;
use warnings;

# Define variables for options
my $verbose = 0;
my $output_file = '';
my $help = 0;
my @input_files = ();

# Parse command line options
GetOptions(
    'verbose|v' => \$verbose,
    'output|o=s' => \$output_file,
    'help|h' => \$help,
    'input=s@' => \@input_files
) or die "Usage: $0 [options]\n";

# Handle help
if ($help) {
    print "Usage: $0 [options]\n";
    print "Options:\n";
    print "  -v, --verbose    Enable verbose output\n";
    print "  -o, --output     Output file\n";
    print "  -h, --help       Show this help\n";
    print "  --input          Input file (can be specified multiple times)\n";
    exit 0;
}

# Use the options
print "Verbose mode: " . ($verbose ? "on" : "off") . "\n";
print "Output file: $output_file\n" if $output_file;
print "Input files: " . join(', ', @input_files) . "\n";
```

#### **Advanced Features**

```perl
use Getopt::Long;
use Pod::Usage;

# Complex option handling
my %config = (
    verbose => 0,
    debug => 0,
    timeout => 30,
    retries => 3
);

my @files = ();
my $mode = 'default';

GetOptions(
    'verbose|v' => \$config{verbose},
    'debug|d' => \$config{debug},
    'timeout|t=i' => \$config{timeout},
    'retries|r=i' => \$config{retries},
    'mode|m=s' => \$mode,
    'file|f=s@' => \@files,
    'help|h' => sub { pod2usage(1) },
    'version' => sub { print "Version 1.0\n"; exit 0 }
) or pod2usage(2);

# Validate options
die "Timeout must be positive" if $config{timeout} <= 0;
die "Retries must be between 0 and 10" unless $config{retries} >= 0 && $config{retries} <= 10;
die "Mode must be 'default', 'fast', or 'safe'" unless $mode =~ /^(default|fast|safe)$/;
```

#### **Option Types**

```perl
use Getopt::Long;

my ($flag, $string, $number, $float, @list, %hash);

GetOptions(
    'flag|f' => \$flag,           # Boolean flag
    'string|s=s' => \$string,     # String value
    'number|n=i' => \$number,     # Integer value
    'float|r=f' => \$float,       # Float value
    'list|l=s@' => \@list,        # Array of strings
    'hash|h=s%' => \%hash         # Hash of key=value pairs
);

# Usage examples:
# script.pl --flag --string "value" --number 42 --float 3.14
# script.pl --list file1.txt --list file2.txt
# script.pl --hash key1=value1 --hash key2=value2
```

#### **Complete Example**

```perl
#!/usr/bin/perl
use strict;
use warnings;
use Getopt::Long;
use File::Basename;

# Script configuration
my %config = (
    verbose => 0,
    dry_run => 0,
    backup => 1,
    max_files => 100
);

my @files = ();
my $output_dir = '.';

# Parse command line
GetOptions(
    'verbose|v' => \$config{verbose},
    'dry-run|n' => \$config{dry_run},
    'no-backup' => sub { $config{backup} = 0 },
    'max-files=i' => \$config{max_files},
    'output|o=s' => \$output_dir,
    'file|f=s@' => \@files,
    'help|h' => \&show_help
) or show_help();

# Validate
die "No input files specified\n" unless @files;
die "Output directory '$output_dir' does not exist\n" unless -d $output_dir;

# Process files
for my $file (@files) {
    if ($config{verbose}) {
        print "Processing: $file\n";
    }

    if ($config{dry_run}) {
        print "Would process: $file\n";
        next;
    }

    # Actual processing here
    process_file($file, $output_dir, \%config);
}

sub show_help {
    my $script = basename($0);
    print <<"EOF";
Usage: $script [options] --file file1 --file file2

Options:
    -v, --verbose      Enable verbose output
    -n, --dry-run      Show what would be done without doing it
    --no-backup        Don't create backup files
    --max-files=N      Maximum number of files to process (default: 100)
    -o, --output=DIR   Output directory (default: current directory)
    -f, --file=FILE    Input file (can be specified multiple times)
    -h, --help         Show this help message

Examples:
    $script -v --file input1.txt --file input2.txt
    $script --dry-run --max-files 50 --output /tmp/processed
EOF
    exit 0;
}

sub process_file {
    my ($file, $output_dir, $config) = @_;
    # File processing logic here
    print "Processing $file to $output_dir\n" if $config->{verbose};
}
```

These additions provide comprehensive coverage of essential Perl topics including variable types, functions, command-line usage, web programming, and argument processing. Each section includes practical examples and best practices for effective Perl development.
