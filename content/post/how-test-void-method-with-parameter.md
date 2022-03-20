---
title: "单元测试如何Mock有参数的void方法"
date: "2017-04-23T21:51:20+08:00"
toc: true
categories:
 - "编程"
tags:
 - code
 - java
 - test
---

测试中如果遇到被测试方法调用void方法，在Mockito中改如何处理?

<!--more-->

假设有如下的服务依赖：

```java
@Service
class DepositSvc {
    @Autowired
    private AccountSvc accSvc;

    public List<Account> dps(String user) {
        List<Account> accounts = new ArrayList();
        List<Account> banks = getBanks();
        accSvc.addLinkedAccounts(user, accounts, banks);//accounts被改动了如何mock?
        return accounts;
    }

}

@Service
class AccountSvc {

    @Autowired
    private RestClient restClient;

    public void addLinkedAccounts(String user, List<Account> accounts, List<Account> banks) {
        acc = restClient.getAcc(user);
        accounts.add(acc);
    }

}
```
这里的AccountSvc只是提供了一个void方法处理了入参accounts,虽然修改入参是被我所不齿的,但是有时改写这类方法挺麻烦的,特别如果方法修改了两个入参的话.
这种情况下如何测试DepositSvc.dps方法呢? mockito的 `doAnswer`就是用于模拟void方法回调的.

```java
class DepositSvcTest {

    @InjectMocks
    private DepositSvc depositSvc;

    @Mock
    private AccountSvc accountSvc;

    void test_dps() {

        // ... arrange

        // mock void method with arguments
        doAnswer((invocation) -> {
            Object[] args = invocation.getArguments();
            List<Account> accounts = (List<Account>) args[1]; //这里可以拿到入参
            accounts.add(new Account(911); //修改入参
            return null;
        }).when(accountSvc).addLinkedAccounts(any(), anyList(), anyList());

        // ...assert
    }
}
```