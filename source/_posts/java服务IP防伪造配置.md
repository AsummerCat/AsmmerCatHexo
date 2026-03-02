---
title: java服务IP防伪造配置
date: 2026-03-02 11:11:41
tags: [java,javaUtil]
---


### 1.Java配置文件:

```
app:
  security:
    ip:
      trusted-proxies:
        - "192.168.99.183/32"   # 测试环境物理机
        - "172.16.0.0/12"      # docker本地环境集群
        - "172.19.0.0/12"     # 测试环境docker集群
        - "10.0.0.0/8"      # Pod 网段# Docker 默认网段范围# 替换为你的 Nginx 或网关 IP
#        - "nginx"   # Docker Compose 服务名
#        - 'nginx-prod'
#        - 'nginx-dev'
#        - 'www'
#        - 'frontend'

```
<!--more-->

注意: 如果用hostname的话 没有会一直提示host为空,需要针对性开启

###  2.nginx配置需要打开转发请求头
```
        location ^~ /xxx-files/ {
            resolver 8.8.8.8 valid=30s;
            proxy_pass http://192.168.1.服务:9000//;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            # proxy_cache_valid 200 7d;
            # add_header Cache-Control "public, max-age=604800";
        }
```
### 3.使用配置文件接受配置

```

package com.xx.main.common.config;

import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

@Data
@Component
@ConfigurationProperties(prefix = "app.security.ip")
public class IpSecurityProperties {
    /**
     * 可信代理 IP 或 CIDR 列表（仅这些来源允许使用 X-Forwarded-For）
     * 示例：
     *   - 192.168.1.10      # Nginx 服务器
     *   - 10.0.0.0/8        # 内网网关段（谨慎使用）
     *   - 203.0.113.100     # 公网 SLB
     */
    private List<String> trustedProxies = new ArrayList<>();
}
```

#### 4.编写ip校验工具类
```
    package com.xx.main.common.util;

    import cn.hutool.core.lang.Validator;
    import cn.hutool.core.net.NetUtil;
    import cn.hutool.core.text.CharSequenceUtil;
    import com.common.exception.IbmsException;
    import com.main.common.config.IpSecurityProperties;
    import jakarta.servlet.http.HttpServletRequest;
    import lombok.RequiredArgsConstructor;
    import lombok.extern.slf4j.Slf4j;
    import org.springframework.stereotype.Component;
    import org.springframework.util.StringUtils;

    import java.net.Inet6Address;
    import java.net.InetAddress;
    import java.net.UnknownHostException;
    import java.util.Arrays;
    import java.util.List;

    /**
     * IP 工具类：获取真实客户端 IP + 防伪造 + 可信代理校验
     */
    @Slf4j
    @Component
    @RequiredArgsConstructor
    public class IpUtil {

        private static final List<String> FORBIDDEN_IPS = Arrays.asList(
                "0.0.0.0", "0:0:0:0:0:0:0:0", "::", "unknown", "-", "", "null"
        );

        private final IpSecurityProperties ipSecurityProperties;

        /**
         * 获取客户端真实 IP（防伪造）
         */
        public String getClientIpAddress(HttpServletRequest request) {
            String remoteAddr = request.getRemoteAddr();
            if (CharSequenceUtil.isBlank(remoteAddr)) {
                throw new IbmsException("Missing remote address");
            }

            // 标准化 remoteAddr（展开 IPv6，处理 mapped 地址）
            String normalizedRemote = normalizeAndStandardizeIp(remoteAddr);
            if (isClearlyInvalidIp(normalizedRemote)) {
                throw new IbmsException("Invalid or forbidden remote IP: " + remoteAddr);
            }

            // 非可信来源：禁止代理头
            if (!isTrustedProxy(normalizedRemote)) {
                checkForSuspiciousHeaders(request);
                return normalizedRemote;
            }

            // 可信代理：解析 X-Forwarded-For
            String xff = request.getHeader("X-Forwarded-For");
            if (StringUtils.hasText(xff)) {
                String realIp = extractRealClientIpFromXff(xff.trim(), normalizedRemote);
                if (realIp != null) {
                    return realIp;
                }
            }

            // Fallback headers（仅取第一个有效 IP）
            String[] fallbackHeaders = {"X-Real-IP", "Proxy-Client-IP", "WL-Proxy-Client-IP"};
            for (String header : fallbackHeaders) {
                String value = request.getHeader(header);
                if (StringUtils.hasText(value)) {
                    String ip = extractFirstValidIp(value);
                    if (ip != null) {
                        return ip;
                    }
                }
            }

            return normalizedRemote;
        }

        /**
         * 从 X-Forwarded-For 提取真实客户端 IP（从右往左找第一个非可信代理）
         */
        private String extractRealClientIpFromXff(String xffHeader, String directRemote) {
            String[] ips = xffHeader.split(",");
            // 优先：从右往左找第一个非可信代理
            for (int i = ips.length - 1; i >= 0; i--) {
                String candidate = normalizeAndStandardizeIp(ips[i].trim());
                if (candidate == null || isClearlyInvalidIp(candidate)) continue;
                if (!isTrustedProxy(candidate)) {
                    return candidate;
                }
            }
            // 兜底：返回最左边的有效 IP（可能是内网客户端）
            for (String raw : ips) {
                String candidate = normalizeAndStandardizeIp(raw.trim());
                if (candidate != null && !isClearlyInvalidIp(candidate)) {
                    return candidate;
                }
            }
            return null;
        }

        /**
         * 从逗号分隔字符串中提取第一个有效 IP（用于 fallback headers）
         */
        private String extractFirstValidIp(String headerValue) {
            if (!StringUtils.hasText(headerValue)) return null;
            String[] parts = headerValue.split(",");
            for (String part : parts) {
                String ip = normalizeAndStandardizeIp(part.trim());
                if (ip != null && !isClearlyInvalidIp(ip)) {
                    return ip;
                }
            }
            return null;
        }

        /**
         * 检查非可信来源是否携带可疑代理头（防伪造）
         */
        private void checkForSuspiciousHeaders(HttpServletRequest request) {
            String[] suspicious = {"X-Forwarded-For", "X-Real-IP", "Proxy-Client-IP", "WL-Proxy-Client-IP"};
            for (String header : suspicious) {
                String value = request.getHeader(header);
                if (StringUtils.hasText(value) && !value.equalsIgnoreCase("unknown")) {
                    throw new 自定义Exception("IP forgery detected: non-trusted client sent '" + header + "' header");
                }
            }
        }

        /**
         * 判断是否为明显无效 IP（含保留地址）
         */
        public static boolean isClearlyInvalidIp(String ip) {
            if (ip == null || FORBIDDEN_IPS.contains(ip.toLowerCase())) {
                return true;
            }
            try {
                InetAddress addr = InetAddress.getByName(ip);
                return addr.isAnyLocalAddress() ||      // 0.0.0.0 / ::
                        addr.isLinkLocalAddress() ||     // fe80::/10 等
                        //todo 不禁止 127.0.0.1
    //                    addr.isLoopbackAddress() || // 注意：这里不禁止 loopback，由业务决定
                        addr.isMulticastAddress();       // 224.0.0.0/4, ff00::/8
            } catch (Exception e) {
                log.debug("Invalid IP format: {}", ip);
                return true;
            }
        }

        /**
         * 标准化并统一 IP 表示（关键！）
         * - 展开 IPv6（::1 → 0:0:0:0:0:0:0:1）
         * - 处理 IPv4-mapped IPv6（::ffff:192.168.1.1 → 192.168.1.1）
         * - 返回标准 hostAddress
         */
        public static String normalizeAndStandardizeIp(String ip) {
            if (!StringUtils.hasText(ip)) return null;
            try {
                // 去除方括号（如 [::1]）
                String clean = ip.startsWith("[") && ip.contains("]")
                        ? ip.substring(1, ip.indexOf(']'))
                        : ip;

                InetAddress addr = InetAddress.getByName(clean);

                // 处理 IPv4-mapped IPv6
                if (addr instanceof Inet6Address) {
                    byte[] bytes = addr.getAddress();
                    if (bytes.length == 16 &&
                            Arrays.equals(Arrays.copyOfRange(bytes, 0, 10), new byte[10]) &&
                            bytes[10] == (byte) 0xff &&
                            bytes[11] == (byte) 0xff) {
                        // 映射回 IPv4
                        byte[] ipv4 = Arrays.copyOfRange(bytes, 12, 16);
                        return InetAddress.getByAddress(ipv4).getHostAddress();
                    }
                }

                return addr.getHostAddress(); // 标准形式
            } catch (Exception e) {
                log.warn("Failed to normalize IP: '{}'", ip, e);
                return null;
            }
        }

        /**
         * 判断 remoteAddr 是否在可信代理列表中
         */
        private boolean isTrustedProxy(String remoteAddr) {
            List<String> trusted = ipSecurityProperties.getTrustedProxies();
            if (trusted == null || trusted.isEmpty()) {
                return false;
            }

            boolean isRemoteIpv4 = Validator.isIpv4(remoteAddr);
            boolean isRemoteIpv6 = Validator.isIpv6(remoteAddr);

            for (String pattern : trusted) {
                if (CharSequenceUtil.isBlank(pattern)) continue;

                // 1. 纯 IP 地址匹配
                if (Validator.isIpv4(pattern) || Validator.isIpv6(pattern)) {
                    if (remoteAddr.equals(pattern)) {
                        log.debug("Trusted by exact IP match: {} == {}", remoteAddr, pattern);
                        return true;
                    }
                    continue;
                }

                // 2. CIDR 匹配（必须同协议族）
                if (pattern.contains("/")) {
                    String[] cidrParts = pattern.split("/", 2);
                    if (cidrParts.length != 2) continue;

                    String networkPart = cidrParts[0];
                    boolean isPatternIpv4 = Validator.isIpv4(networkPart);
                    boolean isPatternIpv6 = Validator.isIpv6(networkPart);

                    if (isRemoteIpv4 && isPatternIpv4) {
                        if (NetUtil.isInRange(remoteAddr, pattern)) {
                            log.debug("Trusted by IPv4 CIDR: {} in {}", remoteAddr, pattern);
                            return true;
                        }
                    } else if (isRemoteIpv6 && isPatternIpv6) {
                        if (NetUtil.isInRange(remoteAddr, pattern)) {
                            log.debug("Trusted by IPv6 CIDR: {} in {}", remoteAddr, pattern);
                            return true;
                        }
                    }
                    // 跨协议跳过
                    continue;
                }

                // 3. 主机名（如 nginx.internal）
                try {
                    for (InetAddress resolved : InetAddress.getAllByName(pattern)) {
                        if (remoteAddr.equals(resolved.getHostAddress())) {
                            log.debug("Trusted by hostname resolution: {} resolves to {}", pattern, remoteAddr);
                            return true;
                        }
                    }
                } catch (UnknownHostException e) {
                    log.warn("Failed to resolve trusted proxy hostname: {}", pattern);
                }
            }
            return false;
        }
    }
```
### 5.使用获取ip并校验
```
      try{
                //  新增：检查 IP 是否可疑（例如伪造）
                getIpUtil().getClientIpAddress(request);
            }catch (Exception e){
                response.setStatus(HttpServletResponse.SC_FORBIDDEN);
                response.setContentType("application/json;charset=UTF-8");
                response.getWriter().write("{\"code\":403,\"message\":\"IP forgery detected or untrusted proxy\"}");
                return; //  直接返回，不继续处理
            }

```