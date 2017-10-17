---
title: Go Package —— bufio包
date: 2017-03-05 22:06:20
tags: golang
---

本文阅读golang的bufio包，常用的函数，结构体以及方法

## _scan.go_

### `func NewScanner(r io.Reader) *Scanner`
> 返回一个新的Scanner从r读取。split函数默认为ScanLines。

### `func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)`
> 是scaner的一个分割函数，将每一个字节作为一个字符返回。

### `func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)`
> ScanLines是一个Scanner的拆分函数，它返回每行文本，删除任何尾随的行尾标记。
> 返回的行可能为空。行结束标记是一个可选的回车，后跟一个强制换行。在正则表达式符号中，它是`\ r？\ n`。
> 最后一个非空行的输入将被返回，即使它没有换行符。

### `func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)`

### `func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)`
> 拆分函数，删除空格，返回空格分割的文字，永远不会返回一个空字符串。
> 空间定义由`unicode.IsSpace`设定。

### `func (s *Scanner) Err() error`

### `func (s *Scanner) Bytes() []byte`

### `func (s *Scanner) Text() string`

### `func (s *Scanner) Scan() bool`

### `func (s *Scanner) Buffer(buf []byte, max int)`

### `func (s *Scanner) Split(split SplitFunc)`
> SplitFunc 有四个：ScanBytes、ScanLines、ScanRunes、ScanWords。



## _bufio.go_

## Reader

### `func NewReader(rd io.Reader) *Reader `

> 创建一个reader，其中buffer的Size是默认大小。
> 其实就是调用`func NewReaderSize(rd io.Reader, size int) *Reader`


### `func (b *Reader) Reset(r io.Reader)`
> Reset放弃所有缓冲数据，重置所有状态和切换从r读取的缓冲读取器。

### `func (b *Reader) Peek(n int) ([]byte, error)`
> Peek返回下一个n字节，而不推进读取器。
> 如果Peek返回少于n个字节，它也返回一个错误，解释为什么读取短。
> 如果n大于b的缓冲区大小,错误是ErrBufferFull。

### `func (b *Reader) Discard(n int) (discarded int, err error)`
> Discard跳过接下来的n个字节，返回丢弃的字节数。
> 如果Discard跳过少于n个字节，它也返回一个错误。
> 如果0 <= n <= b.Buffered()，Discarding能够从底层的io.Reader读取。

### `func (b *Reader) Read(p []byte) (n int, err error) `

### `func (b *Reader) ReadByte() (byte, error)`

### `func (b *Reader) UnreadByte() error`

### `func (b *Reader) ReadRune() (r rune, size int, err error)`

### `func (b *Reader) UnreadRune() error`

### `func (b *Reader) Buffered() int `

### `func (b *Reader) ReadSlice(delim byte) (line []byte, err error)`

### `func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)`

### `func (b *Reader) ReadBytes(delim byte) ([]byte, error)`

### `func (b *Reader) ReadString(delim byte) (string, error)`

### `func (b *Reader) WriteTo(w io.Writer) (n int64, err error)`

## Writer

### `func NewWriterSize(w io.Writer, size int) *Writer `

### `func NewWriter(w io.Writer) *Writer`

### `func (b *Writer) Reset(w io.Writer)`

### `func (b *Writer) Flush() error`

### `func (b *Writer) Available() int`

### `func (b *Writer) Buffered() int`

### `func (b *Writer) Write(p []byte) (nn int, err error)`

### `func (b *Writer) WriteByte(c byte) error`

### `func (b *Writer) WriteRune(r rune) (size int, err error)`

### `func (b *Writer) WriteString(s string) (int, error)`

### `func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)`


















