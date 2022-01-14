---
title: Matrix Rotation / 행렬회전
date: 2022-01-14 +0000
categories: algorithm
tags: [JavaScript, MatrixRotation]
---

## 문제

n\*m 크기의 행렬을 시계방향으로 90\*k도 만큼 회전시킨 행렬을 구하는 알고리즘.
행렬은 JS의 2차원 배열로 주어진다.

```js
const matrix = [
  [1, 2, 3, 4],
  [3, 0, 9, 7],
  [2, 7, 8, 5],
];

console.log(rotateMatrix(matrix, 1)); // 1은 90 * 1도 만큼 회전한다는 뜻
/*
[
  [2, 3, 1],
  [7, 0, 2],
  [8, 9, 3],
  [5, 7, 4]
]
*/
```

## 분석

크기가 `n*m`인 행렬을 90도 만큼 회전시키면 크기가 `m*n`인 행렬이 된다.
또한, 90도 만큼 두 번 회전시키면 크기가 `n*m`인 행렬이 된다.
즉 90도씩 몇 번 돌리냐에 따라 결과 행렬의 크기가 달라진다.
일반화하여 표기하면

```txt
k: 90도씩 회전하는 횟수
n*m: 행렬의 크기

k % 2 = 0 => 결과행렬의 크기는 n*m
k % 2 = 1 => 결과행렬의 크기는 m*n
```

회전 횟수에 따라 결과행렬의 크기가 가변적이므로, 이중 `for`문을 돌리면서 결과행렬의 값을 구할 때, `index`값의 범위도 가변적이다.
그래서 회전횟수에 따라 `iterator`의 범위도 알맞게 정해지도록 하는 로직이 필요하다.
로직은 아래 코드에서 확인.

그 다음은 회전된 결과 행렬의 값을 구하는 로직이 필요하다.
자세히 따져보면서 하면 되는데, 좀 많이 헷갈린다.
자세한 설명은 주석 참고.

## 코드로 구현한 알고리즘

```js
const rotateMatrix = function (matrix, rotateNum = 1) {
  let n = matrix.length; // 행의 수
  let m = matrix[0] && matrix[0].length; // 열의 수

  const IJ = rotateNum % 2 ? [m, n] : [n, m];
  // rotateNum에 따라 가변적으롤 정해지는 길이 2의 1차원 배열.
  // 0번 인덱스에는 결과행렬의 행의 수
  // 1번 인덱스에는 결과행렬의 열의 수

  let result = n === 0 ? [] : Array.from(Array(IJ[0]), () => Array(IJ[1]));
  // 결과 행렬 선언, 초기값은 모두 undefined

  for (let i = 0; i < IJ[0]; i++) {
    for (let j = 0; j < IJ[1]; j++) {
      switch (rotateNum % 4) {
        case 0:
          // 회전할 필요가 없다. 원래 값을 그대로 할당한다
          result[i][j] = matrix[i][j];
          break;
        case 1:
          // 90도 회전할 때 로직
          result[i][j] = matrix[n - 1 - j][i];
          break;
        case 2:
          // 180도 회전할 때 로직
          result[i][j] = matrix[n - 1 - i][m - 1 - j];
          break;
        case 3:
          // 270도 회전할 때 로직
          result[i][j] = matrix[j][m - 1 - i];
          break;
      }
    }
  }

  return result;
};
```

## 실패 케이스

빈 배열을 받았을 때 빈 배열을 리턴해야 한다.

```js
let result = Array.from(Array(IJ[0]), () => Array(IJ[1]));
```

이렇게 작성하면 빈배열을 받았을 때 `[[]]`이렇게 2차 배열을 리턴하게 된다.
이것을 해결하기 위해서 빈배열일 때 빈 배열을 리턴하도록 하는 과정이 필요하다.

```js
let result = n === 0 ? [] : Array.from(Array(IJ[0]), () => Array(IJ[1])
```

이렇게 변경하여 해결했다.
