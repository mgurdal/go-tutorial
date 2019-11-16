# Veri Yapıları

## Array

Arrayler aynı tipteki verileri bir arada muhafaza etmemizi sağlayan yapılardır. Örneğin; 4, 5, 19 ve 25'ten oluşan veri bütünü arrayi tanımlar. 

Go farklı tipteki verilerden array oluşturmamıza izin vermez.

### Tanımlama

Arrayler temel olarak `[Boyut]Tip` biçiminde tanımlanır. Arrayleri oluşturmanın çeşitli yollarını aşağıda görebiliriz.

`int` tipinde 3 birim tutabilen array.

{% code title="" %}
```go
var fruits [3]string
```
{% endcode %}

Böyle tanımlandığında arraydeki her bir alan, veri tipinin **ön tanımlı** değeri ile doldurulur. Bu örneği baz alırsak `[0, 0, 0]` olarak tanımlanacaktır. Bu değerleri bulundukları indexler üzerinden erişerek değiştirebiliriz.

{% code title="" %}
```go
fruits[0] = "🍌"
fruits[1] = "🍏"

fruits // [🍌 🍏  ]
```
{% endcode %}

Array için tanımlananın dışındaki bir veri tipini kullanırsak hata ile karşılaşırız.

{% code title="" %}
```go
arr[2] = 1
```
{% endcode %}

```bash
cannot use 1 (type int) as type string in assignment
```

Aynı arrayi daha kısa bir yol ile de tanımlayabiliriz.

{% code title="" %}
```go
fruits := [3]int{"🍌", "🍏"}

fruits // [🍌 🍏  ]
```
{% endcode %}

Bu tanımlamada ilk 2 elemanını doğrudan tanımladığımız 3 birim uzunluğundaki bir arrayi `arr`değişkenine atadık.  
  
Eğer arrayi değerleri ile birlikte oluşturulacaksak `...` ifadesini kullanılarak, boyut hesaplama kısmı compilera bırakılabiliriz.

{% code title="" %}
```go
fruits := [3]int{"🍌", "🍏", "🍉"}

fruits // [🍌 🍏 🍉]
```
{% endcode %}

[https://play.golang.org/p/AlUl2XBUQaH](https://play.golang.org/p/AlUl2XBUQaH)

### Döngü ile kullanma

Arrayler for döngüsü ile temel olarak 2 şekilde kullanılabilir. 

1. İndex ile erişerek.

{% code title="" %}
```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for index := range fruits {
	fruit := fruits[index]
	fmt.Println(fruit)
}
```
{% endcode %}

2. Doğrudan değere erişerek.

{% code title="" %}
```go
fruits := [3]string{"🍌", "🍏", "🍉"}

for _, fruit := range fruits {
	fmt.Println(fruit)
}
```
{% endcode %}

[https://play.golang.org/p/egu-tgfgl1M](https://play.golang.org/p/egu-tgfgl1M)

##  Slice

Slicelar kullanım bakımından arraylere çok benzer. Sliceların farkı, boyutlarının dinamik olması.

Slice tanımlarken yine arraydeki gibi `[]Tip` yapısını kullanıyoruz fakat köşeli parantezlerin içini boş bırakıyoruz.

{% code title="" %}
```go
// nil slice tanımlama
var slc []int
// nill slice'a değer atama
slc = []int{12, 24, 39}  

// doğrudan tanımlama
slc := []int{12, 24, 39} 
```
{% endcode %}

#### make

Sliceları tanımlamak için \`make\` fonksiyonunu da kullanabiliriz. `make` fonksiyonunun ilk parametresi oluşturulacak sliceın boyutunu belirler.

{% code title="" %}
```go
// boyut ile tanımlama
n_items = 20
slc := make([]int, n_items)

// boyut ve kapasite ile tanımlama
capacity := 40
slc := make([]int, n_items, capacity)
```
{% endcode %}

[https://play.golang.org/p/7IibqMN6eqr](https://play.golang.org/p/7IibqMN6eqr)

## Slice İşlemleri

Slice oluşturulurken kapasitesi kadar, tutacağı veri tipinin ön tanımlı değeri ile doldurulur. Fakat, yalnızca uzunluğu kadar kısmına erişilebilir.

Bir sliceın boyutunu artırmak istiyorsak bunun için `append` ve `copy` fonksiyonlarını kullanabiliriz.

### copy

`copy` bir sliceı aynı veya daha büyük kapasiteye sahip bir diğer slicea kopyalamamızı sağlar ve  kaç adet elemanın kopyalandığını döndürür.

{% code title="" %}
```go
bucket := make([]string, 5)

fruits := []string{"🍎", "🍇", "🍒", "🍉"}

n_copied := copy(bucket, fruits)

n_copied // 5
bucket // [🍎 🍇 🍈 🍉]

```
{% endcode %}

[https://play.golang.org/p/x-UJyc\_db4e](https://play.golang.org/p/x-UJyc_db4e)

 Bir slice kopyalanırken, elemanlar hedef slicedaki aynı indexdeki elemanların yerini alır.

{% code title="" %}
```go
fruits := []string{"🍇", "🍒", "🍉"}
apples := []string{"🍎", "",  "🍏"}

copy(fruits, apples)

fruits // [🍎  🍏]

```
{% endcode %}

[https://play.golang.org/p/jwOcc\_Eg6q4](https://play.golang.org/p/jwOcc_Eg6q4)

### append

`append` fonksiyonu, ilk parametre olarak boyutunu artıracağımız sliceı ve devamında ekleyeceğimiz elemanları alır ve bize tüm elemanları içeren **yeni** bir slice döndürür. Bu nedenle, dönüş değerini bir değişkende\(genellikle eski slice değişkeninde\) tutmak gerekir.

{% code title="" %}
```go
bucket := []string{"🍎", "🍇", "🍈", "🍉"}

green_apple := "🍏"
bucket = append(bucket, green_apple)

bucket // [🍎 🍇 🍈 🍉 🍏]
```
{% endcode %}

[https://play.golang.org/p/iEP9-dXWLFc](https://play.golang.org/p/iEP9-dXWLFc)

{% hint style="danger" %}
Bir sliceı yeniden boyutlandırıp başka bir değişkene atadığımızda, yeni slicedaki değişiklikler orjinal sliceı da etkileyecektir.
{% endhint %}

{% code title="" %}
```go
fruits := []string{"🍎", "🍎", "🍇", "🍒", "🍉"}
apples := fruits[:2]

apples[0] = "🍏"
apples[1] = "🍏"
 
apples // [🍏 🍏]
fruits // [🍏 🍏 🍇 🍒 🍉]


```
{% endcode %}

[https://play.golang.org/p/9Fnj6qt9DPh](https://play.golang.org/p/9Fnj6qt9DPh)

### cut

Slicedaki belli bir aralığı çıkarmak istiyorsak aşağıdaki yöntemi kullanabiliriz. Bu yöntem Sliceda belirlediğimiz başlangıç indexine kadar olan elemanlar ile bitiş indexinden sonraki elemanları ekleyecek. Böylece aradaki elemanları boşa çıkartmış olacağız.

{% code title="" %}
```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

cut_start_index, cut_end_index := 4, 6
 
edibles = append(edibles[:cut_start_index], edibles[cut_end_index:]...)

edibles // [🍎 🍇 🍒 🍉 🌽 🥔]
```
{% endcode %}

[https://play.golang.org/p/a-y-pHDVPVt](https://play.golang.org/p/a-y-pHDVPVt)

### delete

Slicedaki bir elemanı indexini kullanarak silmek istiyorsak, aşağıdaki yöntemi kullanabiliriz.

{% code title="" %}
```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

index_of_cherry := 2
edibles = append(edibles[:index_of_cherry], edibles[index_of_cherry+1:]...)

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```
{% endcode %}

[https://play.golang.org/p/nT9nCG\_E8\_K](https://play.golang.org/p/nT9nCG_E8_K)

Benzer işlemi `copy` fonksiyonu ile de yapabiliriz. Aşağıdaki örnekte, yukarıdakinden farklı olarak 4. satırda:

* Önce`edible` sliceında 🍒elemanından sonraki kısmı seçtik.  Bu bize, tüm kapasiteyi de gösterirsek, \[🍒 🍉 🌶 🍆 🌽 🥔\] biçiminde bir slice verdi.
* Sonrasında 🍒elemanından `1` index sonraki kısmı, seçilen slice ile _değiştirdik_. Böylece \[🍎 🍇 🍉 🌶 🍆 🌽 🥔 🥔\] listesini elde etmiş olduk.

5. satırda da sliceı yeni eleman sayısına göre dilimledik.

{% code title="" %}
```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

n_copied := copy(edibles[index_of_cherry:], edibles[index_of_cherry+1:])
new_length := index_of_cherry + n_copied

edibles = edibles[:new_length]

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```
{% endcode %}

[https://play.golang.org/p/IT51dQhIoG3](https://play.golang.org/p/IT51dQhIoG3)

#### Sıralamayı Dikkate Almadan Silme

Silme işlemini array sıralamasını göz ardı ederek de yapabiliriz. Aşağıdaki örnekte, silmek istediğimiz eleman ile slicedaki en son elemanın yerini değiştirdik. Ardından sliceın boyutunu 1 azalttık.

{% code title="" %}
```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

edibles[index_of_cherry] = edibles[len(edibles)-1]
edibles = edibles[:len(edibles)-1]

edibles // [🍎 🍇 🥔 🍉 🌶 🍆 🌽]
```
{% endcode %}

[https://play.golang.org/p/gqjB2c66O6S](https://play.golang.org/p/gqjB2c66O6S)

{% hint style="danger" %}
**Garbage Collection**

Eğer sliceda kullandığımız elemanlar `pointer` veya `pointer` alanlara sahip `struct` ise yukarıda gösterilen **cut**  ve **delete** işlemleri memory leak oluşturabilir. Bunun sebebi, sliceın silindikten sonra da silinen elemanın referansını tutabilmesindendir. 
{% endhint %}

Aşağıdaki örneklerde daha öncesinde yaptığımız işlemlere ek olarak, işlemdem sonra boşta kalan alanları `nil` değerler ile doldurduk. Böylece garbage collector bu alanların artık kullanılmayacağını anlayıp, sisteme geri kazandırabilir.

### cut \(memory safe\)

{% code title="" %}
```go
apple := "🍎"
banana := "🍌"
fruits := []*string{&apple, &apple, &apple, &banana, &banana, &banana}

cut_start_index, cut_end_index := 1, 5

copy(fruits[cut_start_index:], fruits[cut_end_index:])

cleanup_index := len(fruits) - cut_end_index + cut_start_index

for free_index, free_end := cleanup_index, len(fruits); free_index < free_end; free_index++ {
	fruits[free_index] = nil
}

fruits = fruits[:cleanup_index]

fruits // [🍎 🍌]
```
{% endcode %}

[https://play.golang.org/p/NuHHVJe3OUx](https://play.golang.org/p/NuHHVJe3OUx)

### delete \(memory safe\)

{% code title="" %}
```go
apple := "🍎"
banana := "🍌"
fruits := []*string{&apple, &apple, &apple, &banana, &banana, &banana}
delete_index := 2

copy(fruits[delete_index:], fruits[delete_index+1:])
fruits[len(fruits)-1] = nil
fruits = fruits[:len(fruits)-1]

fruits // [🍎 🍎 🍌 🍌 🍌]
```
{% endcode %}

[https://play.golang.org/p/4EclBmk-4Iz](https://play.golang.org/p/4EclBmk-4Iz)

#### Sıralamayı Dikkate Almadan Silme \(memory safe\)

{% code title="" %}
```go
apple := "🍎"
banana := "🍌"
fruits := []*string{&apple, &apple, &apple, &banana, &banana, &banana}

delete_index := 2

fruits[delete_index] = fruits[len(fruits)-1]
fruits[len(fruits)-1] = nil
fruits = fruits[:len(fruits)-1]

fruits // [🍎 🍎 🍌 🍌 🍌]
```
{% endcode %}

[https://play.golang.org/p/tnaHZL1IGWB](https://play.golang.org/p/tnaHZL1IGWB)

### expand

Bir sliceı, diğeri birleştirmek istersek aşağıdaki yöntemleri kullanabiliriz.

{% code title="" %}
```go
bucket := []string{"🍌", "🍇", "🍉"}
apples := []string{"🍏", "🍎"}

bucket = append(bucket, apples...)

bucket // [🍌 🍇 🍉 🍏 🍎]
```
{% endcode %}

[https://play.golang.org/p/PRfDcarZBbj](https://play.golang.org/p/PRfDcarZBbj)

Aşağıdaki örnekte `7.` satırda `apples` ile belirlediğimiz indexten sonraki elemanları birleştirdik. Sonrasında belirlediğimiz indexe kadar olan `bucket` elemanlarına, bu slice elemanlarını ekledik.

{% code title="" %}
```go
bucket := []string{"🍌", "🍇", "🍉"}
apples := []string{"🍏", "🍎"}
expand_index := 2

bucket = append(
	bucket[:expand_index],
	append(apples, bucket[expand_index:]...)...,
)

bucket // [🍌 🍇 🍏 🍎 🍉]
```
{% endcode %}

[https://play.golang.org/p/u4oLZcGgZOi](https://play.golang.org/p/u4oLZcGgZOi)

### filter

Slice elemanlarını belirli bir kritere göre filtreleyebiliriz.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/IXswvVnq75h](https://play.golang.org/p/IXswvVnq75h)

### insert

Sliceın herhangi bir kısmına yeni bir eleman eklemek istiyorsak; bu yöntemleri kullanabiliriz.

{% code title="" %}
```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"
insert_index := 2

fruits = append(
	fruits[:insert_index],
	append([]string{banana}, fruits[insert_index:]...)...,
)

fruits  // [🍏 🍎 🍌 🍉]
```
{% endcode %}

[https://play.golang.org/p/zaYkSLSN982](https://play.golang.org/p/zaYkSLSN982)

{% hint style="warning" %}
**Memory Concern**

7. satırdaki`append([]string{banana}, fruits[insert_index:]...)` ****bölümü başlı başına bir slice oluşturup,`fruits[insert_index:]`sliceının elemanlarını bu slicea kopyalıyor. Bu elemanlar daha sonra `fruits` sliceına tekrar kopyalanıyor.

Yeni bir slice oluşturulmasının ve 2. kez kopyalama işleminin önüne geçilebilir.
{% endhint %}

{% code title="" %}
```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"
insert_index := 2

fruits = append(fruits, "")
copy(fruits[insert_index+1:], fruits[insert_index:])
fruits[insert_index] = banana

fruits // [🍏 🍎 🍌 🍉]
```
{% endcode %}

[https://play.golang.org/p/P3jN12u9BVR](https://play.golang.org/p/P3jN12u9BVR)

### pop

Aşağıdaki örnekte sliceın son elemanını çıkarıp bir değişkene atadık.

{% code title="" %}
```go
fruits := []string{"🍏", "🍎", "🍉"}

item, fruits := fruits[len(fruits)-1], fruits[:len(fruits)-1]

item // 🍉
fruits // [🍏 🍎]
```
{% endcode %}

[https://play.golang.org/p/rPcKLeJS9Ci](https://play.golang.org/p/rPcKLeJS9Ci)

### push front \(unshift\)

Bu örnekte yeni bir elemanı sliceın başına koyduk.

{% code title="" %}
```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"

fruits = append([]string{banana}, fruits...)

fruits // [🍌 🍏 🍎 🍉]
```
{% endcode %}

[https://play.golang.org/p/i5YdcdSBUaa](https://play.golang.org/p/i5YdcdSBUaa)

### pop front \(shift\)

Bu örnekte de sliceın ilk elemanını çıkarıp bir değişkene atadık.

{% code title="" %}
```go
fruits := []string{"🍌", "🍏", "🍎", "🍉"}

banana, fruits := fruits[0], fruits[1:]

banana // 🍌
fruits // [🍏 🍎 🍉]
```
{% endcode %}

[https://play.golang.org/p/-jUyuax1gF5](https://play.golang.org/p/-jUyuax1gF5)

## Map

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

{% code title="" %}
```go
monty_python_films_by_year := map[string]string{
	"1971": "Monty Python and the Holy Grail",
	"1979": "Life of Brian",
}

monty_python_films_by_year["1983"] = "The Meaning of Life"
```
{% endcode %}

Atadığımız değerlere, yine aynı keyi kullanarak erişebiliriz.

{% code title="" %}
```go
film_name := monty_python_films_by_year["1983"]

film_name // The Meaning of Life
```
{% endcode %}

[https://play.golang.org/p/a4M0DX9-3-6](https://play.golang.org/p/a4M0DX9-3-6)

{% hint style="success" %}
Mapin bir elemanına erişirken, elemanın mapde kayıtlı olup olmadığını gösteren ek bir değere daha erişebiliriz.
{% endhint %}

{% code title="" %}
```go
film_name, registered := monty_python_films_by_year["2019"]

if !registered {
	fmt.Println("Monty python did not relase a film at 2019")
} else {
	fmt.Println(film_name)
}
```
{% endcode %}

[https://play.golang.org/p/A5SE4cYCE-q](https://play.golang.org/p/A5SE4cYCE-q)

### Döngü ile kullanma

Map elemanlarına temelde 2 yolla erişilebilir.

1. For döngüsünde tek değişken kullanıp, keyler üzerinden erişerek.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/q0Zfw7dM5Qt](https://play.golang.org/p/q0Zfw7dM5Qt)

2. Döngüde 2 değişken kullanıp, hem anahtar hem de değere erişerek.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/q0Zfw7dM5Qt](https://play.golang.org/p/q0Zfw7dM5Qt)

### delete

Mapdeki bir elemanı silmek için delete fonksiyonunu kullanabiliriz.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/Ir0oqibITHN](https://play.golang.org/p/Ir0oqibITHN)

## struct

Structlar kullanıcı tarafından tanımlanan ve farklı türdeki verileri isimleri ile tutabilen veri yapılarıdır.

### Tanımlama

Structlar aşağıdaki yapıda tanımlanabilir.

```go
type StructName struct{
    field fieldType
}
```

Structın aynı tipteki alanlarını tek satırda tanımlayabiliriz.

{% code title="" %}
```go
type Film struct {
	name, director string
	imdb           float32
	release_year   int
}
```
{% endcode %}

{% code title="" %}
```go
var new_film = Film{}

new_film // {"" "" 0 0}
```
{% endcode %}

[https://play.golang.org/p/SQJejRKGu0c](https://play.golang.org/p/SQJejRKGu0c)

Struct alanlarına ek olarak etiket de verebiliriz.

{% code title="" %}
```go
type Film struct {
	Name        string  `json:"film_name"`
	Director    string  `json:"director_name"`
	Imdb        float32 `json:"imdb_score"`
	ReleaseYear int     `json:"year_of_release"`
}
```
{% endcode %}

Etiketler tip dönüşümü ile çokça uğraşan ORMler ve serialization kütüphaneleri arasında popülerdir. Bu etiketlere `reflect` modülü ile erişilebilir.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/rgeIJX0Bu4Q](https://play.golang.org/p/rgeIJX0Bu4Q)

### Değer Atama

Bir struct, parametreleri sırasıyla verilerek oluşturulabilir.

{% code title="" %}
```go
new_film := Film{"Mad Max: Fury Road", "George Miller", 8.1, 2015}

new_film // {Mad Max: Fury Road George Miller 8.1 2015}
```
{% endcode %}

Benzer biçimde,  parametreleri ve alacakları değerleri `:` ayracı ile belirterek de oluşturulabilir.

{% code title="" %}
```go
new_film := Film{
		Name:        "Ran",
		Director:    "Akira Kurosawa",
		Imdb:        8.2,
		ReleaseYear: 1985,
}

new_film // {Ran Akira Kurosawa 8.2 1985}
```
{% endcode %}

Struct oluştururken bazı alanlar boş bırakılırsa; bu alanlar o veri tipinin boş değerleri ile tanımlanır.

{% code title="" %}
```go
new_film := Film{
	name:         "Batman v Superman",
	director:     "Zack Snyder",
	release_year: 2016,
}

new_film // {Batman v Superman Zack Snyder 0 2016}
```
{% endcode %}

[https://play.golang.org/p/QqiY\_bguN7R](https://play.golang.org/p/QqiY_bguN7R)

#### Anonymous struct

Bazı özel durumlarda structları fonksiyon gövdesi içerisinde tanımlamamız gerekebilir. Bunu aşağıdaki yöntemle yapabiliriz.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/6eBRXTgWRMO](https://play.golang.org/p/6eBRXTgWRMO)

#### Anonymous fields

Alan isimlerini belirtilmeden de struct tanımlanabilir. Bu tarz bir tanımlamada belirtilen veri tiplerinin birbirinden farklı olması gerekir.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/G6uUy5jvAkx](https://play.golang.org/p/G6uUy5jvAkx)

#### Promoted fields

Bir struct başka bir struct içerisinde alan ismi verilmeden kullanılabilir. Bu durumda içerideki structa kendi ismiyle erişilir. Aşağıdaki örneği baz alırsak `Page` içerisindeki `Content`e `page.Content` şeklinde erişilebilir.

{% code title="" %}
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
{% endcode %}

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/Co-j-ybYBcA](https://play.golang.org/p/Co-j-ybYBcA)

## Pointer

Pointer bir başka değişkenin memory adresini tutan bir değişkendir.  
  
Çalışma esnasında yeni bir nesne oluşturulduğunda, bu nesneyi hafızada tutmamız gerekir. Nesne için hafızada ayrılan alanın yerini değişkenler ile tutarız.

### Tanımlama

Pointer da değişkenlerdir. Diğer değişkenlerden farkı herhangi bir veriyi değil, o verinin yerini belirten değişkenin adresini tutmasıdır.

Pointerlar `var pointer *Type` yapısı kullanılarak tanımlanır. Değer atanmadığı takdirde pointerin değeri `nil` olarak belirlenir.

```go
var ptr *int
ptr // nil
```

Pointerları `new` fonksiyonunu kullanarak da tanımlayabiliriz.

```go
ptr := new(int)
ptr // nil
```

### Değer Atama

Pointera `&` işaretini kullanarak, bir başka değişlenin adresini, değer olarak atayabiliriz. 

`*` işareti ile pointeri kullanarak, işaret ettiği değişkene\(dolayısıyla tutulan veriye\) erişebiliriz.

{% code title="" %}
```go
var carbon string = "Carbon"
var ptrCarbon *string

ptrCarbon = &carbon

ptrCarbon // 0xc000092030
*ptrCarbon // "Carbon"

*ptrCarbon = "Altered " + *ptrCarbon

carbon // "Altered Carbon"
```
{% endcode %}

[https://play.golang.org/p/jbLsd4zyryB](https://play.golang.org/p/jbLsd4zyryB)

## Fonksiyonlar

Genel olarak GO'da fonksiyonlar aşağıdaki yapıda tanımlanır.

```go
func fonksiyon_adı( [parametre listesi] ) [dönüş değerlerinin tipleri]
{
   fonksiyon gövdesi
}
```

{% code title="" %}
```go
func sigmoid(value float64) float64 {
	return 1.0 / (1.0 + math.Exp(-value))
}

sigmoid(0.5) // 0.62246
```
{% endcode %}

[https://play.golang.org/p/HHYqXH8BqIx](https://play.golang.org/p/HHYqXH8BqIx)

### Parametreler

GO fonksiyonlarına parametre olarak girilen değişkenler kopyalanır. Bu nedenle parametreler üzerinde yapılan değişiklikler değişkenlerin fonksiyon dışındaki durumlarını etkilemez.

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

Yukarıdaki kodda basitçe bir cüzdan yapısı ve bu cüzdandan para çekebilen `withdraw` fonksiyonu tanımladık. Aşağıdaki satırlarda da bir adet cüzdan oluşturup bu fonksiyon ile kullandık.

{% code title="" %}
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
{% endcode %}

Görüldüğü üzere cüzdandaki para değişmedi. Bu tarz kazaları 2 yolla önleyebiliriz.

1. Yöntem, cüzdandaki son para durumunu döndürüp, cüzdanı sonradan güncelleyebiliriz.

{% code title="" %}
```go
func withdraw(wallet Wallet, amount float64) float64 {
	new_balance := wallet.balance - amount
	return new_balance
}

wallet.balance = withdraw(wallet, item_price)
wallet.balance // 19977.12
```
{% endcode %}

Bu değişim, işi _cüzdandan para çekmek_ olan withdraw fonksiyonunun kullanım amacını değiştirmiş oluyor.

2. Yöntem, cüzdan değişkeni yerine, cüzdanın adresini parametre olarak verebiliriz.

{% code title="" %}
```go
func withdraw(wallet *Wallet, amount float64) {
	wallet.balance -= amount
}

withdraw(&wallet, item_price)

wallet.balance // 19977.12
```
{% endcode %}

[https://play.golang.org/p/lSuk\_GI88gW](https://play.golang.org/p/lSuk_GI88gW)

Bu yöntemin dezavantajı da farklı concurrency safe olmamasıdır. Farklı threadler ile cüzdandan para çekmek istediğimizde, bakiye durumunu takip etmek zor olacaktır.

#### Dinamik parametreler

GO fonksiyonlarında dinamik parametreler tanımlanabilir. Bu türdeki parametreleri tanımlamak için parametre tipinin önüne `...` işareti yazılır.

{% code title="" %}
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
{% endcode %}

{% code title="" %}
```go
nums := []float64{1., 0.2, 0., 0., -0.1, 0.1}

nums = ReLU(nums...)

nums // [1 0.2 0 0 0 0.1]
```
{% endcode %}

[https://play.golang.org/p/Zf0RJXUcnXx](https://play.golang.org/p/Zf0RJXUcnXx)

### Dönüş Değerleri

#### Birden fazla değer döndürme

GO fonksiyonlarında birden fazla değer döndürülebilir. Dönülecek değerleri `()` arasında sırasıyla tanımlamak gerekmektedir.

{% code title="" %}
```go
func swap(first_arg, second_arg string) (string, string) {
   return second_arg, first_arg
}
```
{% endcode %}

{% code title="" %}
```go
var language1, language2 string = "Python", "GO"

language1, language2 = swap(language1, language2)

language1, language2 // GO Python
```
{% endcode %}

[https://play.golang.org/p/6YdcqTIhJch](https://play.golang.org/p/6YdcqTIhJch)

#### Dönüş değerlerini isimlendirme

Fonksiyonların dönüş değerlerine isim verip, değerleri bu değişkenler ile belirleyebiliriz. İsimlendirilmiş dönüş değerleri veri tipinin boş değerini alacağı için, fonksiyonun ön tanımlı dönüş değerleri olarak da kullanılabilirler.

{% code title="" %}
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
{% endcode %}

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/GftXylldL1T](https://play.golang.org/p/GftXylldL1T)

### Anonim Fonksiyon

GO'da fonksiyonlar da veri yapılarıdır ve diğer fonksiyonların içerisinde tanımlanabilirler. Anonim fonksiyonları, projenin kısıtlı bir bölümünde, spesifik bir işi halletmek için kullanabiliriz.

Aşağıdaki kod örneğinde, projenin başka yerinde bu işleme ihtiyaç duymayacağımızı varsayarak, bir sliceı belirlenen boyutta parçalayan bir anonim fonksiyon kullandık.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/nKR8t8P\_dpA](https://play.golang.org/p/nKR8t8P_dpA)

Bu türdeki fonksiyonları bir değişkene atayıp bir kaç kez de kullanabiliriz.

Aşağıdaki örnekte spesifik bir APIden ayrı ayrı gönderilen ülke kodu ve telefon numaralarını değişken olarak tanımladığımız fonksiyonla birleştirdik.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/AZiShOLjY7s](https://play.golang.org/p/AZiShOLjY7s)

## Metodlar

GO dilinde diğer nesne yönelimli programlama dilleri gibi classlar ve kalıtım yoktur. Fakat fonksiyonlara `receiver` denilen ek bir argüman ekleyerek ile kendi oluşturduğumuz tiplere fonksiyonları bağlayabiliriz. Böylece GO dilinde de nesne yönelimli programlamaya benzer bir deneyim yakalanabilir.

{% code title="" %}
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
{% endcode %}

Yukarıdaki yapıda `Close` fonksiyonuna normal bir fonksiyona ek olarak `(conn *Connection)` kısmını ekledik. Böylece aşağıdaki örnekte görüldüğü gibi `c.Close()`  şeklinde`Connection` tipi üzerinden fonksiyona erişebildik.

{% code title="" %}
```go
c := Connection{
    host:     "0.0.0.0",
    port:     8080,
    timeOut:  1,
    isClosed: false,
}
c.Close() // Connection 0.0.0.0:8080 closed.
```
{% endcode %}

[https://play.golang.org/p/Vqx-Sr48-BO](https://play.golang.org/p/Vqx-Sr48-BO)

{% hint style="warning" %}
#### Pointer receiver \| Value receiver

Fonksiyon parametrelerinde olduğu gibi receiverlerde de yapının değeri ile mi yoksa memory adresi ile mi kullanılacağına karar verilmelidir.
{% endhint %}

{% hint style="success" %}
Receiverları struct dışındaki tipler için de kullanabiliriz.
{% endhint %}

{% code title="" %}
```go
type Iterable []int

func (iterable *Iterable) Append(items ...int) {
	*iterable = append(*iterable, items...)
}

arr := Iterable{1, 2}
arr.Append(4, 5, 6)

arr // [1 2 4 5 6]
```
{% endcode %}

[https://play.golang.org/p/oXVXqs2ue7m](https://play.golang.org/p/oXVXqs2ue7m)

## Error

GOda kendi hata tiplerimizi oluşturabiliriz. Bunun için oluşturduğumuz tipin sadece Error methoduna sahip olması yeterlidir.

### Tanımlama

Aşağıdaki örnekte HTTP kodunu ve hata nedenini belirten `HTTPError` adında yeni bir hata tanımladık.

{% code title="" %}
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
{% endcode %}

### Kullanım

Fonksiyon çalıştırıldığında dönülen hata değerinin `nil` olup olmadığı kontrol edilerek buna göre önlemler alınabilir.

{% code title="" %}
```go
err := MakeRequest()
if err != nil {
	fmt.Println(err)
}
```
{% endcode %}

[https://play.golang.org/p/Ko0tuhYSiMT](https://play.golang.org/p/Ko0tuhYSiMT)

## Interface

Interfaceler bir objenin sahip olabileceği methodları belirten yapılardır. Bu nedenle interfaceler objelerin davranışını ifade eder. Örneğin; bir dosyandan veri okunabilir veya dosyaya veri yazılabilir. Bir interface okuma ve yazma methodlarını barındırıyorsa ve dosya objesi bu methodlara sahipse dosyanın bu interfacei `implement` ettiği söylenebilir.

Interfacelerin asıl amacı methodları, aldıkları parametreleri ve dönüş değerlerini genel bir şekilde tanımlayarak; aynı davranışı gösteren, fakat farklı işlerde kullanılan yapılar için ortak bir protokol oluşturmaktır.

### Tanımlama

GO dilinde interfaceler objelerle _kapalı_ halde uygulanır. Başka bir deyişle; bir tipin bir interfacei kullandığını belirtmek için, diğer çoğu nesne yönelimli dillerde olduğu gibi,`class File implements IO` gibi deyimler kullanmamız gerekmez. GO aynı metodları paylaşan interface ve tipler arasındaki ilişkiyi kendisi kurar.

{% code title="" %}
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
{% endcode %}

{% code title="" %}
```go
file := File{
	name: "test",
}
sock := Socket{
	ip:   "0.0.0.0",
	port: 21,
}

data := []byte("data")
file.Write(data)

sock.Write(file.Read())

string(sock.Read() // "data"
```
{% endcode %}

[https://play.golang.org/p/LAfYuRpt83d](https://play.golang.org/p/LAfYuRpt83d)

Yukarıdaki uygulamada `Read` ve `Write` metodlarını tanımlayan `IOInterface` interfaceini ve bu methodlara sahip`File` ve `Socket` tiplerini tanımladık.

### Kullanım

Aşağıdaki örnekte `Log` methodu `IOInterface` interfaceini parametre olarak aldığı için hangi veri tipinden geldiğini önemsemeden veriyi okuyup, işini yapabilir.

{% code title="" %}
```go
type Logger struct {
}

func (log *Logger) Log(io IOInterface) {
	for _, data := range io.Read() {
		fmt.Print(string(data))
	}
	fmt.Println()
}
```
{% endcode %}

{% code title="" %}
```go
file := &File{
	name: "test",
}
sock := &Socket{
	ip:   "0.0.0.0",
	port: 21,
}
logger := Logger{}

data := []byte("data")

file.Write(data)
sock.Write(file.Read())

logger.Log(file) // "data"
logger.Log(sock) // "data"
```
{% endcode %}

[https://play.golang.org/p/vJOUw8SA9TP](https://play.golang.org/p/vJOUw8SA9TP)

### Birden Fazla interface kullanma

GO tipleri birden fazla interfacei implement edebilir. Bunun için interfacein belirttiği metodlara sahip olmaları yeterlidir.  

{% code title="" %}
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
{% endcode %}

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/8VrzKhJWS\_7](https://play.golang.org/p/8VrzKhJWS_7)

### Tip dönüşümü

{% code title="" %}
```go
s := sock.(*Socket)
string(s.buffer) // "data"
```
{% endcode %}

[https://play.golang.org/p/IWQsEI8\_F7W](https://play.golang.org/p/IWQsEI8_F7W)

{% hint style="info" %}
Dönüştürdüğümüz tipin o interfacei doğru bir şekilde implement edip etmediğini dönüşüm sırasında alacağımız ek bir parametre ile kontrol edebiliriz.
{% endhint %}

{% code title="" %}
```go
type Serializer interface {
	Serialize()
}

s, ok := sock.(Serializer)
s, ok // nil, false
```
{% endcode %}

[https://play.golang.org/p/ELy4NtwL7vm](https://play.golang.org/p/ELy4NtwL7vm)

### Boş interface

Interfaceler de diğer veri yapıları gibi tiplerdir ve aynı şekilde kullanılabilir. GO projelerinde boş interface; `interface{}` şeklinde tanımlanır. 

Hiçbir metodu olmadığı için boş interfacein gereksinimleri bütün tipler tarafından karşılanır. boş interfacein bu özelliği, dinamik parametreler ve değerler tanımlamamıza olanak tanır.

{% code title="" %}
```go
type Booking struct {
	Provider            string
	CustomerID          string
	ExtraProviderParams interface{}
}
```
{% endcode %}

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/aGClLRNzlHA](https://play.golang.org/p/aGClLRNzlHA)

### Tip Kontrolü

Interface tipleri `switch` yapısı ile kontrol edilebilir. Aşağıdaki örnekte `Stringify` fonksiyonu`interface{}` tipinde parametre olarak alarak farklı veri tiplerini fonksiyonda kullanmamıza olanak sağlıyor. 

Bu fonksiyonda int string ve float gelen değerleri tiplerine göre kontrol edip belirlediğimiz şekilde stringe çevireceğiz.

{% code title="" %}
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
{% endcode %}

[https://play.golang.org/p/6f008J1S\_wM](https://play.golang.org/p/6f008J1S_wM)

## Containers

Hemen hemen her dilde standart hale gelmiş ve programlarda çokça kullanılan bazı kompozit veri yapıları vardır. Go bu yapılardan 3ünü bize **container** paketi içinde hazır halde sunuyor.

### List

### Heap

Heap veri yapısı, istediğimiz tipteki elemanları öncelik sırasına göre tutarak; üzerilerinde işlem yapmamızı sağlar.

Heap temelde bir interfacedir ve bir veri yapısının heapi implement edebilmesi için `Len`, `Less` , `Swap` , `Push` ve `Pop` methodlarına sahip olması gerekir.

Örnek olarak sadece int tipinde sayıları tutan bir slice oluşturalım ve bunu sliceı heap olarak kullanalım.

```go
type IntHeap []int
```

Öncelikle heap interfaceinin `Len` metodunu implement edelim. Sliceların uzunluğunu len fonksiyonuyla doğrudan alabildiğimiz için bu metodu yazmamız çok da zor olmayacak.

```go
func (h IntHeap) Len() int {
    return len(h)
}
```

Şimdi de heapdeki iki elemanı birbiriyle kıyaslayan ve yerlerini değiştiren `Less` ve Swap metodlarını tanımlayalım.

```go
func (h IntHeap) Less(i, j int) bool {
    return h[i] < h[j]
}

func (h IntHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}
```

Son olarak eleman eklememizi ve çıkarmamızı sağlayacak `Push` ve `Pop` metodlarını yazalım. Bu methodlar, sliceın uzunluğunu etkilediği için _pointer receiver_ olarak tanımlanması gerekiyor.

```go
func (h *IntHeap) Push(x interface{}) {
	*h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
	old := *h
	n := len(old)
	x := old[n-1]
	*h = old[0 : n-1]
	return x
}
```

IntHeap, bir heapin olmayı gerektiren tüm özelliklere sahip. Artık **container/heap** paketini kullanarak bu sliceı heape çevirebiliriz.

```go
func main() {
	h := &IntHeap{2, 1, 5}
	heap.Init(h)
	heap.Push(h, 3)
	fmt.Printf("minimum: %d\n", (*h)[0])
	
	for h.Len() > 0 {
		fmt.Printf("%d ", heap.Pop(h))
	}
	// Output:
	// minimum: 1
	// 1 2 3 5
}
```

### Ring



## Ek Kaynaklar

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





