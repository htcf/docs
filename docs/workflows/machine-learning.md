# Machine Learning Workflows

## Overview

Complete guide for running machine learning and deep learning workloads on HTCF GPUs.

**Official Documentation:**

- [PyTorch Tutorials](https://pytorch.org/tutorials/) - PyTorch official tutorials
- [TensorFlow Tutorials](https://www.tensorflow.org/tutorials) - TensorFlow official tutorials
- [Hugging Face Documentation](https://huggingface.co/docs) - Transformers, datasets, and model hub
- [scikit-learn User Guide](https://scikit-learn.org/stable/user_guide.html) - Classical ML algorithms
- [Lightning AI Documentation](https://lightning.ai/docs/pytorch/stable/) - PyTorch Lightning framework

## Setup

### Load PyTorch
```bash
eval $(spack load --sh cuda@12.0)
eval $(spack load --sh py-pytorch+cuda)

# Verify
python -c "import torch; print(torch.cuda.is_available())"
```

## Example 1: Single-GPU Training

### Training Script
```python
#!/usr/bin/env python3
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
print(f"Using device: {device}")

# Simple neural network
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.fc1 = nn.Linear(784, 512)
        self.fc2 = nn.Linear(512, 10)

    def forward(self, x):
        x = torch.relu(self.fc1(x))
        return self.fc2(x)

# Training loop
model = Net().to(device)
optimizer = optim.Adam(model.parameters())
criterion = nn.CrossEntropyLoss()

for epoch in range(10):
    for batch_data, batch_labels in train_loader:
        batch_data = batch_data.to(device)
        batch_labels = batch_labels.to(device)

        optimizer.zero_grad()
        outputs = model(batch_data)
        loss = criterion(outputs, batch_labels)
        loss.backward()
        optimizer.step()

    print(f"Epoch {epoch+1}, Loss: {loss.item():.4f}")

# Save model
torch.save(model.state_dict(), 'model.pth')
```

### Job Script
```bash
#!/bin/bash

#SBATCH --job-name=ml_train
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=4:00:00
#SBATCH --output=train_%j.out

eval $(spack load --sh cuda py-pytorch)

python train.py
```

## Example 2: Multi-GPU Training

```python
import torch
import torch.nn as nn
from torch.nn.parallel import DataParallel

model = Net()
if torch.cuda.device_count() > 1:
    print(f"Using {torch.cuda.device_count()} GPUs")
    model = DataParallel(model)
model = model.cuda()

# Training loop same as before
```

### Multi-GPU Job
```bash
#!/bin/bash

#SBATCH --job-name=ml_multigpu
#SBATCH -p gpu
#SBATCH --gpus=4            # Request 4 GPUs
#SBATCH --cpus-per-task=16
#SBATCH --mem=128G
#SBATCH --time=8:00:00

eval $(spack load --sh cuda py-pytorch)

python train_multigpu.py
```

## Example 3: Hyperparameter Search

Use array jobs for parallel hyperparameter search:

```bash
#!/bin/bash

#SBATCH --job-name=hp_search
#SBATCH -p gpu
#SBATCH --gpus=1
#SBATCH --array=1-20        # 20 different configurations
#SBATCH --mem=16G
#SBATCH --time=2:00:00

eval $(spack load --sh cuda py-pytorch)

# Get hyperparameters for this array task
LR=$(sed -n "${SLURM_ARRAY_TASK_ID}p" learning_rates.txt)
BS=$(sed -n "${SLURM_ARRAY_TASK_ID}p" batch_sizes.txt)

python train.py --lr $LR --batch-size $BS --output results_${SLURM_ARRAY_TASK_ID}.json
```

## Best Practices

### Data Loading
```python
# Increase workers for faster data loading
train_loader = DataLoader(
    dataset,
    batch_size=64,
    num_workers=4,      # Matches --cpus-per-task
    pin_memory=True     # Faster GPU transfer
)
```

### Mixed Precision Training
```python
from torch.cuda.amp import autocast, GradScaler

scaler = GradScaler()

for batch in train_loader:
    with autocast():
        output = model(batch)
        loss = criterion(output, labels)

    scaler.scale(loss).backward()
    scaler.step(optimizer)
    scaler.update()
```

### Checkpointing
```python
# Save checkpoint
torch.save({
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': loss,
}, f'checkpoint_epoch_{epoch}.pth')

# Load checkpoint
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
start_epoch = checkpoint['epoch'] + 1
```

## Troubleshooting

**Out of Memory:** Reduce batch size or use gradient accumulation
**Low GPU Utilization:** Increase batch size or num_workers
**Slow Training:** Use mixed precision, optimize data loading

--8<-- "includes/getting-help.md"
