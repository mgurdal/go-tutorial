# Array

Arrayler aynı tipteki verileri bir arada muhafaza etmemizi sağlayan yapılardır. Örneğin; 4, 5, 19 ve 25'ten oluşan veri bütünü arrayi tanımlar. 

Go farklı tipteki verilerden array oluşturmamıza izin vermez.


### Tanımlama

Arrayler temel olarak `[Boyut]Tip` biçiminde tanımlanır. Arrayleri oluşturmanın çeşitli yollarını aşağıda görebiliriz.

`string` tipinde 3 birim tutabilen array.


```go
var fruits [3]string
```


Böyle tanımlandığında arraydeki her bir alan, veri tipinin **ön tanımlı** değeri ile doldurulur. 
Bu örneği baz alırsak `["", "", ""]` olarak tanımlanacaktır. 
Bu değerleri bulundukları indexler üzerinden erişerek değiştirebiliriz.


```go
fruits[0] = "🍌"
fruits[1] = "🍏"

fruits // [🍌 🍏  ]
```


Array için tanımlananın dışındaki bir veri tipini kullanırsak hata ile karşılaşırız.


```go
arr[2] = 1
```


```bash
cannot use 1 (type int) as type string in assignment
```

Aynı arrayi daha kısa bir yol ile de tanımlayabiliriz.


```go
fruits := [3]string{"🍌", "🍏"}

fruits // [🍌 🍏  ]
```


Bu tanımlamada ilk 2 elemanını doğrudan tanımladığımız 3 birim uzunluğundaki bir arrayi `arr`değişkenine atadık.  
  
Eğer arrayi değerleri ile birlikte oluşturulacaksak `...` ifadesini kullanılarak, boyut hesaplama kısmı compilera bırakılabiliriz.


```go
fruits := [...]string{"🍌", "🍏", "🍉"}

fruits // [🍌 🍏 🍉]
```


Döngü ile kullanma

Arrayler for döngüsü ile temel olarak 2 şekilde kullanılabilir. 

1. İndex ile erişerek.


```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for index := range fruits {
	fruit := fruits[index]
	fmt.Println(fruit)
}
```


2. Doğrudan değere erişerek.


```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for _, fruit := range fruits {
	fmt.Println(fruit)
}
```
