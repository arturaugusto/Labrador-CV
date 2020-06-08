# Labrador-CV

Visão Computacional na placa Labrador da Caninos Loucos para aplicações de IOT

## Rede

Alterar o arquivo /etc/networking/interfaces seguindo o modelo:

```
auto eth0
allow-hotplug eth0 
iface eth0 inet static
address 192.168.0.1
netmask 255.255.255.0

allow-hotplug wlan0
#iface wlan0 inet dhcp
iface wlan0 inet static 
address 192.168.0.35
netmask 255.255.255.0
gateway 192.168.1.1
wpa-ssid XXXXXX
wpa-psk XXXXXX
```

reiniciar a rede

```
sudo systemctl restart networking
sudo ifup wlan0 && sudo ifup wlan0
```

## Servidor ssh

Para instalar servidor ssh:

`sudo apt-get install openssh-server`

Agora você poderá acessar sua placa pela rede através do comando `ssh -l caninos@192.168.1.35` ou utilizando aplicativos como o PuTTY para Windows.

## fstab (opcional)

Costumo criar disco de memória para algumas aplicações que geram arquivos
temporários com frequência (exemplo: Tesseract-OCR). Para isso, edite o `fstab` (aqui estou usando o vi):

`sudo vi /etc/fstab`

E crie esta linha:

`tmpfs       /mnt/ramdisk tmpfs   nodev,nosuid,noexec,nodiratime,size=100M   0 0`

Reiniciar a placa para testar:

`sudo /sbin/reboot`

## Ambiente virtual Python 

Considero uma boa prática criar um ambiente virtual em aplicações Python. Para isto, instale os pacotes necessários:

`sudo apt-get update`

`sudo apt-get install python3-venv`

cria ambiente chamado `ocr`

`python3 -m venv ocr`

ativa ambiente:

`source ocr/bin/activate`

## Instalar pacotes 

Caso não consiga instalar normalmente via `pip`, disponibilizo alguns pacotes para aplicações científicas pré-compilados neste link: (https://drive.google.com/file/d/1eS0X1hZbXu6bvRmzvcHNtqj9WFGovqmI/view?usp=sharing)

Você pode copiar os pacotes wheels para bibliotecas científicas via ssh a partir do seu computador:

`scp wheels.tar.gz caninos@192.168.1.35:/home/caninos`


Faça isto *dentro* do ambiente virtual Python (`source ocr/bin/activate`)

```
tar xzvf wheels.tar.gz
cd wheels

pip3 install numpy-1.18.4-cp37-cp37m-linux_armv7l.whl 
pip3 install scipy-1.4.1-cp37-cp37m-linux_armv7l.whl
pip3 install Pillow-7.1.2-cp37-cp37m-linux_armv7l.whl
pip3 install imutils-0.5.3-py3-none-any.whl
pip3 install pytesseract-0.3.4-py2.py3-none-any.whl 
pip3 install PyYAML-5.3.1-cp37-cp37m-linux_armv7l.whl
pip3 install absl_py-0.9.0-py3-none-any.whl
pip3 install termcolor-1.1.0-py3-none-any.whl 
pip3 install wrapt-1.12.1-cp37-cp37m-linux_armv7l.whl
```

Se você não vai usar o `OpenOffice`, pode remover os pacotes para ganhar aproximadamente 1GB de espaço:

```
sudo apt-get purge libreoffice*
sudo apt-get clean
sudo apt-get autoremove
```

## OpenCV

Obs: Não sei se é realmente necessário, mas para compilar instalei os seguintes dependências:

```
sudo apt-get install build-essential cmake unzip pkg-config
sudo apt-get install libjpeg-dev libpng-dev libtiff-dev
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install libgtk-3-dev
sudo apt-get install libcanberra-gtk*
sudo apt-get install libatlas-base-dev gfortran
sudo apt-get install python3-dev
```

Pode obter uma versão que compilei aqui: (https://drive.google.com/file/d/1QFJI8_QBn3Gkn7xHZR6tX_BL9kK93Z_e/view?usp=sharing)

novamente, você pode copiar via ssh
`scp opencv.tar.gz caninos@192.168.1.35:/home/caninos`


Na placa Labrador descompacta arquivo

```
cd ~
tar -zxvf opencv.tar.gz
cd ~/opencv/build/python_loader
```

e instala o OpenCV

`python setup.py install`

cria link para que a instalação funcione no Python

```
cd ~/ocr/lib/python3.7/site-packages
ln -s /home/caninos/opencv/build/lib/python3/cv2.cpython-37m-arm-linux-gnueabihf.so cv2
```

Para testar se o módulo está sendo importado corretamente execute o seguinte comando:

`python3 -c 'import cv2;print(cv2.__version__)'`

deve mostrar na tela `4.2.0`, que é a versão instalada do OpenCV que foi compilada.

## Tesseract OCR

Obtenha versão compilada aqui: (https://drive.google.com/file/d/1uAEoHj22BM4N2l2hNHmI_aaHNBMvzDTr/view?usp=sharing)

copia tesseract 5 alpha via ssh
`scp tesseract.tar.gz caninos@192.168.1.35:/home/caninos/`

no labrador, descompacta arquivo e cria um link para o binário

```
cd ~
tar -zxvf tesseract.tar.gz
mkdir ~/local
mkdir ~/local/bin
ln -s ~/tesseract/tesseract ~/local/bin/tesseract
```

Pode ser necessário instalar algumas dependências. Para isto, basta instalar a versão disponível:

`sudo apt-get install tesseract-ocr`



## Referências:

https://www.pyimagesearch.com/2018/09/26/install-opencv-4-on-your-raspberry-pi/
