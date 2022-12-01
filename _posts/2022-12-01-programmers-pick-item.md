---
title: "[programmers] 아이템 줍기"
categories: 
  - programmers
tags:
  - algorithm
  - programmers
toc: true
---

# programmers lv3. 아이템 줍기

1차 
~~~ javascript
class Queue {
    constructor() {
        this._arr = [];
    }
    enqueue(e) {
        return this._arr.push(e);
    }
    dequeue() {
        return this._arr.shift();
    }
    isEmpty() {
        this._arr.length === 0 ? true : false;
    }
}

function printMap(map) {
    for (let i = 0; i < 50; i++) {
        for (let j = 0; j < 50; j++) {
            process.stdout.write(map[i][j] + "");
        }
        process.stdout.write("\n");
    }
}

function createMap() {
    let array = [];
    for (let i = 0; i < 50; i++) {
        let inner = [];
        for (let j = 0; j < 50; j++) {
            inner.push(0);
        }
        array.push(inner);
    }
    
    return array;
}

function fill(map, rect) {
    let x1 = rect[0];
    let y1 = rect[1];
    let x2 = rect[2];
    let y2 = rect[3];
    
    for (let i=x1; i<=x2; i++) {
        for (let j=y1; j<=y2; j++) {
            map[j][i] = 1;
        }
    }
}

function isValid(map, posY, posX) {
    const x1 = posX - 1;
    const y1 = posY - 1;
    const x2 = posX + 1;
    const y2 = posY + 1;

    if (map[posY][posX] === 0) {
        return false;
    }

    for ( let i = x1; i <= x2; i++ ) {
        for (let j = y1; j <= y2; j++ ) {
            if ( map[j][i] === 0 ) {
                return true;
            }
        }
    }

    return false;
}

function solution(rectangle, characterX, characterY, itemX, itemY) {

    //init
    let map = createMap();
    let visited = createMap();
    for (let rect of rectangle) {
        fill(map, rect);
    }
    map[itemY][itemX] = 2;

    //bfs queue
    let q = new Queue();
    
    visited[characterY][characterX] = 1;

    //init characterPos and enqueue
    if (isValid(map, characterY-1, characterX)) {
        //go top
        q.enqueue([characterY-1, characterX, 2]);

    }
    if (isValid(map, characterY, characterX+1)) {
        //go right
        q.enqueue([characterY, characterX+1, 2]);

    }
    if (isValid(map, characterY+1, characterX)) {
        //go bottom
        q.enqueue([characterY+1, characterX, 2]);

    }
    if (isValid(map, characterY, characterX-1)) {
        //go left
        q.enqueue([characterY, characterX-1, 2]);
    }

    //bfs start
    while (!q.isEmpty()) {
        let [y, x, cnt] = q.dequeue();
        visited[y][x] = cnt;
        cnt += 1;

        if (map[y][x] === 2) {
            return visited[y][x];
        }

        if (isValid(map, y-1, x) && visited[y-1][x] === 0) {
            //go top
            q.enqueue([y-1, x, cnt]);

        }
        if (isValid(map, y, x+1) && visited[y][x+1] === 0) {
            //go right
            q.enqueue([y, x+1, cnt]);

        } 
        if (isValid(map, y+1, x) && visited[y+1][x] === 0) {
            //go bottom
            q.enqueue([y+1, x, cnt]);

        }
        if (isValid(map, y, x-1) && visited[y][x-1] === 0) {
            //go left
            q.enqueue([y, x-1, cnt]);

        }
    }

}
~~~
위 코드로 해결이라고 생각했는데 예측치보다 최단거리가 적게 나왔다.

이유를 모르겠어서 검색해봤더니.. 테스트케이스 1번에서 P(3, 5) P(3, 6) 발생하는 반례가 있었다.

실패 ㅠㅠ
