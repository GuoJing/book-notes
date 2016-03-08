### Support

grpc/src/core/support/

Looks like this is a utils folder.

#### alloc.c

```c
static gpr_allocation_functions g_alloc_functions = {malloc, realloc, free};

gpr_allocation_functions gpr_get_allocation_functions() {
  return g_alloc_functions;
}
```

1. gpr\_set\_allocation\_functions

Set alloc functions to gpc\_alloc\_functions.

1. gpr\_malloc -> g\_alloc\_functions.malloc_fn
2. gpr\_free
3. gpr\_realloc
4. gpr\_malloc\_aligned
5. gpr\_free\_aligned

#### avl.c

This is a avl tree.

#### cpu_xxx.c

from header support/port\_platform.h # don't know where it is. =\_=

#### env.h

```c
Gets the environment variable value with the specified name.
Returns a newly allocated string. It is the responsability of the caller to
gpr_free the return value if not NULL (which means that the environment
variable exists).
```

1. gpr_setenv
2. gpr_getenv

#### env_xx.c

env.h implementations in different OS.

#### histogram

```c
Histograms are stored with exponentially increasing bucket sizes.
The first bucket is [0, m) where m = 1 + resolution
Bucket n (n>=1) contains [m**n, m**(n+1))
There are sufficient buckets to reach max_bucket_start
```
#### host_port.c

1. gpr\_join\_host\_port
2. gpr\_split\_host\_port

host and port string management.

#### load_file.c

Load file.

#### log.c/log_xx.c

implementations of log.h.

#### murmur_hash.c

Murmur hash alg [MurmurHash](https://en.wikipedia.org/wiki/MurmurHash).
