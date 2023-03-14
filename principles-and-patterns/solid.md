# SOLID

| Principle                           | Description                                                                                                                                                                             |
| ----------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Single responsibility principle** | _“A class/function/type should have one, and only one reason to change”_                                                                                                                |
| **Open-Closed principle**           | _“A class should be open for extension but closed for modifications”_                                                                                                                   |
| **Liskov substitution principle**   | _“Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program”_                                                        |
| **Interface segregation principle** | _“Clients should not be forced to depend on methods they don't use”_                                                                                                                    |
| **Dependency inversion**            | _“High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should not depend on abstractions”_ |

#### **Single responsibility principle**

Before - breaks `SRP` because the function `area` will have to be changed for two reasons, the formula of area changes, and second the output of the program changes

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"math"
)

type circle struct {
	radius float64
}

func (c circle) area() {
	// violating Single Responsibility Principle
	fmt.Printf("circle area: %f\n", math.Pi*c.radius*c.radius)
}

func main() {
	c := circle{
		radius: 3,
	}
	c.area()

}
```
{% endcode %}

After - only have to change the function `area` when the formula changes and the function `print` when the printing format changes.

{% code lineNumbers="true" %}
```go
package main

import (
	"fmt"
	"math"
)

type circle struct {
	radius float64
}

func (c circle) area() float64 {
	return math.Pi * c.radius * c.radius
}

func (c circle) print() {
	fmt.Printf("The are of circle is %f \n", c.area())
}

type square struct {
	length float64
}

func (c square) print() {
	fmt.Printf("The are of square is %f \n", c.area())
}

func (c square) area() float64 {
	return c.length * c.length
}

type shape interface {
	name() string
	area() float64
}

func main() {

	c := circle{
		radius: 3,
	}

	c.print()

	s := square{
		length: 3,
	}

	s.print()

}
```
{% endcode %}

\
**Open-Closed principle**

In go, this can be achieved by using interfaces and polymorphism => all structs are open for extension but not for modification

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// This is an interface that defines a shape.
type Shape interface {
    Area() float64
}

// This is a struct that implements the Shape interface.
type Rectangle struct {
    width float64
    height float64
}

// This method calculates the area of a rectangle.
func (r Rectangle) Area() float64 {
    return r.width * r.height
}

// This is a struct that implements the Shape interface.
type Circle struct {
    radius float64
}

// This method calculates the area of a circle.
func (c Circle) Area() float64 {
    return 3.1415926535 * c.radius * c.radius
}

// This function takes a Shape as an argument and calculates its area.
// polymorphism
func calculateArea(s Shape) float64 {
    return s.Area()
}

func main() {
    rect := Rectangle{width: 10, height: 5}
    fmt.Println("Area of rectangle:", calculateArea(rect))

    circle := Circle{radius: 5}
    fmt.Println("Area of circle:", calculateArea(circle))
}
```
{% endcode %}

#### **Liskov substitution principle**

A derived class should be able to substitute its base class without affecting the functionality of the program. e.g. `calculateArea` can take any subtype of `Shape` - `Square`/ `Rectangle` and its behvaiour won't change.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// This is an interface that defines a shape.
type Shape interface {
    Area() float64
}

// This is a struct that implements the Shape interface.
type Rectangle struct {
    width float64
    height float64
}

// This method calculates the area of a rectangle.
func (r Rectangle) Area() float64 {
    return r.width * r.height
}

// This is a struct that implements the Shape interface.
type Square struct {
    side float64
}

// This method calculates the area of a square.
func (s Square) Area() float64 {
    return s.side * s.side
}

// This function takes a Shape as an argument and calculates its area.
func calculateArea(s Shape) float64 {
    return s.Area()
}

func main() {
    rect := Rectangle{width: 10, height: 5}
    fmt.Println("Area of rectangle:", calculateArea(rect))

    square := Square{side: 5}
    fmt.Println("Area of square:", calculateArea(square))
}

```
{% endcode %}

#### **Interface segregation principle**

In Golang, this can be achieved by creating smaller, focused interfaces that provide only the functionality needed by a specific client.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// This is an interface that defines basic operations for a shape.
type BasicShape interface {
    Area() float64
    Perimeter() float64
}

// This is an interface that defines advanced operations for a shape.
type AdvancedShape interface {
    Volume() float64
}

// This is a struct that implements the BasicShape interface.
type Rectangle struct {
    width float64
    height float64
}

// This method calculates the area of a rectangle.
func (r Rectangle) Area() float64 {
    return r.width * r.height
}

// This method calculates the perimeter of a rectangle.
func (r Rectangle) Perimeter() float64 {
    return 2 * (r.width + r.height)
}

// This is a struct that implements the BasicShape and AdvancedShape interfaces.
type Cube struct {
    side float64
}

// This method calculates the area of a cube.
func (c Cube) Area() float64 {
    return 6 * c.side * c.side
}

// This method calculates the perimeter of a cube.
func (c Cube) Perimeter() float64 {
    return 12 * c.side
}

// This method calculates the volume of a cube.
func (c Cube) Volume() float64 {
    return c.side * c.side * c.side
}

// This function takes a BasicShape as an argument and calculates its area.
func calculateArea(s BasicShape) float64 {
    return s.Area()
}

// This function takes an AdvancedShape as an argument and calculates its volume.
func calculateVolume(s AdvancedShape) float64 {
    return s.Volume()
}

func main() {
    rect := Rectangle{width: 10, height: 5}
    fmt.Println("Area of rectangle:", calculateArea(rect))

    cube := Cube{side: 5}
    fmt.Println("Area of cube:", calculateArea(cube))
    fmt.Println("Volume of cube:", calculateVolume(cube))
}
```
{% endcode %}

#### Dependency Inversion Principle

In this example, the **`Service`** struct depends on the **`Database`** interface, and the **`MySQL`** and **`PostgreSQL`** structs implement the **`Database`** interface. This means that the **`Service`** struct can store data in either a **`MySQL`** or **`PostgreSQL`** database, without having to know or care which database it’s using.&#x20;

This is an example of the Dependency Inversion Principle in action. The high-level **`Service`** module depends on an abstraction (the **`Database`** interface), while both the low-level **`MySQL`** and **`PostgreSQL`** modules depend on the same abstraction. This allows the **`Service`** module to be decoupled from the specific implementation of the database, making it more flexible and maintainable.

{% code lineNumbers="true" %}
```go
package main

import "fmt"

// This is an interface that defines basic operations for a database.
type Database interface {
    Connect()
    Store(data string)
}

// This is a struct that implements the Database interface.
type MySQL struct {}

// This method connects to a MySQL database.
func (m MySQL) Connect() {
    fmt.Println("Connecting to MySQL database...")
}

// This method stores data in a MySQL database.
func (m MySQL) Store(data string) {
    fmt.Println("Storing data in MySQL database:", data)
}

// This is a struct that implements the Database interface.
type PostgreSQL struct {}

// This method connects to a PostgreSQL database.
func (p PostgreSQL) Connect() {
    fmt.Println("Connecting to PostgreSQL database...")
}

// This method stores data in a PostgreSQL database.
func (p PostgreSQL) Store(data string) {
    fmt.Println("Storing data in PostgreSQL database:", data)
}

// This is a struct that depends on a Database interface.
type Service struct {
    db Database
}

// This method sets the Database for the Service.
func (s *Service) SetDatabase(db Database) {
    s.db = db
}

// This method stores data in a database using the Database interface.
func (s *Service) StoreData(data string) {
    s.db.Connect()
    s.db.Store(data)
}

func main() {
    // This creates a Service that uses a MySQL database.
    mysqlService := Service{db: MySQL{}}
    mysqlService.StoreData("Hello, world!")

    // This creates a Service that uses a PostgreSQL database.
    postgresService := Service{db: PostgreSQL{}}
    postgresService.StoreData("Hello, world!")
}
```
{% endcode %}

