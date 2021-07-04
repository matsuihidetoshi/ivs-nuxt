# Amazon Interactive Video Service (IVS) と AWS Amplify を使って自分だけのオリジナル配信サイトを作る！
こんにちは！ [**株式会社スタートアップテクノロジー**](https://startup-technology.com/) 所属、  
そして先日、[**AWS Serverless HERO**](https://aws.amazon.com/developer/community/heroes/hidetoshi-matsui/)に選ばれました、[**JAWS-UG** 浜松支部](https://jawsug-hamamatsu.doorkeeper.jp/)の松井です！  

今回は **AWS** の各種マネージドサービスを活用した配信サイトの構築方法のお話をさせていただきます！  

オンライン配信しようとした場合、 **YouTube Live** などを活用する方法が考えられますね。  
しかし、既存サービスを活用する場合カスタマイズが難しく、何かと不自由もあるかと思います。  

そこで今回は、 **AWS** で提供されている **Amazon Interactive Video Service (IVS)** と **AWS Amplify** 、  
また **StreamYard** と言うライブストリームサービスを活用して、自分だけのオリジナル配信サイトを構築する方法をご紹介します！

## 必要要件

- 下記サービスの有効なアカウント
  - **AWS**
  - **GitHub**
- **StreamYard** の **Basic** プラン（本資料内で登録・設定方法をご案内します）以上のアカウント
- 数十円程度の **AWS** 利用料および **StreamYard** の最低 $20/month の利用料

## アーキテクチャ
- 今回のハンズオンで構築するアーキテクチャです。

![ivs-nuxt (1)](https://user-images.githubusercontent.com/38583473/122328796-3bffc400-cf6b-11eb-9d0b-bd146d2801aa.png)

- **Amplify Console** を使い、 **Nuxt.js** で作成した静的ページをデプロイ・ホスティングします。
- **StreamYard** という配信データを送信するWebサービスを使って、 **Amazon IVS** に対してPC上から動画データを送信します。
- Webブラウザ側から、 **amazon-ivs-player** や、 **Video.js** といったライブラリを利用して、 **Amazon IVS** が提供する再生URLからストリーミングデータを取得して表示します。

***

- また、参考として、[**JAWS DAYS 2021 re:Connect**](https://jawsdays2021.jaws-ug.jp/) の配信サイトのアーキテクチャも紹介します。

![JAWS-DAYS-2021-STREAMING-V2 (5)](https://user-images.githubusercontent.com/38583473/121799787-fe4c2400-cc68-11eb-869f-4708aca5a4c3.png)

- **Amazon EventBridge** をトリガーにして　**AWS Lambda** を実行し、 **Amazon IVS** の配信視聴者数を取得し、 **Amazon DynamoDB** に保存する機能を実装しました。
- また、 **Amazon IVS** の **Timed Metadata** という機能を利用して、上記の視聴者数も含めたリアルタイムなデータのクライアントへの反映や、アンケート機能などを実装しました。
- **AWS Amplify** を利用して、配信に付随する各種データの管理ができる仕組みも構築しました。


***

## 目次

1. **IVS** の設定
2. **StreamYard** の登録・設定
3. **Nuxt.js** プロジェクトのセットアップ
4. **Amplify Console** でデプロイ

## 1. **IVS** の設定

### **AWS マネジメントコンソール** にログイン
- [**AWS マネジメントコンソール**](https://console.aws.amazon.com/)にアクセスします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 21 55 41" src="https://user-images.githubusercontent.com/38583473/119227442-db868e00-bb48-11eb-9d74-a43195f3dcf0.png">

- メールアドレスを入力して、**次へ**をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 22 03 06" src="https://user-images.githubusercontent.com/38583473/119227950-6ff1f000-bb4b-11eb-9163-0534c30c4fc0.png">

- セキュリティチェックを入力して、**送信**をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 22 18 01" src="https://user-images.githubusercontent.com/38583473/119228043-032b2580-bb4c-11eb-8de0-ff720c362ef1.png">

- パスワードを入力して、**サインイン**をクリックします。

***

### IVS チャネルを作成

<img width="887" alt="スクリーンショット 2021-05-22 22 30 10" src="https://user-images.githubusercontent.com/38583473/119228605-b137cf00-bb4e-11eb-879c-e062fac00f6c.png">

- **ivs** と入力して、 **Amazon Interactive Video Service** をクリックします。

***

<img width="887" alt="スクリーンショット 2021-05-22 22 44 26" src="https://user-images.githubusercontent.com/38583473/119228776-88640980-bb4f-11eb-8624-219168e7a167.png">

- **リージョンを 米国西部（オレゴン）に変更** をクリックします。

***

<img width="1125" alt="スクリーンショット 2021-05-22 22 51 13" src="https://user-images.githubusercontent.com/38583473/119228927-58693600-bb50-11eb-9cf3-3658f473391a.png">

- **チャネルの作成**をクリックします。

***

<img width="491" alt="スクリーンショット 2021-05-22 23 00 16" src="https://user-images.githubusercontent.com/38583473/119229203-c6fac380-bb51-11eb-9814-492a884a0f63.png">

- **チャネル名**に**ivs-nuxt-1**と入力し、**チャネルの作成**をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 23 07 58" src="https://user-images.githubusercontent.com/38583473/119230716-69b64080-bb58-11eb-9eef-6e9d72495625.png">

- 作成した **ivs-nuxt-1** 詳細ページの、**ストリーム設定**の3点をコピーして控えます。
  - **取り込みサーバー**
  - **ストリームキー**
  - **再生URL**

***

## 2. **StreamYard** の登録・設定

### **StreamYard** に登録
- [**StreamYard のトップページ**](https://streamyard.com/) へアクセスします。

***

<img width="1438" alt="streamyard_email" src="https://user-images.githubusercontent.com/38583473/119226345-8b58fd00-bb43-11eb-96a4-7f14efd390cc.png">

- 上記にメールアドレスを入力して、 **Get started** をクリックします。

***

<img width="866" alt="streamyard_confirmation" src="https://user-images.githubusercontent.com/38583473/119226490-631dce00-bb44-11eb-8169-06a62f75555f.png">

- メールが送信されてくるので、6桁の確認コードをコピーします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 21 30 25" src="https://user-images.githubusercontent.com/38583473/119227518-36b88080-bb49-11eb-971d-e3840e4ff8a4.png">

- 確認コードを入力して、 **Log in** をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 21 33 12" src="https://user-images.githubusercontent.com/38583473/119226708-79785980-bb45-11eb-913c-577782ca94dc.png">

- **Onward!** をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 23 25 33" src="https://user-images.githubusercontent.com/38583473/119229947-21495380-bb55-11eb-8aca-48bf162d1263.png">

- **Upgrade** をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 23 28 08" src="https://user-images.githubusercontent.com/38583473/119230032-79805580-bb55-11eb-8589-24543bfa9b3b.png">

- **Basic** の **Upgrade now** をクリックします。

***

<img width="575" alt="スクリーンショット 2021-05-22 23 30 15" src="https://user-images.githubusercontent.com/38583473/119230154-00353280-bb56-11eb-879d-3a091bf4ec88.png">

- カード情報を入力し、 **monthly** / **annually** を任意で選択し、 **Purchase plan** をクリックします **（注意: 料金が発生します）**。

***

<img width="575" alt="スクリーンショット 2021-05-22 23 36 16" src="https://user-images.githubusercontent.com/38583473/119230353-c31d7000-bb56-11eb-9929-f7183d4b27a8.png">

- **Return to dashboard** をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 23 19 08" src="https://user-images.githubusercontent.com/38583473/119229743-4e493680-bb54-11eb-9aa2-442eb08d7cc2.png">

- **Destinations** をクリックしてから、 **Add a destination** をクリックします。

***

<img width="1198" alt="スクリーンショット 2021-05-22 23 23 02" src="https://user-images.githubusercontent.com/38583473/119229876-d29bb980-bb54-11eb-896b-7cea9e84fa4d.png">

- **Custom RTMP** をクリック

***

<img width="1197" alt="スクリーンショット 2021-05-22 23 51 50" src="https://user-images.githubusercontent.com/38583473/119230963-843ce980-bb59-11eb-9071-2b8c3c4b680a.png">

- 先ほど **1. **IVS** の設定** で最後に控えた下記2つを入力します。
  - **取り込みサーバー**
  - **ストリームキー**
- **Nickname** は **ivs-nuxt-1** と入力し、 **Add RTMP server** をクリックします。

***

## 3. **Nuxt.js** プロジェクトのセットアップ

- 今回ページを公開するための **Nuxt.js** のフロントエンドプロジェクトをセットアップします。
- ハンズオンの中では具体的にコードの変更はしませんが、これ以降はこちらの環境で自由にカスタムしていただけます！

### **AWS Cloud9** 環境の構築

- [**こちら**](https://aws.amazon.com/jp/builders-flash/202008/amplify-crud-app#002)を参考に、 **AWS Cloud9** 環境を構築します。
  -  **Cloud9 の環境が整いました。** の箇所まで進めます。
  -  **注意!!** リージョンとインスタンスタイプの組み合わせによっては、うまく環境が立ち上がらない場合があります。こちらの手順はリージョンやインスタンスタイプに依存しないため、うまくいかない場合はリージョンやインスタンスタイプを変更していただいて構いません。

### **Nuxt.js** プロジェクトのフォーク、クローン、セットアップ(これ以降は **Cloud9** のターミナルで操作してください)

- [**こちら**](https://github.com/matsuihidetoshi/ivs-nuxt)のリポジトリにアクセスします。

***

<img width="1437" alt="スクリーンショット 2021-06-12 23 36 44" src="https://user-images.githubusercontent.com/38583473/121780208-42441800-cbda-11eb-8cc2-25e871d87495.png">

- **Fork** をクリックします。

***

- **GitHub** からコードをクローンして、プロジェクトディレクトリに移動します。
- ※これ以降、**プロジェクトリポジトリに移動しないとうまくコマンドが実行できません**のでお気をつけください。

```
git clone https://github.com/[自分のGitHubアカウントのキー]/ivs-nuxt.git
cd ivs-nuxt
```

***

- **npm** パッケージをインストールします。

```
npm i
```

***

- テスト起動します。

```
npm run dev
```

***

<img width="1440" alt="スクリーンショット 2021-07-04 15 05 30" src="https://user-images.githubusercontent.com/38583473/124374886-bce3ed00-dcd9-11eb-98e5-b816a1264d98.png">

- **Preview** をクリックしてから、 **Preview Running Application** をクリックします。

***

<img width="1439" alt="スクリーンショット 2021-06-12 22 28 07" src="https://user-images.githubusercontent.com/38583473/121777874-369f2400-cbcf-11eb-8f9a-5f731805abba.png">

- デモ用のストリーミング動画が表示されるのを確認します。

***

### **StreamYard** で配信開始

<img width="810" alt="スクリーンショット 2021-06-12 22 48 35" src="https://user-images.githubusercontent.com/38583473/121778122-787c9a00-cbd0-11eb-83f6-80be67b6b320.png">

- **Create a broadcast** をクリックします。

***

<img width="810" alt="スクリーンショット 2021-06-12 22 56 15" src="https://user-images.githubusercontent.com/38583473/121778400-b29a6b80-cbd1-11eb-9241-4f63d9077bf5.png">

- アイコンをクリックし（マウスオーバーで、 `ivs-nuxt-1` と表示されます）、任意のタイトルを入力して、 **Create broadcast** をクリックします。

***

<img width="549" alt="スクリーンショット 2021-06-12 23 00 46" src="https://user-images.githubusercontent.com/38583473/121778558-613eac00-cbd2-11eb-9708-c0c8a98ddb97.png">

- **Enter studio** をクリックします。

***

<img width="1437" alt="スクリーンショット 2021-06-12 23 06 06" src="https://user-images.githubusercontent.com/38583473/121778802-9d264100-cbd3-11eb-9e1d-e94c72236616.png">

- 左下の顔の表示にマウスオーバーすると、 **Add to stream** と表示されるので、クリックします。

***

<img width="1437" alt="スクリーンショット 2021-06-12 23 53 31" src="https://user-images.githubusercontent.com/38583473/121780103-cea20b00-cbd9-11eb-8727-57db6c61d6b2.png">

- **Go live** をクリックし、ダイアログからさらに **Go live** をクリックします。

***

### **Nuxt.js** プロジェクトに自分の **再生URL** を設定

- 一度、起動している開発環境のサーバーを停止します(ターミナルで `ctrl + c`)。

```
ctrl + c
```

***

- ターミナルで下記を実行し、[こちら](https://github.com/matsuihidetoshi/ivs-nuxt/blob/main/README.md#ivs-%E3%83%81%E3%83%A3%E3%83%8D%E3%83%AB%E3%82%92%E4%BD%9C%E6%88%90)で控えた自分の **再生URL** を環境変数に設定します。

```
export IVS_NUXT_STREAM_URL="自分の再生URL"
```

***

- サーバーを再起動します。

```
npm run dev
```

***

- **Preview** しているページを再読み込みして、自分の配信が再生されることを確認します。

***

## 4. **Amplify Console** でデプロイ
- ここまでローカルでプロジェクトを進めてきましたが、いよいよ配信サイトを公開します！
- **AWS Amplify Console** から簡単なステップで公開できますので、その手順を解説します。

***

<img width="1437" alt="スクリーンショット 2021-06-13 0 18 08" src="https://user-images.githubusercontent.com/38583473/121829300-18cfdd00-ccfd-11eb-9809-907575039917.png">

- **AWS マネジメントコンソール**で検索フォームに`amplify`と入力します。
- **AWS Amplify** が表示されたら、クリックします。

***

<img width="1437" alt="スクリーンショット 2021-06-13 0 21 00" src="https://user-images.githubusercontent.com/38583473/121829356-39983280-ccfd-11eb-9c67-776b262e3cd3.png">

- **GET STARTED** をクリックします。

***

<img width="1437" alt="スクリーンショット 2021-06-13 0 21 59" src="https://user-images.githubusercontent.com/38583473/121829380-4ae13f00-ccfd-11eb-9ec6-becaa7463baa.png">

- **Get started** の下の右側のセクションの **Get started** ボタンをクリックします。

***

<img width="1093" alt="スクリーンショット 2021-06-13 0 23 27" src="https://user-images.githubusercontent.com/38583473/121829415-6b10fe00-ccfd-11eb-8439-51044c50c1b0.png">

- **GitHub** を選択し、 **Continue** をクリックします。

***

<img width="588" alt="スクリーンショット 2021-06-13 0 27 25" src="https://user-images.githubusercontent.com/38583473/121829474-985dac00-ccfd-11eb-9350-35712844845e.png">

- **Authorize aws-amplify-console** をクリックします。

***

<img width="799" alt="スクリーンショット 2021-06-13 0 28 40" src="https://user-images.githubusercontent.com/38583473/121829842-9cd69480-ccfe-11eb-9007-41e171884fd2.png">

- **リポジトリ**からForkしたご自身の **ivs-nuxt** リポジトリを選択します。
- **main** ブランチを選択して、 **次へ** をクリックします。

***

![スクリーンショット 2021-06-14 15 48 52](https://user-images.githubusercontent.com/38583473/121999549-69703480-cde8-11eb-9c24-d5eb48d69aff.png)

- **ビルド設定の追加**の **Edit** をクリックして、後述の `amplify.yml` 通り変更してください。
- **Advanced settings** をクリックします。
- **環境変数**の **Key** に `IVS_NUXT_STREAM_URL` を、 **Value** に先ほど[こちら](https://github.com/matsuihidetoshi/ivs-nuxt#ivs-%E3%83%81%E3%83%A3%E3%83%8D%E3%83%AB%E3%82%92%E4%BD%9C%E6%88%90)で控えた **再生URL** を入力します。
- 次に、**次へ**をクリックしてください。

**amplify.yml**
```
version: 1
frontend:
  phases:
    preBuild:
      commands:
        - npm install
    build:
      commands:
        - npm run build
        - npm run generate
  artifacts:
    # IMPORTANT - Please verify your build output directory
    baseDirectory: dist
    files:
      - '**/*'
  cache:
    paths:
      - node_modules/**/*
```

***

<img width="792" alt="スクリーンショット 2021-06-13 19 08 23" src="https://user-images.githubusercontent.com/38583473/121830059-24bc9e80-ccff-11eb-9e63-c902ff769dd9.png">

- **保存してデプロイ**をクリックします。

***

<img width="1122" alt="スクリーンショット 2021-06-13 19 12 38" src="https://user-images.githubusercontent.com/38583473/121830087-369e4180-ccff-11eb-91e4-7f7140f51a23.png">

- プログレスバーのステータスが**検証**になるのを待ちます。
- その後、**ドメイン**の箇所に記載されているURLをクリックしてください。
- 配信ページが **Cloud9** で実行したプレビューページ同様表示されればOKです！

***

## まとめ
いかがでしたでしょうか？  
こちらの配信サイトは **JavaScript(Vue.js/Nuxt.js)** ベースでできておりますので、  
もちろんこのリポジトリからカスタムしていただいて独自の機能を追加したり、思い思いのデザインにしていただくことが可能です！  
例えば、カスタマイズの方向性としては下記などが考えられます。

- **Amazon IVS** の **Timed Metadata** を利用して、バックエンドとのインタラクションを活用した機能を構築する。
- **Amazon IVS** の録画データの保存機能を有効化して、 **Amazon S3** に配信動画を保存する。

ぜひ、色々とカスタムして遊んでみてください！！  
**Happy coding!**

