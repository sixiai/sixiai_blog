# Matlab中的TCP和串口(uart)服务

> 📅 2025-11-08 | 🏷️ matlab

## 简介

在嵌入式开发中，经常需要用 Matlab 与硬件进行通信。本文介绍如何在 Matlab 中实现 TCP 和串口通信。

## 串口通信

### 创建串口对象

```matlab
% 新版本 Matlab (R2019b+)
s = serialport("COM3", 115200);

% 旧版本 Matlab
s = serial("COM3", "BaudRate", 115200);
fopen(s);
```

### 配置参数

```matlab
s.DataBits = 8;
s.StopBits = 1;
s.Parity = "none";
s.FlowControl = "none";
s.Timeout = 10;  % 超时时间（秒）
```

### 发送数据

```matlab
% 发送字符串
write(s, "Hello STM32", "string");

% 发送字节数组
data = [0x01, 0x02, 0x03, 0x04];
write(s, data, "uint8");

% 发送浮点数
write(s, 3.14159, "single");
```

### 接收数据

```matlab
% 读取指定数量的字节
data = read(s, 10, "uint8");

% 读取一行
line = readline(s);

% 读取所有可用数据
data = read(s, s.NumBytesAvailable, "uint8");
```

### 回调函数

```matlab
% 设置数据接收回调
configureCallback(s, "byte", 10, @serialCallback);

function serialCallback(src, ~)
    data = read(src, src.NumBytesAvailable, "uint8");
    disp("Received: " + num2str(data));
end
```

## TCP 通信

### TCP 客户端

```matlab
% 创建 TCP 客户端
client = tcpclient("192.168.1.100", 8080);

% 发送数据
write(client, "Hello Server");

% 接收数据
data = read(client, client.NumBytesAvailable);

% 关闭连接
clear client;
```

### TCP 服务器

```matlab
% 创建 TCP 服务器
server = tcpserver("0.0.0.0", 8080);

% 等待客户端连接
while ~server.Connected
    pause(0.1);
end
disp("Client connected!");

% 接收数据
data = read(server, server.NumBytesAvailable, "string");

% 发送响应
write(server, "ACK");
```

### 完整示例：实时数据采集

```matlab
function realtime_acquisition()
    % 创建串口连接
    s = serialport("COM3", 115200);
    configureTerminator(s, "LF");
    
    % 创建图形窗口
    figure;
    h = animatedline;
    ax = gca;
    ax.YLim = [0, 4096];
    xlabel("Sample");
    ylabel("ADC Value");
    
    % 数据缓冲区
    buffer = zeros(1, 1000);
    idx = 1;
    
    % 开始采集
    write(s, "START", "string");
    
    % 实时绘图
    while ishandle(h)
        if s.NumBytesAvailable > 0
            line = readline(s);
            value = str2double(line);
            
            if ~isnan(value)
                addpoints(h, idx, value);
                buffer(mod(idx-1, 1000)+1) = value;
                idx = idx + 1;
                drawnow limitrate;
            end
        end
    end
    
    % 停止采集
    write(s, "STOP", "string");
    clear s;
end
```

## 常见问题

### 1. 串口打开失败

```matlab
% 查看可用串口
serialportlist("available")

% 强制关闭所有串口
delete(instrfindall);
```

### 2. 数据乱码

确保波特率、数据位、停止位、校验位与设备一致。

### 3. 接收不完整

```matlab
% 增加超时时间
s.Timeout = 30;

% 或使用循环等待
while s.NumBytesAvailable < expectedBytes
    pause(0.01);
end
```

## 总结

Matlab 提供了方便的串口和 TCP 通信接口，非常适合用于嵌入式系统的调试和数据采集。
