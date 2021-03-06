# Promptly

[![Build Status](https://travis-ci.org/aventurella/promptly.png?branch=master)](https://travis-ci.org/aventurella/promptly)

A little python utility to help you build command line prompts that can
be styled using CSS.

# Changes

## v0.6.1

Switched to six for compat
Added travis test for python 3.4

## v0.6.0

Inputs can now specify duplicate keys and a list will be returned, for example:
```python
from promptly import console
from promptly import Form

form = Form()

form.add.string(
    'name',
    'What is your name?',
    default='Ollie'
)

form.add.string(
    'name',
    'What is your other name?',
    default='Potato'
)

form.add.int(
    'number',
    'Pick a number'
)

console.run(form)
print(dict(form))

#
# {
#   'name': ['value1', 'value2'],
#   'number': 9
# }
#
```


## v0.5.4

Altered the console runners.console.ConsoleRunner.render()  to fix inconsistent
rendering in some teminals.

There was an issue on some terminals where passing the full prompt to
to input/raw_input (py3 and py2 respectively) would do some interesting
things with line wrapping and color codes. Part of the color code issue
was probably related to `readline` and this reported documentation issue
re ansi escapes for readline (http://bugs.python.org/issue17337#msg183328)

http://bugs.python.org/issue17337

The fix applied however, skips sending the majority of the prompt through
input/raw_input and instead writes it to stdout. Only the single line
for the actual prompting is now sent to input/raw_input. This appears
to address the issue.


## v0.5.3
Notifications can now be added into forms. The effect for the console runner
will be to simply print the notification and then continue to the next
question in the form. When converting the form to a dict, like branches,
notifications will not be included in the data that is returned.

```python
from promptly import console
from promptly import Form

form = Form()
form.add.notification('Welcome to a promptly form')
form.add.string('favorite_dog', 'Who is your favorite dog?', default='Lucy')
form.add.notification('Thanks for filling out the form!')
console.run(form)

print(dict(form))
```

In the case of the above you would get a prompt sequence like this:

```
···
Welcome to a promptly form
···

Who is your favorite dog?
> Lucy

···
Thanks for filling out the form!
···

{'favorite_dog': 'Lucy'}

```


## v0.5.2
Pyreadline does not supprt `set_startup_hook(lambda: readline.insert_text(default))`
as such the defaults on windows based machines would not show up.

The solution for this release was to alter the console renderer for string
and integer input on those machines who have pyreadline installed.

In that case the prompt will appear as follows:

```
Who is your favorite dog? [Lucy]
>
```

Note that the default is tacked on to the end of the question.

Compare this to the style we get when we have a system that
supports readline:

```
Who is your favorite dog?
> Lucy
```

Integers and String based prompts are rendered like this.


## v0.5.1
Added some love for our Windows friends. We now check weather or not readline is available
and if not install pyreadline in that case.

## v0.5
## WARNING 0.5 is backwards incompatible

This should be the last backwards incompatible update for a while. v0.5
saw a redesign of how forms are run. This was done in the hope that one day
I have time to do a curses or urwid implementation. We will see. On the whole
though it does make it more confirguable for individuals that do not like
the default form rendering as Promptly now supports form runners.

What are form runners? Well put simply, in prior versions you would call:

```python
    # < v0.5
    from promptly import From


    form = Form()
    form.add.string('favorite_food', 'What is your favorite food?')
    form.run()
```

This worked well, but it bound the prompts to a single implementation of the
`Form` object. `v0.5` treats the `Form` object as more of a collection and the
runners figure out how to deal with it. Lets take a look at the example from
above in in `v0.5`:

```python
    # v0.5+
    from promptly import From
    form promptly import console


    form = Form()
    form.add.string('favorite_food', 'What is your favorite food?')
    console.run(form)
```

Pretty much exactly the same, but we just hand the form off to the
run to deal with, instead of the form.

Some additional changes, the `promptly.inputs.*` have all been renamed
and simplified. Now they basically act as marker classes for input types.
They help the runner identify the kind of prompts to generate.

The logic, such as `StringInput.build_prompt`, basically got moved into
`promptly.renderers.console.StringPrompt`. If you were always using the
shortcut syntax for cerating your forms:

```python
    form.add.string(...)
    form.add.bool(...)
    form.add.int(...)
    form.add.select(...)
    form.add.multiselect(...)
```

Then you don't have to worry about anything, everything should still
work fine for you. If you were using the more verbose style:

```python

        form.add(
            'age',
            IntegerInput('What is your age?',
            default=1)
        )
```

Things will break for you. It's probably better to always be using the
shortcuts.

All of the input types now take "notifications" This is a convenient way
to annotate your questions. Lets take a look at a prompt with notifications
and the same prompt without notifications.

First, no notifications:

```python
    from promptly import From
    form promptly import console


    form = Form()
    form.add.string('name', 'What is your name?', default='Lucy')
    console.run(form, prefix='[promptly] ')
```

That will generate a prompt that looks like this:

```
    [promptly] What is your name?
    > Lucy
```

Now lets look at the same prompt with notifications:

```python
    from promptly import From
    form promptly import console


    form = Form()
    form.add.string(
        'name',
        'What is your name?',
        notifications=('This will help to identify you later', 'Identification is fun!')
        default='Lucy')
    console.run(form, prefix='[promptly] ')
```

That will generate a prompt that looks like this:

```
    [promptly] What is your name?
    This will help to identify you later
    Identification is fun!
    ···
    > Lucy
```

The notifications appear after the question, but before the user input.

The available CSS styles have also been updated to account for these.
See the list below for the default styles available.

There is also convenience function for just dropping notifications
to the console without running though a form. They will be styled according
to the notification and prefix styles:

```python
    from promptly import console

    console.notification('Hello World', prefix='[notice] ', stylesheet=None)
```

This will immediately write a message to sys.stdout.

## v0.4
**Migration Guide**
## WARNING 0.4 is backwards incompatible

**Migration Guide**
-   `my_form.add.choice` should be become `my_form.add.select`
-   `ChoiceInput` should become `SelectInput`
-   SelectInput (formerly ChoiceInput) and MultiSelectInput now take
    an option_format callable. By default this callable is
    `promptly.utils.numeric_options`. This will take a list ['foo', 'bar']
    and return a list: [(1, 'foo'), (2, 'bar')]. So if you only need
    numbers for your choices or multi-select input's you don't
    need to worry about, you get them for free. If you were passing
    your own in something like: `zip(range(1,3), ['foo', 'bar'])` you
    no longer need to do that. In fact that will break things for you
    so you should replace it with just your list of choices



### New Features

#### Branches
Forms can now branch. The branch input item takes a callable that will
be executed and is expected to return another `Form` object. This `Form`
object will be merged into the currently running form at the location
where the branch was added. The callable signature is as follows:

`my_branch_building_action(form, *args, **kwargs):`

Example branch usage:

```python

def handler(form, name):
    branch = Form()

    if form.age.value < 30:
        branch.add.string('name', 'What is your name?')
    else:
        branch.add.string('name', 'What is your pet's name?', default=name)

    return branch


form = Form()
form.add.int('age', 'What is your age?', default=age)
form.add.branch(handler, name='Lucy')

# The branch fields will be added here in terms of
# position in the form once the user reaches the branch

form.add.int('number', 'What is your favorite number?')
form.run()
```

#### MultiSelectInput
A new input type has been added, `MultiSelectInput`, a shortcut for creating
one is also available in the form of:
`my_form.add.multiselect(key, label, choices, done_label='Done')`

Note that done_label is optional.

MultiSelectInput lets the user choose multiple options from a SelectInput
style display. It marks the currently selected items. If the user chooses the
same option that has already been selected it will be deselected.

A final option is added to the list of choices provided to represent
the sentinel choice. The `done_label` kwarg sets the value used here
By default it is set to *Done*. The user must select the sentinel choice
in order to continue on in the form.





## Lets Make a Promptly Form

```python
    from promptly import Form
    form promptly import console


    # Build our form
    form = Form()

    # add questions in the sequence you would like them to appear

    form.add.string('name',
        'What is your name?',
        default='Aubrey')

    form.add.int('age',
        'What is your age?',
        default=1)

    # no options_format kwarg is provided for ChoiceInput
    # so it will use the default numeric_options
    form.add.select('color',
        'What is your favorite color',
        ('red', 'green', 'blue'),
        default=1)

    form.add.bool('yaks', 'Do you like yaks?', default=True)

    # Our form is created, lets prompt the user for the answers:

    # promptly comes with a default set of styles or you can
    # provide your own.

    with open('/path/to/my/styles.css') as css:
        console.run(form, prefix='[promptly] ', stylesheet=css.read())

    # control has returned back to our script, lets see what the user said:

    print(form.name.value)
    print(form.age.value)
    print(form.color.value)  # this will be a (key, value) tuple
    print(form.yaks.value)

    if form.age.value < 12:
        print(form.food.value)

    # Or we can just convert the whole form into a dictionary:
    d = dict(form)
    print(d)

```


## CSS Styling
Promptly prompts are styles with a very limited subset of CSS.
Only the following properties apply:

- color
- background-color
- font-weight

The avialable colors are limited to the color names provided by colorama:

```
    Fore: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE, RESET.
    Back: BLACK, RED, GREEN, YELLOW, BLUE, MAGENTA, CYAN, WHITE, RESET.
    Style: DIM, NORMAL, BRIGHT, RESET_ALL
```

In other words:
```css
    .prefix {
        color: white;
        background-color: blue;
    }
```

The font-weight property maps to colorama Style values in the following way:

```
    font-weight: normal;   -> Style.NORMAL
    font-weight: bold;     -> Style.BRIGHT
    font-weight: lighter;  -> Style.DIM
```

### Heads Up

The CSS parser in promptly is **VERY VERY** primitive. It's just enough to parse
what is below and that's all. It is by no means a full implementation of the
CSS spec.


## Default Prompty Stylesheet

Below is the default stylesheet included with promptly. This stylesheet
presents the exhaustive set of selectors that can be used to style
your prompts. If it's not below, promptly doesn't support it.

Remember each selector can support:

```css
    color: <value>
    background-color: <value>
    font-weight: </value>
```

The default stylesheet below does not use every available option
for obvious reasons. But you should feel free too if you so desire.

```body``` will set the default color and font-weight and background color.
The additional styles effectively cascade on top of body.


New selectors in `v0.5`
`.action` represents the Cheveron before the user input is displayed.
`.input` are the style for the user input.
`.notification .footer` are the styles for the 3 dots that appear below
selection choices and after notifications.

```css
    body{
    color:white;
    font-weight:normal;
    }

    .action{
        color:magenta;
        font-weight:bold;
    }

    .input{
        color:white;
        font-weight:bold;
    }

    .prefix{
        color:blue;
        font-weight:bold;
    }

    .notification .label{
        color:white;
        font-weight:bold;
    }

    .notification .footer{
        color:white;
        font-weight:normal;
    }

    .string .label{
        color:white;
    }

    .string .default-wrapper{
        color:white;
        font-weight:bold;
    }

    .string .default-value{
        color:yellow;
    }

    .integer .label{
        color:white;
    }

    .integer .default-wrapper{
        color:white;
        font-weight:bold;
    }

    .integer .default-value{
        color:yellow;
    }

    .boolean .label{
        color:white;
    }

    .boolean .default-wrapper{
        color:white;
        font-weight:bold;
    }

    .boolean .default-value{
        color:yellow;
    }

    .boolean .other-value{
        color:yellow;
    }

    .boolean .seperator{
        color:white;
        font-weight:bold;
    }

    .choices .label{
        color:white;
    }

    .choices .default-wrapper{
        color:white;
        font-weight:bold;
    }

    .choices .default-value{
        color:yellow;
    }

    .choices .option-key{
        color:yellow;
    }

    .choices .seperator{
        color:yellow;
        font-weight:lighter;
    }

    .choices .option-value{
        color:white;
        font-weight:bold;
    }

    .choices .action{
        color:magenta;
        font-weight:bold;
    }

    .choices .selection{
        color:white;
    }

```
