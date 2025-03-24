# Machine Access

## Virtual Machine Considerations

Self objects run in a virtual machine with bytecode operations (push, pop, send, non-local-return, etc.). This works well, but even with the best JIT compilers available, there are limitations which are evidenced by the need to drop into "primitives" to access certain functionality.

We could eliminate some primitives by using system calls more extensively. The Self VM currently allows direct syscalls, which could replace most file access, socket access, etc. This would work well on Linux and probably on BSDs as well.

## Performance and Machine Capabilities

We also drop into primitives for performance reasons, to access machine capabilities not easily accessed at the object level. Examples include specialized CPU instructions or efficient matrix operations on byte vectors. Writing these operations at the Self level would be much slower than dropping down to the CPU level.

## Security Considerations

A key question is whether we want to give objects access to this lower level. In other words, is the object level a security barrier where users are sandboxed within the Self object world?

Systems like Java and WebAssembly take this approach. You stay within your virtual machine, sandboxed, with access only to carefully mediated primitives. You can't reach down into the VM implementation itself, access primitives directly, or access underlying memory.

I don't think this is the direction we should go for two reasons:

1. **It doesn't work well in practice**: Your security is only as good as the VM, which will often have vulnerabilities. Securing the entire VM has proven difficult in practice (as seen with Java and similar systems).

2. **Philosophical/Ideological**: We want to give users access to the full machine. Self should empower programmers to use the full capabilities of the machine, not confine them to a playground where everything is safe but limited.

## OS Level Sandboxing

It's better to allow the Self world to reach down as far as possible to the machine level, with security parameters and barriers implemented at the OS level.

Currently, Psyche uses BSD jail systems, which provide separation between different jail instances (separate network stacks, etc.). This is a tradeoff. A flaw in the jail system might allow breaking out, but because it's a shared kernel system, we're using the machine more efficiently without worrying about memory ballooning issues. If needed, we could shift from jails to KVM, Bhyve, or similar technologies for greater hardware segregation between Self processes, 

Either way, security is handled at the OS level with hardware support rather than relying on sandboxing within the object world.

## Allowing Native Code

Previously, I created a native system for Self where you could write native code by putting machine code in a byte vector and calling it as a C function. This is powerful, and with a 64-bit clean virtual machine, we could implement this approach quickly.

However, we don't currently have a 64-bit virtual machine, which creates two limitations:
1. No clear path to get to a 64-bit machine
2. Limited resources in the current environment

Instead of native code, we need a "sidecar" approach, which I'll discuss later.
