# PHP开发规范
### 基本约定
1. 源文件
    - 代码使用```<?php```开头，忽略闭合标签```?>```
    - 文件格式必须是无```BOM``` ```UTF-8```格式
    - 一个文件只声明一种类型，如```class```和```interface```不能混写在一个源文件中
2. 缩进
    - 使用```4```个空格来缩进，IDE可以设置
3. 行长度
    - 每行```120```个字符
4. 关键字
    - 所有关键字均为小写，如```true```、```false```
5. 命名
    - 类名为大驼峰法，如```UserModel```
    - 类方法名为小驼峰法，如```getUserId()```
    - 函数使用```小写字母```加```_```组合，如```get_cookie()```
    - 变量名使用小驼峰法，如```$userId```
    - 常量定义为```大写字母```加```_```组合，如```IS_DEBUG```
6. 代码注释标签
    - 类文件中对类、方法、属性进行注释，使用```@param @return @throwns```
    - ```@param```注释写出详解，如```@param string $username 用户名```
7. 业务模块
    - 路由为```小写字母```加```_```组成，如```/api/get_user_info```
    - ```View```层负责数据展示
    - ```Controller```层负责输入参数校验，最外层捕捉异常，调用```Logic```和```View```视图层
    - ```Logic```层负责具体业务逻辑，调用```Model```层，返回处理数据
    - ```Model```层负责数据表查询和关联关系
    - 异常类需分清功能，如```ParamException```表示参数错误，```UserException```表示自定义异常
    - 异常需分类定义```code```，使用```PHP```类常量代替，如
    ```
    <?php
    namespace app\exceptions\codes;
    
    class UserExceptionCode extends BaseExceptionCode {
        const NO_AUTH              = 1000001;
        const NO_AUTH_MSG          = '不具有权限';
        const STATUS_EXCEPTION     = 1000002;
        const STATUS_EXCEPTION_MSG = '状态异常';        
    }
    ```
    - 数据表文件如有```Enum```类型，使用```PHP```类常量代替，如
    ```
    <?php
    namespace app\enums;
    
    class UserEnum extends BaseEnum {
        const STATUS_DELETED = -1;// 已删除
        const STATUS_DISABLE = 0;// 禁用
        const STATUS_ENABLE  = 1;// 正常

        const AUTH_GUEST         = 1;// 匿名用户
        const AUTH_GENERAL_ADMIN = 2;// 普通管理员
        const AUTH_SUPER_ADMIN   = 3;// 超级管理员
    }
    ```
    其中```STATUS```和```AUTH```为数据表映射字段名
    - ```Api```接口输出，示例
    ```
    {
        "code" : 0,
        "msg" : "success",
        "data" : {
            "userId" : 100
        }
    }
    ```
    其中```code```与```msg```为必填字段，```data```为空的情况下不填，示例
    ```
    {
        "code" : 100001,
        "msg" : "不具有权限"
    }
    ```
8. 其它
    - 数组，键为字符串时候使用单引号，只有一个键时候使用单行，示例
    ```
    $arr = [ 'userId' => 100 ];
    ```
    多个键时候使用多行，示例
    ```
    $arr = [
        'id'       => 100,
        'username' => 'admin',
    ];
    ```
    - 字符串使用单引号```'```