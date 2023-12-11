# Abstract for Privacy-frs

仅针对`privacy-frs`中已用到的代码

### 1. 添加数据库/添加代码

1. 以下过程所用到的代码均放在`privacy-frs`目录下. (会调用到原框架中的函数)

2. 确保`LFW`数据集放在`datasets`目录下, 并重命名为`LFW`.

### 2. 为数据库添加用户

以张明轩为例. 

1. 首先, 为张明轩拍摄若干照片. 尽量做到在**不同光照条件下**进行采样, 可以有表情变化, 但人物脸部应保持正对摄像头. (这和最终分类效果相关性较大)
2. 手动将照片剪切为`160 * 160`. 剪切时保持人物脸部在中央.
3. 将一系列照片命名为`Zhang_Mingxuan_0001.jpg` , `Zhang_Mingxuan_0002.jpg`... 并把这些照片放在`datasets/LFW/Zhang_Mingxuan`目录下.
4. 运行`dataset_add_user.ipynb`. 结果为更新后的`data_updated.pt`

### 3. 生成patch

运行`patch.ipynb`

可调参数: 

1. `total` : 根据用户照片总数进行修改. **必改**.
2. `base_dir` : 根据用户名称进行修改. **必改**.
3. `target_img` : 选择要进行攻击的目标照片. 可选.
4. `train_with_mask(num_epochs, init_lr, net, patch_size)` : 可对训练轮数和样本大小进行修改. 可选. 也可使用`train_without_mask(num_epochs, init_lr, net, patch_size)` 来训练出针对用户不戴口罩情况下的样本. (即: 用户不戴口罩坐在摄像头前, 期望结果为识别为目标)

运行过程图片保存在`privacy/run_results`目录下, 每次运行**会覆盖之前的运行结果**. **后续实际使用的样本**为`patch.pt(tensor)`.

### 4. 在训练集上的精度验证

运行`patch.ipynb`的最后两个cell. 可以得出添加patch后, 人脸识别模型在用户照片集上的分类结果和相似度. 可用于初步判断, 防止训练错的离谱.

可调参数: 

1. `patch`路径. 实验过程中需要存储不同参数下的训练结果. 此时需要修改`patch`路径来对不同的patch进行测试.

### 5. 在摄像头下的验证

运行`test.ipynb`中的函数. 当前的实验只需要执行**5.3** , 但是如果要对新加入的用户也计算之前已有的实验数据, 则需要用到5.1和5.2

#### 5.1 `identification_vid(target_name)`

1. 用于实时显示识别结果.
2. 返回识别准确率;成功欺骗为target的情况下的平均相似度; 成功欺骗为target的情况下的最大, 最小相似度.

可选参数: 

1. `target_name`. 例如`Yves_Brodeur`.
2. `patch`路径. 

#### 5.2 `test_similarity(target_name, origin_name)`

1. 用于计算平均相似度.  
2. 电子添加对抗样本(程序隐式添加).
3. 平均值计算的分母为运行时间内的所有视频帧.
4. 返回结果为与 (目标, 原始用户)的相似度平均值.

可选参数: 

1. `target_name`, `origin_name`. 例如`Yves_Brodeur`, `Zhang_Mingxuan`
2. `patch`路径. 

#### 5.3 `response_time(target_name)`

1. 用于计算首次成功对抗(分类到目标类别)所需的时间.
2. 先调整位置, **按下空格后开始正式计算.**
3. 返回从开始计算到首次连续3帧识别为目标所需的时间.

可选参数: 

1. `target_name`. 例如`Yves_Brodeur`.
2. `patch`路径. 