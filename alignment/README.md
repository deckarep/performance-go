## Alignment as a mechanical sympathy can reduce memory footprint and performance

## Check your alignment with the help of this tool
* For propriertary structs, give the fields generic names when using this tool.
* http://golang-sizeof.tips/

## Alignment check linter
* https://github.com/opennota/check

```sh
# sample output
aligncheck net/http
net/http: /usr/lib/go/src/net/http/server.go:123:6: struct conn could have size 160 (currently 168)
net/http: /usr/lib/go/src/net/http/server.go:315:6: struct response could have size 152 (currently 176)
net/http: /usr/lib/go/src/net/http/transfer.go:37:6: struct transferWriter could have size 96 (currently 112)
net/http: /usr/lib/go/src/net/http/transport.go:49:6: struct Transport could have size 136 (currently 144)
net/http: /usr/lib/go/src/net/http/transport.go:811:6: struct persistConn could have size 160 (currently 176)
```