# Concurrency
[English](/go-tutorial/concurrency) | [Türkçe](/go-tutorial/tr/concurrency)

Bilgisayar biliminde concurrency\(eşzamanlılık\), bir programdaki iş birimlerinin veya algoritmaların sonucu etkilemeyecek şekilde **sırasız** bir biçimde çalıştırılabilmesidir.

İş parçalarını birbirinden bağımsız çalıştırabilmemiz, süreçleri farklı çekirdeklerde yürüterek, işlemciyi maksimum verimde kullanmamızı sağlar.

GO dilinde concurrency goroutineler ile sağlanır.

## Goroutines

GO routineleri bir kuyrukta çalıştırılmayı bekleyen boyutları küçük threadler olarak düşünülebilir. Bir fonksiyonu goroutine olarak çalıştırmak için çağırırken başına `go` yazmamız yeterli.

Aşağıdaki uygulamada Work fonksiyonu her çağrıldığında, çalışması bitene kadar uygulamayı blokluyor.

![routine](/assets/img/routine.gif)

```go
func Work(msg string) {
	fmt.Println(msg)
	time.Sleep(time.Second * 2)
}

func main() {
	Work("work work work work work work...")
	Work("zxcasfv")
}
```

Bloklayan kısımları goroutine olarak çalıştıralım.

```go
func main() {
	go Work("work work work work work work...")
	go Work("zxcasfv")
}
```
![goroutine](/assets/img/goroutine_unfinished.gif)

Programı çalıştırdığımızda ekrana hiçbir şey yazdırmadan sonlanacak. `main` fonksiyonu, `Work` fonksiyonu henüz ekrana bir şeyler yazacak kadar çalışmadan, bitiyor ve program kapanıyor. Program bitmeden önce biraz bekleterek `Work` fonksiyonunun çıktısını alabiliriz.

![goroutine_sleep](/assets/img/goroutine_sleep.gif)


```go
package main

import (
	"fmt"
	"time"
)

func Work(msg string) {
	fmt.Println(msg)
	time.Sleep(time.Second * 2)
}

func main() {
	go Work("work work work work work work...")
	go Work("zxcasfv")
	time.Sleep(time.Second * 3)
}

```

### waitGroup

Uygulamanın bir anında bazı goroutinelerin bitmesini beklememiz gerekebilir.

WaitGroup ile kaç tane gorouitine bekleyeceğimizi öncesindem belirtip bu goroutinelerin işlerini bitirmelerini bekleyebiliriz. Goroutinelerin burada tek yapması gereken, işi bittiğinde WaitGroup'un `Done()` methodunu çalıştırması.

![waitgroup](/assets/img/waitgroup.gif)

```go
func Work(msg string, wg *sync.WaitGroup) {
	time.Sleep(time.Second * 2)
	fmt.Println(msg)
	wg.Done()
}

func main() {
	nWorkers := 5
	wg := &sync.WaitGroup{}
	wg.Add(nWorkers)

	for i := 0; i < nWorkers; i++ {
		go Work("work", wg)
	}

	wg.Wait()
}

```

## Kanalllar (Channels)

Kanallar goroutinelerin birbirleri ile haberleşmesini sağlar.

Yukarıdaki örnekte `go` ile çalıştırdığımız fonksiyonlardan dönüş değeri alamıyoruz. Kanalları kullanarak bir goroutiniin, ana fonksiyonla veya başka bir goroutine ile haberleşmesini sağlayabiliriz.

Kanallar `make(chan veritipi, buffer)` şeklinde oluşturulabilir.

![channel_simple](/assets/img/channel_simple.gif)

```go
func Work(msg string, ch chan string) {
	time.Sleep(time.Second * 2)
	ch <- msg
}

func main() {
	channel := make(chan string)
	go Work("work-1", channel)
	go Work("work-2", channel)

	for i := 0; i < 2; i++ {
		msg := <-channel
		fmt.Println(msg)
	}
}
```

## Mutex

Mutex, goroutineler tarafından ortak kullanılan bir değişkenin değerininin, **race-condition** oluşmadan değiştirilebilmesini sağlayan yapıdır.

!!! quote "Wikipedia Tanımı"
	In computer science, mutual exclusion is a property of concurrency control, which is instituted for the purpose of preventing race conditions. It is the requirement that one thread of execution never enters its critical section at the same time that another concurrent thread of execution enters its own critical section, which refers to an interval of time during which a thread of execution accesses a shared resource, such as shared memory.


Aşağıdaki örnekte, kullanıcı tıklamalarını saymak için `Counter` yapısını kullanıyoruz.


```go

type  Counter struct {
	Value int
}

func (c *Counter) Inc() {
	c.Value++
}


func Click(c *Counter, wg *sync.WaitGroup) {
	c.Inc()
	wg.Done()
}

func main() {

	nUsers := 500
	wg := &sync.WaitGroup{}
	wg.Add(nUsers)

	counter := &Counter{}

	for i := 0; i < nUsers; i++ {
		go Click(counter, wg)
	}

	wg.Wait()
	fmt.Println(counter.Value)

}
```

İlk bakışta programda yanlış bir şey gözükmüyor. Fakat programı **--race** parametresi ile derleyip çalıştırdığımızda; bize programda **race-condition** olduğunu söylüyor.

![race](/assets/img/race.png)

Bu program çok çekirdekli bir bilgisayarda çalıştırıldığında, `Click` fonksiyonu farklı çekirdeklerde aynı anda çalıştırılabiliyor. Bu nedenle program yeni `c.Value` değerini yanlış güncelliyor.

![race](/assets/img/race.gif)

Bu hatayı `main` fonksiyonu içinde bir Mutex oluşturup; `mx.Lock` ve `mx.Unlock` metodlarını `Value` değerini değiştirdiğimiz yerde kullanarak giderebiliriz.

![race](/assets/img/mutex.gif)

```go
func Click(c *Counter, wg *sync.WaitGroup, mx *sync.Mutex) {
	mx.Lock()
	c.Inc()
	mx.Unlock()
	wg.Done()
}

func main() {

	nUsers := 500
	wg := &sync.WaitGroup{}
	wg.Add(nUsers)

	mx := &sync.Mutex{}
	counter := &Counter{}

	for i := 0; i < nUsers; i++ {
		go Click(counter, wg, mx)
	}

	wg.Wait()
	fmt.Println(counter.Value)

}
```

Mutexlerin struct içerisinde tanımlanması da yaygın bir kullanımdır. 


Bu örnekte, Mutexi, Counter içinde tanımlayıp `c.Lock` ve `c.Unlock` metodları ile
yine problemi çözebilirdik.

```go
type  Counter struct {
	Value int
	*sync.Mutex
}

func (c *Counter) Inc() {
	c.Value++
}


func Click(c *Counter, wg *sync.WaitGroup) {
	c.Lock()
	c.Inc()
	c.Unlock()
	wg.Done()
}

func main() {

	nUsers := 500
	wg := &sync.WaitGroup{}
	wg.Add(nUsers)

	counter := &Counter{Mutex: &sync.Mutex{}}

	for i := 0; i < nUsers; i++ {
		go Click(counter, wg)
	}

	wg.Wait()
	fmt.Println(counter.Value)

}
```

## Select
Kanallara veri göndermek veya kanallardan veri beklemek uygulamayı bloklar. 

Select aynı anda birkaç tane kanalı beklememize olanak sağlar. 

Aşağıdaki örnekte 2 goroutine `chan1` ve `chan2` kanallarına sürekli veri gönderiyor.

İki kanaldan da gelen tüm verileri sonsuz bir döngü içindeki select yapısı ile alabiliriz.
Herhangi bir kanaldan mesaj geldiğinde select o kanalı bekleyen `case`e girecek ve
içinde tanımlanan işlemler bittikten sonra döngü, kanal bekleme işlemini tekrarlayacaktır.

![select](/assets/img/select.gif)

```go
func Work(ch1 chan string, ch2 chan string) {

	// Send data to ch1 every second
	go func() {
		for {
			time.Sleep(1 * time.Second)
			ch1 <- "work-1"
		}
	}()

	// Send data to ch2 every 1.5 seconds
	go func() {
		for {
			time.Sleep(1.5 * time.Second)
			ch2 <- "work-2"
		}
	}()

}

func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go Work(ch1, ch2)

	for {
		select {
		case msg1 := <-ch1:
			fmt.Println(msg1)
		case msg2 := <-ch2:
			fmt.Println(msg2)
		}
	}

}

```

### Zaman Aşımı

Yukarıdaki gibi bir program 2 kanaldan da hiç veri gelmezse kilitlenecektir. 
Sonsuz döngü içerisinde, biteceğine emin olduğumuz başka bir kanalı daha dinleyerek kırabiliriz. 

Go, bu tür durumlar için önceden tanımlanmış yapılara sahiptir. 
Time modülündeki `time.After` fonksiyonu bellirlediğimiz bir süreden sonra kanala sinyal gönderir.

```go
func main() {
	ch1 := make(chan string)
	ch2 := make(chan string)

	go Work(ch1, ch2)

	for {
		select {
		case msg1 := <-ch1:
			fmt.Println(msg1)
		case msg2 := <-ch2:
			fmt.Println(msg2)
		case <-time.After(time.Second):
			fmt.Println("🎵 Brave Sir Robin ran away 🎵")
			fmt.Println("🎵 Bravely ran away away    🎵")
				return
		}

	}
```

## Context

"Context, bir iş iptal edildiğinde veya zaman aşımına uğradığında ilgili birimleri bilgilendirmek için kullanılır."

Context, en az diğer eşzamanlılık yapıları kadar önem arz eder.

`go Func()` şeklinde bir arkaplan işi baştattığımızda, bize goroutine durumunu kontrol edebileceğimiz bir değer dönülmez. 

Sonraki durumlarını önemsemeden bir sürü goroutine oluşturmak; belirgin
hata vermeden uygulamanın performansını önemli ölçüde etkileyebilir.

Bazen bir arkaplan görevi içerisinde, çok fazla kaynağa ihtiyaç duyan başka arkaplan görevleri oluşturmamız gerekir. Bu görevleri oluşturan goroutine bitse bile devam etmesi gerekmeyen diğer yavaş goroutineler çalışmaya devam eder ve kaynaklarımız boşa harcanır. 

Context bize bu gibi durumlarda yardımcı olur.

- Context bir sinyali tüm arkaplan görevlerine yaymamızı sağlar.
- Context hiyerarşileri oluşturabiliriz. Ana context iptal edildiğinde; bağlı tüm contextler de iptal edilir. 

Context Kullanılmadan                   |  Context Kullanılarak
:--------------------------------------:|:-------------------------:
![nocontext](/assets/img/nocontext.gif)  | ![context](/assets/img/context.gif)


### Uygulanış
GO tüm bu işlemleri `context` paketi içerisinde sunar.

Paket içerisinde, tüm contextleri tanımlayan, 4 adet metoda sahip `context.Context` arayüzü bulunur. 

```go
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key interface{}) interface{}
}
```

`Deadline()` contexte bağlı tüm işlerin bitmesi gereken süreyi döndürür. Bu metodu kullanarak; kalan sürede bitiremeyeceğimiz bir işi hiç başlatmayabiliriz.

`Done()` tüm işler tamamlandığında veya context iptal edildiğinde sinyal gönderen bir kanal döndürür.

`Err()` contextin neden bittiğini açıklayan bir hata döndürür. Bu hata `Canceled` veya `DeadlineExceeded` olabilir.

`Value` bir anahtar ile context içerisinde paylaştırılan bir değeri almamızı sağlar. Ana contextin verisi tüm hiyerarşi içerisinde payşaştırıldığı için; girilen anahtar tüm contextlerde aynı değeri dönecektir. 

!!! warning "Uyarı"
	Anahtar ve değerlerin tip kontrolü olmadığı için, bu mekanizmayı kullanarak görevler ve hiyerarşiler üzerinde veri paylaşımı yapmak **kötü bir fikirdir**.

Context paketi bu arayüzü farklı amaçlar için oluşturan fonksiyonlar sunar.

- `context.Background()` En temel contexti döndürür. Dönen context hiç bir iptal mekanizmas içermez. Tüm diğer contextler Background contexi kullanılarak oluşturulur.

- `context.TODO()` temelde `context.Background()` ile aynıdır. Henüz hangi tür contexti kullanacağımızdan emin olmadığımız yerlerde, yer tutması için bu contexti kullanırız.

Paket aynı zamanda bu contextleri sarmalayıp; iptal mekanizması ekleyen fonksiyonlar barındırır.

Bu fonksiyonların tamamı ilk parametre olarak ana contexti alır. Dönüş değeri olarak yeni bir context ve bir fonksiyon döndürür. 

Oluşan contextin kriteri sağlandığında veya bize verdiği fonksiyonu çağırdığımızda, contextin `Done()` kanalına bir sinyal gönderilir. 


- `context.WithCancel(parent)` contexti uygulamanın bir anında kendimiz durdurmak istediğimizde kullanılabilir. Bu fonksiyondan dönen contextin ayrıca çalışan bir iptal olma veya zaman aşımı durumu yoktur.

Aşağıdaki örnekte oluşturduğumuz Background contextini `context.WithCancel` ile sarmaladık.

Ardından bu contexti parametre alan 2 adet `SendRequest` görevi ve 1 saniye sonra `cancel`
fonksiyonunu çağıran bir görev oluşturduk.

Dikkat edeceğiniz üzere, tüm görevler ilk parametre olarak aldığı ve birbiriyle paylaştığı contextin `<- ctx.Done()` kanalını bekliyor.

`cancel` fonksiyonu tetiklendiğinde `ctx.Done()` kanalına sinyal göndererek tüm görevlere contextin bittiğini bildirecek.

![cancel](/assets/img/cancel.gif)

```go
// SearchDB 2 saniye sonra normal bir şekilde sonlanır
func SearchDB(ctx context.Context){
	fmt.Println("Searching Database")
	select {
	case <- ctx.Done():
		fmt.Println("Search Cancelled")
	case <- time.After(time.Second * 2):
		fmt.Println("Search Finished")
	}

}
// SendRequest 2 saniye sonra normal bir şekilde sonlanır
func SendRequest(ctx context.Context){
	fmt.Println("Processing Request")
	
	go SearchDB(ctx)
	go SearchDB(ctx)
	
	select {
	case <- ctx.Done():
		fmt.Println("Request Cancelled")
	case <- time.After(time.Second * 2):
		fmt.Println("Request Finished")
	}
}


func main() {
	ctx := context.Background()
	cancellingCtx, cancel := context.WithCancel(ctx)
	
	go SendRequest(cancellingCtx)
	go SendRequest(cancellingCtx)

	// Tüm requestleri ve database aramalarını 1 saniye sonra iptal edeceğiz.
	go time.AfterFunc(time.Second, cancel)

	select {
	case <- cancellingCtx.Done():
		fmt.Println("Cancelled by Context")
	case <- time.After(time.Second * 3):
		fmt.Println("Finished normally")
	}

	// Context iptal edildikten sonra goroutinelerin ekrana 
	// bir şeyler yazabilmesi için biraz bekliyoruz.
	<- time.After(time.Second * 3)
}
```

- `context.WithTimeout(parent, duration)` contextin belirli bir süre içinde tamamlanmasını istediğimiz durumlarda kullanılabilir. 

Hazırda elimizde olan contextlere yeni zaman sınırları ekleyebiliriz. Bu örnek için;
`SendRequest` fonksiyonuna timeout ekleyerek veritabanından 2 saniye içerisinde veri gelmezse görevi iptal edebiliriz.

![timeout](/assets/img/timeout.gif)


```go
// SendRequest 5 saniye sonra normal bir şekilde biter
// Veritabanı işini 2 saniye içerisinde tamamlanmazsa iptal eder
func SendRequest(ctx context.Context){
	fmt.Println("Processing Request")
	
	// Bu context tüm veritabanı işlemleri 2 saniye içerisinde 
	// tamamlanmazsa iptal edecek
	dbCtx, _ := context.WithTimeout(ctx, time.Second * 2)

	go SearchDB(dbCtx)
	go SearchDB(dbCtx)
	
	select {
	case <- ctx.Done():
		fmt.Println("Request Cancelled")
	case <- dbCtx.Done():
		fmt.Println("Search Timeout -", dbCtx.Err())  // Search Timeout - context deadline exceeded
	case <- time.After(time.Second * 5):
		fmt.Println("Request Finished")
	}
}

```

-  `context.WithDeadline(parent, time)` contextin belli bir saatte tamamlanmasını istediğimiz durumlarda kullanılabilir.


## Örnek Uygulama

Bir gurup işçi demir ve kömür barındıran maden yataklarında
maden arayıp, kazıp bu madenleri işlemektedir. 

Aşağıdaki uygulamada go dilinin sağladığı concurrency araçları ile işçilerin eş zamanlı olarak nasıl çalıştırılabileceğini görebiliriz. 

![Miners](..//assets/img/miners.png)

```go
package main

import (
	"fmt"
	"strconv"
	"sync"
	"time"
)

// Vein madenleri barındırır
type Vein struct {
	Ores []string
}

// Reveal bir sonraki madeni açığa çıkarır
func (vein *Vein) Reveal() (ore string) {
	if len(vein.Ores) == 0 {
		return
	}
	ore = vein.Ores[0]
	vein.Ores = vein.Ores[1:]
	return ore
}

// Worker madenleri bulur, kazar ve işler
type Worker struct {
	name string
}

// Find maden yatağında arama yapar ve bulduğu madeni kazıcılara gönderir
func (w *Worker) Find(vein *Vein, ch chan<- string) {
	ore := vein.Reveal()
	if ore != "" {
		fmt.Println(w.name, "found", ore)
		time.Sleep(time.Second * 2)

		ch <- ore
	}

}

// Mine madeni çıkartır ve işleme kanalına gönderir
func (w *Worker) Mine(finder <-chan string, smelter chan<- string) {
	for ore := range finder {
		fmt.Println(w.name, "mining", ore)
		time.Sleep(time.Second * 1)
		smelter <- ore
	}
}

// Smelt madenleri eritir veya işler
func (w *Worker) Smelt(coal_miner, iron_miner <-chan string, wg *sync.WaitGroup) {
	for {
		select {
		case coal_ore := <-coal_miner:
			time.Sleep(time.Second * 1)
			fmt.Println(w.name, "processed", coal_ore)
		case iron_ore := <-iron_miner:
			time.Sleep(time.Second * 2)
			fmt.Println(w.name, "smelted", iron_ore)
		}
		wg.Done()
	}
}

func main() {
	wg := &sync.WaitGroup{}

	// 100 demir barındıran bir maden oluştur
	n_iron_ores := 100
	iron_vein := &Vein{make([]string, n_iron_ores)}
	for i := range iron_vein.Ores {
		iron_vein.Ores[i] = "iron-ore-" + strconv.Itoa(i)
	}

	// 50 kömür barındıran bir maden oluştur
	n_coal_ores := 50
	coal_vein := &Vein{make([]string, n_coal_ores)}
	for i := range coal_vein.Ores {
		coal_vein.Ores[i] = "coal-ore-" + strconv.Itoa(i)
	}

	// İşçiler arasında maden iletim kanalları oluştur
	finder_to_miner := make(chan string)
	coal_miner_to_smelter := make(chan string)
	iron_miner_to_smelter := make(chan string)

	// Maden arama işçileri oluştur
	n_finders := 6
	finders := make([]*Worker, n_finders)
	for i := range finders {
		finders[i] = &Worker{
			name: "finder-" + strconv.Itoa(i),
		}
	}

	// Maden kazıcıları oluştur
	n_miners := 50
	miners := make([]*Worker, n_miners)
	for i := range miners {
		miners[i] = &Worker{
			name: "miner-" + strconv.Itoa(i),
		}
	}

	// Maden işleyicileri oluştur
	n_smelters := 10
	smelters := make([]*Worker, n_smelters)
	for i := range smelters {
		smelters[i] = &Worker{
			name: "smelter-" + strconv.Itoa(i),
		}
	}

	// Maden aramacıların yarısını demir, diğer yarısını kömür madenine gönder
	for i := 0; i < n_iron_ores+n_coal_ores; i++ {

		wg.Add(1)
		finder_idx := i % len(finders)

		go finders[finder_idx].Find(coal_vein, finder_to_miner)
		go finders[finder_idx].Find(iron_vein, finder_to_miner)
	}

	// Kazıcıların yarısını demir, diğer yarısını kömür madeninde çalıştır 
	for i, miner := range miners {

		if i%2 == 0 {
			go miner.Mine(finder_to_miner, iron_miner_to_smelter)
		} else {
			go miner.Mine(finder_to_miner, coal_miner_to_smelter)
		}

	}

	// Maden işleyen işçileri çalıştır
	for _, smelter := range smelters {
		go smelter.Smelt(coal_miner_to_smelter, iron_miner_to_smelter, wg)
	}

	// Bütün madenlerin bulunmasını, kazılmasını ve işlenmesini bekle
	wg.Wait()

}

```
