# 機械学習による残存歯の自動認識

## Blenderに関して

### Blenderのバージョン

Ubuntuのバージョンによって、インストールされるBlenderのバージョンが変わる。  
Ubuntu 18系だと、Blender 2.8系がインストールされる。  

### インスタンスタイプとBlenderインストールの検証

以下のコマンドはただのメモ。  
実際にはDockerをインストールしてからUbuntuのインストール方法でOK.

#### Ubuntu  

`sudo apt-get update`  
`sudo apt-get install blender`  
もしくは
`sudo apt update`
`sudo apt -yV upgrade`
`sudo apt -yV install blender blender-data`  

Docker では2つ目のコマンドでインストールできた。

`sudo apt update`  
`sudo apt install snapd`
`sudo snap install blender --classic`  
これでどうでしょう？  
これもダメっぽい。

#### Fedora系  

`dnf install blender`  

#### Amazon Linux系  
`sudo yum -y install freetype freetype-devel libpng-devel`  
`sudo yum -y install mesa-libGLU-devel`  
`sudo yum -y install libX11-devel mesa-libGL-devel perl-Time-HiRes`  
`tar -jxvf blender*.tar.bz2 #Use the *`  
`sudo yum -y update`  
`sudo yum -y install libXi`  
`sudo yum -y install libXrender`  

| AMI | インスタンスタイプ | Blenderインストールの可否 |
| :-: | :-: | :-: |
| Amazon Linux2 | t2.micro | CLIでダメ。tarもダメ。 |
| Amazon Linux | t2.micro | x CLI, tar |
| Ubuntu Server 20.04 | t2.micro | x |
| Ubuntu Server 18.04 | t2.micro | x |
| Ubuntu DL 20.04 | t2.micro | x |
| Ubuntu DL 18.04 | t2.micro | x |

どれもダメだった。  
昔どうやってインストールできたのかも分からない。  
今はDockerで代用する。

### スクリプトの実行

`blender -b <blender-file-path> -P <python-file-path> &`

## AWSに関して

### Ubuntuインスタンス

Ubuntuの16か18のGPUインスタンスを使っていたが、毎回Blenderのインストールの時点でうまく行かないので、Dockerを使うことにした。  
DockerのインストールもUbuntuだと手順通りに行かないので、Amazon Linuxを使うことにした。

### Amazon Linux

Dockerもインストールしやすいので、Amazon Linuxを使うことにした。  
Amazon Linux2もあるが、Amazon Linuxをとりあえず使ってみている。

インスタンスの作成はいつもどおり。

インスタンスにログインしたら以下のコマンドでDockerをまずはインストールする。

`sudo yum update -y`  
`sudo yum install docker`  
`sudo service docker start`  
`sudo usermod -a -G docker ec2-user`
一旦ここでログアウトして、再度ログインする。

ログイン後、以下のコマンドで確認  
`docker info`  
`sudo`じゃなくてもdockerコマンド動けばOK

## S3との連携

`aws s3 cp s3://xxxxx/xxxx/xxxx ./`  
ディレクトリごとコピーしたかったら、末尾に`--recursive`を追加。

## Docker

Ubuntu:latestを使用している。  
2020年9月時点で、18系が入る。

### DockerのUbuntuにBlenderをインストールする

`sudo apt-get update`  
`sudo apt-get -y upgrade`  
`sudo apt update`  
`sudo apt -yV upgrade`  
`sudo apt -yV install blender blender-data`  

これでBlenderがインストールされるはず。

### python3のインストール

`python --version`  
`sudo apt update`  
`sudo apt install software-properties-common`  
`sudo add-apt-repository ppa:deadsnakes/ppa`  
`sudo apt update`  
`sudo apt install python3.xxx`

### pip3のインストール

`sudo apt install python3-pip`

### Numpy等のライブラリ

`pip3 install numpy`

