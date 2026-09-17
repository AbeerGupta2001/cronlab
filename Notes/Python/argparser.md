### import argparser

The `argparse` module’s support for command-line interfaces is built around an instance of [`argparse.ArgumentParser`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser "argparse.ArgumentParser"). It is a container for argument specifications and has options that apply to the parser as whole:


**`argparser.ArgumentParser(prog: str | None = None, usage: str | None = None, description: str | None = None, epilog: str | None = None, parents: Iterable[ArgumentParser] = [], formatter_class: _FormatterClass = HelpFormatter, prefix_chars: str = "-", fromfile_prefix_chars: str | None = None, argument_default: Any = None, conflict_handler: str = "error", add_help: bool = True, allow_abbrev: bool = True, exit_on_error: bool = True, *, suggest_on_error: bool = False, color: bool = True) -> ArgumentParser`**

Object for parsing command line strings into Python objects.

Keyword Arguments:
  - prog -- The name of the program (default:
`os.path.basename(sys.argv[0])`)
  - usage -- A usage message (default: auto-generated from arguments)
  - description -- A description of what the program does
  - epilog -- Text following the argument descriptions
  - parents -- Parsers whose arguments should be copied into this one
  - formatter\_class -- HelpFormatter class for printing help messages
  - prefix\_chars -- Characters that prefix optional arguments
  - fromfile\_prefix\_chars -- Characters that prefix files containing
additional arguments
  - argument\_default -- The default value for all arguments
  - conflict\_handler -- String indicating how to handle conflicts
  - add\_help -- Add a -h/-help option
  - allow\_abbrev -- Allow long options to be abbreviated unambiguously
  - exit\_on\_error -- Determines whether or not ArgumentParser exits with
error info when an error occurs
  - suggest\_on\_error - Enables suggestions for mistyped argument choices
and subparser names (default: `False`)
  - color - Allow color output in help messages (default: `False`)


#### prog

Note that the program name, whether determined from `sys.argv[0]`, from the `__main__` module attributes or from the `prog=` argument, is available to help messages using the `%(prog)s` format specifier.

parser = argparse.ArgumentParser(prog='myprogram')
parser.add_argument('--foo', help='foo of the %(prog)s program')
parser.print_help()
usage: myprogram [-h] [--foo FOO]

options:
 -h, --help  show this help message and exit
 --foo FOO   foo of the myprogram program

### formatter_class[](https://docs.python.org/3/library/argparse.html#formatter-class "Link to this heading")

[`ArgumentParser`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser "argparse.ArgumentParser") objects allow the help formatting to be customized by specifying an alternate formatting class. Currently, there are four such classes:

_class_ argparse.RawDescriptionHelpFormatter[](https://docs.python.org/3/library/argparse.html#argparse.RawDescriptionHelpFormatter "Link to this definition")

_class_ argparse.RawTextHelpFormatter[](https://docs.python.org/3/library/argparse.html#argparse.RawTextHelpFormatter "Link to this definition")

_class_ argparse.ArgumentDefaultsHelpFormatter[](https://docs.python.org/3/library/argparse.html#argparse.ArgumentDefaultsHelpFormatter "Link to this definition")

_class_ argparse.MetavarTypeHelpFormatter[](https://docs.python.org/3/library/argparse.html#argparse.MetavarTypeHelpFormatter "Link to this definition")

[`RawDescriptionHelpFormatter`](https://docs.python.org/3/library/argparse.html#argparse.RawDescriptionHelpFormatter "argparse.RawDescriptionHelpFormatter") and [`RawTextHelpFormatter`](https://docs.python.org/3/library/argparse.html#argparse.RawTextHelpFormatter "argparse.RawTextHelpFormatter") give more control over how textual descriptions are displayed. By default, [`ArgumentParser`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser "argparse.ArgumentParser") objects line-wrap the [description](https://docs.python.org/3/library/argparse.html#description) and [epilog](https://docs.python.org/3/library/argparse.html#epilog) texts in command-line help messages:

parser = argparse.ArgumentParser(
    prog='PROG',
    description='''this description
        was indented weird
            but that is okay''',
    epilog='''
            likewise for this epilog whose whitespace will
        be cleaned up and whose words will be wrapped
        across a couple lines''')
parser.print_help()
usage: PROG [-h]

this description was indented weird but that is okay

options:
 -h, --help  show this help message and exit

likewise for this epilog whose whitespace will be cleaned up and whose words
will be wrapped across a couple lines

Passing [`RawDescriptionHelpFormatter`](https://docs.python.org/3/library/argparse.html#argparse.RawDescriptionHelpFormatter "argparse.RawDescriptionHelpFormatter") as `formatter_class=` indicates that [description](https://docs.python.org/3/library/argparse.html#description) and [epilog](https://docs.python.org/3/library/argparse.html#epilog) are already correctly formatted and should not be line-wrapped:

parser = argparse.ArgumentParser(
    prog='PROG',
    formatter_class=argparse.RawDescriptionHelpFormatter,
    description=textwrap.dedent('''\
        Please do not mess up this text!
        --------------------------------
            I have indented it
            exactly the way
            I want it
        '''))
parser.print_help()
usage: PROG [-h]

Please do not mess up this text!
--------------------------------
   I have indented it
   exactly the way
   I want it

options:
 -h, --help  show this help message and exit


## The add_argument() method[](https://docs.python.org/3/library/argparse.html#the-add-argument-method "Link to this heading")

ArgumentParser.add_argument(_name or flags..._, _*_[, _action_][, _nargs_][, _const_][, _default_][, _type_][, _choices_][, _required_][, _help_][, _metavar_][, _dest_][, _deprecated_])[](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_argument "Link to this definition")

Define how a single command-line argument should be parsed. Each parameter has its own more detailed description below, but in short they are:

- [name or flags](https://docs.python.org/3/library/argparse.html#name-or-flags) - Either a name or a list of option strings, e.g. `'foo'` or `'-f', '--foo'`.
    
- [action](https://docs.python.org/3/library/argparse.html#action) - The basic type of action to be taken when this argument is encountered at the command line.
    
- [nargs](https://docs.python.org/3/library/argparse.html#nargs) - The number of command-line arguments that should be consumed.
    
- [const](https://docs.python.org/3/library/argparse.html#const) - A constant value required by some [action](https://docs.python.org/3/library/argparse.html#action) and [nargs](https://docs.python.org/3/library/argparse.html#nargs) selections.
    
- [default](https://docs.python.org/3/library/argparse.html#default) - The value produced if the argument is absent from the command line and if it is absent from the namespace object.
    
- [type](https://docs.python.org/3/library/argparse.html#type) - The type to which the command-line argument should be converted.
    
- [choices](https://docs.python.org/3/library/argparse.html#choices) - A sequence of the allowable values for the argument.
    
- [required](https://docs.python.org/3/library/argparse.html#required) - Whether or not the command-line option may be omitted (optionals only).
    
- [help](https://docs.python.org/3/library/argparse.html#help) - A brief description of what the argument does.
    
- [metavar](https://docs.python.org/3/library/argparse.html#metavar) - A name for the argument in usage messages.
    
- [dest](https://docs.python.org/3/library/argparse.html#dest) - The name of the attribute to be added to the object returned by [`parse_args()`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.parse_args "argparse.ArgumentParser.parse_args").
    
- [deprecated](https://docs.python.org/3/library/argparse.html#deprecated) - Whether or not use of the argument is deprecated.

### action[](https://docs.python.org/3/library/argparse.html#action "Link to this heading")

[`ArgumentParser`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser "argparse.ArgumentParser") objects associate command-line arguments with actions. These actions can do just about anything with the command-line arguments associated with them, though most actions simply add an attribute to the object returned by [`parse_args()`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.parse_args "argparse.ArgumentParser.parse_args"). The `action` keyword argument specifies how the command-line arguments should be handled. The supplied actions are:

- `'store'` - This just stores the argument’s value. This is the default action.
    
- `'store_const'` - This stores the value specified by the [const](https://docs.python.org/3/library/argparse.html#const) keyword argument; note that the [const](https://docs.python.org/3/library/argparse.html#const) keyword argument defaults to `None`. The `'store_const'` action is most commonly used with optional arguments that specify some sort of flag. For example:
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--foo', action='store_const', const=42)
    parser.parse_args(['--foo'])
    Namespace(foo=42)
    
- `'store_true'` and `'store_false'` - These are special cases of `'store_const'` that respectively store the values `True` and `False` with default values of `False` and `True`:
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--foo', action='store_true')
    parser.add_argument('--bar', action='store_false')
    parser.add_argument('--baz', action='store_false')
    parser.parse_args('--foo --bar'.split())
    Namespace(foo=True, bar=False, baz=True)
    
- `'append'` - This appends each argument value to a list. It is useful for allowing an option to be specified multiple times. If the default value is a non-empty list, the parsed value will start with the default list’s elements and any values from the command line will be appended after those default values. Example usage:
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--foo', action='append', default=['0'])
    parser.parse_args('--foo 1 --foo 2'.split())
    Namespace(foo=['0', '1', '2'])
    
- `'append_const'` - This appends the value specified by the [const](https://docs.python.org/3/library/argparse.html#const) keyword argument to a list; note that the [const](https://docs.python.org/3/library/argparse.html#const) keyword argument defaults to `None`. The `'append_const'` action is typically useful when multiple arguments need to store constants to the same list. For example:
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--str', dest='types', action='append_const', const=str)
    parser.add_argument('--int', dest='types', action='append_const', const=int)
    parser.parse_args('--str --int'.split())
    Namespace(types=[<class 'str'>, <class 'int'>])
    
- `'extend'` - This appends each item from a multi-value argument to a list. The `'extend'` action is typically used with the [nargs](https://docs.python.org/3/library/argparse.html#nargs) keyword argument value `'+'` or `'*'`. Note that when [nargs](https://docs.python.org/3/library/argparse.html#nargs) is `None` (the default) or `'?'`, each character of the argument string will be appended to the list. Example usage:
    
    parser = argparse.ArgumentParser()
    parser.add_argument("--foo", action="extend", nargs="+", type=str)
    parser.parse_args(["--foo", "f1", "--foo", "f2", "f3", "f4"])
    Namespace(foo=['f1', 'f2', 'f3', 'f4'])
    
    Added in version 3.8.
    
- `'count'` - This counts the number of times an argument occurs. For example, this is useful for increasing verbosity levels:
    
    parser = argparse.ArgumentParser()
    parser.add_argument('--verbose', '-v', action='count', default=0)
    parser.parse_args(['-vvv'])
    Namespace(verbose=3)
    
    Unless explicitly set, the _default_ will be `None`. If the default value is a non-zero number, the count starts from that number rather than from zero.
    
- `'help'` - This prints a complete help message for all the options in the current parser and then exits. By default a help action is automatically added to the parser. See [`ArgumentParser`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser "argparse.ArgumentParser") for details of how the output is created.
    
- `'version'` - This expects a `version=` keyword argument in the [`add_argument()`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.add_argument "argparse.ArgumentParser.add_argument") call, and prints version information and exits when invoked:
    
    import argparse
    parser = argparse.ArgumentParser(prog='PROG')
    parser.add_argument('--version', action='version', version='%(prog)s 2.0')
    parser.parse_args(['--version'])
    PROG 2.0
    

You may also specify an arbitrary action by passing an [`Action`](https://docs.python.org/3/library/argparse.html#argparse.Action "argparse.Action") subclass (e.g. [`BooleanOptionalAction`](https://docs.python.org/3/library/argparse.html#argparse.BooleanOptionalAction "argparse.BooleanOptionalAction")) or other object that implements the same interface. Only actions that consume command-line arguments (e.g. `'store'`, `'append'`, `'extend'`, or custom actions with non-zero `nargs`) can be used with positional arguments.

The recommended way to create a custom action is to extend [`Action`](https://docs.python.org/3/library/argparse.html#argparse.Action "argparse.Action"), overriding the `__call__()` method and optionally the `__init__()` and `format_usage()` methods. You can also register custom actions using the [`register()`](https://docs.python.org/3/library/argparse.html#argparse.ArgumentParser.register "argparse.ArgumentParser.register") method and reference them by their registered name.

An example of a custom action:

class FooAction(argparse.Action):
    def __init__(self, option_strings, dest, nargs=None, **kwargs):
        if nargs is not None:
            raise ValueError("nargs not allowed")
        super().__init__(option_strings, dest, **kwargs)
    def __call__(self, parser, namespace, values, option_string=None):
        print('%r %r %r' % (namespace, values, option_string))
        setattr(namespace, self.dest, values)

parser = argparse.ArgumentParser()
parser.add_argument('--foo', action=FooAction)
parser.add_argument('bar', action=FooAction)
args = parser.parse_args('1 --foo 2'.split())
Namespace(bar=None, foo=None) '1' None
Namespace(bar='1', foo=None) '2' '--foo'
args
Namespace(bar='1', foo='2')

For more details, see [`Action`](https://docs.python.org/3/library/argparse.html#argparse.Action "argparse.Action").