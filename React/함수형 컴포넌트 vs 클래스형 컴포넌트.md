# React

### 함수형 컴포넌트 vs 클래스형 컴포넌트

----

* 함수형 컴포넌트

1. state, lifeCycle 기능 사용 불가능 (Hook을 통해 해결 / useState 사용)
2. 클래스형 컴포넌트보다 메모리를 덜 사용
3. 코드 간결

* 클래스형 컴포넌트

1. state, lifeCycle 기능 사용 가능 (state의 초기값 및 직접 수정 가능)
2. 함수형 컴포넌트보다 메모리 더 사용
3. 임의 메소드 정의 가능



* 함수형 컴포넌트를 사용하는 이유

> this는 mutable하지만, props는 immutable하다.

1. 리랜더링 될 때의 값을 유지 (immutable)
2. 코드 간결 + 가독성
3. props에 따른 랜더링 결과를 보장받는다.
