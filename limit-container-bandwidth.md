# Docker 限制容器带宽

[Docker](https://www.docker.com/) 可以通过指定参数来限制容器的 CPU, 内存以及磁盘 IO。如 `--memory=64m` 限制容器最多使用 64 M 内存。 但是目前没有提供限制容器带宽的方法。

最近需要限制一个运行在容器中的应用的出口带宽，最后成功达到了效果，可参考：[limited-socks](https://github.com/shdxiang/limited-socks)。以下是尝试的过程。

## trickle

[trickle](http://manpages.ubuntu.com/manpages/trusty/man1/trickle.1.html) 可以限制某个应用的带宽，如运行 `ss-server` 并限制其上传速度 512 KB/s，下载速度 256 KB/s：

```
sudo apt-get install trickle
trickle -u 512 -d 256 ss-server
```

经测试，发现在非容器里使用的时候，确实能达到限速的功能，但把这个命令放在容器里的时候，会导致 `ss-server` 占用 CPU 达到 100%，几乎失去响应。可能的原因是：`trickle` 的原理是给应用的网络调用设置钩子，当网络调用很频繁的时候，开销可能比较大。

## wondershaper

[wondershaper](http://manpages.ubuntu.com/manpages/xenial/man8/wondershaper.8.html) 是一个网络流量调整脚本，它内部是调用的 `tc`（后面会提到），可以限制指定网卡的速度，如限制 `eth0` 的出口速度 512 KB/s，入口速度 256 KB/s：

```
sudo apt-get install wondershaper
sudo wondershaper eth0 1024 2048
```

注意 `wondershaper` 使用的单位是 Kbits/s。如在容器内使用，`docker run` 启动容器时候需要带上 `--cap-add=NET_ADMIN` 选项。

同样地，在非容器里使用的时候，确实能达到限速的功能，但在容器里确导致网络不可用，原因未明。

## tc

[tc](http://manpages.ubuntu.com/manpages/xenial/man8/tc.8.html) 属于 `iproute2` 可以直接修改内核的流控设置。但它的使用相对复杂，可参考其手册。如在容器内使用，`docker run` 启动容器时候需要带上 `--cap-add=NET_ADMIN` 选项。

使用以下命令可限制容器 `eth0` 的出口速度为 512 KB/s（没有研究怎么设置入口速度），而几乎没有带来额外的性能损耗：

```
sudo apt-get install iproute2
tc qdisc add dev eth0 root tbf rate 4mbit peakrate 8mbit burst 64kb latency 50ms minburst 1540
```

## 总结

因为每个容器基本只有一个主要的应用在运行，所以限制容器本身网卡的带宽和限制应用的带宽效果是一致的。
