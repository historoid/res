# 機械学習による残存歯の自動認識

## AWSにあるBlenderで画像生成する方法

1. AWSにDockerをインストールする
1. このリポジトリにあるDockerfileでイメージをビルドし、コンテナを作成する
1. ホスト（AWSのインスタンス）にS3からBlender関連ファイルをダウンロードする
1. ホストとコンテナの共有フォルダに先ほどのBlender関連ファイルを格納する
1. コンテナ内に入ってBlenderスクリプトを実行する

### AWSにDockerをインストールする方法

DockerがインストールしやすいAmazon Linux2のインスタンスを作成する。  
GPUではなく、CPUメインで画像生成するので、c5.2xlargeを使う（別のインスタンスでもOK）。

インスタンス作成後、SSHでインスタンスに入る。  
インスタンスに入ったら以下のコマンドを実行。  
ちなみにAmazon LinuxはCentOS系である。

`sudo yum update -y`  
`sudo yum install docker`  
`sudo service docker start`  
`sudo usermod -a -G docker ec2-user`

一旦ここでログアウトして、再度ログインする。
再ログイン後、以下のコマンドで動作確認

`docker info`  

`sudo`じゃなくてもdockerコマンド動けばOK

なお、インスタンスを再起動時にDocker daemonが動かない場合にも`sudo service docker start`を入力すると良い。

### Dockerfileからイメージをビルドする

短いので手打ちでOK。  
このリポジトリにあるDockerfileの内容を参照。  
Dockerfileの内容は以下のとおり。

`FROM ubuntu:20.10`  
`RUN apt-get -y update && apt-get install -y vim blender`  
`RUN mkdir work`  
`CMD ["/bin/bash"]`  

上記のDockerfileを作成後、ビルドする。

`docker build -t historoid/blender:latest .`

`-t`でイメージ名を指定している。  
最後のピリオドがDockerfileの場所を指定している（上記の例では現在のパスを指定）。
成功すれば`Successfully built <image-ID>`と表示される。  
イメージIDはコピーしておく。

`docker images`でちゃんとできているか確認する。  
問題なければコンテナを作成するが、ホストとの共有ファイルを作りたいのでその準備を行う。

`mkdir work`

共有フォルダ名は何でも良いが今回は`work`とした。  
Dockerコンテナ内部のフォルダも`work`になっている。

`pwd`や`ls`で現在のパスを確認する。  
おそらく`/home/ec2-user/work`が共有したいフォルダになっているはず。

コンテナの作成して起動する。  
`docker run ----name blender -v /home/ec2-user/work:/work -it <image-ID> /bin/bash`  

これでコンテナが起動し、bashの入力状態になっているはず。  
`exit`でコンテナから出て、`docker ps -a`で確認する。  
コンテナ名がblenderになっているものがExited状態ならOK。 

再度コンテナを起動する場合には、`docker start blender`で起動して、`docker exec -it blender bash`する(`/bin/bash`でも可）。

### 踏み台サーバー

`ssh -i <xxxx-1.pem> -oProxyCommand='ssh -i <xxxx-2.pem> -W %h:%p ec2-user@<踏み台インスタンスのIP>' ec2-user@<プライベートインスタンスのIP>`


## Blenderに関するメモ

結論として、AWSにBlenderをインストールすることはできなかった。  
過去にインストールすることができていたが、同じ方法ではできなくなっている。  

以下に参考として様々なウェブサイトからの引用としてインストール方法を載せておくが、どれも成功していない。

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

