# Warden

**Category:** Pwn  
**Sources:** ctf.krauq.com, 0xasta.me  
**Flag:** `0xfun{wh0_w4tch3s_th3_w4rd3n_t0ctou_r4c3}`

## Challenge Summary

This challenge had two defensive layers:

1. a Python jail that blocked obvious dangerous functionality
2. a seccomp-based supervisor that inspected syscall pathnames and blocked access to `/flag*`

The solve required escaping the Python jail first, then bypassing the pathname-based seccomp check.

## Environment and Restrictions

The Python jail blocked:

- `import`
- `eval`, `exec`, and `open`
- attributes beginning with `_`
- string literals containing `__`

The seccomp supervisor used `SECCOMP_RET_USER_NOTIF` and read the target pathname with `process_vm_readv`. It then denied access if the provided path string started with `/flag`.

That filtering was naive because it only validated the literal string passed by the process, not the fully resolved file target.

## Layer 1 — Escape the Python Jail

The first goal was to regain access to useful modules without writing a normal `import` statement.

The trick was to dynamically construct dunder names at runtime:

```python
du = '_' * 2
```

From there, `getattr()` could be used to access blocked internals like `__subclasses__` without ever writing the forbidden text directly.

The chain was:

1. call `object.__subclasses__()`
2. find `_frozen_importlib.BuiltinImporter`
3. use `load_module('posix')`

```python
du = '_'*2
subs = getattr(object, du+'subclasses'+du)()
for t in subs:
    if getattr(t, du+'name'+du) == 'BuiltinImporter' and getattr(t, du+'module'+du) == '_frozen_importlib':
        BI = t
        break
p = BI.load_module('posix')
```

That restored low-level filesystem primitives such as `chdir`, `symlink`, `open`, and `read`.

## Layer 2 — Bypass the Seccomp Path Filter with a Symlink

The seccomp supervisor blocked pathnames beginning with `/flag`, but it did not resolve symlinks before making the decision.

Even better for the attacker, `symlink` and `symlinkat` were not blocked by the BPF policy.

So the bypass was straightforward:

```python
p.chdir('/tmp')
p.symlink('/flag', 'x')
fd = p.open('x', 0)
data = p.read(fd, 65536)
```

From the supervisor’s point of view, the program opened `x`, which did not begin with `/flag`. But the kernel resolved `x` to `/flag` and opened the forbidden file anyway.

## Why the Bypass Worked

This was basically a path-resolution mismatch:

- the warden validated the user-supplied pathname string
- the kernel resolved the actual file object later

That difference created a TOCTOU-style gap. The check happened on the string, while the actual file access happened on the symlink target.

## Flag Retrieval

Once the symlink was opened and read, the file contents revealed the flag:

```text
0xfun{wh0_w4tch3s_th3_w4rd3n_t0ctou_r4c3}
```

## Lessons from the Challenge

The challenge is a nice example of why blacklisting path prefixes is fragile.

- Python jail restrictions can often be bypassed through introspection.
- Syscall policies that inspect raw path strings are weaker than policies that reason about actual resolved file objects.
- Symlinks remain a classic way to defeat naive path validation.
