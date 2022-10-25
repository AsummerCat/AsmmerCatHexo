---
title: Springboot整合Mqtt和protobuf
date: 2022-10-25 16:24:55
tags: [MQTT,SpringBoot,protobuf]
---
# Springboot整合Mqtt和protobuf

## demo地址
https://github.com/AsummerCat/MqttAndProtocolTest.git
<!--more-->
## 引入相关依赖
```
       <!--MQTT依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-integration</artifactId>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.integration</groupId>
            <artifactId>spring-integration-mqtt</artifactId>
        </dependency>


        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.83</version>
        </dependency>
        
                <!--  protobuf 支持 Java 核心包-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.21.8</version>
        </dependency>


        <!--  proto 与 Json 互转会用到-->
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java-util</artifactId>
            <version>3.21.8</version>
        </dependency>
```

## 大致流程
```
1.mqtt连接配置
2. Protocol编码配置
3. 自定义编码转换类

4.自定义编码消费者
5.自定义编码生产者

6.测试类
```

### 配置信息
```
spring:
  mqtt:
    enable: true
    url: tcp://broker.emqx.io
    username:
    password:
    consumerclientid: clientid12342312212_c
    # 自定义编码消费者id
    userconsumerclientid: clientid12342312212_d
    timeout: 5000
    # 心跳时间
    keepalive: 2
    # mqtt-topic
    producertopic: bstes,user
    consumertopic: bstes
```

### MTQQ配置类
```
package com.linjingc.mqttandprotocol.config;

import com.linjingc.mqttandprotocol.converter.UserConverter;
import com.linjingc.mqttandprotocol.mqtt.UserMqttConsumer;
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.integration.annotation.IntegrationComponentScan;
import org.springframework.integration.annotation.ServiceActivator;
import org.springframework.integration.channel.DirectChannel;
import org.springframework.integration.core.MessageProducer;
import org.springframework.integration.mqtt.core.DefaultMqttPahoClientFactory;
import org.springframework.integration.mqtt.core.MqttPahoClientFactory;
import org.springframework.integration.mqtt.inbound.MqttPahoMessageDrivenChannelAdapter;
import org.springframework.integration.mqtt.outbound.MqttPahoMessageHandler;
import org.springframework.messaging.MessageChannel;
import org.springframework.messaging.MessageHandler;


@Configuration
@IntegrationComponentScan
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
@Slf4j
public class MqttConfig {
    @Value("${spring.mqtt.username}")
    private String username;

    @Value("${spring.mqtt.password}")
    private String password;

    @Value("${spring.mqtt.url}")
    private String hostUrl;

    @Value("${spring.mqtt.userproducerclientid}")
    private String userProducerClientId;

    @Value("${spring.mqtt.producertopic}")
    private String producerTopic;

    //生产者和消费者是单独连接服务器会使用到一个clientid（客户端id），如果是同一个clientid的话会出现Lost connection: 已断开连接; retrying...
    @Value("${spring.mqtt.userconsumerclientid}")
    private String userConsumerclientid;

    @Value("${spring.mqtt.timeout}")
    private int timeout;   //连接超时

    @Value("${spring.mqtt.keepalive}")
    private int keepalive;  //连接超时


    @Autowired
    /**
     * 自定义编码器
     */
    private UserConverter userConverter;


    /**
     * MQTT客户端
     *
     * @return {@link org.springframework.integration.mqtt.core.MqttPahoClientFactory}
     */
    @Bean
    public MqttPahoClientFactory mqttClientFactory() {
        DefaultMqttPahoClientFactory factory = new DefaultMqttPahoClientFactory();
        //MQTT连接器选项
        MqttConnectOptions mqttConnectOptions = new MqttConnectOptions();
        mqttConnectOptions.setUserName(username);
        mqttConnectOptions.setPassword(password.toCharArray());
        mqttConnectOptions.setServerURIs(new String[]{hostUrl});
        mqttConnectOptions.setKeepAliveInterval(keepalive);
        factory.setConnectionOptions(mqttConnectOptions);
        return factory;
    }

    /*******************************生产者*******************************************/

    /**
     * MQTT信息通道（生产者）
     *
     * @return {@link MessageChannel}
     */
    @Bean(name = "mqttOutboundChannel")
    public MessageChannel mqttOutboundChannel() {
        return new DirectChannel();
    }


    /*******************************自定义编码生产者*******************************************/

    @Bean(name = "userMqttOutboundChannel")
    public MessageChannel userMqttOutboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息处理器（生产者）
     *
     * @return
     */
    @Bean
    //出站通道名（生产者）
    @ServiceActivator(inputChannel = "userMqttOutboundChannel")
    public MessageHandler userMqttOutbound() {
        MqttPahoMessageHandler messageHandler = new MqttPahoMessageHandler(userProducerClientId, mqttClientFactory());
        //如果设置成true，发送消息时将不会阻塞。
        messageHandler.setAsync(true);
        messageHandler.setDefaultTopic(producerTopic);
        messageHandler.setConverter(userConverter);
        return messageHandler;
    }

    /*******************************自定义编码消费者*******************************************/

    @Bean(name = "userMqttInboundChannel")
    public MessageChannel userMqttInboundChannel() {
        return new DirectChannel();
    }

    /**
     * MQTT消息订阅绑定（消费者）
     *
     * @return {@link org.springframework.integration.core.MessageProducer}
     */
    @Bean
    public MessageProducer userInbound() {
        // 可以同时消费（订阅）多个Topic
        MqttPahoMessageDrivenChannelAdapter adapter = new MqttPahoMessageDrivenChannelAdapter(userConsumerclientid, mqttClientFactory(), "user");
        adapter.setCompletionTimeout(timeout);
        //自定义编码器
        adapter.setConverter(userConverter);
        adapter.setQos(1);
        // 设置订阅通道
        adapter.setOutputChannel(userMqttInboundChannel());
        return adapter;
    }

    @Bean
    //入站通道名（消费者）
    @ServiceActivator(inputChannel = "userMqttInboundChannel")
    public MessageHandler userHandler() {
        return new UserMqttConsumer();
    }
}
 
```
### Protocol类
这里是 proto生成的
```
// Generated by the protocol buffer compiler.  DO NOT EDIT!
// source: User.proto

package com.linjingc.mqttandprotocol.converter;

public final class UserProto {
    private UserProto() {
    }

    public static void registerAllExtensions(
            com.google.protobuf.ExtensionRegistryLite registry) {
    }

    public static void registerAllExtensions(
            com.google.protobuf.ExtensionRegistry registry) {
        registerAllExtensions(
                (com.google.protobuf.ExtensionRegistryLite) registry);
    }

    public interface UserOrBuilder extends
            // @@protoc_insertion_point(interface_extends:com.wxw.notes.protobuf.proto.User)
            com.google.protobuf.MessageOrBuilder {

        /**
         * <pre>
         * 自身属性
         * </pre>
         *
         * <code>int32 id = 1;</code>
         *
         * @return The id.
         */
        int getId();

        /**
         * <code>string code = 2;</code>
         *
         * @return The code.
         */
        java.lang.String getCode();

        /**
         * <code>string code = 2;</code>
         *
         * @return The bytes for code.
         */
        com.google.protobuf.ByteString
        getCodeBytes();

        /**
         * <code>string name = 3;</code>
         *
         * @return The name.
         */
        java.lang.String getName();

        /**
         * <code>string name = 3;</code>
         *
         * @return The bytes for name.
         */
        com.google.protobuf.ByteString
        getNameBytes();
    }

    /**
     * Protobuf type {@code com.wxw.notes.protobuf.proto.User}
     */
    public static final class User extends
            com.google.protobuf.GeneratedMessageV3 implements
            // @@protoc_insertion_point(message_implements:com.wxw.notes.protobuf.proto.User)
            UserOrBuilder {
        private static final long serialVersionUID = 0L;

        // Use User.newBuilder() to construct.
        private User(com.google.protobuf.GeneratedMessageV3.Builder<?> builder) {
            super(builder);
        }

        private User() {
            code_ = "";
            name_ = "";
        }

        @java.lang.Override
        @SuppressWarnings({"unused"})
        protected java.lang.Object newInstance(
                UnusedPrivateParameter unused) {
            return new User();
        }

        @java.lang.Override
        public final com.google.protobuf.UnknownFieldSet
        getUnknownFields() {
            return this.unknownFields;
        }

        public static final com.google.protobuf.Descriptors.Descriptor
        getDescriptor() {
            return com.linjingc.mqttandprotocol.converter.UserProto.internal_static_com_wxw_notes_protobuf_proto_User_descriptor;
        }

        @java.lang.Override
        protected com.google.protobuf.GeneratedMessageV3.FieldAccessorTable
        internalGetFieldAccessorTable() {
            return com.linjingc.mqttandprotocol.converter.UserProto.internal_static_com_wxw_notes_protobuf_proto_User_fieldAccessorTable
                    .ensureFieldAccessorsInitialized(
                            com.linjingc.mqttandprotocol.converter.UserProto.User.class, com.linjingc.mqttandprotocol.converter.UserProto.User.Builder.class);
        }

        public static final int ID_FIELD_NUMBER = 1;
        private int id_;

        /**
         * <pre>
         * 自身属性
         * </pre>
         *
         * <code>int32 id = 1;</code>
         *
         * @return The id.
         */
        @java.lang.Override
        public int getId() {
            return id_;
        }

        public static final int CODE_FIELD_NUMBER = 2;
        private volatile java.lang.Object code_;

        /**
         * <code>string code = 2;</code>
         *
         * @return The code.
         */
        @java.lang.Override
        public java.lang.String getCode() {
            java.lang.Object ref = code_;
            if (ref instanceof java.lang.String) {
                return (java.lang.String) ref;
            } else {
                com.google.protobuf.ByteString bs =
                        (com.google.protobuf.ByteString) ref;
                java.lang.String s = bs.toStringUtf8();
                code_ = s;
                return s;
            }
        }

        /**
         * <code>string code = 2;</code>
         *
         * @return The bytes for code.
         */
        @java.lang.Override
        public com.google.protobuf.ByteString
        getCodeBytes() {
            java.lang.Object ref = code_;
            if (ref instanceof java.lang.String) {
                com.google.protobuf.ByteString b =
                        com.google.protobuf.ByteString.copyFromUtf8(
                                (java.lang.String) ref);
                code_ = b;
                return b;
            } else {
                return (com.google.protobuf.ByteString) ref;
            }
        }

        public static final int NAME_FIELD_NUMBER = 3;
        private volatile java.lang.Object name_;

        /**
         * <code>string name = 3;</code>
         *
         * @return The name.
         */
        @java.lang.Override
        public java.lang.String getName() {
            java.lang.Object ref = name_;
            if (ref instanceof java.lang.String) {
                return (java.lang.String) ref;
            } else {
                com.google.protobuf.ByteString bs =
                        (com.google.protobuf.ByteString) ref;
                java.lang.String s = bs.toStringUtf8();
                name_ = s;
                return s;
            }
        }

        /**
         * <code>string name = 3;</code>
         *
         * @return The bytes for name.
         */
        @java.lang.Override
        public com.google.protobuf.ByteString
        getNameBytes() {
            java.lang.Object ref = name_;
            if (ref instanceof java.lang.String) {
                com.google.protobuf.ByteString b =
                        com.google.protobuf.ByteString.copyFromUtf8(
                                (java.lang.String) ref);
                name_ = b;
                return b;
            } else {
                return (com.google.protobuf.ByteString) ref;
            }
        }

        private byte memoizedIsInitialized = -1;

        @java.lang.Override
        public final boolean isInitialized() {
            byte isInitialized = memoizedIsInitialized;
            if (isInitialized == 1) return true;
            if (isInitialized == 0) return false;

            memoizedIsInitialized = 1;
            return true;
        }

        @java.lang.Override
        public void writeTo(com.google.protobuf.CodedOutputStream output)
                throws java.io.IOException {
            if (id_ != 0) {
                output.writeInt32(1, id_);
            }
            if (!com.google.protobuf.GeneratedMessageV3.isStringEmpty(code_)) {
                com.google.protobuf.GeneratedMessageV3.writeString(output, 2, code_);
            }
            if (!com.google.protobuf.GeneratedMessageV3.isStringEmpty(name_)) {
                com.google.protobuf.GeneratedMessageV3.writeString(output, 3, name_);
            }
            getUnknownFields().writeTo(output);
        }

        @java.lang.Override
        public int getSerializedSize() {
            int size = memoizedSize;
            if (size != -1) return size;

            size = 0;
            if (id_ != 0) {
                size += com.google.protobuf.CodedOutputStream
                        .computeInt32Size(1, id_);
            }
            if (!com.google.protobuf.GeneratedMessageV3.isStringEmpty(code_)) {
                size += com.google.protobuf.GeneratedMessageV3.computeStringSize(2, code_);
            }
            if (!com.google.protobuf.GeneratedMessageV3.isStringEmpty(name_)) {
                size += com.google.protobuf.GeneratedMessageV3.computeStringSize(3, name_);
            }
            size += getUnknownFields().getSerializedSize();
            memoizedSize = size;
            return size;
        }

        @java.lang.Override
        public boolean equals(final java.lang.Object obj) {
            if (obj == this) {
                return true;
            }
            if (!(obj instanceof com.linjingc.mqttandprotocol.converter.UserProto.User)) {
                return super.equals(obj);
            }
            com.linjingc.mqttandprotocol.converter.UserProto.User other = (com.linjingc.mqttandprotocol.converter.UserProto.User) obj;

            if (getId()
                    != other.getId()) return false;
            if (!getCode()
                    .equals(other.getCode())) return false;
            if (!getName()
                    .equals(other.getName())) return false;
            if (!getUnknownFields().equals(other.getUnknownFields())) return false;
            return true;
        }

        @java.lang.Override
        public int hashCode() {
            if (memoizedHashCode != 0) {
                return memoizedHashCode;
            }
            int hash = 41;
            hash = (19 * hash) + getDescriptor().hashCode();
            hash = (37 * hash) + ID_FIELD_NUMBER;
            hash = (53 * hash) + getId();
            hash = (37 * hash) + CODE_FIELD_NUMBER;
            hash = (53 * hash) + getCode().hashCode();
            hash = (37 * hash) + NAME_FIELD_NUMBER;
            hash = (53 * hash) + getName().hashCode();
            hash = (29 * hash) + getUnknownFields().hashCode();
            memoizedHashCode = hash;
            return hash;
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                java.nio.ByteBuffer data)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                java.nio.ByteBuffer data,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data, extensionRegistry);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                com.google.protobuf.ByteString data)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                com.google.protobuf.ByteString data,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data, extensionRegistry);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(byte[] data)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                byte[] data,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws com.google.protobuf.InvalidProtocolBufferException {
            return PARSER.parseFrom(data, extensionRegistry);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(java.io.InputStream input)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseWithIOException(PARSER, input);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                java.io.InputStream input,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseWithIOException(PARSER, input, extensionRegistry);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseDelimitedFrom(java.io.InputStream input)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseDelimitedWithIOException(PARSER, input);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseDelimitedFrom(
                java.io.InputStream input,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseDelimitedWithIOException(PARSER, input, extensionRegistry);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                com.google.protobuf.CodedInputStream input)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseWithIOException(PARSER, input);
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User parseFrom(
                com.google.protobuf.CodedInputStream input,
                com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                throws java.io.IOException {
            return com.google.protobuf.GeneratedMessageV3
                    .parseWithIOException(PARSER, input, extensionRegistry);
        }

        @java.lang.Override
        public Builder newBuilderForType() {
            return newBuilder();
        }

        public static Builder newBuilder() {
            return DEFAULT_INSTANCE.toBuilder();
        }

        public static Builder newBuilder(com.linjingc.mqttandprotocol.converter.UserProto.User prototype) {
            return DEFAULT_INSTANCE.toBuilder().mergeFrom(prototype);
        }

        @java.lang.Override
        public Builder toBuilder() {
            return this == DEFAULT_INSTANCE
                    ? new Builder() : new Builder().mergeFrom(this);
        }

        @java.lang.Override
        protected Builder newBuilderForType(
                com.google.protobuf.GeneratedMessageV3.BuilderParent parent) {
            Builder builder = new Builder(parent);
            return builder;
        }

        /**
         * Protobuf type {@code com.wxw.notes.protobuf.proto.User}
         */
        public static final class Builder extends
                com.google.protobuf.GeneratedMessageV3.Builder<Builder> implements
                // @@protoc_insertion_point(builder_implements:com.wxw.notes.protobuf.proto.User)
                com.linjingc.mqttandprotocol.converter.UserProto.UserOrBuilder {
            public static final com.google.protobuf.Descriptors.Descriptor
            getDescriptor() {
                return com.linjingc.mqttandprotocol.converter.UserProto.internal_static_com_wxw_notes_protobuf_proto_User_descriptor;
            }

            @java.lang.Override
            protected com.google.protobuf.GeneratedMessageV3.FieldAccessorTable
            internalGetFieldAccessorTable() {
                return com.linjingc.mqttandprotocol.converter.UserProto.internal_static_com_wxw_notes_protobuf_proto_User_fieldAccessorTable
                        .ensureFieldAccessorsInitialized(
                                com.linjingc.mqttandprotocol.converter.UserProto.User.class, com.linjingc.mqttandprotocol.converter.UserProto.User.Builder.class);
            }

            // Construct using com.linjingc.mqttandprotocol.converter.UserProto.User.newBuilder()
            private Builder() {

            }

            private Builder(
                    com.google.protobuf.GeneratedMessageV3.BuilderParent parent) {
                super(parent);

            }

            @java.lang.Override
            public Builder clear() {
                super.clear();
                id_ = 0;

                code_ = "";

                name_ = "";

                return this;
            }

            @java.lang.Override
            public com.google.protobuf.Descriptors.Descriptor
            getDescriptorForType() {
                return com.linjingc.mqttandprotocol.converter.UserProto.internal_static_com_wxw_notes_protobuf_proto_User_descriptor;
            }

            @java.lang.Override
            public com.linjingc.mqttandprotocol.converter.UserProto.User getDefaultInstanceForType() {
                return com.linjingc.mqttandprotocol.converter.UserProto.User.getDefaultInstance();
            }

            @java.lang.Override
            public com.linjingc.mqttandprotocol.converter.UserProto.User build() {
                com.linjingc.mqttandprotocol.converter.UserProto.User result = buildPartial();
                if (!result.isInitialized()) {
                    throw newUninitializedMessageException(result);
                }
                return result;
            }

            @java.lang.Override
            public com.linjingc.mqttandprotocol.converter.UserProto.User buildPartial() {
                com.linjingc.mqttandprotocol.converter.UserProto.User result = new com.linjingc.mqttandprotocol.converter.UserProto.User(this);
                result.id_ = id_;
                result.code_ = code_;
                result.name_ = name_;
                onBuilt();
                return result;
            }

            @java.lang.Override
            public Builder clone() {
                return super.clone();
            }

            @java.lang.Override
            public Builder setField(
                    com.google.protobuf.Descriptors.FieldDescriptor field,
                    java.lang.Object value) {
                return super.setField(field, value);
            }

            @java.lang.Override
            public Builder clearField(
                    com.google.protobuf.Descriptors.FieldDescriptor field) {
                return super.clearField(field);
            }

            @java.lang.Override
            public Builder clearOneof(
                    com.google.protobuf.Descriptors.OneofDescriptor oneof) {
                return super.clearOneof(oneof);
            }

            @java.lang.Override
            public Builder setRepeatedField(
                    com.google.protobuf.Descriptors.FieldDescriptor field,
                    int index, java.lang.Object value) {
                return super.setRepeatedField(field, index, value);
            }

            @java.lang.Override
            public Builder addRepeatedField(
                    com.google.protobuf.Descriptors.FieldDescriptor field,
                    java.lang.Object value) {
                return super.addRepeatedField(field, value);
            }

            @java.lang.Override
            public Builder mergeFrom(com.google.protobuf.Message other) {
                if (other instanceof com.linjingc.mqttandprotocol.converter.UserProto.User) {
                    return mergeFrom((com.linjingc.mqttandprotocol.converter.UserProto.User) other);
                } else {
                    super.mergeFrom(other);
                    return this;
                }
            }

            public Builder mergeFrom(com.linjingc.mqttandprotocol.converter.UserProto.User other) {
                if (other == com.linjingc.mqttandprotocol.converter.UserProto.User.getDefaultInstance()) return this;
                if (other.getId() != 0) {
                    setId(other.getId());
                }
                if (!other.getCode().isEmpty()) {
                    code_ = other.code_;
                    onChanged();
                }
                if (!other.getName().isEmpty()) {
                    name_ = other.name_;
                    onChanged();
                }
                this.mergeUnknownFields(other.getUnknownFields());
                onChanged();
                return this;
            }

            @java.lang.Override
            public final boolean isInitialized() {
                return true;
            }

            @java.lang.Override
            public Builder mergeFrom(
                    com.google.protobuf.CodedInputStream input,
                    com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                    throws java.io.IOException {
                if (extensionRegistry == null) {
                    throw new java.lang.NullPointerException();
                }
                try {
                    boolean done = false;
                    while (!done) {
                        int tag = input.readTag();
                        switch (tag) {
                            case 0:
                                done = true;
                                break;
                            case 8: {
                                id_ = input.readInt32();

                                break;
                            } // case 8
                            case 18: {
                                code_ = input.readStringRequireUtf8();

                                break;
                            } // case 18
                            case 26: {
                                name_ = input.readStringRequireUtf8();

                                break;
                            } // case 26
                            default: {
                                if (!super.parseUnknownField(input, extensionRegistry, tag)) {
                                    done = true; // was an endgroup tag
                                }
                                break;
                            } // default:
                        } // switch (tag)
                    } // while (!done)
                } catch (com.google.protobuf.InvalidProtocolBufferException e) {
                    throw e.unwrapIOException();
                } finally {
                    onChanged();
                } // finally
                return this;
            }

            private int id_;

            /**
             * <pre>
             * 自身属性
             * </pre>
             *
             * <code>int32 id = 1;</code>
             *
             * @return The id.
             */
            @java.lang.Override
            public int getId() {
                return id_;
            }

            /**
             * <pre>
             * 自身属性
             * </pre>
             *
             * <code>int32 id = 1;</code>
             *
             * @param value The id to set.
             * @return This builder for chaining.
             */
            public Builder setId(int value) {

                id_ = value;
                onChanged();
                return this;
            }

            /**
             * <pre>
             * 自身属性
             * </pre>
             *
             * <code>int32 id = 1;</code>
             *
             * @return This builder for chaining.
             */
            public Builder clearId() {

                id_ = 0;
                onChanged();
                return this;
            }

            private java.lang.Object code_ = "";

            /**
             * <code>string code = 2;</code>
             *
             * @return The code.
             */
            public java.lang.String getCode() {
                java.lang.Object ref = code_;
                if (!(ref instanceof java.lang.String)) {
                    com.google.protobuf.ByteString bs =
                            (com.google.protobuf.ByteString) ref;
                    java.lang.String s = bs.toStringUtf8();
                    code_ = s;
                    return s;
                } else {
                    return (java.lang.String) ref;
                }
            }

            /**
             * <code>string code = 2;</code>
             *
             * @return The bytes for code.
             */
            public com.google.protobuf.ByteString
            getCodeBytes() {
                java.lang.Object ref = code_;
                if (ref instanceof String) {
                    com.google.protobuf.ByteString b =
                            com.google.protobuf.ByteString.copyFromUtf8(
                                    (java.lang.String) ref);
                    code_ = b;
                    return b;
                } else {
                    return (com.google.protobuf.ByteString) ref;
                }
            }

            /**
             * <code>string code = 2;</code>
             *
             * @param value The code to set.
             * @return This builder for chaining.
             */
            public Builder setCode(
                    java.lang.String value) {
                if (value == null) {
                    throw new NullPointerException();
                }

                code_ = value;
                onChanged();
                return this;
            }

            /**
             * <code>string code = 2;</code>
             *
             * @return This builder for chaining.
             */
            public Builder clearCode() {

                code_ = getDefaultInstance().getCode();
                onChanged();
                return this;
            }

            /**
             * <code>string code = 2;</code>
             *
             * @param value The bytes for code to set.
             * @return This builder for chaining.
             */
            public Builder setCodeBytes(
                    com.google.protobuf.ByteString value) {
                if (value == null) {
                    throw new NullPointerException();
                }
                checkByteStringIsUtf8(value);

                code_ = value;
                onChanged();
                return this;
            }

            private java.lang.Object name_ = "";

            /**
             * <code>string name = 3;</code>
             *
             * @return The name.
             */
            public java.lang.String getName() {
                java.lang.Object ref = name_;
                if (!(ref instanceof java.lang.String)) {
                    com.google.protobuf.ByteString bs =
                            (com.google.protobuf.ByteString) ref;
                    java.lang.String s = bs.toStringUtf8();
                    name_ = s;
                    return s;
                } else {
                    return (java.lang.String) ref;
                }
            }

            /**
             * <code>string name = 3;</code>
             *
             * @return The bytes for name.
             */
            public com.google.protobuf.ByteString
            getNameBytes() {
                java.lang.Object ref = name_;
                if (ref instanceof String) {
                    com.google.protobuf.ByteString b =
                            com.google.protobuf.ByteString.copyFromUtf8(
                                    (java.lang.String) ref);
                    name_ = b;
                    return b;
                } else {
                    return (com.google.protobuf.ByteString) ref;
                }
            }

            /**
             * <code>string name = 3;</code>
             *
             * @param value The name to set.
             * @return This builder for chaining.
             */
            public Builder setName(
                    java.lang.String value) {
                if (value == null) {
                    throw new NullPointerException();
                }

                name_ = value;
                onChanged();
                return this;
            }

            /**
             * <code>string name = 3;</code>
             *
             * @return This builder for chaining.
             */
            public Builder clearName() {

                name_ = getDefaultInstance().getName();
                onChanged();
                return this;
            }

            /**
             * <code>string name = 3;</code>
             *
             * @param value The bytes for name to set.
             * @return This builder for chaining.
             */
            public Builder setNameBytes(
                    com.google.protobuf.ByteString value) {
                if (value == null) {
                    throw new NullPointerException();
                }
                checkByteStringIsUtf8(value);

                name_ = value;
                onChanged();
                return this;
            }

            @java.lang.Override
            public final Builder setUnknownFields(
                    final com.google.protobuf.UnknownFieldSet unknownFields) {
                return super.setUnknownFields(unknownFields);
            }

            @java.lang.Override
            public final Builder mergeUnknownFields(
                    final com.google.protobuf.UnknownFieldSet unknownFields) {
                return super.mergeUnknownFields(unknownFields);
            }


            // @@protoc_insertion_point(builder_scope:com.wxw.notes.protobuf.proto.User)
        }

        // @@protoc_insertion_point(class_scope:com.wxw.notes.protobuf.proto.User)
        private static final com.linjingc.mqttandprotocol.converter.UserProto.User DEFAULT_INSTANCE;

        static {
            DEFAULT_INSTANCE = new com.linjingc.mqttandprotocol.converter.UserProto.User();
        }

        public static com.linjingc.mqttandprotocol.converter.UserProto.User getDefaultInstance() {
            return DEFAULT_INSTANCE;
        }

        private static final com.google.protobuf.Parser<User>
                PARSER = new com.google.protobuf.AbstractParser<User>() {
            @java.lang.Override
            public User parsePartialFrom(
                    com.google.protobuf.CodedInputStream input,
                    com.google.protobuf.ExtensionRegistryLite extensionRegistry)
                    throws com.google.protobuf.InvalidProtocolBufferException {
                Builder builder = newBuilder();
                try {
                    builder.mergeFrom(input, extensionRegistry);
                } catch (com.google.protobuf.InvalidProtocolBufferException e) {
                    throw e.setUnfinishedMessage(builder.buildPartial());
                } catch (com.google.protobuf.UninitializedMessageException e) {
                    throw e.asInvalidProtocolBufferException().setUnfinishedMessage(builder.buildPartial());
                } catch (java.io.IOException e) {
                    throw new com.google.protobuf.InvalidProtocolBufferException(e)
                            .setUnfinishedMessage(builder.buildPartial());
                }
                return builder.buildPartial();
            }
        };

        public static com.google.protobuf.Parser<User> parser() {
            return PARSER;
        }

        @java.lang.Override
        public com.google.protobuf.Parser<User> getParserForType() {
            return PARSER;
        }

        @java.lang.Override
        public com.linjingc.mqttandprotocol.converter.UserProto.User getDefaultInstanceForType() {
            return DEFAULT_INSTANCE;
        }

    }

    private static final com.google.protobuf.Descriptors.Descriptor
            internal_static_com_wxw_notes_protobuf_proto_User_descriptor;
    private static final
    com.google.protobuf.GeneratedMessageV3.FieldAccessorTable
            internal_static_com_wxw_notes_protobuf_proto_User_fieldAccessorTable;

    public static com.google.protobuf.Descriptors.FileDescriptor
    getDescriptor() {
        return descriptor;
    }

    private static com.google.protobuf.Descriptors.FileDescriptor
            descriptor;

    static {
        java.lang.String[] descriptorData = {
                "\n\nUser.proto\022\034com.wxw.notes.protobuf.pro" +
                        "to\".\n\004User\022\n\n\002id\030\001 \001(\005\022\014\n\004code\030\002 \001(\t\022\014\n\004" +
                        "name\030\003 \001(\tB3\n&com.linjingc.mqttandprotoc" +
                        "ol.converterB\tUserProtob\006proto3"
        };
        descriptor = com.google.protobuf.Descriptors.FileDescriptor
                .internalBuildGeneratedFileFrom(descriptorData,
                        new com.google.protobuf.Descriptors.FileDescriptor[]{
                        });
        internal_static_com_wxw_notes_protobuf_proto_User_descriptor =
                getDescriptor().getMessageTypes().get(0);
        internal_static_com_wxw_notes_protobuf_proto_User_fieldAccessorTable = new
                com.google.protobuf.GeneratedMessageV3.FieldAccessorTable(
                internal_static_com_wxw_notes_protobuf_proto_User_descriptor,
                new java.lang.String[]{"Id", "Code", "Name",});
    }

    // @@protoc_insertion_point(outer_class_scope)
}

```
### 自定义消息编码解码
```
package com.linjingc.mqttandprotocol.converter;

import com.fasterxml.jackson.core.JsonProcessingException;
import lombok.extern.slf4j.Slf4j;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.integration.mqtt.support.MqttMessageConverter;
import org.springframework.integration.support.AbstractIntegrationMessageBuilder;
import org.springframework.integration.support.MessageBuilder;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHeaders;
import org.springframework.stereotype.Component;
import org.springframework.util.Assert;

import java.io.IOException;

@Component
@Slf4j
public class UserConverter implements MqttMessageConverter {
    private int defaultQos = 0;
    private boolean defaultRetain = false;

    //消费者->入站消息解码
    @Override
    public AbstractIntegrationMessageBuilder<UserProto.User> toMessageBuilder(String topic, MqttMessage mqttMessage) {
        UserProto.User protocol = null;
        try {
            //反序列化
            protocol = UserProto.User.parseFrom(mqttMessage.getPayload());
        } catch (IOException e) {
            if (e instanceof JsonProcessingException) {
                System.out.println();
                log.error("Converter only support json string");
            }
        }
        assert protocol != null;
        MessageBuilder<UserProto.User> messageBuilder = MessageBuilder
                .withPayload(protocol);
        //使用withPayload初始化的消息缺少头信息，将原消息头信息填充进去
        messageBuilder.setHeader(MqttHeaders.ID, mqttMessage.getId())
                .setHeader(MqttHeaders.RECEIVED_QOS, mqttMessage.getQos())
                .setHeader(MqttHeaders.DUPLICATE, mqttMessage.isDuplicate())
                .setHeader(MqttHeaders.RECEIVED_RETAINED, mqttMessage.isRetained());
        if (topic != null) {
            messageBuilder.setHeader(MqttHeaders.TOPIC, topic);
        }
        return messageBuilder;
    }

    //生产者->出站消息编码
    @Override
    public Object fromMessage(Message<?> message, Class<?> targetClass) {
        MqttMessage mqttMessage = new MqttMessage();
        UserProto.User user = (UserProto.User) message.getPayload();
        //转换成字节数组
        byte[] msg = user.toByteArray();
        mqttMessage.setPayload(msg);

        //这里的 mqtt_qos ,和 mqtt_retained 由 MqttHeaders 此类得出不可以随便取，如需其他属性自行查找
        Integer qos = (Integer) message.getHeaders().get("mqtt_qos");
        mqttMessage.setQos(qos == null ? defaultQos : qos);
        Boolean retained = (Boolean) message.getHeaders().get("mqtt_retained");
        mqttMessage.setRetained(retained == null ? defaultRetain : retained);
        return mqttMessage;
    }

    //此方法直接拿默认编码器的来用的，照抄即可
    @Override
    public Message<?> toMessage(Object payload, MessageHeaders headers) {
        Assert.isInstanceOf(MqttMessage.class, payload,
                () -> "This converter can only convert an 'MqttMessage'; received: " + payload.getClass().getName());
        return this.toMessage(null, (MqttMessage) payload);
    }

    public void setDefaultQos(int defaultQos) {
        this.defaultQos = defaultQos;
    }

    public void setDefaultRetain(boolean defaultRetain) {
        this.defaultRetain = defaultRetain;
    }
}

```

### 自定义生产者
```
package com.linjingc.mqttandprotocol.mqtt;

import com.linjingc.mqttandprotocol.converter.UserProto;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.annotation.MessagingGateway;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.handler.annotation.Header;
import org.springframework.stereotype.Component;

/**
 * 生产者发送消息
 */

//@MessagingGateway是一个用于提供消息网关代理整合的注解，参数defaultRequestChannel指定发送消息绑定的channel。
@MessagingGateway(defaultRequestChannel = "userMqttOutboundChannel")
@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
public interface UserMqttProducer {
    //自定义编码数据
    void sendToMqtt(UserProto.User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, UserProto.User user);

    void sendToMqtt(@Header(MqttHeaders.TOPIC) String topic, @Header(MqttHeaders.QOS) Integer Qos, UserProto.User user);
}
```

### 自定义消费者
```
package com.linjingc.mqttandprotocol.mqtt;

import com.google.protobuf.InvalidProtocolBufferException;
import com.google.protobuf.util.JsonFormat;
import com.linjingc.mqttandprotocol.converter.UserProto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.integration.mqtt.support.MqttHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.MessageHandler;
import org.springframework.messaging.MessagingException;
import org.springframework.stereotype.Component;

/**
 * 自定义编码消息
 */
@Component
@ConditionalOnProperty(value = "spring.mqtt.enable", havingValue = "true")
@Slf4j
public class UserMqttConsumer implements MessageHandler {

    @Override
    public void handleMessage(Message<?> message) throws MessagingException {
        String topic = String.valueOf(message.getHeaders().get(MqttHeaders.TOPIC));
        UserProto.User user = (UserProto.User) message.getPayload();
        try {
            log.info("User接收到 mqtt消息，主题:{} 消息:{}", topic, JsonFormat.printer().print(user));
        } catch (InvalidProtocolBufferException e) {
            e.printStackTrace();
        }
    }
}
```
### 测试类
```
package com.linjingc.mqttandprotocol.controller;

import com.google.protobuf.InvalidProtocolBufferException;
import com.google.protobuf.util.JsonFormat;
import com.linjingc.mqttandprotocol.converter.UserProto;
import com.linjingc.mqttandprotocol.mqtt.UserMqttProducer;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MqttController {


    @Autowired
    private UserMqttProducer userMqttProducer;


    @RequestMapping("/user")
    public String send1() throws InvalidProtocolBufferException {
        UserProto.User.Builder user = UserProto.User.newBuilder();
        user.setId(100).setCode("10086").setName("小明").build();
        //序列化
        UserProto.User build = user.build();
        userMqttProducer.sendToMqtt("user", build);
        return "send message : " + JsonFormat.printer().print(build);
    }


}

```