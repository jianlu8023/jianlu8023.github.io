+++
title = 'go语言发送https请求'
date = '2024-07-01T12:41:37+08:00'
author = 'jianlu'
draft = true
description = "go语言发送https请求"
aliases = ["https"]
+++

# HTTPS 不开启证书验证发送请求

## GOLANG

```go
package main

import (
    "bytes"
    "crypto/tls"
    "fmt"
    "io"
    "mime/multipart"
    "net/http"
    "os"
)

func main() {
    
    client := &http.Client{
        Transport: &http.Transport{
            TLSClientConfig: &tls.Config{
                InsecureSkipVerify: true,
            },
        },
    }
    
    path := "/opt/files/demo.txt"
    
    file, err := os.Open(path)
    if err != nil {
        fmt.Println("Error opening file:", err)
        return
    }
    
    body := &bytes.Buffer{}
    
    writer := multipart.NewWriter(body)
    
    part, err := writer.CreateFormFile("param1", file.Name())
    if err != nil {
        fmt.Println("Error creating form file:", err)
        return
    }
    _, err = io.Copy(part, file)
    if err != nil {
        fmt.Println("Error copying file:", err)
        return
    }
    err = writer.WriteField("param2", "value2")
    if err != nil {
        fmt.Println("Error writing field:", err)
        return
    }
    
    err = writer.Close()
    if err != nil {
        fmt.Println("Error closing writer:", err)
        return
    }
    url := "https://example.com/api/demo"
    request, err := http.NewRequest(http.MethodPost, url, body)
    if err != nil {
        fmt.Println("Error creating request:", err)
        return
    }
    request.Header.Set("Content-Type", writer.FormDataContentType())
    response, err := client.Do(request)
    if err != nil {
        fmt.Println("Error sending request:", err)
        return
    }
    defer func(Body io.ReadCloser) {
        err := Body.Close()
        if err != nil {
            fmt.Println("Error closing response body:", err)
        }
    }(response.Body)
    fmt.Println("Response status:", response.Status)
    fmt.Println("Response body:", response.Body)
}
```