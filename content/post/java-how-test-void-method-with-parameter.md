+++
title = "Java 单元测试如何 Mock 有参数的 void 方法"
date = "2017-04-23T21:51:20+08:00"
categories = [ "编程",]
tags = [ "code", "java", "test",]
toc = "true"
+++


测试中如果遇到被测试方法调用 void 方法，在 Mockito 中改如何处理？

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
        accSvc.addLinkedAccounts(user, accounts, banks);//accounts 被改动了如何 mock?
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
这里的 AccountSvc 只是提供了一个 void 方法处理了入参 accounts，虽然修改入参是被我所不齿的，但是有时改写这类方法挺麻烦的，特别如果方法修改了两个入参的话。
这种情况下如何测试 DepositSvc.dps 方法呢？mockito 的 `doAnswer`就是用于模拟 void 方法回调的。

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