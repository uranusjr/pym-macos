=============================================
PythonUp — The Python Runtime Manager (POSIX)
=============================================

PythonUp helps your install and manage Python runtimes on your computer. This
is the POSIX version.

.. highlights::

    Windows user? Check out `PythonUp-Windows`_.

.. _`PythonUp-Windows`: https://github.com/uranusjr/pythonup-windows

.. highlights::
    This is a work in progress. You are welcomed to try it, and I’ll try to
    resolve any problems you have using it. I’m still experimenting much of the
    internals, however, so anything can break when you upgrade without
    backward-compatibility in mind.


Distribution
============

1. Clone the repository somethere.
2. Create a Python environment for the project.
3. Create a shim to run `pythonup` withe the environment.
4. Add locations PythonUp installs scripts into to your PATH.

This is the steps I use::

    mkdir -p ~/.local/libexec/pythonup-posix
    cd ~/.local/libexec/pythonup-posix
    git clone https://github.com/uranusjr/pythonup-posix.git repo
    python3.6 -m venv --prompt=pythonup-posix venv
    ./venv/bin/python -m pip install --upgrade setuptools pip
    ./venv/bin/python -m pip install click dataclasses packaging
    ln -s $PWD/repo/pythonup ./venv/lib/python3.6/site-packages

Shim to invoke PythonUp::

    #!/bin/sh
    exec $HOME/.local/libexec/pythonup-posix/venv/bin/python -m pythonup $*

Aside from usual Python dependencies, PythonUp also requires

1. `python-build` from pyenv_. You don’t need to install pyenv, only the
   ``python-build`` command. Clone the repository, and ``ln -s`` the command
   (in ``pyenv/plugins/python-build/bin``) into your ``PATH``.

2. Build dependencies for Python. pyenv maintains lists for common package
   managers: https://github.com/pyenv/pyenv/wiki#suggested-build-environment

.. _pyenv: https://github.com/pyenv/pyenv


Quick Start
===========

Install Python 3.6::

    $ pythonup install 3.6

Run Python::

    $ python3

Install Pipenv to Python 3.6::

    $ pip3.6 install pipenv

And use it immediately (DOES NOT WORK YET, see TODO below)::

    $ pipenv --version
    pipenv, version 9.0.1

Install Python 2.7::

    $ pythonup install 3.5-32

Switch to a specific version::

    $ pythonup use 3.5
    $ python3 --version
    Python 3.5.4

Switch back to 3.6::

    $ pythonup use 3.6
    $ python3 --version
    Python 3.6.4
    $ python3.5 --version
    Python 3.5.4

Uninstall Python::

    $ pythonup uninstall 3.5

Use ``--help`` to find more::

    $ pythonup --help
    $ pythonup install --help


Internals
=========

PythonUp uses pyenv’s ``python-build`` command to build the best match, and
install it into ``$HOME/Library/PythonUp/versions/X.Y``. Unlike pyenv, PythonUp
only lets you specify X.Y, not the micro part, so you can upgrade within a
minor version without breaking all your existing virtual environments.


Todo
====

Shims
-----

Similar to pyenv (and PythonUp on Windows), ``pip`` and ``easy_install``
commands should be shimmed to allow auto-publishing hooks after you install a
package. Unlike the Windows implementation, some simple shell scripts will
suffice, fortunately. The script will be generated dynamically, when the user
``use`` versions, to point to the correct version.


Bundle python-build
-------------------

There are several disadvantages depending on Homebrew’s pyenv:

* pyenv does not release a new version to add a new Python definition.
* Homebrew does not always update the pyenv formula when pyenv releases.

Python 3.6.4, for example, was released on 2017-12-19. The python-build
definition landed a few hours later, but is still not available as a versioned
release (as of 2018-01-05). Judging from recent release patterns, availability
of new Python versions can be delayed to up to one month after their official
distribution.

I’m personally working around this by using the ``HEAD`` version of pyenv (
``brew install --HEAD pyenv``), but this is not a good long-term solution. It
would be better to vendor python-build (maybe as a Git subtree), and update
when user queries Python versions (e.g. with ``install`` and ``list``).

Another benefit of vendoring is that we don’t need the ``python-build`` command
to be globally available.


Explain things
--------------

Obvious question: Why not just use pyenv? Because you always want to use the
latest micro of a Python version, but pyenv doesn’t let you do that easily
without breaking all your virtual environments and globally installed tools.
Also the shims are a terrible idea.


Tests
-----

I always say this, but all my projects are under-tested. Hashtag help-wanted.


Documentation
-------------

It *might* be a good idea to unify the documentation? It makes sense from a
user’s perspective because the interfaces are almost identical. The
implementation and all underlying parts are different though. This would
require some very careful planning.
