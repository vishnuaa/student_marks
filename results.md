# SAM 2 (Segment Anything Model 2) Results

**Model Description:** An upgraded segmentation framework. Testing revealed that the SAM 2 Base model outperformed the Large model for tight bounding box crops on leaves, serving as the "sweet spot" before scaling class complexity.

### Performance Metrics

| Dataset Used | Classes | Validation Accuracy | Testing Accuracy | Notes |
| :--- | :--- | :--- | :--- | :--- |
| Normal PlantVillage + Seg PlantDoc | 4 (Base) | 74.00% | 69.00% | Best performing SAM 2 setup. |
| Normal PlantVillage + Seg PlantDoc | 4 (Large) | 70.00% | 59.00% | Large model underperformed compared to Base. |
| Normal PlantVillage + Seg PlantDoc | 13 (Base) | 55.00% | 51.00% | Accuracy dropped when scaling up with 9 tomato classes. |
