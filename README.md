# brain-tumor-segmentation-pytorch
Comparative implementation of U-Net, Attention U-Net, and CBAM-based architectures for LGG brain MRI tumor segmentation using PyTorch.

# LGG MRI Segmentation – Attention U-Net

This repository contains experiments for segmenting **Lower Grade Glioma (LGG) MRI** images using U-Net architectures, including **Attention U-Net** with Focal Tversky loss and hyperparameter optimization.

---

## 1️⃣ Dataset

- **Source:** [Kaggle LGG MRI Segmentation Dataset](https://www.kaggle.com/mateuszbuda/lgg-mri-segmentation)
- **Structure:** Each patient folder contains MRI images (`.tif`) and corresponding masks (`*_mask.tif`)
- **Preprocessing:**
  - Resize images and masks to **128×128**
  - Normalize images to `[0,1]`
  - Convert masks to **binary masks**
- **Split:** 
  - **Training:** 80%
  - **Validation:** 20%
- **DataLoaders:** Batch size = 8, training loader shuffled, validation loader sequential

---

## 2️⃣ Models

### a) UNetSmall / UNet

- Standard U-Net architecture:
  - Encoder: 3 ConvBlocks with MaxPool
  - Bottleneck
  - Decoder: 3 upsampling layers with skip connections
- Input: `1-channel MRI`  
- Output: `1-channel segmentation mask`  

**Training output sample (Dice progression):**  

- Dice increased from **~0.17 → 0.68** over 100 epochs  
- Loss decreased from **~0.54 → 0.15**
- Stable convergence and good segmentation performance

---

### b) Attention U-Net

- Encoder-Decoder architecture with **skip connections**  
- **Attention blocks** highlight important features at skip connections
- Loss: BCE + Focal Tversky  
- Optimizer: Adam, `lr=1e-4`

**Validation Results Sample:**

| Epoch | Loss | Train Dice | Val Dice | Val IoU |
|-------|------|------------|----------|---------|
| 1     | 0.51 | 0.37       | 0.20     | 0.25    |
| 10    | 0.09 | 0.70       | 0.56     | 0.55    |
| 25    | 0.06 | 0.77       | 0.61     | 0.62    |
| 50    | 0.05 | 0.79       | 0.82     | 0.68    |

- Early stopping triggered after validation Dice plateau  
- Training is stable, final Dice ~0.82, IoU ~0.68  

---

## 3️⃣ Loss Functions

- **DiceLoss + BCEWithLogitsLoss:** Measures pixel-wise overlap and classification accuracy
- **Focal Tversky Loss:** Penalizes false positives/negatives; emphasizes hard pixels
- Weighted sum: `0.5 * BCE + 0.5 * Focal Tversky`

---

## 4️⃣ Metrics

- **Dice Coefficient:** Measures segmentation overlap  
- **IoU (Intersection over Union):** Complementary metric  
- Both metrics computed after applying sigmoid and threshold 0.5

---

## 5️⃣ Hyperparameter Optimization

- **Differential Evolution (DE)** used to optimize learning rate  
- Candidate LRs sampled log-uniformly in `[1e-5, 1e-3]`  
- Validation Dice used as fitness  
- Best LR found: **0.000295**  
- Accelerated convergence and improved Dice

**Sample DE Output:**


---

## 6️⃣ Visualization

- Side-by-side plots for qualitative inspection:
  1. MRI Input
  2. Ground Truth Mask
  3. Model Predicted Mask
- Confirms Attention U-Net can segment tumor regions effectively

---

## 7️⃣ Summary / Results

| Model | Best Val Dice | Best Val IoU | Notes |
|-------|---------------|--------------|-------|
| UNetSmall / UNet | 0.68 | 0.65 | Stable training; baseline model |
| Attention U-Net | 0.82 | 0.68 | Attention mechanism improves segmentation; early stopping used |
| Attention U-Net + DE LR tuning | 0.6869 | N/A | Optimized LR; faster convergence |

**Observations:**

- Attention U-Net outperforms standard U-Net  
- BCE + Focal Tversky Loss helps Dice for small tumor regions  
- Early stopping prevents overfitting and maintains high validation Dice  

---

## 8️⃣ Future Work

- Experiment with larger input sizes (e.g., 256×256)  
- Use **full 3D MRI volumes** for volumetric segmentation  
- Try alternative attention mechanisms or residual blocks  
- Fine-tune hyperparameters (batch size, optimizer type, etc.)  

---

## 9️⃣ References

- Mateusz Buda et al., *LGG MRI Segmentation Dataset*, Kaggle  
- Ronneberger et al., *U-Net: Convolutional Networks for Biomedical Image Segmentation*, MICCAI 2015
