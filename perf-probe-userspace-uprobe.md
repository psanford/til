# Perf probe userspace function tracing (uprobes)

## List functions in a binary

```
# list traceable functions starting with main
$ perf probe -x /usr/local/bin/dnsforward -F 'main*'
main.(*classicClient).Exchange
main.(*server).handleRequest
main.(*server).handleRequest-fm
main.(*server).handleRequest.stkobj
main.(*server).handleRequestConcurrent
main.(*server).handleRequestConcurrent.func1
main.(*server).handleRequestConcurrent.func1.stkobj
main.(*server).handleRequestConcurrent.func2
main.(*server).handleRequestConcurrent.func2.stkobj
main.(*server).handleRequestSerially
main.(*server).localOverrideResponse
main.(*server).logFailure
main.(*server).logFailure.stkobj
main.(*server).logFirstResult
main.(*server).logFirstResult.stkobj
main.(*server).logRequest
main.(*server).logRequest.stkobj
main.(*server).logResult
main.(*server).logResult.stkobj
main.(*server).queryBackend
main.(*server).shufClients
main.(*server).shufClients.func1
main.(*server).shufClients.stkobj
main.(*transitMode).String
main.(*transitMode).String.stkobj
main..inittask
main.confFile
main.init
main.loadOverrides
main.loadOverrides.stkobj
main.main
main.main.stkobj
main.msg.String
main.msg.String.stkobj
main.newDOHClient
main.newServer
main.newServer.stkobj
main.transitMode.String
main.transitMode.String.stkobj
```

## Get arguments for function

```
$ perf probe -x /usr/local/bin/dnsforward -V 'main.(*server).handleRequest'
Available variables at main.(*server).handleRequest
        @<main.(*server).handleRequest+0>
                github.com/miekg/dns.Msg*       r
                github.com/miekg/dns.ResponseWriter     w

# libc example:
$ perf probe -x /lib/x86_64-linux-gnu/libc.so.6 -V getaddrinfo
Available variables at getaddrinfo
        @<getaddrinfo+0>
                char*   __PRETTY_FUNCTION__
                char*   name
                char*   service
                struct addrinfo*        hints
                struct addrinfo**       pai
```


## Get struct offsets

For simple structs this is fairly easy:
```
$ perf probe -x /usr/local/bin/dnsforward -D  'main.(*server).handleRequest r->Question'
[]github.com/miekg/dns.Question exceeds max-bitwidth. Cut down to 64 bits.
p:probe_dnsforward/main /usr/local/bin/dnsforward:0x400220 Question=+40(+32(%sp)):x64
```

In this case we want the name, and perf probe doesn't understand golang slices.

This is what the go structs look like.

```
type Msg struct {
	MsgHdr
	Compress bool       `json:"-"` // If true, the message will be compressed when converted to wire format.
	Question []Question // Holds the RR(s) of the question section.
	Answer   []RR       // Holds the RR(s) of the answer section.
	Ns       []RR       // Holds the RR(s) of the authority section.
	Extra    []RR       // Holds the RR(s) of the additional section.
}

type Question struct {
	Name   string
	Qtype  uint16
	Qclass uint16
}
```

This is how slices are laid out in memory: https://research.swtch.com/godata


So to get the Name field we need to dereference the address of []Question to get the 0'th element of the backing array, and then dereference that address to get the Name string value:

```
$ perf probe -x /usr/local/bin/dnsforward  'main.(*server).handleRequest Question=+0(+0(+40(+32(%sp)))):string'
Added new event:
  probe_dnsforward:main (on main.(*server).handleRequest in /usr/local/bin/dnsforward with Question=+0(+0(+40(+32(%sp)))):string)

You can now use it in all perf tools, such as:

        perf record -e probe_dnsforward:main -aR sleep 1

```

You can translate this to the uprobe_event syntax with -D:


```
$ perf probe -x /usr/local/bin/dnsforward -D  'main.(*server).handleRequest Question=+0(+0(+40(+32(%sp)))):string'
p:probe_dnsforward/main /usr/local/bin/dnsforward:0x400220 Question=+0(+0(+40(+32(%sp)))):string
```

## Watch live events:

```
$ perf trace -e  probe_dnsforward:main
     0.000 dnsforward/5546 probe_dnsforward:main:(800220) Question="cnn.com."
     0.072 dnsforward/1505 probe_dnsforward:main:(800220) Question="cnn.com."
    26.500 dnsforward/5546 probe_dnsforward:main:(800220) Question="67.193.101.151.in-addr.arpa."
  4122.467 dnsforward/1505 probe_dnsforward:main:(800220) Question="google.com."
  4122.416 dnsforward/5546 probe_dnsforward:main:(800220) Question="google.com."
  4147.214 dnsforward/6031 probe_dnsforward:main:(800220) Question="46.32.251.142.in-addr.arpa.MT"
 10286.022 dnsforward/6031 probe_dnsforward:main:(800220) Question="d.la5-c1-ia5.salesforceliveagent.com."
 14454.652 dnsforward/6031 probe_dnsforward:main:(800220) Question="signaler-pa.clients6.google.com.0.0.0.0 jgen7.cjt1.net
"
 40977.771 dnsforward/6031 probe_dnsforward:main:(800220) Question="23.client-channel.google.com."
 76303.523 dnsforward/6031 probe_dnsforward:main:(800220) Question="signaler-pa.clients6.google.com.0.0.0.0 adbrite.112.2o7.net
```

## List probes:

```
$ perf probe -l
  probe_dnsforward:main (on main.(*server).handleRequest in /usr/local/bin/dnsforward with Question)
```

## Delete probe:

```
$ perf probe -d probe_dnsforward:main
Removed event: probe_dnsforward:main
```
