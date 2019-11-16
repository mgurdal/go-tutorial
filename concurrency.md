# Concurrency

Bilgisayar biliminde concurrency\(eşzamanlılık\), bir programdaki iş birimlerinin veya algoritmaların sonucu etkilemeyecek şekilde **sırasız** bir biçimde çalıştırılabilmesidir.

İş parçalarını birbirinden bağımsız çalıştırabilmemiz, süreçleri farklı çekirdeklerde yürüterek, işlemciyi maksimum verimde kullanmamızı sağlar.

GO dilinde concurrency goroutineler ile sağlanır.

## Goroutines

GO routineler boyutları küçük threadler olarak düşünülebilir. Goroutine oluşturmak için sadece fonksiyonu çalıştırırken başına `go` yazmamız yeterli.

Aşağıdaki uygulamada Work fonksiyonu her çağrıldığında, çalışması bitene kadar uygulamayı blokluyor.

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

Programı çalıştırdığımızda ekrana hiçbir şey yazdırmadan sonlanacak. `main`fonksiyonu, `Work` fonksiyonu henüz ekrana bir şeyler yazacak kadar çalışmadan, bitiyor ve program kapanıyor. Ufak bir hile ile `Work` fonksiyonunun çıktısını alabiliriz.

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
	time.Sleep(time.Second * 1)
}

```

### waitGroup

Uygulamanın belli anlarında bazı goroutinelerin bitmesini beklememiz gerekebilir.

WaitGroup ile kaç tane gorouitne çalışacağını öncesinde girip sonrasında işlerini bitirmelerini bekleyebiliriz. Goroutinelerin burada tek yapması gereken, işi bittiğinde WaitGroup'un Done methodunu çalıştırması.

```go
func Work(msg string, wg *sync.WaitGroup) {
	time.Sleep(time.Second * 2)
	fmt.Println(msg)
	wg.Done()
}

func main() {
	nWorkers1, nWorkers2 := 5, 4
	wg := &sync.WaitGroup{}
	wg.Add(nWorkers1)

	for i := 0; i < nWorkers1; i++ {
		go Work("work-1", wg)
	}

	wg.Wait()

	wg.Add(nWorkers2)

	for i := 0; i < nWorkers2; i++ {
		go Work("work-2", wg)
	
	wg.Wait()
}

```

## Channels

Channeller goroutinelerin birbirleri ile haberleşmesini sağlar.

Yukarıdaki örnekte `go` ile çalıştırdığımız fonksiyonlardan dönüş değeri alamıyoruz. Channellar ile goroutinin sonucu ana fonksiyona veya başka bir goroutine göndermesini sağlayabiliriz.

channeller `make(chan veritipi, buffer)` şeklinde oluşturulabilir.

```go
func Work(msg string, ch chan string) {
	ch <- msg
	time.Sleep(time.Second * 2)
}

func main() {
	ch1 := make(chan string)
	go Work("work-1", ch1)
	go Work("work-2", ch1)

	for i := 0; i < 2; i++ {
		msg := <-ch1
		fmt.Println(msg)
	}
}
```

### mutex

```go
func Work(work_id int, msg string, wg *sync.WaitGroup, results map[string]string) {
	time.Sleep(time.Second * 1)
	worker_id := msg + "-" + strconv.Itoa(work_id)
	results[worker_id] = msg
	wg.Done()
}

func main() {

	nWorkers1, nWorkers2 := 5, 4
	wg := &sync.WaitGroup{}
	results := make(map[string]string, nWorkers1+nWorkers2)

	wg.Add(nWorkers1)

	for i := 0; i < nWorkers1; i++ {
		go Work(i, "work-1", wg, results)
	}

	wg.Wait()
	fmt.Println(results)
	wg.Add(nWorkers2)

	for i := 0; i < nWorkers2; i++ {
		go Work(i, "work-2", wg, results)
	}
	wg.Wait()
	fmt.Println(results)

}

```

![](.gitbook/assets/image%20%288%29.png)

```go
func Work(work_id int, msg string, wg *sync.WaitGroup, mx *sync.Mutex, results map[string]string) {
	mx.Lock()
	time.Sleep(time.Second * 1)
	worker_id := msg + "-" + strconv.Itoa(work_id)
	results[worker_id] = msg
	mx.Unlock()

	wg.Done()
}

func main() {

	nWorkers1, nWorkers2 := 5, 4
	wg := &sync.WaitGroup{}
	mx := &sync.Mutex{}
	results := make(map[string]string, nWorkers1+nWorkers2)

	wg.Add(nWorkers1)

	for i := 0; i < nWorkers1; i++ {
		go Work(i, "work-1", wg, mx, results)
	}

	wg.Wait()
	fmt.Println(results)
	wg.Add(nWorkers2)

	for i := 0; i < nWorkers2; i++ {
		go Work(i, "work-2", wg, mx, results)
	}
	wg.Wait()
	fmt.Println(results)
}
```

## Select

Channellere veri göndermek veya channellerden veri beklemek uygulamayı dondurur. Channellerin uygulamayı bloklamasını istemiyorsak bunu select yapısı ve default değeri ile yapabiliriz.

```go
chan1 := make(chan string)
chan2 := make(chan string)

go func() {
	for {
		time.Sleep(2 * time.Second)
		chan1 <- "one"
	}
}()
go func() {
	for {
		time.Sleep(1 * time.Second)
		chan2 <- "two"
	}
}()

for {
	select {
	case msg1 := <-chan1:
		fmt.Println(msg1)
	case msg2 := <-chan2:
		fmt.Println( msg2)
	}
}
```

### timeout

Yukarıdaki gibi bir program 2 channela da veri gönderilmezse kilitlenecektir. Bunun önüne geçmek için timeout ekleyebiliriz.

```go
chan1 := make(chan string)
chan2 := make(chan string)

go func() {
	for {
		time.Sleep(5 * time.Second)
		chan1 <- "one"
	}
}()
go func() {
	for {
		time.Sleep(5 * time.Second)
		chan2 <- "two"
	}
}()

for {
	select {
	case msg1 := <-chan1:
		fmt.Println(msg1)
	case msg2 := <-chan2:
		fmt.Println(msg2)
	case <-time.After(time.Second * 1):
		fmt.Println("Bekle dedi gitti. Ben beklemedim. O da gelmedi.")
		return
	}

}
```

{% embed url="https://play.golang.org/p/UFzqS4wYKJZ" %}

{% embed url="https://play.golang.org/p/mtp9cuL9Kno" %}



