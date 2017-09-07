Apache RocketMQ背后的设计思路与最佳实践
https://yq.aliyun.com/articles/71889
https://issues.apache.org/jira/browse/ROCKETMQ-17
如果将分布式系统的领域按照分布式通信、分布式存储、分布式计算以及分布式管理这四大部分进行划分，其实就会发现在这四大技术栈下有很多的子技术。这里列举几个简单的例子，比如在分布式通信领域下对于网络协议的选择上面，像RocketMQ是基于TCP的，而基于RocketMQ内核的Aliware MQ是有基于HTTP协议的网关，同时也提供了MQTT协议网关。除了像网络协议的选择之外，还会有I/O的模型，这也属于分布式通信领域一个非常经典的问题。通常情况下会将I/O模型分成四种，第一种是所谓的Boss与Worker模型；还有一种是Only Boss模型，所谓的Select线程和事件真正处理的线程都统一在Boss里面，这样就减少了线程切换时产生的上下文开销；第三种I/O模型就是将第一种与第二种I/O模型进行了整合，并利用了统计学原理，在某些比如像内存负载比较重的场景下，从模型一切换到模型二，也就成了所谓的Dynamic Model。第四种I/O模型其实并不常见，它在开源领域有一个与Netty齐名的传输层框架Grizzly中会使用到，叫做Leader/Fellow框架，Leader/Fellow其实是将Select线程这部分的选择权交给了Worker，而在Selector线程里处理IO事件。聊完了分布式计算领域再来谈一谈分布式存储领域，这其实也是分布式领域最复杂，最难攻克的领域，包括结构化存储、半结构化、非结构化存储等。除此之外，在分布式计算领域，主要会涉及Streaming计算、图计算等相关的内容。而在分布式管理方面，也会涉及到很多子技术，包括分布式数据管理，这里面会牵扯到数据多副本的问题，还有分布式一致性、协调等，这些都属于分布式管理相关领域。

 C++多核高级编程 - 09 并发模型之 一 Boss-Worker
http://blog.csdn.net/wangzhiyu1980/article/details/7879904

发送普通消息（三种方式）
https://help.aliyun.com/document_detail/29547.html?spm=5176.doc49323.6.588.k6Q9zV

EQueue - 一个C#写的开源分布式消息队列的总体介绍
http://www.cnblogs.com/netfocus/p/3595410.html

Faster parallel processing in Java using Streams and a spliterator
https://www.airpair.com/java/posts/parallel-processing-of-io-based-data-with-java-streams

Java8里面的java.util.Spliterator接口有什么用？
https://segmentfault.com/q/1010000007087438

https://my.oschina.net/ericquan8/blog/817046
服务端的高并发读写主要利用Linux操作系统的PageCache特性，通过Java的MappedByteBuffer直接操作PageCache。MappedByteBuffer能直接将文件直接映射到内存，其实就是Map把文件的内容被映像到计算机虚拟内存的一块区域，这样就可以直接操作内存当中的数据而无需操作的时候每次都通过I/O去物理硬盘写文件的。

http://blog.csdn.net/xxxxxx91116/article/details/50333161
双缓冲
可以起2个队列，一个读，一个写，这两个队列不断交换；这样可以解决一个问题，写和读不会出现并发问题，不需要上锁，能提高效率，但是仅仅适用于对put顺序没有要求的情况

https://github.com/apache/incubator-rocketmq/blob/master/store/src/main/java/org/apache/rocketmq/store/CommitLog.java

https://github.com/apache/incubator-rocketmq/blob/master/store/src/main/java/org/apache/rocketmq/store/MappedFile.java


    private void init(final String fileName, final int fileSize) throws IOException {
        this.fileName = fileName;
        this.fileSize = fileSize;
        this.file = new File(fileName);
        this.fileFromOffset = Long.parseLong(this.file.getName());
        boolean ok = false;

        ensureDirOK(this.file.getParent());
            this.fileChannel = new RandomAccessFile(this.file, "rw").getChannel();
            this.mappedByteBuffer = this.fileChannel.map(MapMode.READ_WRITE, 0, fileSize);
            TOTAL_MAPPED_VIRTUAL_MEMORY.addAndGet(fileSize);
            TOTAL_MAPPED_FILES.incrementAndGet();
            ok = true;
    }

    public boolean appendMessage(final byte[] data) {
        int currentPos = this.wrotePosition.get();

        if ((currentPos + data.length) <= this.fileSize) {
                this.fileChannel.position(currentPos);
                this.fileChannel.write(ByteBuffer.wrap(data));
            this.wrotePosition.addAndGet(data.length);
            return true;
        }

        return false;
    }

        public AppendMessageResult doAppend(final long fileFromOffset, final ByteBuffer byteBuffer, final int maxBlank, final MessageExtBrokerInner msgInner) {
            // STORETIMESTAMP + STOREHOSTADDRESS + OFFSET <br>

            // PHY OFFSET
            long wroteOffset = fileFromOffset + byteBuffer.position();

            this.resetByteBuffer(hostHolder, 8);
            String msgId = MessageDecoder.createMessageId(this.msgIdMemory, msgInner.getStoreHostBytes(hostHolder), wroteOffset);

            // Record ConsumeQueue information
            keyBuilder.setLength(0);
            keyBuilder.append(msgInner.getTopic());
            keyBuilder.append('-');
            keyBuilder.append(msgInner.getQueueId());
            String key = keyBuilder.toString();
            Long queueOffset = CommitLog.this.topicQueueTable.get(key);
            if (null == queueOffset) {
                queueOffset = 0L;
                CommitLog.this.topicQueueTable.put(key, queueOffset);
            }

            /**
             * Serialize message
             */
            final byte[] propertiesData =
                msgInner.getPropertiesString() == null ? null : msgInner.getPropertiesString().getBytes(MessageDecoder.CHARSET_UTF8);

            final int propertiesLength = propertiesData == null ? 0 : propertiesData.length;

            if (propertiesLength > Short.MAX_VALUE) {
                log.warn("putMessage message properties length too long. length={}", propertiesData.length);
                return new AppendMessageResult(AppendMessageStatus.PROPERTIES_SIZE_EXCEEDED);
            }

            final byte[] topicData = msgInner.getTopic().getBytes(MessageDecoder.CHARSET_UTF8);
            final int topicLength = topicData.length;

            final int bodyLength = msgInner.getBody() == null ? 0 : msgInner.getBody().length;

            final int msgLen = calMsgLength(bodyLength, topicLength, propertiesLength);

            // Determines whether there is sufficient free space
            if ((msgLen + END_FILE_MIN_BLANK_LENGTH) > maxBlank) {
                this.resetByteBuffer(this.msgStoreItemMemory, maxBlank);
                // 1 TOTALSIZE
                this.msgStoreItemMemory.putInt(maxBlank);
                // 2 MAGICCODE
                this.msgStoreItemMemory.putInt(CommitLog.BLANK_MAGIC_CODE);
                // 3 The remaining space may be any value
                //

                // Here the length of the specially set maxBlank
                final long beginTimeMills = CommitLog.this.defaultMessageStore.now();
                byteBuffer.put(this.msgStoreItemMemory.array(), 0, maxBlank);
                return new AppendMessageResult(AppendMessageStatus.END_OF_FILE, wroteOffset, maxBlank, msgId, msgInner.getStoreTimestamp(),
                    queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
            }

            // Initialization of storage space
            this.resetByteBuffer(msgStoreItemMemory, msgLen);
            // 1 TOTALSIZE
            this.msgStoreItemMemory.putInt(msgLen);
            // 2 MAGICCODE
            this.msgStoreItemMemory.putInt(CommitLog.MESSAGE_MAGIC_CODE);
            // omit for breavity
            // 16 TOPIC
            this.msgStoreItemMemory.put((byte) topicLength);
            this.msgStoreItemMemory.put(topicData);
            // 17 PROPERTIES
            this.msgStoreItemMemory.putShort((short) propertiesLength);
            if (propertiesLength > 0)
                this.msgStoreItemMemory.put(propertiesData);

            final long beginTimeMills = CommitLog.this.defaultMessageStore.now();
            // Write messages to the queue buffer
            byteBuffer.put(this.msgStoreItemMemory.array(), 0, msgLen);

            AppendMessageResult result = new AppendMessageResult(AppendMessageStatus.PUT_OK, wroteOffset, msgLen, msgId,
                msgInner.getStoreTimestamp(), queueOffset, CommitLog.this.defaultMessageStore.now() - beginTimeMills);
            return result;
        }
