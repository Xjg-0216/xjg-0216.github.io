# ONNX

在深度学习算法开发过程中，模型训练与部署是两个环节，pytorch通常只用于训练，获得模型权重文件，而最终部署还有专门的部署平台，例如TensorRT、NCNN、OpenVINO等几十种部署推理平台。

如何将pytorch模型文件让几十种部署推理平台能接收与读取是个大问题。即使各推理平台都适配pytorch，那还有其他训练框架也要适配，是非常麻烦的。

假设有N个训练框架，M个推理框架，互相要适配，那就是O(NM)的复杂度。如果能有一种中间格式作为一个标注，能被所有框架所适配，那复杂度顺便降低为O(N+M)。

onnx就是为了降低深度学习模型从训练到部署的复杂度，由微软和meta在2017年提出的一种开放神经网络交换格式，目的在于方便的将模型从一个框架转移到另一个框架


`pytorch`模型导出`onnx`调用`torch.onnx.export`函数
`torch.onnx.export(model, args, f, export_params=True, verbose=False, training=, input_names=None, output_names=None, operator_export_type=, opset_version=None, do_constant_folding=True, dynamic_axes=None, keep_initializers_as_inputs=None, custom_opsets=None, export_modules_as_functions=False)`

`model`: 需要被转换的模型，可以有三种类型， `torch.nn.Module`, `torch.jit.ScriptModule` or `torch.jit.ScriptFunction`
`args`：`model`输入时所需要的参数，这里要传参时因为构建计算图过程中，需要采用数据对模型进行一遍推理，然后记录推理过程需要的操作，然后生成计算图。`args`要求是tuple或者是Tensor的形式。一般只有一个输入时，直接传入`Tensor`，多个输入时要用`tuple`包起来。
`export_params`: 是否需要保存参数。默认为`True`，通常用于模型结构迁移到其它框架时，可以用`False`。
`input_names`：输入数据的名字， `(list of str, default empty list)` ，在使用`onnx`文件时，数据的传输和使用，都是通过`name: value`的形式。
`output_names`：同上。
`opset_version`：使用的算子集版本。
`dynamic_axes`：动态维度的指定，例如`batchsize`在使用时随时会变，则需要把该维度指定为动态的。默认情况下计算图的数据维度是固定的，这有利于效率提升，但缺乏灵活性。用法是，对于动态维度的输入、输出，需要设置它哪个轴是动态的，并且为这个轴设定名称。这里有3个要素，数据名称，轴序号，轴名称。因此是通过`dict`来设置的。例如`dynamic_axes={ "x": {0: "my_custom_axis_name"} }`， 表示名称为`x`的数据，第0个轴是动态的，动态轴的名字叫`my_custom_axis_name`。通常用于`batchsize`或者是对于`h,w`是不固定的模型要设置动态轴。

下面以`ResNet18`进行举例，`onnx`文件可通过以下代码获得：
```python
import torchvision
import torch

model = torchvision.models.resnet18(pretrained=True)
dummy_data = torch.randn((1, 3, 512, 512))
with torch.no_grad():
    torch.onnx.export(model, (dummy_data), "resnet18.onnx",
                    opset_version=19,
                    input_names=["input0"],
                    output_names=["output0"])

```


onnxsim-让导出的onnx模型更精简
仓库地址： [https://github.com/daquexian/onnx-simplifier](https://github.com/daquexian/onnx-simplifier)

网页版：
在 convertmodel.com 上发布了 ONNX Simplifier。它开箱即用，不需要任何安装。请注意，它在本地浏览器中运行，您的模型是完全安全的。

本地版:
`pip install onnxsim`
然后，在终端：
`onnxsim input_onnx_model output_onnx_model`

