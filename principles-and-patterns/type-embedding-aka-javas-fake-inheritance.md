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
{% endtab %}

{% tab title="Go" %}
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

```
meow
I'm a kind of cat!
```
{% endtab %}
{% endtabs %}
