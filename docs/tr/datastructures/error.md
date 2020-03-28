# Error

GOda kendi hata tiplerimizi oluşturabiliriz. Bunun için oluşturduğumuz tipin sadece Error methoduna sahip olması yeterlidir.

### Tanımlama

Aşağıdaki örnekte HTTP kodunu ve hata nedenini belirten `HTTPError` adında yeni bir hata tanımladık.


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


### Kullanım

Fonksiyon çalıştırıldığında dönülen hata değerinin `nil` olup olmadığı kontrol edilerek buna göre önlemler alınabilir.


```go
err := MakeRequest()
if err != nil {
	fmt.Println(err)
}
```
