zBug
====

zBug (pronounced *zee-bug*) is a front end to the LLDB debugger (see http://lldb.llvm.org).

License
-------

zBug is licensed under a BSD 2-clause open-source license; see LICENSE for details.

Requirements
------------

zBug is written in Python and uses the PySide toolkit to provide a user interface.  zBug communicates with LLDB using the LLDB Python module.

To use zBug, you must meet the following requirements:

- You must be using Python version 2.7 or later.

- You must have PySide installed.  PySide can be installed using your OS package manager (or MacPorts on OS X), or can be obtained directly from http://qt-project.org/wiki/PySide.

- You must have LLDB installed with its Python bindings.  LLDB comes with Xcode on OS X; on other platforms, it can be installed using the OS package manager or it can be obtained directly from http://lldb.llvm.org.

Installation
------------

Simply copy the single file "zBug" to somewhere on your PATH and ensure that it is executable.  You must also ensure that the PySide and LLDB Python modules are on your PYTHONPATH.

- On OS X, the LLDB Python module that comes with Xcode is not on the PYTHONPATH by default.  However, zBug should automatically find the LLDB module anyway!

Using zBug
----------

Currently, zBug must be started from the command line.  Simply follow it with the path to the program to run as well as any optional arguments you wish to pass.  For example::
    
    zBug ./myTestProg optionalArgument

You will be presented with a window that contains all of the various debugging Windows; at the centre is a Window that provides a command prompt (marked 'lldb:') and the debugger log window.  Simply execute commands from this command prompt.