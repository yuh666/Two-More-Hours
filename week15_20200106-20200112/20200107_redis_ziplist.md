## 压缩列表场景

![image-20200107063118113](https://note.youdao.com/yws/public/resource/4762addbbb207565dafe6a1264ea04a1/xmlnote/EACDB2A218E54528982EFB406B856FE0/9010)

1. 当一个列表键只包含少量的列表项时,并且每个列表要么是小整数和长度短的字符串,redis通过压缩列表来做列表键的底层实现
2. 如上图,字典压缩列表hashmap01 zset的压缩链表zset

## 压缩列表格式

![image-20200107064019335](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/9C1E98609513422C9EA33F4C51F7CC96/9043)

1. zlbytes:指压缩列表占的内存字节数
2. zltail:代表列表的尾指针到头指针的长度,通过头指针(zlbytes)+zltail得到尾指针的位置
3. zlend:压缩列表的结尾特殊值:255
4. Entry1....entry3:压缩列表的节点元素,节点分为三部分
   - previous_entry_length:如果前一个节点长度小于254字节,则通过1个字节存储前一个节点的长度,如果大于254个字节,通过5个字节来存储长度,其中第一个为0XFE(254),后4个字节为前一个节点的长度,这样计算前一个节点的地址 preEntry = currentEntry-previous_entry_length
   - encoding:当前节点元素使用的编码
   - content:存储节点的内容

![image-20200107064859834](https://note.youdao.com/yws/public/resource/0913f7b1edd0a739cddfc0fb16cfac5b/xmlnote/B3AFABBDA7274147ADBCD422AF241B61/9083)

## 连续更新

1. 比如节点e1-eN对应的节点字节数都在250-253个字节,如果新加入的节点大于254个字节,此时会加入到头部节点的位置;当时由于e1存储的previous_entry_length为1则存储不下需要通过5个字节存储同理存储5个节点后e2存储不下e1依次类推到eN节点,每次分配的最坏时间复杂度为O(N)  N次则O(N^2)

