```java
import java.nio.charset.StandardCharsets;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.InvalidKeySpecException;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;
import java.util.HashMap;
import java.util.Map;

/**
 * 使用SHAx算法对数据进行哈希,然后使用RSA算法对哈希值进行数字签名.
 *
 * <p>用于确保消息的完整性和验证消息发送者的身份.
 */
public enum SHAwithRSA {

    SHA1withRSA("SHA1withRSA"),
    SHA256withRSA("SHA256withRSA"),
    SHA384withRSA("SHA384withRSA"),
    SHA512withRSA("SHA512withRSA"),

    ;

    private final String algorithm;

    SHAwithRSA(String algorithm) {
        this.algorithm = algorithm;
    }

    public byte[] signature(byte[] rsaPri, String plaintext) throws NoSuchAlgorithmException,
            InvalidKeySpecException, InvalidKeyException, SignatureException {

        // 私钥证书
        PKCS8EncodedKeySpec keySpec = new PKCS8EncodedKeySpec(rsaPri);
        PrivateKey privateKey = KeyFactory.getInstance("RSA").generatePrivate(keySpec);

        // 进行签名
        Signature spi = Signature.getInstance(algorithm);
        spi.initSign(privateKey);
        spi.update(plaintext.getBytes(StandardCharsets.UTF_8));

        return spi.sign();
    }

    public boolean verify(byte[] rsaPub, String plaintext, byte[] signature) throws NoSuchAlgorithmException, InvalidKeySpecException, SignatureException, InvalidKeyException {

        // 公钥证书
        X509EncodedKeySpec keySpec = new X509EncodedKeySpec(rsaPub);
        PublicKey publicKey = KeyFactory.getInstance("RSA").generatePublic(keySpec);

        Signature spi = Signature.getInstance(algorithm);
        spi.initVerify(publicKey);
        spi.update(plaintext.getBytes(StandardCharsets.UTF_8));

        // 验签
        return spi.verify(signature);
    }

    public static Map<String, String> genKeyPair(int keysize) throws Exception {

        KeyPairGenerator keyPairGen = KeyPairGenerator.getInstance("RSA");
        keyPairGen.initialize(keysize);
        KeyPair keyPair = keyPairGen.generateKeyPair();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();

        Map<String, String> keyMap = new HashMap<>(2);
        keyMap.put("private", encodeToString(privateKey.getEncoded()));
        keyMap.put("public", encodeToString(publicKey.getEncoded()));

        return keyMap;
    }

    private static String encodeToString(byte[] encoded) {
        return java.util.Base64.getEncoder().encodeToString(encoded);
    }

    private static byte[] decodeToByte(String decoded) {
        return java.util.Base64.getDecoder().decode(decoded);
    }

    public static void main(String[] args) throws Exception {

        Map<String, String> keyPair = genKeyPair(2048);

        String plaintext = "helloworld";

        byte[] signature = SHAwithRSA.SHA1withRSA.signature(decodeToByte(keyPair.get("private")), plaintext);
        System.out.println(HexUtils.toHexString(signature));

        boolean verify = SHAwithRSA.SHA1withRSA.verify(decodeToByte(keyPair.get("public")), plaintext, signature);
        System.out.println(verify);

    }
}
```
