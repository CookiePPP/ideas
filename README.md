# ideas

---

🟥🟧🟨🟩🟦🟪🟫⬛⬜🔲🔳⏹☑✅❎

---

## 🟥 PnG BERT++

![image](https://user-images.githubusercontent.com/42448678/218235075-a1731746-bcb5-4d1f-83ce-a18ebe144539.png)

In the PnG BERT paper they note that 

> the benefit of PnG BERT primarily lies in better natural language understanding, rather than potential improvements on G2P conversion.

For an emotive TTS downstream task, I imagine the understanding of the text is far more important than the G2P element.

So I propose some changes that may or may not improve the model.

1. Use both word-level and character-level embeddings.

The paper already finds a significant benefit from adding word-level position encoding. I imagine adding word-level value embeddings is just as important for understanding the text.

[Text-writing diffusion](https://github.com/nostalgebraist/improved-diffusion) and [Parti](https://parti.research.google/) both show that the task of WORD-ID to letters is extremely challenging, with [Parti](https://parti.research.google/) needing 20 BILLION parameters for this emergent ability and [Text-writing diffusion](https://github.com/nostalgebraist/improved-diffusion) learning the ability to write using far less parameters once a character-level encoder is added to the architecture.

Also [Mixed-Phoneme BERT](https://arxiv.org/pdf/2203.17190.pdf) who found a large benefit from using phoneme AND subphoneme value embeddings. (Note - MPBERT doesn't use subphone-level position encoding. another thing to test.)

2. Additional teachers.

PnG BERT doesn't use ground truth phoneme data, it actually uses a pretrained text -> phoneme model. Because of this, the model is effectively just a student of a G2P model that also has to learn MLM objective.

The paper shows that despite not being ground truth data, it still improves downstream performance, so I propose adding an emotion classification objective using the latents of pretrained model [DeepMoji](https://github.com/bfelbo/DeepMoji) as ground truth 'emotion' for each sample, and training PnG BERT to also predict this latent.

3. More varied dataset

PNG BERT uses Wikipedia as their pretraining dataset which makes perfect sense given their model needs to see a large variety of vocab in order to learn from it's G2P teacher. I propose using the [twemoji](https://uvaauas.figshare.com/articles/dataset/Twemoji_Dataset/5822100) dataset as additional training data since it contains a variety of emotion which will help PnG BERT learn from the proposed DeepMoji teacher.
