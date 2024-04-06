---
weight: 6
title: "البنى والتوابع والواجهات"
---

# البنى (Structs), التوابع (methods) الواجهات (interfaces)

**[يمكنك العثور على جميع الشفرات المصدرية لهذا الفصل هنا](https://github.com/quii/learn-go-with-tests/tree/main/structs)**

لنفترض اننا نحتاج لكود هندسي لحساب محيط مستطيل ما بعرض وارتفاع ما. يمكننا كتابة دالة

`Perimeter(width float64, height float64)` 

حيث `float64` هو نوع للارقام العشرية مثل `123.45`.

روتين TDD اصبح واضحا لك الان

## نكتب الاختبار اولا

```go {filename="shapes_test.go"}
func TestPerimeter(t *testing.T) {
	got := Perimeter(10.0, 10.0)
	want := 40.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}
```

لاحظ كيفية تنسيق الرسالة الخطأ. يمكنك استخدام النائب `%.2f` لطباعة رقم عشري برقمين بعد النقطة.

## جرب تشغيل الاختبار

`./shapes_test.go:6:9: undefined: Perimeter`

## لنكتب الحد الادنى من الكود لتشغيل الاختبار والتحقق من النتائج الفاشلة


```go {filename="shapes.go"}
func Perimeter(width float64, height float64) float64 {
	return 0
}
```

النتيجة بعد تشغيل الاختبار

 `shapes_test.go:10: got 0.00 want 40.00`.

## لنكتب الحد الادنى من الكود لنجاح الاختبار


```go {filename="shapes.go"}
func Perimeter(width float64, height float64) float64 {
	return 2 * (width + height)
}
```

حتى الان كل شي سهل وبسيط. الان دعنا نكتب دالة `Area(width, height float64)` التي تقوم بحساب مساحة المستطيل وترجعة.

حاول ان تقوم بذلك بنفسك، وفقا لدورة TDD.

يجب ان يكون لديك اختبار مشابه لهذا

```go {filename="shapes_test.go"}
func TestPerimeter(t *testing.T) {
	got := Perimeter(10.0, 10.0)
	want := 40.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}

func TestArea(t *testing.T) {
	got := Area(12.0, 6.0)
	want := 72.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}
```

وداله جديدة مثل هذه

```go {filename="shapes.go"}
func Perimeter(width float64, height float64) float64 {
	return 2 * (width + height)
}

func Area(width float64, height float64) float64 {
	return width * height
}
```

## لنعيد الكتابة الان

الكود الذي قمنا بكتابته يقوم بما هو مطلوب منه بالشكل الصحيح، ولكنه لا يحتوي على اي شيء يدل على انه يتعامل مع المستطيلات. مطور غير متمكن قد يحاول استخدام هذه الدوال مع مثلث مثلا دون ان يدرك ان النتيجة ستكون خاطئة.

بأمكاننا اعطاء الدوال اسماء اكثر تحديدا مثل `RectangleArea`. لكن الحل الافضل هو تعريف نوع خاص بنا يسمى `Rectangle` يحتوي على هذا المفهوم.

يمكننا انشاء نوع بسيط باستخدام البنى (**struct**). [struct (البنى)](https://golang.org/ref/spec#Struct_types) هي مجموعة مسماة تحوي العديد من الحقول حيث يمكنك تخزين البيانات.

يتم الاعلان وتعريف البنى كالتالي

```go {filename="shapes.go"}
type Rectangle struct {
	Width  float64
	Height float64
}
```

الان دعونا نعيد كتابة الاختبارات لتستخدم `Rectangle` بدلا من `float64`

```go {filename="shapes_test.go"}
func TestPerimeter(t *testing.T) {
	rectangle := Rectangle{10.0, 10.0}
	got := Perimeter(rectangle)
	want := 40.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}

func TestArea(t *testing.T) {
	rectangle := Rectangle{12.0, 6.0}
	got := Area(rectangle)
	want := 72.0

	if got != want {
		t.Errorf("got %.2f want %.2f", got, want)
	}
}
```

تذكر انه يمكنك تشغيل الاختبارات قبل الشروع في اصلاح الكود وستقوم بدورها بأرجاع رسائل مفيدة

```text {filename="terminal"}
./shapes_test.go:7:18: not enough arguments in call to Perimeter
    have (Rectangle)
    want (float64, float64)
```

يمكنك الوصول الى حقول البنى بالطريقة التالية `myStruct.field`.

قم بتغيير الدوال لتصحيح الاختبار

```go {filename="shapes.go"}
func Perimeter(rectangle Rectangle) float64 {
	return 2 * (rectangle.Width + rectangle.Height)
}

func Area(rectangle Rectangle) float64 {
	return rectangle.Width * rectangle.Height
}
```

اتمنى ان توافقنا على ان استخدام `Rectangle` في الدوال يوضح النية بشكل افضل، ولكن هناك المزيد من الفوائد الاخرى في استخدام البنى سنغطيها لاحقا.


متطلبنا القادم هو كتابة دالة `Area` لحساب مساحة الدوائر ايضاً.

## نكتب الاختبار اولا

```go {filename="shapes_test.go"}
func TestArea(t *testing.T) {

	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12, 6}
		got := Area(rectangle)
		want := 72.0

		if got != want {
			t.Errorf("got %g want %g", got, want)
		}
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		got := Area(circle)
		want := 314.1592653589793

		if got != want {
			t.Errorf("got %g want %g", got, want)
		}
	})

}
```

ربما تلاحظ اننا قمنا بأستبدال النائب `f` بالنائب الاخر `g` لاسباب جيدة. استخدام النائب `g` سيطبع رقم عشري اكثر دقة في رسالة الخطأ. على سبيل المثال، عند استخدام نصف قطر 1.5 في حساب مساحة الدائرة، النائب `f` سيطبع `7.068583` بينما النائب `g` سيطبع `7.0685834705770345`.

## قم بتشغيل الاختبار

`./shapes_test.go:28:13: undefined: Circle`

## لنكتب ما يكفي لتشغيل الاختبار والتحقق من النتائج الفاشلة

نحتاج لتعريف نوع `Circle` الخاص بنا


```go {filename="shapes.go"}
type Circle struct {
	Radius float64
}
```

الان قم بتشغيل الاختبار مجددا 

`./shapes_test.go:29:14: cannot use circle (type Circle) as type Rectangle in argument to Area`

رسالة الخطأ تفيد بأنه لا يمكن استخدام `Circle` بدلا من `Rectangle` في الدالة `Area`.

هنالك بعض اللغات البرمجية التي تسمح لك بفعل شيء مثل هذا

```go
func Area(circle Circle) float64       {}
func Area(rectangle Rectangle) float64 {}
```

لكن هذا غير ممكن في Go

`./shapes.go:20:32: Area redeclared in this block`

نحن الان امام خيارين:

* اما ان نقوم بكتابة الدالة في حزمة جديدة خاصة بالدوائر. 
* او ان نقوم بكتابة دوال توابع للانواع التي لدينا.

والخيار الثاني هو الافضل في هذه الحالة.

### ماهي التوابع؟

حتى الان كنا نكتب دوال فقط ولكننا استخدمنا بعض التوابع. عندما نقوم بكتابة `t.Errorf` نقوم بأستدعاء تابع `Errorf` على النسخة `t` من \(`testing.T`\).

التوابع هي دوال تحتوي على متغير يسمى المستقبل \(receiver\).
وعند تعريف التابع يتم ربط المعرف، اسم التابع، بالتابع، ويربط التابع بنوع المستقبل.

التوابع مشابهة جدا للدوال ولكن يتم استدعائها عن طريق نسخة من نوع معين (شئ). حيث يمكنك استدعاء الدوال في اي مكان تريد، مثل `Area(rectangle)` يمكنك استدعاء التوابع فقط على "الاشياء".

لنأخذ مثالا لايضاح ذلك، دعنا نقم بتغيير الاختبارات لاستدعاء التوابع بدلا من الدوال ومن ثم نقوم بتصحيح الكود.

```go
func TestArea(t *testing.T) {

	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12, 6}
		got := rectangle.Area()
		want := 72.0

		if got != want {
			t.Errorf("got %g want %g", got, want)
		}
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		got := circle.Area()
		want := 314.1592653589793

		if got != want {
			t.Errorf("got %g want %g", got, want)
		}
	})

}
```

اذا قمنا بتشغيل الاختبارات الان 

```text {filename="terminal"}
./shapes_test.go:19:19: rectangle.Area undefined (type Rectangle has no field or method Area)
./shapes_test.go:29:16: circle.Area undefined (type Circle has no field or method Area)
```

> النوع Rectangle و Circle لا يحتوي على حقل او تابع يسمى Area

نود التوقف هنا للحظة ونلاحظ فوائد المترجم. من المهم جدا ان تقرأ رسائل الخطأ بتأني، ستساعدك كثيرا في المستقبل.

## لنكتب ما يكفي لتشغيل الاختبار والتحقق من النتائج الفاشلة

لنقم بإضافة بعض التوابع لانواعنا

```go {filename="shapes.go"}
type Rectangle struct {
	Width  float64
	Height float64
}

func (r Rectangle) Area() float64 {
	return 0
}

type Circle struct {
	Radius float64
}

func (c Circle) Area() float64 {
	return 0
}
```

طريقة تعريف التابع مشابهة جدا للدوال ولكن الفرق الوحيد هو في تعريف المستقبل

`func (receiverName ReceiverType) MethodName(args)`.

وعند استدعاء التابع على متغير من هذا النوع، ستصل الى بياناته عن طريق المتغير `receiverName`.

ومن المتعارف عليه في Go ان يكون اسم المتغير المستقبل اول حرف من النوع.

```
r Rectangle
```

اذا قمت بتشغيل الاختبارات الان  ستحصل على رسالة خطأ

## لنكتب ما يكفي لتشغيل الاختبار والتحقق من نجاح الاختبار

الان لنصلح الدوال لتعمل بشكل صحيح

```go {filename="shapes.go"}
func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}
```

اذا قمت بتشغيل الاختبارات الان ستجد ان الاختبارات الخاصة بالمستطيل تمر بنجاح ولكن الدوائر لا تمر.

To make circle's `Area` function pass we will borrow the `Pi` constant from the `math` package \(remember to import it\).

حتى نصلح اختبار الدائرة، سنقوم بأستعارة الثابت `Pi` من حزمة `math` \(لا تنسى استيرادها\).

```go {filename="shapes.go"}
func (c Circle) Area() float64 {
	return math.Pi * c.Radius * c.Radius
}
```

## اعادة الكتابة

هنالك تكرار في اختباراتنا.

كل ما نريدة هو اخذ مجموعة من الاشكال، استدعاء تابع `Area()` عليها ومن ثم التحقق من النتيجة.

نريد ان نكون قادرين على كتابة دالة `checkArea` يمكننا تمرير `Rectangle` و `Circle` اليها، ولكن لا يمكننا تمرير شيء ليس شكل.

في Go بالامكان تحقيق هذا الهدف عن طريق **الواجهات**.

[الواجهات](https://golang.org/ref/spec#Interface_types) هي مفهوم قوي جدا في اللغات ذات النوع الثابت مثل Go لانها تسمح لك بكتابة دوال يمكن استخدامها مع انواع مختلفة وتصميم الكود بشكل مستقل بشكل كبير وفي نفس الوقت للحفاظ على سلامة النوع.

لنتعلم ذلك من خلال اعادة كتابة الاختبارات.

```go {filename="shapes_test.go"}
func TestArea(t *testing.T) {

	checkArea := func(t testing.TB, shape Shape, want float64) {
		t.Helper()
		got := shape.Area()
		if got != want {
			t.Errorf("got %g want %g", got, want)
		}
	}

	t.Run("rectangles", func(t *testing.T) {
		rectangle := Rectangle{12, 6}
		checkArea(t, rectangle, 72.0)
	})

	t.Run("circles", func(t *testing.T) {
		circle := Circle{10}
		checkArea(t, circle, 314.1592653589793)
	})

}
```

هنا قمنا بكتابة داله مساعدة مثلما قمنا في التمارين السابقة ولكن هذه المرة نطلب منك ان تمرر `Shape` الى الدالة. اذا حاولت استدعاء هذه الدالة بشيء ليس شكل، فلن يتم الترجمة.

كيف يمكن لاي نوع ان يصبح شكل؟ نقوم بأخبار Go ما هو `Shape` عن طريق تعريف واجهة

```go {filename="shapes.go"}
type Shape interface {
	Area() float64
}
```

هنا كل ما قمنا به هو تعريف نوع جديد مثلما قمنا بذلك مع `Rectangle` و `Circle` ولكن هذه المرة هو واجهة بدلا من بنية.

بمجرد اضافة هذا الى الكود، ستنجح الاختبارات!.

### ماذا قلت؟

هذا مختلف تماما عن الواجهات في معظم لغات البرمجة الاخرى. عادة ما يجب عليك كتابة الكود لتقول `نوعي Foo يتبع الواجهة Bar`.

ايه انه يجب عليك اعلان ان النوع الفلاني يتبع الواجهة الفلانية.

لكن في حالتنا هنا

* `Rectangle` لديه تابع يسمى `Area` يرجع `float64` لذلك يتوافق مع الواجهة `Shape`
* `Circle` لديه تابع يسمى `Area` يرجع `float64` لذلك يتوافق مع الواجهة `Shape`
* `string` ليس لديه تابع مثل هذا، لذلك لا يتوافق مع الواجهة
* وهلم جرا

في Go **التوافق مع الوجهات يتم بشكل ضمني**. اذا كان النوع الذي تمرره يتوافق مع ما تطلبه الواجهة، سيتم الترجمة.

### (Decoupling) فض الاقتران او الفصل 

لاحظ كيف ان دالتنا المساعدة لا تحتاج الى القلق بشأن ما اذا كان الشكل هو `Rectangle` او `Circle` او `Triangle`. عن طريق تعريف واجهة، تكون الدالة المساعدة _مفصولة_ عن الانواع الفعلية ولديها فقط التابع الذي تحتاجه للقيام بعملها.

هذه الطريقة في استخدام الواجهات لتعريف **فقط ما تحتاج اليه** هي مهمة جدا في تصميم البرمجيات وسيتم تغطيتها بتفصيل اكثر في الاقسام القادمة.

## المزيد من اعادة الكتابة 

الان وبما انه لديك بعض الفهم حول البنى يمكننا ان نقدم لك "الاختبارات المجدولة".

[Table driven tests](https://go.dev/wiki/TableDrivenTests) are useful when you want to build a list of test cases that can be tested in the same manner.

```go
func TestArea(t *testing.T) {

	areaTests := []struct {
		shape Shape
		want  float64
	}{
		{Rectangle{12, 6}, 72.0},
		{Circle{10}, 314.1592653589793},
	}

	for _, tt := range areaTests {
		got := tt.shape.Area()
		if got != tt.want {
			t.Errorf("got %g want %g", got, tt.want)
		}
	}

}
```

The only new syntax here is creating an "anonymous struct", `areaTests`. We are declaring a slice of structs by using `[]struct` with two fields, the `shape` and the `want`. Then we fill the slice with cases.

We then iterate over them just like we do any other slice, using the struct fields to run our tests.

You can see how it would be very easy for a developer to introduce a new shape, implement `Area` and then add it to the test cases. In addition, if a bug is found with `Area` it is very easy to add a new test case to exercise it before fixing it.

Table driven tests can be a great item in your toolbox, but be sure that you have a need for the extra noise in the tests.
They are a great fit when you wish to test various implementations of an interface, or if the data being passed in to a function has lots of different requirements that need testing.

Let's demonstrate all this by adding another shape and testing it; a triangle.

## Write the test first

Adding a new test for our new shape is very easy. Just add `{Triangle{12, 6}, 36.0},` to our list.

```go
func TestArea(t *testing.T) {

	areaTests := []struct {
		shape Shape
		want  float64
	}{
		{Rectangle{12, 6}, 72.0},
		{Circle{10}, 314.1592653589793},
		{Triangle{12, 6}, 36.0},
	}

	for _, tt := range areaTests {
		got := tt.shape.Area()
		if got != tt.want {
			t.Errorf("got %g want %g", got, tt.want)
		}
	}

}
```

## Try to run the test

Remember, keep trying to run the test and let the compiler guide you toward a solution.

## Write the minimal amount of code for the test to run and check the failing test output

`./shapes_test.go:25:4: undefined: Triangle`

We have not defined `Triangle` yet

```go
type Triangle struct {
	Base   float64
	Height float64
}
```

Try again

```text
./shapes_test.go:25:8: cannot use Triangle literal (type Triangle) as type Shape in field value:
    Triangle does not implement Shape (missing Area method)
```

It's telling us we cannot use a `Triangle` as a shape because it does not have an `Area()` method, so add an empty implementation to get the test working

```go
func (t Triangle) Area() float64 {
	return 0
}
```

Finally the code compiles and we get our error

`shapes_test.go:31: got 0.00 want 36.00`

## Write enough code to make it pass

```go
func (t Triangle) Area() float64 {
	return (t.Base * t.Height) * 0.5
}
```

And our tests pass!

## Refactor

Again, the implementation is fine but our tests could do with some improvement.

When you scan this

```
{Rectangle{12, 6}, 72.0},
{Circle{10}, 314.1592653589793},
{Triangle{12, 6}, 36.0},
```

It's not immediately clear what all the numbers represent and you should be aiming for your tests to be easily understood.

So far you've only been shown syntax for creating instances of structs `MyStruct{val1, val2}` but you can optionally name the fields.

Let's see what it looks like

```
        {shape: Rectangle{Width: 12, Height: 6}, want: 72.0},
        {shape: Circle{Radius: 10}, want: 314.1592653589793},
        {shape: Triangle{Base: 12, Height: 6}, want: 36.0},
```

In [Test-Driven Development by Example](https://g.co/kgs/yCzDLF) Kent Beck refactors some tests to a point and asserts:

> The test speaks to us more clearly, as if it were an assertion of truth, **not a sequence of operations**

\(emphasis in the quote is mine\)

Now our tests - rather, the list of test cases - make assertions of truth about shapes and their areas.

## Make sure your test output is helpful

Remember earlier when we were implementing `Triangle` and we had the failing test? It printed `shapes_test.go:31: got 0.00 want 36.00`.

We knew this was in relation to `Triangle` because we were just working with it.
But what if a bug slipped in to the system in one of 20 cases in the table?
How would a developer know which case failed?
This is not a great experience for the developer, they will have to manually look through the cases to find out which case actually failed.

We can change our error message into `%#v got %g want %g`. The `%#v` format string will print out our struct with the values in its field, so the developer can see at a glance the properties that are being tested.

To increase the readability of our test cases further, we can rename the `want` field into something more descriptive like `hasArea`.

One final tip with table driven tests is to use `t.Run` and to name the test cases.

By wrapping each case in a `t.Run` you will have clearer test output on failures as it will print the name of the case

```text
--- FAIL: TestArea (0.00s)
    --- FAIL: TestArea/Rectangle (0.00s)
        shapes_test.go:33: main.Rectangle{Width:12, Height:6} got 72.00 want 72.10
```

And you can run specific tests within your table with `go test -run TestArea/Rectangle`.

Here is our final test code which captures this

```go
func TestArea(t *testing.T) {

	areaTests := []struct {
		name    string
		shape   Shape
		hasArea float64
	}{
		{name: "Rectangle", shape: Rectangle{Width: 12, Height: 6}, hasArea: 72.0},
		{name: "Circle", shape: Circle{Radius: 10}, hasArea: 314.1592653589793},
		{name: "Triangle", shape: Triangle{Base: 12, Height: 6}, hasArea: 36.0},
	}

	for _, tt := range areaTests {
		// using tt.name from the case to use it as the `t.Run` test name
		t.Run(tt.name, func(t *testing.T) {
			got := tt.shape.Area()
			if got != tt.hasArea {
				t.Errorf("%#v got %g want %g", tt.shape, got, tt.hasArea)
			}
		})

	}

}
```

## Wrapping up

This was more TDD practice, iterating over our solutions to basic mathematic problems and learning new language features motivated by our tests.

* Declaring structs to create your own data types which lets you bundle related data together and make the intent of your code clearer
* Declaring interfaces so you can define functions that can be used by different types \([parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism)\)
* Adding methods so you can add functionality to your data types and so you can implement interfaces
* Table driven tests to make your assertions clearer and your test suites easier to extend & maintain

This was an important chapter because we are now starting to define our own types. In statically typed languages like Go, being able to design your own types is essential for building software that is easy to understand, to piece together and to test.

Interfaces are a great tool for hiding complexity away from other parts of the system. In our case our test helper _code_ did not need to know the exact shape it was asserting on, only how to "ask" for its area.

As you become more familiar with Go you will start to see the real strength of interfaces and the standard library. You'll learn about interfaces defined in the standard library that are used _everywhere_ and by implementing them against your own types, you can very quickly re-use a lot of great functionality.