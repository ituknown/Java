```java
import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * HMAC(Hash-based Message Authentication Code) 是一种基于散列
 * 函数的消息认证码算法. 它结合了散列函数和一个密钥, 用于生成具有一定
 * 长度的固定大小的哈希值. <em>用于验证消息的完整性和认证消息发送者的身份</em>.
 */
public enum Hmac {
    HMAC_MD5("HmacMD5"),
    HMAC_SHA1("HmacSHA1"),
    HMAC_SHA256("HmacSHA256"),
    HMAC_SHA384("HmacSHA384"),
    HMAC_SHA512("HmacSHA512"),
    ;

    private final String algorithm;

    Hmac(String algorithm) {
        this.algorithm = algorithm;
    }

    public byte[] hmac(String secret, String plaintext) throws NoSuchAlgorithmException, InvalidKeyException {
        Mac sha256HMAC = Mac.getInstance(algorithm);
        SecretKeySpec keySpec = new SecretKeySpec(secret.getBytes(), algorithm);
        sha256HMAC.init(keySpec);
        return sha256HMAC.doFinal(plaintext.getBytes());
    }

    public static void main(String[] args) throws NoSuchAlgorithmException, InvalidKeyException {

        String secret = "f$sf1@9.";
        String plaintext = "helloworld";

        System.out.println(HexUtils.toHexString(HMAC_MD5.hmac(secret, plaintext)));
        System.out.println(HexUtils.toHexString(HMAC_SHA1.hmac(secret, plaintext)));
        System.out.println(HexUtils.toHexString(HMAC_SHA256.hmac(secret, plaintext)));
        System.out.println(HexUtils.toHexString(HMAC_SHA384.hmac(secret, plaintext)));
        System.out.println(HexUtils.toHexString(HMAC_SHA512.hmac(secret, plaintext)));
    }
}
```
