# AB测试

命令行

`ab -c 100 -n 10000 http://127.0.0.1:9501/`

执行结果如下

    Server Software:        swoole-http-server
    Server Hostname:        127.0.0.1
    Server Port:            9501

    Document Path:          /
    Document Length:        348 bytes

    Concurrency Level:      100
    Time taken for tests:   0.883 seconds
    Complete requests:      10000
    Failed requests:        168
    (Connect: 0, Receive: 0, Length: 168, Exceptions: 0)
    Total transferred:      4914560 bytes
    HTML transferred:       3424728 bytes
    Requests per second:    11323.69 [#/sec] (mean)
    Time per request:       8.831 [ms] (mean)
    Time per request:       0.088 [ms] (mean, across all concurrent requests)
    Transfer rate:          5434.67 [Kbytes/sec] received

    Connection Times (ms)
                min  mean[+/-sd] median   max
    Connect:        0    0   0.2      0       2
    Processing:     0    9   9.6      6      96
    Waiting:        0    9   9.6      6      96
    Total:          0    9   9.6      6      96

    Percentage of the requests served within a certain time (ms)
    50%      6
    66%      9
    75%     11
    80%     12
    90%     19
    95%     27
    98%     43
    99%     51
    100%     96 (longest request)
