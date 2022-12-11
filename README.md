# SendServe
The theoretical fastest static content server possible

## Motivation

Serving static content should be easy, all the kernel needs to do is call [sendfile()](https://man7.org/linux/man-pages/man2/sendfile.2.html) with the appropriate headers. This project is an exploration into how far this can be improved.

## Optimizations

The theoretical fastest way to serve the response would be to keep the kernel and response (header + body) in L1 cache, then to serve the content using [send()](https://man7.org/linux/man-pages/man2/send.2.html) or [write()](https://man7.org/linux/man-pages/man2/write.2.html). To avoid multiple `send()` calls, the response should be in a contiguous block. Since the `Date` header changes every second, this should be generated every second and cached.

The theoretical fastest way to process requests would be to use SIMD to parse the headers. See https://stackoverflow.com/questions/26586060/why-is-strcmp-not-simd-optimized. There might also be a performance improvement by having multiple threads handling requests. In this case, we might need some message passing / signal handling to share the pointer of the reponse across all the threads, though, the cost of that might outweigh any benifits so each thread might also want to generate their own reponses independently.

Other optimizations that should be explored:
- `Send()` `MSG_DONTWAIT` flag
- `O_NONBLOCK` flag (via the [fcntl()](https://man7.org/linux/man-pages/man2/fcntl.2.html) `F_SETFL` operation)
- https://stackoverflow.com/questions/19078319/c-linux-socket-send-performance
- https://stackoverflow.com/questions/3251327/design-choices-for-high-performance-file-serving
- https://stackoverflow.com/questions/6866611/linux-multicast-sendto-performance-degrades-with-local-listeners
- https://stackoverflow.com/questions/19580957/performance-of-socket-between-windows-and-linux
- https://stackoverflow.com/questions/27948953/high-performance-packet-handling-in-linux
- https://stackoverflow.com/questions/13937096/linux-select-performance-scenarios
- https://stackoverflow.com/questions/7928241/optimizing-linux-socket
- https://stackoverflow.com/questions/21975577/udp-sendto-performance-over-loopback

## Benchmarking 

In alignment with the [TechEmpower Plaintext benchmark](https://www.techempower.com/benchmarks/#test=plaintext), this ASCII should the response for any HTTP method on `/plaintext`:
```
HTTP/1.1 200 OK\r\n
Date: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT\r\n
Server: SendServe\r\n
Content-Length: 13\r\n
Content-Type: text/plain\r\n
\r\n
Hello, World!
```
