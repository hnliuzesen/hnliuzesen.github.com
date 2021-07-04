---
title: TensorFlow 代码模板
date: 2017-06-07 19:33:25
categories:
- Computers & Technology
- Machine Learning
tags:
- TensorFlow
- Python
---

通过看《面向机器智能的TensorFlow实践》总结的使用 TensorFlow 的推荐编程方式和注意事项

<!--more-->

## 训练模型框架
```python
import tensorflow as tf

# 这里写模型的主要算法

def inference(X):
    # 计算输入 X 时，模型的输出
def loss(X, Y):
    # 根据输入 X 和期望 Y 算出损失
def input(total_loss):
    # 读取训练数据
def train(total_loss):
    # 根据计算的总损失调整模型参数
def evaluate(sess, X, Y):
    # 对模型进行评估

# Saver 对象用于保存检查点
saver = tf.train.Saver()

with tf.Session() as sess:
    tf.initialize_all_variables().run()
    
    X, Y = inputs()
    
    # 好像少了个 inference，感觉应该是 total_loss = loss(inference(X), Y) 才对，算出来损失
    total_loss = loss(X, Y)
    # 根据损失调教模型
    train_op = train(total_loss)
    
    # TODO: 这个应该是多线程的代码？
    coord = tf.train.Coordinator()
    threads = tf.train.start_queue_runners(sess=sess, coord=coord)
    
    # 训练次数
    training_steps = 1000
    
    # 使用 Saver 获取检查点
    initial_step = 0
    ckpt = tf.train.get_checkpoint_state(os.path.dirname(__file__))
    if chkpt and ckpt.model_checkpoint_path:
        saver.restore(sess, ckpt.model_checkpoint_path)
        initial_step = int(ckpt.model_checkpoint_path.rsplit('-', 1)[1])
    
    # 打印出每次训练的结果
    for step in range(initial_step, training_steps):
        sess.run([train_op])
        if step % 10 == 0:
            print "loss:", sess.run([total_loss])
    
    # 模型评估
    evaluate(sess, X, Y)
    
    # 每 1000 次训练保存检查点一次
    saver.save(sess, 'my-model', global_step=training_steps)
    saver.close()
    
    coord.request_stop()
    coord.join(threads)
    sess.close()
```

这段代码中最重要的应该有两部分，一个是最开始的计算代码，对于数据应该如何进行计算来得到期望的值，也许不确定各种参数，但是思路一定要对，剩下的交给引
擎来调整参数。另一个就是得到损失之后如何去调整参数的算法，这个一般有很多内置模型可以用。

## 书写风格
```python
import tensorflow as tf

# 不创建图会使用默认的 Graph，获取方式是 tf.get_default_graph()
graph = tf.Graph()

with graph.as_default():

    # 使用命名空间区分不同目的的代码
    with tf.name_scope("variables"):
        # 这里主要声明变量，分别是数据流图的运行次数和每次输出的累加和
        global_step = tf.Variable(0, dtype=tf.int32, trainable=False, name="global_step")
        # 用于统计的变量如果不希望机器在学习的时候误更改他们，可以将 trainable 设置为 False
        total_output = tf.Variable(0.0, dtype=tf.float32, trainable=False, name="total_output")

    # 主要负责数据变换的代码
    with tf.name_scope("transformation"):
        with tf.name_scope("input"):
            a = tf.placeholder(tf.float32, shape=[None], name="input_placeholder_a")

        # 计算积和和
        with tf.name_scope("intermediate_layer"):
            b = tf.reduce_prod(a, name="product_b")
            c = tf.reduce_sum(a, name="sum_c")

        # 输出层
        with tf.name_scope("output"):
            output = tf.add(b, c, name="output")

    # 用计算结果更新全局变量
    with tf.name_scope("update"):
        # 更新变量的同时用 update_total 存储
        update_total = total_output.assign_add(output)
        increment_step = global_step.assign_add(1)

    # 汇总所有操作
    with tf.name_scope("summaries"):
        # tf.cast 数据转换
        avg = tf.div(update_total, tf.cast(increment_step, tf.float32), name="average")

        # 创建汇总数据
        tf.summary.scalar('Output', output)
        tf.summary.scalar('Sum of outputs over time', update_total)
        tf.summary.scalar('Average of outputs over time', avg)

    with tf.name_scope("global_ops"):
        init = tf.initialize_all_variables()
        # 将上面的所有汇总数据放入一个 OP 节点，该操作最好和全局变量初始化放在一起
        merged_summaries = tf.summary.merge_all()

sess = tf.Session(graph=graph)
# 保存汇总数据
writer = tf.summary.FileWriter('./improved_graph', graph)
sess.run(init)


# 创建一个调用函数方便多次调用流程
def run_graph(input_tensor):
    feed_dict = {a: input_tensor}
    _, step, summary = sess.run([output, increment_step, merged_summaries], feed_dict=feed_dict)
    writer.add_summary(summary, global_step=step)


run_graph([2, 8])
run_graph([3, 1, 3, 3])
run_graph([8])
run_graph([1, 2, 3])
run_graph([11, 4])
run_graph([4, 1])
run_graph([7, 3, 1])
run_graph([6, 3])
run_graph([0, 2])
run_graph([4, 5, 6])

# 将运算结果保存到磁盘
writer.flush()

writer.close()
sess.close()

```
