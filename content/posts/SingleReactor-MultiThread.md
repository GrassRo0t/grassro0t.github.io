---
title: "项目实践-单reactor多线程模型实例"
slug: "SingleReactor-MultiThread"
date: 2026-09-12T12:00:00+08:00
draft: false   # true=草稿，构建默认忽略
tags: ["Reactor", "并发编程"]
categories: ["项目实践"]
summary: "一个单reactor多线程模型实例，一个 Reactor 线程用 epoll 管所有连接的 IO，一个线程池管所有耗时业务。"
toc: true
comments: true
description: "单reactor多线程模型"
---

# 单reactor多线程模型实例
> **一个 Reactor 线程用 epoll 管所有连接的 IO，一个线程池管所有耗时业务**
> **本项目源码来自 https://github.com/grassro0t/computer-science-practice**
> **仅支持 Linux**
---
## 1. 基本架构
- 特征：
	- 线程池无界任务队列
	- 连接用cid标识，epoll_event注册事件时带上cid作为数据
	- fd全非阻塞
	- epoll水平触发
	- 优雅退出：通过信号转换成进程级退出标识`static volatile sig_atomic_t g_stop = 0;`，事件循环每轮检查
	- 数据流：
		- Reactor → worker：线程池的`tasks_` + 条件变量 `cv_`
		- worker → Reactor：`pending_` 响应队列 + `eventfd` 唤醒
	- 锁：
		- `conns_`、`fd2cid_`、各连接的读/写缓冲与 epoll 事件集被reactor独占访问无锁
		- `pending_` 被 worker 和 Reactor 两端读写 → 用一把 `mu_` 保护
		- `eventfd` 写/读内核保证天然线程安全
		- `stop_` 是 `atomic`，任何线程可以申请关闭Reactor
```
 client1 --+
 client2 --+--- TCP 连接 ----+
 client3 --+                  |
                             v
        +------------------------------------------------------+
        |  Reactor 线程（整个进程唯一的事件循环线程）            |
        |  每轮循环：                                          |
        |    ① epoll_wait() 阻塞等待就绪事件                    |
        |    ② eventfd 可读 → 取 worker 算好的响应并 send      |
        |    ③ 监听 fd 可读 → accept，新连接 ADD 进 epoll      |
        |    ④ 连接可读 → recv → 按行拆包 → 投递线程池         |
        |    ⑤ 连接可写 → 把写缓冲剩余数据发完                 |
        +------+-----------------------------------^----------+
               | ① 投递: 每个完整请求                | ② 回传: 算好的响应
               v                                   + (pending_ 队列 + eventfd 唤醒)
        +-----------------------+
        |  业务线程池 ThreadPool |
        |  worker 1..N          |
        |  只跑 HandleRequest() |  ← sleep / fib 等耗时、CPU 密集计算
        +-----------------------+
```
- 数据流：
```
客户端 --请求--> Reactor 线程(recv/拆包/ADD 进 epoll) --投递--> 线程池(计算)
客户端 <--响应--- Reactor 线程(send)                 <--回传--- 线程池(pending_ + eventfd)
```
---
## 2. 核心方法

| 位置           | 成员                                          | 一句话职责                                                       |
| ------------ | ------------------------------------------- | ----------------------------------------------------------- |
| `ThreadPool` | `Submit()`                                  | 任务入队 + 唤醒一个 worker                                          |
| `ThreadPool` | `WorkLoop()` / `Shutdown()`                 | worker 等任务并锁外执行；停机并 join                                    |
| `Reactor`    | 构造                                          | 装信号 → `epoll_create1` → 建监听 → `eventfd` → 注册 listen/eventfd |
| `Reactor`    | `Run()`                                     | 主循环：`epoll_wait` → 分发事件                                     |
| `Reactor`    | `OnListenReadable()`                        | `accept` 新连接，分配 cid，注册 `EPOLLIN`                            |
| `Reactor`    | `OnConnReadable()`                          | `recv` → 攒进 `inbuf` → 按 `\n` 拆请求 → 投线程池                     |
| `Reactor`    | `OnWakeupReadable()`                        | 取出 `pending_` 里 worker 的结果并 `send`                          |
| `Reactor`    | `CloseConn()`                               | `epoll_ctl DEL` → 删表 → `close(fd)`                          |
| `Reactor`    | `RunBusiness()` / `PostResponse()`          | （worker 线程）执行业务 / 结果入队+唤醒                                   |
| 自由函数         | `HandleRequest()`                           | 协议实现（worker 内执行，不碰 socket）                                  |
| 自由函数         | `SetNonBlocking()` / `CreateListenSocket()` | fd 非阻塞化 / 监听 socket 初始化                                     |
## 3. 代码详解
### 连接封装（Reactor内部）
``` cpp
// 一条 TCP 连接的读侧状态（只被 Reactor 线程访问）。
struct Conn {
	uint64_t cid = 0;      // 连接编号
	int fd = -1;           // socket 文件描述符
	std::string inbuf;     // 读缓冲：攒到 '\n' 才算一个完整请求
};
```
### 响应封装
``` cpp
struct Response {
  uint64_t cid = 0;  // 目标连接编号（不是 fd）
  std::string body;  // 一行响应正文（不含结尾 '\n'）
};
```
### 线程池
- 内部有task队列、mutex和cv管理线程同步
``` cpp
class ThreadPool {
 public:
  explicit ThreadPool(size_t n) {
    for (size_t i = 0; i < n; ++i)
      workers_.emplace_back([this] { WorkLoop(); });
  }
  ~ThreadPool() { Shutdown(); }

  ThreadPool(const ThreadPool&) = delete;
  ThreadPool& operator=(const ThreadPool&) = delete;

  // 投递一个任务并唤醒一个空闲 worker。
  template <typename F>
  void Submit(F&& f);

  // 停止接收新任务，join 所有 worker（等已投递任务执行完）。
  void Shutdown() {
    {
      std::lock_guard<std::mutex> lock(mu_);
      if (stopping_) return;
      stopping_ = true;
    }
    cv_.notify_all();
    for (std::thread& t : workers_) {
      if (t.joinable()) t.join();
    }
    workers_.clear();
  }

 private:
  void WorkLoop();

  std::vector<std::thread> workers_;
  std::mutex mu_;
  std::condition_variable cv_;
  std::queue<std::function<void()>> tasks_;
  bool stopping_ = false;
};
```
- 线程池等待任务循环：
``` cpp
  void WorkLoop() {
    for (;;) {
      std::function<void()> task;
      {
        std::unique_lock<std::mutex> lock(mu_);
        cv_.wait(lock, [this] { return stopping_ || !tasks_.empty(); });
        if (tasks_.empty()) return;
        task = std::move(tasks_.front());
        tasks_.pop();
      }
      task();  // 锁外执行，避免任务互相串行
    }
  }
```
- 线程池发布任务：
``` cpp
  template <typename F>
  void Submit(F&& f) {
    {
      std::lock_guard<std::mutex> lock(mu_);
      if (stopping_) return;
      tasks_.emplace(std::forward<F>(f));
    }
    cv_.notify_one();
  }
```
### Reactor
``` cpp
class Reactor {
 public:
  Reactor(uint16_t port, size_t workers);
  ~Reactor();

  Reactor(const Reactor&) = delete;
  Reactor& operator=(const Reactor&) = delete;

  void Run();                        // 事件循环，阻塞运行
  void Stop() { stop_.store(true); } // 请求退出，可被任意线程调用

 private:
  // 一条 TCP 连接的读侧状态（只被 Reactor 线程访问）。
  struct Conn;

  // ---- Reactor 线程内调用 ----
  void OnWakeupReadable();              // eventfd 可读：取响应并发送
  void OnListenReadable();              // listen 可读：accept 新连接
  void OnConnReadable(uint64_t cid);    // 连接可读：recv + 拆行 + 投线程池
  void CloseConn(uint64_t cid);         // 从 epoll 摘除并关闭

  // ---- worker 线程内调用（不碰 socket）----
  void RunBusiness(uint64_t cid, std::string request);
  void PostResponse(uint64_t cid, std::string body);

  // ---- Reactor 线程独占数据 ----
  int epfd_ = -1;          // epoll 实例
  int listen_fd_ = -1;     // 监听 socket
  int wake_fd_ = -1;       // eventfd：worker 唤醒 Reactor
  uint64_t next_cid_ = 2;  // 连接编号从 2 起（0/1 保留给 eventfd/listen 标记）
  std::unordered_map<uint64_t, Conn> conns_;  // 记录cid对应连接
  ThreadPool pool_;  // 业务线程池

  // ---- Reactor 与 worker 共享（唯一需要加锁的数据）----
  std::deque<Response> pending_;  // 已完成、待 Reactor 发送的响应
  std::mutex mu_;
  std::atomic<bool> stop_{false};
  uint64_t total_requests_ = 0;  // 收到的请求总数
  uint64_t total_replies_ = 0;   // 成功发出的响应总数
};

Reactor::~Reactor() {
  // 先停线程池并 join：worker 回调会访问 pending_/wake_fd_，必须先结束
  pool_.Shutdown();
  // 剩余连接直接关闭（简化版不负责在退出时补发最后的响应）
  for (auto& kv : conns_) ::close(kv.second.fd);
  conns_.clear();
  ::close(wake_fd_);
  ::close(listen_fd_);
  ::close(epfd_);
}
```
- 关闭连接
	- 删除epoll事件
	- 删除connect表项
	- 关闭connect_fd
``` cpp
void Reactor::CloseConn(uint64_t cid) {
  auto it = conns_.find(cid);
  if (it == conns_.end()) return;
  const int fd = it->second.fd;
  cout << "[close] cid=" << cid << " fd=" << fd << endl;

  ::epoll_ctl(epfd_, EPOLL_CTL_DEL, fd, nullptr);
  conns_.erase(it);
  ::close(fd);
}
```
- 初始化:
	- 构造epoll_fd
	- 构建TCP连接的listen_fd（socket、bind、listen之类的）
	- eventfd调用构建wake_fd
	- 以上事件预先注册进epoll循环
``` cpp
Reactor::Reactor(uint16_t port, size_t workers) : pool_(workers == 0 ? 1 : workers) {
  // 忽略 SIGPIPE（坏连接不炸进程）；SIGINT/SIGTERM 触发优雅退出
  ::signal(SIGPIPE, SIG_IGN);
  ::signal(SIGINT, OnSignal);
  ::signal(SIGTERM, OnSignal);

  epfd_ = ::epoll_create1(EPOLL_CLOEXEC);
  if (epfd_ < 0) { perror("epoll_create1"); exit(EXIT_FAILURE); }

  listen_fd_ = CreateListenSocket(port);

  // eventfd：worker 写好结果后写 1，唤醒阻塞在 epoll_wait 的 Reactor
  wake_fd_ = ::eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
  if (wake_fd_ < 0) { perror("eventfd"); exit(EXIT_FAILURE); }

  // 把 listen 和 eventfd 注册进 epoll（各带一个不会重复的标记）
  auto AddInit = [this](int fd, uint64_t key) {
    epoll_event ev;
    memset(&ev, 0, sizeof(ev));
    ev.events = EPOLLIN;
    ev.data.u64 = key;
    if (::epoll_ctl(epfd_, EPOLL_CTL_ADD, fd, &ev) != 0) {
      perror("epoll_ctl(ADD)");
      exit(EXIT_FAILURE);
    }
  };
  AddInit(wake_fd_, kWakeTag);
  AddInit(listen_fd_, kListenTag);

  cout << "[info] listening on 0.0.0.0:" << port
       << ", worker threads = " << (workers == 0 ? 1 : workers) << " (epoll, simple)"
       << endl;
}

static volatile sig_atomic_t g_stop = 0;
static void OnSignal(int) { g_stop = 1; }

static int CreateListenSocket(uint16_t port) {
  int fd = ::socket(AF_INET, SOCK_STREAM, 0);
  if (fd < 0) { perror("socket"); exit(EXIT_FAILURE); }

  int on = 1;
  ::setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &on, sizeof(on));  // 便于反复重启
  sockaddr_in addr;
  memset(&addr, 0, sizeof(addr));
  addr.sin_family = AF_INET;
  addr.sin_port = htons(port);
  addr.sin_addr.s_addr = htonl(INADDR_ANY);
  if (::bind(fd, (sockaddr*)&addr, sizeof(addr)) != 0) { perror("bind"); exit(EXIT_FAILURE); }
  if (::listen(fd, 128) != 0) { perror("listen"); exit(EXIT_FAILURE); }
  SetNonBlocking(fd);
  return fd;
}
```
- 主循环：
	- epoll_wait循环开启
	- 处理epoll_wait失败和超时重新循环
	- 接收到的事件列表处理：
		- wake_fd：工作线程响应
		- listen_fd：新连接进入
		- connect_fd：
			- 连接错误，关闭连接
			- 连接读取数据
``` cpp
void Reactor::Run() {
  const int kMaxEvents = 64;
  vector<epoll_event> events(kMaxEvents);

  while (!stop_.load() && !g_stop) {
    int n = ::epoll_wait(epfd_, events.data(), kMaxEvents, kEpollTimeoutMs);
    if (n < 0) {
      if (errno == EINTR) continue;
      perror("epoll_wait");
      break;
    }
    if (n == 0) continue;  // 纯超时，周期性检查退出标志

    for (int i = 0; i < n; ++i) {
      const uint64_t key = events[i].data.u64;
      const uint32_t rev = events[i].events;

      if (key == kWakeTag) {          // worker 算完，来取结果发送
        OnWakeupReadable();
        continue;
      }
      if (key == kListenTag) {        // 新连接
        OnListenReadable();
        continue;
      }

      // 普通连接：cid 查不到说明已关闭（旧事件/迟到响应），直接跳过
      if (conns_.find(key) == conns_.end()) continue;
      if (rev & EPOLLERR) {
        CloseConn(key);
      } else if (rev & (EPOLLIN | EPOLLHUP)) {
        OnConnReadable(key);
      }
    }
  }

  cout << "\n===== reactor stopped =====" << endl;
  cout << "  total_requests   = " << total_requests_ << endl;
  cout << "  total_replies    = " << total_replies_ << endl;
  cout << "  remaining_conns  = " << conns_.size() << endl;
}
```
- 响应发送：
	- wake_fd计数读到一次直接清零
	- 直接拷贝消息队列pending_循环处理响应
		- 响应对应连接还在，直接一次send发完，出错直接关闭连接
		- 连接已关闭，直接丢弃
``` cpp
void Reactor::OnWakeupReadable() {
  // eventfd 计数读一次即清零，循环排空到 EAGAIN
  uint64_t v = 0;
  for (;;) {
    ssize_t n = ::read(wake_fd_, &v, sizeof(v));
    if (n > 0) continue;
    if (n < 0 && errno == EINTR) continue;
    break;
  }

  deque<Response> batch;
  {
    lock_guard<mutex> lock(mu_);
    batch.swap(pending_);
  }

  for (Response& r : batch) {
    auto it = conns_.find(r.cid);
    if (it == conns_.end()) continue;  // 连接已关闭：丢弃迟到响应

    // 简化版不做写缓冲：一次 send 发完；EAGAIN/部分发送/出错一律关闭连接
    const string resp = r.body + "\n";
    ssize_t n = ::send(it->second.fd, resp.data(), resp.size(), 0);
    if (n == static_cast<ssize_t>(resp.size())) {
      ++total_replies_;
    } else {
      CloseConn(r.cid);
    }
  }
}
```
- 接收新连接（accept）
	- 设置非阻塞、禁止Nagle算法加速、cid
	- 加入connect表
	- 插入epoll事件（epoll_events里头带cid）
``` cpp
void Reactor::OnListenReadable() {
  for (;;) {
    sockaddr_in peer;
    socklen_t addrlen = sizeof(peer);
    int fd = ::accept(listen_fd_, (sockaddr*)&peer, &addrlen);
    if (fd < 0) {
      if (errno == EINTR) continue;
      if (errno == EAGAIN || errno == EWOULDBLOCK) break;  // 没有排队的了
      perror("accept");
      break;
    }
    SetNonBlocking(fd);
    int one = 1;
    ::setsockopt(fd, IPPROTO_TCP, TCP_NODELAY, &one, sizeof(one));

    uint64_t cid = next_cid_++;
    Conn conn;
    conn.cid = cid;
    conn.fd = fd;
    conns_.emplace(cid, std::move(conn));

    epoll_event ev;
    memset(&ev, 0, sizeof(ev));
    ev.events = EPOLLIN;
    ev.data.u64 = cid;                  // 事件里带 cid，后面按 cid 处理
    ::epoll_ctl(epfd_, EPOLL_CTL_ADD, fd, &ev);

    char ip[INET_ADDRSTRLEN];
    inet_ntop(AF_INET, &peer.sin_addr, ip, sizeof(ip));
    cout << "[accept] cid=" << cid << " fd=" << fd
         << "  from " << ip << ":" << ntohs(peer.sin_port) << endl;
  }
}
```
- 读取连接数据
	- 循环recv将数据拷贝到连接封装的输入缓冲区中
	- 对端如果FIN，直接关闭连接
	- 以\n为界分割出一条请求并提交给线程池
``` cpp
void Reactor::OnConnReadable(uint64_t cid) {
  auto it = conns_.find(cid);
  if (it == conns_.end()) return;
  Conn& conn = it->second;

  char buf[4096];
  for (;;) {
    ssize_t n = ::recv(conn.fd, buf, sizeof(buf), 0);
    if (n > 0) {
      conn.inbuf.append(buf, static_cast<size_t>(n));
      if (conn.inbuf.size() > kMaxInbufSize) { CloseConn(cid); return; }
      continue;  // 非阻塞：把本轮可读数据尽量读完
    }
    if (n == 0) {                 // 对端 EOF（FIN）
      CloseConn(cid);
      return;
    }
    if (errno == EINTR) continue;
    if (errno == EAGAIN || errno == EWOULDBLOCK) break;  // 本轮读完
    CloseConn(cid);
    return;
  }

  it = conns_.find(cid);
  if (it == conns_.end()) return;
  Conn& c = it->second;

  // 拆行：inbuf 里攒到 '\n' 才算一个完整请求，拆出一个投一个
  for (;;) {
    size_t pos = c.inbuf.find('\n');
    if (pos == string::npos) break;            // 半包：留在 inbuf 等下次
    string line = c.inbuf.substr(0, pos);
    c.inbuf.erase(0, pos + 1);
    if (!line.empty() && line.back() == '\r') line.pop_back();  // 兼容 CRLF
    if (line.empty()) continue;

    pool_.Submit([this, cid, request = std::move(line)]() mutable {
      RunBusiness(cid, std::move(request));
    });
    ++total_requests_;
  }
}
```
- 线程池处理请求（工作线程执行）
	- 处理请求（Handler）
	- 封装发送请求（pending_+wake_fd）
``` cpp
void Reactor::RunBusiness(uint64_t cid, string request) {
  string body = HandleRequest(std::move(request));
  if (!body.empty()) PostResponse(cid, std::move(body));
}

static string HandleRequest(string raw) {
  string line = Trim(std::move(raw));
  if (line.empty()) return "";
  if (line == "ping") return "pong";

  if (StartsWith(line, "sleep ")) {  // 模拟耗时业务
    int ms = atoi(line.c_str() + 6);
    if (ms < 0) ms = 0;
    if (ms > 10000) ms = 10000;
    this_thread::sleep_for(chrono::milliseconds(ms));
    return "slept " + to_string(ms) + " ms";
  }
  if (StartsWith(line, "fib ")) {    // 模拟 CPU 密集业务
    int n = atoi(line.c_str() + 4);
    if (n < 0) n = 0;
    if (n > 46) n = 46;
    unsigned long long a = 0, b = 1;
    for (int i = 0; i < n; ++i) { unsigned long long t = a + b; a = b; b = t; }
    return "fib(" + to_string(n) + ") = " + to_string(a);
  }
  if (StartsWith(line, "upper ")) {
    string t = line.substr(6);
    transform(t.begin(), t.end(), t.begin(), ::toupper);
    return "upper: " + t;
  }
  return "echo: " + line;
}

void Reactor::PostResponse(uint64_t cid, string body) {
  {
    lock_guard<mutex> lock(mu_);
    pending_.push_back(Response{cid, std::move(body)});
  }
  // 简化版：每个完成的请求都唤醒一次 Reactor（不合并批量）
  uint64_t one = 1;
  ssize_t n = ::write(wake_fd_, &one, sizeof(one));
  (void)n;
}
```
## 4. 优点缺点
### 优点
1. **实现简单**
2. **慢业务不阻塞 IO**：耗时/阻塞计算全部在 worker 线程
3. **共享数据少临界区小**
### 缺点
1. **无发送缓冲（outbuf + EPOLLOUT）**：响应一次 `send` 发不完（对端接收慢、
   内核发送缓冲满、大响应）就直接关闭连接
2. **FIN 即关连接**：读到 `recv==0` 立即关闭，丢弃剩余响应
3. **唤醒未合并**：高频小请求下，每个请求都触发一次 `eventfd` 写 + 唤醒，系统调用开销偏高；
4. 占着不发的连接会一直占用 cid 与 epoll 条目；
5. **单 Reactor 单点风险**：所有 IO 集中在一个线程，成为吞吐上限（这是该模型的固有边界）。
## 5. 可扩展方向

| #   | 扩展             | 怎么做                                                                                             | 解决什么问题         |
| --- | -------------- | ----------------------------------------------------------------------------------------------- | -------------- |
| 1   | 写缓冲 + EPOLLOUT | `Conn` 增加 `outbuf`；发送时若 `EAGAIN` 保留数据，并为连接注册 `EPOLLOUT`；可写事件里续发，发完注销                            | 大响应 / 慢客户端不再断连 |
| 2   | EOF 在途保护       | `Conn` 增加 `peer_closed`、`inflight`：EOF 只标记；每条响应发出 `inflight--`；`peer_closed && inflight==0` 才关闭 | 客户端半关闭不再丢响应    |
| 3   | 批量唤醒           | `PostResponse` 只在 `pending_` 由空变非空时写 eventfd；唤醒后 `swap` 整批取走                                    | 减少高频小请求的唤醒次数   |
| 4   | epoll 边缘触发(ET) | 注册 `EPOLLET`，读事件必须一次读到 `EAGAIN`（当前读循环已满足）                                                       | 降低就绪事件重复上报     |
| 5   | 连接超时/心跳        | 定时器或时间轮定期扫描 `last_active`，超时踢除                                                                  | 防僵尸连接占用资源      |
| 6   | 线程池增强          | 有界任务队列 + 拒绝/降级策略；核心/最大线程                                                                        | 背压与资源保护        |
| 7   | 协议通用化          | `HandleRequest` 换成命令路由/回调注册 + 帧编解码                                                              | 从演示到通用服务       |
| 8   | io_uring       | 把 `read/write` 也异步化（Linux 5.1+）                                                                 | 进一步减少系统调用      |

## 6. 开发过程中的问题与解决方法
| #   | 问题         | 原因                              | 解决方法                                                             |
| --- | ---------- | ------------------------------- | ---------------------------------------------------------------- |
| 1   | 单连接阻塞全服务   | 在事件循环线程里做阻塞式 `recv/send/accept` | 所有 fd 设 `O_NONBLOCK`；无数据立即 `EAGAIN`，靠 `epoll_wait` 驱动            |
| 2   | SIGPIPE信号坑 | `SIGPIPE` 默认杀进程                 | `signal(SIGPIPE, SIG_IGN)`，让 `send` 返回 `EPIPE` 走正常清理             |
| 3   | 粘包         | 没有消息边界                          | 每连接 `inbuf` 攒数据，`find('\n')` 才算完整请求                              |
| 4   | 大响应        | 内核发送缓冲满                         | 最简版选择**关连接止损**；若需支持大响应，加 outbuf + `EPOLLOUT`                     |
| 5   | 关闭顺序错误     | 析构顺序错误                          | 先 `pool_.Shutdown()`（join worker）再析构 `pending_`/`wake_fd_` 等共享成员 |
| 6   | 连接fd复用     | epoll 返回的是就绪快照                  | 处理每条事件都用 cid 反查 `conns_`，查不到即跳过                                  |
## 7. 有用知识点
- listenfd的理解：监听套接字，只负责**被动等待客户端 connect**，本身不传输数据，客户端发起 `connect()` → 内核 TCP 三次握手完成，连接放到 listenfd 的**半连接 / 全连接队列**，用户进程调用 `accept(listenfd)`，内核从全连接队列取出已完成握手的连接，**生成全新的 connectionfd**
- `eventfd`：虚拟的fd，让工作线程能唤醒epoll_wait循环，epoll自带，替代管道
- `std::lock_guard`与`std::unique_lock`的区别：
	- `std::lock_guard`：自动管理互斥锁，出作用域自动析构解锁，不支持手动lock/unlock
	- `std::unique_lock`：需要手动lock/unlock，常常配合条件变量cv一起使用，而且cv的wait一般会使用带谓词版本CAS避免假唤醒
- 中途被信号中断继续：`if (errno == EINTR) continue;`
- 信号量广播：大都是退出时`cv_.notify_all();`
- 防止端口占用：setsockopt SO_REUSEADDR防止TIME_WAIT（旧进程已经完全死掉，但端口残留 TIME_WAIT，打开后可以直接bind，如果是旧进程还活着，正常占用端口，会bind失败）
- 关 Nagle 提高性能：setsockopt TCP_NODELAY
- fd一般都要设置成非阻塞的
- 常见信号：无数据时 recv/send/accept 立即返回 EAGAIN
- CRLF规则：换行符
	- Windows：**CRLF = \r\n**
	- Linux /macOS (新版)：**LF = \n**
	- 旧 Mac（Mac OS9）：CR = \r（现在几乎见不到）