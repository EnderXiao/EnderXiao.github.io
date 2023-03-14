---
title: python进阶-OOP
date: 2021-07-26 11:39:13
categories:
  - - 机器学习
  - - 计算机语言
  - - 研究生课程基础
  - - Python
  - - pytorch_lightning
tags:
  - 机器学习
  - python
  - pytorch
  - pytorch_lightning
  - 进阶
mathjax: true
headimg: 'https://s1.ax1x.com/2023/03/14/ppl1fAA.jpg'
---

## Pytorch Lightning

pl是一个轻量级的PyTorch库，用于对深度学习模型进行轻量化，但它并不提供样板，而仅仅是轻量化

它能对日常使用的样板代码进行抽象和自动化。

![img](https://img-blog.csdnimg.cn/20210519115829655.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQyNjQzNzM=,size_16,color_FFFFFF,t_70)

![img](https://img-blog.csdnimg.cn/20210519115842964.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQyNjQzNzM=,size_16,color_FFFFFF,t_70)

从上面我们可以发现 pl 的三个优势

1. 通过抽象出样板工程代码，可以更容易地识别和理解ML代码。
2. Lightning的统一结构使得在现有项目的基础上进行构建和理解变得非常容易。
3. Lightning 自动化的代码是用经过全面测试、定期维护并遵循ML最佳实践的高质量代码构建的。

## pl.LightningModule

`pl.LightningModule`是Pytorch中的模型部分(`nn.Module`)和训练逻辑（训练、验证、测试）部分的结合。

继承自该类的函数通常包含三个部分：

1. 模型相关部分：
   1. `__init__`
   2. `forward`
2. 优化器相关部分:
   1. `configure_optimizers`
3. 模型训练逻辑部分：
   1. `training_step`
   2. `validation_step`
   3. `test_step`

模型相关部分一般涉及模型的构建，包括超参数定义、模型初始化、模型运行逻辑(`forward`函数)

优化器相关部分一般涉及模型的优化器初始化，学习率的schedule设置等

训练逻辑部分一般是每个训练、验证、预测步骤需要进行的操作。

```python
import pytorch_lightning as pl

class CIFARModule(pl.LightningModule):
    def __init__(self, ) -> None:
        super().__init__()

    def forward(self, imgs):

    def configure_optimizers(self):

    def training_step(self, batch, batch_idx):

    def validation_step(self, batch, batch_idx):

    def test_step(self, batch, batch_idx):
```

## pl 流程

PL的流程很简单，生产流水线，有一个固定的顺序：

初始化 `def __init__(self)` -->训练`training_step(self, batch, batch_idx)` --> 校验`validation_step(self, batch, batch_idx)` --> 测试 `test_step(self, batch, batch_idx)`. 就完事了，总统是实现这三个函数的重写。

当然，除了这三个主要的，还有一些其他的函数，为了方便我们实现其他的一些功能，因此更为完整的流程是在training_step 、validation_step、test_step 后面都紧跟着其相应的 `training_step_end(self，batch_parts)`和`training_epoch_end(self, training_step_outputs)` 函数，当然，对于校验和测试，都有相应的`*_step_end`和`*_epoch_end`函数。因为校验和测试的`*_step_end`函数是一样的，因此这里只以训练为例。

`*_step_end` -- 即每一个 * 步完成后调用

`*_epoch_end` -- 即每一个 * 的epoch 完成之后会自动调用

### Train

训练主要是重写`def training_setp(self, batch, batch_idx)`函数，并返回要反向传播的`loss`即可，其中batch 即为从 train_dataloader 采样的一个batch的数据，batch_idx即为目前batch的索引。

```python
def training_setp(self, batch, batch_idx):
    image, label = batch
    pred = self.forward(iamge)
    loss = ...
    # 一定要返回loss
    return loss
```

### Validation

#### 设置校验的频率

##### 每训练n个epochs 校验一次

默认为每1个`epoch`校验一次，即自动调用`validation_step()`函数

```python
trainer = Trainer(check_val_every_n_epoch=1)
```

##### 单个epoch内校验频率

当一个epoch 比较大时，就需要在单个epoch 内进行多次校验，这时就需要对校验的调动频率进行修改， 传入`val_check_interval`的参数为`float`型时表示百分比，为`int`时表示`batch`：

```python
# 每训练单个epoch的 25% 调用校验函数一次，注意：要传入float型数
trainer = Trainer(val_check_interval=0.25)
# 当然也可以是单个epoch训练完多少个batch后调用一次校验函数，但是一定是传入int型
trainer = Trainer(val_check_interval=100) # 每训练100个batch校验一次
```

校验和训练是一样的，重写`def validation_step(self, batch, batch_idx)`函数，不需要返回值：

```python
def validation_step(self, batch, batch_idx):
    image, label = batch
    pred = self.forward(iamge)
    loss = ...
    # 标记该loss，用于保存模型时监控该量
    self.log('val_loss', loss)
```

### Test

在`pytoch_lightning`框架中，test 在训练过程中是不调用的，也就是说是不相关，在训练过程中只进行training和validation，因此如果需要在训练过中保存validation的一些信息，就要放到validation中。

关于测试，测试是在训练完成之后的，因此这里假设已经训练完成：

```python
# 获取恢复了权重和超参数等的模型
model = MODEL.load_from_checkpoint(checkpoint_path='my_model_path/heiheihei.ckpt')
# 修改测试时需要的参数，例如预测的步数等
model.pred_step = 1000
# 定义trainer, 其中limit_test_batches表示取测试集中的0.05的数据来做测试
trainer = pl.Trainer(gpus=1, precision=16, limit_test_batches=0.05)
# 测试，自动调用test_step(), 其中dm为数据集，放在下面讲
trainer.test(model=dck, datamodule=dm)
```

### 数据集

数据集有两种实现方法：

当然，首先要自己先实现Dataset的定义，可以用现有的，例如MNIST等数据集，若用自己的数据集，则需要自己去继承`torch.utils.data.dataset.Dataset`，自定义类，这一部分不再细讲，查其他的资料。

#### 直接实现

直接实现是指在Model中重写`def train_dataloader(self)`等函数来返回dataloader：

```python
class ExampleModel(pl.LightningModule):
    def __init__(self, args):
        super().__init__()
        self.train_dataset = ...
        self.val_dataset = ...
        self.test_dataset = ...
        ...
    def train_dataloader(self):
        return DataLoader(self.train_dataset, batch_size=self.batch_size, shuffle=False, num_workers=0)

    def val_dataloader(self):
        return DataLoader(self.val_dataset, batch_size=self.batch_size, shuffle=False)
    def test_dataloader(self):
        return DataLoader(self.test_dataset, batch_size=1, shuffle=True)
```

这样就完成了数据集和dataloader的编程了，**注意，要先自己完成dataset的编写，或者用现有的normal数据集**

#### 自定义DataModule

这种方法是继承`pl.LightningDataModule`来提供训练、校验、测试的数据。

```python
class MyDataModule(pl.LightningDataModule):
    def __init__(self):
        super().__init__()
        ...blablabla...
    def setup(self, stage):
        # 实现数据集的定义，每张GPU都会执行该函数, stage 用于标记是用于什么阶段
        if stage == 'fit' or stage is None:
            self.train_dataset = DCKDataset(self.train_file_path, self.train_file_num, transform=None)
            self.val_dataset = DCKDataset(self.val_file_path, self.val_file_num, transform=None)
        if stage == 'test' or stage is None:
            self.test_dataset = DCKDataset(self.test_file_path, self.test_file_num, transform=None)
    def prepare_data(self):
        # 在该函数里一般实现数据集的下载等，只有cuda:0 会执行该函数
        pass
    def train_dataloader(self):
        return DataLoader(self.train_dataset, batch_size=self.batch_size, shuffle=False, num_workers=0)

    def val_dataloader(self):
        return DataLoader(self.val_dataset, batch_size=self.batch_size, shuffle=False)

    def test_dataloader(self):
        return DataLoader(self.test_dataset, batch_size=1, shuffle=True)
```

#### 使用

```python
dm = MyDataModule(args)
if not is_predict:# 训练
    # 定义保存模型的callback，仔细查看后文
    checkpoint_callback = ModelCheckpoint(monitor='val_loss')
    # 定义模型
    model = MyModel()
    # 定义logger
    logger = TensorBoardLogger('log_dir', name='test_PL')
    # 定义数据集为训练校验阶段
    dm.setup('fit')
    # 定义trainer
    trainer = pl.Trainer(gpus=gpu, logger=logger, callbacks=[checkpoint_callback]);
    # 开始训练
    trainer.fit(dck, datamodule=dm)
else:
    # 测试阶段
    dm.setup('test')
    # 恢复模型
    model = MyModel.load_from_checkpoint(checkpoint_path='trained_model.ckpt')
    # 定义trainer并测试
    trainer = pl.Trainer(gpus=1, precision=16, limit_test_batches=0.05)
    trainer.test(model=model, datamodule=dm)
```

### 模型保存

#### 自动保存

Lightning 会自动保存最近训练的epoch的模型到当前的工作空间(`or.getcwd()`)，也可以在定义Trainer的时候指定：

```python
trainer = Trainer(default_root_dir='/your/path/to/save/checkpoints')
```

当然，也可以关闭自动保存模型：

```python
trainer = Trainer(checkpoint_callback=False)
```

#### ModelCheckpoint（call backs）

1. 计算需要监控的量，例如校验误差：`loss`
2. 使用`log()`函数标记该要监控的量
3. 初始化`ModelCheckpoint`回调，并设置要监控的量，下面有详细的描述
4. 将其传回到`Trainer`中

```python
from pytorch_lightning.callbacks import ModelCheckpoint

class LitAutoEncoder(pl.LightningModule):
    def validation_step(self, batch, batch_idx):
        x, y = batch
        y_hat = self.backbone(x)

        # 1. 计算需要监控的量
        loss = F.cross_entropy(y_hat, y)

        # 2. 使用log()函数标记该要监控的量,名字叫'val_loss'
        self.log('val_loss', loss)

# 3. 初始化`ModelCheckpoint`回调，并设置要监控的量
checkpoint_callback = ModelCheckpoint(monitor='val_loss')

# 4. 将该callback 放到其他的callback 的list中
trainer = Trainer(callbacks=[checkpoint_callback])
```

其中`ModuleCheckpoint`的参数如下：

```python
CLASS pytorch_lightning.callbacks.model_checkpoint.ModelCheckpoint(filepath=None, monitor=None, verbose=False, save_last=None, save_top_k=None, save_weights_only=False, mode='auto', period=1, prefix='', dirpath=None, filename=None)
```

> 参数说明：所有参数均为optional。
> `filepath` -- 不建议使用，在后续版本中会被删除；保存的模型文件的路径，后面的参数会有另外两个参数来代替这个。
> `monitor` -- 需要监控的量，`string`类型。例如`'val_loss'`（在`training_step()` or `validation_step()`函数中通过self.log('val_loss', loss)进行标记）；默认为`None`，只保存最后一个epoch的模型参数,（我的理解是只保留最后一个epoch的模型参数，但是还是每训练完一个epoch之后会保存一次，然后覆盖上一次的模型)
> `verbose`：冗余模式，默认为False.
> `save_last`:`bool`类型; 默认`None`，当为`True`时，表示在每个epoch 结果的时候，总是会保存一个模型`last.ckpt`，也就意味着会覆盖保存，只会有一个文件保留。
> `save_top_k`：`int`类型；当`save_top_k==k`，根据`monitor`监控的量，保存`k`个最好的模型，而最好的模型是当`monitor`监控的量最大时表示最好，还是最小时表示最好，在后面的参数`mode`中进行设置。**当`save_top_k==0`时，不保存**；当`save_top_k==-1`时，保存所有的模型，即每个次保存模型不进行覆盖保存，全都保存下来；当`save_top_k>=2`，并且在单个epoch内多次调用保存模型的函数，则模型的名字最后会追加版本号，从`v0`开始。
> `mode` ：`string`类型，只能取{'auto', 'min', 'max'}中的一个；当`save_top_k!=0`时，保存模型时就会覆盖保存，如果`monitor`监控的是`val_loss`等越小就表示模型越好的，这个参数应该被设置成`'min'`，当`monitor`监控的是`val_acc`（校验准确度）等越大就表示模型训练的越好的量，则应该设置成`'max'`。`auto`会自动根据`monitor`的名字来判断（`auto`模式是个人理解，可能会出错，例如你编程的时候，你就喜欢用`val_loss`表示模型准确度这样就会导致保存的模型是最差的模型了）。
> `save_weights_only`: `bool` 类型；`True`只保存模型权重(`model.save_weights(flepath)`)，否则保存整个模型。建议保存权重就可以了，保存整个模型会消耗更多时间和存储空间。
> `period`: `int`类型。保存模型的间隔，单位为epoch，即隔多少个epoch自动保存一次。
> `prefix`: `string`类型；保存模型文件的**前缀**。
> `dirpath`: `string`类型。例如：`dirpath='my/path_to_save_model/'`
> `filename`: `string`类型；前面就说过不建议使用`filepath`变量，推荐使用 `dirpath+filename`的形式来作为模型路径。例如：
> 文件名会以epoch、val_loss、和其他的一些指标作为名称来保存
> 模型名称: my/path/epoch=2-val_loss=0.02-other_metric=0.03.ckpt
> `checkpoint_callback = ModelCheckpoint( ... , dirpath='my/path', ... filename='{epoch}-{val_loss:.2f}-{other_metric:.2f}' ... ) `

#### 使用案例

```python
from pytorch_lightning import Trainer
from pytorch_lightning.callbacks import ModelCheckpoint

# saves checkpoints to 'my/path/' at every epoch
checkpoint_callback = ModelCheckpoint(dirpath='my/path/')
trainer = Trainer(callbacks=[checkpoint_callback])

# save epoch and val_loss in name
# saves a file like: my/path/sample-mnist-epoch=02-val_loss=0.32.ckpt
checkpoint_callback = ModelCheckpoint(monitor='val_loss', dirpath='my/path/', filename='sample-mnist-{epoch:02d}-{val_loss:.2f}')
```

#### 获取最好的模型

因为根据上面保存的参数，可能保存了多个模型，根据`best_model_path`恢复最好的模型。

```python
from pytorch_lightning import Trainer
from pytorch_lightning.callbacks import ModelCheckpoint
checkpoint_callback = ModelCheckpoint(dirpath='my/path/')
trainer = Trainer(callbacks=[checkpoint_callback])
model = ...
trainer.fit(model)
# 训练完成之后，保存了多个模型，下面是获得最好的模型，也就是将原来保存的模型中最好的模型权重apply到当前的网络上
checkpoint_callback.best_model_path
```

#### 手动保存模型

```python
from collections import deque
import os
# 维护一个队列
self.save_models = deque(maxlen=3)
# 这里的self 是指这个函数放到继承了pl.LightningModule的类里，跟training_step()是同级的
def manual_save_model(self):
    model_path = 'your_model_save_path_%s' % (your_loss)
    if len(self.save_models) >= 3:
        # 当队列满了，取出最老的模型的路径，然后删除掉
        old_model = self.save_models.popleft()
        if os.path.exists(old_model):
            os.remove(old_model)
    # 手动保存
    self.trainer.save_checkpoint(model_path)
    # 将保存的模型路径加入到队列中
    self.save_models.append(model_path)
```

```python
# 保存最新的路径
def save_latest_model(self):
        checkpoint_callbacks = [c for c in self.trainer.callbacks if isinstance(c, ModelCheckpoint)]
        print("Saving latest checkpoint...")
        model = self.trainer.get_model()
        [c.on_validation_end(self.trainer, model) for c in checkpoint_callbacks]
```

#### 自定义文件名

`format_checkpoint_name`(*epoch*, *step*, *metrics*, *ver=None*)

在上面的`filename`参数中，定义了模型文件的保存格式，这个函数就是给其中的变量赋值的，返回`string`类型，文件名

```powershell
>>> tmpdir = os.path.dirname(__file__)
>>> ckpt = ModelCheckpoint(dirpath=tmpdir, filename='{epoch}')
>>> os.path.basename(ckpt.format_checkpoint_name(0, 1, metrics={}))
'epoch=0.ckpt'
>>> ckpt = ModelCheckpoint(dirpath=tmpdir, filename='{epoch:03d}')
>>> os.path.basename(ckpt.format_checkpoint_name(5, 2, metrics={}))
'epoch=005.ckpt'
>>> ckpt = ModelCheckpoint(dirpath=tmpdir, filename='{epoch}-{val_loss:.2f}')
>>> os.path.basename(ckpt.format_checkpoint_name(2, 3, metrics=dict(val_loss=0.123456)))
'epoch=2-val_loss=0.12.ckpt'
>>> ckpt = ModelCheckpoint(dirpath=tmpdir, filename='{missing:d}')
>>> os.path.basename(ckpt.format_checkpoint_name(0, 4, metrics={}))
'missing=0.ckpt'
>>> ckpt = ModelCheckpoint(filename='{step}')
>>> os.path.basename(ckpt.format_checkpoint_name(0, 0, {}))
'step=0.ckpt'
```

#### 手动保存

除了自己保存，还可以手动保存和加载模型

```python
model = MyLightningModule(hparams)
trainer.fit(model)
trainer.save_checkpoint("example.ckpt")
new_model = MyModel.load_from_checkpoint(checkpoint_path="example.ckpt")
```

#### 加载检查点

加载模型的**权重、偏置和超参数**：

```python
model = MyLightingModule.load_from_checkpoint(PATH)

print(model.learning_rate)
# prints the learning_rate you used in this checkpoint

model.eval()
y_hat = model(x)
```

如果需要修改超参数，在写Module的时候进行覆盖:

```python
class LitModel(LightningModule):
    def __init__(self, in_dim, out_dim):
        super().__init__()
        self.save_hyperparameters()
        # 在这里使用新的超参数，而不是从模型中加载的超参数
        self.l1 = nn.Linear(self.hparams.in_dim, self.hparams.out_dim)
```

这样的话，可以如下恢复模型：

```python
# 例如训练的时候初始化in_dim=32, out_dim=10
LitModel(in_dim=32, out_dim=10)
# 下面的方式恢复模型，使用in_dim=32和out_dim=10为保存的参数
model = LitModel.load_from_checkpoint(PATH)
# 当然也可以覆盖这些参数，例如改成in_dim=128, out_dim=10
model = LitModel.load_from_checkpoint(PATH, in_dim=128, out_dim10)
```

#### load_from_checkpoint 方法

```python
LightningModule.load_from_checkpoint(checkpoint_path, map_location=None, hparams_file=None, strict=True, **kwargs)
```

#### 恢复模型和Trainer

如果不仅仅是想恢复模型，而且还要接着训练，则可以恢复Trainer

```python
model = LitModel()
trainer = Trainer(resume_from_checkpoint='some/path/to/my_checkpoint.ckpt')
# 自动恢复模型、epoch、step、学习率信息（包括LR schedulers），精度等
# automatically restores model, epoch, step, LR schedulers, apex, etc...
trainer.fit(model)
```

### 训练辅助

#### Early Stopping

监控`validation_step()`中某一个量，如果其不能再变得更优，则提前停止训练

```python
pytorch_lightning.callbacks.early_stopping.EarlyStopping(monitor='early_stop_on', min_delta=0.0, patience=3, verbose=False, mode='auto', strict=True)
```

> **monitor** (`str`) – 监控的量；默认为:`early_stop_on`；可以通过`self.log('var_name', val_loss)`来标记要监控的量
> **min_delta** (`float`) – 最小的改变量；默认：0.0；即当监控的量的绝对值变量量小于该值，则认为没有新的提升
> **patience** (`int`) - 默认：3；如果监控的量持续patience 个epoch没有得到更好的提升，则停止训练；
> **verbose** (`bool`) – 默认:False；
> **mode** (`str`) – {auto, min, max}中的一个，跟前面的`ModelCheckpoint`中的`mode`是一样的含义。如果`monitor`监控的是`val_loss`等越小就表示模型越好的，这个参数应该被设置成`'min'`，当`monitor`监控的是`val_acc`（校验准确度）等越大就表示模型训练的越好的量，则应该设置成`'max'`。`auto`会自动根据`monitor`的名字来判断（`auto`模式是个人理解，可能会出错，例如你编程的时候，你就喜欢用`val_loss`表示模型准确度这样就会导致保存的模型是最差的模型了）。
> **strict** (`bool`) – 默认True；如果监控器没有在`validation_step()`函数中找到你监控的量，则强制报错，中止训练；

#### Logging

这里只涉及`Tensorboard`， 其它有需要的可参考官方文档[Logging](https://link.zhihu.com/?target=https%3A//pytorch-lightning.readthedocs.io/en/latest/logging.html)，tensorboard 有两种基本的方法：一种是只适用于scaler，可直接使用`self.log()`，另一种是图像、权重等。

```python
# 在定义Trainer对象的时候，传入tensorboardlogger
logger = TensorBoardLogger(args['log_dir'], name='DCK_PL')
trainer = pl.Trainer(logger=logger)
# 获取tensorboard Logger， 以在validation_step()函数为例
def validation_step():
    tensorboard = self.logger.experiment
    # 例如求得validation loss为：
    loss = ...
    # 直接log
    self.log('val_loss', loss, on_step=True, on_epoch=True, prog_bar=True, logger=True)
    # 如果是图像等，就需要用到tensorboard的API
    tensorboard.add_image()
    # 同时log多个
    other_loss = ...
    loss_dict = {'val_loss': loss, 'loss': other_loss}
    tensorboard.add_scalars(loss_dict)
    # log 权重等
    tensorboard.add_histogram(...)
```

注意如果是用anaconda的话，要先激活你的env，另外要注意的是，`--logdir=my_log_dir/`， 这里的logdir要到`version_0/`目录，该目录下保存有各种你log的变量的文件夹

```text
# 查看的方法跟tensorboard是一样的，在终端下
tensorboard --logdir=my/log_path
```

当然也可以继承`LightningLoggerBase`类来自定义Logger，这个自己看官方文档

#### optimizer 和 lr_scheduler

当然，在训练过程中，对学习率的掌控也是非常重要的，合理设置学习率有利于提高效果，学习率衰减可查看[四种学习率衰减方法](https://zhuanlan.zhihu.com/p/93624972)。那在pytorch_lightning 中如何设置呢？其实跟pytorch是一样的，基本上不需要修改：

```python
# 重写configure_optimizers()函数即可
# 设置优化器
def configure_optimizers(self):
    weight_decay = 1e-6  # l2正则化系数
    # 假如有两个网络，一个encoder一个decoder
    optimizer = optim.Adam([{'encoder_params': self.encoder.parameters()}, {'decoder_params': self.decoder.parameters()}], lr=learning_rate, weight_decay=weight_decay)
    # 同样，如果只有一个网络结构，就可以更直接了
    optimizer = optim.Adam(my_model.parameters(), lr=learning_rate, weight_decay=weight_decay)
    # 我这里设置2000个epoch后学习率变为原来的0.5，之后不再改变
    StepLR = torch.optim.lr_scheduler.MultiStepLR(optimizer, milestones=[2000], gamma=0.5)
    optim_dict = {'optimizer': optimizer, 'lr_scheduler': StepLR}
    return optim_dict
```

这样就OK了，只要在`training_step()`函数中返回了`loss`，就会自动反向传播，并自动调用`loss.backward()`和`optimizer.step()`和`stepLR.step()`了

#### **多优化器用于多模型等网络结构**

**当我们训练的是复杂的网络结构时，可能有多个模型，需要不同的训练顺序，不同的训练学习率等，这时候就需要设计多个优化器，并手动调用梯度反传函数**

```python
 # multiple optimizer case (e.g.: GAN)
 def configure_optimizers(self):
     opt_d = Adam(self.model_d.parameters(), lr=0.01)
     opt_g = Adam(self.model_g.parameters(), lr=0.02)
     return opt_d, opt_g
```

**然后要关掉自动优化，这样就可以跟pytorch一样手动控制优化器的权重更新了**，达到了跟pytorch一样可以进行复杂地更新顺序等地控制，同时pytorch lightning的优势还在，例如多GPU下batchnorm的参数同步等。

```python
 # 在new Trainer对象的时候，把自动优化关掉
 trainer = Trainer(automatic_optimization=False)
```

**这时候 `training_step()`函数也就不是直接返回loss 或者 字典了，而是不需要返回loss了，因为在该函数里就手动完成权重更新函数地调用**, 另外需要注意的是：1. 不再使用`loss.backward()`函数，改用`self.manual_backward(loss, opt)`，就可以实现**半精度训练**。 2. 忽略`optimizer_idx`参数。

```python
 def training_step(self, batch, batch_idx, opt_idx):
     # 获取在configure_optimizers()中返回的优化器
     (opt_d, opt_g) = self.optimizers()
     loss_g = self.acquire_loss_g()
     
     # 注意:不再使用loss.backward(). 另外以GAN为例，因为生成器的动态图还要保持给判别器用于更新，因此retain_graph=True.
     self.manual_backward(loss_g, opt_g, retain_graph=True)
     # 销毁动态图
     self.manual_backward(loss_g, opt_g)
     opt_g.step()
     # 在更新判别器的时候，保存生成器是0梯度的
     opt_g.zero_grad()
     
     # 更新判别器
     loss_d = self.acquire_loss_d()
     self.manual_backward(loss_d, opt_d)
```

### 其他

#### 多GPU训练

如果是CPU训练，在定义Trainer时不管`gpus`这个参数就可以了，或者设置该参数为`0`：

```python
trainer = pl.Trainer(gpus=0)
```

而多GPU训练，也是很方便，只要将该参数设置为你要用的gpu数就可以，例如用4张GPU：

```python
trainer = pl.Trainer(gpus=4)
```

而如果你有很多张GPU，但是要跟同学分别使用，只要在程序最前面设置哪些GPU可用就可以了，例如服务器有4张卡，但是你只能用0和2号卡：

```python
import os
os.environ['CUDA_VISIBLE_DEVICES'] = '0, 2'
trainer = pl.Trainer(gpus=2)
```

#### 半精度训练

半精度训练也是Apex的一大特色，可以在几乎不影响效果的情况下降低GPU显存的使用率（大概50%），提高训练速度，现在pytorch_lightning 统统都给你，可以只要设置一下参数就可以：

```python
trainer = pl.Trainer(precision=16)
```

#### 累积梯度

默认情况是每个batch 之后都更新一次梯度，当然也可以`N`个batch后再更新，这样就有了大batch size 更新的效果了，例如当你内存很小，训练的batch size 设置的很小，这时候就可以采用累积梯度：

```python
# 默认情况下不开启累积梯度
trainer = Trainer(accumulate_grad_batches=1)
```

#### 自动缩放batch_size

**这方法还有很多限制，直接`trainer.fit(model)`是无效的**，感觉挺麻烦，不建议用

大的batch_size 通过可以获得更好的梯度估计。但同时也要更长的时间，另外，如果内存满了，电脑会卡住动不了。 `'power'` -- 从batch size 为1 开始翻倍地往上找，例如``1-->2 --> 4 --> ...`一直到内存溢出(out-of-memory, OOM)；`binsearch`也是翻倍地找，直OOM，但是之后还要继续进行一个二叉搜索，找到一个更好的batch size。另外，搜索的batch size 最大不会超过数据集的尺寸。

```python
# 默认不开启
trainer = Trainer(auto_scale_batch_size=None)

# 自动找满足内存的 batch size
trainer = Trainer(auto_scale_batch_size=None|'power'|'binsearch')

# 加载到模型
trainer.tune(model)
```

#### 保存所有超参数到模型中

将所有的模型超参数都保存到模型中，恢复模型时再也不用自己去拖动恢复模型中的超参数了，这点是太有特色了：

```python
# 例如你传入的超参数字典为params_dict
self.hparams.update(params_dict)    # 直接将你的超参数更新到pl模型的超参数字典中
# 这样，在保存的时候就会保存超参数了
self.save_hyperparameters()
```

当然，对于无们训练的不同的模型，我们还是需要查看其超参数，可以通过将超参数字典保存到本地txt的方法，来以便后期查看

```python
def save_dict_as_txt(list_dict, save_dir):
    with open(save_dir, 'w') as fw:
        if isinstance(list_dict, list):
            for dict in list_dict:
                for key in dict.keys():
                    fw.writelines(key + ': ' + str(dict.get(key)) + '\n')
        else:
            for key in list_dict.keys():
                fw.writelines(key + ': ' + str(list_dict.get(key)) + '\n')
        fw.close()
# 保存超参数字典到txt        
save_dict_as_txt(self.hparams, save_dir)
```

#### 梯度剪裁

当需要避免发生梯度爆炸时，可以采用梯度剪裁的方法，这个梯度范数是通过所有的模型权重计算出来的：

```python
# 默认不剪裁
trainer = Trainer(gradient_clip_val=0)

# 梯度范数的上限为0.5
trainer = Trainer(gradient_clip_val=0.5)
```

#### 设置训练的最小和最大epochs

默认最小训练1个epoch，最大训练1000个epoch。

```python
trainer = Trainer(min_epochs=1, max_epochs=1000)
```

#### 小数据集

当我们的数据集过大或者当我们进行`debug`时，不想要加载整个数据集，则可以只加载其中的一小部分：

默认是全部加载，即下面的参数值都为`1.0`

```python
# 参训练集、校验集和测试集分别只加载 10%, 20%, 30%，或者使用int 型表示batch
trainer = Trainer(
    limit_train_batches=0.1,
    limit_val_batches=0.2,
    limit_test_batches=0.3
)
```

其中比较需要注意的是训练集和测试集比例的设置，因为pytorch_lightning 每次validation和test时，都是会计算一个epoch，而不是一个step，因此在训练过程中，如果你的validation dataset比较大，那就会消耗大量的时间在validation上，而我们实际上只是想要知道在训练过程中，模型训练的怎么样了，不需要跑完整个epoch，因此就可以将`limit_val_batches`设置的小一些。对于test，在训练完成后，如果我们不希望对所有的数据都进行test，也可以通过这个参数来设置。

另外，该框架有个参数`num_sanity_val_steps`，用于设置在开始训练前先进行`num_sanity_val_steps`个 batch的validation，以免你训练了一段时间，在校验的时候程序报错，导致浪费时间。该参数在获得trainer的时间传入：

```text
# 默认为2个batch的validation
trainer = Trainer(num_sanity_val_steps=2)

# 关闭开始训练前的validaion,直接开始训练
trainer = Trainer(num_sanity_val_steps=0)

# 把校验集都运行一遍（可能会浪费很多时间）
trainer = Trainer(num_sanity_val_steps=-1)
```

## PL当前版本的BUG

该框架10月份才推出，有BUG才是正常的，下面给出我遇到的BUG（也有的是自己踩的坑）：

#### 多GPU CUDA设备不同步问题

在进行多GPU训练过程中，当完成一个epoch或者运行到epoch的指定百分比后，会进行validation过程，完成validation后，报了一个错：

```
RuntimeError: All input tensors must be on the same device. Received cuda:2 and cuda:0
```

我在github上发起了一个[issue](https://link.zhihu.com/?target=https%3A//github.com/PyTorchLightning/pytorch-lightning/issues/4877%23issuecomment-734905823)，有人已经修复了这个bug，在后面新版的`pytorch_lightning`中应该会被修改了，但是如果你用的时候也报这个错，可以通过下面的步骤进行修复：

[https://github.com/PyTorchLightning/pytorch-lightning/pull/4138/files/b20f383acaac4662caee86b76ec56c5c478f44a0](https://link.zhihu.com/?target=https%3A//github.com/PyTorchLightning/pytorch-lightning/pull/4138/files/b20f383acaac4662caee86b76ec56c5c478f44a0)

#### DataLoader的问题

**RuntimeError: DataLoader worker (pid(s) 6700, 10620) exited unexpectedly**

这个问题一般是多GPU跑的时候才会出现，主要是加载DataLoader的时候，`num_works=0`就可以了，另外，我在一个task里，设置的是`num_works=8`是OK了，但是到了另一个task中，图像更大了，可能是内存不够，加载数据集特别特别特别慢，几乎不动。

如遇到这个报错，减小batch_size，设置`num_works=0`，在定义trainer的时候，设置

```python
trainer = pl.Trainer(distributed_backend='ddp')
# or
trainer = pl.Trainer(distributed_backend='dp')
```

## 一键导出模型训练过程中记录的loss

在训练过程当中,我们会用 `self.log` 函数去把训练的loss 和 校验的loss 等信息保存起来,下面就是我写的代码,一键导出某个训练模型中的所有log 数据,支持一键保存成excel,不同的loss 保存到同一个excel 文件中的不同表格,也提供了将excel 文件保存成 .mat 文件,方便用于matlab 的绘图放到论文中. 不需要 tensorflow, 有 tensorboard 就可以了

[烟酒僧：Pytorch-lightning tensorboard导出数据（代码）6 赞同 · 1 评论文章](https://zhuanlan.zhihu.com/p/339074010)
