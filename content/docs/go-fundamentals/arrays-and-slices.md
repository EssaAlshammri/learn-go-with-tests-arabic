---
weight: 5
title: "المصفوفات (Arrays) والمصفوفات المرنة (Slices)"
---



**[يمكنك العثور على جميع الشفرات المصدرية لهذا الفصل هنا](https://github.com/quii/learn-go-with-tests/tree/main/arrays)**

تسمح لك المصفوفات بتخزين عناصر متعددة من نفس النوع في متغير بترتيب معين.

عندما يكون لديك مصفوفة، فمن الشائع جدًا أن تقوم بالتكرار عليها. لذلك دعونا
نستخدم [معرفتنا الجديدة بـ `for`](../iteration) لإنشاء دالة `Sum`. "مجموع" والتي ستقوم باخذ مجموعة من الأرقام وأرجع المجموع.

لنستخدم مهاراتنا في TDD

## لنكتب الاختبار أولاً

أنشئ مجلدًا جديدًا للعمل فيه. أنشئ ملفًا جديدًا يسمى `sum_test.go` واكتب ما يلي:

```go {filename="sum_test.go"}
package main

import "testing"

func TestSum(t *testing.T) {

	numbers := [5]int{1, 2, 3, 4, 5}

	got := Sum(numbers)
	want := 15

	if got != want {
		t.Errorf("got %d want %d given, %v", got, want, numbers)
	}
}
```

المصفوفات لها سعة ثابتة تحددها عندما تعلن عن المتغير. يمكننا انشاء المصفوفة بطريقتين:

* \[السعة\]النوع{القيمة1, القيمة2, ..., القيمة} على سبيل المثال. `numbers := [5]int{1, 2, 3, 4, 5}`
* \[...\]النوع{القيمة1, القيمة2, ..., القيمة} على سبيل المثال. `numbers := [...]int{1, 2, 3, 4, 5}`

من المفيد في بعض الأحيان أيضًا طباعة مدخلات الدالة في رسالة الخطأ.
هنا، نستخدم العنصر النائب `%v` لطباعة التنسيق "الافتراضي" للنوع، والذي يعمل بشكل جيد مع المصفوفات.

[اقرأ اكثر عن كيفية تنسيق النصوص](https://golang.org/pkg/fmt/)

## شغل الاختبار

إذا قمت بأنشاء المشروع باستخدام 

```text {filename="terminal"}
go mod init main
```

 فسوف يظهر لك خطأ 
 
 ```text {filename="terminal"}
 _testmain.go:13:2: cannot import "main"
 ```
 
ذلك لأنه وفقًا للممارسة الشائعة ستحتوي الحزمة الرئيسية (main) فقط على الحزم الأخرى وليس التعليمات البرمجية القابلة للاختبار للوحدة ومن ثم لن يسمح لك Go باستيراد حزمة بالاسم "main".

لإصلاح ذلك، يمكنك إعادة تسمية الوحدة الرئيسية في ملف `go.mod` إلى أي اسم آخر.

بمجرد إصلاح الخطأ أعلاه، إذا قمت بتشغيل `go test`، فسوف يفشل المترجم مع ارجاع هذه الرسالة `./sum_test.go:10:15: undefined: Sum`. يمكننا الآن متابعة كتابة الكود المراد اختباره.

## اكتب الحد الأدنى من الكود حتى نتمكن من تشغيل الاختبار لنتحقق من المخرجات الفاشلة

في `sum.go`

```go {filename="sum.go"}
package main

func Sum(numbers [5]int) int {
	return 0
}
```

الاختبار سيفشل الان ويقوم بطباعة _رسالة واضحة_

`sum_test.go:13: got 0 want 15 given, [1 2 3 4 5]`

## لنقم بكتابة الكود الان حتى ينجح الاختبار

```go {filename="sum.go"}
func Sum(numbers [5]int) int {
	sum := 0
	for i := 0; i < 5; i++ {
		sum += numbers[i]
	}
	return sum
}
```

للحصول على القيمة من مصفوفة في مكان معين داخلها، ما عليك سوى استخدام `array[index]`.  (index هو مكان العنصر داخل المصفوفة)
في هذه الحالة، نستخدم `for` للتكرار 5 مرات ونقوم بجلب العنصر من المصفوفة ثم نقوم باضافة كل عنصر إلى`sum`.

## إعادة الكتابة

دعنا نقدم [`range`](https://gobyexample.com/range) للمساعدة في تحسين الكود الخاص بنا

```go {filename="sum.go"}
func Sum(numbers [5]int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

تتيح لك `range` "المدى" التكرار عبر مصفوفة. في كل تكرار، تقوم `range`  بارجاع قيمتين الاولى - الفهرس (مكان العنصر داخل المصفوفة) والثانية - القيمة المخزنة في ذلك المكان.
قمنا تجاهل قيمة الفهرس باستخدام `_` [متغير فارغ](https://golang.org/doc/efficiency_go.html#blank). `_` يعني اننا لا نريد استخدام القيمة التي تكون بداخلة.

### المصفوفات وأنواعها

من الخصائص المثيرة للاهتمام للمصفوفات في Go أن الحجم يكون ضمن النوع. إن قمت بتمرير `[4]int` إلى دالة تتوقع `[5]int`، فلن يقبل بذلك المترجم. لانهم تختلف انواعهم، لذا فهي تمامًا مثل محاولة تمرير "سلسلة نصية" `string` إلى دالة تريد `int`.

ربما تعتقد أنه من المرهق جدًا أن يكون للمصفوفات حجم ثابت.

تحتوي Go على المصفوفات المرنة _slices_ والتي لا تقوم بتضمين حجم المجموعة مع النوع بل ويمكنها ايضا ان يكون لها اي حجم تريد.

سيكون الاختبار التالي هو جمع عناصر مصفوفات ذات أحجام مختلفة.

## اكتب الاختبار أولاً

سوف نستخدم الآن [نوع المصفوفات المرنة](https://go.dev/doc/effective_go#slices) الذي يسمح لنا بالحصول على مصفوفات باي حجم. بناء الجملة البرمجية مشابه جدًا للمصفوفات العادية، ما عليك سوى حذف الحجم عندما تقوم بإعلان المتغير

`mySlice := []int{1,2,3}` بدلا من `myArray := [3]int{1,2,3}`

```go {filename="sum_test.go"}
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := [5]int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

## قم الان بتشغيل الاختبار

لن يسمح لك المترجم بفعل ذلك وسيقوم بطباعة الخطأ التالي

```text {filename="terminal"}
./sum_test.go:22:13: cannot use numbers (type []int) as type [5]int in argument to Sum
```

## قم بكتابة ما يكفي حتى نرى مخرجات الاختبار الفاشل

المشكلة هنا تكمن في اننا امام خيارين

* نقوم بتغيير واجهة برمجة التطبيقات الحالية عن طريق تغيير المدخلات إلى `Sum` لتكون مصفوفة مرنة بدلاً من مصفوفة عادية. عندما نفعل هذا، فمن المحتمل أن ندمر  يوم شخص ما لأن اختباراتنا الاخرى لن يقوم المترجم بقبولهم! (على افتراض اننا قمنا ببرمجة برنامج كامل ومستخدم من قبل اناس اخرين)
* او اننا نقوم بإنشاء دالة جديدة.

في حالتنا، لا أحد يستخدم دالتنا حتى الان، لذا بدلاً من أن يكون لدينا دالتين نقوم بالتطوير عليهما، فلنحصل على واحدة فقط.

```go {filename="sum.go"}
func Sum(numbers []int) int {
	sum := 0
	for _, number := range numbers {
		sum += number
	}
	return sum
}
```

إذا حاولت تشغيل الاختبارات، فلن يتم يقبل المترجم حتى الان، وسيتعين عليك تغيير الاختبار الأول وتقوم بتمرير مصفوفة مرنة بدلاً من المصفوفة العادية.

## اكتب كود كافي لنجاح الاختبار

اتضح أن إصلاح مشاكل المترجم كان كل ما يتعين علينا القيام به هنا وستنجح الاختبارات الاختبارات!


## إعادة الكتابة

لقد قمنا بالفعل بإعادة كتابة `Sum` - كل ما فعلناه هو استبدال المصفوفات بالمصفوفات المرنة، لذلك لا يلزم إجراء تغييرات إضافية.
تذكر أنه يجب علينا ألا نهمل كود الاختبار الخاص بنا في مرحلة إعادة الكتابة - يمكننا تحسين اختبارات `sum` بشكل أكبر.

```go {filename="sum_test.go"}
func TestSum(t *testing.T) {

	t.Run("collection of 5 numbers", func(t *testing.T) {
		numbers := []int{1, 2, 3, 4, 5}

		got := Sum(numbers)
		want := 15

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

	t.Run("collection of any size", func(t *testing.T) {
		numbers := []int{1, 2, 3}

		got := Sum(numbers)
		want := 6

		if got != want {
			t.Errorf("got %d want %d given, %v", got, want, numbers)
		}
	})

}
```

من المهم ان تتسائل عن ماهية القيمة المقدمة من اختباراتك. لا يجب أن يكون الهدف من الاختبارات هو وجود أكبر عدد ممكن من الاختبارات، ولكن يجب أن يكون الهدف هو الحصول على أكبر قدر ممكن من الثقة في الكود الخاص بك. إذا كان لديك الكثير من الاختبارات، فقد يتحول الأمر إلى مشكلة حقيقية ويضيف المزيد من العبء في الصيانة. **كل اختبار له تكلفته**.


في حالتنا، يمكنك أن ترى أن وجود اختبارين لهذه الدالة هو غير ضروري. إذا قمت بكتابة اختبار لمصفوفة من اي حجم، فمن المرجح جدًا أنها ستعمل لمصفوفة من أي حجم اخر \(في حدود المعقول\).


Go تحتوي على أداة اختبار مدمجة تسمى [أداة التغطية](https://blog.golang.org/cover). تمكنك هذه الاداة من معرفة مدى تغطية اختباراتك لكودك من خلال اعطائك نسبة مئوية. على الرغم من أن السعي للحصول على 100% من التغطية ليس هو الهدف النهائي، إلا أن أداة التغطية يمكن أن تساعدك في تحديد المناطق التي لم تغطيها اختباراتك. إذا كنت صارمًا في TDD، فمن المرجح أن تكون لديك تقريبًا 100% من التغطية بالفعل.

قم بتجريب تشغيل اداة التغطية على الاختبارات الخاصة بك عن طريق تشغيل الأمر التالي

```text {filename="terminal"}
go test -cover
```

سيقوم بطباعة شيء مثل


```text {filename="terminal"}
PASS
coverage: 100.0% of statements
```

الان قم بحذف احد الاختبارات وقم بتشغيل اداة التغطية مرة اخرى


الان بعد ان قمت بكتابة الاختبارات والكود الخاص بك، يجب عليك ان تقوم بعمل commit للكود الخاص بك قبل ان تقوم بالمهمة القادمة.


نحتاج ان نكتب دالة جديدة تسمى `SumAll` والتي ستأخذ عدد متغير من المصفوفات المرنة، وتعيد مصفوفة مرنة جديدة تحتوي على المجموع لكل مصفوفة تم تمريرها.

على سبيل المثال 

`SumAll([]int{1,2}, []int{0,9})` ستقوم بأرجاع `[]int{3, 9}`

او

`SumAll([]int{1,1,1})` ستقوم بأرجاع `[]int{3}`

## لنكتب الاختبار أولاً


```go {filename="sum_test.go"}
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if got != want {
		t.Errorf("got %v want %v", got, want)
	}
}
```

قم بتشغيل الاختبار الان

`./sum_test.go:23:9: undefined: SumAll`

## اكتب الكود الكافي لجعل الاختبار يعمل ويرجع نتيجة الفشل

نحتاج الان لكتابة دالة `SumAll` وفقا لما يريده اختبارنا.


Go تمكنك من كتابة [_دوال تأخذ عددا متغير من المدخلات (variadic)_](https://gobyexample.com/variadic-functions) التي يمكن أن تأخذ عددًا متغيرًا من المدخلات.



```go go {filename="sum.go"}
func SumAll(numbersToSum ...[]int) []int {
	return nil
}
```

هذا الكود سليم لكن الاختبارات لن تعمل بعد! بسبب المترجم

`./sum_test.go:26:9: invalid operation: got != want (slice can only be compared to nil)`

Go لا تسمح لك باستخدام عمليات المقارنة مع المصفوفات المرنة. يمكنك كتابة دالة للتكرار على كل `got` و `want` والتحقق من قيمهم ولكن للتسهيل، يمكننا استخدام [`reflect.DeepEqual`](https://golang.org/pkg/reflect/#DeepEqual) والتي تعتبر مفيدة لمعرفة ما إذا كان أي متغيرين متطابقين.


```go {filename="sum_test.go"}
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := []int{3, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

تأكد من انك قمت بأستدعاء `import reflect` في اعلى الملف لكي تتمكن من استخدام `DeepEqual`


يجب ان تلاحظ ان `reflect.DeepEqual` ليست "آمنة" - سيتم  تجميع وترجمة الكود حتى لو قمت بكتابة شئ غير صحيح. لرؤية ذلك قم بتغيير الاختبار مؤقتًا إلى:

```go {filename="sum_test.go"}
func TestSumAll(t *testing.T) {

	got := SumAll([]int{1, 2}, []int{0, 9})
	want := "bob"

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

ما قمنا بفعلة هنا هو محاولة مقارنة مصفوفة مرنة مع سلسلة نصية. هذا لا يمكن ان يحدث او ان يكون منطقياً، ولكن الاختبار يترجم! لذلك، على الرغم من أن استخدام `reflect.DeepEqual` طريقة مريحة لمقارنة المصفوفات \(وأشياء أخرى\) يجب أن تكون حذرًا عند استخدامها.


في الاصدار 1.21 من Go، توجد حزمة قياسية تسمى [slices](https://pkg.go.dev/slices#pkg-overview)، والتي تحتوي على [slices.Equal](https://pkg.go.dev/slices#Equal) والتي تقوم بعملية مقارنة بسيطة على المصفوفات المرنة، حيث لا تحتاج للقلق بشأن الامان مثل الحالة السابقة. يجب ملاحظة أن هذه الدالة تتوقع أن تكون العناصر [قابلة للمقارنة](https://pkg.go.dev/builtin#comparable). لذلك، لا يمكن تطبيق هذه الدالة على المصفوفات التي تحتوي على عناصر غير قابلة للمقارنة مثل المصفوفات ثنائية الأبعاد.

قم بأعادة الاختبار مرة اخرى وقم بتشغيله. يجب ان تحصل على نتيجة الاختبار التالية

`sum_test.go:30: got [] want [3 9]`

## لنقم بكتابة كود كافي لجعل الاختبار ينجح

هنا نحتاج إلى التكرار على المدخلات المتغيرة، وحساب المجموع باستخدام دالتنا `Sum` السابقة، ثم إضافتها إلى المصفوفة التي سنعيدها

```go {filename="sum.go"}
func SumAll(numbersToSum ...[]int) []int {
	lengthOfNumbers := len(numbersToSum)
	sums := make([]int, lengthOfNumbers)

	for i, numbers := range numbersToSum {
		sums[i] = Sum(numbers)
	}

	return sums
}
```

اشياء جديدة كثيرة لتعلمها!

هنالك طريقة جديدة لانشاء مصفوفة. `make` تسمح لك بانشاء مصفوفة بسعة ابتدائية تساوي `len` (حجم المصفوفة المدخلة) من `numbersToSum` التي نحتاج العمل عليها.

بأمكاننا الحصول على عنصر داخل المصفوفة المرنة بأستخدام الفهرسة مثلما كنا نفعل مع المصفوفة العادية `mySlice[N]` للحصول على قيمة العنصر او تعديلة مثلا.

يجب ان ينجح الاختبار الان


## Refactor

As mentioned, slices have a capacity. If you have a slice with a capacity of
2 and try to do `mySlice[10] = 1` you will get a _runtime_ error.

However, you can use the `append` function which takes a slice and a new value,
then returns a new slice with all the items in it.

```go
func SumAll(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		sums = append(sums, Sum(numbers))
	}

	return sums
}
```

In this implementation, we are worrying less about capacity. We start with an
empty slice `sums` and append to it the result of `Sum` as we work through the varargs.

Our next requirement is to change `SumAll` to `SumAllTails`, where it will
calculate the totals of the "tails" of each slice. The tail of a collection is
all items in the collection except the first one \(the "head"\).

## Write the test first

```go
func TestSumAllTails(t *testing.T) {
	got := SumAllTails([]int{1, 2}, []int{0, 9})
	want := []int{2, 9}

	if !reflect.DeepEqual(got, want) {
		t.Errorf("got %v want %v", got, want)
	}
}
```

## Try and run the test

`./sum_test.go:26:9: undefined: SumAllTails`

## Write the minimal amount of code for the test to run and check the failing test output

Rename the function to `SumAllTails` and re-run the test

`sum_test.go:30: got [3 9] want [2 9]`

## Write enough code to make it pass

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		tail := numbers[1:]
		sums = append(sums, Sum(tail))
	}

	return sums
}
```

Slices can be sliced! The syntax is `slice[low:high]`. If you omit the value on
one of the sides of the `:` it captures everything to that side of it. In our
case, we are saying "take from 1 to the end" with `numbers[1:]`. You may wish to
spend some time writing other tests around slices and experiment with the
slice operator to get more familiar with it.

## Refactor

Not a lot to refactor this time.

What do you think would happen if you passed in an empty slice into our
function? What is the "tail" of an empty slice? What happens when you tell Go to
capture all elements from `myEmptySlice[1:]`?

## Write the test first

```go
func TestSumAllTails(t *testing.T) {

	t.Run("make the sums of some slices", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}

		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	})

}
```

## Try and run the test

```text
panic: runtime error: slice bounds out of range [recovered]
    panic: runtime error: slice bounds out of range
```

Oh no! It's important to note that while the test _has compiled_, it _has a runtime error_.  

Compile time errors are our friend because they help us write software that works,  
runtime errors are our enemies because they affect our users.

## Write enough code to make it pass

```go
func SumAllTails(numbersToSum ...[]int) []int {
	var sums []int
	for _, numbers := range numbersToSum {
		if len(numbers) == 0 {
			sums = append(sums, 0)
		} else {
			tail := numbers[1:]
			sums = append(sums, Sum(tail))
		}
	}

	return sums
}
```

## Refactor

Our tests have some repeated code around the assertions again, so let's extract those into a function.

```go
func TestSumAllTails(t *testing.T) {

	checkSums := func(t testing.TB, got, want []int) {
		t.Helper()
		if !reflect.DeepEqual(got, want) {
			t.Errorf("got %v want %v", got, want)
		}
	}

	t.Run("make the sums of tails of", func(t *testing.T) {
		got := SumAllTails([]int{1, 2}, []int{0, 9})
		want := []int{2, 9}
		checkSums(t, got, want)
	})

	t.Run("safely sum empty slices", func(t *testing.T) {
		got := SumAllTails([]int{}, []int{3, 4, 5})
		want := []int{0, 9}
		checkSums(t, got, want)
	})

}
```

We could've created a new function `checkSums` like we normally do, but in this case, we're showing a new technique, assigning a function to a variable. It might look strange but, it's no different to assigning a variable to a `string`, or an `int`, functions in effect are values too. 

It's not shown here, but this technique can be useful when you want to bind a function to other local variables in "scope" (e.g between some `{}`). It also allows you to reduce the surface area of your API. 

By defining this function inside the test, it cannot be used by other functions in this package. Hiding variables and functions that don't need to be exported is an important design consideration.

A handy side-effect of this is this adds a little type-safety to our code. If
a developer mistakenly adds a new test with `checkSums(t, got, "dave")` the compiler
will stop them in their tracks.

```bash
$ go test
./sum_test.go:52:21: cannot use "dave" (type string) as type []int in argument to checkSums
```

## Wrapping up

We have covered

* Arrays
* Slices
  * The various ways to make them
  * How they have a _fixed_ capacity but you can create new slices from old ones
    using `append`
  * How to slice, slices!
* `len` to get the length of an array or slice
* Test coverage tool
* `reflect.DeepEqual` and why it's useful but can reduce the type-safety of your code

We've used slices and arrays with integers but they work with any other type
too, including arrays/slices themselves. So you can declare a variable of
`[][]string` if you need to.

[Check out the Go blog post on slices][blog-slice] for an in-depth look into
slices. Try writing more tests to solidify what you learn from reading it.

Another handy way to experiment with Go other than writing tests is the Go
playground. You can try most things out and you can easily share your code if
you need to ask questions. [I have made a go playground with a slice in it for you to experiment with.](https://play.golang.org/p/ICCWcRGIO68)

[Here is an example](https://play.golang.org/p/bTrRmYfNYCp) of slicing an array
and how changing the slice affects the original array; but a "copy" of the slice
will not affect the original array.
[Another example](https://play.golang.org/p/Poth8JS28sc) of why it's a good idea
to make a copy of a slice after slicing a very large slice.

[for]: ../iteration.md#
[blog-slice]: https://blog.golang.org/go-slices-usage-and-internals
[deepEqual]: https://golang.org/pkg/reflect/#DeepEqual
[slice]: https://golang.org/doc/effective_go.html#slices