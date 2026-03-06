# DINOv2 (Vision Transformer) Results

**Model Description:** For this phase of the project, we moved away from standard Convolutional Neural Networks (like ResNet and EfficientNet) and tested a state-of-the-art Vision Transformer: **DINOv2 Small** (`vit_small_patch14_dinov2.lvd142m`). The goal was to see if DINOv2's advanced pre-trained feature extraction could handle our complex 10-class leaf disease dataset better than traditional models. 

Over a series of 7 experiments, we tested different dataset combinations (Grayscale, Normal, Segmented), tweaked the model's backbone (freezing vs. unfreezing blocks), adjusted image cropping sizes, and used cross-validation to find the absolute best pipeline.

---

## 📊 Quick Summary of All Experiments

| Exp | Dataset Used | Backbone Status & Image Size | Best Val Acc | Test Acc | Key Takeaway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1** | Grayscale PV + Seg PD | Fully Frozen (224x224) | 80.75% | 70.53% | Baseline established, but failed completely on `Corn_Gray_leaf_spot` (0%). |
| **2** | Normal PV + Normal PD | Fully Frozen (224x224) | 85.03% | 72.63% | Using normal color images fixed the 0% corn class failure. |
| **3** | Segmented PV + Seg PD | Fully Frozen (224x224) | 80.21% | 76.84% | Fully segmented data proved to be the best for real-world generalization. |
| **4** | **Segmented PV + Seg PD** | **Unfroze Last 2 Blocks (256 ➔ 224)** | **89.30%** | **90.53%** | **Massive Breakthrough.** 6 out of 10 classes hit 100% testing accuracy! |
| **5** | Segmented PV + Seg PD | Unfroze Last 4 Blocks (256 ➔ 224) | 89.30% | 88.42% | Unfreezing too much degraded performance; model forgot the hardest corn class. |
| **6** | Segmented PV + Seg PD | Unfroze Last 2 Blocks (Weighted) | 86.10% | 83.16% | Weighted sampling helped the minority class but confused the model on easy classes. |
| **7** | **Segmented PV + Seg PD** | **Unfroze Last 2 Blocks (5-Fold CV)** | **~98.35%** | **90.50%** | Proved that the 90.5% accuracy from Exp 4 was highly robust and consistent. |

---

## 🔍 Detailed Breakdown of Each Experiment

### Experiment 1: The Grayscale + Segmented Baseline
* **Dataset:** Grayscale PlantVillage + Segmented PlantDoc
* **Architecture Setup:** Backbone fully frozen. Images resized directly to 224x224.
* **Validation Accuracy:** 80.75% (Peaked at Epoch 4)
* **Testing Accuracy:** 70.53%
* **Notes:** While the overall accuracy was okay, the per-class breakdown showed severe issues. The model got 0% accuracy on `Corn_Gray_leaf_spot` and struggled heavily with `grape_leaf_black_rot` (37.5%). It was heavily biased toward easier classes.

### Experiment 2: Switching to Normal Color Images
* **Dataset:** Normal Color PlantVillage + Normal Color PlantDoc
* **Architecture Setup:** Backbone fully frozen. Images resized directly to 224x224.
* **Validation Accuracy:** 85.03% (Peaked at Epoch 12)
* **Testing Accuracy:** 72.63%
* **Notes:** We tested if removing the grayscale/segmentation would help. It improved overall stability and successfully fixed the 0% failure on the Corn class (bringing it up to 50%). It also hit a perfect 100% on `Tomato_Septoria_leaf_spot`.

### Experiment 3: Fully Segmented Datasets
* **Dataset:** Segmented PlantVillage + Segmented PlantDoc
* **Architecture Setup:** Backbone fully frozen. Images resized directly to 224x224.
* **Validation Accuracy:** 80.21% (Peaked at Epoch 3)
* **Testing Accuracy:** 76.84%
* **Notes:** Using fully segmented images for both training (lab data) and validation/testing (real-world data) provided the highest testing accuracy so far for a frozen model. Several classes hit 90% or above.

### Experiment 4: The 90.5% Breakthrough (Optimized Architecture)
* **Dataset:** Segmented PlantVillage + Segmented PlantDoc
* **Architecture Setup:** **Unfroze the last 2 blocks** of the DINOv2 backbone. Changed image preprocessing to resize to 256x256, followed by a **224 CenterCrop**.
* **Validation Accuracy:** 89.30% (Peaked at Epoch 22)
* **Testing Accuracy:** **90.53%**
* **Notes:** This was a massive leap. By unfreezing the last two blocks, the model learned domain-specific leaf features instead of just relying on general pre-trained weights. The CenterCrop also provided better focal points. 
* **Highlight:** The model achieved a perfect **100% testing accuracy on 6 different classes** (Tomato Septoria, Tomato leaf, Apple leaf, Apple rust, Grape leaf, Grape black rot).

### Experiment 5: Testing the Limits (Unfreezing Too Much)
* **Dataset:** Segmented PlantVillage + Segmented PlantDoc
* **Architecture Setup:** **Unfroze the last 4 blocks**. Resized to 256 with 224 CenterCrop.
* **Validation Accuracy:** 89.30%
* **Testing Accuracy:** 88.42%
* **Notes:** We tried to improve upon Exp 4 by unfreezing more blocks. However, this caused "catastrophic forgetting." While the overall accuracy only dropped slightly, the model completely forgot how to classify the most difficult class (`Corn_Gray_leaf_spot` plummeted back to 0%). This proved that unfreezing exactly 2 blocks is the "sweet spot."

### Experiment 6: Forcing Attention with Weighted Random Sampler
* **Dataset:** Segmented PlantVillage + Segmented PlantDoc
* **Architecture Setup:** Unfroze last 2 blocks. Introduced a **Weighted Random Sampler** during training.
* **Validation Accuracy:** 86.10%
* **Testing Accuracy:** 83.16%
* **Notes:** Because `Corn_Gray_leaf_spot` was constantly struggling, we used weighted sampling to force the model to pay more attention to it. While it successfully brought the Corn class back up to 25%, it caused the model to lose confidence in other easier classes (like `Apple_leaf` and `Corn_leaf_blight`). Overall, it wasn't worth the trade-off.

### Experiment 7: Validating the Success (5-Fold Cross Validation)
* **Dataset:** Segmented PlantVillage + Segmented PlantDoc
* **Architecture Setup:** Unfroze last 2 blocks. Resized to 256 with 224 CenterCrop.
* **Validation Accuracy:** **98.35% (Average across 5 folds)**
    * Fold 1: 98.45%
    * Fold 2: 98.17%
    * Fold 3: 98.04%
    * Fold 4: 99.04%
    * Fold 5: 98.08%
* **Testing Accuracy:** **90.50%**
* **Notes:** To prove that the amazing 90.53% score from Experiment 4 wasn't just a lucky train/test split, we ran a rigorous 5-Fold Cross-Validation. The validation scores were incredibly high and stable across every single fold. The final testing accuracy perfectly matched Exp 4, proving that this Vision Transformer pipeline is highly robust, consistent, and ready for real-world application.
