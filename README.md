--Download Dataset (62498 dutch annotated files)--  
https://mega.nz/file/QWRinJyQ#OfmBtfg3LwTdM0y9DmuFgom10ny78NPxp3Zvr3sp__o  

--PREREQUISITES--  
Make dir to write the trained networks, /media/data/networks in the example  
  
--BUILD DOCKER FILE--  
sudo docker docker build .    

--START DOCKER CONTAINER--  
sudo docker run --gpus 2 -v ~HOME/Desktop/dataset:/DeepSpeech/dataset -v /media/data/networks:/DeepSpeech/export -it 6770340f42fd /bin/bash  
  
--ENABLE GPU GROWTH--  
Run command in container  
export TF_FORCE_GPU_ALLOW_GROWTH=true  
  
--FIX DATASET--  
Run command in container  
bin/import_cv2.py --filter_alphabet data/alphabet.txt --audio_dir dataset/clips/clips dataset  
  
--START TRAINING--  
Run command in container  
dattime=`date "+%Y%m%d%H%M%S%3N"` && python3 ./DeepSpeech.py --drop_source_layers 1 alphabet_config_path /data/alphabet.txt --save_checkpoint_dir export/checkpoints/$dattime --load_checkpoint_dir export/checkpoints/$dattime --train_files dataset/clips/clips/validated.csv --dev_files dataset/clips/clips/dev.csv --test_files dataset/clips/clips/test.csv --dropout_rate=0.4 --epochs=300 --early_stop=true --export_dir export/checkpoints/$dattime/export --export_tflite=true --learning_rate=0.0001 --lm_alpha=0.9 --lm_beta=1.15 --train_batch_size=70 --export_language="nl-Latn-BE" --automatic_mixed_precision=True --beam_width=500 --train_cudnn=True --es_epochs=200 --summary_dir=export/checkpoints/$dattime/tensorboard
  
--MONITOR GPU('S)--  
Run command out of container  
watch -n1 nvidia-smi  
  
--REMOVE CHECKPOINTS DIRECTORY--  
rm -R checkpoints  
  
  
Extras:  
  
--SPLIT AUDIOBOOK BY SILENCES--
sox -V3 INPUT.wav OUTPUT.wav silence 1 0.50 0.1% 1 0.3 0.1% : newfile : restart
  
OR    
    
--GATHERING DATA FROM YOUTUBE (audio+subs)--  
downloaden as mp3 (youtubepp.com)  
srt + txt download via https://savesubs.com/  
  
-SPLIT VIDEO BY SRT TIMESTAMPS--  
bash ./split.sh foldername/*.mp3 foldername/*.srt  
  
--RENAME FILES--  
j=1;for i in *.mp3; do mv "$i" YT_YOUTUBEID"$j".mp3; let j=j+1;done  

--CONVERT TO MONO--  
for f in *.mp3; do ffmpeg -i "$f" -c:a libmp3lame -q:a 2 -ac 1 mono-"$f"; done  
  
--EXPORT FILE LIST--  
ls -1v > lijst.txt  
