# STeAMed haM-ifiER (STAMMER)

E se você reorganizasse [Steamed Hams](https://www.youtube.com/watch?v=4jXEuIHY9ic) na ordem certa, você conseguiria recriar o [FitnessGram Pacer Test](https://www.youtube.com/watch?v=Y82jDHRrswc)?

https://github.com/ArdenButterfield/stammer/assets/29317321/0f69d48b-5f99-4c8e-b9a2-8b4c83c1e19d

STAMMER é um utilitário python para recriar qualquer fonte de áudio com quadros de qualquer outra fonte de áudio ou vídeo.
Para cada quadro na fonte de áudio que está sendo recriada, o STAMMER seleciona o quadro no sou


## Instructions

STAMMER requer python 3, e as seguintes bibliotecas:

```
numpy
scipy
Pillow
```

_Psiu!_ Elas podem ser instaladas usando `pip install -r requirements.txt`.

Ele também precisa do `ffmpeg`.

STAMMER pode ser executado diretamente no prompt de comando.

```sh
python stammer.py <carrier track> <modulator track> <output file>
```

aonde `<carrier track>` é o caminho de onde o áudio/vídeo vai ter os frames removidos (i.e. Steamed Hams in the above example), `<modulator track>` é o caminho pro arquivo que será reconstruído, e `<output file>` é o caminho aonde o resultado será exportado. `<output file>` deve ter uma extensão de arquivo de vídeo ou de áudio (como `.wav`, `.mp3`, `.mp4`, etc).

## Por que e como?

For several years, making increasingly outlandish edits of the skit "Steamed Hams" from *The Simpsons* has been a [trend online](https://knowyourmeme.com/memes/steamed-hams). This project was primarily inspired by an edit of "Steamed Hams" in which [the frames were sorted by pitch](https://www.youtube.com/watch?v=iWFRKZek0FI). It really is worth a listen: the choppy untintelligible vocals, steadily rising in pitch, builds tension in a strange way. Even chopped to the granularity of a frame, the music stabs and dialogue from the skit is still recognizable.

I've messed around with audio in numpy enough that this got me thinking: what else could I do with these frames? The idea of turning "Steamed Hams" into another song by manipulating its frequencies isn't completely novel: [other videos](https://www.youtube.com/watch?v=QUb3stxzTpE) have done it using a [vocoder](https://www.youtube.com/watch?v=QUb3stxzTpE). Roughly speaking vocoder divides the carrier signal (the other song) into frequency bands, and then sets the amplitude of each band to the amplitude of the modulator signal ("Steamed Hams") at that point in time. The result is an amalgamation of the two signals, where the pitch of the carrier is "sung" through the timbre of the modulator.

Like the vocoder, STAMMER divides the carrier and modulator signal into frequency bands, and calculates the amplitude of each band. The algorithm is as follows:

Let the carrier $C$ be broken into frames $C_i, 0 \leq i < n_c$ for each of its frames, and the modulator $M$ be broken into frames $M_i, 0 \leq i < n_m$ for each of the frames. We replace each frame in $M$ with the frame of $C$ where the spectrum $c_i$ of $C_i$ is most similar to the spectrum $m_i$ of $M_i$. 

The spectrum can be calculated from the frame using the Fast Fourier Transform. This algorithm returns a spectrum with far higher resolution than we need for these purposes, especially at high frequencies. To better reflect the frequencies that the human ear is sensitive to, we average the spectra of nearby frequencies into "bins," using larger bins for higher frequencies. 

To give the algorithm more options, for a cleaner-sounding result, I decided to allow STAMMER to change the volume of carrier frames to match the volume of modulator frames. This isn't quite as involved as in a vocoder, where we change the volume of the frequencies in each bin independently. Instead, we are merely linearly scaling the spectrum vector. 

After normalizing, we want to minimize the angle between the two vectors, which maximizes their similarity. In other words, we find $c_j$ that maximizes the dot product $\frac{c_j}{|c_j|}\cdot\frac{m_i}{|m_i|}$. For future work, it may be worth looking at [Accuracy limitations and the measurement of errors in the stochastic simulation of chemically reacting systems](https://www.sciencedirect.com/science/article/pii/S0021999105003074?casa_token=-OjUKZ73eCwAAAAA:2-eLNPyv6uj7Kucqmt-ex8wKHWZQVh7A9vcFX1UKqP2QnGyovpxg4jFkxUMeAiuiuS_0be4zzQ) for more advanced ways of comparing histograms than merely averaging nearby frequency lines into bins and calculating the dot products.

