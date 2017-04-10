# Manticore

[![Build Status](https://travis-ci.com/trailofbits/manticore.svg?token=m4YsYkGcyttTxRXGVHMr&branch=master)](https://travis-ci.com/trailofbits/manticore)
[![Slack Status](https://empireslacking.herokuapp.com/badge.svg)](https://empireslacking.herokuapp.com)

Manticore is a prototyping tool for dynamic binary analysis, with support for symbolic execution, taint analysis, and binary instrumentation.

## Features

- **Input Generation**: Manticore automatically generates inputs that trigger unique code paths
- **Crash Discovery**: Manticore discovers inputs that crash programs via memory safety violations
- **Execution Tracing**: Manticore records an instruction-level trace of execution for each generated input
- **Programmatic Interface**: Manticore exposes programmatic access to its analysis engine via a Python API

Manticore supports binaries of the following formats, operating systems, and architectures. It has been primarily used on binaries compiled from C and C++.

- OS/Formats: Linux ELF, Windows Minidump
- Architectures: x86, x86_64, ARMv7 (partial)

## Requirements

Manticore is officially supported on Linux and uses Python 2.7.

## Installation

These install instructions require pip 7.1.0, due to `--no-binary`. If you have an older pip version, you might be able to use `--no-use-wheel` instead.

We recommend using Manticore in a virtual environment with [virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/). Run the following commands in the root of the Manticore repository to setup a virtual environment (note: The `--no-binary` flag is a workaround for a known Capstone [issue](https://github.com/aquynh/capstone/issues/445)).

```
mkvirtualenv manticore
pip install --no-binary capstone .
```

or, if you didn't use a virtualenv and would like to do a user install:

```
pip install --user --no-binary capstone .
```

This installs the Manticore CLI tool `manticore` and the Python API.

Then, install the Z3 Theorem Prover. Download the [latest release](https://github.com/Z3Prover/z3/releases/latest) for your platform and place the `z3` binary in your `$PATH`.

### For developers

For a dev install that includes dependencies for tests, run:

```
pip install -e --no-binary capstone --no-binary keystone-engine .[dev]
```

You can run the tests with the commands below:

```
cd /path/to/manticore/
# all tests
nosetests
# just one file
nosetests tests/test_armv7cpu.py
# just one test class
nosetests tests/test_armv7cpu.py:Armv7CpuInstructions
# just one test
nosetests tests/test_armv7cpu.py:Armv7CpuInstructions.test_mov_imm_min
```

## Quick start

Install and try Manticore in a few shell commands:

```
# follow install instructions in README.md before beginning
cd /path/to/manticore/
cd examples/linux
make
manticore basic
cat mcore_*/*1.stdin | ./basic
cat mcore_*/*2.stdin | ./basic
cd ../script
python count_instructions.py ../linux/helloworld # ok if the insn count is different
```

Here's an asciinema of what it should look like: https://asciinema.org/a/567nko3eh2yzit099s0nq4e8z

## Usage

```
$ manticore ./path/to/binary  # runs, and creates a directory with analysis results
```

or

```python
# example Manticore script
from manticore import Manticore

hook_pc = 0x400ca0

m = Manticore('./path/to/binary')

@m.hook(hook_pc)
def hook(state):
  cpu = state.cpu
  print 'eax', cpu.EAX
  print cpu.read_int(cpu.SP)

  m.terminate()  # tell Manticore to stop

m.run()
```

### Examples

Some example scripts using the Manticore API can be found in `examples/script`.

## FAQ

### How does Manticore compare to angr?

Manticore is simpler. It has a smaller codebase, fewer dependencies and features, and an easier learning curve. If you
come from a reverse engineering or exploitation background, you may find Manticore intuitive due to its lack of intermediate representation and overall emphasis on staying close to machine abstractions.
