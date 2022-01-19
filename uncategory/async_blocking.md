# 개요

보통 blocking = sync, non-blocking = async 라고 생각을 한다.

일반적인 상황에서는 크게 문제가 되지 않는다.

대부분은 sync&blocking, async&non-blocking 의 조합으로 사용된다고 보면 된다.

# blocking | non-blocking

blocking 이냐, non-blocking 이냐의 관점은 호출된 쪽에서 제어권을 바로 넘겨주느냐 이다.

## blocking

제어권을 넘겨주지 않아서 호출한 쪽에서 제어권을 기다리느라 다른 작업을 진행하지 못하는 경우이다.

## non-blocking

반대로, 호출된 쪽에서 제어권을 바로 넘겨주어 호출한 쪽에서 다른 작업을 바로 진행할 수 있는 경우이다.

# sync | async

sync 냐, async 냐의 관점은 일련의 작업들이 시간을 맞춰 진행되는지 이다.

## sync

일련의 함수들이 동시에 시작하거나, 하나가 끝나면 다른 하나가 시작되는 등 시간에 맞추어 작업이 진행되는 경우이다.

또는 호출한 쪽에서 호출한 함수가 완료되었는 지를 계속해서 확인하는 경우이다.

## async

반대로, 일련의 함수들이 서로의 작업 시간에 대해 영향이 전혀 없는 경우이다.

# References

- [joooing](https://joooing.tistory.com/entry/%EB%8F%99%EA%B8%B0%EB%B9%84%EB%8F%99%EA%B8%B0-%EB%B8%94%EB%A1%9C%ED%82%B9%EB%85%BC%EB%B8%94%EB%A1%9C%ED%82%B9)
