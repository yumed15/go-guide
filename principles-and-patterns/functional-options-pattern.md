# Functional Options Pattern

* when you want to provide optional configuration

From:

{% code lineNumbers="true" %}
```go
type Server struct{
    host string
    port int
    protocol string
}

func NewServer(host string, port int) *Server{
    return &Server{
        host: host,
        port: port,
        protocol: "http",
    }
}
```
{% endcode %}

To:

{% code lineNumbers="true" %}
```go
type ServerOption func(*Server)

func WithPort(port int) ServerOption{
    return func(s *Server){
        s.port = port
    }
}

func NewServer(host string, opts ...ServerOption) *Server{
    server := &Server{
        host: host,
        port: 443,
        protocol: "http",
    }
    
    for _, opt := range = opts{
        opt(server)
    }
    
    return server
}

server1 := NewServer("localhost")  
server2 := NewServer("localhost", WithPort(8080))
```
{% endcode %}
