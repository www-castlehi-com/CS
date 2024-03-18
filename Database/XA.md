# XA란?
- X/Open CAE에서 정의한 DTP(Distributed Transaction Processing)에 대한 규약
	> **분산 트랜잭션** : 두 개 이상의 DB, MSG, FileSystem 조합을 사용하는 것
- 트랜잭션 매니저, 리소스 매니저 사이의 계약
## 2Phase-Commit
![](https://i.imgur.com/dDs2hDj.png)
- 분산 환경에서 트랜잭션의 무결성을 보장하기 위해서 사용
- First Phase + Second Phase
### 1️⃣ First Phase
- 커밋 준비 단계
- 트랜잭션 커밋 준비가 완료되면 **Ready** 응답을, 준비가 안되었다면 **Fail** 응답을 트랜잭션 매니저에게 보냄
### 2️⃣ Second Phase
- 커밋, 롤백 단계
- 모든 노드에게 준비 완료 응답을 받을 경우 트랜잭션을 **Commit**하고, 하나라도 실패 응답을 받을 경우 **Rollback**함

```java
import javax.transaction.UserTransaction;
import javax.transaction.xa.XAResource;

public void execute2PC(UserTransaction utx, XAResource xaRes1, XAResource xaRes2) {
    try {
        utx.begin();
        
        // 두 리소스에 대한 XA 연산 준비
        int status1 = xaRes1.prepare(xaRes1.getXid());
        int status2 = xaRes2.prepare(xaRes2.getXid());

        // 모든 리소스가 준비되었으면 커밋
        if ((status1 == XAResource.XA_OK) && (status2 == XAResource.XA_OK)) {
            xaRes1.commit(xaRes1.getXid(), false);
            xaRes2.commit(xaRes2.getXid(), false);
        } else {
            xaRes1.rollback(xaRes1.getXid());
            xaRes2.rollback(xaRes2.getXid());
        }
        
        utx.commit();
    } catch (Exception e) {
        // 예외 처리 및 롤백
        utx.rollback();
    }
}
```