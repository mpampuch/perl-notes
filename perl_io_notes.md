# Input/Output (IO) in Perl

Perl provides a rich set of IO capabilities for handling different types of input and output operations. This note covers the major IO mechanisms available in Perl.

## Standard File Handles

### STDIN, STDOUT, and STDERR

Perl provides three predefined file handles for standard input, output, and error streams:

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

### File Handle Operations

```perl
# Opening files
open(my $fh, '<', 'input.txt') or die "Cannot open file: $!";
open(my $out, '>', 'output.txt') or die "Cannot create file: $!";
open(my $append, '>>', 'log.txt') or die "Cannot append to file: $!";

# Reading from file
while (my $line = <$fh>) {
    chomp $line;
    print "Read: $line\n";
}

# Writing to file
print $out "Some data\n";

# Always close file handles
close($fh);
close($out);
close($append);
```

## Sockets

Perl supports both client and server socket programming for network communication.

### TCP Sockets

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

### UDP Sockets

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

## Pipes

Pipes allow communication between processes or execution of external commands.

### Named Pipes (FIFOs)

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

### Anonymous Pipes

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

### Bidirectional Pipes

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

## String IO

String IO allows you to treat strings as file handles for reading and writing operations.

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

## Ties

Ties allow you to associate built-in Perl variables with custom classes, enabling special behavior for IO operations.

### Tied File Handles

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

### Tied Scalars for IO

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

## Memory Files

Memory files allow you to work with in-memory data as if it were a file.

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

### Using Anonymous Temporary Files

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

## DBI Handles

Database handles (DBI) provide IO operations for database connectivity and data manipulation.

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

### Advanced DBI Features

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

## Best Practices and Error Handling

### Error Handling

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

### Resource Management

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

### Performance Considerations

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

## Conclusion

Perl's IO system is comprehensive and flexible, supporting everything from simple file operations to complex network programming and database interactions. The key to effective IO in Perl is:

1. Understanding the appropriate tool for each task
2. Proper error handling and resource management
3. Leveraging Perl's powerful features like ties for custom behavior
4. Using modern modules and best practices for robust applications

Whether you're working with files, networks, databases, or custom IO mechanisms, Perl provides the tools needed for efficient and reliable input/output operations.