```java
import java.nio.charset.StandardCharsets;  
import java.security.*;  
import java.security.spec.InvalidKeySpecException;  
import java.security.spec.PKCS8EncodedKeySpec;  
import java.security.spec.X509EncodedKeySpec;  
import java.util.concurrent.Callable;  
import java.util.function.Function;  
  
/**  
 * <p>该类提供常见的 HashWithRSA 组合算法，用于提供数据签名和验签。</p>  
 * * <p>HashWithRSA 指的是使用 RSA 加密算法结合哈希函数进行数字签名或数据加密。  
 * RSA（Rivest–Shamir–Adleman）是一种非对称加密算法，而哈希函数则用于生成消息  
 * 摘要（即签名）。</p>  
 * * <p>加密和签名的区别:  
 * 加密和签名都是为了安全性考虑，但有所不同。加密是为了防止信息被泄露，签名是为了防止信息被篡改。</p>  
 * * <h3>加密过程：</h3>  
 * <ul> * <li>A生成一对密钥（公钥和私钥）。私钥不公开，A自己保留。公钥为公开的，任何人可以获取。</li>  
 * <li>A传递自己的公钥给B，B使用A的<em>公钥对消息进行加密</em>。</li>  
 * <li>A接收到B加密的消息，利用A自己的<em>私钥对消息进行解密</em></li>  
 * </ul> * <p>整个过程中，只用A的私钥才能对消息进行解密，防止消息被泄露。</p><br>  
 * * <h3>签名过程:</h3>  
 * <ul> * <li>A生成一对密钥(公钥和私钥)。私钥不公开，A自己保留。公钥为公开的，任何人可以获取。</li>  
 * <li>A用自己的私钥对消息进行加签，形成签名，并将签名和消息本身一起传递给B。</li>  
 * <li>B收到消息后，通过A的公钥进行验签。如果验签成功，则证明消息是A发送的。</li>  
 * </ul> * <p> * 整个过程，只有使用A私钥签名的消息才能被验签成功。即使知道了消息内容，也无法伪造签名，防止消息被篡改。  
 */  
public enum HashWithRSA {  
  
    MD5withRSA("MD5withRSA"),  
    SHA1withRSA("SHA1withRSA"),  
    SHA256withRSA("SHA256withRSA"),  
    SHA384withRSA("SHA384withRSA"),  
    SHA512withRSA("SHA512withRSA"),  
  
    ;  
  
    private final String algorithm;  
  
    HashWithRSA(String algorithm) {  
        this.algorithm = algorithm;  
    }  
  
    /**  
     * 私钥签名  
     *  
     * @param priKey    私钥  
     * @param plaintext 原文  
     */  
    public byte[] signature(byte[] priKey, byte[] plaintext) throws NoSuchAlgorithmException, InvalidKeySpecException, InvalidKeyException, SignatureException {  
        // 私钥证书  
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(priKey);  
        PrivateKey privateKey = KeyFactory.getInstance("RSA").generatePrivate(keySpec);  
  
        // 进行签名  
        Signature spi = Signature.getInstance(algorithm);  
        spi.initSign(privateKey);  
        spi.update(plaintext);  
  
        return spi.sign();  
    }  
  
    public <R> R signature(byte[] priKey, byte[] plaintext, Function<byte[], R> production) throws Exception {  
        return production.apply(signature(priKey, plaintext));  
    }  
  
    public byte[] signature(Callable<byte[]> priKey, Callable<byte[]> plaintext) throws Exception {  
        return signature(priKey.call(), plaintext.call());  
    }  
  
    public <R> R signature(Callable<byte[]> priKey, Callable<byte[]> plaintext, Function<byte[], R> production) throws Exception {  
        return production.apply(signature(priKey, plaintext));  
    }  
  
    public byte[] signature(String priKeyBase64, String plaintext) throws Exception {  
        return signature(decodeBase64ToByte(priKeyBase64), plaintext.getBytes(StandardCharsets.UTF_8));  
    }  
  
    public <R> R signature(String priKeyBase64, String plaintext, Function<byte[], R> production) throws Exception {  
        return production.apply(signature(priKeyBase64, plaintext));  
    }  
  
    /**  
     * 验签  
     *  
     * @param pubKey    公钥  
     * @param plaintext 原文  
     * @param signature 私钥签名{@link #signature(byte[], byte[])}  
     */    public boolean verify(byte[] pubKey, byte[] plaintext, byte[] signature) throws NoSuchAlgorithmException, InvalidKeySpecException, SignatureException, InvalidKeyException {  
  
        // 公钥证书  
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(pubKey);  
        PublicKey publicKey = KeyFactory.getInstance("RSA").generatePublic(keySpec);  
  
        Signature spi = Signature.getInstance(algorithm);  
        spi.initVerify(publicKey);  
        spi.update(plaintext);  
  
        // 验签  
        return spi.verify(signature);  
    }  
  
    public boolean verify(String pubKeyBase64, byte[] plaintext, byte[] signature) throws NoSuchAlgorithmException, InvalidKeySpecException, SignatureException, InvalidKeyException {  
        return verify(decodeBase64ToByte(pubKeyBase64), plaintext, signature);  
    }  
  
    public boolean verify(String pubKeyBase64, String plaintext, byte[] signature) throws NoSuchAlgorithmException, InvalidKeySpecException, SignatureException, InvalidKeyException {  
        return verify(decodeBase64ToByte(pubKeyBase64), plaintext.getBytes(StandardCharsets.UTF_8), signature);  
    }  
  
    public boolean verify(String pubKeyBase64, String plaintext, String signature) throws NoSuchAlgorithmException, InvalidKeySpecException, SignatureException, InvalidKeyException {  
        return verify(decodeBase64ToByte(pubKeyBase64), plaintext.getBytes(StandardCharsets.UTF_8), signature.getBytes(StandardCharsets.UTF_8));  
    }  
  
    public static String toBase64String(byte[] encoded) {  
        return java.util.Base64.getEncoder().encodeToString(encoded);  
    }  
  
    public static byte[] decodeBase64ToByte(String decoded) {  
        return java.util.Base64.getDecoder().decode(decoded);  
    }  
}
```
