---
title: "[알고리즘] Javascript로 DFS, BFS구현하기"
categories: 
  - Algorithm
tags:
  - algorithm
  - dfs
  - bfs
  - javascript
toc: true
---
## DFS란?  
DFS란 Depth First Search의 약자로 하나의 정점을 깊게 파고들어 탐색하는 방식입니다.  
스택을 이용하여 구현할 수 있으며, 재귀 함수의 특징을 통한 구현이 일반적입니다.  
<br>

## BFS란?  
BFS란 Breadth First Search의 약자로 DFS처럼 하나의 경로를 깊게 탐색하지 않고 넓게 검사합니다.  
큐를 이용하여 구현할 수 있습니다.
<br>

## 구현 전체 코드 (Javascript)
~~~ javascript
class Queue {
    constructor() {
        this._arr = [];
    }
    enqueue(elem) {
        return this._arr.push(elem);
    }
    dequeue() {
        return this._arr.shift();
    }
    isEmpty() {
        return this._arr.length === 0 ? true : false;
    }
}

function createMap(w, h) {
    let map = [];
    map.__proto__.w = w;
    map.__proto__.h = h;
    map.__proto__.sx = 0;
    map.__proto__.sy = 0;
    map.__proto__.dx = w - 1;
    map.__proto__.dy = h - 1;

    for (let a=0; a<h; a++) {
        let row = [];
        for (let b=0; b<w; b++) {
            row.push( Math.random() < 0.3 ? 0 : 1 );
        }
        map.push(row);
    }

    map[map.sy][map.sx] = 8;
    map[map.dy][map.dx] = 9;

    return map;
}

function printMap(map) {
    let w = map[0].length;
    let h = map.length;

    for (let a=0; a<h; a++) {
        for (let b=0; b<w; b++) {
            process.stdout.write(map[a][b] + " ");
        }
        console.log();
    }
}

function dfs(map) {
    let dfs_do = (y, x) => {
        console.log("=");
        printMap(map);

        // down
        if ( y < map.h - 1 && map[y + 1][x] === 1 ) {
            map[y + 1][x] = 2;
            dfs_do(y + 1, x);
        }
        // right
            if ( y < map.w - 1 && map[y][x + 1] === 1 ) {
                map[y][x + 1] = 2;
                dfs_do(y, x + 1);
            }
        // up
        if ( y > 0 && map[y - 1][x] === 1 ) {
            map[y - 1][x] = 2;
            dfs_do(y - 1, x);
        }
        // left
        if ( x > 0 && map[y][x - 1] === 1 ) {
            map[y][x - 1] = 2;
            dfs_do(y, x - 1);
        }

    }

    dfs_do(map.sy, map.sx);
    return map;
}

function bfs(map) {
    let q = new Queue();
    q.enqueue([map.sy, map.sx]);
    while (!q.isEmpty()) {
        const [y, x] = q.dequeue();
        // escape
        if (map[y][x] === 9) {
            map[y][x] === 2;
            break;
        }

        console.log("=");
        printMap(map);

        // up
        if ( y > 0 && map[y - 1][x] === 1 ) {
            map[y - 1][x] = 2;
            q.enqueue([y - 1, x]);
        }
        // down
        if ( y < map.h - 1 && map[y + 1][x] === 1 ) {
            map[y + 1][x] = 2;
            q.enqueue([y + 1, x]);
        }
        // left
        if ( x > 0 && map[y][x - 1] === 1 ) {
            map[y][x - 1] = 2;
            q.enqueue([y, x - 1]);
        }
        // right
        if ( y < map.w - 1 && map[y][x + 1] === 1 ) {
            map[y][x + 1] = 2;
            q.enqueue([y, x + 1]);
        }
    }
 
    return map;
}

function main() {
    let map = createMap(5, 5);
    dfs(map);
    bfs(map);
}

main();
~~~
