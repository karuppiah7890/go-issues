https://github.com/golang/go/issues/38300

PR of sorts - https://go-review.googlesource.com/c/go/+/206658/

Changes / Commit - https://go.googlesource.com/go/+/06314b620d782bf295a07745c7f14b4738e7109a

On checking this

https://go.googlesource.com/go/+/06314b620d782bf295a07745c7f14b4738e7109a/src/cmd/compile/internal/logopt/log_opts.go

I found out that when you have a golang file say `main.go`

```go
package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/", func (w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "Welcome to my website!")
    })

    http.ListenAndServe(":80", nil)
}
```

And when you do

```bash
$ go get -v golang.org/dl/go1.15beta1
$ go1.15beta1 download
$ go1.15beta1 tool compile -json=0,file://dest-directory main.go
```

Then some JSON file is created

```bash
$ ls dest-directory/%00/main.json
dest-directory/%00/main.json
```

And it looks like this

```json
{"version":0,"package":"\u0000","goos":"darwin","goarch":"amd64","gc_version":"go1.15beta1","file":"main.go"}
{"range":{"start":{"line":8,"character":6},"end":{"line":8,"character":6}},"severity":3,"code":"cannotInlineFunction","source":"go compiler","message":"unhandled op CLOSURE"}
{"range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}},"severity":3,"code":"cannotInlineCall","source":"go compiler","message":"net/http.(*ServeMux).Handle cannot be inlined","relatedInformation":[{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2441,"character":12},"end":{"line":2441,"character":12}}},"message":"inlineLoc"}]}
{"range":{"start":{"line":9,"character":26},"end":{"line":9,"character":26}},"severity":3,"code":"canInlineFunction","source":"go compiler","message":"cost: 62"}
{"range":{"start":{"line":9,"character":26},"end":{"line":9,"character":26}},"severity":3,"code":"escape","source":"go compiler","message":"func literal escapes to heap","relatedInformation":[{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":26},"end":{"line":9,"character":26}}},"message":"escflow:    flow: http.handler = \u0026{storage for func literal}:"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":26},"end":{"line":9,"character":26}}},"message":"escflow:      from func literal (spill)"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:      from http.pattern, http.handler = \u003cN\u003e (assign-pair)"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:    flow: http.handler = http.handler:"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:      from http.pattern, http.handler = \u003cN\u003e (assign-pair)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:    flow: {heap} = http.handler:"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2441,"character":33},"end":{"line":2441,"character":33}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:      from http.Handler(http.HandlerFunc(http.handler)) (interface-converted)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2441,"character":33},"end":{"line":2441,"character":33}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":9,"character":20},"end":{"line":9,"character":20}}},"message":"escflow:      from http.mux.Handle(http.pattern, http.Handler(http.HandlerFunc(http.handler))) (call parameter)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2453,"character":28},"end":{"line":2453,"character":28}}},"message":"inlineLoc"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":2441,"character":12},"end":{"line":2441,"character":12}}},"message":"inlineLoc"}]}
{"range":{"start":{"line":9,"character":26},"end":{"line":9,"character":26}},"severity":3,"code":"escape","source":"go compiler","message":""}
{"range":{"start":{"line":9,"character":32},"end":{"line":9,"character":32}},"severity":3,"code":"leak","source":"go compiler","message":"parameter w leaks to {heap} with derefs=0","relatedInformation":[{"location":{"uri":"file://main.go","range":{"start":{"line":10,"character":20},"end":{"line":10,"character":20}}},"message":"escflow:    flow: {heap} = w:"},{"location":{"uri":"file://main.go","range":{"start":{"line":10,"character":20},"end":{"line":10,"character":20}}},"message":"escflow:      from w (interface-converted)"},{"location":{"uri":"file://main.go","range":{"start":{"line":10,"character":20},"end":{"line":10,"character":20}}},"message":"escflow:      from fmt.Fprintf(w, \"Welcome to my website!\", []interface {}(nil)...) (call parameter)"}]}
{"range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}},"severity":3,"code":"cannotInlineCall","source":"go compiler","message":"net/http.(*Server).ListenAndServe cannot be inlined","relatedInformation":[{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3091,"character":30},"end":{"line":3091,"character":30}}},"message":"inlineLoc"}]}
{"range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}},"severity":3,"code":"escape","source":"go compiler","message":"\u0026http.Server literal escapes to heap","relatedInformation":[{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3090,"character":12},"end":{"line":3090,"character":12}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}}},"message":"escflow:    flow: http.server·4 = \u0026{storage for \u0026http.Server literal}:"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3090,"character":12},"end":{"line":3090,"character":12}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}}},"message":"escflow:      from \u0026http.Server literal (spill)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3090,"character":12},"end":{"line":3090,"character":12}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}}},"message":"escflow:      from http.server·4 = \u0026http.Server literal (assign)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3090,"character":9},"end":{"line":3090,"character":9}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}}},"message":"escflow:    flow: {heap} = http.server·4:"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3091,"character":30},"end":{"line":3091,"character":30}}},"message":"inlineLoc"},{"location":{"uri":"file://main.go","range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}}},"message":"escflow:      from http.server·4.ListenAndServe() (call parameter)"},{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3091,"character":30},"end":{"line":3091,"character":30}}},"message":"inlineLoc"}]}
{"range":{"start":{"line":13,"character":24},"end":{"line":13,"character":24}},"severity":3,"code":"escape","source":"go compiler","message":"","relatedInformation":[{"location":{"uri":"file:///Users/karuppiahn/sdk/go1.15beta1/src/net/http/server.go","range":{"start":{"line":3090,"character":12},"end":{"line":3090,"character":12}}},"message":"inlineLoc"}]}
```

I tried the same with `go` version `1.14.4` but some other different and smaller
JSON was the output. Hmm.
