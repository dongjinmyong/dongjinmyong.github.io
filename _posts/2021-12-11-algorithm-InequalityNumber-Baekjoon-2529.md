---
title: 백준 알고리즘 2529번 - 부등호
date: 2022-01-14 +0000
categories: [algorithm, BOJ]
tags: [JavaScript, 백준알고리즘, "2529", dfs]
---

## 문제

[백준 알고리즘 2529번](https://www.acmicpc.net/problem/2529)을 읽어보면 알 수 있겠지만, 이 문제는 주어진 일련의 부등호 사이에 숫자를 넣는 문제이다. 부등호 사이에 들어간 수들은 부등식 관계를 만족시켜야 한다.
문제에서는 그러한 수들 중 가장 큰 수와 가장 작은 수를 구해야 한다.
예를 들어 부등호 문자열이 `"> < < >"` 라고 주어졌을 때, 가장 큰 수는 96785(9>6<7<8>5), 가장 작은 수는 10243(1>0<2<4>)이다.

## 분석

### 일반적인 방식

1. 9부터 적합한 위치에 찾아 넣는다(제일 작은 수를 구할 때에는 0부터)
2. `>` 을 만나기 전까지 만난 `<` 의 갯수만큼 빈칸을 두어야 한다.
   즉 그 갯수만큼 앞에 위치할 숫자가 있다.(제일 작은 수를 구할 때에는 부호가 반대)
3. 2에서 구한 숫자를 카운팅할 때, 왼쪽에서 오른쪽으로, 빈칸들만 카운팅한다(이미 채워진 자리들은 카운팅하지 않는다)

### DFS 방식

## 코드로 구현

### 방식

```js
let fs = require("fs");
let input = fs.readFileSync("/dev/stdin").toString().split("\n");
const [count, signs] = input;

let signArr = signs.split(" ");
const len = signArr.length + 1;
let maxArr = Array(len);
let minArr = Array(len);
let max = 9;
let min = 0;

while (max > 9 - len) {
  let step = 0;
  while (signArr.length && signArr[step] === "<") step++;
  let startIdx = 0;
  while (maxArr[startIdx] !== undefined) startIdx++;
  let i = startIdx;
  while (step > 0) {
    if (maxArr[i] === undefined) {
      i++;
      step--;
    } else {
      i++;
    }
  }
  maxArr[i] = max--;
  signArr.shift();
}

signArr = signs.split(" ");
while (min < len) {
  let step = 0;
  while (signArr.length && signArr[step] === ">") step++;
  let startIdx = 0;
  while (minArr[startIdx] !== undefined) startIdx++;
  let i = startIdx;
  while (step > 0) {
    if (minArr[i] === undefined) {
      i++;
      step--;
    } else {
      i++;
    }
  }
  minArr[i] = min++;
  signArr.shift();
}

console.log(maxArr.join(""));
console.log(minArr.join(""));
```

### DFS 방식

```js

```

## 마치며

이 문제를 DFS로 접근할 수 있다고 하는 블로깅을 여럿 보았다.
나는 DFS로 풀 생각을 선뜻 하지 못했다.
하나하나 따지면서 수를 만들어나가는 방법이 있을 것 같다고 느꼈고, 결국 그렇게 풀었다.
그러면 DFS로 풀지 않았다고 나쁜 코드라고 생각지는 않는다.
실제로 백준에 제출한 결과가 나쁘지 않았다. `node.js`로 제출한 풀이 중에 3위에 랭킹되었다.
실행시간이나 코드길이, 메모리 사용량이 나쁜지 않았다.
다만 방식으로 풀면 일일히 따지기 귀찮다.
chrome 개발자도구에 코드를 띄어 놓고 디버깅하면서 코드를 작성했다.
시간이 많이 들었다. 거의 3시간 들었던 것 같다.
이 정도 알고리즘 한 문제 푸는 데 3시간이라니...
