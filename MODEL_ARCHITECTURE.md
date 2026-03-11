# GenAuth AI — Model Architecture

## Module 1: Face Recognition Engine

### ArcFace-R100
- **Architecture**: ResNet-100 backbone + ArcFace additive angular margin loss
- **Embedding dim**: 512 (L2-normalized)
- **Training data**: MS1MV3 (5.8M images, 93K identities)
- **Accuracy**: 99.83% @ LFW, 98.47% @ MegaFace
- **Latency**: 48ms (A100 GPU), 280ms (CPU)
- **Anti-spoofing**: Depth map + texture analysis + blink detection

### FaceNet-512
- **Architecture**: Inception-ResNet-V1 + triplet loss
- **Embedding dim**: 512
- **Training data**: VGGFace2 (3.3M images, 9K identities)
- **Accuracy**: 99.65% @ LFW

---

## Module 2: DNA Processing Engine

### DNA-STR-Encoder
- **Input**: Raw DNA sequence / FASTA / STR allele calls
- **Loci**: CODIS-20 standard STR loci
- **Output**: 256-dim float encoding vector
- **Matching**: Forensic CPI (Combined Probability of Inclusion)
- **Accuracy**: 99.4% true match rate, 0.001% false positive

---

## Module 3: DNA→Face Prediction

### GeneticDiffuser-v0.8
- **Architecture**: Latent Diffusion Model (LDM) + CLIP text/genetic conditioning
- **Base model**: Stable Diffusion v1.5
- **Conditioning**: ControlNet with genetic trait vectors
- **Output**: 512×512 photorealistic face (expandable to 1024×1024 with upscaler)
- **Inference**: 30 DDPM steps, classifier-free guidance scale 7.5
- **Phenotype accuracy**: 87% on held-out genetic-phenotype pairs

### Predicted Traits
| Trait | Accuracy | Method |
|-------|----------|--------|
| Skin tone | 91% | SNP regression (MC1R, SLC24A5, OCA2) |
| Eye color | 89% | HIRISP+ model (6 SNPs) |
| Hair color | 85% | IrisPlex-S (41 SNPs) |
| Face structure | 78% | GWAS landmarks |
| Age estimate | 72% | Epigenetic clock markers |

---

## Module 4: Biometric Fusion

### Decision Engine
```
biometric_score = 0.60 × face_confidence + 0.40 × dna_confidence

GRANT_ACCESS if:
  face_confidence >= 0.80
  AND dna_confidence >= 0.85
  AND liveness_score >= 0.75
  AND biometric_score >= 0.82
```

### Security Layers
1. Anti-spoofing (liveness detection)
2. Deepfake detection (EfficientNet-B7 classifier)
3. Replay attack prevention (session nonce + timestamp)
4. AES-256 biometric data encryption
5. Zero-knowledge proof for identity verification
