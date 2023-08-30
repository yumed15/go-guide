# Interfaces and Embedding

**An interface** = type that specifies a set of method signatures. When a concrete type provides definitions for all the methods in an interface, it is said to implement the interface.

{% code lineNumbers="true" %}
```go
type Shape interface {
  Area() float64
  Perimeter() float64
}

func (r Rectangle) Area() float64 {
  return r.Length * r.Width
}

func (r Rectangle) Perimeter() float64 {
  return 2 * (r.Length + r.Width)
}

type Rectangle struct {
  Length, Width float64
}

func main() {
  var r Shape = Rectangle{Length: 3, Width: 4}
  fmt.Printf("Type of r: %T, Area: %v, Perimeter: %v.", r, r.Area(), r.Perimeter())
}
```
{% endcode %}

**Embedding** - allows one struct type to include another struct, inheriting the fields and methods of the embedded type. It is Go's mechanism to achieve composition over traditional inheritance.

{% code lineNumbers="true" %}
```go
type Address struct {
    Street, City, State string
}

type Person struct {
    Name string
    Address
}

p := Person{
    Name:    "Alice",
    Address: Address{"123 Main St", "Anytown", "CA"},
}
fmt.Println(p.Street)  // Output: 123 Main St
```
{% endcode %}

Embedding provides a way to "inherit" methods.&#x20;

But it's not inheritance because Go doesn't have the keyword <mark style="color:red;">**this**</mark>**,** also we <mark style="color:red;">**can't override**</mark> the embedded's method implementation.

{% hint style="danger" %}
**Type embedding isn't inheritance, and pretending it is will lead to bugs.**
{% endhint %}

{% tabs %}
{% tab title="Java" %}
{% code lineNumbers="true" %}
```java
public class Animals {
  public static void main(String[] args) {
    Animals example = new Animals();
    example.Run();
  }
	
  public void Run() {
    Animal a = new Tiger();
    a.Greet();
  }
	
  interface Animal {
    public void Speak();
    public void Greet();
  }

  class Cat implements Animal {
    public void Speak() {
      System.out.println("meow");
    }

    public void Greet() {
      this.Speak();
      System.out.println("I'm a kind of cat!");
    }
  }

  class Tiger extends Cat {
    public void Speak() {
      System.out.println("roar");
    }
  }
}

```
{% endcode %}

```
roar
I'm a kind of cat!
```

In Java, `this` is a special implicit pointer that always has the runtime type of the class the method was originally invoked on. So `Tiger.Greet()` dispatches to `Cat.Greet()`, but the latter has a `this` pointer of type `Tiger`, and so `this.Speak()` dispatches to `Tiger.Speak()`.
{% endtab %}

{% tab title="Go" %}
{% code lineNumbers="true" %}
```go
type Animal interface {
	Speak()
	Greet()
}

type Cat struct {}

func (c Cat) Speak() {
	fmt.Printf("meow\n")
}

func (c Cat) Greet() {
	c.Speak()
	fmt.Printf("I'm a kind of cat!\n")
}

type Tiger struct {
	Cat
}

func (t Tiger) Speak() {
	fmt.Printf("roar\n")
}

func main() {
	var x Animal
	x = Tiger{}
	x.Greet()
}
```
{% endcode %}

```
meow
I'm a kind of cat!
```

In Golang, none of this happens. The `Cat.Greet()` method doesn't have a `this` pointer, it has a `Cat` receiver. When you call `Tiger.Greet()`, it's simply shorthand for `Tiger.Cat.Greet()`. The static type of the receiver in `Cat.Greet()` is the same as its runtime type, and so it dispatches to `Cat.Speak()`, not `Tiger.Speak()`.
{% endtab %}
{% endtabs %}

#### Resources:

* [https://www.dolthub.com/blog/2023-02-22-golangs-fake-inheritance/](https://www.dolthub.com/blog/2023-02-22-golangs-fake-inheritance/)
