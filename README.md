# flatMap과 flatten: 중첩된 컬렉션 안의 원소 처리
Book으로 표현한 책에 대한 정보를 저장하는 도서관이 있다고 가정해보자.
<code>
<pre>
class Book(val title: String, val authors: List<String>)
</code>
</pre>
각 책은 저마다 저자가 한 명 또는 여러명 있다고 할 때 도서관에 있는 책의 저자를 모두 모은 집합을 다음과 같이 가져올 수 있다.
<code>
<pre>
book.flatMap{ it.authors }.toSet()
</code>
</pre>
flatMap 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용하고 람다를 적용한 결과 얻어지는 여러 리스트를 한 리스트로 재구성한다.

<code>
<pre>
val strings = listOf("abc", "def")
println(strings.flatMap{ it.toList() })
[a, b, c, d, e, f]
</code>
</pre>

toList 함수를 문자열에 적용하면 그 문자열에 속한 모든 문자로 이뤄진 리스트가 만들어진다. map과 toList를 함께 사용하면 리스트를 이용한 새로운 리스트가 생기는 것이다.   
flatMap 함수는 다음 단계로 리스트의 리스트에 들어있던 모든 원소로 이뤄진 단일 리스트를 반환한다.
<code>
<pre>
val books = listOf(Book("Thursday Next", listOf("Jasper Fforde")),
                   Book("Mort", listOf("JMJ, MC")),
                   Book("Hello", listOf("MC")))
println(books.flatMap {it.authour}.toSet())
[Jasper Fforde, JMJ, MC]
</code>
</pre>
book.authors 프로퍼티는 작가를 모아둔 컬렉션이다. flatMap 함수는 모든 책의 작가를 평평한(문자열로 이뤄진) 리스트 하나로 모은다.   
toSet은 flatMap의 결과 리스트에서 중복을 제거하고 집합으로 만든다. 따라서 최종적으로 MC는 한번만 볼 수 있다.   
하지만 특별히 변환해야 할 내용이 없다면 리스트의 리스트를 펼치기만 하면 된다. 이럴 경우 listOfLists.flatten()처럼 flatten 함수를 사용해볼 수 있다.

# 지연 계산(lazy) 컬렉션 연산
앞에서 본 몇 가지의 컬렉션 한수들은 결과 컬렉션을 즉시 생성한다. 이는 컬렉션 함수를 연쇄하면 매 단계마다 계산 중간 결과를 새로운 컬렉션을 만들어 담아둔다는 얘기이다.   
시퀀스를 사용하면 중간 임시 칼렉션을 사용하지 않고도 컬렉션 연산을 할 수 있다.
<code>
<pre>
people.map(Person::name).filter{ it.startsWith("A")}
</code>
</pre>
코틀린 표준 라이브러리 참조 문서에는 filter와 map이 리스트를 반환한다고 써 있다. 이는 연쇄 호출이 리스트를 2개 만들어서 하나는 filter의 결과를 담고, 다른 하나는 map의 결과를 담는다고 한다.
원본 리스트에 원소가 적으면 문제가 없지만, 원소가 수백만 개가 되어버리면 효율이 떨어진다.   
이를 더 효율적으로 사용하기 위해서는 각 연산이 컬렉션을 직접 사용하는 대신 시퀀스를 사용하게 하면 좋다.
<code>
<pre>
people.asSequence()
.map(Person::name)
.filter{it.startsWith("A")}
.toList
</code>
</pre>
이렇게 사용하면 중간 결과를 저장하는 컬렉션이 생기지 않으므로 성능이 눈에 띄게 좋아진다.
코틀린 지연 계산 시퀀스는 Sequence 인터페이스에서 시작하고 단지 한 번에 하나씩 열거될 수 있는 원소의 시퀀스를 표현한다. Sequence 안에는 iterator라는 단 하나의 메소드가 있는데 이 메소드를 통해 시퀀스로부터 원소 값을 얻을 수 있다.   
asSequence 확장 함수를 호출하면 어떤 컬렉션이든 시퀀스로 바꿀 수 있다. 시퀀스를 리스트로 만들 때는 toList를 사용하면 된다.   

큰 컬렉션에 대해서 연산을 연쇄해야할 때는 시퀀스를 사용하는 것을 규칙으로 삼는게 더 좋다.
