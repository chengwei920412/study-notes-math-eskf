1) 过程噪声协方差(Q); 2) 测量噪声协方差(R).

Q和R是相对应设置的。Q：R决定了“相信”预测还是观测，两个值越小“相信”的成分越大。换句话说，Q和R分别代表预测和观测时候误差，因此误差越小越值得“相信”。

每个完美的理论背后都有一个黑暗的角落。在卡尔曼滤波器中，Q & R 就是其黑暗的地方。

# 现有几个卡尔曼滤波器中的 Q & R

# 参考文件
1. [KF，EKF怎样设置Q和R阵?](https://www.zhihu.com/question/30481204/answer/50092960)
1. [卡尔曼滤波中的噪声协方差矩阵(R和Q)应该怎么取值,和噪声分布之间的关系是什么？](https://www.zhihu.com/question/53788909)
