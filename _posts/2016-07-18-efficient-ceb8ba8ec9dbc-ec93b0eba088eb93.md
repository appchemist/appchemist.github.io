---
layout: post
title: EFFICIENT C++_단일 쓰레드 메모리 풀링
date: '2016-07-18T09:49:54+09:00'
tags:
- c++
- Efficient C++
- Memory
- Performance
- Pool
- Single Thread
tumblr_url: https://appchemist.tumblr.com/post/147593483631/efficient-ceb8ba8ec9dbc-ec93b0eba088eb93
---
메모리를 자주 할당하고 해지하는 것은 응용프로그램 성능을 저하시키는 주요한 요소이다. 성능 저하는 기본 메모리 관리자가 일반적인 목적을 가지고 있기 때문에 발생한다. 특화된 메모리 관리자를 개발하여 이러한 사항을 극복할 수 있다. 이러한 메모리 관리자를 개발하고자 할때, 메모리의 크기와 동시성이라는 문제를 생각해야 한다.
메모리 크기는 다음 2가지를 생각할 수 있다.
- 고정 크기 – 고정 크기의 메모리 블록을 할당
- 가변 크기 – 다양한 크기의 메모리 블록을 할당
동시성에서도 2가지를 생각할 수 있다.
- 단일 쓰레드 – 하나의 쓰레드가 메모리 관리자를 사용
- 멀티 쓰레드 – 여러 쓰레드가 메모리 관리자를 사용
이렇게 메모리 관리자는 4버전으로 만들 수 있다. 이번에는 단일 쓰레드 차원에서 메모리 관리자간의 성능을 확인해 보자.
**버전 0: 전역 new()와 delete()**
기본 메모리 관리자는 일반적인 역활을 수행한다. 기본 메모리 관리자는 전역 new()와 delete()를 호출할 때 사용이 된다. 기본 메모리 관리자는 멀티쓰레드 환경에서 작동할 수 있다. 더불어 메모리 크기는 요청이 발생할 때마다 변할 수 있다. 이러한 유연성은 속도와 절충된다.
클라이언트 코드가 전역 new()와 delete()의 전 기능이 필요하지 않는 경우가 있다. 이런 경우, 해당 함수의 전 기능을 사용하는것은 CPU 주기 낭비이다.
아래는 전역 new와 delete를 사용한 코드이다.


```cpp
    class Rational {
    public:
        Rational(int a = 0, int b = 0) : n(a), d(b) { }

    private:
        int n; // 분자
        int d; // 분모
    };
```

Rational 클래스를 사용하여 전역 new(), delete()의 성능을 측정해보자. 


```cpp
TEST(Chapter6, Normal_NEW_DELETE) {
    Rational *array[1000];

    for (int j = 0; j < 5000; j++) {
        for (int i = 0; i < 1000; i++) {
            array[i] = new Rational(i);
        }
        for (int i = 0; i < 1000; i++) {
            delete array[i];
        }
    }
}
```


성능은 gtest에서 해당 테스트의 걸린 시간으로 측정했다. 약 2600 millisecond로 측정이 됐다.
**버전 1: Rational에 특화된 메모리 관리자**
기본 메모리 관리자를 너무 자주 호출하는 것을 방지하기 위해서 Rational 클래스는 미리 할당된 Rational 객체들의 정적 연결 리스트로 유지한다.  새로운 Rational 객체가 필요하면, 해당 리스트에서 하나를 가져온다. 사용이 끝나면 이후에 사용할 수 있도록 이것을 리스트로 반환한다. 


```cpp
    class NextOnFreeList {
    public:
        NextOnFreeList *next;
    };

    class RationalVer1 {
    private:
        static void expandTheFreeList();

    public:
        RationalVer1(int a = 0, int b = 0) : n(a), d(b) { }

        inline void *operator new(size_t size);
        inline void operator delete(void *doomed, size_t size);
        static void newMemPool() { expandTheFreeList(); }
        static void deleteMemPool();

    private:
        static NextOnFreeList *freeList;

        enum { EXPASION_SIZE = 32 };

        int n; // 분자
        int d; // 분모
    };

    inline void* RationalVer1::operator new(size_t size) {
        if (0 == freeList) {
            expandTheFreeList();
        }

        NextOnFreeList *head = freeList;
        freeList = head->next;

        return head;
    }

    inline void RationalVer1::operator delete(void *doomed, size_t size) {
        NextOnFreeList *head = static_cast<NextOnFreeList*>(doomed);

        head->next = freeList;
        freeList = head;
    }

    NextOnFreeList *RationalVer1::freeList = NULL;

    void RationalVer1::expandTheFreeList() {
        size_t size = (sizeof(RationalVer1) > sizeof(NextOnFreeList *)) ? sizeof(RationalVer1) : sizeof(NextOnFreeList *);

        NextOnFreeList *runner = (NextOnFreeList *)new char[size];

        freeList = runner;
        for (int i = 0; i < EXPASION_SIZE; i++) {
            runner->next = (NextOnFreeList *)new char[size];
            runner = runner->next;
        }
        runner->next = 0;
    }

    void RationalVer1::deleteMemPool() {
        NextOnFreeList *nextPtr;

        for (nextPtr = freeList; nextPtr != NULL; nextPtr = freeList) {
            freeList = freeList->next;
            delete [] nextPtr;
        }

    }
```

한번에 한 블록씩 할당 받지만 미리 받은 메모리 블록은 재활용한다는 점에서 버전 0과는 다른 점이다.


```cpp
TEST(Chapter6, VER1_NEW_DELETE) {
    RationalVer1*array[1000];

    Rational::newMemPool();

    for (int j = 0; j < 5000; j++) {
        for (int i = 0; i < 1000; i++) {
            array[i] = new RationalVer1(i);
        }
        for (int i = 0; i < 1000; i++) {
            delete array[i];
        }
    }

    Rational::deleteMemPool();
}
```

이번 버전1의 경우 약 100 millisecond가 걸렸다. 기본 메모리 관리자에 비해서 약 26배 정도 속도 차이가 나는 것을 확인할 수 있다.
**버전 2: 고정 크기 객체 메모리 풀**
이번에는 템플릿을 이용하여 관리하고자 하는 다양한 클래스를 관리할 수 있도록 작성할 것이다.


```cpp
    template<class T>
    class MemoryPool {
    public:
        MemoryPool(size_t size = EXPANSION_SIZE);

        ~MemoryPool();

        inline void *alloc(size_t size);

        inline void free(void *someElement);

    private:
        MemoryPool<T> *next;

        enum {
            EXPANSION_SIZE = 32
        };

        void expandTheFreeList(int howMany = EXPANSION_SIZE);
    };

    template <class T>
    inline
    void* MemoryPool<T>::alloc(size_t)
    {
        if (!next) {
            expandTheFreeList();
        }

        MemoryPool<T> *head = next;
        next = head->next;

        return head;
    }

    template <class T>
    inline
    void MemoryPool<T>::free(void *doomed)
    {
        MemoryPool<T> *head = static_cast<MemoryPool<T> *>(doomed);

        head->next = next;
        next = head;
    }

    template <class T>
    void MemoryPool<T>::expandTheFreeList(int howMany) {
        size_t size = (sizeof(T) > sizeof(MemoryPool<T> *)) ? sizeof(T) : sizeof(MemoryPool<T> *);

        MemoryPool<T> *runner = (MemoryPool<T> *)new char[size];

        next = runner;
        for (int i = 0; i < howMany; i++) {
            runner->next = (MemoryPool<T> *)new char[size];
            runner = runner->next;
        }

        runner->next = 0;
    }

    template <class T >
    MemoryPool<T>::MemoryPool(size_t size) {
        expandTheFreeList(size);
    }

    template <class T>
    MemoryPool<T>::~MemoryPool() {
        MemoryPool<T> *nextPtr = next;
        for (; nextPtr != NULL; nextPtr = next) {
            next = next->next;
            delete [] nextPtr;
        }
    }
```





```cpp
    class NextOnFreeList {
    public:
        NextOnFreeList *next;
    };

    class RationalVer1 {
    private:
        static void expendTheFreeList();

    public:
        RationalVer1(int a = 0, int b = 0) : n(a), d(b) { }

        inline void *operator new(size_t size);
        inline void operator delete(void *doomed, size_t size);
        static void newMemPool() { expendTheFreeList(); }
        static void deleteMemPool();

    private:
        static NextOnFreeList *freeList;

        enum { EXPASION_SIZE = 32 };

        int n; // 분자
        int d; // 분모
    };

    inline void* RationalVer1::operator new(size_t size) {
        if (0 == freeList) {
            expendTheFreeList();
        }

        NextOnFreeList *head = freeList;
        freeList = head->next;

        return head;
    }

    inline void RationalVer1::operator delete(void *doomed, size_t size) {
        NextOnFreeList *head = static_cast<NextOnFreeList*>(doomed);

        head->next = freeList;
        freeList = head;
    }

    NextOnFreeList *RationalVer1::freeList = NULL;

    void RationalVer1::expendTheFreeList() {
        size_t size = (sizeof(RationalVer1) > sizeof(NextOnFreeList *)) ? sizeof(RationalVer1) : sizeof(NextOnFreeList *);

        NextOnFreeList *runner = (NextOnFreeList *)new char[size];

        freeList = runner;
        for (int i = 0; i < EXPASION_SIZE; i++) {
            runner->next = (NextOnFreeList *)new char[size];
            runner = runner->next;
        }
        runner->next = 0;
    }

    void RationalVer1::deleteMemPool() {
        NextOnFreeList *nextPtr;

        for (nextPtr = freeList; nextPtr != NULL; nextPtr = freeList) {
            freeList = freeList->next;
            delete [] nextPtr;
        }

    }
```

테스트 코드는 이전과 유사하다.


```cpp
TEST(Chapter6, VER2_NEW_DELETE) {
    RationalVer2* array[1000];

    Rational::newMemPool();

    for (int j = 0; j < 5000; j++) {
        for (int i = 0; i < 1000; i++) {
            array[i] = new RationalVer2(i);
        }
        for (int i = 0; i < 1000; i++) {
            delete array[i];
        }
    }

    Rational::deleteMemPool();
}
```

이번에 만든 메모리 풀 템플릿 버전의 수행시간은 139 millisecond가 걸렸다. 약간이지만 버전 1에 비해서 느린것을 알 수 있다.
**버전 3: 단일 쓰레드 가변 크기 메모리 관리자**
가변 크기의 메모리를 필요로하는 응용프로그램들이 있는데, 그 중에 하나가 웹 서버이다. 웹 서버는 문자열 처리를 굉장히 많이 한다. 이로 인해서 가변 크기의 메모리 관리자가 필요로 하는데, 이번에 살펴보도록 하자.


```cpp
    class MemoryChunk {
    public:
        MemoryChunk(MemoryChunk *nextChunk = NULL, size_t chunkSize = DEFAULT_CHUNK_SIZE);
        ~MemoryChunk() { delete mem; }

        inline void *alloc(size_t size);
        inline void free(void* someElement);

        MemoryChunk *nextMemChunk() { return next; }

        size_t spaceAvailable()
        {
            return chunkSize - bytesAlreadyAllocated;
        }

        enum { DEFAULT_CHUNK_SIZE = 4096 };

    private:
        MemoryChunk *next;
        void        *mem;

        // 단일 메모리 청크 크기
        size_t chunkSize;

        // 현재 메모리 청크에 할당된 바이트 수
        size_t bytesAlreadyAllocated;
    };

    inline void* MemoryChunk::alloc(size_t requestSize) {
        void *addr = (void*)((size_t)(mem) + bytesAlreadyAllocated);
        bytesAlreadyAllocated += requestSize;

        return addr;
    }

    inline void MemoryChunk::free(void *someElement) { 
        // 할당한 메모리를 재활용하지 않기 때문
    }

    MemoryChunk::MemoryChunk(MemoryChunk *nextChunk, size_t reqSize) {
        chunkSize = (reqSize > DEFAULT_CHUNK_SIZE) ?
                    reqSize : DEFAULT_CHUNK_SIZE;
        next = nextChunk;
        bytesAlreadyAllocated = 0;
        mem = new char[chunkSize];
    }
```





```cpp
    class ByteMemoryPool {
    public:
        ByteMemoryPool(size_t initSize = MemoryChunk::DEFAULT_CHUNK_SIZE);
        ~ByteMemoryPool();

        inline void *alloc(size_t requestSize);

        inline void free(void* someElement);
    private:
        MemoryChunk *listOfMemoryChunks;
        void expandStorage(size_t reqSize);
    };

    inline void* ByteMemoryPool::alloc(size_t requestSize) {
        size_t space = listOfMemoryChunks->spaceAvailable();
        if (space < requestSize) {
            expandStorage(requestSize);
        }

        return listOfMemoryChunks->alloc(requestSize);
    }

    inline void ByteMemoryPool::free(void *doomed)
    {
        listOfMemoryChunks->free(doomed);
    }

    ByteMemoryPool::ByteMemoryPool(size_t initSize) {
        expandStorage(initSize);
    }

    ByteMemoryPool::~ByteMemoryPool() {
        MemoryChunk *memChunk = listOfMemoryChunks;

        while(memChunk != NULL) {
            listOfMemoryChunks = memChunk->nextMemChunk();
            delete memChunk;
            memChunk = listOfMemoryChunks;
        }
    }

    void ByteMemoryPool::expandStorage(size_t reqSize) {
        listOfMemoryChunks = new MemoryChunk(listOfMemoryChunks, reqSize);
    }
```




```cpp
    class RationalVer3 {
    public:
        RationalVer3(int a = 0, int b = 1) : n(a), d(b) {  }
        ~RationalVer3() {  }

        void *operator new(size_t size) { return memPool->alloc(size); }
        void operator delete(void *doomed, size_t size) { memPool->free(doomed); }

        static void newMemPool() { memPool = new ByteMemoryPool; }
        static void deleteMemPool() { delete memPool; }

    private:
        int n;
        int d;

        static ByteMemoryPool *memPool;
    };

    ByteMemoryPool *RationalVer3::memPool = NULL;
```

이번 버전의 경우 176 millisecond가 걸렸다. 
**키 포인트**
- 메모리 관리의 강력함과 유연성은 성능에 영향을 준다.
- 전역 메모리 관리자는 일반적 목적을 가지고 구현, 성능에는 안 좋다.
- 고정된 크기의 메모리 블록을 주로 할당하면, 특화된 고정 크기 메모리 관리자가 성능에 좋다.
- 단일 쓰레드와 한정된 메모리 블록을 주로 할당하면 위와 같이 성능 개선이 가능.
