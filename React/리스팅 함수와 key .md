# React

### map vs foreach

컴포넌트 안에서 map을 통해 jsx를 만들던 중, foreach는 왜 사용하지 않는지 의문점이 들었다.

```
{columns.map((column, index) => (
                <TableCell key={index}>{column}</TableCell>
              ))}
```

foreach는 결과값을 return하지 않기 때문에 함수 안에서 컴포넌트 jsx를 출력할 수 없다.

### 리스트와 key

Map 등을 사용하여 테이블 행, 리스트 등에서 key를 사용하는 것이 권장되고 있다.

```
<TableRow>
  {columns.map((column, index) => (
    <TableCell key={index}>{column}</TableCell>
  ))}
</TableRow>
```

여기서는 map의 index를 이용하여 key로 사용하고 있는데, 만약 항목의 순서가 바뀔 수 있는 경우 key에 인덱스를 사용하는 것은 권장되지 않는다.

이로 인해 성능이 저하되고나 컴포넌트 state와 관련된 문제가 생길 수 있음.

- key가 필요한 이유

DOM 노드 자식들을 재귀적으로 처리 할 때 (새로운 노드 추가, 삭제, 순서 변경 등), 각 리스트를 순회하고 차이점이 있으면 변경을 생성한다.

만약 key가 없다면 리스트의 앞에 추가하는 경우 모든 자식을 변경하므로 매우 비효율적으로 동작 할 것이다.

```html
<!-- Duke와 Connecticut을 넣기 위해 아래 리스트를 모두 변경한다. -->

<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

만약 key를 사용한다면, key를 통해 일치 여부를 확인하므로 변경되지 않는 자식들은 새로 만들 필요가 없어진다.

```html
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

