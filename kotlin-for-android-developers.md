# [Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers)

- [Kotlin-for-Android-Developers in github](https://github.com/antoniolg/Kotlin-for-Android-Developers)
- [RecyclerView in Android](http://antonioleiva.com/recyclerview/)
- [Wether json url](http://http://api.openweathermap.org/data/2.5/forecast/daily?APPID=15646a06818f61f7b8d7823ca833e1ce&q=94043&mode=json&units=metric&cnt=7)

- [kotlin cheat sheet](https://gist.github.com/dodyg/5823184)
[TOC]

## 1. Introduction
interoperability between both languages is excellent.

### 1.1 Expresiveness

avoid bolierplate - data class

```
1 data class Artist(
2     var id: Long,
3     var name: String,
4     var url: String,
5     var mbid: String)
```

### 1.2 Null Safety

```
 1 // Artist가 null일 수 없어서 컴파일이 안됨
 2 var notNullArtist: Artist = null
 3
 4 // ? 연산을 이용하면 null을 할당할 수 있다.
 5 var artist: Artist? = null
 6
 7 // artist가 null일 수 있으므로 적절하게 다뤄야 한다. 이 코드는 컴파일되지 않는다.
 8 artist.print()
 9
10 // ?를 이용해서 artist가 null이 아닌 경우만 print가 동작
11 artist?.print()
12
13 // Smart cast. null 조사를 사전에 한다면 safe call operator('?')를 사용할 필요가 없다.
15 if (artist != null) {
16     artist.print()
17 }
18
19 // artist가 null 아닌 것이 확실한 경우만 아래와 같이 사용. 그렇지 않다면 Exception이 발생
20 artist!!.print()
21
22 // 객체가 null인 경우 대안 값을 사용하기 위해 Elvis operator 사용
23 val name = artist?.name ?: "empty"
```

### 1.3 Extension uu1GjFunctions
We can add new functions to any class(even existing 3rd party class)

```
fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
	Toast.makeText(getActivity(), message, duration).show()
}

fragment.toast("Hello world!")
```

### 1.4 Functional support (Lambdas)

```
view.setOnClickListener { toast("Hello world!") }
```

## 2. Getting Ready

## 3. Creating a new project

convert java to kotlin(MainActivity.java to MainActivity.kt)
- Open the file and select Code -> Convert Java File to Kotlin File.

get access to that TextView directly
- message.text = "Hello Kotlin!" instead of findById ...

## 4. Classes and functions

[try.kotlinlang.org](http://try.kotlinlang.org)

### 4.1 How to declare a class

```
class Person(name: String, surname: String)
```

### 4.2 Class inheritance

Root class: Any(Object in java)

Classes are closed by default (final), so we can only extend a class if it’s explicitly declared as open or abstract:

```
open class Animal(name: String)

class Person(name: String, surname: String) : Animal(name)
```

### 4.4 Constructor and functions parameters

#### default value

```
fun toast(message: String, length: Int = Toast.LENGTH_SHORT) {
	Toast.makeText(this, message, length).show()
}
```

#### String templates

can use template expressions directly in your string

`Your name is ${user.name}"`

## 5 Writing your first class

### 5.1 Creating the layout

```
val forecastList = findViewById(R.id.forecast_list) as RecyclerView
forecastList.layoutManager = LinearLayoutManager(this)
```

- `as`를 이용해서 타입 캐스팅
- 객체 생성시 new 생략
- Default visibility is public

#### List creation

listOf, setOf, mutableListOf or hashSetOf ...

## 6 Variables and properties

everything is an object.

### 6.1 Basic types

no automatic conversions among numeric types. 

```
vali:Int=7
val d: Double = i.toDouble()
```

```
1 // Java
2 int bitwiseOr = FLAG1 | FLAG2;
3 int bitwiseAnd = FLAG1 & FLAG2;

1 // Kotlin
2 val bitwiseOr = FLAG1 or FLAG2
3 val bitwiseAnd = FLAG1 and FLAG2
```

```
class Person {
	var name: String = ""
    	get() = field.toUpperCase()
	set(value) {
    	field = "Name: $value"
    }
}
```

## 7 Anko and Extension Functions

[Anko](https://github.com/Kotlin/anko) is a powerful library developed by JetBrains. Its main purpose is the generation of UI layouts by using code instead of XML. 

```
verticalLayout {
    val name = editText()
    button("Say Hello") {
        onClick { toast("Hello, ${name.text}!") }
    }
}
```

![](https://github.com/Kotlin/anko/raw/master/doc/helloworld.png)

Anko includes a lot of extremely helpful functions. Anko uses extension functions to add new features to Android framework.

### 7.3 Extension functions
An extension function is a function that adds a new behaviour to a class, even if we don’t have access to the source code of that class. 

```
fun Context.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
	Toast.makeText(this, message, duration).show()
}

toast("Hello world!")
toast("Hello world!", Toast.LENGTH_LONG)
```

Properties example
```
public var TextView.text: CharSequence
    get() = getText()
    set(v) = setText(v)
```

## 8 Retrieving data from API

### 8.1 Performing a request

```
class Request(val url: String) {
	fun run() {
    	val forecastJsonStr = URL(url).readText()
    }
}
```

### 8.2 Performing the request out of the main thread
HTTP requests are not allowed to be done in the main thread, it will throw an exception.
The common solution in Android is to use an AsyncTask. But these classes are ugly and difficult to implement without any side effects

Anko provides a very easy DSL to deal with asynchrony. provides an async function that will execute its code in another thread, with the option to return to the main thread by calling uiThread.

```
async() {
	Request(url).run()
    uiThread {
    	longToast("Request Performed")
    }
}

val executor = Executors.newScheduledThreadPool(4)
async(executor) {
	// some task
}
```

async returns a java Future, also can use asyncResult

## 9 Data Classes

### 9.1 Extra functions
getter, setter 외에 

- equals()
- hashCode()
- copy(): you can copy an object, modifying the properties you need.
- A set of numbered functions that are useful to map an object into variables.

### 9.2 Copying a data class

```
1 val f1 = Forecast(Date(), 27.5f, "Shiny day")
2 val f2 = f1.copy(temperature = 30f)
```

### 9.3 Mapping an object into variables

```
1 val f1 = Forecast(Date(), 27.5f, "Shiny day")
2 val (date, temperature, details) = f1

1 for ((key, value) in map) {
2 	Log.d("map", "key:$key, value:$value")
3 }
```

## 10 Parsing data

### 10.1 Converting json to data classes

A good practice explained in most software architectures is to use different models for the different layers in our app to decouple them from each other. 

**Companion objects**
Kotlin allows to declare objects to define static behaviours. In Kotlin, we can’t create static properties or functions, but we need to rely on objects. However, these objects make some well known patterns such as Singleton very easy to implement.
If we need some static properties, constants or functions in a class, we can use a companion object. This object will be shared among all instances of the class, the same as a static field or method would do in Java.

```
class ForecastByZipCodeRequest(val zipCode: Long, val gson: Gson = Gson()) {
    companion object {
        private val APP_ID = "15646a06818f61f7b8d7823ca833e1ce"
        private val URL = "http://api.openweathermap.org/data/2.5/forecast/daily?mode=json&units=metric&cnt=7"
        private val COMPLETE_URL = "$URL&APPID=$APP_ID&q="
    }

    fun execute(): ForecastResult {
        val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
        return gson.fromJson(forecastJsonStr, ForecastResult::class.java)
    }
}
```

As we are using two classes with the same name, we can give a specific name to one of them so that we don’t need to write the complete package:
`import com.antonioleiva.weatherapp.domain.model.Forecast as ModelForecast`

`return list.map { convertForecastItemToDomain(it) }`
In a single line, we can loop over the collection and return a new list with the converted items.

**with function**
It basically receives an object and an extension function as parameters, and makes the object execute the function. This means that all the code we define inside the brackets acts as an extension function of the object we specify in the first parameter, and we can use all its public functions and properties, as well as this. Really helpful to simplify code when we do several operations over the same object.

```
with(weekForecast.dailyForecast[position]) {
	holder.textView.text = "$date - $description - $high/$low"
}
```

## 11 Operator overloading

```
data class ForecastList(val city: String, val country: String,
                        val dailyForecast: List<Forecast>) {
	operator fun get(position: Int): Forecast = dailyForecast[position]
    fun size(): Int = dailyForecast.size
}

```

위와 같이 get([]) operator, size를 정의함으로써 아래와 같이 with 함수로 간결한 사용이 가능

```
override fun onBindViewHolder(holder: ViewHolder, position: Int) {
    with(weekForecast[position]) {
    	holder.textView.text = "$date - $description - $high/$low"
    }
```

### 11.3 Operators in extension functions

```
operator fun ViewGroup.get(position: Int): View
	= getChildAt(position)
```
이렇게 extension을 정의하면

```
1 val container: ViewGroup = find(R.id.container)
2 val view = container[2]
```
2라인과 같은 사용이 가능

## 12 Making the forecast list clickable

## Conclusion
- scala를 안다면 쉽게 배울 수 있을 듯. scala보다 덜 복잡한 듯
- android에 최적화된 기능(anko)
- data class: scala case class
- fun이 아니라 func였으면...
- extention이 독인지... 약인지...


----
## Summary I
주말에 본 kotlin책...[Kotlin for Android Developers](https://leanpub.com/kotlin-for-android-developers)
1/3 정도를 봤네요

기본 문법은 http://try.kotlinlang.org 에서 학습 가능

[Kotlin-for-Android-Developers in github](https://github.com/antoniolg/Kotlin-for-Android-Developers)

- scala를 조금 본 분이라면 쉽게 이해할 것 같아요.
- 느낌이 scala보다 좀 쉬운 scala 같은 언어
- android에 특화된 편이 기능을 제공하여 안드로이드 코드도 간결하게 해 주는 언어
- data class(scala의 case class)가 있어서 표현력이 좋고
- null safety: 변수 선언시 null이 될 수 있음을 명시적으로 선언해야 함. 또 null이 될 수 있는 객체에 대한 호출과 아닌 호출을 구분하는 문법 존재
- extension functions: duck typing과 유사. 3rd party 클래스를 포함한 기존 클래스에 기능을 추가 가능
- lamda 지원
- java -> kotlin을 지원(MainActivity.java to MainActivity.kt))
- ui 위젯에 id로 직접 접근 가능
  - message.text = “Hello Kotlin!” // TextView의 id가 message인 경우, instead of findById …
- 기본적으로 클래스 선언시 상속에 닫혀있음. 'open' 키워드를 사용해서 명시적으로 선언해야 상속 가능
- String Template: ex. Your name is ${user.name}"
- as를 이용한 타입 캐스팅: ex. val forecastList = findViewById(R.id.forecast_list) as RecyclerView
- List creation Helper: listOf, setOf, mutableListOf or hashSetOf …
- Anko and Extension Functions
  - Anko(https://github.com/Kotlin/anko) is a powerful library developed by JetBrains. Its main purpose is the generation of UI layouts by using code instead of XML.
  ```
    verticalLayout {
        val name = editText()
        button("Say Hello") {
            onClick { toast("Hello, ${name.text}!") }
        }
    }
  ```
- 예제는 Robert C Martin의 Clean Architecture에 기반하고 있는 느낌.

## References
[Articles about Kotlin](http://antonioleiva.com/kotlin-android-developers/)

## Tips

- anko로 widget을 접근할 경우. 위젯이 없어져도 컴파일 오류는 발생하지 않는다.