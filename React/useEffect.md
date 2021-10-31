# React

### UseEffect

#### 기본 정의

- 리액트 컴포넌트가 *렌더링* 될 때마다 특정 작업을 실행할 수 있도록 도와주는 Hook
- *렌더링*
  - Component > Mount, Unmount, Update 시 특정 작업 실행 가능
- 클래스형 컴포넌트에서 사용하던 생명주기 메소드를 함수형 컴포넌트에서 사용해 줄 수 있게 한다.



#### 사용법

* 기본 형태

  * useEffect(function, deps)

* function - 수행하고자 하는 특정 작업

* deps - 배열 형태, 특정 값 or 빈 배열이 들어간다.

  * 컴포넌트 mount 시, 한 번만 실행하고 싶으면 deps를 비워둔다.

  * ~~~react
    useEffect(() => {
    	console.log('한 번만 실행된다.')
    },[])
    
    // 배열 생략 시 리렌더링 될 때마다 실행
    ~~~

  * 컴포넌트 update 시, 배열 안에 원하는 props, state를 넣는다.

  * ~~~react
      /*========== useEffect ==========*/
      useEffect(() => {
        try {
          setLoading(true);
          getGrantServiceList(page, rowsPerPage).then((data) => {
            setGrant(data.response);
            setError(data.error);
          });
          setLoading(false);
        } catch (e) {
          setError(e);
          alert('getGrantServiceList error message: ' + error);
        }
      }, [page]);
    
    // page가 변경될 때마다 useEffect 내부가 실행된다. (mount 시에도 실행)
    ~~~



#### Tip

* 만약 mount 시에는 useEffect 실행을 원하지 않는다면 다음과 같은 방법을 사용한다.

~~~react
/*
초기 mount시에는 mount의 값을 변경해주는 값만 실행되고,
이후에는 원하는 함수가 실행된다.
*/

const mounted = useRef(false);

useEffect(() => {
  if(!mounted.current) {
    mounted.current = true;
  } else {
    // 원하는 함수 실행
  }
}, [deps]);
~~~



### 




