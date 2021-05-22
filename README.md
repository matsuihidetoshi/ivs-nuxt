# Amazon Interactive Video Service (IVS) と AWS Amplify を使って自分だけのオリジナル配信サイトを作る！
こんにちは！ 株式会社スタートアップテクノロジー所属、 **JAWS-UG** 浜松支部の松井です！  
オンライン配信しようとした場合、 **YouTube Live** などを活用する方法が考えられますね。  
しかし、既存サービスを活用する場合カスタマイズが難しく、何かと不自由もあるかと思います。  
そこで今回は、 **AWS** で提供されている **Amazon Interactive Video Service (IVS)** と **AWS Amplify** 、  
また **StreamYard** と言うライブストリームサービスを活用して、自分だけのオリジナル配信サイトを構築する方法をご紹介します！

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

- パスワードを入力して、**サインイン**をクリック

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

## Build Setup

```bash
# install dependencies
$ yarn install

# serve with hot reload at localhost:3000
$ yarn dev

# build for production and launch server
$ yarn build
$ yarn start

# generate static project
$ yarn generate
```

## Installed packages(Requied if you want to build a project from scratch)

```bash
$ yarn add copy-webpack-plugin@6 amazon-ivs-player video.js
```

For detailed explanation on how things work, check out [Nuxt.js docs](https://nuxtjs.org).
