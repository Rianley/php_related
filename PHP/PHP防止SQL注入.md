# PHP防止SQL注入
### SQL注入
SQL注入是通过web表单或其它方式来模拟请求等方式达到执行恶意SQL命令，示例
```
mysql_query("INSERT INTO `user` ( username ) VALUES ('" . $username . "')");
```
假如客户端输入的是
```
qiaoweizhen'); DROP TABLE user#
```
最终拼接而成的SQL是
```
INSERT INTO `user` (username) VALUES ('VALUE');DROP TABLE user;#')
```
执行之后将会把user表删除
### 防止SQL注入
1. 前端校验
    - 对参数类型校验，如只能填写数字的不要出现其它字符
2. 服务端校验
    1. 输入过滤
        - 通过PHP自带函数或自定义校验库对参数格式校验
        - 通过```addslashes()```等方法对特殊字符转义
        - 使用框架类库，如```Laravel```中的```Validate```
    2. SQL预处理
        - 通过```PDO/MySQLi```类库进行预处理，```PDO```示例
        ```
        <?php
        $username = $_POST['username'];
        $pdo      = new PDO('mysql:host=127.0.0.1;dbname=test;charset=utf8', 'root', 'root');
        $stmt     = $pdo->prepare('INSERT INTO `user`(username) VALUES (:username)');
        $stmt->execute(['username' => $username]);
        ```
        ```MySQLi```示例
        ```
        <?php
        $username = $_POST['username'];
        $conn     = new mysqli('127.0.0.1', 'root', 'root', 'test');
        $stmt     = $conn->prepare('INSERT INTO `user`(username) VALUES (:username)');
        $stmt->bind_param('s', $username);
        $stmt->execute();
        ```