---
title: 哈夫曼树与哈夫曼编码器（python实现）
tags:
  - 数据结构
  - 算法
url: 26.html
id: 26
categories:
  - 算法
date: 2016-08-23 19:38:40
---

![](http://7xqgks.com1.z0.glb.clouddn.com/bd831b25b5ba1b236a4fa05e0abe01b1f3c0c68124577-hJCX6a.jpg)

# 1.关于哈夫曼树

哈夫曼树基于加权二叉树，其基本要素包含左右子树、双亲、权重和编码。

    class Node:
        def  __init__(self,right=None,left=None,parent=None,weight=0,charcode=None):
        self.right = right
        self.left = left
        self.parent = parent
        self.weight = weight
        self.charcode = charcode


# 2.哈弗曼算法

哈弗曼算法属于贪心法的一种，其基本思路是：编码以字符出现的频率作为权重，每次选权重最小的两个节点作为生成最优二叉树的左右孩子，并将权重之和作为根节点的权重，自底向上生成一颗带权路径长度最短的最优二叉树。

    def sort(list):
        return sorted(list,key=lambda node:node.weight)

    def Huffman(listOfNode):
        listOfNode = sort(listOfNode)
        while len(listOfNode) != 1:
            a,b = listOfNode[0],listOfNode[1]
            new = Node()
            new.weight, new.left, new.right = a.weight + b.weight, a, b
            a.parent, b.parent = new, new
            listOfNode.remove(a), listOfNode.remove(b)
            listOfNode.append(new)
            listOfNode = sort(listOfNode)
        return listOfNode


# 3.导入字符-权重文件（非算法思想部分）

为方便操作，将字符-权重字典保存为文本文件后，直接导入进行编码，使用Python文件曹组完成。

    def inPutFile():
        global filename
        global listForEveryByte
        filename = raw_input("请输入要编码的文件(存放在源代码目录下)：")
        global codeDict
        with open(filename,'rb') as f:
            data = f.read()
            for Byte in data:
                codeDict.setdefault(Byte,0) #每个字节出现的次数默认为0
                codeDict[Byte] += 1
                listForEveryByte.append(Byte)

    def outputCompressedFile():
        global listForEveryByte
        fileString = ""
        with open(filename.split(".")[0]+".jbj","wb") as f:
            for Byte in listForEveryByte:
                fileString += encodeDict[Byte]  #构成一个长字符序列
            leng = len(fileString)
            more = 16-leng%16
            fileString = fileString+"0"*more          #空位用0补齐

            leng = len(fileString)
            i,j = 0,16
            while j <= leng:
                k = fileString[i:j]
                a = int(k,2)
                print(a)
                print(repr(struct.pack(">H",a)))
                f.write(struct.pack(">H",a))
                f.write(str(a))
                i=i+16
                j=j+16


# 4.编码

通过哈弗曼算法构造哈夫曼树后，编码过程为找到字符所在的叶子节点，以及从根节点到该叶子节点的路径，使用先序遍历的节点左树记为0，右树记为1，即可得到编码结果。

    def encode(head,listOfNode):
        global encodeDict
        for e in listOfNode:
            ep = e
            encodeDict.setdefault(e.charcode,"")
            while ep != head:

                if ep.parent.left == ep:
                    encodeDict[e.charcode] = "1"+encodeDict[e.charcode]
                else:
                    encodeDict[e.charcode] = "0"+encodeDict[e.charcode]
                ep=ep.parent


# 5.执行算法与所得结果

    if __name__ == '__main__':
        inPutFile()
        listOfNode = []
        for e in codeDict.keys():
            listOfNode.append(Node(weight=codeDict[e],charcode=e))
        head=Huffman(listOfNode)[0]
        encode(head,listOfNode)

        for i in encodeDict.keys():
            print(i,encodeDict[i])