[TOC]

# 测试

试试图片

![](../img/微信截图_20180414161945.png)


```java
@RestController
@RequestMapping(path = "/minialiapp/user")
@Api(description = "支付宝小程序用户相关业务接口")
public class MiniAliUserController extends BaseController {

    @Resource
    private MiniAliUserService miniAliAppService;

    @ApiOperation(value = "小程序用户授权", response = UserInfoPO.class, produces = "application/json")
    @ApiImplicitParPams({
            @ApiImplicitParam(name = "params", value = "{"
                    + "<br/>\"auth_code\": \"de49a8f1dcea4abc92666bdbb15aUX36\"<br/>}")})
    @RequestMapping(value = "/auth", method = RequestMethod.POST)
    public Map<String, Object> auth(@RequestHeader(name = "key", defaultValue = KEY) String key,
                                    @RequestHeader(name = "alipayVersion", defaultValue = VERSION) String alipayVersion,
                                    @RequestBody String params) throws Exception {
        JSONObject obj = ParmsUtils.fromObject(params);
        if (!ParmsUtils.compareToExist(obj, "auth_code")) {
            return ParmsUtils.formatData(new Head(MsgConstant.STR_201, MsgConstant.ERROR_MSG_100_1));
        }
        Map<String, Object> resultMap = new HashMap<>(8);
        ResultData result = miniAliAppService.auth(obj);
        resultMap.put("head", new Head(result.getCode(), result.getMsg()));
        resultMap.put("details", result.getObj());
        return resultMap;
    }

    @ApiOperation(value = "小程序登录", response = UserInfoPO.class, produces = "application/json", hidden = true)
    @RequestMapping(value = "/login", method = RequestMethod.POST)
    public Map<String, Object> login(@RequestBody String params) throws Exception {
        JSONObject obj = ParmsUtils.fromObject(params);
        if (!ParmsUtils.compareToExist(obj, "userToken") ||
                !ParmsUtils.compareToExist(obj, "uuid")) {
            return ParmsUtils.formatData(new Head(MsgConstant.STR_201, MsgConstant.ERROR_MSG_100_1));
        }
        Map<String, Object> resultMap = new HashMap<>(8);
        ResultData result = miniAliAppService.login(obj);
        resultMap.put("head", new Head(result.getCode(), result.getMsg()));
        resultMap.put("details", result.getObj());
        return resultMap;
    }

}
```

