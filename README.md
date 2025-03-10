*this repo is not official*

# Environment
* Windows 11
* python 3.10 (conda)
* pytorch 2.4.1 + cuda12.1
* [whisper](https://huggingface.co/openai/whisper-large-v3)
* [MeloTTS](https://github.com/myshell-ai/MeloTTS)

# Install
1. using conda, create a python 3.10 venv: `conda create -n winmelotts python=3.10`
2. activate winmelotts: `conda activate winmelotts	`
3. clone this repo:

```bash
git clone https://github.com/harryzhu/melotts-train-win.git
cd melotts-train-win
pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu121
pip install -r requirements.txt

# 3rd/unidic/unidic.zip.download.url.txt
# https://cotonoha-dic.s3-ap-northeast-1.amazonaws.com/unidic-3.1.0.zip
python ./02_unidic_fix_download.py

# 3rd/nltk_data/nltk_data.zip.download.url.txt
# https://github.com/nltk/nltk_data/tree/gh-pages
python ./03_nltk_data_fix.py
cd MeloTTS
pip install -e .
```

4. prepare your dataset, i.e.: put your wav files in: `D:/dataset/_test/source`
5. open `melotts-train-win/melosteps.py`, update the following vars as yours:

```python
DATASET_DIR_ROOT='D:/dataset/_test'
#
TRAIN_DATA_DIR='D:/svc/_train/MeloTTS/melo/data'
TRAIN_DATA_WAV_DIR=f'{TRAIN_DATA_DIR}/_wav'
TRAIN_DATA_TXT_DIR=f'{TRAIN_DATA_DIR}/_txt'
TRAIN_DATA_CFG_DIR=f'{TRAIN_DATA_DIR}/harry'
```

5. run the following steps:

```bash
# step 0: rename the files, format: foldername_num.wav
python melosteps.py --step=0

# step 1: split long wav file into 10-second segments
python melosteps.py --step=1

#  step 2: split the silence of the 10-second wav file
python melosteps.py --step=2

# step 3: normalize the wav files: 44.1k, pcm_s16le, 1
python melosteps.py --step=3

# step 4: go to your whisper env, run this step only, wav to txt
python melosteps.py --step=4

# step 5: delete wav files which cannot be transribled by whisper
python melosteps.py --step=5

# step 6: generate metadata.list
python melosteps.py --step=6

# step 7: generate *.bert.pt
# python preprocess_text.py --metadata D:/dataset/_test/metadata.list 
python melosteps.py --step=7

# step 8: delete wav files which cannot generate .bert.pt by preprocess_text.py
python melosteps.py --step=8

# step 9: copy valid wav files into train data_dir: melotts-train-win/MeloTTS/melo/data/[_wav, _txt]
python melosteps.py --step=9

# step 10: re-generate metadata.list, start train, results will be in melotts-train-win/MeloTTS/melo/logs
# python preprocess_text.py --metadata data/harry/metadata.list 
python melosteps.py --step=10
```
Notice: `step 7` and `step 10`, preprocess the wav and txt files, but in different folder:

```bash
python preprocess_text.py --metadata D:/dataset/_test/metadata.list 
```

6. train:

```bash
python ./train.py --config D:/svc/_train/MeloTTS/melo/data/harry/config.json --model harry
```

7. use your trained model:

```python
from melo.api import TTS

device = "cuda:0"
speed = 1.0

model = TTS(language='ZH', device=device, use_hf=False, 
    config_path='D:/svc/_train/MeloTTS/melo/logs/harry/config.json', 
    ckpt_path='D:/svc/_train/MeloTTS/melo/logs/harry/G_37000.pth')
speaker_ids = model.hps.data.spk2id
print(speaker_ids)


def txt2tts(fpath=""):
    fwav = fpath.replace(".txt",".wav")
    with open(fpath, 'r', encoding="utf-8") as f:
        words = f.read()
        print(words)
        model.tts_to_file(words, speaker_ids['harry'], fwav, speed=speed)

txt2tts("D:/test.txt")
```

8. check loss in browser:
```bash
# run tensorboard, then open the url: http://localhost:6006/
#
tensorboard --logdir=D:/svc/_train/MeloTTS/melo/logs/harry
```

<hr/>

** MeloTTS is owned by the following citation **

## Citation

```
@software{zhao2024melo,
  author={Zhao, Wenliang and Yu, Xumin and Qin, Zengyi},
  title = {MeloTTS: High-quality Multi-lingual Multi-accent Text-to-Speech},
  url = {https://github.com/myshell-ai/MeloTTS},
  year = {2023}
}
```

