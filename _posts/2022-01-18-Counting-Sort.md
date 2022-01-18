---
title: Counting Sort / 계수정렬
date: 2022-01-18 +0000
categories: algorithm
tags: [JavaScript, CountingSort, 계수정렬, 정렬알고리즘]
---

## 문제

Counting Sort(계수 정렬)을 이용해 오름차순 정렬한다.
입력으로는 문자열이 될 수도 있고, 정수가 될 수도 있다.
먼저 정수배열을 오름차순으로 정렬해보도록 하자.

```
입력: [4, 3, 1, 3, 0, 9, 8]
출력: [0, 1, 3, 3, 4, 8, 9]
```

기본 로직은 다음과 같다.

- 수의 범위 `[max, min]`을 계산하고, 그만한 길이를 가지는 빈 배열을 만든다. 여기에 각 수들이 나타나는 횟수를 기록할 것이기 때문에 `count`라는 이름으로 배열을 만든다. 모든 값을 0으로 초기화한다.
  ```
  arr = [4, 3, 1, 3, 0, 9, 8]
  count = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
  ```
- 입력 배열 `arr`이 정렬된 결과가 될 배열 `output`을 만든다.
  ```
  output = [0, 0, 0, 0, 0, 0, 0]
  ```
- `count`배열에 값을 넣는다. `count[i]`의 값은 `arr` 배열에서 `i` 값을 가지는 수의 개수이다.
  ```
  count = [1, 1, 0, 2, 1, 0, 0, 0, 1, 1]
  ```
- `count` 배열을 누적 배열(누적함수와 비슷)로 변환한다.
  ```
  count = [1, 2, 2, 4, 5, 5, 5, 5, 6, 7]
  ```
- 여기까지 진행하면 다음과 같은 관계가 성립한다.
  `arr`배열의 어떤 값 `n`이 `output` 배열에서 위치하는 인덱스는 `count[n]`이 된다. 예를 들어 `arr`배열의 수인 `8`은 `count[8] = 6`이므로 `output`배열의 `6`번째 위치에 있다. 익덱스로는 5번이다.
  이와 같은 관계를 확인했으므로 `output`배열에 값을 하나하나 넣어주면 된다.
- 이해하기 제일 어려웠던 부분이다. 따져 보면 맞긴 한데, 바로 코드로 작성하기에는 시간이 걸리는 부분이다. 외우자!
  ```js
  for (let i = arr.length - 1; i >= 0; i++) {
    output[count[arr[i] - min] - 1] = arr[i];
    count[arr[i] - min]--; // 같은 수가 여러개 있는 경우를 대비해
  }
  ```

## 코드(JavaScript)

```js
function countSort(arr) {
  const max = Math.max(...arr);
  const min = Math.min(...arr);
  const range = max - min + 1;
  const len = arr.length;

  const count = Array(range).fill(0);
  const output = Array(len).fill(0);

  for (let i = 0; i < len; i++) {
    count[arr[i] - min]++;
  }

  for (let i = 1; i < count.length; i++) {
    count[i] += count[i - 1];
  }

  for (let i = len - 1; i >= 0; i--) {
    output[count[arr[i] - min] - 1] = arr[i];
    count[arr[i] - min]--;
  }

  return output;
}
```
