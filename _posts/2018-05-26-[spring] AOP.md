---
layout: post
title: "[spring] Aop"
categories: Development
excerpt_separator: "<!--more-->"
---


# AOP

### 메소드 추출

- 트랜잭션 경계설정과 비즈니스 로직이 공존하는 메소드
```java
public void upgradeLevels() throw Exception{
    //트랜잭션 경계시작
    TransactionStatus status = this.transactionManager
        .getTransaction(new DefaultTransactionDefinition());

    try{
        //비즈니스 로직
        List<User> user = userDao.getAll();
        for (User user : user){
            if(canUpgradeLevel(user)){
                upgradeLevel(user);
            }
        }

        //트랜잭션 끝
        this.transactionManager.commit(status);
    }
    catch (Exception e){
        this.transactionMangaer.rollback(status);
        throw e;
    }
}

```

현재는 하나의 메소드에 트랜잭션 기능과 비즈니스 로직이 같이 있다. 위 비즈니스 로직을 따로 메소드로 빼면 비즈니스 로직과 트랜잭션이 분리가 된다. 하지만 트랜잭션과 비즈니스 로직이 여전히 같이 존재한다. 이를 인터페이스를 DI로 더욱 깔끔한 코드를 만들 수 있다.

interface UserService 아래에 비즈니스 로직을 담당하는 UserServiceImpl과 트랜잭션을 담당하는 UserServiceTx 구현체를 통해 메소드 분리가 가능하다.
- 인터페이스
```java
public interface UserService{
    void add(User user);
    void upgradeLevels();
}
```
- 비즈니스 로직
```java
public class UserServiceImpl implements UserService{
    @Autowired
    UserDao userDao;
    @Autowired
    MailSender mailSender;

    public void upgradeLevels(){
    List<User> user = userDao.getAll();
        for (User user : user){
            if(canUpgradeLevel(user)){
                upgradeLevel(user);
            }
        }
    }
}
```
-트랜잭션
```java
public class UserServiceTx implements UserService{
    @Autowired @Qualifier("UserServiceImpl")
    UserService userService;
    PlatformTransactionManager transactionManager;

    public void add(User user){
        this.userService.add(User);
    }

    try{
        
        userService.upgradeLevels();

        this.transactionManager.commit(status);
    }
    catch (Exception e){
        this.transactionMangaer.rollback(status);
        throw e;
    }
}
```

트랜잭션에 비즈니스 로직을 주입함으로 트랜잭션 기능이 성공적으로 수행되는 형태의 메소드 추출이 가능해졌다.

여기서 핵심 기능은 비즈니스 로직인 UserServiceImpl이고 부가 기능은 트랜잭션인 UserServiceTx 이다. 이렇게 구현된 구조에서 비즈니스로직을 사용자가 구현할 때의 핵심은 구현자는 핵심 기능 구현에만 집중하고, 부가기능은 신경을 쓰지 않게 하는것이다. 하지만 사용자가 핵심 기능을 직접 선언하여 사용하면 부가기능이 끼어들 틈이 없게 된다. 따라서 부가기능을 사용하는지도 모르게 UserService를 사용하게 만드는 것이 포인트이다. 인터페이스를 통해 접근하게 하면 위의 문제점을 해소할 수 있다.


