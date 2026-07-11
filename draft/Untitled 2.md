这里为您提供一份使用 PyTorch 实现的手写数字识别代码。

该代码实现了一个包含一个隐藏层和一个输出层（即通常所说的**二层神经网络**）的模型，使用随机梯度下降（SGD）更新参数，数据集直接采用 `torchvision.datasets` 导入的 MNIST。代码设置了 mini-batch 大小为 100，总共训练 10,000 次（即 10,000 个 mini-batch 迭代），每训练 100 次计算并记录一次测试集上的准确率，并在最后绘制出准确率的变化图表。

### 准备工作
在运行代码之前，请确保已安装以下依赖库：
```bash
pip install torch torchvision matplotlib
```

### Python 实现代码

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torchvision import datasets, transforms
from torch.utils.data import DataLoader
import matplotlib.pyplot as plt

# 1. 定义超参数
INPUT_SIZE = 784       # 输入层大小 (28x28 像素)
HIDDEN_SIZE = 128      # 隐藏层神经元个数
OUTPUT_SIZE = 10       # 输出层大小 (0-9 共10个数字)
BATCH_SIZE = 100       # Mini-batch 大小
TOTAL_STEPS = 10000    # 总训练次数（迭代步数）
EVAL_INTERVAL = 100    # 每 100 次训练评估一次准确率
LEARNING_RATE = 0.1    # 学习率

# 检测设备 (若有GPU则使用GPU加速)
device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# 2. 准备 MNIST 数据集
# 对图像进行归一化处理
transform = transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize((0.1307,), (0.3081,))
])

train_dataset = datasets.MNIST(root='./data', train=True, download=True, transform=transform)
test_dataset = datasets.MNIST(root='./data', train=False, download=True, transform=transform)

train_loader = DataLoader(train_dataset, batch_size=BATCH_SIZE, shuffle=True)
# 测试时可一次性评估或分批评估，这里采用 batch_size=1000 提高评估效率
test_loader = DataLoader(test_dataset, batch_size=1000, shuffle=False)


# 3. 构建二层神经网络
class TwoLayerNet(nn.Module):
    def __init__(self, input_size, hidden_size, output_size):
        super(TwoLayerNet, self).__init__()
        # 第一层（输入层到隐藏层）
        self.fc1 = nn.Linear(input_size, hidden_size)
        # 激活函数
        self.relu = nn.ReLU()
        # 第二层（隐藏层到输出层）
        self.fc2 = nn.Linear(hidden_size, output_size)

    def forward(self, x):
        # 将输入的三维图像 (batch, 1, 28, 28) 展平为二维 (batch, 784)
        x = x.view(x.size(0), -1)
        x = self.relu(self.fc1(x))
        x = self.fc2(x)
        return x


model = TwoLayerNet(INPUT_SIZE, HIDDEN_SIZE, OUTPUT_SIZE).to(device)

# 4. 定义损失函数和优化器
criterion = nn.CrossEntropyLoss()
# 使用随机梯度下降法 (SGD) 更新参数
optimizer = optim.SGD(model.parameters(), lr=LEARNING_RATE)


# 5. 定义模型评估函数
def evaluate_accuracy(model, data_loader, device):
    model.eval()  # 设置为评估模式
    correct = 0
    total = 0
    with torch.no_grad():
        for images, labels in data_loader:
            images, labels = images.to(device), labels.to(device)
            outputs = model(images)
            _, predicted = torch.max(outputs.data, 1)
            total += labels.size(0)
            correct += (predicted == labels).sum().item()
    return 100.0 * correct / total


# 6. 开始训练过程
steps = []
accuracies = []

current_step = 0
running_loss = 0.0
finished = False

print("开始训练...")
while not finished:
    for images, labels in train_loader:
        if current_step >= TOTAL_STEPS:
            finished = True
            break

        model.train()  # 设置为训练模式
        images, labels = images.to(device), labels.to(device)

        # 前向传播
        outputs = model(images)
        loss = criterion(outputs, labels)

        # 反向传播与优化
        optimizer.zero_grad()
        loss.backward()
        optimizer.step()

        running_loss += loss.item()
        current_step += 1

        # 每训练 100 次，计算一次在测试集上的准确率
        if current_step % EVAL_INTERVAL == 0:
            avg_loss = running_loss / EVAL_INTERVAL
            accuracy = evaluate_accuracy(model, test_loader, device)
            
            steps.append(current_step)
            accuracies.append(accuracy)
            
            print(f"Step [{current_step}/{TOTAL_STEPS}] | Loss: {avg_loss:.4f} | Test Accuracy: {accuracy:.2f}%")
            running_loss = 0.0

print("训练完成。")

# 7. 绘制准确率随训练变化的图表
plt.figure(figsize=(10, 6))
plt.plot(steps, accuracies, linestyle='-', color='b', label='Test Accuracy')
plt.title('Accuracy vs. Training Steps (MNIST)')
plt.xlabel('Training Steps (Mini-batches)')
plt.ylabel('Accuracy (%)')
plt.grid(True)
plt.legend()
plt.show()
```

### 代码逻辑说明：
1. **数据准备**：通过 `torchvision.datasets` 下载并加载 MNIST 图像，对图像数据进行了归一化处理。由于 MNIST 单个 epoch（一轮完整训练）包含 60,000 张图像，在 Batch Size 为 100 的情况下，一个 epoch 对应 600 次迭代。10,000 次训练意味着代码会循环读取数据集大约 16.7 个 epoch。
2. **网络结构**：`TwoLayerNet` 包含一个隐藏层（128 个神经元）和一个输出层（10 个神经元），在隐藏层后应用了非线性激活函数 ReLU。输入数据在进入全连接层前被展平（`view`）为大小为 784 的一维向量。
3. **参数更新**：采用标准的 `optim.SGD` 优化器来实现随机梯度下降。
4. **评估与记录**：在外部设置了 `current_step` 计数器，当达到 `TOTAL_STEPS` (10,000 次) 时程序停止。每迭代 100 次，模型会被切换到评估模式 `model.eval()`，在测试集上计算准确率并记录到列表中，最后使用 `matplotlib` 绘制折线图展示准确率的变化趋势。