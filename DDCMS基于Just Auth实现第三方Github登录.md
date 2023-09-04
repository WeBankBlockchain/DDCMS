# DDCMS基于Just Auth实现Github第三方注册登录

> 作者: 张宇豪
>
> 学校: 深圳职业技术大学

## 1.Just Auth

官网:https://www.justauth.cn/

介绍: 开箱即用的整合第三方登录的开源组件

![image-20230905024924782](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050249863.png)



### 1.1 申请应用

根据这个官网的教程即可,我这里使用的是Github.

https://www.justauth.cn/guide/oauth/github/#_1-%E7%94%B3%E8%AF%B7%E5%BA%94%E7%94%A8



### 1.2 添加依赖

```groovy
implementation group: 'me.zhyd.oauth', name: 'JustAuth', version: '1.16.5'
```



### 1.3 添加Github的接口

这个API接口主要和就是为了授权Github,也可以类似的接入其他应用,一样的.

```java

/**
 * @author 张宇豪
 * @date 2023/9/3 23:53
 * @desc Github授权接口
 */

@RestController
@RequestMapping("/api/github")
public class GithubController {

    /**
     * 发送Github的授权请求
     * @param response 响应体
     * @throws IOException 抛出异常
     */
    @GetMapping("/render")
    public void renderAuth(HttpServletResponse response) throws IOException {
        AuthRequest authRequest = getAuthRequest();
        String authorize = authRequest.authorize(AuthStateUtils.createState());
        response.sendRedirect(authorize);
//        response.sendRedirect(authRequest.authorize(AuthStateUtils.createState()));
    }

    /**
     * 回调函数获取账户信息数据
     * @param callback 回调函数
     * @return 返回对象类型数据
     */
    @GetMapping("/callback")
    public Object login(AuthCallback callback) {
        AuthRequest authRequest = getAuthRequest();
        AuthResponse login = authRequest.login(callback);
        JSONObject jsonData = JSONUtil.parseObj(login.getData());
        // 获取Github的字段
        CommuserInfoEntity commuserInfoEntity = new CommuserInfoEntity();
        commuserInfoEntity.setType(1);
        commuserInfoEntity.setSource(jsonData.getStr("source"));
        commuserInfoEntity.setAvatar(jsonData.getStr("avatar"));
        commuserInfoEntity.setCommUsername(jsonData.getStr("username"));

        // 获取accessToken
        JSONObject jsonToken = JSONUtil.parseObj(jsonData.get("token"));
        commuserInfoEntity.setAccessToken(jsonToken.getStr("accessToken"));
        return commuserInfoEntity;
    }

    /**
     * 连接Github的个人应用信息
     * @return 返回结果
     */
    private AuthRequest getAuthRequest() {
        return new AuthGithubRequest(AuthConfig.builder()
                .clientId("3d90c8473462cfa79a01")
                .clientSecret("c7beff7b2690494b8b3f71cb7662318fe7c83")
                .redirectUri("http://localhost:10880/api/github/callback/")
                .scopes(AuthScopeUtils.getScopes(AuthGithubScope.values()))
                .httpConfig(HttpConfig.builder()
                        .timeout(15000)
                        .proxy(new Proxy(Proxy.Type.HTTP, new InetSocketAddress("127.0.0.1", 7890)))
                        .build())
                .build());
    }
}
```



## 2.设计思路

### 2.1 数据库的第三方账户表

添加一张新的数据库表单，这个表单的作用就是存储第三方账户的信息，比如注册的时候，就需要在这里使用第三方进行注册的操作，然后插入数据库表中，使用第三方进行登录的时候才可以判断是否已经注册，这里因为注册的时候需要绑定链上的账户地址以及私钥相关的信息，所有不得不创建一个新的表进行隔离开。

```sql
create table t_commuser_info
(
    pk_id         int auto_increment comment '主键'
        primary key,
    comm_username varchar(30)  not null comment '第三方账户用户名',
    type          int          not null comment '第三方类型（1、Gitee 2、Github）',
    avatar        varchar(255) not null comment '第三方账户头像',
    source        varchar(255) not null comment '第三方账户来源'
);
```

逆向模型分析：

![image-20230905020526899](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050205966.png)

注册登录需要用到这三张表：

- t_account_info
- t_company_info
- t_commuse_info(我自己添加的)

我加个表存储第三方账户，用于查询是否注册，并可以根据对应得绑定得账户，然后查询该账户信息，	最重要得是需要通过`springsecurity`的`UsernamePasswordAuthenticationToken`的认证。



### 2.2 前端的注册和登录

>  注册部分：

- 用户可以选择绑定第三方进行用户注册，假如同意协议并提交这个按钮下方有四个对应的图标，比如gitee、github、微信、qq。现在我们假设就是点击github图标，然后我们会调用`/api/github/render`的接口，其他的信息还是要填写的，然后我在github中进行授权管理，授权完成之后，我们会返回JSON的字段给前端。就是github的相关的用户信息。表单其他的信息填写完成之后，需要把该JSON的字段一起发送给后端的注册用户的接口即可。
- 保留了用户密码注册的方式+Github账号绑定，因为这里我考虑到了用户名和密码不能为null所以就没修改字段

![image-20230904040536778](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309040405119.png)

> 登录部分：

- 登录部分的思路与上面一样，在登录按钮下面放其他的第三方登录图标。点击Github登录，然后还是一样调用github的授权，然后这里的回调函数还是刚刚的那一个，我们会先去数据库中进行查询，如果该用户都没有注册绑定，那我们就提示用户该用户并没有注册，请先注册绑定第三方账号。否则直接放行。
- 我这里点击第三方登录按钮不需要显示这里的用户名密码，点完就是直接登录。

![image-20230904041045740](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309040410769.png)

我不会react的前端，只会Vue的前端，所以改代码比较难受，前端的部分就没有进行开发，我就负责实现完整的gitub第三方登录的后端实现的示例。

### 2.3 授权认证的注册流程

> 我使用的是泳道图，刚好分别区分一下三个操作的角色
>
> - 用户：用户默认就是注册，调用Github授权的接口拿到信息
> - Github/第三方：Github平台进行授权的时候，会有授权失败，那就返回，授权成功直接调用回调API拿取信息数据
> - 平台：DDCMS注册的时候根据如上的信息，进行绑定到区块链的账户地址以及私钥，最后再同步到数据库中。

![image-20230904234625536](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309042346580.png)

### 2.4 前端的注册字段

我这里就加了commJson,前端可以先调用Github授权,然后回调函数获取到当前的github账户个人信息,然后发送请求的时候携带上即可.

```js
{
  "accountType": "1",
  "password": "000000",
  "userName": "CompanyA",
  "detailJson": {
    "companyName": "测试企业A",
    "companyContact": "12345678910",
    "companyCertType": "busiID",
    "companyCertNo": "441622200305244851",
    "companyCertFileUri": "c83c1af0aacf4eefb8b474af05617393.jpg"
  },
  // 这里我加了如下的字段 用于绑定注册第三方账户的
  "commJson": {
      "commUsername": "CN-ZHANGYH",
      "type": "1",
      "avatar": "https://avatars.githubusercontent.com/u/84267606?v=4",
      "source": "GITHUB"
    }
}
```



### 2.5 授权认证的登录流程

大同小异，其实没什么去别的，我只是登录的时候使用的是Github授权 + SpringBoot Security的认证。因为如果是直接使用Github进行授权登录，那将会绕过SpringBoot Security的安全认证，那这个登录就没有那么安全，我这里的登录不需要用户名密码，就是你发送Github的授权之后，会调用我的回调函数，然后就可以直接拿着GitHub的个人信息进行登录校验。

> 可能这里我没有考虑accessToken的问题，但是可以用redis去解决过期时间，或者前端设置过期的时间为7200s。

![image-20230905021057514](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050210554.png)

### 2.6 前端的登录字段

这里我是直接额外写了一个接口,所以这个就是默认json的格式,如果使用第三方登录,可以在登录页面加上Github的图标即可,直接登录就行,过程就是拿到个人账户信息之后再发送一次请求进行登录校验.

```json
{
    "commUsername": "CN-ZHANGYH",
    "type": 1,
    "avatar": "https://avatars.githubusercontent.com/u/84267606?v=4",
    "source": "GITHUB",
    "accessToken": "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"
}
```



## 3.后端代码实现

### 3.1 Dao层代码

添加一个CommuserInfoEntity的实体类.

```java
@Data
@Accessors(chain = true)
public class CommuserInfoEntity {

    private Long pkId;
	
    // 第三方账户用户名
    private String commUsername;
	// 本地账户用户名
    private String accountUsername;
	// 第三方类型（1、Gitee 2、Github）
    private Integer type;
	// 第三方账户头像
    private String avatar;
	// 第三方账户来源
    private String source;
	// 第三方的AccessToken 
    private transient String accessToken;
}
```

然后添加对应的Mapper接口类.

```java
/**
 * @author 张宇豪
 * @date 2023/9/4 3:55
 * @desc 第三方登录的数据库操作接口
 */
public interface CommUserInfoMapper {

    /**
     * 新增第三方账户信息
     * @param commuserInfoEntity 第三方账户信息实体类
     * @return 返回结果
     */
    @Insert(
            "INSERT INTO t_commuser_info (comm_username,account_username,type, avatar, source) values(#{commUsername},#{accountUsername}, #{type}, #{avatar}, #{source})")
    @Options(useGeneratedKeys = true, keyProperty = "pkId", keyColumn = "pk_id")
    int insertCommUser(CommuserInfoEntity commuserInfoEntity);

    /**
     * 根据用户名查询第三方账号信息
     * @param commUsername 第三方用户名
     * @return 返回结果
     */
    @Select("SELECT * FROM t_commuser_info WHERE comm_username=#{commUsername}")
    @ResultType(CommuserInfoEntity.class)
    CommuserInfoEntity selectByUserName(@Param("commUsername") String commUsername);

}
```



### 3.2 Service层

主要添加了三个接口,分别是:

- `githubLogin` github登录的接口
- `selectByUsername` 根据用户查询第三方账号信息
- `insertCommUser` 新增第三方账户信息

```java

/**
 * @author 张宇豪
 * @date 2023/9/4 3:55
 * @desc Github第三方业务接口
 */
public interface CommUserService {

    /**
     * 通过Github第三方实现登录
     * @param commuserInfoEntity 第三方登录的信息
     * @return 返回结果
     */
    CommonResponse githubLogin(CommuserInfoEntity commuserInfoEntity);


    /**
     * 根据用户查询第三方账号信息
     * @param username 用户名
     * @return 返回结果
     */
    CommuserInfoEntity selectByUsername(String username);


    /**
     * 新增第三方账户信息
     * @param commuserInfoEntity 第三方账户信息
     * @return 返回结果
     */
    int insertCommUser(CommuserInfoEntity commuserInfoEntity);
}
```

然后实现CommUserService这个接口,下面是主要的一些业务逻辑:

1. `githubLogin`：根据传入的第三方登录信息 `commuserInfoEntity` ，首先查询该第三方账户是否已经注册，如果未注册则返回错误信息。如果该账户已经注册，则根据其绑定的 `AccountInfo` 查询账户信息并进行SpringBoot Security的权限校验。如果认证通过，则生成 token 并返回成功响应；否则返回错误信息。
2. `selectByUsername`：根据用户名查询第三方账户信息。
3. `insertCommUser`：插入一条第三方账户信息到数据库中。

```java
/**
 * @author 张宇豪
 * @date 2023/9/4 3:55
 * @desc 第三方账户登录的实现类
 */
@Service
public class CommUserServiceImpl implements CommUserService {

    @Autowired private CommUserInfoMapper commUserInfoMapper;

    @Autowired private AccountInfoMapper accountInfoMapper;

    @Autowired private AuthenticationManager authenticationManager;

    @Autowired private JwtTokenHandler tokenHandler;

    /**
     * 实现第三方登录的信息
     * @param commuserInfoEntity 第三方登录的信息
     * @return
     */
    @Override
    public CommonResponse githubLogin(CommuserInfoEntity commuserInfoEntity) {
        // 查询当前的第三方账户是否已经注册
        CommuserInfoEntity result = this.selectByUsername(commuserInfoEntity.getCommUsername());
        if (Objects.isNull(result))
        {
            return CommonResponse.error(CodeEnum.USER_NOT_EXISTS);
        }
        // 根据第三方的账户查询当前的AccountUser的信息
        AccountInfoEntity accountInfoEntity =
                accountInfoMapper.selectByUserName(result.getAccountUsername());
        if (Objects.isNull(accountInfoEntity))
        {
            // 用户未绑定第三方账户
            return CommonResponse.error(CodeEnum.USER_NOT_EXISTS);
        }
        // 判断当前第三方账户是否绑定成功
        if (!result.getAccountUsername().equals(accountInfoEntity.getUserName()))
        {
            return CommonResponse.error(CodeEnum.USER_NOT_EXISTS);
        }
        // 登录的权限校验
        if (accountInfoEntity.getAccountType() != AccountType.ADMIN.getRoleKey()
                && accountInfoEntity.getStatus() != AccountStatus.Approved.ordinal()) {
            return CommonResponse.error(CodeEnum.ACCOUNT_NOT_APPROVED);
        }
        // 这里使用的是SpringSecurity所以我保留了默认的 生成Token的时候我使用的是DID + Github的AccessToken
        try {
            // 使用auth进行用户认证
            UsernamePasswordAuthenticationToken authenticationToken =
                    new UsernamePasswordAuthenticationToken(accountInfoEntity.getUserName(), "0");
            // 调用UserDetailService实现类的认证方法
            Authentication authentication = authenticationManager.authenticate(authenticationToken);

            // 认证通过，则生成token，并返回
            LoginUserBO loginInfoBo = (LoginUserBO) authentication.getPrincipal();

            // 这里的Token组成使用DID + Github的AccessToken
            String token =
                    JwtTokenHandler.TOKEN_PREFIX
                            + tokenHandler.generateToken(loginInfoBo.getEntity().getDid() + commuserInfoEntity.getAccessToken());
            LoginResponse response = new LoginResponse();
            response.setToken(token);
            response.setAccountType(String.valueOf(accountInfoEntity.getAccountType()));
            return CommonResponse.success(response);
        } catch (AuthenticationException e) {
            return CommonResponse.error(CodeEnum.LOGIN_FAILED);
        }
    }

    /**
     * 根据用户名查询第三方账户信息
     * @param username 用户名
     * @return 返回结果
     */
    @Override
    public CommuserInfoEntity selectByUsername(String username) {
        return commUserInfoMapper.selectByUserName(username);
    }

    /**
     * 新增第三方账户信息
     * @param commuserInfoEntity 第三方账户信息
     * @return 返回结果
     */
    @Override
    public int insertCommUser(CommuserInfoEntity commuserInfoEntity) {
        return commUserInfoMapper.insertCommUser(commuserInfoEntity);
    }
}
```



> 下面这个是魔改之后的注册实现类

其实我这里做的并不多,就后面更新到了数据库中

```java
  @Transactional(rollbackFor = Exception.class)
  @Override
  public CommonResponse registerAccount(RegisterRequest request)
      throws TransactionException, JsonProcessingException {
    // Args
    int accountType = Integer.parseInt(request.getAccountType());
    if (accountType != AccountType.WITNESS.getRoleKey()
        && accountType != AccountType.COMPANY.getRoleKey()) {
      throw new DDCMSException(CodeEnum.ADMIN_NOT_ALLOWED);
    }
    // Generation private key
    CryptoSuite cryptoSuite = keyPairHandler.getCryptoSuite();
    CryptoKeyPair keyPair = null;
    if (!StringUtils.isEmpty(request.getHexPrivateKey())) {
      keyPair = cryptoSuite.loadKeyPair(request.getHexPrivateKey());
    } else {
      keyPair = cryptoSuite.generateRandomKeyPair();
    }
    // Save to blockchain
    AccountContract accountContract =
        AccountContract.load(sysConfig.getContractConfig().getAccountContract(), client, keyPair);
    TransactionReceipt txReceipt =
        accountContract.register(
            BigInteger.valueOf(Long.parseLong(request.getAccountType())),
            cryptoSuite.hash(request.getUserName().getBytes()));
    byte[] didBytes = accountContract.getRegisterOutput(txReceipt).getValue1();
    BlockchainUtils.ensureTransactionSuccess(txReceipt, txDecoder);

    AccountInfoEntity accountInfoEntity = new AccountInfoEntity();
    accountInfoEntity.setAccountType(Integer.parseInt(request.getAccountType()));
    accountInfoEntity.setDid(Base64.encode(didBytes));
    accountInfoEntity.setPassword(bCryptPasswordEncoder.encode(request.getPassword()));
    accountInfoEntity.setStatus(AccountStatus.Registered.ordinal());
    accountInfoEntity.setPrivateKey(keyPair.getHexPrivateKey());
    accountInfoEntity.setUserName(request.getUserName());
    if (accountInfoEntity.getAccountType() == AccountType.ADMIN.getRoleKey()) {
      if (accountInfoMapper.selectTheFirstOne() != null) {
        throw new DDCMSException(CodeEnum.ADMIN_NOT_ALLOWED);
      }
      accountInfoEntity.setStatus(AccountStatus.Approved.ordinal());
    }
    accountInfoMapper.insertAccount(accountInfoEntity);

    CompanyInfoEntity companyInfo =
        objectMapper.readValue(request.getDetailJson(), CompanyInfoEntity.class);
    companyInfo.setAccountId(accountInfoEntity.getPkId());
    companyInfoMapper.insertCompany(companyInfo);

    // 在这里完成对第三方用户的绑定操作
    CommuserInfoEntity commuserInfoEntity =
            objectMapper.readValue(request.getCommJson(), CommuserInfoEntity.class);
    commuserInfoEntity.setAccountUsername(accountInfoEntity.getUserName());
    if (Objects.isNull(commuserInfoEntity)) {
      // 如果这里是空说明用户没有选择第三方进行注册
      return CommonResponse.success();
    } else {
      if (Objects.isNull(commUserService.selectByUsername(commuserInfoEntity.getCommUsername()))) {
        // 如果不是空的说明用户选择了第三方进行注册绑定操作
        commUserService.insertCommUser(commuserInfoEntity);
        return CommonResponse.success();
      }
      return CommonResponse.error(400,"当前用户已经注册");
    }
  }
```

这里改了一下RegisterRequest.

```java
@Data
@EqualsAndHashCode(callSuper = true)
public class RegisterRequest extends CommonRequest {
  @NotBlank(message = "用户名不能为空.")
  private String userName;

  @NotBlank(message = "密码不能为空.")
  private String password;

  @Pattern(regexp = "[12]", message = "accountType must be 1 or 2")
  private String accountType;

  // Github的认证信息
  private String detailJson;

  @NotBlank(message = "第三方的信息不能为空")
  private String commJson;

  private String hexPrivateKey;
}

```



### 3.3 Controller层

#### POST GitHub注册

POST /api/account/register

> Body 请求参数

```json
{
  "accountType": "1",
  "password": "0",
  "userName": "companyA",
  "detailJson": "{\"companyName\":\"测试企业A\",\"companyContact\":\"13411553801\",\"companyCertType\":\"busiID\",\"companyCertNo\":\"8676867466565\",\"companyCertFileUri\":\"c83c1af0aacf4eefb8b474af05617393.jpg\"}",
  "commJson": "{\"commUsername\":\"CN-ZHANGYH\",\"type\":\"1\",\"avatar\":\"https://avatars.githubusercontent.com/u/84267606?v=4\",\"source\":\"GITHUB\"}"
}
```

##### 请求参数

| 名称          | 位置 | 类型   | 必选 | 说明 |
| ------------- | ---- | ------ | ---- | ---- |
| body          | body | object | 否   | none |
| » accountType | body | string | 是   | none |
| » password    | body | string | 是   | none |
| » userName    | body | string | 是   | none |
| » detailJson  | body | string | 是   | none |
| » commJson    | body | string | 是   | none |

> 返回示例

> 200 Response

```json
{
    "code": 0,
    "msg": "success",
    "debugMsg": null,
    "data": {
        "token": "Bearer eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiJBQUg2bXBJNzhITnNpUFNaUHhzbU92QTVVazZZelpXYngyTTJpRXVuc0xnPWdob19vbGJtcW9naVVHamhoOE5CTEpKc1FMelhjRFVwUUcwM0dta0wiLCJpYXQiOjE2OTM4NTI5NzgsImV4cCI6MTY5Mzg2MTYxOH0.u6PSSR41ltHdCnE-jM1EJtv_5KeBbP4nM4OaPVg7MB3TNMW5WQIsIXGn9bo_RwA5XTDs2LDsy0105AiuIrsh6g",
        "accountType": "1"
    }
}
```

##### 返回结果

| 状态码 | 状态码含义                                              | 说明 | 数据模型 |
| ------ | ------------------------------------------------------- | ---- | -------- |
| 200    | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | 成功 | Inline   |

##### API接口

```java
  @PostMapping("register")
  public CommonResponse register(@RequestBody @Valid RegisterRequest request) throws Exception {
    return accountService.registerAccount(request);
  }
```



#### POST GitHub登录

POST /api/account/githubLogin

> Body 请求参数

```json
{
  "commUsername": "CN-ZHANGYH",
  "type": 1,
  "avatar": "https://avatars.githubusercontent.com/u/84267606?v=4",
  "source": "GITHUB",
  "accessToken": "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"
}
```

##### 请求参数

| 名称           | 位置 | 类型    | 必选 | 说明 |
| -------------- | ---- | ------- | ---- | ---- |
| body           | body | object  | 否   | none |
| » commUsername | body | string  | 是   | none |
| » type         | body | integer | 是   | none |
| » avatar       | body | string  | 是   | none |
| » source       | body | string  | 是   | none |
| » accessToken  | body | string  | 是   | none |

> 返回示例

> 200 Response

```json
{
    "code": 0,
    "msg": "success",
    "debugMsg": null,
    "data": null
}
```

##### 返回结果

| 状态码 | 状态码含义                                              | 说明 | 数据模型 |
| ------ | ------------------------------------------------------- | ---- | -------- |
| 200    | [OK](https://tools.ietf.org/html/rfc7231#section-6.3.1) | 成功 | Inline   |

##### API接口

```java
  @PostMapping("githubLogin")
  public CommonResponse githubLogin(@RequestBody @Valid CommuserInfoEntity commuserInfoEntity) {
    return commUserService.githubLogin(commuserInfoEntity);
  }
```



## 4.测试注册登录

我的应用

![image-20230905025703119](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050257208.png)

访问http://localhost:10880/api/github/render, 跳转成功 进行授权

![image-20230905025904656](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050259731.png)

模拟前端拿到该字段数据,直接在发送一次注册请求绑定一下信息.

![image-20230905030009364](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050300414.png)

如下是基于API Fox的调用接口测试:
![image-20230905030140533](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050301592.png)

如下是登录的情况:

> 记得使用admin先去审核一下账户

![image-20230905030305438](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050303507.png)

> 使用账户密码登录:

![](https://blog-1304715799.cos.ap-nanjing.myqcloud.com/imgs/202309050304209.png)

