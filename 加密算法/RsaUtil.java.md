```java
import javax.crypto.Cipher;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

/**
 * Rsa 非对称加密解密工具类
 */
public class RsaUtil {

    /**
     * 公钥加密
     *
     * @param plainText 带加密文本
     * @param pubKey    公钥
     */
    public static <R> R encrypt(String plainText, PublicKey pubKey, Production<R> production) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.ENCRYPT_MODE, pubKey);
        byte[] encrypted = cipher.doFinal(plainText.getBytes()); // 密文
        return production.apply(encrypted);
    }

    /**
     * 公钥加密
     *
     * @param pubKeyBase64 base64格式公钥
     */
    public static <R> R encrypt(String plainText, String pubKeyBase64, Production<R> production) throws Exception {
        // 公钥证书
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(decodeBase64ToByte(pubKeyBase64));
        PublicKey publicKey = KeyFactory.getInstance("RSA").generatePublic(keySpec);

        return encrypt(plainText, publicKey, production); // 加密
    }

    /**
     * 公钥加密
     *
     * @param pubKeyBase64 base64格式公钥
     */
    public static String encryptToBase64String(String plainText, String pubKeyBase64) throws Exception {
        // 公钥证书
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(decodeBase64ToByte(pubKeyBase64));
        PublicKey publicKey = KeyFactory.getInstance("RSA").generatePublic(keySpec);

        return encrypt(plainText, publicKey, RsaUtil::toBase64String); // 加密
    }

    /**
     * 私钥解密
     *
     * @param encryptedText 密文
     * @param priKey        私钥
     */
    public static <R> R decrypt(String encryptedText, PrivateKey priKey, Production<R> production) throws Exception {
        Cipher cipher = Cipher.getInstance("RSA");
        cipher.init(Cipher.DECRYPT_MODE, priKey);
        byte[] decrypted = cipher.doFinal(decodeBase64ToByte(encryptedText));
        return production.apply(decrypted);
    }

    public static String decryptToString(String encryptedText, PrivateKey priKey) throws Exception {
        return decrypt(encryptedText, priKey, String::new);
    }

    /**
     * 私钥解密
     *
     * @param priKeyBase64 base64格式私钥
     */
    public static <R> R decrypt(String encryptedText, String priKeyBase64, Production<R> production) throws Exception {
        // 私钥证书
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(decodeBase64ToByte(priKeyBase64));
        PrivateKey privateKey = KeyFactory.getInstance("RSA").generatePrivate(keySpec);

        return decrypt(encryptedText, privateKey, production);
    }

    public static String decryptToString(String encryptedText, String privateKeyBase64) throws Exception {
        return decrypt(encryptedText, privateKeyBase64, String::new);
    }

    public static String toBase64String(byte[] encoded) {
        return java.util.Base64.getEncoder().encodeToString(encoded);
    }

    public static byte[] decodeBase64ToByte(String decoded) {
        return java.util.Base64.getDecoder().decode(decoded);
    }

    @FunctionalInterface
    public interface Production<R> {
        /**
         * @return the function result
         */
        R apply(byte[] data);
    }

    /**
     * 生成密匙对
     *
     * @param keySize 密钥大小
     */
    public static KeyPair generateKeyPair(int keySize) throws NoSuchAlgorithmException {
        // 初始化一个RSA算法的密钥对生成器
        KeyPairGenerator keyGen = KeyPairGenerator.getInstance("RSA");
        keyGen.initialize(keySize);
        return keyGen.generateKeyPair();
    }

    public static Map<String, String> generateBase64KeyPair(int keySize) throws Exception {
        KeyPair keyPair = generateKeyPair(keySize);
        RSAPublicKey pubKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey priKey = (RSAPrivateKey) keyPair.getPrivate();
        Map<String, String> keyMap = new HashMap<>(2);
        keyMap.put("private", toBase64String(priKey.getEncoded()));
        keyMap.put("public", toBase64String(pubKey.getEncoded()));

        return keyMap;
    }
}
```