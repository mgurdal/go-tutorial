# Map


En sık kullanılan veri tiplerinden birisi de maplerdir. `map` unique anahtarları, değerlerle eşleştirebilen sonrasında bu anahtarla eşleştirilen değere erişilmesini sağlayan yapılardır. 

### Tanımlama

Mapler `map[AnahtarTipi]DeğerTipi` veya `make(map[AnahtarTipi]DeğerTipi)`şeklinde tanımlanabilir.

```go
var mymap = map[string]string{}
```

```go
mymap = make(map[string]string)
```

### Değer Atama


```go
monty_python_films_by_year := map[string]string{
	"1971": "Monty Python and the Holy Grail",
	"1979": "Life of Brian",
}

monty_python_films_by_year["1983"] = "The Meaning of Life"
```


Atadığımız değerlere, yine aynı keyi kullanarak erişebiliriz.


```go
film_name := monty_python_films_by_year["1983"]

film_name // The Meaning of Life
```


> Mapin bir elemanına erişirken, elemanın mapde kayıtlı olup olmadığını gösteren ek bir değere daha erişebiliriz.



```go
film_name, registered := monty_python_films_by_year["2019"]

if !registered {
	fmt.Println("Monty python did not relase a film at 2019")
} else {
	fmt.Println(film_name)
}
```

### Döngü ile kullanma

Map elemanlarına temelde 2 yolla erişilebilir.

1. For döngüsünde tek değişken kullanıp, keyler üzerinden erişerek.


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


2. Döngüde 2 değişken kullanıp, hem anahtar hem de değere erişerek.


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

Mapdeki bir elemanı silmek için delete fonksiyonunu kullanabiliriz.


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
