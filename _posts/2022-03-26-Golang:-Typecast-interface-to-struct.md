

### Here's my problem!
I have 20 different API endpoint, and each endpoint does it's own thing.
Let's say we have a struct for each of them and a `decoder` function to read the body whenever a request comes in. and this `decoder` function is common for every API endpoint. So we decided have this `decoder` function in app middleware package.

Now, I have request payload validator methods defined for each of these struct which does it's own thing.
I would like to trigger these validator functions from middleware `decoder` function itself.

Here's the thing though:
I don't know the type of these structs since `decoder` function was accepting an empty interface in `decoder` function, But i still like to call these `struct specific` validator function from decoder(`common function`) itself.

*Enter: Type casting of empty interface to structs*

Here's an example program for everyone to understand the typecasting!

```
// Golang typecasting to struct
package main

import (
	"errors"
	"fmt"
	"strings"
)

type someInterface interface {
	inputValidatorMethod() error
}

type myStruct1 struct {
	name string
}

type myStruct2 struct {
	id string
}

func main() {
	var s1 myStruct1 = myStruct1{name: "roshan"}
	var s2 myStruct2 = myStruct2{id: "rohan"}

	validate(&s1)
	validate(&s2)
}

func validate(v interface{}) {
	getValidFuncToRun, ok := v.(someInterface)
	if !ok {
		fmt.Println("Oh no!, struct doesn't have validator function")
	}
	err := getValidFuncToRun.inputValidatorMethod()

	if err != nil {
		fmt.Println(err)
	}
}

// inputValidatorMethod throws error if name doesn't start with "ro"
func (a *myStruct1) inputValidatorMethod() error {
	if result := strings.HasPrefix(a.name, "ro"); result {
		return errors.New("Name doesn't start with ro")
	}
	return nil
}

// inputValidatorMethod throws error if id doesn't start with "i-"
func (b *myStruct2) inputValidatorMethod() error {
	if result := strings.HasPrefix(b.id, "i-"); result {
		return errors.New("ID doesn't start with i-")
	}
	return nil
}

```

Happy learning!