# Module dialog


## func dialog

This is a generic wrapper around [dialog](https://invisible-island.net/dialog/#screenshot) which adds `--hide` and
`--trace` options across the board so that we don't have to implement wrappers for every widget. Moreover, it also deals
with dialog behavior of returning non-zero for lots of not-fatal reasons. We don't want callers to throw fatal errors
when that happens.  Intead they should inspect the error codes and output and take action accordingly. Secondly, we need
to capture the stdout from dialog and then parse it accordingly. Using the tryrc idiom addresses these issues by
capturing the return code into `dialog_rc` and the output into `dialog_output` for subsequent inspection and parsing.

## func dialog_cancel

`dialog_cancel` is a helper function to cancel the dialog process that we spawned in a consistent reusable way.

## func dialog_checklist

`dialog_checklist` provides a very simple interface around the dialog checklist widget by simplying operating on an
array variable. Instead of taking in raw strings and worrying about quoting and escaping this simply takes in the
**name** of an array and then directly operates on it. Each entry in the widget is composed of the first three elements
in the array.

So, typically you would format the array as follows:

```shell
array=()
array+=( tag "item text with spaces" status )
```

Where `status` is either `on` or `off`

Each 3-tuple in the array will be parsed and presented in a checklist widget. At the end, the output is parsed to
determine which ones were selected and the input array is updated for the caller. With the `--delete` flag (on by
default) it will delete anything in the array which was not selected. If this flag is not used, then the caller can
manually look at the `status` field in each array element to see if it is `on` or `off`.

`dialog_checklist` tries to intelligently auto detect the geometry of the window but the caller is always allowed to
override this with `--geometry` option.

## func dialog_error

Helper function to make it easier to display simple error boxes inside dialog. This is similar in purpose and
usage to `eerror` and will display a dialog msgbox with the provided text.

## func dialog_info

Helper function to make it easier to display simple information boxes inside dialog. This is similar in purpose and
usage to `einfo` and will display a dialog msgbox with the provided text.

## func dialog_kill

`dialog_kill` is a helper function to provide a consistent and safe way to kill dialog subprocess that we spawn.

## func dialog_prgbox

Helper function to make it easier to use `dialog --prgbox` without buffering. This is done using stdbuf which can then
disable buffering on stdout and stderr before invoking the command requested by the caller. This way we have a nice
uniform way to call external programs and ensure their output is displayed in real-time instead of waiting until the
program completes.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --geometry, -g <value>
         Optional geometry in 'HxW format.

   --ok-label <value>
         Optional override of the OK button text.


ARGUMENTS

   text
         Text to display in the program box.

   command
         Command to execute and display the output from inside the program box.

```

## func dialog_prompt

`dialog_prompt` provides a very simple interface for the caller to prompt for one or more values from the user using the
[dialog](https://invisible-island.net/dialog/#screenshot) ncurses tool. Each named option passed into `dialog_prompt`
will be displayed as a field within dialog.  By default each value is initially empty, but the caller can override this
by using `option=value` syntax wherein `value` would be the initial value for `option` and displayed in the dialog
interface. By default all options are **required** and the user will be unable to exit the dialog interface until all
required fields are provided. The caller can prefix an option with a `?` to annotate that it is optional. In the dialog
interface, required options are marked as required with a preceeding `*`. After the user fills in all required fields,
the provided option names will be set to the user provided values. This is done using the "eval command invocation
string" idiom so that the code to set variables is executed in the caller's environment.

For example:

```shell
$(dialog_prompt field)
```

`dialog_prompt` tries to intelligently auto detect the geometry of the window based on the number of fields being prompted
for. It overcomes some annoyances with dialog not scaling very well with how it pads the fields in the window. But the
caller is always allowed to override this with the `--geometry` option.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --backtitle <value>
         Text to display on backdrop at top left of the screen.

   --declare
         Declare variables before assigning to them. This is almost always required unless the caller has already declared the variables before calling
         into dialog_prompt and disires to simply reuse the existing variables.

   --geometry <value>
         Geometry of the box (HEIGHTxWIDTHxMENU-HEIGHT).

   --help-callback <value>
         Callback to invoke when 'Help' button is pressed.

   --help-label <value>
         Override label used for 'Help' button.

   --hide
         Hide ncurses output from screen (useful for testing).

   --instructions
         Include instructions on how to navigate dialog window.

   --title <value>
         String to display as the top of the dialog box.

   --trace
         If enabled, enable extensive dialog debugging to stderr.

   --transform (&)
         Accumulator of sed-like replace expressions to perform on dialog labels. Expressions are 's/regexp/replace/[flags]' where regexp is a regular
         expression and replace is a replacement for each label matching regexp. For more details see the sed manpage.


ARGUMENTS

   fields
         List of option fields to prompt for. Field names may not contain spaces, newlines or special punctuation characters.
```

## func dialog_prompt_username_password

dialog_prompt_username_password is a special case of dialog_prompt that is specialized to deal with username and password
authentication in a secure manner by not displaying the passwords in plain text in the dialog window. It also deals
with pecularities around a password wherein we want to present a second inbox box to confirm the password being
entered is valid. If they don't match the caller is prompted to re-enter the password(s). Otherwise it functions the
same as dialog_prompt does with the "eval command invocation string" idiom so that the code to set variables is
executed in the caller's environment. For example: $(dialog_prompt_username_password). The names of the variables it sets
are 'username' and 'password'.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --declare
         Declare variables before assigning to them. This is almost always required unless the caller has already declared the variables before calling
         into dialog_prompt and disires to simly reuse the existing variables.

   --optional, -o
         If true, the username and password are optional. In this case the user will be allowed to exit the dialog menu without providing username
         and passwords. Otherwise it will sit in a loop until the user provides both values.

   --title <value>
         Title to put at the top of the dialog box.

```

## func dialog_prompt_username_password_UI

This function separates the UI from the business logic of the username/password function. This allows us to unit test
the business logic without user interaction.

```Groff
OPTIONS
(*) Denotes required options
(&) Denotes options which can be given multiple times

   --password <value>
         Password to display (obscured), if any

   --title <value>
         Text for title bar of dialog

   --username <value>
         Username to display, if any

```

## func dialog_radiolist

`dialog_radiolist` provides a very simple interface around the dialog radiolist widget by simplying operating on an
array variable. Instead of taking in raw strings and worrying about quoting and escaping this simply takes in the
**name** of an array and then directly operates on it. Each entry in the widget is composed of the first three elements
in the array.

So, typically you would format the array as follows:

```shell
array=()
array+=( tag "item text with spaces" status )
```

WHere `status` is either `on` or `off`

Each 3-tuple in the array will be parsed and presented in a radiolist widget. At the end, the output is parsed to
determine which ones were selected and the input array is updated for the caller. With the `--delete` flag (on by
default) it will delete anything in the array which was not selected. If this flag is not used, then the caller can
manually look at the `status` field in each array element to see if it is `on` or `off`.

`dialog_radiolist` tries to intelligently auto detect the geometry of the window but the caller is always allowed to
override this with `--geometry` option.

NOTE: A radiolist is almost identical to a checklist only the radiolist only allows a single element to be selected
whereas a checklist allows multiple rows to be selected.

## func dialog_read

Helper function for `dialog_prompt` to try to read a character from stdin. Unfortunately some arrow keys and other
control characters are represented as multi-byte characters (see `EBASH_KEY_UP`, `EBASH_KEY_RIGHT`, `EBASH_KEY_DOWN`,
`EBASH_KEY_RIGHT`, `EBASH_KEY_DELETE`, `EBASH_KEY_BACKSPACE`, `EBASH_KEY_SPACE`). So this function helps by reading a
character and checking if it looks like the start of a multi-byte control character. If so, it will read the next
character and so on until it has read the required 4 characters to know if it is indeed a multi-byte control character
or not.

```Groff
ARGUMENTS

   output
        output

```

## func dialog_warn

Helper function to make it easier to display simple warning boxes inside dialog. This is similar in purpose and
usage to `ewarn` and will display a dialog msgbox with the provided text.
