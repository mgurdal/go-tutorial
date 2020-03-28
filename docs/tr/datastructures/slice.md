# Slice

Slicelar kullanım bakımından arraylere çok benzer. Sliceların farkı, boyutlarının dinamik olması.

Slice tanımlarken yine arraydeki gibi `[]Tip` yapısını kullanıyoruz fakat köşeli parantezlerin içini boş bırakıyoruz.


```go
// nil slice tanımlama
var slc []int
// nill slice'a değer atama
slc = []int{12, 24, 39}  

// doğrudan tanımlama
slc := []int{12, 24, 39} 
```


#### make

Sliceları tanımlamak için \`make\` fonksiyonunu da kullanabiliriz. `make` fonksiyonunun ilk parametresi oluşturulacak sliceın boyutunu belirler.


```go
// boyut ile tanımlama
n_items = 20
slc := make([]int, n_items)

// boyut ve kapasite ile tanımlama
capacity := 40
slc := make([]int, n_items, capacity)
```


### Slice İşlemleri

Slice oluşturulurken kapasitesi kadar, tutacağı veri tipinin ön tanımlı değeri ile doldurulur. Fakat, yalnızca uzunluğu kadar kısmına erişilebilir.

Bir sliceın boyutunu artırmak istiyorsak bunun için `append` ve `copy` fonksiyonlarını kullanabiliriz.

### copy

`copy` bir sliceı aynı veya daha büyük kapasiteye sahip bir diğer slicea kopyalamamızı sağlar ve  kaç adet elemanın kopyalandığını döndürür.


```go
bucket := make([]string, 5)

fruits := []string{"🍎", "🍇", "🍒", "🍉"}

n_copied := copy(bucket, fruits)

n_copied // 5
bucket // [🍎 🍇 🍈 🍉]

```

Bir slice kopyalanırken, elemanlar hedef slicedaki aynı indexdeki elemanların yerini alır.


```go
fruits := []string{"🍇", "🍒", "🍉"}
apples := []string{"🍎", "",  "🍏"}

copy(fruits, apples)

fruits // [🍎  🍏]

```


### append

`append` fonksiyonu, ilk parametre olarak boyutunu artıracağımız sliceı ve devamında ekleyeceğimiz elemanları alır ve bize tüm elemanları içeren **yeni** bir slice döndürür. Bu nedenle, dönüş değerini bir değişkende\(genellikle eski slice değişkeninde\) tutmak gerekir.


```go
bucket := []string{"🍎", "🍇", "🍈", "🍉"}

green_apple := "🍏"
bucket = append(bucket, green_apple)

bucket // [🍎 🍇 🍈 🍉 🍏]
```


> Bir sliceı yeniden boyutlandırıp başka bir değişkene atadığımızda, yeni slicedaki değişiklikler orjinal sliceı da etkileyecektir.


```go
fruits := []string{"🍎", "🍎", "🍇", "🍒", "🍉"}
apples := fruits[:2]

apples[0] = "🍏"
apples[1] = "🍏"
 
apples // [🍏 🍏]
fruits // [🍏 🍏 🍇 🍒 🍉]


```


### cut

Slicedaki belli bir aralığı çıkarmak istiyorsak aşağıdaki yöntemi kullanabiliriz. Bu yöntem Sliceda belirlediğimiz başlangıç indexine kadar olan elemanlar ile bitiş indexinden sonraki elemanları ekleyecek. Böylece aradaki elemanları boşa çıkartmış olacağız.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

cut_start_index, cut_end_index := 4, 6
 
edibles = append(edibles[:cut_start_index], edibles[cut_end_index:]...)

edibles // [🍎 🍇 🍒 🍉 🌽 🥔]
```


### delete

Slicedaki bir elemanı indexini kullanarak silmek istiyorsak, aşağıdaki yöntemi kullanabiliriz.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}

index_of_cherry := 2
edibles = append(edibles[:index_of_cherry], edibles[index_of_cherry+1:]...)

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```


Benzer işlemi `copy` fonksiyonu ile de yapabiliriz. Aşağıdaki örnekte, yukarıdakinden farklı olarak 4. satırda:

* Önce`edible` sliceında 🍒elemanından sonraki kısmı seçtik.  Bu bize, tüm kapasiteyi de gösterirsek, \[🍒 🍉 🌶 🍆 🌽 🥔\] biçiminde bir slice verdi.
* Sonrasında 🍒elemanından `1` index sonraki kısmı, seçilen slice ile _değiştirdik_. Böylece \[🍎 🍇 🍉 🌶 🍆 🌽 🥔 🥔\] listesini elde etmiş olduk.

5. satırda da sliceı yeni eleman sayısına göre dilimledik.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

n_copied := copy(edibles[index_of_cherry:], edibles[index_of_cherry+1:])
new_length := index_of_cherry + n_copied

edibles = edibles[:new_length]

edibles // [🍎 🍇 🍉 🌶 🍆 🌽 🥔]
```


#### Sıralamayı Dikkate Almadan Silme

Silme işlemini array sıralamasını göz ardı ederek de yapabiliriz. Aşağıdaki örnekte, silmek istediğimiz eleman ile slicedaki en son elemanın yerini değiştirdik. Ardından sliceın boyutunu 1 azalttık.


```go
edibles := []string{"🍎", "🍇", "🍒", "🍉", "🌶", "🍆", "🌽", "🥔"}
index_of_cherry := 2

edibles[index_of_cherry] = edibles[len(edibles)-1]
edibles = edibles[:len(edibles)-1]

edibles // [🍎 🍇 🥔 🍉 🌶 🍆 🌽]
```

> **Garbage Collection**
> Eğer sliceda kullandığımız elemanlar `pointer` veya `pointer` alanlara sahip `struct` ise yukarıda gösterilen **cut**  ve **delete** işlemleri memory leak oluşturabilir. Bunun sebebi, sliceın silindikten sonra da silinen elemanın referansını tutabilmesindendir. 

Aşağıdaki örneklerde daha öncesinde yaptığımız işlemlere ek olarak, işlemdem sonra boşta kalan alanları `nil` değerler ile doldurduk. Böylece garbage collector bu alanların artık kullanılmayacağını anlayıp, sisteme geri kazandırabilir.

### cut \(memory safe\)


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


### delete \(memory safe\)


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


#### Sıralamayı Dikkate Almadan Silme \(memory safe\)


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


### expand

Bir sliceı, diğeri birleştirmek istersek aşağıdaki yöntemleri kullanabiliriz.


```go
bucket := []string{"🍌", "🍇", "🍉"}
apples := []string{"🍏", "🍎"}

bucket = append(bucket, apples...)

bucket // [🍌 🍇 🍉 🍏 🍎]
```

Aşağıdaki örnekte `7.` satırda `apples` ile belirlediğimiz indexten sonraki elemanları birleştirdik. Sonrasında belirlediğimiz indexe kadar olan `bucket` elemanlarına, bu slice elemanlarını ekledik.


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


### filter

Slice elemanlarını belirli bir kritere göre filtreleyebiliriz.


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

### insert

Sliceın herhangi bir kısmına yeni bir eleman eklemek istiyorsak; bu yöntemleri kullanabiliriz.


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


{% hint style="warning" %}
**Memory Concern**

7. satırdaki`append([]string{banana}, fruits[insert_index:]...)` ****bölümü başlı başına bir slice oluşturup,`fruits[insert_index:]`sliceının elemanlarını bu slicea kopyalıyor. Bu elemanlar daha sonra `fruits` sliceına tekrar kopyalanıyor.

Yeni bir slice oluşturulmasının ve 2. kez kopyalama işleminin önüne geçilebilir.
{% endhint %}


```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"
insert_index := 2

fruits = append(fruits, "")
copy(fruits[insert_index+1:], fruits[insert_index:])
fruits[insert_index] = banana

fruits // [🍏 🍎 🍌 🍉]
```


### pop

Aşağıdaki örnekte sliceın son elemanını çıkarıp bir değişkene atadık.


```go
fruits := []string{"🍏", "🍎", "🍉"}

item, fruits := fruits[len(fruits)-1], fruits[:len(fruits)-1]

item // 🍉
fruits // [🍏 🍎]
```


### push front \(unshift\)

Bu örnekte yeni bir elemanı sliceın başına koyduk.


```go
fruits := []string{"🍏", "🍎", "🍉"}
banana := "🍌"

fruits = append([]string{banana}, fruits...)

fruits // [🍌 🍏 🍎 🍉]
```


### pop front \(shift\)

Bu örnekte de sliceın ilk elemanını çıkarıp bir değişkene atadık.


```go
fruits := []string{"🍌", "🍏", "🍎", "🍉"}

banana, fruits := fruits[0], fruits[1:]

banana // 🍌
fruits // [🍏 🍎 🍉]
```
