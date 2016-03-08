### Overview

grpc/src/core/census/

A resource measurement and tracing system of gRPC.

This directory contains code for Census, which will ultimately provide the following features for any gRPC-using system:

1. A dapper-like tracing system, enabling tracing across a distributed infrastructure.
RPC statistics and measurements for key metrics, such as latency, bytes transferred, number of errors etc.
2. Resource measurement framework which can be used for measuring custom metrics. Through the use of tags, these can be broken down across the entire distributed stack.

#### aggregation.h

```c
/* GRPC_INTERNAL_CORE_CENSUS_AGGREGATION_H */
struct census_aggregation_ops {
    /* create new aggregation, the pointer returned can be used */
    /* in future calls to clone(), free(), record(), data() and rest()*/
}
```

#### context.c

```c
// Functions in this file support the public context API, including
// encoding/decoding as part of context propagation across RPC's. The overall
// requirements (in approximate priority order) for the
// context representation:
// 1. Efficient conversion to/from wire format
// 2. Minimal bytes used on-wire
// 3. Efficient context creation
// 4. Efficient lookup of tag value for a key
// 5. Efficient iteration over tags
// 6. Minimal memory footprint
```

#### grpc_context.c

1. grpc\_census\_call\_set\_context

#### grpc_filter.c

```c
typedef struct call_data {
  census_op_id op_id;
  census_context *ctxt;
  gpr_timespec start_ts;
  int error;

  /* recv callback */
  grpc_metadata_batch *recv_initial_metadata;
  grpc_closure *on_done_recv;
  grpc_closure finish_recv;
} call_data;
```

```c
typedef struct channel_data { uint8_t unused; } channel_data;
```

1. server\_start\_transport\_op
2. client\_init\_call\_elem
3. client\_init\_call\_ele
4. server\_init\_call\_elem
5. server\_destroy\_call\_elem
6. init\_channel\_elem
7. destroy\_channel\_elem

#### mlog.c/mlog.h

Padding log for difference OS.


#### operation.c

1. census\_start\_rpc\_op\_timestamp
2. census\_start\_client\_rpc\_op
3. census\_start\_server\_rpc\_op
4. census\_start\_op

#### placeholders.c

This is a Place holder for pending APIs

#### tracing.c

Mask for placeholder implementations

