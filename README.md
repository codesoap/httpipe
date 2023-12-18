httpipe is a format that allows communicating and storing collections
of HTTP requests and responses. It is line based and uses JSON, so that
it works well with UNIX tools and is easy to implement with common
libraries.

# Basics
A httpipe communication is a unidirectional stream of JSON objects,
one per line. Each JSON object contains the information about one
HTTP request and/or response. Here is an example for a stream of two
requests:

```
{"host":"test.net","port":80,"tls":false,"req":"GET /foo HTTP/1.1\r\nHost: test.net:80\r\n\r\n"}
{"host":"test.net","port":80,"tls":false,"req":"GET /bar HTTP/1.1\r\nHost: test.net:80\r\n\r\n"}
```

Here is a single line containing a request and response:

```
{"host":"test.net","port":443,"tls":true,"req":"GET /src HTTP/1.1\r\nHost: test.net:443\r\n\r\n","resp":"HTTP/1.1 200 OK\r\nContent-Type: text/html; charset=utf-8\r\nContent-Length: 59\r\n\r\n\u003cpre\u003e\n\u003ca href=\"/\"\u003eNothing to see here, go back.\u003c/a\u003e\n\u003c/pre\u003e\n"}
```

# JSON fields
No field in the JSON is required by this specification. However,
different tools will require different fields. It is recommended to
include as many fields as possible when generating data and require only
the fields that are absolutely necessary when consuming data.

- Field `host`: The host that a request should be sent to or a response was received from.
- Field `port`: The port that is used with `host`. Recommended defaults are 443 if `tls` is `true` and 80 if it is `false`.
- Field `tls`: Whether TLS (HTTPS) is used or not (HTTP). Recommended default is `true`.
- Field `req`: The full HTTP request, including HTTP method, header fields and, if applicable, payload.
- Field `resp`: The full HTTP response, including HTTP status line, response header fields and payload (if applicable).

## WIP
Here I'm collecting ideas for fields that may be added in the future:

- Field `time`: The timestamp when the request was sent.
- Field `ping` or `dur`: The delay measured between sending the request and receiving a response in ms.
- Field `err` and/or `errno`: A field with information about errors that occurred after the request. Things like timeouts, DNS problems or TLS misconfiguration.

# Tools
Here are some tools—some written with httpipe in mind, some not—that can
create, transform, filter and consume httpipe streams.

## jq
`jq` is a powerful tool that can be used to filter, manipulate and
pretty print httpipe streams. Although not designed for httpipe, it
works with httpipe input and can produce httpipe output with the -c
flag.

```bash
# Pretty print every line:
jq . /path/to/httpipe/file

# Filter out every line, that matches some criteria:
jq -c 'select(.host == "test.net")' /path/to/httpipe/file

# Change a value within every line:
jq -c '.host = "newhost.net"' /path/to/httpipe/file

# Print the decoded response of the first line:
head -n1 /path/to/httpipe/file | jq -r .resp
```
