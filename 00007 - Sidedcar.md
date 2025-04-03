# Sidecar

A 'sidecar' was mentioned in RDF:0004 - Machine Access. The general idea is that a second process is run, which the main Self VM talks to over a communications channel. This second process is a machine level process which can mount libraries, run optimised machine-code etc.

One possible model for a sidecar is to allow it to have full access to a range of shared memory libraries. A sidecar would be a 64-bit process running a simple loop. The loop looks at its message queue or shared memory, sees a message, copies it into its own memory, and executes it. It has the ability to dynamically load libraries, and there is a known and agreed set of libraries for it to load.

It might be easier to get started by creating a Lisp-level interface to the sidecar rather than building from scratch. This might allow some debugging. If we were to go down that path, Scheme, Forth, or Lisp would be three obvious candidates - small and self-contained.

Ultimately, we start with nothing, and then the first thing the main world does is load in a more friendly runtime. The main world needs to be able to talk to whatever communications channel we've chosen, so we might potentially need to compile a C-level language to machine code or assemble assembly to machine code. That would have to be possible within the running jail.

Here's the plan: In the jail, the jail is executing pure 64-bit machine language. There is Self-level code to create what's necessary for a sidecar. The first thing the world does, if it's not already running, is compile the sidecar, save it, run it, and start stopping it. That's the first bootstrap stage.

The second bootstrap stage is that it compiles a REPL (like Lisp, Forth, or Scheme) and loads it into the sidecar. Once that is running, it can then talk to that slightly higher level and request the sidecar do things such as load shared libraries, load network and disk resources, or perform more complex, time-consuming computations.
Questions About Sidecars

## Should a Self world be able to create more than one sidecar?

If we can create multiple sidecars, we can probably run multiple long-running computations simultaneously, but at the risk of complicating the model.

Should the barrier between jail instances of Self be merely a security barrier, or is it also a way of parallelizing computation? We already have multiple processor threads or processes in a shared object space. Why can't we also have multiple concurrent primitives? That would be an argument for allowing more than one sidecar.

Alternatively, we could make the sidecar itself multi-threaded, though this becomes quite complex. If a sidecar is a single thread, we're limited to one primitive at a time, which might not be what we want. We want to push a shared memory model and give more options, but maybe we should have multiple sidecars, even though they don't share memory. This needs further consideration.

## Requirements for Implementation

To make this work in our jail, we need either:

- A C compiler or assembler
- Or an assembler writing in Self itself

It should be possible to create an assembler in Self for a core set of AMD64 instructions, which can be expanded later. But it's a balancing act. We want to provide libraries that can be used by the sidecar. It's not meant to be a completely self-contained system; it's meant to be a system that can reuse other people's code.

Currently, we have the Unix model of reusing code, which is put in the shared library, and we have language-specific models of shared code, where we import a Python library, Rust library, or Go library. Those don't travel well into other languages, so it's not entirely clear how to handle them.

We could have a WebAssembly interpreter, but what's the point when we're already in a virtual machine? What we want is access to the underlying machine code.

We should compile a list of useful libraries. What are the assumptions of things such as BLAS? They obviously need to run their algorithms on multiple cores. We want to be able to do that too. We want to do matrix math at high speed. We cannot do that inside Self; we need to take advantage of other people's code, like NumPy, BLAS, and similar libraries.
