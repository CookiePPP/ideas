# ideas

---

๐ฅ๐ง๐จ๐ฉ๐ฆ๐ช๐ซโฌโฌ๐ฒ๐ณโนโโโ

---

## ๐ฅ PnGnW BERT (Phoneme and Grapheme and Word BERT)

![image](https://user-images.githubusercontent.com/42448678/218235075-a1731746-bcb5-4d1f-83ce-a18ebe144539.png)

In the PnG BERT paper they note that 

> the benefit of PnG BERT primarily lies in better natural language understanding, rather than potential improvements on G2P conversion.

For an emotive TTS downstream task, I imagine the understanding of the text is far more important than the G2P element.

So I propose some changes that may or may not improve the model.

---

1. Use both word-level and character-level embeddings.

The paper already finds a significant benefit from adding word-level position encoding. I imagine adding word-level value embeddings is just as important for understanding the text.

[Text-writing diffusion](https://github.com/nostalgebraist/improved-diffusion) and [Parti](https://parti.research.google/) both show that the task of WORD-ID to letters is extremely challenging, with [Parti](https://parti.research.google/) needing 20 BILLION parameters for this emergent ability and [Text-writing diffusion](https://github.com/nostalgebraist/improved-diffusion) learning the ability to write using far less parameters once a character-level encoder is added to the architecture.

Also [Mixed-Phoneme BERT](https://arxiv.org/pdf/2203.17190.pdf) who found a large benefit from using phoneme AND subphoneme value embeddings. (Note - MPBERT doesn't use subphone-level position encoding. another thing to test.)

---

2. Additional teachers.

PnG BERT doesn't use ground truth phoneme data, it actually uses a pretrained text -> phoneme model. Because of this, the model is effectively just a student of a G2P model that also has to learn MLM objective.

The paper shows that despite not being ground truth data, it still improves downstream performance, so I propose adding an emotion classification objective using the latents of pretrained model [DeepMoji](https://github.com/bfelbo/DeepMoji) as ground truth 'emotion' for each sample, and training PnG BERT to also predict this latent.

---

3. More varied dataset

PNG BERT uses Wikipedia as their pretraining dataset which makes perfect sense given their model needs to see a large variety of vocab in order to learn from it's G2P teacher. I propose using the [twemoji](https://uvaauas.figshare.com/articles/dataset/Twemoji_Dataset/5822100) dataset as additional training data since it contains a variety of emotion which will help PnG BERT learn from the proposed DeepMoji teacher.

---

## ๐ง Speaker Embed Enhancer

Following [Latent space crawling](https://github.com/DanRuta/xVA-Synth/commit/5325d7e1a4ffc9ecc9df55ee0c848a2b28ed6f77) to control pitch in TTS models without explicit pitch conditioning. I propose training a model that converts noisy speaker embeddings into clean ones.

The training process would be extremely simple. Take a bunch of clean audio files, calculate speaker embeddings from the original file and the same file with added noise and sound effects, then train a DDPM to predict the clean embedding from the noisy one. Classifier-Free Guidance or multiple passes can be used during inference to clean the embeddings beyond GT level (albeit, with unknown side effects).

---

## ๐จ iSTFT-WaveGrad

DDPM's show exceptional performance when modelling spectrograms [related](https://github.com/CookiePPP/papers/blob/main/README.md#grad-tts-a-diffusion-probabilistic-model-for-text-to-speech-and-diff-tts-a-denoising-diffusion-model-for-text-to-speech), however when applied to waveforms they tend to add white noise to the background of samples, and require special non-linear sampling schedules for performant inference.

[ISTFTNET](https://arxiv.org/pdf/2203.02395.pdf) shows that predicting the spectrogram+phase directly can improve performance significantly for downstream TTS, so I propose running WaveGrad on the (log)spectrogram directly, which will allow the model to output silence without special inference schedules and precise learning rates.

There is the issue of phase being a circular uniform distribution, so predicting 359ยฐ when the GT sample is 0ยฐ, would give incorrect error, however this is just an implementation detail and shouldn't be too hard to solve.

---

## ๐ฉ MOS-TTS

Following my experience with [rating images](https://github.com/CookiePPP/derpy-score-predictor) by assigning percentiles, then training Stable-Diffusion with score-percentile-given-content conditioning to bias the model towards high quality outputs, I propose MOS-TTS.

[VoiceMOS Challenge 2022](https://arxiv.org/pdf/2203.11389.pdf) showed that SOTA MOS prediction models are reaching MSE's around 0.1. With that much accuracy I believe you could train a TTS model with predicted MOS per sample or segment as conditioning, and use that conditioning to improve the output quality.

Something like
```python
mos_cond = 5.0
mos_uncond = 1.0
CFG_scale = 8.0
pr_eps = DDPM(text, mos_cond)
pr_eps_uncond = DDPM(text, mos_uncond)
pr_eps = pr_eps_uncond + CFG_scale*(pr_eps-pr_eps_uncond)
```

seems like decent default parameters for this theoretical inference code.
