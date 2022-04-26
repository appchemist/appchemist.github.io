---
layout: post
title: 'Functors: Function Objects in C++'
date: '2016-06-20T11:18:23+09:00'
tags:
- c++
- function pointer
- functor
- template
- virtual function
tumblr_url: https://appchemist.tumblr.com/post/146209418541/functors-function-objects-c
---
C와 C++ 모두 function pointer를 지원한다. function pointer는 특정 명령을 수행하는 함수를 전달할 수 있는 방법 중 하나이다.

하지만, function pointer는 매우 제한적인데, function pointer는 컴파일 시점에 해당 함수가 정의되어 있어야 한다. 그럼 왜 이것 때문에 function pointer가 제한적이라고 하는 것일까?

수신함을 보는 메일 프로그램을 작성하는 예를 들어보자. 사용자가 다양한 필드(to, from, date 등)를 이용해서 수신함을 정렬을 할 수 있도록 하고 싶다. 메시지를 비교하는 정렬 루틴을 메시지 비교를 하는 function pointer를 사용하고자 할것이다. 메일의 필드만 달리 비교하는 함수를 만들수도 있지만, 이는 하드 코딩된 필드만 정렬할 수 있을 것이다. 결국 정렬 루틴에서 사용할 다양한 함수들을 만들게 된다.

비교 함수에 어떤 필드를 비교할지 알려주는 제 3의 인자를 전달 하고 싶을 것이다. 그러나 이렇게 하기 위해서는 제 3의 인자를 아는 정렬 루틴을 만들어야 한다. 그러면 제 3의 인자를 comparator에 전달 할수 없으므로 STL과 같은 generic routine을 사용할 수 없다. 대신, 함수 내부에서 어떤 필드를 정렬할지 아는 함수가 필요 하다.

위와 같은 문제를 해결하기 위해서 C++에서 function object(functors)를 사용할 수 있다. functor는 function과 function pointer와 같이 다룰 수 있는 객체이다. 아래와 같은 코드를 작성할 수 있다.


```cpp
myFunctorClass functor;
 functor(1, 2, 3);
```

C++은 operator()(function call)를 오버로드할 수 있으므로 위의 코드는 작동한다. function call operator는 여러개의 인자와 다양한 타입을 받을 수 있고 어떤 것이든 결과 값으로 전달 할 수 있다. 이것은 operator를 오버로드할 수 있으므로써 생기는 유연성이다. 이후 해당 글에서 객체의 operator()를 호출할 때 객체를 호출했다고 이야기 하겠다.

오버로딩한 operator()가 좋지만, functor의 진면목은 functor의 life cycle이 function 보다 더 유연하다는 점이다. functor의 생성자에 추후에 operator()에서 사용할 정보들 삽입할 수 있다.

다음 예제는 integer 인자를 받는 생성자를 가진 functor class를 생성한다. 해당 클래스의 객체가 호출되면 저장된 값과 인자 값을 더한 결과를 반환한다.


```cpp
class myFunctorClass
{
public:
    myFunctorClass(int x) : _x(x) {}
    int operator() (int y) { return _x + y; }

private:
    int _x;
};

TEST(myFunctorClassTest, TestSimpleAdding) {
    myFunctorClass addFive(5);
    EXPECT_EQ(11, addFive(6));
}
```

결론적으로 생성자에 인자를 넘겨주는면, 이후 해당 인수를 사용하는 함수와 같은 동작을 한다.
### Sorting Mail
이제 정렬 functor를 구현하기 위해서 코드를 어떻게 작성하면 될지 생각해보자.

두개의 인자를 받는 operator()가 필요하고 정렬하고자 하는 필드 정보를 저장하는 생성자가 필요하다.


```cpp
class Message
{
public:
    std::string getHeader(const std::string& header_name) const;
};

class MessageSorter
{
public:
    MessageSorter (const std::string& field) : _field(field) {}
    bool operator() (const Message& lhs, const Message& rhs)
    {
        return lhs.getHeader(_field) < rhs.getHeader(_field);
    }
private:
    std::string _field;
};
```

이제 메시지 vector가 있다면, 우리는 STL sort 함수를 사용해서 정렬을 할 수 있다.


```cpp
TEST(MessageSorterTest, TestMessageSorter)
{
    std::vector<Message> messages;
    // read in messages
    MessageSorter comparator("to");
    std::sort(messages.begin(), messages.end(), comparator);
}
```

### Functors Compared with Function Pointers
Function Pointer를 인자로 받는 함수는 Functor를 인자로 넘길수 없다. 심지어 Functor가 Function Pointer와 동일한 인자와 결과값을 가진다고 해도 넘길수 없다.

이와 비슷하게 Functor를 인자로 받는 함수는 Function pointer를 인자로 넘길수 없다.
### Functors, Function Pointers and Templates
Function Pointer와 Fuctor를 동일한 함수에 인자로 넘기고자 한다면, 템플릿을 사용해야 한다. 템플릿 함수는 Functor와 Function Pointer를 적절한 타입으로 추론하고 Functor와 Function Pointer를 동일한 방식으로 사용한다. 둘 다 함수 호출과 같은 형태를 취한다. 그래서 해당 함수는 컴파일이 무사히 된다.

예를 들어 보면 아래와 같은 함수가 있다.


```cpp
std::vector<int> find_matching_numbers(std::vector<int> vec, bool (*pred)(int))
{
    std::vector<int> ret_vec;
    std::vector<int>::iterator itr = vec.begin(), end = vec.end();
    while (itr != end)
    {
        if (pred(*itr))
        {
            ret_vec.push_back(*itr);
        }
        ++itr;
    }
}
```

위의 함수를 Functor와 Function Pointers를 모두 사용하도록 두번째 인자를 템플릿화하도록 수정했다.


```cpp
template <typename FuncType>
std::vector<int> find_matching_numbers(std::vector<int> vec, FuncType pred)
{
    std::vector<int> ret_vec;
    std::vector<int>::iterator itr = vec.begin(), end = vec.end();
    while (itr != end)
    {
        if (pred(*itr))
        {
            ret_vec.push_back(*itr);
        }
        ++itr;
    }
}
```

### Functor vs. Virtual Functions
Functor와 Virtual Function은 매우 관련있다고 한다. 이 둘은 어떤 코드가 알고리즘을 어떻게 선택하도록 할지에 대한 문제를 해결 한다.

자, 상상해보자. doMath라는 함수가 있다. 이 함수는 3개의 인자를 받는다. 2개의 인자는 integer이고, 나머지 하나는 어떤 연산을 수행하는 루틴이다. Functor로 작성한 예는 아래와 같다.


```cpp
template <typename FuncType>
int doMath(int x, int y, FuncType func)
{
    return func(x, y);
}
```

다른 방법으로 computeResult 메소드를 가진 인터페이스 클래스를 생성할 수 있다.


```cpp
class MathComputer
{
public:
    virtual int computeResult(int x, int y) = 0;
};

int doMath(int x, int y, MathComputer* p_computer)
{
    return p_computer->computeResult(x, y);
}
```

위의 두 예제는 virtual function과 functor 모두 특정 알고리즘이 실행되는 시점에 동적으로 선택을 할 수 있다.
두 개의 주요한 차이점은 virtual function은 템플릿 함수를 사용하더라도 function pointer를 인자로 받을 수 없다. 