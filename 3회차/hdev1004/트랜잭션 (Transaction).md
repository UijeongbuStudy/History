# 트랜잭션 (Transaction)

**📘 트랜잭션이란?**

---

데이터베이스의 **상태**를 변화시키기 위해 수행하는 작업의 단위

**상태**를 변화한다는건?

⇒ SQL 질의어를 이용하여 데이터베이스를 접근한다는것을 의미한다.

- Select
- Insert
- Delete
- Update

작업의 단위는 하나의 쿼리가 아닌 사용자가 지정하는 것.

**기능에 따른 단위의 묶음**을 트랜잭션이라고 한다.

📘 **중요 기능 : Commit, Rollback**

---

1. Commit
Commit이란 하나의 트랜잭션이 성공적으로 끝 마쳤고, 
데이터베이스가 일관이 있는 상태가 되었을 때, 
트랜잭션의 **마침을 알려주기 위해 사용하는 연산** 

1. Rollback
Rollback이란 하나의 트랜잭션 처리가 비정상적으로 종료되어 트랜잭션의 원자성이 깨지는 경우, 
**트랜잭션을 처음부터 다시 시작**하거나, **부분적으로만 취소** 시킨다.

📘 **DB 격리 수준**

---

[[MySQL] 트랜잭션의 격리 수준(Isolation Level)에 대해 쉽고 완벽하게 이해하기](https://mangkyu.tistory.com/299)

DB 격리 수준에는 4가지가 존재한다. 

 (Serializable, Repeatable Read, Read Commited, Read Uncommited)

1. Serializable (직렬화 가능)
가장 엄격한 수준이며, 동시에 여러 트랜잭션에서 데이터를 수정하려고 할 때 완전한 격리를 제공한다
트랜잭션이 순차적으로 처리되어야 하므로 동시 처리 성능이 매우 떨어진다.

2. Repeatable Read (반복 가능한 읽기)
일반적인 RDBMS는 변경 전의 레코드를 Undo 공간에 백업해둔다.
그러면 변경 전/후 데이터가 모두 존재하므로, 동일한 레코드에 대해 여러 버전이 존재한다고 하여 이를 **MVCC**(Multi-Version Concurrency Control, 다중 버전 도시성 제어)라고 부른다. 

한 트랜잭션 내에서 동일한 결과를 보이지만, 새로운 레코드가 추가되는 경우에 **부정합이** 발생할 수 있다
하지만, 웬만한 상황에서는 발생하지 않는다!

![Untitled](%E1%84%90%E1%85%B3%E1%84%85%E1%85%A2%E1%86%AB%E1%84%8C%E1%85%A2%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20(Transaction)%2033a47411630f4ae7a92c495d4449409f/Untitled.png)

커밋이 되었어도, Undo 로그를 참고하여 같은 결과를 반환하도록 한다.

유령 읽기(Phantom Read) : SELECT로 조회한 경우 트랜잭션이 끝나기 전에 다른 트랜잭션에 의해 추가된 레코드가 발견 되는 현상
일반적인 조회의 경우 MVCC로 유령 읽기가 존재하지 않는다.

1. Read Commited (확인 읽기)
커밋 여부에 따라 트랜잭션에 대한 데이터가 다르다.
**트랜잭션이 커밋된 데이터만 읽을 수 있으므로**, 더티 리드 문제를 방지한다.
하지만, Phantom Read(유령 읽기)와 Non-Repeatable Read(반복 읽기 불가능) 문제가 발생한다.

커밋된 데이터만 읽을수는 있지만, 트랜잭션 B가 실행되고 있는 중에는 계속 값이 변경될 수 있어서, Phantom Read와 Non-Repeatable Read(반복 읽기 불가능) 문제가 발생한다.

![Untitled](%E1%84%90%E1%85%B3%E1%84%85%E1%85%A2%E1%86%AB%E1%84%8C%E1%85%A2%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20(Transaction)%2033a47411630f4ae7a92c495d4449409f/Untitled%201.png)

1. Read Uncommitted
커밋하지 않은 데이터 조차도 접근할 수 있는 격리 수준이다.
서로다른 트랜잭션이 실행중이어도 접근이 가능하다.
데이터가 조회되었다가 사라지는 더티 리드(Dirty Read) 현상이 발생하게 되어 시스템에 상당한 혼란을 주게 된다

![Untitled](%E1%84%90%E1%85%B3%E1%84%85%E1%85%A2%E1%86%AB%E1%84%8C%E1%85%A2%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20(Transaction)%2033a47411630f4ae7a92c495d4449409f/Untitled%202.png)

다음을 요약하여 표로 나타내면 다음과 같다.

Repeatable Read의 격리 수준정도가 DB 성능이슈에도 제일 좋다.

|  | Dirty Read | Non-Repeatable Read | Phantom Read |
| --- | --- | --- | --- |
| Read Uncommited | 발생 | 발생 | 발생 |
| Read Committed | 없음 | 발생 | 발생 |
| Repeatable Read | 없음 | 없음 | 발생(MySQL은 거의 없음) |
| Serializable | 없음 | 없음 | 없음 |

📘 **DB 격리 수준 확인**

---

MySQL 트랜잭션 격리 수준 확인

```sql
SELECT @@GLOBAL.transaction_isolation;
```

![Untitled](%E1%84%90%E1%85%B3%E1%84%85%E1%85%A2%E1%86%AB%E1%84%8C%E1%85%A2%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20(Transaction)%2033a47411630f4ae7a92c495d4449409f/Untitled%203.png)

MySQL 트랜잭션 격리 수준 변경

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

📘 **용어 설명**

---

1. Race Condition (경쟁 상태)
둘 이상의 **트랜잭션이 동시에 같은 데이터를 수정**하려고 시도할 때 발생한다.
이로 인해 데이터의 일관성이 깨질 수 있다.
이럴 경우 고립성을 제공하여 문제를 방지한다.

격리 수준이 **Read Committed** 이하일 때 나타나는 현상

2. Dirty Read (더티 리드)
하나의 트랜잭션에서 작업이 완료 되지 않았는데도 다른 트랜잭션에서 읽을 수 있게 되는 현상

격리 수준이 **Read UnCommited** 이하일 때 나타나는 현상

3. Non-Repeatable Read (반복 가능한 읽기)
한 트랜잭션 내에서 반복 읽기를 수행하면 다른 트랜잭션의 커밋 여부에 따라 조회 결과가 달라지는 현상

격리 수준이 **Read Commited** 이하일 때 나타나는 현상

4. Lost Update (유실된 갱신)
여러 트랜잭션들이 **동시에 실행될 때 예상했던 결과와는 다른 결과가 생기는 현상**
    
    ![Untitled](%E1%84%90%E1%85%B3%E1%84%85%E1%85%A2%E1%86%AB%E1%84%8C%E1%85%A2%E1%86%A8%E1%84%89%E1%85%A7%E1%86%AB%20(Transaction)%2033a47411630f4ae7a92c495d4449409f/Untitled%204.png)
    
    격리 수준이 **Repeatable Read** 이하일 때 나타나는 현상
    
5. Phantom Read (파생 무결성)
하나의 트랜잭션에서 두번의 동일한 조회를 하였을 떄, 다른 트랜잭션의 Insert로 인해 없던 데이터가 조회되는 현상

격리 수준이 **Repetable Read** 이하일 때 나타나는 현상