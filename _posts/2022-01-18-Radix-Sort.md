---
title: Radix Sort / 기수정렬
date: 2022-01-18 +0000
categories: algorithm
tags: [JavaScript, RadixSort, 기수정렬, 정렬알고리즘]
---

## 문제

정수 배열을 입력받아 오름차순 정렬한다.

```
입력: 170, 45, 75, 90, 802, 24, 2, 66
출력: 2, 24, 45, 66, 75, 90, 170, 802
```

## 기본로직

- 1의 자리 숫자를 기준으로 정렬한다.
  ```
  170, 90, 802, 2, 24, 45, 75, 66
  ```
- 얻어진 배열을 10의 자리 숫자를 기준으로 정렬한다.
  ```
  802, 2, 24, 45, 66, 75, 170, 90
  ```
- 다시 100의 자리 숫자를 기준으로 정렬한다.
  ```
  2, 24, 45, 66, 75, 90, 170, 802
  ```
  정렬 횟수는 입력값 중 가장 긴 수의 자릿수가 된다.
  예제에서는 가장 긴 수인 170과 802의 길이가 3이고, 정렬 3번 만에 결과를 얻는다.

각각의 정렬은 `Counting Sort`로 구현한다.
계수 정렬을 사용하는 이유는 정렬하려는 값의 개수가 적을 때에는 계수정렬이 극한의 효율을 나타내기 때문이다.
최대값이 k이고 입력 값의 개수가 n일 때, 계수 정렬의 시간복잡도는 n + k이다. `Radix Sort`에 내부에서 사용하는 정렬은 0~9, 10개의 숫자들의 비교이기 때문에 계수 정렬을 사용하는 것이 좋다.

## 코드(JavaScript)

```js
countingSort = function (arr, exp) {
  const len = arr.length;
  const count = Array(10).fill(0);
  const output = Array(len).fill(0);

  for (let i = 0; i < len; i++) {
    count[Math.floor(arr[i] / exp) % 10]++;
  }

  for (let i = 1; i < 10; i++) {
    count[i] += count[i - 1];
  }

  for (let i = len - 1; i >= 0; i--) {
    output[count[Math.floor(arr[i] / exp) % 10] - 1] = arr[i];
    count[Math.floor(arr[i] / exp) % 10]--;
  }

  for (let i = 0; i < len; i++) {
    arr[i] = output[i];
  }
};

radixSort = function (arr) {
  const posArr = [];
  const negArr = [];

  // 음수 배열, 양수 배열로 나누기
  for (let item of arr) {
    if (item >= 0) posArr.push(item);
    else negArr.push(-item);
  }

  // 양수 배열을 정렬하기
  let posMax = Math.max(...posArr);
  for (let exp = 1; Math.floor(posMax / exp) > 0; exp *= 10) {
    countingSort(posArr, exp);
  }

  // 음수 배열을 정렬하기
  let negMax = Math.max(...negArr);
  for (let exp = 1; Math.floor(negMax / exp) > 0; exp *= 10) {
    countingSort(negArr, exp);
  }

  // 합쳐서 출력
  return negArr
    .reverse()
    .map((item) => -item)
    .concat(posArr);
};
```

## 추가 설명

위 코드는 입력 배열이 음수를 포함하는 경우까지 고려했다.
음수, 양수를 나누어 각각 정리한 다음, 최종적으로 붙여 출력했다.
