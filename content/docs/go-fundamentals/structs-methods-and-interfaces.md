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

[الاختبارات المجدولة](https://go.dev/wiki/TableDrivenTests) مفيدة عندما تريد بناء قائمة من حالات الاختبار التي يمكن اختبارها بنفس الطريقة.

```go {filename="shapes_test.go"}
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

الشئ الوحيد الجديد هنا هو اننا قمنا بانشاء "بنية مجهولة"، `areaTests`. قمنا بتعريف قائمة من البنى عن طريق استخدام `[]struct` مع حقلين، `shape` و `want`. ثم قمنا بملئ القائمة بالحالات.

نقوم بالتكرار عليهم بنفس الطريقة التي نقوم بها مع اي مصفوفة اخرى، ويمكننا الحصول على البيانات من خلال حقول البنى لتشغيل الاختبارات.

بأمكانك رؤية كيف يمكن لاي مطور ان يقوم بأضافة شكل جديد، كل ما يحتاجة هو كتابة داله `Area` على الشكل الجديد ومن ثم اضافته الى حالات الاختبار. بالاضافة الى ذلك، اذا تم العثور على خطأ في `Area` فمن السهل جدا اضافة حالة اختبار جديدة لاختبارها قبل اصلاحها.

يمكن للاختبارات المجدول ان تكون اداة رائعة في مجموعة ادواتك. تكون مناسبةً جداً عندما تريد اختبار مجموعة من الدوال التي تتبع او تتوافق مع واجهة معينة.

Let's demonstrate all this by adding another shape and testing it; a triangle.

لنستعرض كل ذلك من خلال اضافة شكل جديد (مثلث) واختبارة

## نكتب الاختبار اولا

اضافة اختبار جديد لشكلنا الجديد سهل جدا. فقط قم بأضافة `{Triangle{12, 6}, 36.0},` الى قائمتنا.


```go {filename="shapes_test.go"}
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

## شغل الاختبار

تذكر دائما ان تحاول تشغيل الاختبار ودع المترجم يرشدك نحو الحل.

## لنكتب ما يكفي للتحقق من النتائج الفاشلة

`./shapes_test.go:25:4: undefined: Triangle`

لم نقم بتعريف `Triangle` بعد، لذا قم بتعريفه

```go {filename="shapes.go"}
type Triangle struct {
	Base   float64
	Height float64
}
```

حاول مجدداً

```text {filename="terminal"}
./shapes_test.go:25:8: cannot use Triangle literal (type Triangle) as type Shape in field value:
    Triangle does not implement Shape (missing Area method)
```

It's telling us we cannot use a `Triangle` as a shape because it does not have an `Area()` method, so add an empty implementation to get the test working

المترجم يخبرنا انه لا يمكن استخدام `Triangle` كشكل لانه لا يحتوي على دالة `Area()`، لذا قم بأضافة تابع فارغ لتجعل الاختبار يعمل

```go {filename="shapes.go"}
func (t Triangle) Area() float64 {
	return 0
}
```

اخيرا الكود يترجم ونحصل على خطأ

`shapes_test.go:31: got 0.00 want 36.00`

## الان اكتب ما يكفي لنجاح الاختبار

```go {filename="shapes.go"}
func (t Triangle) Area() float64 {
	return (t.Base * t.Height) * 0.5
}
```

وستنجح الاختبارات الان

## اعادة الكتابة

مجدداً ما قمنا به جيد ولكن الاختبارات تحتاج الى بعض التحسين.

عندما تنظر لهذا

```go
{Rectangle{12, 6}, 72.0},
{Circle{10}, 314.1592653589793},
{Triangle{12, 6}, 36.0},
```

لن تعرف ماذا تعني الارقام في الحالات الاختبارية بشكل سهل، ويجب ان تهدف الى جعل الاختبارات سهلة الفهم.

حتى الان كل ما عرضنا عليك هو كيفية انشاء نسخ من البنى `MyStruct{val1, val2}` ولكن يمكنك اختياريا تسمية الحقول عند الانشاء.

Let's see what it looks like

لنرى كيف يبدو ذلك


``` go
{shape: Rectangle{Width: 12, Height: 6}, want: 72.0},
{shape: Circle{Radius: 10}, want: 314.1592653589793},
{shape: Triangle{Base: 12, Height: 6}, want: 36.0},
```

في كتاب [Test-Driven Development by Example](https://g.co/kgs/yCzDLF) يقوم Kent Beck بتحسين بعض الاختبارات ويؤكد:

> الاختبار يتحدث الينا بشكل اوضح، كما لو كان تأكيدا على الحقيقة، **وليس تسلسل عمليات**

## تأكد من ان مخرجات الاختبارات مفيدة

تتذكر مسبقا عندما كنا نقوم بكتابة `Triangle` وكان هناك اختبار فاشل؟ قام بطباعة

`shapes_test.go:31: got 0.00 want 36.00`.

كنا نعلم انه مرتبط بالمثلث لاننا كنا نعمل عليه. لكن ماذا لو ان خطأً دخل الى النظام في احدى الحالات العشرين على سبيل المثال في الجدول؟

كيف يمكن للمطور ان يعرف اي حالة فشلت؟

هذه ليست بالتجربة الجيدة للمطور، سيضطر الى البحث يدويا عن الحالة التي فشلت.

بالامكان تغيير رسالة الخطأ الى `%#v got %g want %g`. النائب `%#v` سيطبع البنية مع القيم في حقولها، لذا يمكن للمطور ان يرى بلمح البصر الخصائص التي يتم اختبارها.

وحتى نحسن من مقروئية حالات الاختبار اكثر، يمكننا تغيير حقل `want` الى شيء اكثر وصفا مثل `hasArea`.

معلومة اخيرة عن الاختبارات المجدولة هي استخدام `t.Run` وتسمية حالات الاختبار.

من خلال تضمين كل حالة في `t.Run` ستحصل على مخرجات اختبار اوضح عند الفشل حيث سيطبع اسم الحالة

```text {filename="terminal"}
--- FAIL: TestArea (0.00s)
    --- FAIL: TestArea/Rectangle (0.00s)
        shapes_test.go:33: main.Rectangle{Width:12, Height:6} got 72.00 want 72.10
```

ويمكنك ايضا تشغيل اختبارات معينة ضمن جدولك باستخدام 

```text {filename="terminal"}
go test -run TestArea/Rectangle
```


هنا الكود النهائي للاختبارات والذي يحتوي على هذه التحسينات

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

## ختامًا


كان هذا تمريناً على TDD و تكرار حلولنا لمشاكل رياضية بسيطة وتعلم ميزات جديدة في اللغة مستندين الى اختباراتنا.

* اعلان البنى لانشاء انواع بيانات خاصة بك والتي تسمح لك بتجميع البيانات ذات الصلة معًا وجعل مقصود الكود الخاص بك اكثر وضوحًا
* اعلان الواجهات لتعريف الدوال التي يمكن استخدامها من قبل انواع مختلفة \([تعددية الشكل polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism)\)
* اضافة توابع لكي تضيف وظائف لانواع البيانات الخاصة بك ولكي تستطيع تنفيذ الواجهات
* الاختبارات المجدولة لتجميع حالات الاختبار المتشابهة معًا وتجعل الاختبارات اكثر وضوحًا واسهل في الصيانة والتوسع

هذا الفصل كان مهمًا لأننا بدأنا الآن في تعريف أنواعنا الخاصة. في لغات البرمجة ذات النوع الثابت مثل Go، القدرة على تصميم أنواعك الخاصة أمر أساسي لبناء برمجيات سهلة الفهم وسهلة التجميع والاختبار.

الواجهات هي أداة رائعة لإخفاء التعقيدات بعيدًا عن أجزاء أخرى من النظام وهذا هو المقصود من فض الاقتران او الفصل. في حالتنا، لم يحتاج كود مساعد الاختبارات إلى معرفة الشكل الدقيق الذي كان يؤكد عليه، فقط كيفية "طلب" مساحته.

كلما اصبحت تعرف اكثر على Go ستبدأ في رؤية قوة الواجهات والمكتبة القياسية. ستتعلم عن الواجهات المعرفة مسبقا في المكتبة القياسية والتي تستخدم في كل مكان ومن خلال تنفيذها ضد انواعك الخاصة، يمكنك اعادة استخدام الكثير من الوظائف الرائعة بسرعة كبيرة.