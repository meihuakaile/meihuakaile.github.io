---
title: 朴素贝叶斯（二）实现NBCorpus分类（附代码和数据）
tags: ['机器学习', '朴素贝叶斯', 'NBCorpus']
categories: ['机器学习']
copyright: true
---
理论可参考 ： [ 朴素贝叶斯（一） ](http://blog.csdn.net/u010668907/article/details/79399972)

公式： ![](http://wiki.corp.qunar.com/confluence/download/thumbnails/191162695/im
age2018-2-28_11-53-25.png?version=1&modificationDate=1519790006000&api=v2)
（P(x)为常数，可忽略不考虑）

平滑： ![](http://wiki.corp.qunar.com/confluence/download/thumbnails/191162695/im
age2018-2-28_11-57-47.png?version=1&modificationDate=1519790267000&api=v2)

N  y  k  是类别为  y  k  的样本个数，n是特征的维数，  N  y  k  ,  x  i  是类别为  y  k
的样本中，第i维特征的值是  x  i  的样本个数，  α  是平滑值。

  

在对NBCorpus词分类时，带入上面的公式可得：

某词属于某类别的概率 = （该类别该词的个数  + 1/ 该类别词的总数 + 所有类别所有不重复单词总数） ×（该类别样本个数 / 所有类别总样本个数） /
（所有类别该词个数 / 所有类别所有词个数）

所有类别该词个数 / 所有类别所有词个数   就是上面公式的P(x) ，它的值与类别无关，值相同，所有把它去掉，公式变成：

_ ** （该类别该词的个数  + 1/ 该类别词的总数 + 所有类别所有不重复单词总数） ×（该类别样本个数 / 所有类别总样本个数） ** _

（该类别样本个数 / 所有类别总样本个数） 就是上面公式的P(yk)，即先验概率。没有使用平滑。

（该类别该词的个数  + 1/ 该类别词的总数 + 所有类别所有不重复单词总数） 就是上面公式的P(x|yk)，就是可能性。使用了平滑，分子加的1，分母加的
所有类别所有不重复单词总数  都是平滑值。

如果按照上面的公式，就和上面的例子一样，对于测试的样本里的每个属性计算出属于每个类别的概率，对于每个类别每个属性的概率相乘。但是，小数越乘越小，最终可能会导
致数据会丢失成0，无法比较。解决办法是对公式加ln操作，把乘法变成加法，且不破坏单调性。

分词简单例子：

![](http://wiki.corp.qunar.com/confluence/download/attachments/191162695/image
2018-2-28_14-8-11.png?version=1&modificationDate=1519798091617&api=v2)

![](http://wiki.corp.qunar.com/confluence/download/attachments/191162695/image
2018-2-28_14-8-22.png?version=1&modificationDate=1519798102000&api=v2)

![](http://wiki.corp.qunar.com/confluence/download/attachments/191162695/image
2018-2-28_14-8-35.png?version=1&modificationDate=1519798115335&api=v2)

上面的词量比较少，没有进行ln操作。但是如果词量太大，举例最后的计算变成 ln(3/4 * (3/7)^3 * 1/14 * 1/14 ) =
ln(3/4) + 3*ln(3/7) + ln(1/14) + ln(1/14) 。

数据下载地址：http://download.csdn.net/download/u010668907/10263175

。。。。无语了，想上传全部的NBCorpus库，csdn一直说已经存在，整体的库后续再说吧，上面的链接里的是做下面算法计算的那部分，够用了。

下面的代码里加了ln操作，否则计算之后每个类别都是0，无法比较

    
    
    # -*- coding:utf-8 -*-
    # __author__='chenliclchen'
    from __future__ import division
    import os
    import pickle
    
    
    # 统计每个类别每个单词出现的次数
    # （该类别文档数） / （所有类别文档数）
    # （该类别该单词个数 + 1）/（该类别单词个数 + 所有类别无重复单词个数）
    
    # step1，统计每个类别每个单词出现的次数
    def count_every_sort_every_word(sort_path):
        count = {}
        for item in os.listdir(sort_path):
            file_path = os.path.join(sort_path, item)
            with open(file_path) as file:
                words = file.readlines()
            for word in words:
                word = word.replace('\r\n', '')
                if word in count.keys():
                    count[word] += 1
                else:
                    count[word] = 1
        return count
    
    
    # 保存step1的结果
    def save_every_sort_every_word(count_dict, sort_path):
        file = open(sort_path, 'wb')
        pickle.dump(count_dict, file)
        file.close()
    
    
    # 计算所有类别不重复单词总数， 总词数包括重复
    def count_all_words(count_path):
        all_words_no_repeat = set()
        all_words_count = 0
        for item in os.listdir(count_path):
            file = open(os.path.join(count_path, item), 'rb')
            one_sort = pickle.load(file)
            all_words_no_repeat.update(one_sort.keys())
            for key in one_sort.keys():
                all_words_count += one_sort[key]
    
        return len(all_words_no_repeat), all_words_count
    
    
    # 每个类别样本数量
    def count_every_sort_docu(sort_path):
        count_docu = {}
        all_docu_num = 0
        for item in os.listdir(sort_path):
            one_sort_path = os.path.join(sort_path, item)
            count_docu[item] = len(os.listdir(one_sort_path))
            all_docu_num += count_docu[item]
        count_docu['all'] = all_docu_num
        return count_docu
    
    
    # 返回某个类别所有词的数据量
    def count_one_sort(sort_path):
        all_words_count = 0
        file = open(sort_path, 'rb')
        one_sort = pickle.load(file)
        file.close()
        for key in one_sort.keys():
            all_words_count += one_sort[key]
        return all_words_count
    
    
    def get_test_words(test_path):
        file = open(test_path)
        words = file.readlines()
        file.close()
        for ind, word in enumerate(words):
            words[ind] = word.replace('\r\n', '')
        return words
    
    
    if __name__ == '__main__':
        base_path = './NBCorpus'
        sort_path = os.path.join(base_path, 'train')
        count_path = os.path.join(base_path, 'count')
        # 计算并保存step1的数据
        for one_sort in os.listdir(sort_path):
            one_sort_path = os.path.join(sort_path, one_sort)
            one_sort_count = count_every_sort_every_word(one_sort_path)
            sort_name = os.path.split(one_sort_path)
            one_count_path = os.path.join(count_path, sort_name[-1])
            save_every_sort_every_word(one_sort_count, one_count_path)
    
        sort_prob = {}
        test_path = os.path.join(base_path, 'test', '487141newsML.txt')
        test_words = get_test_words(test_path)
        all_words_num, all_words_num_repeat = count_all_words(count_path)
        every_docu = count_every_sort_docu(sort_path)
        for item in os.listdir(count_path):
            one_sort_count_path = os.path.join(count_path, item)
            one_sort_num = count_one_sort(one_sort_count_path)
            file = open(one_sort_count_path, 'rb')
            one_sort_count = pickle.load(file)
            prior = every_docu[item] / every_docu['all']
            import math
            # 加入ln操作
            all_words_prob = math.log(prior)
            for word in test_words:
                word_num = one_sort_count.get(word, 0)
                word_prob = (word_num + 1) / (one_sort_num + all_words_num)
                # all_words_prob *= word_prob
                # 加入ln操作
                all_words_prob += math.log(word_prob)
            sort_prob[item] = all_words_prob
            file.close()
        print filter(lambda x:max(sort_prob.values()) == sort_prob[x], sort_prob)[0]

最终的输出结果就是 ：AFRICA  
  

