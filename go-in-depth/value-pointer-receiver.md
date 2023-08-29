# Value/Pointer Receiver

```go
func (n Node) Name() string {...}   // Value Receiver     <-- creates a copy of the struct
func (n *Node) Name() string {...}  // Pointer Receiver   <-- doesn't create a copy
```

## Function vs Method

**A function** is declared by specifying the types of the arguments, the return values, and the function body.

{% code lineNumbers="true" %}
```go
type Person struct {
    Name string
    Age  int
}

func NewPerson(name string, age int) *Person {
  return &Person{
     Name: name,
     Age:  age,
  }
}
```
{% endcode %}

**A method** is just a function with a receiver argument. It is declared with the same syntax with the addition of the **receiver**.

{% code lineNumbers="true" %}
```go
func (p *Person) isAdult bool {
  return p.Age > 18
}
```
{% endcode %}

**Value receiver** makes a copy of the type and passes it to the function. The function stack now holds an equal object but at a different location on memory. That means any changes done on the passed object will remain local to the method. The original object will remain unchanged.

**Pointer receiver** passes the address of a type to the function. The function stack has a reference to the original object. So any modifications on the passed object will modify the original object.

{% code lineNumbers="true" %}
```go
package main
import "fmt"
type Person struct {
    Name string
    Age  int
}
func ValueReceiver(p Person) {
    p.Name = "John"
    fmt.Println("Inside ValueReceiver : ", p.Name)
}
func PointerReceiver(p *Person) {
    p.Age = 24
    fmt.Println("Inside PointerReceiver model: ", p.Age)
}
func main() {
    p := Person{"Tom", 28}
    p1:= &Person{"Patric", 68}
    ValueReceiver(p)
    fmt.Println("Inside Main after value receiver : ", p.Name)
    PointerReceiver(p1)
    fmt.Println("Inside Main after value receiver : ", p1.Age)
}

// Inside ValueReceiver :  John
// Inside Main after value receiver :  Tom
// Inside PointerReceiver :  24
// Inside Main after pointer receiver :  24
```
{% endcode %}

\
