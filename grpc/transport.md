### Transport

gRPC 传输层。需要先看 [chttp2](chttp2.md) 的实现。

#### byte_stream.h

```c
struct grpc_byte_stream {
  uint32_t length;
  uint32_t flags;
  int (*next)(grpc_exec_ctx *exec_ctx, grpc_byte_stream *byte_stream,
              gpr_slice *slice, size_t max_size_hint,
              grpc_closure *on_complete);
  void (*destroy)(grpc_exec_ctx *exec_ctx, grpc_byte_stream *byte_stream);
};
```

1. grpc\_byte\_stream\_next
2. grpc\_byte\_stream\_destroy
3. grpc\_slice\_buffer\_stream\_init

buffer stream 结构定义：

```c
typedef struct grpc_slice_buffer_stream {
  grpc_byte_stream base;
  gpr_slice_buffer *backing_buffer;
  size_t cursor;
} grpc_slice_buffer_stream;
```

byte_stream.c 文件没什么可看的。
