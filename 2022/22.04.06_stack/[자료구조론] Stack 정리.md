# [자료구조론] Stack 정리



웹 개발자 일을 하면서 `Stack` 을 사용하는 경우를 겪어보지 못하였습니다. 대학교 2학년때 배웠던 데이터구조론에서 자료구조에서 배웠던 기억과 코딩 테스트를 준비하면서 배웠던 부분들을 제외하고는 실제 개발 환경에서는 사용해보았던 적이 없습니다. 

`Stack` 의 자료구조는 마치 지금 내 책상위에 올려져 있는 책들과 비슷해보입니다.

![쌓여져있는 책들](/Users/lsh/Desktop/00_StudyWork/02_MARKDOWN_TISTORY/22.04.06_stack_정리/images/books.jpeg)

(물론 현재 제 책상은 아닙니다만...)



모든 소스 코드는 Github를 통해서 확인 가능합니다. - [소스바로가기](https://github.com/codeleesh/study-code/tree/main/java-basic-daily/src/main/java/com/lovethefeel/stack)



## Stack이란?

이미지에서 보시는것처럼 `Stack` 은 쌓아올린다는 것을 의미합니다.



## Stack 특징

`Stack` 은 한 쪽 끝에서만 자료를 넣거나 뺄 수 있는 선형 구조(`LIFO`) 입니다.

자료를 넣거나 뺄 수 있는 것은 `top` 이라고 하며, 이 `top` 에는 가장 최근에 들어온 데이터를 가리키고 있습니다. 새로 삽입되는 데이터는 `top` 이 가리키는 데이터의 위에 쌓이게 됩니다. `Stack` 에서 자료를 삭제할 때도 `top` 을 통해서 만 가능합니다.

`Stack` 에서 데이터를 삽입하는 연산을 `push` , 데이터를 삭제하는 연산은 `pop` 라고 한다.



키워드를 정리해보면 다음과 같습니다.

- top으로만 정한 곳을 통해서만 접근
- 삽입하는 연산과 삭제하는 연산
- 가장 마지막에 삽입된 자료가 가장 먼저 삭제(후입선출)



## Stack 활용

실제 프로그램에서 다음과 같이 활용 가능할것으로 보여집니다.

- 웹 브라우저 방문기록 - 뒤로가기
- 역순 문자열 만들기
- 후위 표기법 계산



## Stack 메소드

`Stack` 은 `Vector` 를 확장한 클래스입니다.

`Vector` 의 주요 기능은 넘어가고 `Stack` 에서 사용하는 주요 메소드는 다음과 같습니다.



### push

```java
    public E push(E item) {
        addElement(item);

        return item;
    }
```

`Stack` 의 데이터를 삽입하는 메소드이며, `Vector` 의 `addElement(item)` 를 이용합니다.

반환값으로는 해당 데이터의 객체를 반환합니다.



### pop

```java
    public synchronized E pop() {
        E       obj;
        int     len = size();

        obj = peek();
        removeElementAt(len - 1);

        return obj;
    }
```

`Stack` 의 데이터를 삭제하는 메소드이며,  `Vector` 의 `removeElementAt` 를 이용합니다.

반환값으로는 해당 삭제된 데이터의 객체를 반환합니다.

`Stack` 안에 `pop` 할 데이터가 더이상 없을 때 `pop` 을 수행하면 `EmptyStackException` 를 던집니다.



### peek

```java
    public synchronized E peek() {
        int     len = size();

        if (len == 0)
            throw new EmptyStackException();
        return elementAt(len - 1);
    }
```

`Stack` 의 `top` 이 가리키는 데이터를 반환합니다.  `Vector` 의 `elementAt` 를 이용합니다.

`pop` 과 다른 점은 삭제하지 않고 데이터만 반환한다는 점입니다.



### empty

```java
    public boolean empty() {
        return size() == 0;
    }
```

`Stack` 의 `size` 가 0일 경우 `true` 를 반환하며, 그렇지 않을 경우 `false` 를 반환합니다.



### search

```java
    public synchronized int search(Object o) {
        int i = lastIndexOf(o);

        if (i >= 0) {
            return size() - i;
        }
        return -1;
    }
```

`Stack` 내 데이터를 검색할 수 있으며, 데이터 객체를 인자로 받아서 `index` 를 반환합니다.

`Vector` 의 `lastIndexOf(item)` 를 이용합니다.



## 테스트 진행

책상위에 있는 책을 보고 다음과 같은 클래스들을 만들었습니다.

책의 정보를 담고 있는 `Book` 객체는 다음과 같습니다.

```java
public class Book {
    private String isbn_id;
    private String name;

    private Book(final String isbn_id, final String name) {
        validate(isbn_id, name);
        this.isbn_id = isbn_id;
        this.name = name;
    }

    private void validate(final String isbn_id, final String name) {
        if (isbn_id == null || isbn_id == "") {
            throw new IllegalArgumentException("isbn_id가 없습니다.");
        }
        if (name == null || name == "") {
            throw new IllegalArgumentException("name이 없습니다.");
        }
    }

    public static Book of(final String isbn_id, final String name) {
        return new Book(isbn_id, name);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Book book = (Book) o;
        return Objects.equals(isbn_id, book.isbn_id) && Objects.equals(name, book.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(isbn_id, name);
    }

    @Override
    public String toString() {
        return "Book{" +
                "isbn_id='" + isbn_id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}
```

책상위에 있는 책들을 나타내기 위해서 `BookShelf` 객체를 만들었습니다.

```java
public class BookShelf {

    private static final int NON_EXIST_ITEM = -1;
  
    private Stack<Book> bookShelfs;

    private BookShelf(final Stack<Book> bookShelfs) {
        this.bookShelfs = bookShelfs;
    }

    public static BookShelf init() {
        return new BookShelf(new Stack<>());
    }

    public static BookShelf from(final Stack<Book> bookShelfs) {
        return new BookShelf(bookShelfs);
    }

    public Book recentItem() {
        return this.bookShelfs.peek();
    }

    public void push(final Book book) {
        this.bookShelfs.push(book);
    }

    public Book pop() {
        return this.bookShelfs.pop();
    }

    public boolean search(final Book book) {
        if (this.bookShelfs.search(book) != NON_EXIST_ITEM) {
            return true;
        }
        return false;
    }

    public boolean isEmpty() {
        return this.bookShelfs.empty();
    }

    public List<Book> list() {
        return this.bookShelfs.stream()
                .collect(Collectors.toList());
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        BookShelf bookShelf = (BookShelf) o;
        return Objects.equals(bookShelfs, bookShelf.bookShelfs);
    }

    @Override
    public int hashCode() {
        return Objects.hash(bookShelfs);
    }

    @Override
    public String toString() {
        return "BookShelf{" +
                "bookShelfs=" + bookShelfs +
                '}';
    }
}
```



위 도메인을 통해서 책을 쌓는 방법을 `Stack` 을 활용해서 실습을 해보도록 하겠습니다.



### 도서 추가

새로운 도서를 추가하는 테스트를 진행해보도록 하겠습니다.

```java
    @Test
    void 새로운_도서_추가() {
        // given
        Book 클린아키텍처 = Book.of("4", "클린아키텍처");
        BookShelf bookShelf = BookShelf.init();

        // when
        bookShelf.push(클린아키텍처);

        // then
        assertEquals(bookShelf.recentItem(), 클린아키텍처);
    }
```



### 도서 삭제

책상위 3개의 책들이 쌓여있는 상태에서 맨위에 있는 책을 제거해보도록 하겠습니다.

```java
    private Book 자바의정석 = Book.of("1", "자바의정석");
    private Book 클린코드 = Book.of("2", "클린코드");
    private Book 이펙티브자바 = Book.of("3", "이펙티브자바");
    private BookShelf bookShelf;

    @BeforeEach
    void init() {
        bookShelf = BookShelf.init();
        bookShelf.push(자바의정석);
        bookShelf.push(클린코드);
        bookShelf.push(이펙티브자바);
    }

		@Test
    void 도서_삭제() {
        // given, when
        Book popItem = bookShelf.pop();

        // then
        assertEquals(popItem, 이펙티브자바);
    }
```



### 가장 맨위에 있는 도서 확인

현재 가장 맨위에 있는 도서를 확인해보는 테스트를 진행해보겠습니다.

```java
    @Test
    void 맨위에_있는_도서() {
        // given, when
        Book book = bookShelf.recentItem();

        // then
        assertEquals(book, 이펙티브자바);
    }
```



### 도서 검색

쌓아져 있는 곳에서 찾고자 하는 도서가 있는 확인해볼 수 있습니다.

```java
    @Test
    void 도서_검색() {
        // given, when
        boolean isSearch = bookShelf.search(자바의정석);

        // then
        assertEquals(isSearch, true);
    }
```



### 삭제할 도서가 없을 경우 예외

더이상 삭제할 도서가 없을 경우 `pop` 을 하게 되면 `EmptyStackException` 발생하게 됩니다.

```java
    @Test
    void 더이상_제거할_도서가_없을_경우() {
        // given, when
        BookShelf empty = BookShelf.init();

        // then
        EmptyStackException exception = assertThrows(EmptyStackException.class, () -> empty.pop());
    }
```

