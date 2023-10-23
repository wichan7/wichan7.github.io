---
title: "GraphQL 서버 구성해보기"
categories: 
  - Web
tags:
  - web
  - graphql
  - nosql
toc: true
---

## GraphQL이란 무엇인가
> GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data.  
...  
GraphQL gives clients the power to ask for exactly `what they need and nothing more`, makes it easier to evolve APIs over time, and enables powerful developer tools.

공식 문서의 표현을 보면, 필요한 데이터만을 정확히 응답하는 Query Language라고 이야기한다.  
Client는 GraphQL을 작성해 GraphQL 서버와 통신하고, 데이터를 얻을 수 있다.

## REST의 문제점
RESTful한 요청은 보통 다음과 같이 수행된다.  
`GET https://example.com/users`  

이 때의 응답은 다음과 같이 예상할 수 있다.  
``` json
{
    "count": 3,
    "users": [
        {
            "id": 1,
            "firstName": "gildong",
            "lastName": "hong",
            "age": 25,
            "address": "서울시 강남구 역삼동"
        }, {
            "id": 2,
            "firstName": "chulsoo",
            "lastName": "kim",
            "age": 23,
            "address": "경기도 남양주시 별내동"
        }, {
            "id": 3,
            "firstName": "wichan",
            "lastName": "kang",
            "age": 24,
            "address": "서울시 중랑구 신내동"
        }
    ]
}
```

내가 사람들의 주소가 필요해서 `/users`를 요청한다면, 주소를 얻을 수 있지만 필요 없는 이름, 나이, 주소 등의 데이터도 함께 받게 된다.  
이런 현상을 `Over Fetching`이라고 부른다.  

두번째로, 주소와 함께 사람들의 프로필 사진이 필요해서 `/user/:id/profile`을 호출한다고 가정하자.  
우리는 주소와 사람들의 프로필 이미지를 얻기 위해, `/users`를 호출한 다음, users 만큼 `/user/:id/profile`을 반복 호출 해야한다.  
이렇게 한번의 Request로 원하는 데이터를 모두 얻지 못하면 `Under Fetching`이라고 부른다.  

## GraphQL의 솔루션
GraphQL은 Over/Under fetching을 해결하기 위해, 하나의 통합된 인터페이스를 제공한다.  
마치 RDB interface에서 Select문을 작성하듯이, GraphQL에 맞는 Query를 날려주기만 하면 된다.  

## GraphQL의 Request
GraphQL의 구조는 다음과 같다.  
``` graphql
{
    users {
        id
        address
        profile
    }
}
```

이렇게 GraphQL 서버로 Post 요청을 보내면, GraphQL 서버가 쿼리를 파싱하고, 적절한 동작을 수행한다.  

우리는 GQL서버가 자동으로 수행하는 적절한 동작 사이에 소스 코드를 작성해, 데이터를 불러오고, 결합하는 등 원하는대로 데이터를 다룰 수 있게 된다.  

Data에 대한 Structure 및 Documentation도 어썸하게 작성이 가능하다.

## 구현에 앞서
GraphQL은 스펙일 뿐이어서, 누가 만든 구현체를 가져와서 활용할 수 있다.  
해당 게시글에서는 Apollo Server라고 부르는 GraphQL 구현체를 통해 구현했다.  

목표는 GraphQL의 이해이기 때문에 실제 Database를 연결하지는 않고, 메모리 상에서만 데이터가 유지된다.  

## server.js
``` javascript
import { ApolloServer, gql } from "apollo-server";
import { db } from "./database.js";

let {tweets, users} = db;
const typeDefs = gql`
    """
    사용자 정보를 나타냅니다.
    """
    type User {
        id: ID!
        firstName: String!
        lastName: String!
        fullName: String!
    }

    """
    게시글을 나타냅니다.
    """
    type Tweet {
        id: ID!
        text: String!
        author: User
    }
    
    type Query {
        """
        모든 트윗을 가져옵니다.
        """
        allTweets: [Tweet!]!

        """
        지정한 :id의 트윗을 가져옵니다.
        """
        tweet(id: ID!): Tweet
        
        """
        모든 사용자의 정보를 가져옵니다.
        """
        allUsers: [User!]!
    }
    type Mutation {
        postTweet(text: String!, userId: ID!): Tweet
        deleteTweet(id: ID!): Boolean
    }
`;

const resolvers = {
    Query: {
        allTweets: () => tweets,
        tweet: (_, {id}) => tweets.find(t => t.id === id),
        allUsers: () => users
    },
    Mutation: {
        postTweet(_, {text, userId}) {
            const newTweet = {
                id: tweets.length + 1,
                text,
            };
            tweets.push(newTweet);
            return newTweet;
        },
        deleteTweet(_, {id}) {
            const tweet = tweets.find(tweet => tweet.id === id);
            if (!tweet) return false;
            tweets = tweets.filter( t => t.id !== tweet.id );
            return true;
        }
    },
    User: {
        fullName: ({firstName, lastName}) => firstName + " " + lastName
    },
    Tweet: {
        author({userId}) {
            return users.find(u => u.id === userId)
        }
    }
}

const server = new ApolloServer({typeDefs, resolvers});

server.listen().then(({url}) => {
    console.log(`Running on ${url}`);
})
```

## database.js
``` javascript
const tweets = [{
    id: "1",
    text: "my first tweet",
    userId: "2"
}, {
    id: "2",
    text: "second tweet",
    userId: "1"
}];

const users = [{
    id: "1",
    firstName: "gildong",
    lastName: "hong"
}, {
    id: "2",
    firstName: "chulsoo",
    lastName: "kim"
}];

export const db = {
    tweets, users
};
```