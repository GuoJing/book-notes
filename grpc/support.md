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

#### slice.c

```c
gpr_slice gpr_empty_slice(void) {
  gpr_slice out;
  out.refcount = 0;
  out.data.inlined.length = 0;
  return out;
}
```

###### gpr\_slice\_malloc

```c
/* Memory layout used by the slice created here:
+-----------+----------------------------------------------------------+
| refcount  | bytes                                                    |
+-----------+----------------------------------------------------------+
refcount is a malloc_refcount
bytes is an array of bytes of the requested length
Both parts are placed in the same allocation returned from gpr_malloc */

malloc_refcount *rc = gpr_malloc(sizeof(malloc_refcount) + length);
gpr_ref_init(&rc->refs, 1);
rc->base.ref = malloc_ref;
rc->base.unref = malloc_unref;
```

#### stack_lockfree.h

```c
typedef struct gpr_stack_lockfree gpr_stack_lockfree;

/* This stack must specify the maximum number of entries to track.
   The current implementation only allows up to 65534 entries */

gpr_stack_lockfree *gpr_stack_lockfree_create(size_t entries);
void gpr_stack_lockfree_destroy(gpr_stack_lockfree *stack);

/* Pass in a valid entry number for the next stack entry */
/* Returns 1 if this is the first element on the stack, 0 otherwise */

int gpr_stack_lockfree_push(gpr_stack_lockfree *, int entry);

/* Returns -1 on empty or the actual entry number */
int gpr_stack_lockfree_pop(gpr_stack_lockfree *stack);
```

#### stack_lockfree.c

```c
/* The lockfree node structure is a single architecture-level
   word that allows for an atomic CAS to set it up. */
```

lockfree\_node\_contents

```c
struct lockfree_node_contents {
  /* next thing to look at. Actual index for head, next index otherwise */
  uint16_t index;
#ifdef GPR_ARCH_64
  uint16_t pad;
  uint32_t aba_ctr;
#else
#ifdef GPR_ARCH_32
  uint16_t aba_ctr;
#else
#error Unsupported bit width architecture
#endif
#endif
};
```

1. gpr\_stack\_lockfree\_create
2. gpr\_stack\_lockfree\_destroy
3. gpr\_stack\_lockfree\_push
4. gpr\_stack\_lockfree\_pop

#### string

Crose platform string implementation.

#### time

Crose platform time implementation

#### thd.c

```
/* Posix implementation for gpr threads. */
```

Code below.

```c
enum { GPR_THD_JOINABLE = 1 };

gpr_thd_options gpr_thd_options_default(void) {
  gpr_thd_options options;
  memset(&options, 0, sizeof(options));
  return options;
}

void gpr_thd_options_set_detached(gpr_thd_options* options) {
  options->flags &= ~GPR_THD_JOINABLE;
}

void gpr_thd_options_set_joinable(gpr_thd_options* options) {
  options->flags |= GPR_THD_JOINABLE;
}

int gpr_thd_options_is_detached(const gpr_thd_options* options) {
  if (!options) return 1;
  return (options->flags & GPR_THD_JOINABLE) == 0;
}

int gpr_thd_options_is_joinable(const gpr_thd_options* options) {
  if (!options) return 0;
  return (options->flags & GPR_THD_JOINABLE) == GPR_THD_JOINABLE;
}
```

#### sync

The important function is, still need to look into the code.

**gpr_atm_rel_store**

in file *include/grpc/impl/codegen/atm.h*

```c
#if defined(GPR_GCC_ATOMIC)
#include <grpc/impl/codegen/atm_gcc_atomic.h>
#elif defined(GPR_GCC_SYNC)
#include <grpc/impl/codegen/atm_gcc_sync.h>
#elif defined(GPR_WIN32_ATOMIC)
#include <grpc/impl/codegen/atm_win32.h>
#else
#error could not determine platform for atm
#endif
```

#### wrap_memcpy.c

Provide a wrapped memcpy for targets that need to be backwards compatible with older libc's. Enable by setting LDFLAGS=-Wl,-wrap,memcpy when linking.

```c
#ifdef __linux__
#ifdef __x86_64__
__asm__(".symver memcpy,memcpy@GLIBC_2.2.5");
void *__wrap_memcpy(void *destination, const void *source, size_t num) {
  return memcpy(destination, source, num);
}
#else /* !__x86_64__ */
void *__wrap_memcpy(void *destination, const void *source, size_t num) {
  return memmove(destination, source, num);
}
#endif
#endif
```
