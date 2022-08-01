---
title: 整合hutool进行加解密
date: 2022-08-01 13:48:41
tags: [java,工具类]
---

# 整合hutool进行加解密
<!--more-->
```

import cn.hutool.core.util.CharsetUtil;
import cn.hutool.core.util.StrUtil;
import cn.hutool.crypto.SecureUtil;
import cn.hutool.crypto.asymmetric.KeyType;
import cn.hutool.crypto.asymmetric.RSA;
import com.ls.cygnus.common.exception.BizException;
import com.ls.cygnus.common.exception.ResultCode;
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.lang3.ArrayUtils;

import javax.crypto.Cipher;
import java.nio.charset.StandardCharsets;
import java.security.*;
import java.security.interfaces.RSAPrivateKey;
import java.security.interfaces.RSAPublicKey;
import java.security.spec.PKCS8EncodedKeySpec;
import java.security.spec.X509EncodedKeySpec;

/**
 * 基于 RSA 算法的非对称加密工具类
 */
public class RsaUtils {
	static final String RSA = "RSA";

	// MAX_DECRYPT_BLOCK应等于密钥长度/8（1byte=8bit），所以当密钥位数为2048时，最大解密长度应为256.
	// 128 对应 1024，256对应2048
	private static final int KEYSIZE = 1024;

	// RSA最大加密明文大小
	private static final int MAX_ENCRYPT_BLOCK = 117;

	private static final int MAX_DECRYPT_BLOCK = KEYSIZE / 8;

	/**
	 * 公钥加密
	 *
	 * @return
	 */
	public static String publicKeyEncrypt(String pubkeyStr, String data) throws Exception {
		PublicKey publicKey = SecureUtil.generatePublicKey("RSA", cn.hutool.core.codec.Base64.decode(pubkeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPublicKey(publicKey);
		try {
			byte[] result = rsa.encrypt(StrUtil.bytes(data, CharsetUtil.CHARSET_UTF_8), KeyType.PublicKey);
			return	cn.hutool.core.codec.Base64.encode(result, StandardCharsets.UTF_8);
		} catch (Exception e) {
			throw new BizException(ResultCode.PUBLIC_KEY_ENCRYPT_ERROR);
		}
	}

	/**
	 * 私钥解密
	 *
	 * @return
	 */
	public static String privateKeyDecrypt(String prikeyStr, String data) throws Exception {
		PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", cn.hutool.core.codec.Base64.decode(prikeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPrivateKey(privateKey);
		try {
			byte[] result = rsa.decrypt(cn.hutool.core.codec.Base64.decode(data, StandardCharsets.UTF_8), KeyType.PrivateKey);
			return	StrUtil.str(result, CharsetUtil.CHARSET_UTF_8);
		} catch (Exception e) {
			throw new BizException(ResultCode.PRIVATE_KEY_DECRYPT_ERROR);
		}
	}


	/**
	 * 公钥加密  分段加密
	 *
	 * @return
	 */
	public static String publicKeyEncryptByDataLong(String pubkeyStr, String data) throws Exception {
		PublicKey publicKey = SecureUtil.generatePublicKey("RSA", cn.hutool.core.codec.Base64.decode(pubkeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPublicKey(publicKey);
		try {
			Cipher cipher = Cipher.getInstance(RSA);

			byte[] bytes = data.getBytes("UTF-8");
			cipher.init(Cipher.ENCRYPT_MODE, publicKey);
			// 加密时超过117字节就报错。为此采用分段加密的办法来加密
			byte[] enBytes = null;
			for (int i = 0; i < bytes.length; i += MAX_ENCRYPT_BLOCK) {
				// 注意要使用2的倍数，否则会出现加密后的内容再解密时为乱码
//				byte[] doFinal = cipher.doFinal(ArrayUtils.subarray(bytes, i, i + MAX_ENCRYPT_BLOCK));
				byte[] doFinal = rsa.encrypt(ArrayUtils.subarray(bytes, i, i + MAX_ENCRYPT_BLOCK), KeyType.PublicKey);

				enBytes = ArrayUtils.addAll(enBytes, doFinal);
			}
			return cn.hutool.core.codec.Base64.encode(enBytes, StandardCharsets.UTF_8);
		} catch (Exception e) {
			throw new BizException(ResultCode.PUBLIC_KEY_ENCRYPT_ERROR);
		}
	}

	/**
	 * 私钥解密 分段解密
	 *
	 * @return
	 */
	public static String privateKeyDecryptByDataLong(String prikeyStr, String data) throws Exception {
		PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", cn.hutool.core.codec.Base64.decode(prikeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPrivateKey(privateKey);
		try {

			byte[] decode = cn.hutool.core.codec.Base64.decode(data, StandardCharsets.UTF_8);
			Cipher cipher = Cipher.getInstance(RSA);
			cipher.init(Cipher.DECRYPT_MODE, privateKey);
			StringBuilder sb = new StringBuilder();
			for (int i = 0; i < decode.length; i += MAX_DECRYPT_BLOCK) {
//				byte[] doFinal = cipher.doFinal(ArrayUtils.subarray(decode, i, i + MAX_DECRYPT_BLOCK));
				byte[] doFinal = rsa.decrypt(ArrayUtils.subarray(decode, i, i + MAX_DECRYPT_BLOCK), KeyType.PrivateKey);
				sb.append(new String(doFinal, StandardCharsets.UTF_8));
			}
			return sb.toString();
		} catch (Exception e) {
			throw new BizException(ResultCode.PRIVATE_KEY_DECRYPT_ERROR);
		}
	}

	/**
	 * 私钥加密
	 *
	 * @return
	 */
	public static String privateKeyEncrypt(String prikeyStr, String data) throws Exception {
		PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", cn.hutool.core.codec.Base64.decode(prikeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPrivateKey(privateKey);
		try {
			byte[] result = rsa.encrypt(data.getBytes(StandardCharsets.UTF_8), KeyType.PrivateKey);
			String resData = cn.hutool.core.codec.Base64.encode(result, StandardCharsets.UTF_8);
			return resData;
		} catch (Exception e) {
			throw new BizException(ResultCode.PRIVATE_KEY_ENCRYPT_ERROR);
		}
	}


	/**
	 * 公钥解密
	 *
	 * @return
	 */
	public static String publicKeyDecrypt(String pubkeyStr, String data) throws Exception {
		PublicKey publicKey = SecureUtil.generatePublicKey("RSA", cn.hutool.core.codec.Base64.decode(pubkeyStr));
		cn.hutool.crypto.asymmetric.RSA rsa = new RSA();
		rsa.setPublicKey(publicKey);
		try {
			byte[] result = rsa.decrypt(cn.hutool.core.codec.Base64.decode(data, StandardCharsets.UTF_8), KeyType.PublicKey);
			return	StrUtil.str(result, CharsetUtil.CHARSET_UTF_8);
		} catch (Exception e) {
			throw new BizException(ResultCode.PUBLIC_KEY_DECRYPT_ERROR);
		}
	}


	public static void main(String[] args) throws Exception {
		//// 1.私钥加密、公钥解密——加密
		String pubkeyStr = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDickKr0faYmMK61Fc+zMrh+Of0XoUL2Dt2ohIyduIbyZCtWgR+MsUUl5fVhX77F94Y2LDVxTZJYkiZ/D27DV6D2AKsnBKX+LndlKstqOG7VFMU4rOmRjo8S323+vaP4WILoezjT5+0MpCDS5VWYUZmgI4L6uBQC0qYOF68CojkyQIDAQAB";
		//////// 自定义非对称私钥
		String prikeyStr = "MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAOJyQqvR9piYwrrUVz7MyuH45/RehQvYO3aiEjJ24hvJkK1aBH4yxRSXl9WFfvsX3hjYsNXFNkliSJn8PbsNXoPYAqycEpf4ud2Uqy2o4btUUxTis6ZGOjxLfbf69o/hYguh7ONPn7QykINLlVZhRmaAjgvq4FALSpg4XrwKiOTJAgMBAAECgYBAft6yZWjg6ZF8+QGoZ1fZqLUYCtvGFd5J2btpGCcqVuyYEy14bClpxgt+yzjxd0jQttcW68acfBvFj+xdHF+wkGpSTxH410uKUVvXn7BaHZhExDp0cBd6jD2DSbuqZyHvyvl2QvqfdNDKT8tJC/fQDPAS6asgxELV8zUGTQPIqQJBAPODwxVf2L3S+iW97H+R7ckIsM2oNi9aDg9cedQbJUY1af5axxTPIt30YOwvmlXE/64sbK4xwdVNjgSNF8EBrIMCQQDuDnfy4GQXfDFZLEYv+jmzFenLYSxpxNocv+balwFZF4Z/pfPK49g+XHRvELEARmaCgTkQxD8OvgD8ScMzJP/DAkEA2yxap6BOyftcHiAk/mTvqiNiTpf5vQDG6tiG5ntQPzLQJZi62mXcsfzER5BIzq2ymqdtYhNyrHNTQZFkMdk51QJBAIWXpQStnD35ug/a4sCF4d94Sq2RqMTqbaR4pOrCl0USCK6VyMxxNKc6ZzT03v/SgjB2qDmah/CT/CWYl2yaNNUCQC5xHvokv5tP62v3x72mn3gfY0Os2RTJfMoDVseitbOLYGg4e55X24G2b3t/CJvNAIpd4ZlApusMR7DAY6w8gCw=";

		PublicKey publicKey = SecureUtil.generatePublicKey("RSA", cn.hutool.core.codec.Base64.decode(pubkeyStr));
		PrivateKey privateKey = SecureUtil.generatePrivateKey("RSA", cn.hutool.core.codec.Base64.decode(prikeyStr));

		String data = "{小明的输出}";

//		//公钥加密
//		String s1 = publicKeyEncrypt(pubkeyStr, data);
//		System.out.println("公钥加密:" + s1);
//		String s6 = privateKeyDecrypt(prikeyStr, s1);
//		System.out.println("私钥解密:" + s6);
//
//
//		String s2 = privateKeyEncrypt(prikeyStr, data);
//		System.out.println("私钥加密:" + s2);
//		String s4 = publicKeyDecrypt(pubkeyStr, s2);
//		System.out.println("公钥解密:" + s4);

//
//		//公钥加密 分段加密
		String f1 = publicKeyEncryptByDataLong(pubkeyStr, data);
		System.out.println("公钥加密 分段加密:" + f1);
		String f2 = privateKeyDecryptByDataLong(prikeyStr, f1);
		System.out.println("私钥解密 分段解密:" + f2);


	}


}


```