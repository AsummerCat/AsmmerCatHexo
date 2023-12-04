---
title: go读取yarn配置文件
date: 2022-10-19 14:04:24
tags: [go]
---
# 导入第三方包 viper


```
package main

import (
	"fmt"
	"github.com/spf13/viper"
)

// 1.使用viper包 来读取 json ,yaml,properties等配置文件
//go get github.com/spf13/viper

type YarnMysqlConfig struct {
	Url  string `mapstructure:"url"`
	Port int    `mapstructure:"port"`
}

type YarnRedisConfig struct {
	Host string `mapstructure:"host"`
	Port int    `mapstructure:"port"`
}
type ServiceConfig struct {
	Mysql YarnMysqlConfig `mapstructure:"mysql"`
	Redis YarnRedisConfig `mapstructure:"redis"`
}

func main() {
	//设置配置文件的名称
	viper.SetConfigName("myconfig")
	//设置配置文件的类型
	viper.SetConfigType("yaml")
	//添加配置文件的理解,指定位置读取
	viper.AddConfigPath("./day5")
	//寻找配置文件并且读取
	err := viper.ReadInConfig()
	if err != nil {
		println("读取配置文件失败", err)
		return
	}

	//这里是直接读取的
	fmt.Println(viper.Get("mysql"))
	fmt.Println(viper.Get("mysql.url"))
	fmt.Println(viper.Get("mysql.port"))
	fmt.Println(viper.Get("redis"))
	fmt.Println(viper.Get("redis.host"))
	fmt.Println(viper.Get("redis.port"))
	
	//序列化
	var config ServiceConfig
	viper.Unmarshal(&config)
	fmt.Println(config)





	//new一个viper对象
	v := viper.New()
	//获取配置文件
	v.SetConfigFile("./day5/myconfig.yml")

	//将配置读入viper对象中
	if err := v.ReadInConfig(); err != nil {
		println(err)
	}
	//这里可以转换为实体对象映射
	ServiceConfig := ServiceConfig{}
	v.Unmarshal(&ServiceConfig)
	fmt.Println(ServiceConfig)

}

```