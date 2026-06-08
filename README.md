这个是最初始的版本。


1.数据集处理trans_dateset.py:   
    将expertC修饰的rgb色域图片转换到cmyk色域。
    sRGB.icc是从fiveK图片中提取的，


2.数据集加载datasets_icc.py:
    加载rgb图片(原图)和cmyk图片(经过icc转换后的expertC图)。

3.模型代码models_icc.py:
        DownSample     Encoder     Encoder     Encoder      Decoder     Decoder     Decoder     UpSample
    原图----------->1/2-------->1/4-------->1/8-------->1/16-------->1/8-------->1/4-------->1/2--------->区域权重
                                                         |
                                                         ↓
                                                     Classifier
    Classifier: 金字塔结构，提取不同尺度信息，用于输出各区域对各个组件(Matrix/LUT1D/LUT3D)的权重，每个组件都有n_ranksX个基底，根据权重混合生成各个区域的组件。 
    Matrix: 简单的3x3对角矩阵，有多个基底(恒等/冷/暖色调等)。
    LUT1D:  R/G/B通道公用的LUT，有多个基底(恒等/亮度/对比度等)。
    LUT3D:  R/G/B的查找表，有多个基底(恒等/高饱和/低饱和等)。

4.训练代码train_icc.py:
    loss的组成: 
        l_cmyk: 主要loss
        l_lab:  转换到lab色域，约束亮度和色调
        l_cm:   约束Matrix为对角矩阵(目标是白平衡，避免通道混合)
        l_mono: 约束LUT1D的单调性，且避免溢出(<0或>1)
        l_tv:   约束LUT3D的平滑性
        l_div:  避免基底都训练成一样的
    由于图片大小不一，训练的时候是一张张训练的，所以Matrix容易一开始学成负的，导致后续很难调过来，
        可以将models.py里的w_matrix = global_weight[:, 0:idx_1]改成w_matrix = global_weight[:, 0:idx_1].softmax(dim=1),     /***没测完，CIE2000在4.几***/
        或者重新训练(概率问题，如果先训练到负的，后续就很难拉回来，如果先训练到正的，后续就还行)。

数据集处理的路径如"dataset.png"所示()。    
模型和测试图片保存路径如"train.png"所示。
