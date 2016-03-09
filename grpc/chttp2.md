### HTTP/2 specification:

#### Overview

1. [HTML](http://http2.github.io/http2-spec/index.html)
2. [TEXT](http://http2.github.io/http2-spec/index.txt)
3. [ALNP RCF](http://http2.github.io/http2-spec/index.txt)

Tips:

1. The basic protocol unit in HTTP/2 is a frame. Each frame type serves a different purpose. For example, HEADERS and DATA frames form the basis of HTTP requests and responses. Other frame types like SETTINGS, WINDOW\_UPDATE, and PUSH\_PROMISE are used in support of other HTTP/2 features.
2. Multiplexing of requests is achieved by having each HTTP request/response exchange associated with its own stream. Streams are largely independent of each other, so a blocked or stalled request or response does not prevent progress on other streams.
3. HTTP/2 adds a new interaction mode whereby a server can push responses to a client.
4. Because HTTP header fields used in a connection can contain large amounts of redundant data, frames that contain them are compressed.


#### Frame format

All frames begin with a fixed 9-octet header followed by a variable-length payload.

```
 +---------------------------------------------------------------+
 |                 Length (24)                                   |
 +---------------+---------------+-------------------------------+
 |   Type (8)    |   Flags (8)   |
 +-+-------------+---------------+-------------------------------+
 |R|                 Stream Identifier (31)                      |
 +=+=============================================================+
 |                   Frame Payload (0...)                      ...
 +---------------------------------------------------------------+
```

* Length: The length of the frame payload expressed as an unsigned 24-bit integer. Values greater than 214 (16,384) MUST NOT be sent unless the receiver has set a larger value for SETTINGS_MAX_FRAME_SIZE.
* Type: The 8-bit type of the frame. The frame type determines the format and semantics of the frame. Implementations MUST ignore and discard any frame that has a type that is unknown.
* Flags: An 8-bit field reserved for boolean flags specific to the frame type.
* R: A reserved 1-bit field. The semantics of this bit are undefined, and the bit MUST remain unset (0x0) when sending and MUST be ignored when receiving.
* Stream Identifier: A stream identifier (see Section 5.1.1) expressed as an unsigned 31-bit integer. The value 0x0 is reserved for frames that are associated with the connection as a whole as opposed to an individual stream.

Tips:

Flags are assigned semantics specific to the indicated frame type. Flags that have no defined semantics for a particular frame type MUST be ignored and MUST be left unset (0x0) when sending.

#### Header Compression and Decompression

Tips:

Header compression is stateful. One compression context and one decompression context are used for the entire connection. A decoding error in a header block MUST be treated as a connection error of type COMPRESSION_ERROR.

#### Streams and Multiplexing

A "stream" is an independent.

1. A single HTTP/2 connection can contain multiple concurrently open streams, with either endpoint interleaving frames from multiple streams.
2. Streams can be established and used unilaterally or shared by either the client or server.
3. Streams can be closed by either endpoint.
4. The order in which frames are sent on a stream is significant.
5. Streams are identified by an integer.

Streams have the following states:

1. idle: All streams start in the "idle" state.
2. reserved (local)
3. reserved (remote)
4. open
5. half-closed (local)
6. half-closed (remote)
7. closed

#### Stream Priority

A client can assign a priority for a new stream by including prioritization information in the HEADERS frame that opens the stream.

#### Stream Dependencies

Each stream can be given an explicit dependency on another stream. Including a dependency expresses a preference to allocate resources to the identified stream rather than to the dependent stream.

And Stream could have dependency weight.

#### DATA

```
 +---------------------------------------------------------------+
 |Pad Length? (8)|
 +---------------+-----------------------------------------------+
 |                            Data (*)                         ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

#### HEADERS

```
 +---------------------------------------------------------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |E|                 Stream Dependency? (31)                     |
 +-+-------------+-----------------------------------------------+
 |  Weight? (8)  |
 +-+-------------+-----------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

#### PRIORITY

```
 +-+-------------------------------------------------------------+
 |E|                  Stream Dependency (31)                     |
 +-+-------------+-----------------------------------------------+
 |   Weight (8)  |
 +-+-------------------------------------------------------------+
```
 
#### RST_STREAM

The RST_STREAM frame (type=0x3) allows for immediate termination of a stream. RST_STREAM is sent to request cancellation of a stream or to indicate that an error condition has occurred.

```
 +---------------------------------------------------------------+
 |                        Error Code (32)                        |
 +---------------------------------------------------------------+
```
 
#### SETTINGS Format

```
 +---------------------------------------------------------------+
 |       Identifier (16)         |
 +-------------------------------+-------------------------------+
 |                        Value (32)                             |
 +---------------------------------------------------------------+
```
 
#### PUSH_PROMISE

The PUSH_PROMISE frame (type=0x5) is used to notify the peer endpoint in advance of streams the sender intends to initiate.

```
 +---------------------------------------------------------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |R|                  Promised Stream ID (31)                    |
 +-+-----------------------------+-------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```
 
 1. Pad Length: An 8-bit field containing the length of the frame padding in units of octets. This field is only present if the PADDED flag is set.
 2. R: A single reserved bit.
 3. Promised Stream ID: An unsigned 31-bit integer that identifies the stream that is reserved by the PUSH_PROMISE. The promised stream identifier MUST be a valid choice for the next stream sent by the sender.
 4. Header Block Fragment: A header block fragment containing request header fields.
 5. Padding: Padding octets.

#### PING

The PING frame (type=0x6) is a mechanism for measuring a minimal round-trip time from the sender, as well as determining whether an idle connection is still functional. PING frames can be sent from any endpoint.

```
 +---------------------------------------------------------------+
 |                                                               |
 |                      Opaque Data (64)                         |
 |                                                               |
 +---------------------------------------------------------------+
```

#### GOAWAY

The GOAWAY frame (type=0x7) is used to initiate shutdown of a connection or to signal serious error conditions. GOAWAY allows an endpoint to gracefully stop accepting new streams while still finishing processing of previously established streams. This enables administrative actions, like server maintenance.

If the receiver of the GOAWAY has sent data on streams with a higher stream identifier than what is indicated in the GOAWAY frame, those streams are not or will not be processed.

The receiver of the GOAWAY frame can treat the streams as though they had never been created at all, thereby allowing those streams to be retried later on a new connection.

Endpoints SHOULD always send a GOAWAY frame before closing a connection so that the remote peer can know whether a stream has been partially processed or not.

```
 +-+-------------------------------------------------------------+
 |R|                  Last-Stream-ID (31)                        |
 +-+-------------------------------------------------------------+
 |                      Error Code (32)                          |
 +---------------------------------------------------------------+
 |                  Additional Debug Data (*)                    |
 +---------------------------------------------------------------+
```
 
#### WINDOW_UPDATE

The WINDOW_UPDATE frame (type=0x8) is used to implement flow control.

```
 +-+-------------------------------------------------------------+
 |R|              Window Size Increment (31)                     |
 +-+-------------------------------------------------------------+
```

#### Error Codes
 
 1. NO_ERROR: 0x0
 2. PROTOCOL_ERROR: 0x1
 3. INTERNAL_ERROR: 0x2
 4. FLOW_CONTROL_ERROR: 0x3
 5. SETTINGS_TIMEOUT: 0x4
 6. STREAM_CLOSED: 0x5
 7. FRAME_SIZE_ERROR: 0x6
 8. REFUSED_STREAM: 0x7
 9. CANCEL: 0x8
 10. COMPRESSION_ERROR: 0x9
 11. CONNECT_ERROR: 0xa
 12. ENHANCE_YOUR_CALM: 0xb
 13. INADEQUATE_SECURITY: 0xc
 14. HTTP_1_1_REQUIRED: 0xd
 
#### HTTP Request/Response Exchange

A client sends an HTTP request on a new stream, using a previously unused stream identifier. A server sends an HTTP response on the same stream as the request.

An HTTP message (request or response) consists of:

1. for a response only, zero or more HEADERS frames (each followed by zero or more CONTINUATION frames) containing the message headers of informational (1xx) HTTP responses.
2. one HEADERS frame (followed by zero or more CONTINUATION frames) containing the message headers.
3. zero or more DATA frames containing the payload body.
4. optionally, one HEADERS frame, followed by zero or more CONTINUATION frames containing the trailer-part, if present.

#### Server Push

Server push is semantically equivalent to a server responding to a request; however, in this case, that request is also sent by the server, as a PUSH_PROMISE frame.

Once a client receives a PUSH_PROMISE frame and chooses to accept the pushed response, the client SHOULD NOT issue any requests for the promised response until after the promised stream has closed.

If the client determines, for any reason, that it does not wish to receive the pushed response from the server or if the server takes too long to begin sending the promised response, the client can send a RST\_STREAM frame, using either the CANCEL or REFUSED\_STREAM code and referencing the pushed stream's identifier.

A client can use the SETTINGS\_MAX\_CONCURRENT\_STREAMS setting to limit the number of responses that can be concurrently pushed by a server.

#### Connection Reuse

Connections that are made to an origin server, either directly or through a tunnel created using the CONNECT method, MAY be reused for requests with multiple different URI authority components.

A server that does not wish clients to reuse connections can indicate that it is not authoritative for a request by sending a 421 (Misdirected Request) status code in response to the request.
