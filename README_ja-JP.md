<div align="center">
  <h1 align="center"> avp_teleoperate </h1>
  <h3 align="center"> Unitree Robotics </h3>
  <p align="center">
    <a href="README.md"> English </a> | <a href="README_zh-CN.md">中文</a> | <a>日本語</a>
  </p>
</div>

# 📺 ビデオデモ

<p align="center">
  <a href="https://www.youtube.com/watch?v=OTWHXTu09wE" target="_blank">
    <img src="https://img.youtube.com/vi/OTWHXTu09wE/maxresdefault.jpg" alt="Watch the video" style="width: 35%;">
  </a>
</p>




# 0. 📖 イントロダクション
このリポジトリは、**Apple Vision Pro** を使用して **Unitree ヒューマノイドロボット** を遠隔操作するためのものです。

サポートされるロボットは以下の通りです。

<table>
  <tr>
    <th style="text-align: center;"> &#129302; ロボット </th>
    <th style="text-align: center;"> &#9898; ステータス </th>
    <th style="text-align: center;"> &#128221; 備考 </th>
  </tr>
  <tr>
    <td style="text-align: center;"> G1 (29DoF) + Dex3-1 </td>
    <td style="text-align: center;"> &#9989; 完了 </td>
    <td style="text-align: center;">  </td>
  </tr>
  <tr>
    <td style="text-align: center;"> H1 (アーム 4DoF) </td>
    <td style="text-align: center;"> &#9201; 進行中 </td>
    <td style="text-align: center;"> <a href="https://github.com/unitreerobotics/avp_teleoperate/tree/h1" target="_blank">このブランチのikを一時的に参照</a> </td>
  </tr>
  <tr>
    <td style="text-align: center;"> H1_2 (アーム 7DoF) + Inspire </td>
    <td style="text-align: center;"> &#9989; 完了 </td>
    <td style="text-align: center;"> <a href="https://github.com/unitreerobotics/avp_teleoperate/tree/h1_2" target="_blank">このブランチを参照</a> </td>
  </tr>
  <tr>
    <td style="text-align: center;"> ··· </td>
    <td style="text-align: center;"> ··· </td>
    <td style="text-align: center;"> ··· </td>
  </tr>
</table>


必要なデバイスと配線図は以下の通りです。

<p align="center">
  <a href="https://oss-global-cdn.unitree.com/static/e0ca680eda164e31bd0ff6f8fb50604c_5050x2590.png">
    <img src="https://oss-global-cdn.unitree.com/static/e0ca680eda164e31bd0ff6f8fb50604c_5050x2590.png" alt="Watch the Document" style="width: 65%;">
  </a>
</p>



# 1. 📦 前提条件

私たちは Ubuntu 20.04 と Ubuntu 22.04 でコードをテストしました。他のオペレーティングシステムでは異なる設定が必要かもしれません。

詳細については、[公式ドキュメント](https://support.unitree.com/home/zh/Teleoperation) および [OpenTeleVision](https://github.com/OpenTeleVision/TeleVision) を参照してください。

## 1.1 🦾 逆運動学

```bash
unitree@Host:~$ conda create -n tv python=3.8
unitree@Host:~$ conda activate tv
# `pip install` を使用する場合、pinocchio のバージョンが 3.1.0 であることを確認してください
(tv) unitree@Host:~$ conda install pinocchio -c conda-forge
(tv) unitree@Host:~$ pip install meshcat
(tv) unitree@Host:~$ pip install casadi
```

> p.s. コマンドの前にあるすべての識別子は、**どのデバイスとディレクトリでコマンドを実行するべきか**を示すためのものです。
>
Ubuntu システムの `~/.bashrc` ファイルでは、デフォルトの設定は次の通りです: `PS1='${debian_chroot:+($debian_chroot)}\u@\h:\w\$ '`
>
> 例として、`(tv) unitree@Host:~$ pip install meshcat` コマンドを取り上げます。
>
> - `(tv)` はシェルが conda 環境 `tv` にあることを示します。
>- `unitree@Host:~` はユーザー `\u` `unitree` がデバイス `\h` `Host` にログインしており、現在の作業ディレクトリ `\w` が `$HOME` であることを示します。
> - `$` は現在のシェルが Bash であることを示します（非ルートユーザーの場合）。
> - `pip install meshcat` は `unitree` が `Host` で実行したいコマンドです。
> 
> 詳細については、[Harley Hahn's Guide to Unix and Linux](https://www.harley.com/unix-book/book/chapters/04.html#H) および [Conda User Guide](https://docs.conda.io/projects/conda/en/latest/user-guide/getting-started.html) を参照してください。

## 1.2 🕹️ unitree_sdk2_python

```bash
# unitree_sdk2_python をインストールします。
(tv) unitree@Host:~$ git clone https://github.com/unitreerobotics/unitree_sdk2_python.git
(tv) unitree@Host:~$ cd unitree_sdk2_python
(tv) unitree@Host:~$ pip install -e .
```



# 2. ⚙️ TeleVision と Apple Vision Pro の設定

## 2.1 📥 基本

```bash
(tv) unitree@Host:~$ cd ~
(tv) unitree@Host:~$ git clone https://github.com/unitreerobotics/avp_teleoperate.git 
(tv) unitree@Host:~$ cd ~/avp_teleoperate
(tv) unitree@Host:~$ pip install -r requirements.txt
```

## 2.2 🔌 ローカルストリーミング

**Apple** は非 HTTPS 接続での WebXR を許可していません。アプリケーションをローカルでテストするには、自己署名証明書を作成し、クライアントにインストールする必要があります。Ubuntu マシンとルーターが必要です。Apple Vision Pro と Ubuntu **ホストマシン** を同じルーターに接続します。

1. mkcert をインストールします: https://github.com/FiloSottile/mkcert
2. **ホストマシン** のローカル IP アドレスを確認します:

```bash
(tv) unitree@Host:~/avp_teleoperate$ ifconfig | grep inet
```

**ホストマシン** のローカル IP アドレスが `192.168.123.2` であると仮定します。

> p.s. `ifconfig` コマンドを使用して **ホストマシン** の IP アドレスを確認できます。

3. 証明書を作成します:

```bash
(tv) unitree@Host:~/avp_teleoperate$ mkcert -install && mkcert -cert-file cert.pem -key-file key.pem 192.168.123.2 localhost 127.0.0.1
```

生成された `cert.pem` と `key.pem` ファイルを `teleop` に配置します。

```bash
(tv) unitree@Host:~/avp_teleoperate$ cp cert.pem key.pem ~/avp_teleoperate/teleop/
```

4. サーバーでファイアウォールを開きます:

```bash
(tv) unitree@Host:~/avp_teleoperate$ sudo ufw allow 8012
```

5. Apple Vision Pro に ca-certificates をインストールします:

```bash
(tv) unitree@Host:~/avp_teleoperate$ mkcert -CAROOT
```

`rootCA.pem` を AirDrop 経由で Apple Vision Pro にコピーし、インストールします。

設定 > 一般 > 情報 > 証明書信頼設定。「ルート証明書の完全な信頼を有効にする」の下で、証明書の信頼をオンにします。

設定 > アプリ > Safari > 高度な設定 > 機能フラグ > WebXR 関連機能を有効にします。

## 2.3 🔎 テスト環境

このステップは、環境が正しくインストールされているかを確認するためのものです。

1. Isaac Gym をダウンロードします: https://developer.nvidia.com/isaac-gym/download

    現在のディレクトリに解凍し、`IsaacGym_Preview_4_Package/isaacgym/python` ディレクトリに移動して、次のコマンドを実行します:

    ```bash
    (tv) unitree@Host:~/IsaacGym_Preview_4_Package/isaacgym/python$ pip install -e .
    ```

2. 上記の手順に従ってローカルでストリーミングを設定した後、Isaac Gym で 2 つのロボットハンドを遠隔操作してみてください:

    ```bash
    (tv) unitree@Host:~/avp_teleoperate$ cd teleop
    (tv) unitree@Host:~/avp_teleoperate/teleop$ python teleop_test_gym.py
    ```

3. Apple Vision Pro デバイスを装着します。

4. Apple Vision Pro で Safari を開き、次の URL にアクセスします: https://192.168.123.2:8012?ws=wss://192.168.123.2:8012

    > p.s. この IP アドレスは **ホストマシン** の IP アドレスと一致する必要があります。

5. `Enter VR` をクリックし、`Allow` を選択して VR セッションを開始します。

6. 3D で手を確認します！





# 3. 🚀 使用方法

このプログラムを開始する前に、[公式ドキュメント](https://support.unitree.com/home/zh/Teleoperation) を少なくとも一度は読んでください。


## 3.1 🖼️ 画像サーバー

`avp_teleoperate/teleop/image_server` ディレクトリにある `image_server.py` を Unitree ロボット (G1/H1/H1_2 など) の **開発用コンピューティングユニット PC2** にコピーし、**PC2** で次のコマンドを実行します:

```bash
# p.s.1 scp コマンドを使用して image_server.py を PC2 に転送し、ssh を使用して PC2 にリモートログインして実行できます。
# p.s.2 画像転送プログラムは現在、双眼 RGB カメラ用に設定されています。

# 現在、Unitree ロボット PC2 端末にいます
unitree@PC2:~/image_server$ python image_server.py
# 端末に次のように出力されます:
# Image server has started, waiting for client connections...
# Image Resolution: width is 640, height is 480
```

画像サービスが開始された後、**ホスト** 端末で `image_client.py` を使用して通信が成功したかどうかをテストできます:

```bash
(tv) unitree@Host:~/avp_teleoperate/teleop/image_server$ python image_client.py
```

## 3.2 ✋ Inspire ハンドサーバー（オプション）

> 注意: 選択したロボット構成に Inspire デクスタラスハンドが使用されていない場合、このセクションは無視してください。

関連する環境を設定し、制御プログラムをコンパイルするには、[デクスタラスハンド開発](https://support.unitree.com/home/zh/H1_developer/Dexterous_hand) を参照できます。まず、[このリンク](https://oss-global-cdn.unitree.com/static/0a8335f7498548d28412c31ea047d4be.zip) を使用してデクスタラスハンド制御インターフェースプログラムをダウンロードし、Unitree ロボットの **PC2** にコピーします。

Unitree ロボットの **PC2** で次のコマンドを実行します:

```bash
unitree@PC2:~$ sudo apt install libboost-all-dev libspdlog-dev
# プロジェクトをビルドします
unitree@PC2:~$ cd h1_inspire_service & mkdir build & cd build
unitree@PC2:~/h1_inspire_service/build$ cmake .. -DCMAKE_BUILD_TYPE=Release
unitree@PC2:~/h1_inspire_service/build$ make
# ターミナル 1. h1 inspire ハンドサービスを実行します
unitree@PC2:~/h1_inspire_service/build$ sudo ./inspire_hand -s /dev/ttyUSB0
# ターミナル 2. サンプルを実行します
unitree@PC2:~/h1_inspire_service/build$ ./h1_hand_example
```

2 つの手が連続して開閉する場合、成功を示します。成功したら、ターミナル 2 で `./h1_hand_example` プログラムを閉じます。

## 3.3 🚀 スタート

> ![Warning](https://img.shields.io/badge/Warning-Important-red) 
>
> 1. すべての人は、潜在的な危険を防ぐためにロボットから安全な距離を保つ必要があります！
>
> 2. このプログラムを実行する前に、[公式ドキュメント](https://support.unitree.com/home/zh/Teleoperation) を少なくとも一度は読んでください。
>
> 3. 常にロボットが [デバッグモード (L2+R2)](https://support.unitree.com/home/zh/H1_developer/Remote_control) に入っていることを確認し、モーションコントロールプログラムを停止して、潜在的なコマンドの競合問題を回避します。
>

このプログラムを実行するには、**オペレーター A** と **オペレーター B** と呼ばれる 2 人のオペレーターがいるのが最適です。

今、**オペレーター B** は **ホストマシン** で次のコマンドを実行します:

```bash
(tv) unitree@Host:~/avp_teleoperate/teleop$ python teleop_hand_and_arm.py --record
```

そして、**オペレーター A** は

1. Apple Vision Pro デバイスを装着します。

2. Apple Vision Pro で Safari を開き、次の URL にアクセスします: https://192.168.123.2:8012?ws=wss://192.168.123.2:8012

   > p.s. この IP アドレスは **ホストマシン** の IP アドレスと一致する必要があります。

3. `Enter VR` をクリックし、`Allow` を選択して VR セッションを開始します。

ホスト端末が "Please enter the start signal (enter 'r' to start the subsequent program):" と出力したら、**オペレーター B** は端末で **r** キーを押して遠隔操作プログラムを開始できます。

この時点で、**オペレーター A** はロボットのアームとデクスタラスハンドを遠隔操作できます。

次に、**オペレーター B** は開いている "record image" ウィンドウで **s** キーを押してデータの記録を開始し、再度 **s** キーを押して停止できます。必要に応じてこれを繰り返すことができます。

> p.s. 記録されたデータはデフォルトで `avp_teleoperate/teleop/data` に保存されます。使用方法については、このリポジトリを参照してください: [unitree_IL_lerobot](https://github.com/unitreerobotics/unitree_IL_lerobot/tree/main?tab=readme-ov-file#data-collection-and-conversion)。

## 3.4 🔚 終了

プログラムを終了するには、**オペレーター B** は 'record image' ウィンドウで **q** キーを押すことができます。

>  ![Warning](https://img.shields.io/badge/Warning-Important-red) 
>
> ロボットを損傷しないようにするために、**オペレーター A** がロボットのアームを自然に下げた位置または適切な位置に配置した後、**オペレーター B** が **q** を押して終了するのが最適です。



# 4. 🗺️ コードベースチュートリアル

```
avp_teleoperate/
│
├── assets                    [ロボット URDF 関連ファイルの保存]
│
├── teleop
│   ├── image_server
│   │     ├── image_client.py [ロボット画像サーバーから画像データを受信するために使用]
│   │     ├── image_server.py [カメラから画像をキャプチャし、ネットワーク経由で送信（ロボットのオンボードコンピュータで実行）]
│   │
│   ├── open_television
│   │      ├── television.py    [Apple Vision Pro から Vuer を使用して手首と手のデータをキャプチャ]  
│   │      ├── tv_wrapper.py    [キャプチャされたデータの後処理]
│   │
│   ├── robot_control
│   │      ├── robot_arm_ik.py        [アームの逆運動学]  
│   │      ├── robot_arm.py           [デュアルアームジョイントを制御し、他の部分をロック]
│   │      ├── robot_hand_inspire.py  [Inspire ハンドジョイントを制御]
│   │      ├── robot_hand_unitree.py  [Unitree ハンドジョイントを制御]
│   │
│   ├── utils
│   │      ├── episode_writer.py          [模倣学習のデータを記録するために使用]  
│   │      ├── mat_tool.py                [いくつかの小さな数学ツール]
│   │      ├── weighted_moving_filter.py  [ジョイントデータをフィルタリングするため]
│   │
│   │──teleop_hand_and_arm.py   [遠隔操作の起動実行コード]
|   |——teleop_test_gym.py       [環境が正しくインストールされているかを確認するために使用できます]
```



# 5. 🛠️ ハードウェア

## 5.1 📋 リスト

|             アイテム             | 数量 |                             リンク                             |                           備考                           |
| :--------------------------: | :------: | :----------------------------------------------------------: | :---------------------------------------------------------: |
|     **Unitree ロボット G1**     |    1     |                  https://www.unitree.com/g1                  |               開発用コンピューティングユニット付き               |
|     **Apple Vision Pro**     |    1     |           https://www.apple.com/apple-vision-pro/            |                                                             |
|          **ルーター**          |    1     |                                                              |                                                             |
|         **ユーザー PC**          |    1     |                                                              | 推奨グラフィックカード性能は RTX 4080 以上 |
|    **ヘッドステレオカメラ**    |    1     | [参考のみ] http://e.tb.cn/h.TaZxgkpfWkNCakg?tk=KKz03Kyu04u |                          ヘッド用                           |
|    **ヘッドカメラマウント**     |    1     | https://github.com/unitreerobotics/avp_teleoperate/blob/g1/hardware/head_stereo_camera_mount.STEP |          ヘッドステレオカメラの取り付け用, FOV 130°          |
|     Intel RealSense D405     |    2     |      https://www.intelrealsense.com/depth-camera-d405/       |                          リスト用                          |
|       リストリングマウント       |    2     | https://github.com/unitreerobotics/avp_teleoperate/blob/g1/hardware/wrist_ring_mount.STEP |                リストカメラマウントと一緒に使用                 |
|   左リストカメラマウント    |    1     | https://github.com/unitreerobotics/avp_teleoperate/blob/g1/hardware/left_wrist_D405_camera_mount.STEP |       左リスト RealSense D405 カメラの取り付け用        |
|   右リストカメラマウント   |    1     | https://github.com/unitreerobotics/avp_teleoperate/blob/g1/hardware/right_wrist_D405_camera_mount.STEP |       右リスト RealSense D405 カメラの取り付け用       |
|         M3 六角ナット          |    4     |         [参考のみ] https://a.co/d/1opqtOr          |                     リストファスナー用                      |
|         M3x12 ネジ         |    4     |       [参考のみ] https://amzn.asia/d/aU9NHSf       |                     リストファスナー用                      |
|         M3x6 ネジ          |    4     |       [参考のみ] https://amzn.asia/d/0nEz5dJ       |                     リストファスナー用                      |
|       **M4x14 ネジ**       |    2     |       [参考のみ] https://amzn.asia/d/cfta55x       |                      ヘッドファスナー用                      |
| **M2x4 自タッピングネジ** |    4     |       [参考のみ] https://amzn.asia/d/1msRa5B       |                      ヘッドファスナー用                      |

> 注意: 太字のアイテムは遠隔操作タスクに必要な機器であり、他のアイテムは [データセット](https://huggingface.co/unitreerobotics) を記録するためのオプション機器です。

## 5.2 🔨 インストール図

<table>
    <tr>
        <th align="center">アイテム</th>
        <th align="center" colspan="2">シミュレーション</th>
        <th align="center" colspan="2">実物</th>
    </tr>
    <tr>
        <td align="center">ヘッド</td>
        <td align="center">
            <p align="center">
                <img src="./img/head_camera_mount.png" alt="head" width="100%">
                <figcaption>ヘッドマウント</figcaption>
            </p>
        </td>
        <td align="center">
            <p align="center">
                <img src="./img/head_camera_mount_install.png" alt="head" width="80%">
                <figcaption>組み立ての側面図</figcaption>
            </p>
        </td>
        <td align="center" colspan="2">
            <p align="center">
                <img src="./img/real_head.jpg" alt="head" width="20%">
                <figcaption>組み立ての正面図</figcaption>
            </p>
        </td>
    </tr>
    <tr>
        <td align="center">リスト</td>
        <td align="center" colspan="2">
            <p align="center">
                <img src="./img/wrist_and_ring_mount.png" alt="wrist" width="100%">
                <figcaption>リストリングとカメラマウント</figcaption>
            </p>
        </td>
        <td align="center">
            <p align="center">
                <img src="./img/real_left_hand.jpg" alt="wrist" width="50%">
                <figcaption>左手の組み立て</figcaption>
            </p>
        </td>
        <td align="center">
            <p align="center">
                <img src="./img/real_right_hand.jpg" alt="wrist" width="50%">
                <figcaption>右手の組み立て</figcaption>
            </p>
        </td>
    </tr>
</table>

> 注意: リストリングマウントは、画像の赤い円で示されているように、ロボットのリストのシームと一致する必要があります。



# 6. 🙏 謝辞

このコードは、以下のオープンソースコードベースに基づいて構築されています。各ライセンスを確認するには、以下の URL を訪れてください。

1) https://github.com/OpenTeleVision/TeleVision
2) https://github.com/dexsuite/dex-retargeting
3) https://github.com/vuer-ai/vuer
4) https://github.com/stack-of-tasks/pinocchio
5) https://github.com/casadi/casadi
6) https://github.com/meshcat-dev/meshcat-python
7) https://github.com/zeromq/pyzmq
8) https://github.com/unitreerobotics/unitree_dds_wrapper
9) https://github.com/tonyzhaozh/act
10) https://github.com/facebookresearch/detr
11) https://github.com/Dingry/BunnyVisionPro