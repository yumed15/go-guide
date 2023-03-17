# Type Embedding aka Java's Fake Inheritance

{% hint style="danger" %}
**Type embedding isn't inheritance, and pretending it is will lead to bugs.**
{% endhint %}

It's possible to embed a type inside another type by using a struct.

```go
type inner struct {
    a int
}

type outer struct {
    inner
    b int
}

var x outer
fmt.Printf("inner.a is %d", x.a)
```

which allows you to share methods on the embedded type (which looks like inheritance but it's not)

```go
type printer interface {
	print()
}

type inner struct {
	a int
}

func (i inner) print() {
	fmt.Printf("a is %d", i.a)
}

type outer struct {
	inner
	b int
}

func main() {
	var x printer
	x = outer{inner{1}, 2}
	x.print()
}
```

not inheritance because we **can't override** the embedded's method implementation.

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
