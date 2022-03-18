# Parallel Foreach

최근 회사에서 병렬 프로그래밍을 할 일이 있었다.

처음으로 병렬 코드를 작성하여 api 응답속도를 30초 -> 3초로 줄이는 데 성공했다.

하지만 api 응답에서 각 병렬 처리에서 처리된 데이터 수를 반환해줘야 하는데 변수에 값이 전혀 담기지 않았다. (항상 0으로 응답)

왜 그런지 찾아보니 내가 작성한 코드가 thread safe하지 못하기 때문에 변수에 값이 담기지 않았다고 한다.

그래서 thread safe가 뭐야?

## Thread Safe - 위키백과 정의

> 스레드 안전(thread 安全, 영어: thread safety)은 멀티 스레드 프로그래밍에서 일반적으로 어떤 함수나 변수, 혹은 객체가 여러 스레드로부터 동시에 접근이 이루어져도 프로그램의 실행에 문제가 없음을 뜻한다. 보다 엄밀하게는 하나의 함수가 한 스레드로부터 호출되어 실행 중일 때, 다른 스레드가 그 함수를 호출하여 동시에 함께 실행되더라도 각 스레드에서의 함수의 수행 결과가 올바로 나오는 것으로 정의한다.

## 내가 작성한 코드

내가 기존에 잘못 작성한 코드를 보자. (회사코드는 비공개이기 때문에 축약해서)

```csharp
int result = 0; // 변경된 데이터의 수를 담기 위한 변수

var list = GetLists();

Parallel.ForEach(list, (item) => {
    result += item.save();
});

return result;
```

이 코드에서 `result += item.save();`가 바로 thread safe하지 못한 코드다.

왜냐하면 병렬프로그래밍이라는 건 결국 멀티 쓰레드를 사용하는 것이기 때문에 `result`라는 변수에 담긴 값이 각 쓰레드에서 처리할 때마다 다르기 때문이다.

따라서 단순히 저런 식으로 변수에 더하기를 해서 처리하는 것이 아니라 추가적인 처리를 해줘야 한다.

## 올바르게 변경한 코드

```csharp
int result = 0; // 변경된 데이터의 수를 담기 위한 변수

var list = GetLists();

Parallel.ForEach(
    list,
    () => 0, // partition-local variable 초기화
    (item, loopState, sum) => {
        sum += item.save();
        return sum;
    },
    (finalSum) => Interlocked.Add(ref result, finalSum) // 최종 공유자원 전달
);

return result;
```

Parallel 클래스에 굉장히 다양한 ForEach 오버로드가 있는데 그 중, 공유자원을 처리해야할 때 써야하는 메소드이다.

## 메소드 설명

```csharp
public static System.Threading.Tasks.ParallelLoopResult ForEach<TSource,TLocal> (System.Collections.Generic.IEnumerable<TSource> source, Func<TLocal> localInit, Func<TSource,System.Threading.Tasks.ParallelLoopState,TLocal,TLocal> body, Action<TLocal> localFinally);
```

아.. 굉장히 복잡해 보인다.

간략하게 변경해보자면

```csharp
ForEach (source, lacalInit, body, localFinally);
```

`source`

- 병렬처리할 데이터 소스이다. `IEnumerable<T>`를 구현해야 한다.

`localInit`

- 파티션 지역 변수를 초기화하는 함수이다. 이 함수는 ForEach 작업이 실행되는 각 파티션에 대해 한 번 호출된다.

`body`

- 루프가 반복될 때마다 호출되는 함수. 파라미터는 `(TSource, ParallelLoopState, TLocal)`이다.
- `TSource`는 `IEnumerable<T>`의 현재 요소
- `ParallelLoopState`는 루프의 상태를 검토하기 위해 대리자 코드에 사용할 수 있는 변수
- `TLocal`은 파티션 지역 변수.

`localFinally`

- 각 파티션의 루프 작업이 완료될 때 호출되는 대리자이다. 이 루프 파티션에 대한 파티션 지역 변수의 최종 값을 대리자에게 전달하고, 사용자는 이 파티션의 결과를 다른 파티션의 결과와 결합하는 데 필요한 작업을 수행하는 코드를 제공한다.
