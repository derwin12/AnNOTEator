NOTES

Test for GPU
python -c "import tensorflow as tf; print(tf.config.list_physical_devices('GPU'))"
python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"

Installed automatic v9.x so that I could copy in this older version
Had to install cudann v8.9
Update path to include the v8 files.

Example:
python main.py -l https://youtu.be/g-QKTLiQPPk -km performance -bpm 100 -on demo 

export TF_GPU_ALLOCATOR=cuda_malloc_async

Settings for Music21:
%appdata%\music21-settings.xml

python -m pip install torchaudio==2.2.2+cu118 torchvision==0.17.2+cu118 torch==2.2.2+cu118 -f https://download.pytorch.org/whl/torch_stable.html

c:\Users\daryl\anaconda3\condabin\conda.bad install -p c:/users/daryl/anaconda3/envs/AnNOTEator -c conda-forge cudatoolkit=11.2 cudnn=8.1.0
