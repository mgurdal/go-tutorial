---
description: >-
  Best Practice, alternatiflerine karşı defalarca, açık bir şekilde daha fazla
  başarı göstermiş olan teknik veya yöntemdir. Bu sayfada GO için belirlenen en
  iyi yöntemlerden bazıları anlatılmaktadır.
---

# Best Practices

_A language that doesn’t affect the way you think about programming is not worth knowing. — Alan Perlis_

## Genel Prensipler

**Keep it simple**

**Don’t Repeat Yourself**

**You ain't gonna need it**

#### Avoid Nesting

İyi GO kodu dilin kendi kodları gibidir. Standart olarak bu yazım tercih edilmelidir.

```go
// Reset clears the state of the Writer z such that it is equivalent to its
// initial state from NewWriterLevel or NewWriterLevelDict, but instead writing
// to w.
func (z *Writer) Reset(w io.Writer) {
	z.w = w
	// z.level and z.dict left unchanged.
	if z.compressor != nil {
		z.compressor.Reset(w)
	}
	if z.digest != nil {
		z.digest.Reset()
	}
	z.err = nil
	z.scratch = [4]byte{}
	z.wroteHeader = false
}
```

[https://go-proverbs.github.io/](https://go-proverbs.github.io/)

## Kodlama

### Composition

#### Fragile Base Class Problemi

Fragile base class problemi nesne yönelimli programlama sistemlerindeki temel bir mimari problemidir. Bu mimarilerde ana sınıflar kırılgan kabul edilir. Bunun sebebi üst sınıftaki bir değişikliğin alt sınıflara da yansıması ve alt sınıflarda işlev bozukluğuna sebep olabilmesidir. Yazılımcı bu tür mimarilerde ana sınıfın metodlarını izole edemez ve bu nedenle ana sınıfta yapılan değişikliklerin güvenli olup olmadığınından emin olamaz.

Bu tür dillerde kalıtım yerine kompozisyon tercih edilmelidir. Kompleks ve hiyerarşik yapılar kodun esnekliğini kaybettirir.  
[https://en.wikipedia.org/wiki/Composition\_over\_inheritance](https://en.wikipedia.org/wiki/Composition_over_inheritance)

Kompozisyon yönteminde ise bir sınıfın diğerinin tüm özelliklerini miras almak yerine parametre olarak belli özellikleri seçilebilir.

_GO dilininin yazılımcıları kompozisyona itmesi en güçlü özelliklerinden birisidir._

### Loose Coupling

Yazılım dünyasında, coupling iki birimin birbirine ne kadar bağımlı olduğunun ölçüsüdür.

Parçalarının birbirine sıkı sıkıya bağlı olduğu programlarda değişiklik yapmak çok zordur. Bu duruma **shotgun surgery** efekti de denir.

Shotgun surgery terimi, projenin bir kısmında yapılan değişikliğin diğer kısımlarında sıralı halde ufak değişikliklere yol açmasıdır. Örnek vermek gerekirse:

```go
type Booking struct {
	ID     int
	Status string
}

type DB struct{}

func (d DB) Booking(ID int) *Booking {}

type Client struct{}

func Cancel(ID int) error {
	db := DB{}
	if booking := db.Booking(ID); booking.Status == "active"  {
		return errors.New("booking does not exists")
	}
	return nil
}

func Get(ID int) error {
	db := DB{}
	if booking := db.Booking(ID); booking.Status == "active"  {
		return errors.New("booking does not exists")
	}
	return nil
}
```

Yukarıdaki kodda Bookinge ait methodlarda tek bir kontrol var. O da bookingin aktif olması. Her metoda bu kontrolü ekleyerek _code smell_ oluşturduk. Şimdi `"active"` kontrolü değişirse Bookinge ait tüm metodlarda değişiklik yapmak gerekecek.

Bu sıkıntıyı gidermek için ilk adım, kontrol kısmını ayrı bir fonksiyonda tanımlayabiliriz.

```go
func validateBooking(db *DB, ID int) bool {
	if booking := db.Booking(ID); booking.Status == "active" {
		return true
	}
	return false
}

func Cancel(ID int) error {
	db := DB{}
	if isActive := validateBooking(&db, ID); isActive {
		return errors.New("booking does not exists")
	}
	return nil
}

func Get(ID int) error {
	db := DB{}
	if isActive := validateBooking(&db, ID); isActive {
		return errors.New("booking does not exists")
	}
	return nil
}
```

Doğrulamada bir değişiklik olduğunda artık sadece validateBooking fonksiyonun değiştirilmesi yeterli.

### Single Responsibility 

Yukarıdaki kod parçasında hala düzeltilmesi gereken kısımlar var. `validateBooking` fonksiyonu `DB` tipini doğrudan kullanıyor. Bu hem `tight coupling` oluşmasına hem de işi sadece _bookinglerin durumunu kontrol etmek_ olan fonksiyonun veritabanına da erişerek birden fazla iş yapmasına sebep oluyor. 

```go
// booking.go
type Booking struct {
	ID     int
	Status string
}

func (b Booking) Validate() bool {
	if b.Status == "active" {
		return true
	}
	return false
}

// db.go
type DB struct{}
func (d DB) Get(ID int) *Booking { return nil }

// client.go
type Client struct{}

func Cancel(ID int) error {
	db := DB{}
	booking := db.Get(ID)

	if booking.Validate() {
		return errors.New("booking does not exists")
	}
	return nil
}

func Get(ID int) error {
	db := DB{}
	booking := db.Get(ID)

	if booking.Validate() {
		return errors.New("booking does not exists")
	}
	return nil
}
```

### Liskov Substitution

Nesne yönelimli dillerde liskov substitution prensibi; miras alan sınıfın, üst sınıf ile yer değiştirdiği zaman, aynı davranışı sergilemesi gerektiğini söyler. GOda bu tarz soyut hiyerarşiler tanımlanamaz fakat substitution GO interfacelerinin çalışma mantığına çok benzer.

### Interface

 Nesne yönelimli programlamada öğrenilen yöntemler GOya taşınmamalıdır. Diğer dillerden gelen yazılımcılar interfaceleri genellikle aşağıdaki şekilde tanımlar.

```go
package rockband

type Player interface {
	Play()
}

type BandMemeber struct{}

func (b BandMemeber) Play(){ }

```

```go
package stage

func Perform(p Player) {
	p.Play()
}
```

Bu tarz kullanımda

1. Interface tanımlanır
2. Interfacei kullanmak için tip tanımlanır
3. Tanımlanan tiplerin interfacei implement eden metodları yazılır

Bu GO dilinde net bir şekilde yanlıştır çünkü; interfacei uygulayacak başka tipler oluşmadan yani net bir nedenimiz olmadan tanımladık.

Daha ideali, önce tipleri tanımlamak ve kullanmak. İhtiyaç duyulan yerde interfacelere başvurmak olacaktır.

```go
package rockband

type BandMemeber struct{}

func (b BandMemeber) Play(){ }

```

```go
package stage

type Player interface {
	Play()
}

func Perform(p Player) {
	p.Play() 
}
```

> Be conservative in what you send, be liberal in what you accept.  
> [Jon Postel](https://www.wikizeroo.org/index.php?q=aHR0cHM6Ly9lbi53aWtpcGVkaWEub3JnL3dpa2kvSm9uX1Bvc3RlbA)

Postel yasasını GOya uyarlamak gerekirse. Fonksiyonlar parametre olarak interface almalı, struct\(belirli bir veri tipi\) dönmelidir.

> _A great rule of thumb for Go is accept interfaces, return structs.  
> Jack Lindamood_

 Bununla birlikte fonksiyonların parametrelerinin de tamamen soyut olmamasında fayda vardır. 

> [the empty interface says nothing](https://www.youtube.com/watch?v=PAAkCSZUG1c&t=7m36s)   
> Rob Pike



Postel yasasının önemini test yazarken açık şekilde görebiliriz. Yukarıdaki fonksiyondaki `Perform` parametresinin belirli bir tip yerine interface olması, bizi test yazarken gerçek bir player objesi oluşturmaktan kurtarır

### Open-Closed  <a id="open-closed-principle"></a>

> Software entities should be open for extension, but closed for modification.  
> Bertrand Meyer

```go
package main

import "fmt"

type Cat struct {
	Name string
}

func (c Cat) Legs() int { return 4 }

func (c Cat) PrintLegs() {
	fmt.Printf("I have %d legs\n", c.Legs())
}

type OctoCat struct {
	Cat
}

func (o OctoCat) Legs() int { return 5 }

func main() {
	var octo OctoCat
	fmt.Println(octo.Legs()) // 5
	octo.PrintLegs()         // I have 4 legs
}

// source: https://dave.cheney.net/2016/08/20/solid-go-design
```

Yukarıdaki örnekte `Cat` tipi ve bu tipe ait ayak sayısını gösteren `Legs` metodu var. OctoCat tipine bu tipi gömdük ve `Legs` fonksiyonunu tanımladık. Bu durumda `OctoCat` Legs metodundan 5 sayısını döndürüyor ama `PrintLegs` metodu yine `Legs` metodunu kullanmasına rağmen 4 döndürüyor.

Bunun sebebi `PrintLegs` metodunun Cat tipi için tanımlanmış olması ve receiver olarak Cati alması.

### Interface Segregation 

_Clients should not be forced to depend on methods they do not use.  
Robert C. Martin_

Interface segregation, pek çok davranış içeren büyük veri yapıları oluşturmamamızı ve veri tipinin davranışını izole etmemizi söyler.

GO dilinde kompozisyon ve küçük interface kullanımları benimsenir.

  


### Dependency Inversion

_High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.   
Robert C. Martin_

Bu prensibin GOda uygulamasını paket seviyesinde görebiliriz. __

Önceki prensipler uygulandığında elimizde amaçları düzgünce tanımlanmış kod paketleri  olacaktır. Böyle bir yapıda kod içindeki bağımlılıkların interfaceler ile tanımlanmış olması ve bu interfacelerin sadece tanımladıkları fonksiyonlarla alakalı olması gerekir.

GOda importlar **acyclic** olmalıdır. Yani A paketi B paketini import ediyorsa, B paketi A'yı import etmemelidir. __Bu bağlamda Go projelerinin geniş ve düz şekilde tasarlanmalıdır. İç içe paketler tanımlamaktan kaçınılmalıdır.

Eğer bir paketteki fonksiyon başka bir paket olmadan düzgün çalışamıyorsa; bu, projenin paket seviyesinde iyi şekilde düzenlenmediğini gösterir.

Dependency inversion prensibi, projedeki asıl işlerin üst paketlerde, soyutlama/interface işlerinin alt paketlere yapmaya teşvik eder.

### Closure

GO'nun sağladığı interfaceler ve closurelar sayesinde çok estetik ve kullanışlı APIler yazabiliriz.

**Closure** kendi gövdesi dışındaki değişkenlere erişen fonksiyonlardır. Closure bir değişkene atanıp istenildiği zaman çağrılabilir veya başka bir fonksiyona parametre olarak verilebilir.

HTTP server oluştururken bu teknik sıklıkla kullanılır. Aşağıdaki örnekte `home` fonksiyonunu `timed` ile sarmalayıp `http.HandleFunc` metoduna parametre olarak verdik. Bu da `home/` end-pointine her istek geldiğinde, bu isteğin işlenme süresini hesaplamamızı sağladı.

```go
func main() {
  http.HandleFunc("/hello", timed(home))
  http.ListenAndServe(":3000", nil)
}

func timed(f func(http.ResponseWriter, *http.Request)) func(http.ResponseWriter, *http.Request) {
  return func(w http.ResponseWriter, r *http.Request) {
    start := time.Now()
    f(w, r)
    end := time.Now()
    fmt.Println("The request took", end.Sub(start))
  }
}

func home(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Home")
}
```

GO kaynak kodlarında da Closurelara bolca yer verilmiştir.

{% embed url="https://github.com/golang/go/blob/4e215554aa0e76e534327b9e6721971b9e82c113/src/sort/sort.go\#L307" %}

## Concurrency

Goroutineler her ne kadar boyutları küçük ve çok kullanışlı yapılar olsa da bağımsız olmaları kontrol edilmesi güç hatalara ve kod içinde veri akışının karmaşıklaşmasına yol açmakta. Bu yüzden goroutinelerin çalışma rutini API içerisinde ve mümkün olduğunca dar bir alanda tutulmalıdır.

Projede goroutine kullanılmaya karar verildiğinde race-conditionların önüne geçmek için öncelikle kod senkron olarak tanımlanmalıdır. Sonrasında, veri tipleri eşzamanlı erişmeye karşı korunmalıdır. Son olarak da korunan tipler goroutineler ile kullanılmalıdır.

`make(chan struct{})`, `sync.WaitGroup`, `context.Context` and `select` yapıları goroutineler söz konusu olduğunda sıkça kullanılmalı ve goroutinelerin çalışma süreci kontrol edilmelidir. 

İdeal çalışma sürecinde, goroutineler istenildiği zaman durdurulabilmeli. Ayrıca tüm sonuçlar gelene kadar goroutinelerin beklenmelidir.

Aynı WaitGroup ile farklı tipte goroutineleri sayılmamalıdır.

```go
type Parent struct {
  wgFoo sync.WaitGroup
  wgBar sync.WaitGroup
}

func (p *Parent) foo() {
  defer p.wgFoo.Done()
}

func (p *Parent) bar() {
  defer p.wgBar.Done()
}

func (p *Parent) Go() {
  p.wgFoo.Add(1)
  go p.foo()

  p.wgBar.Add(1)
  go.bar()
}
```

> _Concurrency is a way of structuring your program to make it easy to understand and scaleable. Parallelism is simply the execution of goroutines in parallel._

### Channels

Channel kullanıcısı channeli keyfi kapatamamalıdır. Daha önce kapatılmış bir channeli kapatmaya çalışmak `panic` oluşturacaktır.

Channeller consumer ve producer olarak modellenmelidir. GO channellerin yönlerini belirlememize olanak tanır.

Bir işi dağıtmak için channelleri gerektiğinde sıralı halde işlemek için mutexleri kullanmalıyız.

\_\_

## Yorumlama

GO dilinde yorum satırları, yazdığımız kodlar kadar önemlidir. 

`godoc` aracı kaynak kodlardaki paketlerin dökümanları toplayıp işler. Paketlerin, tiplerin, fonksiyonların başına yazılan yorum satırları, tanımlamaları ile birlikte işlenip o birim işin döküman oluşturur. 

Yazdığımız yorumların stili ve kalitesi `godoc` aracının oluşturacağı dökümanı da etkiler. 

Eğer paket basitse açıklaması kısa olabilir. 

```go
// Package draw provides image composition functions.
//
package draw

```

Çok yaygın kullanılacak bir kodu detaylıca açıklamamız gerekir.

```go
// The cap built-in function returns the capacity of v, according to its type:
//	Array: the number of elements in v (same as len(v)).
//	Pointer to array: the number of elements in *v (same as len(v)).
//	Slice: the maximum length the slice can reach when resliced;
//	if v is nil, cap(v) is zero.
//	Channel: the channel buffer capacity, in units of elements;
//	if v is nil, cap(v) is zero.
// For some arguments, such as a simple array expression, the result can be a
// constant. See the Go language specification's "Length and capacity" section for
// details.
func cap(v Type) int
```

### Döküman

Bir projenin _anlaşılır_ ve _sürdürülebilir_ olması için döküman çok önemlidir. Dökümanların da tıpkı iyi dizayn edilmiş kodlar gibi, amacına uygun yazılması gerekir. Aynı zamanda dökümanların kolay yazılıp; düzenlenebilir olması gerekir. 

GO dilinde dökümanların kodla bir arada olması ve kodla birlikte geliştirilmesi uygun görülür. 

Bir go paketinde public olan her kod satırı için **mutlaka** döküman yazılmalıdır. Paket içinde kullanılan kodlar için de yazmak zaruri olmasa da projenin sağlığı açısından oldukça faydalı olacaktır.

Eğer go paketi birden fazla dosyadan oluşuyorsa döküman `doc.go` adındaki ayrı bir dosyada tutulmalıdır.

Döküman satırları tam cümleler halinde yazılmalıdır. İlk cümlenin ilk kelimesi paketin/tipin/fonksiyonun ismiyle başlamalıdır.

{% hint style="danger" %}
Dökümanların içeriği kodun nasıl yazıldığı, neleri kullandığı ve nasıl kullandığı olmamalıdır. Ayrıca kullanılmayan kodlar yorum satırı olarak bırakılmamalıdır.
{% endhint %}

```go
// Ping the target server with a GET request
func Ping(url string, ch chan<- *Stat, waitGroup *sync.WaitGroup, logger *Log, sem chan bool) {
	// ping start time
	start := time.Now()
	// make a get request to url
	resp, err := http.Get(url)
	if err != nil {
		log.Fatal(err)
	}
	// close the body regardless
	defer resp.Body.Close()
	// ping end time
	end := time.Now()
	
	// create a stat struct to hold the ping information
	stat := &Stat{start, end, url, resp.StatusCode, "GET", resp.ContentLength}
	logger.AddStat(*stat)
	
	// extra := &Stat{}

	// send stat to channel
	ch <- stat
	// close semaphore
	sem <- true
	// wait for semaphore to be closed
	defer func() { <-sem }()
	// notify waitgroup
	waitGroup.Done()
}
```

{% hint style="success" %}
Dökümanların içeriği kodun neden yazıldığı, içeriği ve nasıl kullanıldığı olmalıdır.
{% endhint %}

{% tabs %}
{% tab title="ring\_full" %}
```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

// Package ring implements operations on circular lists.
package ring

// A Ring is an element of a circular list, or ring.
// Rings do not have a beginning or end; a pointer to any ring element
// serves as reference to the entire ring. Empty rings are represented
// as nil Ring pointers. The zero value for a Ring is a one-element
// ring with a nil Value.
//
type Ring struct {
	next, prev *Ring
	Value      interface{} // for use by client; untouched by this library
}

func (r *Ring) init() *Ring {
	r.next = r
	r.prev = r
	return r
}

// Next returns the next ring element. r must not be empty.
func (r *Ring) Next() *Ring {
	if r.next == nil {
		return r.init()
	}
	return r.next
}
```
{% endtab %}

{% tab title="ring\_license" %}
```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

```
{% endtab %}

{% tab title="ring\_package\_desc" %}
```go

// Package ring implements operations on circular lists.
package ring
```
{% endtab %}

{% tab title="ring\_type\_desc" %}
```go

// A Ring is an element of a circular list, or ring.
// Rings do not have a beginning or end; a pointer to any ring element
// serves as reference to the entire ring. Empty rings are represented
// as nil Ring pointers. The zero value for a Ring is a one-element
// ring with a nil Value.
//
type Ring struct {
	next, prev *Ring
	Value      interface{} // for use by client; untouched by this library
}

```
{% endtab %}

{% tab title="ring\_method\_desc" %}
```go

// Next returns the next ring element. r must not be empty.
func (r *Ring) Next() *Ring {
	if r.next == nil {
		return r.init()
	}
	return r.next
}
```
{% endtab %}
{% endtabs %}

{% embed url="https://github.com/golang/go/blob/master/src/container/ring/ring.go" %}

## İsimlendirme

### Paket

Paketlerin isimleri belirlenirken paketin içeriğini anımsatan isimler belirlenmelidir. Paket isimleri kısa, net, açıklayıcı, küçük karakterlerle yazılmış tek bir kelime olmalıdır. snake\_case veya camelCase isimlere ihtiyaç yoktur.

Paketlerin içeriğinindeki isim çakışmaları hakkında endişe edilmemelidir. Paket isimleri, sadece paketler için belirlenen ön tanımlı isimdir ve proje genelinde tekil olmasına gerek yoktur. Çakışma durumunda paketler farklı bir isimle import edilebilirler.

![](.gitbook/assets/image%20%2811%29.png)

![](.gitbook/assets/image%20%281%29.png)

### Kod Organizasyonu

GO projelerinde network, veritabanı gibi işler için oluşturulan paketler diğer paketler tarafından kullanılabilir fakat terminal komutu, http server başlatma gibi işleri yöneten paketler kendi başına çalışır. Proje yapısında bu öncelikle bu iki türü ayırmak faydalı olacaktır.

Birbiriyle alakasız paketlerin birbirinden bağımsız olması gerekir.

```text
github.com/peterbourgon/foo/
  circle.yml
  Dockerfile
  cmd/
    foosrv/
      main.go
    foocli/
      main.go
  pkg/
    fs/
      fs.go
      fs_test.go
      mock.go
      mock_test.go
    merge/
      merge.go
      merge_test.go
    api/
      api.go
      api_test.go
```

[https://github.com/perkeep/perkeep](https://github.com/perkeep/perkeep)

### Interface

Tek metod içeren interface isimleri genellikle method ismine _-er_ eklenerek tanımlanır. Örneğin Logger, Formatter.

```go
// Scanner is implemented by any value that has a Scan method, which scans
// the input for the representation of a value and stores the result in the
// receiver, which must be a pointer to be useful. The Scan method is called
// for any argument to Scan, Scanf, or Scanln that implements it.
type Scanner interface {
	Scan(state ScanState, verb rune) error
}

// Scanln is similar to Scan, but stops scanning at a newline and
// after the final item there must be a newline or EOF.
func Scanln(a ...interface{}) (n int, err error) {
	return Fscanln(os.Stdin, a...)
}

```

### Fonksiyon

`Read`, `Write`, `Close`, `Flush`, `String` vb. fonksiyonların çok belirgin ve dil içinde yaygın kullanımları vardır. Karmaşıklığı önlemek adına kendi fonksiyonlarımıza bu isimleri, aynı işi, aynı anlamda yapmıyorsa, kullanmamamız gerekir. 

### Erişim

GO paketindeki yalnızca baş harfi büyük yazılmış tiplere/değişkenlere/fonksiyonlara erişilebilir. Bu yöntem sayesinde paket için API belirleyebiliriz.

GO paket içindeki veri tipleri için doğrudan _getter_ ve _setter_ tanımlamaz fakat paket içindeki bir veriye erişim için kendimiz getter/setter tanımlayabiliriz. Getter/setterler metodların isimlerinin Get veya Set ile başlamasına gerek yoktur. Örneğin `List` adındaki bir tipin `len` alanına `GetLen()` veya `GetLength()` şekinde değil `Len()` şeklinde erişilmesi uygundur.

```go
// List represents a doubly linked list.
// The zero value for List is an empty list ready to use.
type List struct {
	root Element // sentinel list element, only &root, root.prev, and root.next are used
	len  int     // current list length excluding (this) sentinel element
}

// Len returns the number of elements of list l.
// The complexity is O(1).
func (l *List) Len() int { return l.len }

```

## Error Handling

GOda hata kontrolleri de normal yazılım akışının bir parçasıdır. Bu yüzden tüm hatalar düzgünce kontrol edilmelidir.

Hatalar `_` değişkenine atanıp görmezden gelinmemelidir. Eğer bir fonksiyon hata dönüyorsa bu hata kontrol edilmeli ve fonksiyonun başarılı şekilde çalıştığına emin olunmalıdır. 

Çok nadir durumlar haricinde `panic` hata kontrolü için kullanılmamalıdır. Eğer yazdığımız fonksiyon içinde bir hata oluşabilecekse, bu hata dönüş değeri olarak tanımlanmalıdır.

### Indent Errors

Normal kodlar minimum indentation ile yazılmalıdır. Normal çalışma akışının dışındaki durumlar\(hatalar\) kodun başında if ile kontrol edilip; fonksiyondan erkenden dönülmelidir.

Bu, yazılımcının fonksiyonun akışını kavramasını önemli ölçüde artıracaktır.

```go
if err != nil {
	// error handling
} else {
	// normal code
}
```

```go
if err != nil {
	// error handling
	return // or continue, etc.
}
// normal code
```

Dönüş değeri kullanılacaksa bu if dışında yapılmalıdır.

```go
if x, err := f(); err != nil {
	// error handling
	return
} else {
	// use x
}
```

```go
x, err := f()
if err != nil {
	// error handling
	return
}
// use x
```

## Testing

Test yazmak, geliştirme sürecinin bir parçası olmalıdır.

Testlerde piskolojik olarak minimum %80 total coverage ve tercihen %100 happy-path coverage hedeflenmelidir.

Düzgün testi yazılmamış her kod silinip yeniden testleri ile birlikte yazılabilir. 

Monkey-patchingden mümkün olduğunca kaçınılmalı; 3. parti bağımlılıkları mocklamak için önce temel yöntemlere başvurulmalıdır.



