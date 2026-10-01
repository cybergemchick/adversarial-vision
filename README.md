# Adversarial Vision Attacks

FGSM and PGD adversarial examples against a pretrained ResNet-50, plus test images for visual prompt injection against multimodal LLMs.

**Built by:** [CyberGemChick](https://github.com/cybergemchick) | AI Red Team

## Problem

Applications trust the output of image models and multimodal LLMs. Two ways that trust can fail:

1. A tiny, bounded change to the pixels changes a classifier's decision.
2. Text inside an image is treated as an instruction instead of data.

## Threat model

| | Part 1: FGSM and PGD | Part 2: Visual prompt injection |
|---|---|---|
| Attacker access | White-box: model weights and gradients | Can place an image or scanned document in the victim's pipeline |
| Goal | Make the classifier predict anything other than the true class | Make a multimodal LLM follow text hidden in an image |
| Constraint | Per-pixel change of at most epsilon (pixel units, for example 4/255) | Text must be hard for a human to notice |
| MITRE ATLAS | AML.T0043 Craft Adversarial Data, AML.T0043.000 White-Box Optimization | AML.T0051.001 LLM Prompt Injection: Indirect |

## Method

**Part 1.** FGSM takes one signed-gradient step. PGD takes 10 smaller steps (alpha = epsilon / 4) and projects back into the epsilon box each time, with no random start. Epsilon is set at 1, 2, 4, 8 and 16 out of 255, and both attacks are clamped to the valid image range. The notebook asserts that PGD never exceeds its epsilon budget.

**Part 2.** The notebook generates a clean document image, a low-contrast injection and an LSB-steganography injection. The payload only asks a model to reply with the canary string `INJECTION-CANARY-7421`, so no harmful content is involved. It also measures two properties directly: the largest pixel difference of the low-contrast text, and whether the LSB message survives JPEG and resizing.

## Results

**Executed results are not included yet.** The notebook ships without saved outputs. To produce them, open `adversarial_vision_attacks.ipynb` in Colab or Jupyter, run all cells, and commit the executed notebook. The tables to look at:

- FGSM vs PGD at each epsilon: whether the top-1 prediction changed and the true-class confidence.
- LSB survival: expected to fail after JPEG and resizing, which is the finding that shows it is not a practical vector against hosted models.

What has been checked without the real model (random weights, synthetic image): the perturbation never exceeds epsilon, outputs stay in the valid pixel range, FGSM raises the loss, PGD raises it at least as much as FGSM, and the LSB round trip works on PNG and fails after JPEG and resizing.

## Security implication

Classifier confidence is not evidence that an input is untampered. Any pipeline that feeds images to an LLM should treat text found in images as untrusted data, and re-encode or resize images at ingestion.

## Run

```bash
pip install torch torchvision matplotlib numpy Pillow
jupyter notebook adversarial_vision_attacks.ipynb
```

The notebook downloads the ImageNet labels and one sample image, and the ResNet-50 weights through torchvision.

## Limitations

- One model (ResNet-50) and one image by default. Results do not generalize to other models or to moderation systems.
- Part 2 does not send images to any model. Test your own authorized target with the generated PNG files.
- The low-contrast technique is model-dependent and unverified here.

## References

- Goodfellow et al., [Explaining and Harnessing Adversarial Examples](https://arxiv.org/abs/1412.6572) (2014)
- Madry et al., [Towards Deep Learning Models Resistant to Adversarial Attacks](https://arxiv.org/abs/1706.06083) (2017)
- Qi et al., [Visual Adversarial Examples Jailbreak Aligned Large Language Models](https://arxiv.org/abs/2306.13213) (2023)
- [MITRE ATLAS](https://atlas.mitre.org/)
