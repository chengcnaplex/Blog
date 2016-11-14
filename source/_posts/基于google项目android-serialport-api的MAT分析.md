---
title: 基于google项目android-serialport-api的MAT分析
date: 2016-11-09 14:58:19
tags:
  - Android
  - MAT
categories:
  - Android
---
先看一段google code的一段串口代码，在这段代码有这么一行**mInputStream.read(buffer)**，这是输入流的阻塞方法。如果mInputStream.read(buffer)未接收到数据，就重复关闭开启含有这段代码的Activity.就会造成Activity泄露，泄露现象通过重复开启关闭Activity,记录每一次的内存快照，再用MAT进行分析.

```java
	private class ReadThread extends Thread {
	        @Override
	        public void run() {
	            super.run();
	            while(!isInterrupted()) {
	                int size;
	                try {
	                    byte[] buffer = new byte[64];
	                    if (mInputStream == null) return;
	                    size = mInputStream.read(buffer);
	                    if (size > 0) {
	                        onDataReceived(buffer, size);
	                    }
	                    SystemClock.sleep(100);
	                } catch (IOException e) {
	                    e.printStackTrace();
	                    return;
	                }
	            }
	        }
	    }
```
<!-- more -->	

解决方法很简单，添加一个mInputStream.available() > 0，如果有数据，就进行读取。没有数据就不读取。这样就避开了阻塞问题。
 
```java
	private class ReadThread extends Thread {
	    @Override
	    public void run() {
	        super.run();
	        while(!isInterrupted()) {
	            int size;
	            try {
	                byte[] buffer = new byte[64];
	                if (mInputStream == null) return;
	                if (mInputStream.available() > 0) {
	                    size = mInputStream.read(buffer);
	                    if (size > 0) {
	                        onDataReceived(buffer, size);
	                    }
	                }
	                SystemClock.sleep(100);
	            } catch (IOException e) {
	                e.printStackTrace();
	                return;
	            }
	        }
	    }
	}
```
	