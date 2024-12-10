```java
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.binary.Hex;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;

/**
 * <p>HMAC(Hash-based Message Authentication Code)是一种使用哈希函数
 * 和秘密密钥来生成消息认证码的算法.</p>
 *
 * <p>HMAC通过将密钥混入哈希运算中,提供了一种更强大的消息认证方法,防范了一
 * 些针对普通哈希的攻击.</p>
 *
 * <p>用于验证消息的完整性和认证消息发送者的身份.</p>
 */
public enum Hmac {
    MD5("HmacMD5"),
    SHA1("HmacSHA1"),
    SHA256("HmacSHA256"),
    SHA384("HmacSHA384"),
    SHA512("HmacSHA512"),
    ;

    private final String algorithm;

    Hmac(String algorithm) {
        this.algorithm = algorithm;
    }

    public byte[] hmac(byte[] secret, byte[] plaintext) throws NoSuchAlgorithmException, InvalidKeyException {
        Mac mac = Mac.getInstance(this.name());
        SecretKeySpec keySpec = new SecretKeySpec(secret, algorithm);
        mac.init(keySpec);
        return mac.doFinal(plaintext);
    }

    public <R> R hmac(byte[] secret, byte[] plaintext, Production<R> production) throws NoSuchAlgorithmException, InvalidKeyException {
        Mac mac = Mac.getInstance(this.name());
        SecretKeySpec keySpec = new SecretKeySpec(secret, algorithm);
        mac.init(keySpec);
        return production.apply(mac.doFinal(plaintext));
    }

    public String hmacHex(String secret, String plaintext) throws NoSuchAlgorithmException, InvalidKeyException {
        return hmac(secret.getBytes(StandardCharsets.UTF_8), plaintext.getBytes(StandardCharsets.UTF_8), Hex::encodeHexString);
    }

    public String hmacBase64(String secret, String plaintext) throws NoSuchAlgorithmException, InvalidKeyException {
        return hmac(secret.getBytes(StandardCharsets.UTF_8), plaintext.getBytes(StandardCharsets.UTF_8), Base64::encodeBase64String);
    }

    @FunctionalInterface
    public interface Production<R> {
        /**
         * @return the function result
         */
        R apply(byte[] data);
    }

    public static void main(String[] args) throws NoSuchAlgorithmException, InvalidKeyException {

        String secret = "f$s1f@9.";
        String plaintext = "hello,world";

        System.out.println(SHA1.hmacHex(secret, plaintext));
        System.out.println(SHA1.hmacBase64(secret, plaintext));
    }
}
```
