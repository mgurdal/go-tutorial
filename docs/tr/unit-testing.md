---
description: >-
  Bu sayfada unit testlerin yazılış amacı, uygulanış biçimleri, test için uygun
  tasarım desenleri ve test yazarken önüne geçilebilecek hatalar
  anlatılmaktadır.
---

# Unit Testing

> Code without tests is broken as designed.  
> [Jacob Kaplan-Moss](https://jacobian.org/)

## Giriş

Unit testing, projedeki fonksiyonel her bir birimin tüm davranışlarının test edildiği yazılım test yöntemidir. Biraz daha açmak gerekirse; bir birim, testi mümkün olan en küçük yazılım bileşenidir ve çoğunlukla tek bir iş, yapmakla görevlidir. 

Unit testin amacı, programın küçük bölümlere ayırıp, tüm bölümlerin beklenildiği gibi çalıştığınından emin olmaktır.

Unit testleri yazılan bir projede:

* Projeye yeni eklenen kodun, istenildiği gibi çalışıp, çalışmadığını kontrol edilebilir.
* Varolan kod düzenlendiğinde, projenin davranışının beklenmedik şekilde değiştirmediğinden emin olunur.
* Yazılımcı, projeye eklenecek karmaşık özellikleri küçük, anlaşılır parçalara bölmesine dolayısıyla işe yarar minimum fonksiyonelliğe hızlıca erişip gereksiz optimizasyondan kaçınmasına yardımcı olur.
* Ekibe yeni katılan yazılımcılar, testleri döküman olarak kullanabilir.


> Testlerin bir projedeki bütün hataları çözmesi beklenmemelidir. Projelerde her zaman hata vardır. Bu hatalar, test edilmeyen bir durumdan, yanlış test etmekten veya testlerin kontrol edemeyeceği 3. parti kütüphanelerden, APIlerden, konfigürasyon sorunlarınlarında kaynaklı olabilir.

## Test etmek

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, world")
}
```

Yukarıdaki kodu test etmeden önce yan etkilerden arındırmalıyız. Buradaki side-effect `Println` fonksiyonu.

```go
package main

import "fmt"

func Hello() string {
    return "Hello, world"
}

func main() {
    fmt.Println(Hello())
}
```

Şimdi `Hello` fonksiyonunu için test yazalım.

```go
func TestHello(t *testing.T) {
    got := Hello()
    want := "Hello, world"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

Go testlerini çalıştırmak için ek araçlara veya frameworklere ihtiyacımız yok. Sadece `go test` komutu ile yazdığımız testleri çalıştırabiliriz.

![](.gitbook/assets/image%20%283%29.png)

Test yazmanın belirli kurallar çerçevesinde fonksiyonlar yazmaktan farkı yoktur. Bu fonksiyonların test olarak kabul edilmesi için:

* `xxx_test.go` gibi bir dosya içinde olması
* Fonksiyon isminin `Test` ile başlaması
* Fonksiyonun yalnızca `t *testing.T` parametresini alması gerekir.

## **Yeni Özellikler Eklemek**

![Source: https://openclassrooms.com/en/courses/4554386-enhance-an-existing-app-using-test-driven-development/5095731-understand-the-red-green-refactor-methodology](.gitbook/assets/image%20%2812%29.png)

Test başarılı bir şekilde çalıştığına göre artık uygulamaya yeni özellikler ekleyebiliriz.

Şimdi Hello fonksiyonu parametre olarak bir isim alsın ve o ismi selamlasın.  
Bunun için öncellikle kodun yeni halini test eden ve başarısız olan bir test yazacağız.

```go
func TestHello(t *testing.T) {
    got := Hello("Mehmet")
    want := "Hello, Mehmet"

    if got != want {
        t.Errorf("got %q want %q", got, want)
    }
}
```

![](.gitbook/assets/image%20%287%29.png)

Testi geçirecek kadar kod yazalım.

```go
func Hello(name string) string {
	return "Hello, Mehmet"
}
```


> Yukarıdaki örnekte amacımızın dışında bir kod yazarak testin geçmesini sağladık. Bu örnek her ne kadar abes dursada, bu tür durumlar daha büyük ölçekli işlerde başımıza gelebilir ve fark edilmediğinde anlaşılması zor buglar ortaya çıkartabilir.


Eklediğimiz özelliğin düzgün çalıştığından emin olmak için bir test daha ekleyelim.

```go
func TestHello(t *testing.T) {
	got := Hello("Mehmet")
	want := "Hello, Mehmet"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}

func TestHelloWithAnotherName(t *testing.T) {
	got := Hello("Ahmet")
	want := "Hello, Ahmet"

	if got != want {
		t.Errorf("got %q want %q", got, want)
	}
}
```

![](.gitbook/assets/image%20%284%29.png)

Şimdi de testi geçecek kodu yazalım.

```go
func Hello(name string) string {
	return "Hello, " + name
}
```


> Bu aşamada elimizde testleri ile birlikte çalışan yeni özelliği commit edebiliriz.


Bu aşamada, TDD yaşam döngüsü gereği kod refactor edilmeye çalışılır. "Hello, " her durumda sabit olduğu için globalde tanımlayabiliriz.

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
    return englishHelloPrefix + name
}
```


> Refactor yapıldıktan sonra mutlaka testler yeniden çalıştırılmalı ve kod davranışının değişmediğinden emin olunmalıdır.


Şimdi uygulamamıza bir özellik daha ekleyelim. Bu kez `Hello` fonksiyonuna boş isim verilince ön tanımlı mesajla selamlasın.

Tabiki önce testini yazıyoruz.

Aynı kod parçası için yazılan testleri tek fonksiyon altında toplayabiliriz. `t.Run` yapısı sayesinde verdiğimiz farklı test mesajları ile birlikte testleri bir araya getirebiliriz. 

```go
func TestHello(t *testing.T) {

    t.Run("Say hello to a specific name", func(t *testing.T) {
        got := Hello("Mehmet")
        want := "Hello, Mehmet"

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })

    t.Run("Say 'Hello, Dude' when an empty string is provided", func(t *testing.T) {
        got := Hello("")
        want := "Hello, Dude"

        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    })
}
```


> Refactoring sadece _business logic_ içeren kodlar için değildir. Test kodlarına da projede tıpkı diğer kodlar gibi yaklaşılmalıdır.


Bu aşamada testlerimizde tekrar eden kısımlar için bir fonksiyon yazabiliriz.

```go
func TestHello(t *testing.T) {

    assertCorrectMessage := func(t *testing.T, got, want string) {
        t.Helper()
        if got != want {
            t.Errorf("got %q want %q", got, want)
        }
    }

    t.Run("Say hello to a specific name", func(t *testing.T) {
        got := Hello("Mehmet")
        want := "Hello, Mehmet"
        assertCorrectMessage(t, got, want)
    })

    t.Run("Say 'Hello, Dude' when an empty string is provided", func(t *testing.T) {
        got := Hello("")
        want := "Hello, Dude"
        assertCorrectMessage(t, got, want)
    })
}
```

Şimdi elimizde düzgün bir şekilde yazılmış hata veren bir test var. Uygulamaya testi geçmek için yeterli kodu yazalım.

```go
const englishHelloPrefix = "Hello, "

func Hello(name string) string {
    if name == "" {
        name = "Dude"
    }
    return englishHelloPrefix + name
}
```

Böylece uygulamamıza önceki özellikleri etkilemeden yeni bir özellik eklemiş olduk.


> Bu aşamada eski commiti amend edip, fonksiyonumuzu yeni versiyonu ile değiştirebiliriz.


### Geliştirme Disiplini

* Yeni özelliği kullanan bir test yaz
* Kodun derlendiğinden emin ol
* Testi çalıştır, hatayı gözlemle
* Testi geçirecek minimum kodu yaz
* Refactor

Bu kadar adım zahmetli ve paranoyakça gelebilir fakat her adımda kodun durumunu kontrol etmek oldukça önemlidir.

Bu disipline sadık kalarak sadece amacına uygun testler yazmakla kalmayız aynı zamanda otomatikleştirilmiş testlerle sürekli düzenlenen ve iyi şekilde tasarlanan bir yazılım geliştirmiş oluruz.

Testlerin ne için yazıldığını anlaşılır bir şekilde hata mesajında göstermek çok önemlidir. Yazılımcının, testin neden hata verdiğini anlayamaması problemi çözmeyi zorlaştıracaktır.

Testlerin hızlı çalışması ve kolay bir şekilde çalıştırılabilmesi gerekir. Bu akıcı bir şekilde kod yazmamızı sağlayacaktır.

Testsiz kod yazarak, uygulamanın akışını ve olası her durumunu kontrol etme işini kendinize yüklemiş olursunuz. Bu da uzun dönemde ciddi vakit kaybına yol açacaktır.

### Daha fazla özellik

Yeni isteğe göre uygulamamıza dil desteği ekleyeceğiz. Eğer dil sistemimiz tarafından destekleniyorsa selamlama mesajı verilen dilde sağlanacak. Desteklenmiyorsa ön tanımlı olan İngilizce mesaj gönderilecek.

Başlangıç olarak Fransızca dil desteği için test yazalım.

```go
t.Run("in French", func(t *testing.T) {
    got := Hello("Lara Fabian", "French")
    want := "Bonjour, Lara Fabian"
    assertCorrectMessage(t, got, want)
})
```


> Bu haliyle `Hello` tek parametre aldığı için compiler hata verecektir. Bu adımda, hile yapıp doğrudan kodu yazmaktan kaçınmalıyız.


Fonksiyona gerekli olan 2. parametreyi ekleyelim.

```go
func Hello(name string, language string) string {
    if name == "" {
        name = "Dude"
    }
    return englishHelloPrefix + name
}
```

Testleri çalıştırdığımızda, yeni özelliğe göre tasarlanmadığı için eski testler hata verecektir. Eski testlere parametre olarak boş string vererek compiler hatasından kurtulalım. Bu durumda, sadece yeni eklediğimiz testin fail olması gerekir.

Şimdi response bodyi  

![](.gitbook/assets/image%20%286%29.png)

Yeni bir `if` ekleyerek Fransızca için yazdığımız testi geçebiliriz.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "Dude"
	}
	if language == "French" {
		return "Bonjour, " + name
	}
	return englishHelloPrefix + name
}
```

Bu haliyle bütün testleri başarılı olması gerekiyor.

Şimdi kodumuzu _refactor_  edebiliriz. İşe, kod içinde tanımlı stringlerden başlayabiliriz.   
_Yapılan her değişiklikte testlerin başarılı olduğunu kontrol etmeliyiz._

```go
const french = "French"
const englishHelloPrefix = "Hello, "
const frenchHelloPrefix = "Bonjour, "

func Hello(name string, language string) string {
	if name == "" {
		name = "Dude"
	}
	if language == french {
		return frenchHelloPrefix + name
	}
	return englishHelloPrefix + name
}
```

#### Yeni dil

Şimdi Koreceye de destek verelim. Bunun için daha önce yaptığımız gibi:

* Korean yazdığımızda 안녕 ile başlayan bir mesaj bekleyen test yazacağız
* Testi çalıştırıp hata alacağız ve hata mesajının düzgün olduğuna dikkat edeceğiz
* Kod üzerinde testi geçecek minimum değişikliği yapacağız

Değişikliklerden sonra elimizde aşağı yukarı böyle bir kod olacak:

```go
func TestHello(t *testing.T) {

	assertCorrectMessage := func(t *testing.T, got, want string) {
		t.Helper()
		if got != want {
			t.Errorf("got %q want %q", got, want)
		}
	}

	t.Run("Say hello to a specific name", func(t *testing.T) {
		got := Hello("Mehmet", "")
		want := "Hello, Mehmet"
		assertCorrectMessage(t, got, want)
	})

	t.Run("Say 'Hello, Dude' when an empty string is provided", func(t *testing.T) {
		got := Hello("", "")
		want := "Hello, Dude"
		assertCorrectMessage(t, got, want)
	})

	t.Run("in French", func(t *testing.T) {
		got := Hello("Lara Fabian", "French")
		want := "Bonjour, Lara Fabian"
		assertCorrectMessage(t, got, want)
	})

	t.Run("in Korean", func(t *testing.T) {
		got := Hello("세상", "Korean")
		want := "안녕 세상"
		assertCorrectMessage(t, got, want)
	})
}
```

```go
const french = "French"
const korean = "Korean"

const englishHelloPrefix = "Hello, "
const frenchHelloPrefix = "Bonjour, "
const koreanHelloPrefix = "안녕 "

func Hello(name string, language string) string {
	if name == "" {
		name = "Dude"
	}
	if language == french {
		return frenchHelloPrefix + name
	}
	if language == korean {
		return koreanHelloPrefix + name
	}
	return englishHelloPrefix + name
}
```

Yeni özelliğimizi, çalışan testi ile eklediğimize göre şimdi kodumuzu refactor edebiliriz.

```go
func Hello(name string, language string) string {
	if name == "" {
		name = "Dude"
	}

	prefix := englishHelloPrefix

	switch language {
	case french:
		prefix = frenchHelloPrefix
	case korean:
		prefix = koreanHelloPrefix
	}
	return prefix + name
}
```

## Real World Example

Gerçek bir uygulamaya testlerin nasıl yazıldığını incelemek için https://randomuser.me/api/ APIsinden kullanıcıları çeken bir HTTP Client yazalım. UserClient yapısı ile, verilerin çekileceği URLi, kaç sayfa veri çekeceğini ve her sayfada ne kadar veri alacağı  ve de HTTP isteklerinin atılacağı kütüphane belirliyoruz.

```go
type UserClient struct {
	BaseUrl                  string
	TaskCount, ResultPerPage int
}

uc := &UserClient{
    BaseUrl:       "https://randomuser.me/api/",
    TaskCount:     20,
    ResultPerPage: 100,
    requester:     &http.Client{},
}

user, _ := uc.Get()
users, _ := uc.FetchAll()
fmt.Println(len(res)) // 2000
```

### Mocking

Unit testlerin çalıştığı ortamdan, dış kaynaklardan ve kütüphanelerden bağımsız olması gerekir. Bu yüzden `http.Client` aynı davranışı sergileyen fakat sahte veri dönen başka bir modül ile değiştirilmeli.

Ortak davranışlara sahip modüller geliştirmenin en kolay yolu ortak interface kullanmak. http.Clientin kullanacağımız methodlarına sahip Requester adında bir interface tanımlayalım. 

```go
type Requester interface {
	Get(url string) (resp *http.Response, err error)
}

type UserClient struct {
	BaseUrl                  string
	TaskCount, ResultPerPage int

	requester Requester
}

func (d *UserClient) Get() (response map[string]interface{}, err error) {
}

func (d *UserClient) FetchAll() (result []map[string]interface{}, err error) {
}
```

Testleri çalıştırırken kullanacağımız, http.Client gibi Requesteri implement eden sahte bir client oluşturalım ve FakeClientı kullanarak UserClient için bir test yazalım.

```go
type FakeClient struct {
	Body  []byte
	Error error
}

func (c *FakeClient) Get(url string) (resp *http.Response, err error) {
	resp = &http.Response{
		Body: ioutil.NopCloser(bytes.NewBuffer(c.Body)),
	}
	return resp, c.Error
}

func TestGetUser(t *testing.T) {
	userclient := &UserClient{}

	t.Run("get userclient returns err on bad response", func(t *testing.T) {

		userclient.requester = &FakeClient{
			Error: errors.New("Bad Request"),
		}

		_, err := userclient.Get()
		if err == nil {
			t.Error("Expected to get an error")
		}
	})

}
```

http.Clienti, sahtesi ile aşağıdaki şekilde değiştirdik.

```go
userclient.requester = &FakeClient{
    Body: []byte(`{"status": "ok"}`),
}
```

FakeClient aldığı Body verisini Get methodu çalıştırıldığında bize http.Clientinkine benzer şekilde dönecek.

```go
func (c *FakeClient) Get(url string) (resp *http.Response, err error) {
	resp = &http.Response{
		Body: ioutil.NopCloser(bytes.NewBuffer(c.Body)),
	}
	return resp, c.Error
}
```

Artık elimizde fail olan bir test var. Bu testi geçecek kodu yazabiliriz.

```go
func (d *UserClient) Get() (response map[string]interface{}, err error) {
	err := erros.New("Error")
	return
}
```

Şimdi de response bodyi kontrol eden bir test yazalım.

```go
t.Run("get userclient returns err if response body invalid", func(t *testing.T) {
	userclient.requester = &FakeClient{}

	_, err := userclient.Get()
	if err == nil {
		t.Error("Expected to get an error")
	}
})

```

Artık testi geçmek için kod yazabiliriz.

```go
func (d *UserClient) Get() (response map[string]interface{}, err error) {
	url := fmt.Sprintf("%s?results=%d", d.BaseUrl, d.ResultPerPage)

	res, err := d.requester.Get(url)
	if err != nil {
		return nil, err
	}
}
```

Son olarak başarılı response durumunu test edelim.

```go
t.Run("get userclient returns unmarshalled response body on success", func(t *testing.T) {

	userclient.requester = &FakeClient{
		Body: []byte(`{"status": "ok"}`),
	}
	got, _ := userclient.Get()
	want := map[string]interface{}{
		"status": "ok",
	}
	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %q want %q", got, want)
	}
})
```

Artık byte JSONu düzgünce _parse_ eden bir kod yazmamız gerekiyor.

```go
func (d *UserClient) Get() (response map[string]interface{}, err error) {
	url := fmt.Sprintf("%s?results=%d", d.BaseUrl, d.ResultPerPage)

	res, err := d.requester.Get(url)
	if err != nil {
		return nil, err
	}
	defer res.Body.Close()
	buf := new(bytes.Buffer)
	buf.ReadFrom(res.Body)
	newStr := buf.String()
	var data map[string]interface{}
	err = json.Unmarshal([]byte(newStr), &data)
	response = data
	return
}
```

### Table Test 

Kod tekrarını azaltmak için _table driven test_  yöntemini kullanabiliriz.

```go
func TestFetchAll(t *testing.T) {

	testCases := []struct {
		name     string
		client   *UserClient
		expected []map[string]interface{}
	}{
		{
			"get userclient returns err on bad response",
			&UserClient{
				ResultPerPage: 1,
				TaskCount:     0,
				requester: &FakeClient{
					Body: []byte(`{"results": [{"test": "ok"}]`),
				},
			},
			[]map[string]interface{}{},
		},
		{
			"get userclient returns err if response body invalid",
			&UserClient{
				ResultPerPage: 0,
				TaskCount:     1,
				requester: &FakeClient{
					Body: []byte(`{"results": [{"test": "ok"}]`),
				},
			}, []map[string]interface{}{},
		},
		{
			"get userclient returns unmarshalled response body on success",
			&UserClient{
				ResultPerPage: 0,
				TaskCount:     1,
				requester: &FakeClient{
					Body: []byte(`{"results": [{"test": "ok"}]`),
				},
			}, []map[string]interface{}{},
		},
	}

	for _, tt := range testCases {
		t.Run(tt.name, func(t *testing.T) {
			got, _ := tt.client.FetchAll()
			if !reflect.DeepEqual(got, tt.expected) {
				t.Errorf("got %q want %q", got, tt.expected)
			}
		})
	}
}

```

## Anti-Patternler

**İkinci Sınıf Vatandaşlık  
D**iğer kodları yazarken uyguladığımız yazılım prensipleri test kodları için de geçerli.  
**Obsessive Coverage**  
Test coverageın yüksek olması, projenin bütün durumlarının test edildiğini göstermez. Bir projenin düzgün test edildiğinin göstergesi sadece coverage raporları değildir.  
**Structural Inspection**  
Projede structların, ve diğer veri yapılarının aldıkları değerleri değil methodları ve fonksiyonların davranışlarını test etmeliyiz.  
**God Object**  
Test edilmesi zor, büyük yapılar parçalanmalıdır. Eğer test ettiğimiz kod çok fazla mock kullandırıyorsa bu God Objectin belirtisi olabilir.  
**Mockery**  
Testlerde aşırı mock kullanımı, uygulama davranışını değil mockerların değerlerini test etmemizle sonuçlanır.  
**Instance Reuse**  
Test edilen yapılar, çalıştırılan her test için yeniden üretilmelidir.  
**Local Hero**  
Testler çalıştığı ortamdan ve makineden bağımsız olmalıdır.  
**Happy Path Testing**  
Uygulamada sadece başarılı sonuçları değil, olası bütün durumları test etmeliyiz.  
**Kötü İsimlendirme \(Sıralı İsimlendirme\)**  
Testlerin isimleri ve mesajları açıklayıcı ve ayırt edici olmalıdır.  
**Yeni Test Yerine Assertion Ekleme**  
Birime yeni özellik eklendiğinde, veya bir kısmı değiştirildiğinde var olan bir teste ekstra bir kontrol eklenmemelidir. Sadece bu durumu yeni bir test yazılmalıdır.


```go
package main

import (
	"bytes"
	"encoding/json"
	"fmt"
	"net/http"
)

type Requester interface {
	Get(url string) (resp *http.Response, err error)
}

type UserClient struct {
	BaseUrl                  string
	TaskCount, ResultPerPage int

	requester Requester
}

func (d *UserClient) Get() (response map[string]interface{}, err error) {
	url := fmt.Sprintf("%s?results=%d", d.BaseUrl, d.ResultPerPage)

	res, err := d.requester.Get(url)
	if err != nil {
		return nil, err
	}
	defer res.Body.Close()
	buf := new(bytes.Buffer)
	buf.ReadFrom(res.Body)
	newStr := buf.String()
	var data map[string]interface{}
	err = json.Unmarshal([]byte(newStr), &data)
	response = data
	return
}

func (d *UserClient) FetchAll() (result []map[string]interface{}, err error) {
	ch1 := make(chan map[string]interface{}, 1)

	expectedCount := d.TaskCount * d.ResultPerPage
	result = []map[string]interface{}{}

	if d.TaskCount == 0 || d.ResultPerPage == 0 {
		return
	}
	for i := 0; i < d.TaskCount; i++ {
		go func(ch chan map[string]interface{}) {
			response, err := d.Get()
			if err != nil {
				for j := 0; j < d.ResultPerPage; j++ {
					ch <- map[string]interface{}{}
				}
			} else {
				data := response["results"].([]interface{})
				for _, datum := range data {
					ch <- datum.(map[string]interface{})
				}
			}

		}(ch1)
	}

	for data := range ch1 {
		result = append(result, data)
		if len(result) == expectedCount {
			close(ch1)
		}
	}

	return result, nil
}

func main() {
	uc := &UserClient{
		BaseUrl:       "https://randomuser.me/api/",
		TaskCount:     20,
		ResultPerPage: 100,
		requester:     &http.Client{},
	}
	res, _ := uc.FetchAll()
	fmt.Println(len(res))
}

```

````go
package main

import (
	"bytes"
	"errors"
	"io/ioutil"
	"net/http"
	"reflect"
	"testing"
)

type FakeClient struct {
	Body  []byte
	Error error
}

func (c *FakeClient) Get(url string) (resp *http.Response, err error) {
	resp = &http.Response{
		Body: ioutil.NopCloser(bytes.NewBuffer(c.Body)),
	}
	return resp, c.Error
}

func TestGetUser(t *testing.T) {
	userclient := &UserClient{}

	t.Run("get userclient returns err on bad response", func(t *testing.T) {

		userclient.requester = &FakeClient{
			Error: errors.New("Bad Request"),
		}

		_, err := userclient.Get()
		if err == nil {
			t.Error("Expected to get an error")
		}
	})

	t.Run("get userclient returns err if response body invalid", func(t *testing.T) {
		userclient.requester = &FakeClient{}

		_, err := userclient.Get()
		if err == nil {
			t.Error("Expected to get an error")
		}
	})
	t.Run("get userclient returns unmarshalled response body on success", func(t *testing.T) {

		userclient.requester = &FakeClient{
			Body: []byte(`{"status": "ok"}`),
		}
		got, _ := userclient.Get()
		want := map[string]interface{}{
			"status": "ok",
		}
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %q want %q", got, want)
		}

	})
}

func TestFetchAll(t *testing.T) {

	t.Run("fetchall returns empty list if task count is 0", func(t *testing.T) {

		userclient := &UserClient{
			ResultPerPage: 1,
			TaskCount:     0,
		}
		userclient.requester = &FakeClient{
			Body: []byte(`{"results": [{"test": "ok"}]`),
		}
		got, _ := userclient.FetchAll()
		want := []map[string]interface{}{}
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %q want %q", got, want)
		}

	})
	t.Run("fetchall returns empty list if per page is 0", func(t *testing.T) {

		userclient := &UserClient{
			ResultPerPage: 0,
			TaskCount:     1,
		}
		userclient.requester = &FakeClient{
			Body: []byte(`{"results": [{"test": "ok"}]`),
		}
		got, _ := userclient.FetchAll()
		want := []map[string]interface{}{}
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %q want %q", got, want)
		}

	})

	t.Run("fetchall returns the same amount of records with item count", func(t *testing.T) {

		userclient := &UserClient{
			TaskCount:     5,
			ResultPerPage: 2,
		}
		userclient.requester = &FakeClient{
			Body: []byte(`{"results": [{"test": "ok"}]`),
		}
		res, _ := userclient.FetchAll()
		got := len(res)
		want := 10
		if got != want {
			t.Errorf("got %d want %d", got, want)
		}

	})

}

````



