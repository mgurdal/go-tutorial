
# Interface

Interfaceler bir objenin sahip olabileceği methodları belirten yapılardır. Bu nedenle interfaceler objelerin davranışını ifade eder. Örneğin; bir dosyandan veri okunabilir veya dosyaya veri yazılabilir. Bir interface okuma ve yazma methodlarını barındırıyorsa ve dosya objesi bu methodlara sahipse dosyanın bu interfacei `implement` ettiği söylenebilir.

Interfacelerin asıl amacı methodları, aldıkları parametreleri ve dönüş değerlerini genel bir şekilde tanımlayarak; aynı davranışı gösteren, fakat farklı işlerde kullanılan yapılar için ortak bir protokol oluşturmaktır.

### Tanımlama

GO dilinde interfaceler objelerle _kapalı_ halde uygulanır. Başka bir deyişle; bir tipin bir interfacei kullandığını belirtmek için, diğer çoğu nesne yönelimli dillerde olduğu gibi,`class File implements IO` gibi deyimler kullanmamız gerekmez. GO aynı metodları paylaşan interface ve tipler arasındaki ilişkiyi kendisi kurar.


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


Yukarıdaki uygulamada `Read` ve `Write` metodlarını tanımlayan `IOInterface` interfaceini ve bu methodlara sahip`File` ve `Socket` tiplerini tanımladık.

### Kullanım

Aşağıdaki örnekte `Log` methodu `IOInterface` interfaceini parametre olarak aldığı için hangi veri tipinden geldiğini önemsemeden veriyi okuyup, işini yapabilir.


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


### Birden Fazla interface kullanma

GO tipleri birden fazla interfacei implement edebilir. Bunun için interfacein belirttiği metodlara sahip olmaları yeterlidir.  


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

### Tip dönüşümü


```go
s := sock.(*Socket)
string(s.buffer) // "data"
```


> Dönüştürdüğümüz tipin o interfacei doğru bir şekilde implement edip etmediğini dönüşüm sırasında alacağımız ek bir parametre ile kontrol edebiliriz.



```go
type Serializer interface {
	Serialize()
}

s, ok := sock.(Serializer)
s, ok // nil, false
```


### Boş interface

Interfaceler de diğer veri yapıları gibi tiplerdir ve aynı şekilde kullanılabilir. GO projelerinde boş interface; `interface{}` şeklinde tanımlanır. 

Hiçbir metodu olmadığı için boş interfacein gereksinimleri bütün tipler tarafından karşılanır. boş interfacein bu özelliği, dinamik parametreler ve değerler tanımlamamıza olanak tanır.


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


### Tip Kontrolü

Interface tipleri `switch` yapısı ile kontrol edilebilir. Aşağıdaki örnekte `Stringify` fonksiyonu`interface{}` tipinde parametre olarak alarak farklı veri tiplerini fonksiyonda kullanmamıza olanak sağlıyor. 

Bu fonksiyonda int string ve float gelen değerleri tiplerine göre kontrol edip belirlediğimiz şekilde stringe çevireceğiz.


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

