## Node.js如何实现异步非阻塞
Nodejs利用线程池(libuv threadPool)执行IO调用，避免阻塞主线程，执行结束后的结果和请求放入一个队列，再利用事件循环(eventloop)取出队列中的请求，最后执行回调，达到异步非阻塞的效果。