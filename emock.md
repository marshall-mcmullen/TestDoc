# Module emock

`emock` is a [mocking](https://en.wikipedia.org/wiki/Mock_object) framework for bash. It integrates well into the rest
of ebash and etest in particular to make it very easy to mock out real system binaries and replace them with mock
instances which do something else. It is very flexible in the behavior of the mocked function utilizing the many
provided option flags.

If you aren't familiar with mocking, it is by far one of the most powerful test strategies and is not not limited to
just higher-level object-oriented languages. It’s actually really powerful for low-level OS testing as well where you
basically want to test just your code and not the entire OS. The typical strategy here is to essentially create a mock
function or script which gets called instead of the real OS level
component.

In bash you could simply create a function in order to mock out external binaries. But doing this manually all over the
place is tedious and error-prone and not very feature rich. `emock` makes this both easier and far more powerful.

The simplest invocation of `emock` is to simply supply the name of the binary you wish to mock, such as:

```shell
emock "dmidecode"
```

This will create and export a new function called `dmidecode` that can be invoked instead of the real `dmidecode`
binary at `/usr/bin/dmidecode`. This also creates a function named `dmidecode_real` which can be invoked to get access
to the real underlying dmidecode binary at `/usr/sbin/dmidecode`.

By default, this mock function will simply return `0` and produce no stdout or stderr. This behavior can be customized
via `--return-code`, `--stdout`, and `--stderr`.

Mocking a real binary with a simplex name like this is the simplest, but doesn't always work. In particular, if at the
call site you call it with the fully-qualified path to the binary, as in `/usr/sbin/dmidecode`, then our mocked function
won't be called. In this scenario, you need to  mock it with the fully qualified path just as you would invoke it at the
call site. For example:

```shell
emock "/usr/sbin/dmidecode"
```

Just as before, this will create and export a new function named `/usr/sbin/dmidecode` (yes, function names in bash CAN
have slashes in them!!) which will be called in place of the real `dmidecode` binary. It will also create a new
function `/usr/sbin/dmidecode_real` which will call the real binary in case you need to call it instead.

`emock` tracks various metadata about mocked binaries for easier testability. This includes the number of times a mock is
called, as well as the arguments, return code, stdout, and stderr for each invocation. By default this is created in a
local hidden directory named '.emock-$$' (where $$ is the current process PID) and there will be a directory beneath
that for each mock:

```shell
.emock-$$/dmidecode/called
.emock-$$/dmidecode/0/{args,return_code,stdout,stderr,timestamp}
.emock-$$/dmidecode/1/{args,return_code,stdout,stderr,timestamp}
```

## func assert_emock_called

`assert_emock_called` is used to assert that a mock is called the expected number of times. For example:

```shell
assert_emock_called "func" 25
```

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   times
         Number of times we expect the mock to have been called.

```

## func assert_emock_called_with

`assert_emock_called_with` is used to assert that a particular invocation of a mock was called with the expected
arguments. All arguments are fully quoted to ensure whitepace is properly perserved.

For example:

```shell
assert_emock_called_with "func" 0 "1" "2" "3" "docks and cats" "Anarchy"
expected=( "1" "2" "3" "dogs and cats" "Anarchy" )
assert_emock_caleld_with "func" 1 "${expected[@]}"
```

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to look at the arguments for.

   args
         Argument array we expect the mock function to have been called with.
```

## func assert_emock_return_code

`assert_emock_return_code` is used to assert that a particular invocation of a mock produced the expected return code.

For example:

```shell
assert_emock_return_code "func" 0 0
assert_emock_return_code "func" 0 1
```

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to look at the arguments for.

   return_code
         The expected return code.

```

## func assert_emock_stderr

`assert_emock_stderr` is used to assert that a particular invocation of a mock produced the expected standard error.

For example:

```shell
assert_emock_stderr "func" 0 "This is the expected standard error for call #0"
assert_emock_stderr "func" 1 "This is the expected standard error for call #1"
```

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to look at the arguments for.

   stderr
         The expected stdandard error.

```

## func assert_emock_stdout

`assert_emock_stdout` is used to assert that a particular invocation of a mock produced the expected standard output.

For example:

```shell
assert_emock_stdout "func" 0 "This is the expected standard output for call #0"
assert_emock_stdout "func" 1 "This is the expected standard output for call #1"
```

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to look at the arguments for.

   stdout
         The expected stdandard output.

```

## func emock

`emock` is used to mock out a real function or external binary with a fake instance which we control the return code,
stdandard output, and standard error.

The simplest invocation of `emock` is to simply supply the name of the binary you wish to mock, such as:

```shell
emock "dmidecode"
```

This will create and export a new function called `dmidecode` that can be invoked instead of the real `dmidecode`
binary at `/usr/bin/dmidecode`. This also creates a function named `dmidecode_real` which can be invoked to get access
to the real underlying dmidecode binary at `/usr/sbin/dmidecode`.

By default, this mock function will simply return `0` and produce no stdout or stderr. This behavior can be customized
via `--return-code`, `--stdout`, and `--stderr`.

You can also mock out binaries that are invoked using fully-qualified paths, as in:

```shell
emock "/usr/sbin/dmidecode"
```

Just as before, this will create and export a new function named `/usr/sbin/dmidecode` (yes, function names in bash CAN
have slashes in them!!) which will be called in place of the real `dmidecode` binary. It will also create a new
function `/usr/sbin/dmidecode_real` which will call the real binary in case you need to call it instead.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --delete, -d
         Delete existing mock state inside statedir from prior mock invocations.

   --return-code, --rc, -r <value>
         What return code should the mock script use. By default this is 0.

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the return code, stdout, and stderr for each invocation.

   --stderr, -e <value>
         What standard error should be returned by the mock.

   --stdout, -o <value>
         What standard output should be returned by the mock.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode). This must match the calling convention at the call site.

   body
         This allows fine-grained control over the body of the mocked function that is created. Instead of using return_code, stdout, and stderr, you
         can directly provide the entire body of the script in this string. The syntax of this is single quotes with enclosing curly braces. This is
         identical to what you would use with override_function.

```

## func emock_args

`emock_args` is a utility function to make it easier to get the argument array from a particular invocation of a mocked
function. This is stored on-disk and is easy to manually retrieve, but this function should always be used to provide a
clean abstraction. If the call number is not provided, this will default to the most recent invocation's argument array.

Inside the statedir, the argument array is stored with each argument fully quoted so that whitespace encapsulated
arguments preserve whitespace. To convert this back into an array, the best thing to do is to use array_init:

```shell
array_init args "$(emock_args func)"
```

Alternatively, the helper `assert_emock_called_with` is an extremely useful way to validate the arguments passed
into a particular invocation.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to get the standard error for.

```

## func emock_called

`emock_called` makes it easier to check how many times a mock has been called. This is tracked on-disk in the statedir
in the file named `called`. While it's easy to manually retrieve from this file, this function should always be used to
provide a clean abstraction.

Just like a typical array in any language, the size, or count of the number of times that the mock has been called is
1-based but the actual index values we use to store the state files for each invocation is zero-based (again, just like
an array).

So if this has never been called, then `emock_called` will echo `0`, and there will be no on-disk state directory. The
first time you call it, `emock_called` will echo `1`, and there will be a `${statedir}/0` directory storing the state
files for that invocation.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

```

## func emock_return_code

`emock_return_code` is a utility function to make it easier to get the return code from a particular invocation of a
mocked function. This is stored on-disk and is easy to manually retrieve, but this function should always be used to
provide a clean abstraction. If the call number is not provided, this will default to the most recent invocation's
return code.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to get the standard error for.

```

## func emock_stderr

`emock_stderr` is a utility function to make it easier to get the standard error from a particular invocation of a
mocked function. This is stored on-disk and is easy to manually retrieve, but this function should always be used to
provide a clean abstraction. If the call number is not provided, this will default to the most recent invocation's
standard error.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to get the standard error for.

```

## func emock_stdout

`emock_stdout` is a utility function to make it easier to get the standard output from a particular invocation of a
mocked function. This is stored on-disk and is easy to manually retrieve, but this function should always be used to
provide a clean abstraction. If the call number is not provided, this will default to the most recent invocation's
standard output.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --statedir <value>
         This directory is used to track state about mocked binaries. This will hold metadata information such as the number of times the mock was
         called as well as the exit code, stdout, and stderr for each invocation.


ARGUMENTS

   name
         Name of the binary to mock (e.g. dmidecode or /usr/sbin/dmidecode).

   num
         The call number to get the standard output for.

```
