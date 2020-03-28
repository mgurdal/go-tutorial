# Struct


Structlar kullanıcı tarafından tanımlanan ve farklı türdeki verileri isimleri ile tutabilen veri yapılarıdır.

### Tanımlama

Structlar aşağıdaki yapıda tanımlanabilir.

```go
type StructName struct{
    field fieldType
}
```

Structın aynı tipteki alanlarını tek satırda tanımlayabiliriz.


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



Struct alanlarına ek olarak etiket de verebiliriz.


```go
type Film struct {
	Name        string  `json:"film_name"`
	Director    string  `json:"director_name"`
	Imdb        float32 `json:"imdb_score"`
	ReleaseYear int     `json:"year_of_release"`
}
```


Etiketler tip dönüşümü ile çokça uğraşan ORMler ve serialization kütüphaneleri arasında popülerdir. Bu etiketlere `reflect` modülü ile erişilebilir.


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


### Değer Atama

Bir struct, parametreleri sırasıyla verilerek oluşturulabilir.


```go
new_film := Film{"Mad Max: Fury Road", "George Miller", 8.1, 2015}

new_film // {Mad Max: Fury Road George Miller 8.1 2015}
```


Benzer biçimde,  parametreleri ve alacakları değerleri `:` ayracı ile belirterek de oluşturulabilir.


```go
new_film := Film{
		Name:        "Ran",
		Director:    "Akira Kurosawa",
		Imdb:        8.2,
		ReleaseYear: 1985,
}

new_film // {Ran Akira Kurosawa 8.2 1985}
```


Struct oluştururken bazı alanlar boş bırakılırsa; bu alanlar o veri tipinin boş değerleri ile tanımlanır.


```go
new_film := Film{
	name:         "Batman v Superman",
	director:     "Zack Snyder",
	release_year: 2016,
}

new_film // {Batman v Superman Zack Snyder 0 2016}
```


#### Anonymous struct

Bazı özel durumlarda structları fonksiyon gövdesi içerisinde tanımlamamız gerekebilir. Bunu aşağıdaki yöntemle yapabiliriz.


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

Alan isimlerini belirtilmeden de struct tanımlanabilir. Bu tarz bir tanımlamada belirtilen veri tiplerinin birbirinden farklı olması gerekir.


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

Bir struct başka bir struct içerisinde alan ismi verilmeden kullanılabilir. Bu durumda içerideki structa kendi ismiyle erişilir. Aşağıdaki örneği baz alırsak `Page` içerisindeki `Content`e `page.Content` şeklinde erişilebilir.


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


```go
var carbon string = "Carbon"
var ptrCarbon *string

ptrCarbon = &carbon

ptrCarbon // 0xc000092030
*ptrCarbon // "Carbon"

*ptrCarbon = "Altered " + *ptrCarbon

carbon // "Altered Carbon"
```


## Fonksiyonlar

Genel olarak GO'da fonksiyonlar aşağıdaki yapıda tanımlanır.

```go
func fonksiyon_adı( [parametre listesi] ) [dönüş değerlerinin tipleri]
{
   fonksiyon gövdesi
}
```


```go
func sigmoid(value float64) float64 {
	return 1.0 / (1.0 + math.Exp(-value))
}

sigmoid(0.5) // 0.62246
```

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


Görüldüğü üzere cüzdandaki para değişmedi. Bu tarz kazaları 2 yolla önleyebiliriz.

1. Yöntem, cüzdandaki son para durumunu döndürüp, cüzdanı sonradan güncelleyebiliriz.


```go
func withdraw(wallet Wallet, amount float64) float64 {
	new_balance := wallet.balance - amount
	return new_balance
}

wallet.balance = withdraw(wallet, item_price)
wallet.balance // 19977.12
```


Bu değişim, işi _cüzdandan para çekmek_ olan withdraw fonksiyonunun kullanım amacını değiştirmiş oluyor.

2. Yöntem, cüzdan değişkeni yerine, cüzdanın adresini parametre olarak verebiliriz.


```go
func withdraw(wallet *Wallet, amount float64) {
	wallet.balance -= amount
}

withdraw(&wallet, item_price)

wallet.balance // 19977.12
```


Bu yöntemin dezavantajı da farklı concurrency safe olmamasıdır. Farklı threadler ile cüzdandan para çekmek istediğimizde, bakiye durumunu takip etmek zor olacaktır.

#### Dinamik parametreler

GO fonksiyonlarında dinamik parametreler tanımlanabilir. Bu türdeki parametreleri tanımlamak için parametre tipinin önüne `...` işareti yazılır.


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


### Dönüş Değerleri

#### Birden fazla değer döndürme

GO fonksiyonlarında birden fazla değer döndürülebilir. Dönülecek değerleri `()` arasında sırasıyla tanımlamak gerekmektedir.


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


#### Dönüş değerlerini isimlendirme

Fonksiyonların dönüş değerlerine isim verip, değerleri bu değişkenler ile belirleyebiliriz. İsimlendirilmiş dönüş değerleri veri tipinin boş değerini alacağı için, fonksiyonun ön tanımlı dönüş değerleri olarak da kullanılabilirler.


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

### Anonim Fonksiyon

GO'da fonksiyonlar da veri yapılarıdır ve diğer fonksiyonların içerisinde tanımlanabilirler. Anonim fonksiyonları, projenin kısıtlı bir bölümünde, spesifik bir işi halletmek için kullanabiliriz.

Aşağıdaki kod örneğinde, projenin başka yerinde bu işleme ihtiyaç duymayacağımızı varsayarak, bir sliceı belirlenen boyutta parçalayan bir anonim fonksiyon kullandık.


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

Bu türdeki fonksiyonları bir değişkene atayıp bir kaç kez de kullanabiliriz.

Aşağıdaki örnekte spesifik bir APIden ayrı ayrı gönderilen ülke kodu ve telefon numaralarını değişken olarak tanımladığımız fonksiyonla birleştirdik.


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


## Metodlar

GO dilinde diğer nesne yönelimli programlama dilleri gibi classlar ve kalıtım yoktur. Fakat fonksiyonlara `receiver` denilen ek bir argüman ekleyerek ile kendi oluşturduğumuz tiplere fonksiyonları bağlayabiliriz. Böylece GO dilinde de nesne yönelimli programlamaya benzer bir deneyim yakalanabilir.


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


Yukarıdaki yapıda `Close` fonksiyonuna normal bir fonksiyona ek olarak `(conn *Connection)` kısmını ekledik. Böylece aşağıdaki örnekte görüldüğü gibi `c.Close()`  şeklinde`Connection` tipi üzerinden fonksiyona erişebildik.


```go
c := Connection{
    host:     "0.0.0.0",
    port:     8080,
    timeOut:  1,
    isClosed: false,
}
c.Close() // Connection 0.0.0.0:8080 closed.
```


> #### Pointer receiver \| Value receiver
>
> Fonksiyon parametrelerinde olduğu gibi receiverlerde de yapının değeri ile mi yoksa memory adresi ile mi kullanılacağına karar verilmelidir.
>
> Receiverları struct dışındaki tipler için de kullanabiliriz.


```go
type Iterable []int

func (iterable *Iterable) Append(items ...int) {
	*iterable = append(*iterable, items...)
}

arr := Iterable{1, 2}
arr.Append(4, 5, 6)

arr // [1 2 4 5 6]
```
