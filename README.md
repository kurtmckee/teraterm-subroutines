What is subroutine.ttl?
=======================

Tera Term Macro has limited support for writing reusable code. It is possible
to ``include`` a file but it is not possible to use a ``goto`` or ``call``
command to easily jump to a specific block of code in a file.

subroutine.ttl emulates a function call syntax and allows you to write more
modular, compartmentalized code. For example, given two files (main.ttl and
util.ttl), you could write something like this in main.ttl:

    callsub = 'util:clear_system_logs'
    include 'subroutine.ttl'

This will call the ``clear_system_logs`` subroutine in util.ttl:

    :clear_system_logs
        ; Put your code here!
    return

subroutine.ttl requires Tera Term version 4.66. It is licensed under the terms
of [the MIT license](http://opensource.org/licenses/MIT). See LICENSE.txt for
the full text of the license.


How do I use it?
================

You will need to copy subroutine.ttl into your source code directory and copy
the following code to the top of each of your modular .ttl files:

    ; Confirm that ``__include_goto_label`` is defined and is a string.
    ifdefined __include_goto_label
    if result == 3 then
        ; Confirm that ``__include_goto_label`` is not an empty string.
        strlen __include_goto_label
        if result then
            ; Confirm that ``__include_goto_label`` is an actual label.
            sprintf2 __execstr 'ifdefined %s' __include_goto_label
            execcmnd __execstr
            if result == 4 then
                ; The label exists. Call the subroutine and return.
                sprintf2 __execstr 'call %s' __include_goto_label
                execcmnd __execstr
                exit
            endif
            result = GOTO_LABEL_NOT_DEFINED
            exit
        endif
    endif
    ; This is the default subroutine to call. Change the name if desired.
    call default
    exit


Then, back in main.ttl, set the ``callsub`` variable. You can use one of the
following formats:

* ``<include file>:<goto label>``
* ``<include file>``

If the goto label is not specified, the hook code above will call the default
subroutine. For readability, you can also leave the '.ttl' extension off of
the filename.

Second, include subroutine.ttl in your main file.

    Example: main.ttl
    -----------------------------------------------------------------------
    ; The hook in util.ttl will call the default subroutine
    ; if you just include util.ttl.
    include 'util.ttl'
    
    ; Call the default subroutine.
    callsub = 'util'
    include 'subroutine.ttl'
    
    ; Call the ``clear_system_logs`` subroutine.
    callsub = 'util:clear_system_logs'
    include 'subroutine.ttl'
    -----------------------------------------------------------------------


What happens if there is a problem?
===================================

As much as possible, subroutine.ttl attempts to avoid throwing errors. To avoid
errors, it attempts to detect errors before they happen and will return an
error code in the ``result`` variable. It also defines several constants that
you can use for more readable error checking. For instance:

    callsub = 'util:label_with_typoo'
    include 'subroutine.ttl'
    if result == GOTO_LABEL_NOT_DEFINED then
        messagebox 'The goto label is not defined!' 'Shwoops'
    endif

Here is the complete list of error code constants and their meanings:

| Constant name           | Meaning                                           |
|-------------------------|---------------------------------------------------| 
| CALLSUB_IS_NOT_DEFINED  | The ``callsub`` variable has not been set.        |
| CALLSUB_IS_NOT_A_STRING | The ``callsub`` variable is not a string.         |
| CALLSUB_IS_EMPTY        | The ``callsub`` variable is an empty string.      |
| GOTO_LABEL_NOT_DEFINED  | The specified goto label is not valid.            |
| INCLUDE_DEPTH_EXCEEDED  | The maximum include depth (9) has been exceeded.  |
| INCLUDE_FILE_NOT_FOUND  | The specified file does not exist.                |


What else can I do with subroutine.ttl?
=======================================

Tera Term is designed so that everything is in a global namespace. Because of
this, it's possible to treat subroutines like functions. For instance, you can
define argument variables before calling a subroutine and check return
variables defined in the subroutines. For instance:


    Example: main.ttl
    -----------------------------------------------------------------------
    __arg_ip_address = '192.168.1.1'
    callsub = 'util:get_serial_number'
    include 'subroutine.ttl'
    messagebox serial_number 'Serial number'
    
    
    Example: util.ttl
    -----------------------------------------------------------------------
    :get_serial_number
        ; Connect to __arg_ip_address
        sendln 'echo $SERIAL_NUMBER'
        recvln
        serial_number = inputstr
    return


It's also possible to modify subroutine.ttl to automatically add a directory
prefix to all of the filenames by changing the ``__include_filename`` variable.
For instance, if you have a directory structure like this:

    project
    +-- includes
    |   +-- logging.ttl
    |   +-- network.ttl
    |   `-- util.ttl
    `-- main.ttl

You can change the ``__include_filename`` variable from ``''`` to
``'includes\'``:

    __include_filename = 'includes\'
