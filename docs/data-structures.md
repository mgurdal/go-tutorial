[English](/go-tutorial/data-structures) | [Türkçe](/go-tutorial/tr/data-structures)

This page describes the data structures of go language and their usages in general programming.

Array
--------

Arrays are structures that enable us to keep the same types of data together. For example; A list consisting of 4, 5, 19 and 25 defines an array.
Go does not allow us to create arrays from different data types.

### Definition

Arrays are basically defined as `[Size]Type`. We can see several ways to create arrays below.

An array capable of holding 3 units of type `string`.


```go
var fruits[3]string
```


When defined as such, each field in the array is filled with the **default** value of the data type. Based on this example, it will be defined as `[0, 0, 0]`. We can change these values ​​by accessing them on their indexes.


```go
fruits[0] = "🍌"
fruits[1] = "🍏"

fruits // [🍌 🍏]
```


If we use a data type other than the one defined for Array, we will encounter an error.


```go
arr [2] = 1
```


```Bash
cannot use 1 (type int) as type string in assignment
```

We can define the same array in a shorter way.


```go
fruits := [3]int{"🍌", "🍏"}

fruits // [🍌 🍏]
```


In this definition, we assign a 3 unit long array whose first 2 elements we directly define to the variable 'arr'.
  
If we are going to create the arrayi with the values, we can leave the dimension calculation part to compiler using the expression `...`


```go
fruits := []int{"🍌", "🍏", "🍉"}

fruits // [🍌 🍏 🍉]
```


Using with Loop

Arrays can be used basically in 2 ways with the for loop.

1- Accessing with Index.
```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for index := range fruits {
    fruit := fruits[index]
    fmt.Println(fruit)
}
```

2- By directly accessing the value.

```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for _, fruit := range fruits {
    fmt.Println(fruit)
}
```


Slice
---------

Slices are very similar to arrays in terms of usage. The difference of slices is that their size is dynamic.

When defining slice, we use the structure of `[]Type` as in the array, but we leave the brackets empty.


```go
// define nil slice
var slc []int
// assigning value to nill slice
slc = []int{12, 24, 39}

// direct identification
slc := []int{12, 24, 39}
```


#### make

We can also use the `make` function to define the slice. The first parameter of the `make` function determines the size of the slice to be created.


```go
// define by size
n_items = 20
slc := make([]int, n_items)

// definition by size and capacity
capacity := 40
slc := make([]int, n_items, capacity)
```


### Slice Operations

While creating the slice, it is filled with the default value of the data type that it will hold, as much as its capacity. However, only part of its length can be accessed.

If we want to increase the size of a slice, we can use the `append` and` copy` functions for this.

### copy

`copy` allows us to copy one slice of another slicea of ​​the same or larger capacity and returns how many elements are copied.


```go
bucket := make([]string, 5)

fruits := []string{"🍎", "🍇", "🍒", "🍉"}

n_copied := copy(bucket, fruits)

n_copied // 5
bucket // [🍎 🍇 🍈 🍉]

```

When copying a slice, the elements replace the elements in the same index in the target slice.


```go
fruits := []string{"🍇", "🍒", "🍉"}
apples := []string{"🍎", "", "🍏"}

copy(fruits, apples)

fruits // [🍎 🍏]

```


### append

The `append` function takes the slice that we will increase in size as the first 
parameter and the elements we will add later and return a ** new ** slice containing 
all the elements to us. Therefore, it is necessary to keep the return value 
in a variable (usually the old slice variable).


```go
bucket := []string{"🍎", "🍇", "🍈", "🍉"}

green_apple := "🍏"
bucket = append(bucket, green_apple)

bucket // [🍎 🍇 🍈 🍉 🍏]
```


> When we resize a slice and assign it to another variable, changes in the new slice will also affect the original slice.


```go
fruits := []string{"🍎", "🍎", "🍇", "🍒", "🍉"}
apples := fruits[:2]

apples[0] = "🍏"
apples[1] = "🍏"
 
apples // [🍏 🍏]
fruits // [🍏 🍏 🍇 🍒 🍉]


```


### cut

If we want to subtract a certain range in the slice, we can use the method below. This method will add the elements up to the start index we have specified in Slice and the elements after the end index. Thus, we will frustrate the elements in between.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

cut_start_index, cut_end_index := 4, 6
 
edibles = append(edibles[:cut_start_index], edibles[cut_end_index:] ...)

edibles // [🍎 🍇 🍒 🍉 🌽 🥔]
```

### delete

If we want to delete an element in the slice using its index, we can use the following method.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

index_of_cherry := 2
edibles = append(edibles[:index_of_cherry], edibles[index_of_cherry + 1:] ...)

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```


We can do the same with the `copy` function. In the example below, unlike the above, line 4:

* First, we selected the section after the element of éedible in slice. This gave us a slice in the format \ [🍒 🍉 🌶 🍆 🌽 🥔 \], if we show all the capacity.
* Then, after the element `` 1 '' index, we have _ replaced_ with the selected slice. Thus, we have obtained the list of [[🍎 🍇 🍉 🌶 🍆 🌽 🥔 🥔 \].

In line 5, we slice the slice according to the number of new staff.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

n_copied := copy(edibles[index_of_cherry:], edibles[index_of_cherry + 1:])
new_length := index_of_cherry + n_copied

edibles = edibles[:new_length]

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```


#### Delete Without Preserving Order

We can also do the deletion by ignoring the array order. In the example below, we have replaced the last element in the slice with the element we want to delete. Then we reduced the size of the slice by 1.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

edibles[index_of_cherry] = edibles[len(edibles) -1]
edibles = edibles[:len(edibles) -1]

edibles // [🍎 🍇 🥔 🍉 🌶 🍆 🌽]
```

> ** Garbage Collection **
> If the elements we use in slice are `struct` with` pointer` or `pointer` fields, the ** cut ** and ** delete ** operations shown above can create a memory leak. The reason for this is that after slice is deleted, it can keep the reference of the deleted element.

In addition to the operations we have done in the examples below, we filled in the fields that were idle after my process with `nil` values. Thus, garbage collector can understand that these areas will no longer be used and bring them back to the system.

### cut (memory safe)


```go
apple := "🍎"
banana := "🍌"
fruits := []*string {&apple, &apple, &apple, &banana, &banana, &banana}

cut_start_index, cut_end_index := 1, 5

copy(fruits[cut_start_index:], fruits[cut_end_index:])

cleanup_index := len(fruits) - cut_end_index + cut_start_index

for free_index, free_end := cleanup_index, len(fruits); free_index <free_end; free_index ++ {
    fruits[free_index] = nil
}

fruits = fruits[:cleanup_index]

fruits // [🍎 🍌]
```


### delete (memory safe)


```go
apple := "🍎"
banana := "🍌"
fruits := []*string {&apple, &apple, &apple, &banana, &banana, &banana}
delete_index := 2

copy(fruits[delete_index:], fruits[delete_index + 1:])
fruits[len(fruits) -1] = nil
fruits = fruits[:len(fruits) -1]

fruits // [🍎 🍎 🍌 🍌 🍌]
```


#### Delete Without Preserving Order (memory safe)


```go
apple := "🍎"
banana := "🍌"
fruits := []*string {&apple, &apple, &apple, &banana, &banana, &banana}

delete_index := 2

fruits[delete_index] = fruits[len(fruits) -1]
fruits[len(fruits) -1] = nil
fruits = fruits[:len(fruits) -1]

fruits // [🍎 🍎 🍌 🍌 🍌]
```


### expand

If we want to combine one slice and the other, we can use the following methods.


```go
bucket := []string{"🍌", "🍇", "🍉"}
apples := []string{"🍏", "🍎"}

bucket = append(bucket, apples...)

bucket // [🍌 🍇 🍉 🍏 🍎]
```

In the example below, we have combined the elements after the index determined by `apples` in line` 7`. Afterwards, we added these slice elements to the bucket items up to the index we determined.


```go
bucket := []string{"🍌", "🍇", "🍉"}
apples := []string{"🍏", "🍎"}
expand_index := 2

bucket = append(
    bucket [:expand_index]
    append(apples, bucket [expand_index:]...)...,
)

bucket // [🍌 🍇 🍏 🍎 🍉]
```


### filter

We can filter the slice elements according to a certain criterion.


```go
fruits := []string{"🍏", "🍎", "🍉", "🍏", "🍎"}
apple_criteria := "🍎"

apples := []string{}
for _, fruit := range fruits {
    if fruit == apple_criteria {
        apples = append(apples, fruit)
    }
}

apples // [🍎 🍎]
```
### insert

If we want to add a new element to any part of the slice; we can use these methods.


```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"
insert_index := 2

fruits = append(
    fruits[:insert_index],
    append([]string{banana}, fruits[insert_index:]...)...,
)

fruits // [🍏 🍎 🍌 🍉]
```


> ** Memory Concern **
>
> The `append([]string{banana}, fruits[insert_index:] ...)` section in line 7 creates a slice itself and copies the elements of the slice `fruits[insert_index:]` to this slicea. These elements are then copied to the `fruits` slice.
>
> Creating a new slice and copying for the 2nd time can be avoided.



```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"
insert_index := 2

fruits = append(fruits, "")
copy(fruits[insert_index + 1:], fruits[insert_index:])
fruits[insert_index] = banana

fruits // [🍏 🍎 🍌 🍉]
```


### pop

In the example below, we have removed the last element of the slice and assigned it to a variable.


```go
fruits := []string{"🍏", "🍎", "🍉"}

item, fruits := fruits[len(fruits) -1], fruits[:len(fruits) -1]

item // 🍉
fruits // [🍏 🍎]
```


### push front (unshift)

In this example, we put a new element at the beginning of the slice.


```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"

fruits = append([]string{banana}, fruits ...)

fruits // [🍌 🍏 🍎 🍉]
```


### pop front (shift)

In this example, we removed the first element of the slice and assigned it to a variable.


```go
fruits := []string{"🍌", "🍏", "🍎", "🍉"}

banana, fruits := fruits[0], fruits[1:]

banana // 🍌
fruits // [🍏 🍎 🍉]
```


Map
--------


One of the most frequently used data types is maps. `map` holds values with a unique keys.

### Identification

Maps can be defined as `map[KeyType]ValueType` or `make(map[KeyType]ValueType)`.

```go
var mymap = map[string]string{}
```

```go
mymap = make(map[string]string)
```

### Assigning Value


```go
monty_python_films_by_year := map [string] string {
    "1971": "Monty Python and the Holy Grail",
    "1979": "Life of Brian",
}

monty_python_films_by_year["1983"] = "The Meaning of Life"
```


We can access the values ​​we assign using the same key.


```go
film_name := monty_python_films_by_year["1983"]

film_name // The Meaning of Life
```


> When accessing an element of Mapin, we can access an additional value that indicates whether the element is registered in the map.



```go
film_name, registered := monty_python_films_by_year["2019"]

if !registered {
    fmt.Println("Monty python did not relase a film at 2019")
} else {
    fmt.Println(film_name)
}
```


### Accessing to values using with for

Map values are accessible in 2 ways.

1- By using one variable in the for loop, by accessing through keys.


```go
monty_python_films_by_year := map[string]string{
	"1971": "Monty Python and the Holy Grail",
	"1979": "Life of Brian",
	"1983": "The Meaning of Life",
}

for year := range monty_python_films_by_year {
	film_name := monty_python_films_by_year[year]
	fmt.Println(film_name, "was released at", year)
}
```


2- Using 2 variables in the loop, accessing both key and value.


```go
for year, film_name := range monty_python_films_by_year {
	fmt.Println(film_name, "was released at", year)
}

/*
Monty Python and the Holy Grail was released at 1971
Life of Brian was released at 1979
The Meaning of Life was released at 1983
*/
```


### delete

We can use the `delete` function to delete an element on the map.


```go
monty_python_films_by_year := map[string]string{
	"1971": "Monty Python and the Holy Grail",
	"2016": "Batman v Superman",
}

delete(monty_python_films_by_year, "2016")
_, registered := monty_python_films_by_year["2016"]

registered // false

len(monty_python_films_by_year) // 1
```


struct
-------------

Structs are user-defined data structures that can hold different types of data with their names.

### Definition

Structs can be defined as follows.

```go
type StructName struct{
    field fieldType
}
```

We can define the fields of the same type of struct in one line.


```go
type Film struct {
	name, director string
	imdb           float32
	release_year   int
}
```



```go
var new_film = Film{}

new_film // {"" "" 0 0}
```


We can also give tags to the struct fields.


```go
type Film struct {
	Name        string  `json:"film_name"`
	Director    string  `json:"director_name"`
	Imdb        float32 `json:"imdb_score"`
	ReleaseYear int     `json:"year_of_release"`
}
```


Tags are popular among ORMs and serialization libraries that deal with type conversion. These tags can be accessed with the 'reflect` module.

```go
f := Film{}
t := reflect.TypeOf(f)
for _, f := range []string{"Name", "Director", "Imdb", "ReleaseYear"} {
	field, found := t.FieldByName(f)
	if !found {
		continue
	}
	fmt.Println(field.Tag.Get("json"))
}

/*
film_name
director_name
imdb_score
year_of_release
*/
```


### Assigning Value

A struct can be created by giving its parameters in order.

```go
new_film := Film{"Mad Max: Fury Road", "George Miller", 8.1, 2015}

new_film // {Mad Max: Fury Road George Miller 8.1 2015}
```


Similarly, it can be created by specifying the parameters and the values with the separator `:`.

```go
new_film := Film{
		Name:        "Ran",
		Director:    "Akira Kurosawa",
		Imdb:        8.2,
		ReleaseYear: 1985,
}

new_film // {Ran Akira Kurosawa 8.2 1985}
```

If some fields are left blank while creating Struct; these fields are defined by the null values of that data type.


```go
new_film := Film{
	name:         "Batman v Superman",
	director:     "Zack Snyder",
	release_year: 2016,
}

new_film // {Batman v Superman Zack Snyder 0 2016}
```


#### Anonymous struct

In some special cases, we may need to define structs in the function body. We can do this by the following method.

```go
provider_config := struct {
	name    string
	baseUrl string
	apiKey  string
}{
	name:    "mapbox",
	baseUrl: "https://api.mapbox.com/geocoding/v5/",
	apiKey:  "123asds123",
}

provider_config // {mapbox https://api.mapbox.com/geocoding/v5/ 123asds123}
```


#### Anonymous fields

Struct can also be defined without specifying field names. 
The data types specified in such a definition must be different from each other.

```go
type Item struct {
	string  // code
	int     // quantity
	float32 // price
}

item := Item{
	"child-seat-1",
	5,
	20.1,
}

item // {child-seat 5 20.1}
```

#### Promoted fields

A struct can be used in another struct without giving the domain name. In this case, 
the structure inside is accessed by its own name. Based on the example below, `Content` in `Page` can be accessed as `page.Content`.

```go
type Content struct {
	body string
	size int
}

type Page struct {
	nextUrl     string
	previousURL string
	Content
}
```


```go
content := Content{
	body: "<h1>Hi</h1>",
	size: 11,
}

page := Page{
	nextUrl:     "http://.../3",
	previousURL: "http://.../1",
	Content:     content,
}

page // {http://.../3 http://.../1 {<h1>Hi</h1> 11}}

page.Content // {<h1>Hi</h1> 11}
```


## Pointer
Pointer is a variable that holds the memory address of another variable.
  
When a new object is created in runtime, we need to keep it in memory. 
We substitute the space allocated in the memory for the object with variables.

### Definition
Pointers are also variables. The difference from other variables is that it keeps the address of the variable that indicates the location of that data, not any data.

A pointer can be defined using the `var pointer * Type` structure. 
If no value is assigned, the value of the pointer is determined as `nil`.
    
```go
var ptr *int
ptr // nil
```

We can also define pointers using the `new` function.

```go
ptr := new(int)
ptr // nil
```

### Assigning Value

Assigning Value

We can assign the address of another variable as a value to the Pointer by using the `&` sign.

Using the pointer with the `*` sign, we can access the variable it is pointing to (hence the data).

```go
var carbon string = "Carbon"
var ptrCarbon *string

ptrCarbon = &carbon

ptrCarbon // 0xc000092030
*ptrCarbon // "Carbon"

*ptrCarbon = "Altered " + *ptrCarbon

carbon // "Altered Carbon"
```


## Functions

In general, functions in GO are defined in the following structure.

```go
func fonksiyon_name([list of parameters]) [returning data types]
{
   function body
}
```


```go
func sigmoid(value float64) float64 {
	return 1.0 / (1.0 + math.Exp(-value))
}

sigmoid(0.5) // 0.62246
```

### Parameters
Parameters given to functions are copied. 
For this reason, changes made on the parameters do not affect the status of variables outside the function.

```go
type Wallet struct {
	userID  int
	balance float64
	deposit float64
}

func withdraw(wallet Wallet, amount float64) {
	wallet.balance -= amount
}
```

In the above code, we have simply defined a wallet structure and the 'withdraw' function that can withdraw money from the wallet. 
In the lines below, we created a wallet and used it with this function.

```go
wallet := Wallet{
	userID:  1,
	balance: 30000.12,
	deposit: 40000.01,
}

item_price := 10023.0
withdraw(wallet, item_price)

wallet.balance // 3000.12
```

As you can see, the money in the wallet has not changed. 
We can prevent this type of accident in 2 ways.

Method 1, we can return the new balance of the wallet and update the wallet later.
```go
func withdraw(wallet Wallet, amount float64) float64 {
	new_balance := wallet.balance - amount
	return new_balance
}

wallet.balance = withdraw(wallet, item_price)
wallet.balance // 19977.12
```

This change changes the intended use of the withdraw function, 
whose job is to withdraw money from the wallet.

Method 2, Instead of the wallet variable, we can give the wallet's address as a parameter.

```go
func withdraw(wallet *Wallet, amount float64) {
	wallet.balance -= amount
}

withdraw(&wallet, item_price)

wallet.balance // 19977.12
```

The disadvantage of this method is that it is not thread safe. 
When we want to withdraw money from the wallet with different threads, 
it will be difficult to keep track of the balance status.

#### Dynamic parameters

Dynamic parameters can be defined in GO functions. 
To define the parameters of this type, the sign `...` is written before the parameter type.

```go
func ReLU(nums ...float64) []float64 {
	new_nums := make([]float64, len(nums))
	for idx, value := range nums {
		if value > 0 {
			new_nums[idx] = value
		} else {
			new_nums[idx] = 0.
		}
	}
	return new_nums
}
```



```go
nums := []float64{1., 0.2, 0., 0., -0.1, 0.1}

nums = ReLU(nums...)

nums // [1 0.2 0 0 0 0.1]
```


### Return Values

#### Returning multiple values

Multiple values can be returned in GO functions. 
It is necessary to define the values to be returned in order between `()`.

```go
func swap(first_arg, second_arg string) (string, string) {
   return second_arg, first_arg
}
```


```go
var language1, language2 string = "Python", "GO"

language1, language2 = swap(language1, language2)

language1, language2 // GO Python
```


#### Naming Return Values

We can name the return values of the functions and determine the values with these variables. 
Since the named return values will take the null value of the data type, 
they can also be used as the default return values of the function.

```go
type Record struct {
	id   int
	data string
}
type DB struct {
	records []Record
	locked  bool
}

func BulkCreate(records []Record, db *DB) (n_created int, err error) {
	if db.locked {
		err = errors.New("DB is locked")

	} else {
		db.records = append(db.records, records...)
		n_created = len(records)
	}
	return n_created, err
}
```



```go
db := &DB{
	records: []Record{},
	locked:  true,
}

data := []Record{}

for i := 0; i < 100; i++ {
	rec := Record{
		id:   i,
		data: "test",
	}
	data = append(data, rec)
}
n_created, err := BulkCreate(data, db)
if err != nil {
	fmt.Println(err)
}
```

### Anonymous Function

Functions in GO are also variable and can be defined within other functions. 
We can use anonymous functions in a limited part of the project to get a specific job done.
In the code example below, we assumed that we would not need this process elsewhere 
in the project, we used an anonymous function to break up a slice of the specified size.

```go
records := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
batchSize := 3

batchedRecords := func(data []int, batchSize int) [][]int {
	var batches [][]int

	for batchSize < len(data) {
		data, batches = data[batchSize:], append(batches, data[0:batchSize:batchSize])
	}
	batches = append(batches, data)
	return batches
}(records, batchSize)

batchedRecords // [[0 1 2] [3 4 5] [6 7 8] [9]]
```

We can assign  this type of functions to a variable and use it several times.

In the example below, we combine the country code and phone numbers that are sent 
separately from a specific API with the function we define as variables.

```go
type Vendor struct {
	name        string
	countryCode string
	phone       string
}

vendor := Vendor{
	name:        "Volswagen Group",
	countryCode: "1",
	phone:       "403120120",
}

formatPhoneNumbers := func(contact Vendor) string {
	return "+" + contact.countryCode + contact.phone
}

formatPhoneNumbers(vendor) // +1403120120
```

## Methods

There are no classes or inheritance in GO language like other object oriented programming languages. 

An experience similar to object oriented programming can be achieved in GO language 
by adding an additional argument called `receiver 'to the functions. 


```go
type Connection struct {
	host     string
	port     int
	timeOut  int
	isClosed bool
}

func (conn *Connection) Close() {
	conn.isClosed = true
	fmt.Println("Connection", conn.host+":"+strconv.Itoa(conn.port), "closed.")
}
```


In the above structure, we have added `(conn * Connection)` to the `Close` 
function in addition to a normal function. Thus, as we can see in the example below, 
we were able to access the function via the `Connection` type as `c.Close ()`.

```go
c := Connection{
    host:     "0.0.0.0",
    port:     8080,
    timeOut:  1,
    isClosed: false,
}
c.Close() // Connection 0.0.0.0:8080 closed.
```


> #### Pointer receiver | Value receiver
>
> As with the function parameters, it should be decided whether the receivers will be used 
> with the value of the structure or the memory address.


> We can also use receivers for types other than struct.


```go
type Iterable []int

func (iterable *Iterable) Append(items ...int) {
	*iterable = append(*iterable, items...)
}

arr := Iterable{1, 2}
arr.Append(4, 5, 6)

arr // [1 2 4 5 6]
```


## Errors

In GO, we can create our own error types.
To achieve this, the data type must implement the Error method.

### Definition

In the example below, we have defined a new error called 'HTTPError', 
which indicates the HTTP code and the reason for the error.

```go
type HTTPError struct {
	Status int
	Reason string
}

func (e HTTPError) Error() string {
	return fmt.Sprintf("%v: %v", e.Status, e.Reason)
}

func MakeRequest() error {
	return HTTPError{
		Status: 400,
		Reason: "Bad Request",
	}
}
```


### Usa Case

When the function is executed, it can be checked whether the returned error 
value is `nil' and measures can be taken accordingly.

```go
err := MakeRequest()
if err != nil {
	fmt.Println(err)
}
```


## Interface

Interfaces are structures that specify the methods an object can have. 
Therefore, interfaces express the behavior of objects. 
For example; data can be read from or written to a file. 
If an interface contains reading and writing methods and the file object has 
these methods, it can be said that the file has implemented this interface.

The main purpose of Interfaces is to create a common protocol for different data structures 
that show the same behavior but are used in different cases by defining the methods, 
the parameters they take and the return values in a general way; 
.

### Definition

To indicate that one type uses an interface, we do not need to use expressions like `class File implements IO`, 
as in most other object oriented languages. GO establishes the relationship 
between interfaces and types by sharing the same methods.

```go
type IOInterface interface {
	Read() []byte
	Write([]byte) int
}

type File struct {
	name    string
	content []byte
}

func (file *File) Read() []byte {
	return file.content
}

func (file *File) Write(content []byte) int {
	file.content = append(file.content, content...)
	return len(content)
}

type Socket struct {
	ip     string
	port   int
	buffer []byte
}

func (sock *Socket) Read() []byte {
	return sock.buffer
}

func (sock *Socket) Write(stream []byte) int {
	sock.buffer = append(sock.buffer, stream...)
	return len(stream)
}
```


In the above application, we have defined the 'IO' 
interface that defines the 'Read' and 'Write' methods, and the 'File' and 'Socket' types with these methods.

### Use Case

In the example below, since the `Log` method takes the `IO` interface as a parameter,
 it can read the data and do its job without worrying 
 about which data type it comes from.

```go
type Logger struct {
}

func (log *Logger) Log(io IO) {
	for _, data := range io.Read() {
		fmt.Print(string(data))
	}
	fmt.Println()
}
```

```go
file := &File{name: "test"}
sock := &Socket{ip: "0.0.0.0", port: 21}
logger := Logger{}

data := []byte("data")

file.Write(data)
sock.Write(file.Read())

logger.Log(file) // "data"
logger.Log(sock) // "data"
```


### Using Multiple Interfaces

GO types can implement multiple interfaces. 
For this, it only needs to have the methods of the interface.

```go
type Reader interface {
	Read() []byte
}

type Writer interface {
	Write([]byte) int
}

type File struct {
	name    string
	content []byte
}

func (file *File) Read() []byte {
	return file.content
}

func (file *File) Write(content []byte) int {
	file.content = append(file.content, content...)
	return len(content)
}

```


```go
var file Reader = &File{
	name:    "test",
	content: []byte("data"),
}
var sock Writer = &Socket{
	ip:   "0.0.0.0",
	port: 21,
}

sock.Write(file.Read())

sock.buffer // sock.buffer undefined (type Writer has no field or method buffer
```

### Type Conversion

If we need to take advantage of certain properties of data type hierarchies, 
we need to convert assets from one data type to another.

> We can check whether the type we are converting correctly implements 
> that interface with an additional parameter that we can get from conversion.


```go
s := sock.(*Socket)
string(s.buffer) // "data"
```


```go
type Serializer interface {
	Serialize()
}

s, ok := sock.(Serializer)
s, ok // nil, false
```


### Empty Interface

Interfaces are types like other data structures and can be used in the same way. Empty interface in GO projects; It is defined as `interface {}`.

Since it has no method, the requirements of the empty interface are met by all types. This feature of the empty interface allows us to define dynamic parameters and values.

```go
type Booking struct {
	Provider            string
	CustomerID          string
	ExtraProviderParams interface{}
}
```



```go
booking1 := Booking{
	Provider:   "ACL",
	CustomerID: "customer-01a",
	ExtraProviderParams: map[string]interface{}{
		"Passengers":  4,
		"PaymentTime": "after-booking",
	},
}

booking2 := Booking{
	Provider:   "DCM",
	CustomerID: "customer-03f",
	ExtraProviderParams: []interface{}{
		4, "after-booking",
	},
}

booking1 // {ACL customer-01a map[Passengers:4 PaymentTime:after-booking]}
booking2 // {DCM customer-01a [4 after-booking]}
```


### Type Checking

Interface types can be controlled by the switch structure. 

In the example below, the `Stringify` function allows us to use different data types in the function by taking the parameter as `interface {}`.
In this function, we will check the int string and float incoming values according to their types and convert them to string as we determine.

```go
func Stringify(value interface{}) (string, error) {
	switch value.(type) {
	case string:
		return value.(string), nil
	case float32, float64:
		return fmt.Sprintf("%.2f", value), nil
	case int:
		return fmt.Sprintf("%d", value), nil
	default:
		return "", errors.New("Invalid type")
	}
}

Stringify(12) // "12"
Stringify(12.52312313) // "12.52"
Stringify("test")  // "test"
```


Additional Resources
----------------
[https://medium.com/rungo/structures-in-go-76377cc106a2](https://medium.com/rungo/structures-in-go-76377cc106a2)  
[https://golangbot.com/arrays-and-slices/](https://golangbot.com/arrays-and-slices/)  
[https://blog.golang.org/go-slices-usage-and-internals](https://blog.golang.org/go-slices-usage-and-internals)  
[https://gobyexample.com/slices](https://gobyexample.com/slices)  
[https://medium.com/rungo/the-anatomy-of-slices-in-go-6450e3bb2b94](https://medium.com/rungo/the-anatomy-of-slices-in-go-6450e3bb2b94)  
[https://opensource.com/article/18/7/introduction-go-arrays-and-slices](https://opensource.com/article/18/7/introduction-go-arrays-and-slices)  
[https://appliedgo.net/slices/](https://appliedgo.net/slices/)  
[https://gobyexample.com/maps](https://gobyexample.com/maps)  
[https://blog.golang.org/go-maps-in-action](https://blog.golang.org/go-maps-in-action)  
[https://medium.com/rungo/the-anatomy-of-maps-in-go-79b82836838b](https://medium.com/rungo/the-anatomy-of-maps-in-go-79b82836838b)  
[https://github.com/Pungyeon/clean-go-article](https://github.com/Pungyeon/clean-go-article)  
[https://medium.com/rungo/error-handling-in-go-f0125de052f0](https://medium.com/rungo/error-handling-in-go-f0125de052f0)  
[https://github.com/golang/go/wiki/SliceTricks](https://github.com/golang/go/wiki/SliceTricks)  
[https://gobyexample.com/variadic-functions](https://gobyexample.com/variadic-functions)  
[https://github.com/emirpasic/gods](https://github.com/emirpasic/gods)  
[https://github.com/avelino/awesome-go](https://github.com/avelino/awesome-go)  
[https://golangbot.com/pointers/](https://golangbot.com/pointers/)  
[http://www.golang-book.com/books/intro/8](http://www.golang-book.com/books/intro/8)  
[https://medium.com/rungo/the-anatomy-of-arrays-in-go-24429e4491b7](https://medium.com/rungo/the-anatomy-of-arrays-in-go-24429e4491b7)  
[https://medium.com/rungo/achieving-concurrency-in-go-3f84cbf870ca](https://medium.com/rungo/achieving-concurrency-in-go-3f84cbf870ca)  
[https://medium.com/rungo/interfaces-in-go-ab1601159b3a](https://medium.com/rungo/interfaces-in-go-ab1601159b3a)  
[https://medium.com/@dotronglong/interface-naming-convention-in-golang-f53d9f471593](https://medium.com/@dotronglong/interface-naming-convention-in-golang-f53d9f471593)  
[https://yourbasic.org/golang/interfaces-explained/](https://yourbasic.org/golang/interfaces-explained/)  
[https://medium.com/rungo/structures-in-go-76377cc106a2](https://medium.com/rungo/structures-in-go-76377cc106a2)  
[https://talks.golang.org/2012/10things.slide\#9](https://talks.golang.org/2012/10things.slide#9)  
[https://flaviocopes.com/go-empty-interface/](https://flaviocopes.com/go-empty-interface/)  
[https://research.swtch.com/interfaces](https://research.swtch.com/interfaces)  
[https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go](https://jordanorelli.com/post/32665860244/how-to-use-interfaces-in-go)





