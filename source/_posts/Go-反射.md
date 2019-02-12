---
title: Go-反射
date: 2018-11-22 14:50:07
tags: ["Go"]
---

## 什么是反射？
反射是程序在运行时检查其变量和值并找到其类型的能力。

```Go
package main

import (
	"fmt"
	"reflect"
)

type order struct {
	ordId      int
	customerId int
}

type employee struct {
	name    string
	id      int
	address string
	salary  int
	country string
}

func createQuery(q interface{}) {

	if reflect.ValueOf(q).Kind() == reflect.Struct {
		tableName := reflect.TypeOf(q).Name()
		query := fmt.Sprintf("insert into %s(", tableName)

		t := reflect.TypeOf(q)
		for i := 0; i < t.NumField(); i++ {
			if i == 0 {
				query = fmt.Sprintf("%s%s", query, t.Field(i).Name)
			} else {
				query = fmt.Sprintf("%s, %s", query, t.Field(i).Name)
			}
		}

		query = fmt.Sprintf("%s%s", query, ")")
		query = query + " values("

		v := reflect.ValueOf(q)

		for i := 0; i < v.NumField(); i++ {
			switch v.Field(i).Kind() {
			case reflect.Int:
				if i == 0 {
					query = fmt.Sprintf("%s%d", query, v.Field(i).Int())
				} else {
					query = fmt.Sprintf("%s, %d", query, v.Field(i).Int())
				}
			case reflect.String:
				if i == 0 {
					query = fmt.Sprintf("%s\"%s\"", query, v.Field(i).String())
				} else {
					query = fmt.Sprintf("%s, \"%s\"", query, v.Field(i).String())
				}
			default:
				fmt.Println("Unsupported type")
				return
			}
		}
		query = fmt.Sprintf("%s)", query)
		fmt.Println(query)
		return

	}
	fmt.Println("unsupported type")

}

func main() {
	e := employee{
		name:    "Naveen",
		id:      565,
		address: "Science Park Road, Singapore",
		salary:  90000,
		country: "Singapore",
	}
	createQuery(e)
}

```