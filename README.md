目标

提供简单，模块化，高可读性的代码
保证训练吞吐最优，默认开启且仅支持最优配置
其他


尽量使用面向对象的编程
训练数据格式：统一使用 kanyun dataset
补充单元测试和集成测试
未来将会逐步抛弃 megatron.training，只使用 megatron.core


目录结构

.
├── data  # 数据处理
├── examples  # 示例代码
├── huggingface  # huggingface 代码，比如多模态的 processor
├── megatron_patch  # 对 megatron 的 patch
├── model  # 模型
├── model_convert  # 模型转换
├── scripts  # 脚本
├── tests  # 测试
├── training_utils  # 训练工具
├── train.py  # 训练入口



样例脚本
当前版本基于 Megatron-core V0.13.0, Transformer Engine V2.4

cd examples
bash train_vision.sh # qwen2.5-vl 视觉多模态训练
bash pretrain.sh # qwen2.5 文本模态预训练
bash convert_to_megatron.sh # 将 huggingface 模型转换为 megatron 模型
bash convert_to_huggingface.sh # 将 megatron 模型转换为 huggingface 模型



数据准备教程

数据预处理
代码库：https://gitlab-ee.zhenguanyu.com/yuanli/nlp-mllm-common-data

每个数据集执行单独的 python 文件

读取数据文件，构建 dataset
使用 map 处理每一条样本，处理过程中：

判断图片是否存在/长或宽小于28/长宽比大于200/processor失败；
图片后的文本是否过短。


使用 filter 过滤错误样本
保存在 /mnt/data/nlp-mllm-data/common-data/{dataset_name}




数据采样（本地）
代码库：https://gitlab-ee.zhenguanyu.com/yuanli/nlp-mllm-common-data

在 common/contant.py 中设置各类别设定采样的比例
使用  export HF_DATASETS_CACHE="" 禁用 datasets 缓存
执行 python prepare_data/sample_with_ratio.py 采样
采样结果保存在 /mnt/data/nlp-mllm-data/sample-result/{date}


数据采样（spark）
代码库：https://gitlab-ee.zhenguanyu.com/yuanli/nlp-mllm-common-data

将预处理好的数据上传到 hdfs  hadoop fs -put -t 32 YourLocalPath hdfs://ky02/research/nlp-mllm-data/sample/xxx

在 common/contant.py 中设置各类别设定采样的比例 SAMPLE_RATIO_BY_DATA_PATH
执行 bash scripts/spark_sample.sh hdfs://ky02/research/nlp-mllm-data/sample/xxx hdfs://ky02/research/yourname/sample/xxx

结果保存在hdfs上，不需要拉下来


数据构建（本地）
代码库：https://gitlab-ee.zhenguanyu.com/yuanli/nlp-llm-train
构建成实际训练的数据集。如果数量不多，可以本地构建。参考 example/build_dataset.sh

数据构建+packing（spark）
当数据量为千万级时，推荐使用spark，build+packing。
环境要求

使用火山集群
使用nlp-llm镜像
测试命令 hadoop fs -ls hdfs://ky02/research


上传数据 （如果数据采样sample也是spark则不需要上传）
把本地 jsonl 传到 hdfs

hadoop fs -put xxx.jsonl hdfs://ky02/research/yourname/xxxxxxx/


build
参考 example/spark_build_dataset.sh
packing & 下载到本地
参考 example/spark_packing.sh
下载到本地后，会保存为
/mnt/data/yantanbj/data/xxxx/packed_length_8192_0_epoch_0_v4
后面训练时填写 /mnt/data/yantanbj/data/xxxx 即可

模型训练教程
