# Homelab 学習ノート - Proxmox / Ubuntu サーバー構築 (2026-05-23)

## 今日の目標

- 使っていないPCに Proxmox をインストール
- メインPCからリモート接続できる環境を作る
- Ubuntu VM を作成
- 今後 Docker や個人サービスを運用できる基盤を作る

---

# 1. ネットワーク問題の解決

## 状況

現在のサーバーPCは、
LANケーブルを直接接続できない場所に設置されていた。

最初は USB Wi-Fi アダプタを使用して
ネット接続を試した。

しかし Proxmox は Linux ベースのサーバー環境のため、

- USB Wi-Fi ドライバ問題
- ブリッジ構成の難しさ
- 接続の不安定さ

などがあり、
正常なネットワーク接続ができなかった。

---

## 解決方法

### 中継機（ブリッジモード対応ルーター）を購入

構成:

```text
家のルーター (Wi-Fi)
   ↓
中継機 / ブリッジルーター
   ↓ LAN
サーバーPC (Proxmox)
```

これにより、
サーバーPCを有線環境として利用可能にした。

---

# 2. Proxmox UI に接続できない問題

## 問題

Proxmox インストール後、
メインPCのブラウザからアクセスできなかった。

原因:
- インストール時に設定した固定IP
- 現在のネットワーク環境

が一致していなかった。

---

## 確認に使用したコマンド

### ネットワーク状態確認

```bash
ip a
```

確認内容:
- ネットワークインターフェース
- 現在のIPアドレス
- 接続状態

---

# 3. Proxmox ネットワーク設定修正

## 作業内容

Proxmox コンソールから
ネットワーク設定ファイルを修正。

### 編集ファイル

```bash
nano /etc/network/interfaces
```

---

## 修正内容

以前の固定IP設定を削除し、
現在のネットワーク帯に合わせたIPへ変更。

例:

```text
192.168.x.x
```

---

## 設定反映

```bash
systemctl restart networking
```

または

```bash
reboot
```

---

# 4. メインPCから Proxmox UI 接続成功

## 接続URL

```text
https://サーバーIP:8006
```

例:

```text
https://192.168.x.x:8006
```

---

## 結果

- Proxmox Web UI 接続成功
- メインPCからリモート管理可能になった

---

# 5. Ubuntu VM 作成

## 作成目的

- Linux 学習
- Docker
- nginx
- Next.js サーバー運用
- SSH 練習
- ネットワーク学習

など。

---

# VM 作成時の設定

## CPU

サーバー構成:

```text
Intel i3-2100
2 Core / 4 Thread
```

設定:

```text
Socket: 1
Core: 2
CPU Type: host
```

---

## RAM

```text
4096MB (4GB)
```

設定。

---

## ディスク

初期設定:
- 32GB

後から確認した結果:
- Proxmox 全体ディスクは正常
- Ubuntu VM にだけ 32GB が割り当てられていた

学習用として一旦そのまま使用。

今後:
- 本番練習用 Ubuntu は 64GB 以上を予定。

---

# Ubuntu インストール中の問題

## 問題

インストール中、
誤って `/` のマウントを解除。

エラー:

```text
Mount a filesystem at /
```

発生。

---

## 解決方法

LVM 論理ボリューム:

```text
ubuntu-lv
```

を再度:

```text
/
```

へマウント。

最終構成:

```text
/       28GB
/boot    2GB
```

正常確認。

---

# ディスク構成確認

## 使用コマンド

```bash
lsblk
```

---

## 確認内容

```text
sda 465GB
 └ pve-data-tpool 337GB
```

つまり:
- SSD 全体は Proxmox で正常使用中
- 以前の Windows データは実質削除済み

であることを確認。

---

# 今日理解した概念

## Proxmox

仮想マシン(VM)を管理するハイパーバイザー。

1台の物理サーバー上で:

- Ubuntu
- Debian
- Windows
- Docker サーバー

などを同時に運用可能。

---

## VM (仮想マシン)

PCの中に存在する
もう1台のコンピュータ。

ISOファイルがあれば:
- サーバーを複数作成可能
- 役割分離可能
- テスト環境構築可能

---

## QEMU Agent

Proxmox ↔ Ubuntu VM 間の通信機能。

有効化すると:
- IP表示
- 正常シャットダウン
- 状態確認
- バックアップ安定化

などが可能。

Ubuntu 内でのインストール:

```bash
sudo apt update
sudo apt install qemu-guest-agent -y
sudo systemctl enable qemu-guest-agent
sudo systemctl start qemu-guest-agent
```

---

# 今後の予定

## 学習用サーバー

```text
Ubuntu-Test
32GB
```

用途:
- Linux 練習
- Docker 実験
- ネットワーク学習
- 自由な検証

---

## 運用練習用サーバー

```text
Ubuntu-Prod
64GB+
```

用途:
- Docker
- nginx
- Next.js
- 個人ホームページ
- API サーバー
- Reverse Proxy
- Cloudflare
- SSH

---

# 感想

最初は:
- ネットワーク
- 固定IP
- LVM
- VM
- ディスク構造

などがかなり複雑に感じた。

しかし、
実際に:

- 自分でインストール
- 問題解決
- コンソール操作
- 構成確認

を行うことで、
Linux サーバーやインフラの構造が
少しずつ理解でき始めた。

特に:

「自分のPCでプログラムを動かす」

ではなく、

「サービスを運用するサーバーを構築する」

という感覚を学び始めた。

今後:
- Docker
- nginx
- AWS
- DevOps
- ネットワーク

学習にも自然につながっていきそう。