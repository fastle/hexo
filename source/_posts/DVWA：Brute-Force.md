---
title: DVWA：Brute Force
date: 2024-09-25 16:23:07
tags:
 - 网络攻防
 - DVWA
---

暴力破解基本逻辑是使用弱密码和枚举法， 

# Low 
在 low 模式下进行攻击获取流程为， 首先发送验证信息并进行抓包， 然后使用 Intruder 进行爆破。 

# Medium
使用了符号转义， 可以防止 sql 注入， 不影响我们直接爆破。

# High
使用了 token ， 每次查询失败会 sleep 随机0到3秒。

对于 token ， 需要在爆破时分析 response 的返回值， 找到 token 的位置， 然后在爆破时将 token 添加到 payload 中。

# Impossible
在 high 的基础上对用户登录次数有所限制， 三次后会锁柱15秒。 就比较难爆破了。


建立一个go的可中断文件下载库，从指定url获取文件，支持中止下载，继续下载，停止下载， 中止和继续过程支持断点续传。

```go

package resumable

import (
	"context"
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"sync"
	"time"
)

type Downloader struct {
	URL         string        // 下载地址
	DestPath    string        // 目标路径
	TempFile    string        // 临时文件路径
	ProgressFile string       // 进度记录文件
	ctx         context.Context
	cancel      context.CancelFunc
	mu          sync.Mutex    // 保护共享状态
	downloaded  int64         // 已下载字节数
	err         error         // 下载错误
	resumable   bool          // 是否支持断点续传
}

type Progress struct {
	Total     int64   // 文件总大小
	Downloaded int64  // 已下载字节
	Progress  float64 // 进度百分比
}

type Config struct {
	MaxConcurrent int           // 最大并发数
	Timeout       time.Duration // 超时时间
	ProgressFunc  func(Progress) // 进度回调
}

func NewDownloader(url, dest string, cfg Config) *Downloader {
	dl := &Downloader{
		URL:         url,
		DestPath:    dest,
		TempFile:    dest + ".tmp",
		ProgressFile: dest + ".progress",
	}
	dl.ctx, dl.cancel = context.WithCancel(context.Background())
	return dl
}

// 启动/恢复下载
func (d *Downloader) Start() error {
	d.mu.Lock()
	defer d.mu.Unlock()

	// 读取进度文件
	if err := d.loadProgress(); err != nil && !os.IsNotExist(err) {
		return err
	}

	req, err := http.NewRequest("GET", d.URL, nil)
	if err != nil {
		return err
	}

	// 设置Range请求头
	if d.downloaded > 0 {
		req.Header.Set("Range", fmt.Sprintf("bytes=%d-", d.downloaded))
	}

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	// 处理服务器响应
	switch resp.StatusCode {
	case http.StatusPartialContent:
		d.resumable = true
	case http.StatusOK:
		d.resumable = false
		d.downloaded = 0
	default:
		return fmt.Errorf("unexpected status: %s", resp.Status)
	}

	// 创建/打开临时文件
	file, err := os.OpenFile(d.TempFile, os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	if _, err = file.Seek(d.downloaded, io.SeekStart); err != nil {
		return err
	}

	// 开始下载
	reader := io.TeeReader(resp.Body, &progressWriter{d: d})
	if _, err = io.Copy(file, reader); err != nil {
		return err
	}

	// 下载完成后处理
	if err := d.finalize(); err != nil {
		return err
	}
	return nil
}

// 暂停下载
func (d *Downloader) Pause() error {
	d.cancel()
	d.mu.Lock()
	defer d.mu.Unlock()
	return d.saveProgress()
}

// 终止下载（清除临时文件）
func (d *Downloader) Abort() error {
	d.cancel()
	d.mu.Lock()
	defer d.mu.Unlock()
	os.Remove(d.TempFile)
	return os.Remove(d.ProgressFile)
}

// 保存下载进度
func (d *Downloader) saveProgress() error {
	f, err := os.Create(d.ProgressFile)
	if err != nil {
		return err
	}
	defer f.Close()
	_, err = fmt.Fprintf(f, "%d", d.downloaded)
	return err
}

// 加载进度
func (d *Downloader) loadProgress() error {
	data, err := os.ReadFile(d.ProgressFile)
	if err != nil {
		return err
	}
	_, err = fmt.Sscanf(string(data), "%d", &d.downloaded)
	return err
}

// 下载完成后的收尾工作
func (d *Downloader) finalize() error {
	if err := os.Rename(d.TempFile, d.DestPath); err != nil {
		return err
	}
	os.Remove(d.ProgressFile)
	return nil
}

// 进度记录writer
type progressWriter struct {
	d *Downloader
}

func (pw *progressWriter) Write(p []byte) (int, error) {
	n := len(p)
	pw.d.mu.Lock()
	pw.d.downloaded += int64(n)
	pw.d.mu.Unlock()
	
	// 定期保存进度
	if pw.d.downloaded%1024 == 0 {
		if err := pw.d.saveProgress(); err != nil {
			return 0, err
		}
	}
	return n, nil
}package resumable

import (
	"context"
	"fmt"
	"io"
	"net/http"
	"os"
	"path/filepath"
	"sync"
	"time"
)

type Downloader struct {
	URL         string        // 下载地址
	DestPath    string        // 目标路径
	TempFile    string        // 临时文件路径
	ProgressFile string       // 进度记录文件
	ctx         context.Context
	cancel      context.CancelFunc
	mu          sync.Mutex    // 保护共享状态
	downloaded  int64         // 已下载字节数
	err         error         // 下载错误
	resumable   bool          // 是否支持断点续传
}

type Progress struct {
	Total     int64   // 文件总大小
	Downloaded int64  // 已下载字节
	Progress  float64 // 进度百分比
}

type Config struct {
	MaxConcurrent int           // 最大并发数
	Timeout       time.Duration // 超时时间
	ProgressFunc  func(Progress) // 进度回调
}

func NewDownloader(url, dest string, cfg Config) *Downloader {
	dl := &Downloader{
		URL:         url,
		DestPath:    dest,
		TempFile:    dest + ".tmp",
		ProgressFile: dest + ".progress",
	}
	dl.ctx, dl.cancel = context.WithCancel(context.Background())
	return dl
}

// 启动/恢复下载
func (d *Downloader) Start() error {
	d.mu.Lock()
	defer d.mu.Unlock()

	// 读取进度文件
	if err := d.loadProgress(); err != nil && !os.IsNotExist(err) {
		return err
	}

	req, err := http.NewRequest("GET", d.URL, nil)
	if err != nil {
		return err
	}

	// 设置Range请求头
	if d.downloaded > 0 {
		req.Header.Set("Range", fmt.Sprintf("bytes=%d-", d.downloaded))
	}

	resp, err := http.DefaultClient.Do(req)
	if err != nil {
		return err
	}
	defer resp.Body.Close()

	// 处理服务器响应
	switch resp.StatusCode {
	case http.StatusPartialContent:
		d.resumable = true
	case http.StatusOK:
		d.resumable = false
		d.downloaded = 0
	default:
		return fmt.Errorf("unexpected status: %s", resp.Status)
	}

	// 创建/打开临时文件
	file, err := os.OpenFile(d.TempFile, os.O_CREATE|os.O_WRONLY, 0644)
	if err != nil {
		return err
	}
	defer file.Close()

	if _, err = file.Seek(d.downloaded, io.SeekStart); err != nil {
		return err
	}

	// 开始下载
	reader := io.TeeReader(resp.Body, &progressWriter{d: d})
	if _, err = io.Copy(file, reader); err != nil {
		return err
	}

	// 下载完成后处理
	if err := d.finalize(); err != nil {
		return err
	}
	return nil
}

// 暂停下载
func (d *Downloader) Pause() error {
	d.cancel()
	d.mu.Lock()
	defer d.mu.Unlock()
	return d.saveProgress()
}

// 终止下载（清除临时文件）
func (d *Downloader) Abort() error {
	d.cancel()
	d.mu.Lock()
	defer d.mu.Unlock()
	os.Remove(d.TempFile)
	return os.Remove(d.ProgressFile)
}

// 保存下载进度
func (d *Downloader) saveProgress() error {
	f, err := os.Create(d.ProgressFile)
	if err != nil {
		return err
	}
	defer f.Close()
	_, err = fmt.Fprintf(f, "%d", d.downloaded)
	return err
}

// 加载进度
func (d *Downloader) loadProgress() error {
	data, err := os.ReadFile(d.ProgressFile)
	if err != nil {
		return err
	}
	_, err = fmt.Sscanf(string(data), "%d", &d.downloaded)
	return err
}

// 下载完成后的收尾工作
func (d *Downloader) finalize() error {
	if err := os.Rename(d.TempFile, d.DestPath); err != nil {
		return err
	}
	os.Remove(d.ProgressFile)
	return nil
}

// 进度记录writer
type progressWriter struct {
	d *Downloader
}

func (pw *progressWriter) Write(p []byte) (int, error) {
	n := len(p)
	pw.d.mu.Lock()
	pw.d.downloaded += int64(n)
	pw.d.mu.Unlock()
	
	// 定期保存进度
	if pw.d.downloaded%1024 == 0 {
		if err := pw.d.saveProgress(); err != nil {
			return 0, err
		}
	}
	return n, nil
}


```