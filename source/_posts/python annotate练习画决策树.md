---
title: 'python annotate练习画决策树'
date: "2018/05/04"
tags: [python, 机器学习]
categories: [机器学习]
copyright: true
---
# annotate 画箭头例子
```python
# -*- coding:utf-8 -*-
# __author__='chenliclchen'

from matplotlib import pyplot as plt

def plot_node(node_text, parent_pt, center_parent, nodetype):
    # s 标签； xy 注释的位置（就是箭头的起始位置）；xytext 标签的位置；
    # xycoords 注释位置（xy）坐标的类型；textcoords 标签位置(xytext)坐标的类型；arrowprops 字典，箭头的属性
    # https: // matplotlib.org / users / annotations.html
    ax.annotate(node_text, xy=parent_pt, xytext=center_parent,
                xycoords='axes fraction', textcoords='axes fraction',
                va='center', ha='center', bbox=nodetype, arrowprops=arrow_args)

tree = {'no surfacing': {0: 'no', 1: {'flippers': {0: 'no', 1: 'yes'}}}}
decisionNode = dict(boxstyle="sawtooth", fc="0.8")
leafNode = dict(boxstyle="round4", fc="0.8")
arrow_args = dict(arrowstyle="<-")
# 1(num)是作为一个id，如果这个id已经存在就激活返回引用否则就创建返回引用
# figsize 元组，图片大小，宽高; dpi 图片分辨率； facecolor 是背景色; edgecolor 边框色；
fig = plt.figure(1, facecolor='w')
fig.clf()
# frameon=False 是否显示坐标轴架构
ax = plt.subplot(111, frameon=False)
plot_node('de', (0.1, 0.5), (0.5, 0.1), decisionNode)
plot_node('leaf', (0.3, 0.8), (0.8, 0.1), leafNode)
plt.show()
```
效果：![](1.png)

# annotate 画决策树
```python
# -*- coding:utf-8 -*-
# __author__='chenliclchen'

from matplotlib import pyplot as plt

# 画每个节点
def plot_node(node_text, parent_pt, center_parent, nodetype):
    # s 标签； xy 注释的位置（就是箭头的起始位置）；xytext 标签的位置；
    # xycoords 注释位置（xy）坐标的类型； textcoords 标签位置(xytext)坐标的类型； arrowprops 字典，箭头的属性
    ax.annotate(node_text, xy=parent_pt, xytext=center_parent,
                xycoords='axes fraction', textcoords='axes fraction',
                va='center', ha='center', bbox=nodetype, arrowprops=arrow_args)

# 画箭头上的内容，也就是每个节点的选择值
def plot_text(parent_pt, center_parent, text):
    x = (parent_pt[0] - center_parent[0]) / 2 + center_parent[0]
    y = (parent_pt[1] - center_parent[1]) / 2 + center_parent[1]
    ax.text(x, y, text)

# 计算树的深度，为了画树的y坐标做准备
def get_tree_deep(tree):
    if isinstance(tree, str):
        return 1
    elif isinstance(tree, dict):
        key = tree.keys()[0]
        value = tree[key]
        deep = 0
        for item in value.keys():
            this_deep = 1 + get_tree_deep(value[item])
            deep = this_deep if this_deep > deep else deep
        return deep

# 计算叶子节点，为画树时的x坐标准备
def get_leafnode_num(tree, leafnode_num):
    if isinstance(tree, str):
        leafnode_num += 1
        return leafnode_num
    elif isinstance(tree, dict):
        key = tree.keys()[0]
        value = tree[key]
        for item in value.keys():
            leafnode_num = get_leafnode_num(value[item], leafnode_num)
        return leafnode_num

# node_text是节点里的内容； parent_pt是箭头出发的位置； center_parent箭头结束的位置 ；
# key是箭头上的标示，也是决策树的选择值； had_draw_leafnode 已经画过的叶子节点，为了计算下一个叶子节点的x坐标
def draw_tree(node_text, parent_pt, center_parent, key, had_draw_leafnode):
    if isinstance(node_text, str):
        plot_text(parent_pt, center_parent, key)
        plot_node(node_text, parent_pt, center_parent, decisionNode)
        had_draw_leafnode += 1
        return had_draw_leafnode
    elif isinstance(node_text, dict):
        next_key = node_text.keys()[0]
        value = node_text[next_key]
        plot_text(parent_pt, center_parent, key)
        plot_node(next_key, parent_pt, center_parent, leafNode)
        for item in value.keys():
            this_leafnum_node = get_leafnode_num(value[item], 0)
            next_xind = had_draw_leafnode * x_off + (this_leafnum_node - 1) * x_off / 2
            had_draw_leafnode = draw_tree(value[item], center_parent, (next_xind, center_parent[1]-y_off), item, had_draw_leafnode)
        return had_draw_leafnode

# tree = {'no surfacing': {0: 'no', 1: {'flippers': {0: 'no', 1: 'yes'}}}}
# tree = {'no surfacing': {0: 'no', 1: {'flippers': {0: {'head': {0: 'no', 1: 'yes'}}, 1: 'no'}}}}
# tree = {'no surfacing': {0: 'no', 1: {'flippers': {0: 'no', 1: 'yes'}}, 3: 'maybe'}}
tree = {'no surfacing': {0: 'no', 1: {'flippers': {0: {'head': {0: 'no', 1: 'yes'}}, 1: 'no'}}, 3: 'maybe'}}

decisionNode = dict(boxstyle="sawtooth", fc="0.8")
leafNode = dict(boxstyle="round4", fc="0.8")
arrow_args = dict(arrowstyle="<-")
# 1(num)是作为一个id，如果这个id已经存在就激活返回引用否则就创建返回引用
# figsize 元组，图片大小，宽高; dpi 图片分辨率； facecolor 是背景色; edgecolor 边框色；
fig = plt.figure(1, facecolor='w')
fig.clf()
# 不显示坐标数据
axprops = dict(xticks=[], yticks=[])
# frameon=False 是否显示坐标轴架构
ax = plt.subplot(111, frameon=False, **axprops)

# 为了计算画每个节点时xy坐标需要移动的大小
tree_deep = get_tree_deep(tree)
leafnum_node = get_leafnode_num(tree, 0)
x_off = 1.0 / (leafnum_node - 1)
y_off = 1.0 / (tree_deep - 1)
# 画树并展示
draw_tree(tree, (0.5, 1.0), (0.5, 1.0), '', 0)
plt.show()
```
效果：![](2.png)
ax.annotate方法的xy（箭头起始位置）和xytext（箭头结束位置）一样时，箭头会消失，只画方格里的内容，用此原理画根节点。

以上代码最麻烦的就是如何画每个节点的x坐标。
我最开始的计划是每个节点的子节点以当前节点左右分开，因此使用当前节点的x坐标通过子节点的下标加减得到子节点的x坐标。但是效果不太好，而且仔细思考有很多的问题，比如说当前层的其他节点的子节点可能会和当前节点的子节点重合到一起等等一堆的问题。
我后来苦思冥想啊，上厕所时终于想到一个极好的办法。简单描述就是：用叶子节点推断当前节点的x坐标的方法。由下往上推导的方法。
具体：
（1）所有叶子节点在x轴均匀分布，因此算出两个叶子节点的x轴距离x_off。
（2）假设当前节点有四个叶子节点，然后当前节点的x坐标肯定在这四个叶子节点的中间啊，因此可以算出当前节点在四个叶子节点的x坐标是`(this_leafnum - 1) * x_off / 2`
（3）这时我们只需知道当前节点的四个叶子节点的最左边叶子节点的x轴刻度，然后加上步骤（2）的结果就可以得到当前节点的x轴的刻度。这时我们只需知道我们已经画了多少个叶子节点（因为我们是从左往右画的），已经画的叶子节点数乘以x_off就得到四个叶子节点最左边叶子节点的x轴刻度。
因此代码`had_draw_leafnode * x_off + (this_leafnum - 1) * x_off / 2`得到x轴坐标。
# 代码下载地址
[代码地址](https://github.com/meihuakaile/decision_tree)