# Module assert


## func assert

Executes a command (simply type the command after assert as if you were running it without assert) and calls die if
that command returns a bad exit code.

For example:
    assert test 0 -eq 1

There's a subtlety here that I don't think can easily be fixed given bash's semantics.  All of the arguments get
evaluated prior to assert ever seeing them.  So it doesn't know what variables you passed in to an expression, just
what the expression was.  This is pretty handy in cases like this one:

    a=1 b=2 assert test "${a}" -eq "${b}"

because assert will tell you that the command that it executed was

    test 1 -eq 2

There it seems ideal.  But if you have an empty variable, things get a bit annoying.  For instance, this command will
blow up because inside assert bash will try to evaluate [[ -z ]] without any arguments to -z.  (Note -- it still blows
up, just not in quite the way you'd expect)

    empty="" assert test -z ${empty}

To make this particular case easier to deal with, we also have assert_empty which you could use like this:

    assert_empty empty


IMPLEMENTATION NOTE: This doesn't work with bash double-bracket expressions.  Use test instead.  To the best of my
(Odell) understanding (as of 2016-07-01) bash must treat [[ ]] syntax specially.  The problem is that we have two
options for trying to run what is passed in to assert.

    1) Pass it through eval, which will drop one more layer of quoting that we'd normally expect (because it already
       dropped 1 layer of quoting when assert was called)
    2) Just execute the command as something inside an array.  For instance, we could run it like this:
           "${cmd[@]"
       It works great for most commands, but blows up complaining that [[ isn't known as a valid command.  Perhaps you
       can't run builtins in this manner?  Or at the very least you can't run [[.

## func assert_archive_contents

Assert that the provided archive contains the expected content. If there are any additional files in the archive not
specified on the list of expected files then this assertion will fail.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --type, -t <value>
         Override automatic type detection and use explicit archive type.


ARGUMENTS

   archive
         Archive whose contents should be listed.

```

## func assert_docker_image_exists

This function asserts that a docker image exists locally.

```Groff
ARGUMENTS

   image
        image

```

## func assert_docker_image_not_exists

This function asserts that a docker image does not exists locally.

```Groff
ARGUMENTS

   image
        image

```

## func assert_empty

All arguments passed to assert_empty must be empty strings or else it will die and display the first that is not.

## func assert_exists

Accepts any number of filenames.  Blows up if any of the named files exist.

## func assert_not_empty

All arguments passed to assert_not_empty must be non-empty strings or else it will die and display the first that is
not.

## func assert_var_empty

Accepts variable names as parameters.  All passed in variable names must be either unset or must contain only an empty
string.

Note: there is not an analogue assert_var_not_empty.  Use argcheck instead.
