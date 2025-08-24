# Perl Regular Expressions - Comprehensive Notes

## Overview
Perl is renowned for its powerful regular expression capabilities. Regular expressions (regex) in Perl are patterns used to match character combinations in strings. They are fundamental to text processing and pattern matching in Perl.

## Basic Syntax

### Pattern Matching Operator
```perl
if ($string =~ /pattern/) { ... }      # Matches pattern in $string
if ($string !~ /pattern/) { ... }     # Does NOT match pattern in $string
```

### Basic Pattern Delimiters
```perl
/pattern/         # Standard delimiter
m/pattern/        # Explicit match operator
m!pattern!        # Alternative delimiter
m{pattern}        # Balanced delimiters
m(pattern)        # Parentheses as delimiter
```

## Core Operations

### 1. Match
```perl
if ($string =~ /pattern/) {
    print "Match found!\n";
}
```

### 2. Match with Capture
```perl
if ($string =~ /(\d+)-(\d+)-(\d+)/) {
    my ($year, $month, $day) = ($1, $2, $3);
    print "Date: $month/$day/$year\n";
}
```

### 3. Substitute
```perl
$string =~ s/pattern/replacement/;      # Replace first occurrence
$string =~ s/pattern/replacement/g;     # Replace all occurrences (global)
```

### 4. Global Substitute
```perl
$string =~ s/old/new/g;                 # Replace all occurrences
$string =~ s/pattern/replacement/gi;    # Case-insensitive global replace
```

### 5. Substitute with Modifiers
```perl
$string =~ s/pattern/replacement/gix;   # Global, case-insensitive, extended
```

### 6. Match and Modify
```perl
$string =~ s/(pattern)/$1 modified/;    # Capture and modify
```

### 7. Global Match
```perl
my @matches = ($string =~ /pattern/g);  # Find all matches
while ($string =~ /pattern/g) {         # Process each match
    print "Found: $&\n";
}
```

### 8. Split
```perl
my @array = split /pattern/, $string;   # Split string using pattern as delimiter
```

### 9. Positive Lookahead
```perl
if ($string =~ /pattern(?=lookahead)/) { ... }  # Matches pattern only if followed by lookahead
```

### 10. Negative Lookahead
```perl
if ($string =~ /pattern(?!lookahead)/) { ... }  # Matches pattern only if NOT followed by lookahead
```

### 11. Positive Lookbehind
```perl
if ($string =~ /(?<=lookbehind)pattern/) { ... } # Matches pattern only if preceded by lookbehind
```

### 12. Negative Lookbehind
```perl
if ($string =~ /(?<!lookbehind)pattern/) { ... } # Matches pattern only if NOT preceded by lookbehind
```

### 13. Find and Print
```perl
while ($string =~ /pattern/g) {
    print "Match: $&\n";               # $& contains the matched text
}
```

### 14. Replace in File
```perl
perl -pe 's/pattern/replacement/g' file.txt    # One-liner to replace in file
```

## Regular Expression Modifiers

### Common Modifiers
- `i` - Case insensitive
- `g` - Global (find/replace all occurrences)
- `m` - Multiline mode (^ and $ match line boundaries)
- `s` - Single line mode (. matches newlines)
- `x` - Extended mode (ignore whitespace, allow comments)
- `o` - Compile once (for efficiency with variables)

### Examples with Modifiers
```perl
$string =~ /pattern/i;          # Case insensitive
$string =~ /pattern/g;          # Global match
$string =~ /pattern/gi;         # Case insensitive + global
$string =~ /pattern/gix;        # Case insensitive + global + extended
```

## Character Classes and Metacharacters

### Basic Character Classes
```perl
.           # Any character except newline
\d          # Digit [0-9]
\D          # Non-digit [^0-9]
\w          # Word character [a-zA-Z0-9_]
\W          # Non-word character [^a-zA-Z0-9_]
\s          # Whitespace [ \t\n\r\f]
\S          # Non-whitespace [^ \t\n\r\f]
```

### Custom Character Classes
```perl
[abc]       # Match a, b, or c
[a-z]       # Match any lowercase letter
[A-Z]       # Match any uppercase letter
[0-9]       # Match any digit
[^abc]      # Match anything except a, b, or c
[a-zA-Z0-9] # Match alphanumeric characters
```

### Quantifiers
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

### Anchors
```perl
^           # Beginning of string/line
$           # End of string/line
\b          # Word boundary
\B          # Non-word boundary
\A          # Beginning of string only
\Z          # End of string only
\z          # Absolute end of string
```

## Capture Groups

### Basic Capturing
```perl
if ($string =~ /(\d+)-(\d+)-(\d+)/) {
    print "Year: $1, Month: $2, Day: $3\n";
}
```

### Named Captures (Perl 5.10+)
```perl
if ($string =~ /(?<year>\d+)-(?<month>\d+)-(?<day>\d+)/) {
    print "Year: $+{year}, Month: $+{month}, Day: $+{day}\n";
}
```

### Non-capturing Groups
```perl
/(?:pattern)/       # Non-capturing group
```

## Advanced Features

### Backreferences
```perl
/(\w+)\s+\1/        # Match repeated words (e.g., "the the")
```

### Recursive Patterns (Perl 5.10+)
```perl
/\( (?: [^()]+ | (?R) )* \)/x    # Match balanced parentheses
```

### Conditional Patterns
```perl
/(condition)(?(1)then|else)/     # If condition matches, use "then", else "else"
```

## Practical Examples

### Email Validation
```perl
if ($email =~ /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/) {
    print "Valid email\n";
}
```

### Phone Number Extraction
```perl
my @phones = ($text =~ /\b\d{3}-\d{3}-\d{4}\b/g);
```

### URL Parsing
```perl
if ($url =~ m{^(https?)://([^/]+)(/.*)$}) {
    my ($protocol, $host, $path) = ($1, $2, $3);
    print "Protocol: $protocol, Host: $host, Path: $path\n";
}
```

### Log File Processing
```perl
while (<LOG>) {
    if (/^(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) \[(\w+)\] (.+)$/) {
        my ($date, $time, $level, $message) = ($1, $2, $3, $4);
        print "[$level] $date $time: $message\n" if $level eq 'ERROR';
    }
}
```

### Text Cleaning
```perl
$text =~ s/^\s+|\s+$//g;        # Trim whitespace
$text =~ s/\s+/ /g;             # Normalize whitespace
$text =~ s/[^\w\s]//g;          # Remove non-word characters
```

## Special Variables

### Match Variables
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

### Example Usage
```perl
if ($string =~ /(perl) (regex)/) {
    print "Before: '$`'\n";     # Text before match
    print "Match: '$&'\n";      # The full match
    print "After: '$''\n";      # Text after match
    print "Group 1: '$1'\n";    # First capture group
    print "Group 2: '$2'\n";    # Second capture group
}
```

## Performance Tips

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

## Common Pitfalls

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

## Best Practices

1. Use meaningful variable names for captures
2. Comment complex regex patterns using the `x` modifier
3. Test regex patterns thoroughly with edge cases
4. Consider using `qr//` for reusable patterns
5. Use appropriate anchors to avoid unintended matches
6. Validate user input with regex carefully to avoid security issues

## Example: Complex Pattern with Comments
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