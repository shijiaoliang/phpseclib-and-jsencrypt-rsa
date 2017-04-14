# phpseclib-and-jsencrypt-rsa


## 用phpseclib 和 jsencrypt 实现前端js加密，服务端php解密 ##
***

#### php ###
```php
<?php

class rsa {
    //生成RSA公钥和私钥
    public static function rsaCreate() {
        Yii::$enableIncludePath = false;
        set_include_path(get_include_path() . PATH_SEPARATOR . APP_PATH . '/lib/phpseclib');
        include_once "Crypt/RSA.php";
        $rsa = new Crypt_RSA();

        //$rsa->setPrivateKeyFormat(CRYPT_RSA_PRIVATE_FORMAT_PKCS1);
        //$rsa->setPublicKeyFormat(CRYPT_RSA_PUBLIC_FORMAT_PKCS1);
        //define('CRYPT_RSA_EXPONENT', '65537');
        $res = $rsa->createKey(1024);

        //写入缓存
        RedisModels::setRsaPublickey($res['publickey']);
        RedisModels::setRsaPrivatekey($res['privatekey']);

        return $res;
    }

    //加密
    public static function encrypt($password) {
        $pubKey = RedisModels::getRsaPublickey();

        Yii::$enableIncludePath = false;
        set_include_path(get_include_path() . PATH_SEPARATOR . APP_PATH . '/lib/phpseclib');
        include_once "Crypt/RSA.php";
        $rsa = new Crypt_RSA();

        $rsa->loadKey($pubKey);
        $rsa->setEncryptionMode(CRYPT_RSA_ENCRYPTION_PKCS1);

        $password = $rsa->encrypt($password);
        $password = base64_encode($password);//二进制转base64

        return $password;
    }

    //解密
    public static function decrypt($encrypted) {
        $priKey = RedisModels::getRsaPrivatekey();

        Yii::$enableIncludePath = false;
        set_include_path(get_include_path() . PATH_SEPARATOR . APP_PATH . '/lib/phpseclib');
        include_once "Crypt/RSA.php";
        $rsa = new Crypt_RSA();

        $encrypted = bin2hex(base64_decode($encrypted));//二进制转base64 & base64转成16进制
        $encrypted = pack('H*', $encrypted);//16进制转成二进制
        $rsa->loadKey($priKey);
        $rsa->setEncryptionMode(CRYPT_RSA_ENCRYPTION_PKCS1);

        return $rsa->decrypt($encrypted);
    }
}
```



### js ###
```javascript
var options = {
	default_key_size : 1024,
	default_public_exponent : '010001',
};
var crypt = new JSEncrypt(options);
crypt.setPublicKey($('#publicKey').val());

var input = '123456';
console.log(crypt.encrypt(input));
```



***
- phpseclib: https://github.com/phpseclib/phpseclib
- jsencrypt: https://github.com/travist/jsencrypt


