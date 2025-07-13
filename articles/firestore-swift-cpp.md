---
title: "[Firestore] Firestoreを使ってSwiftとC++でリアルタイム同期した話"
emoji: "📝"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift", "Cpp", "Firestore"]
published: false
---

# はじめに
こんにちは！
大学の授業で、1か月間でC言語とSDLを使ってゲームをつくるというものがあるのですが、そこで普通につくってもおもしろくないな...「せやSwift使ってiOSも使ってやろう！」と思い立ちました。

PC側はC++を使い、iOS側はSwiftを使って、そのデータの同期はPC側で適当にNodeでWebsocketのプログラムを書いてやり取りをしようと考えていました。

ですが、そこで少し問題が起きてしまいました。

プログラムは大学のPCで実行できるようにしなければいけないのですが、なんと私のiPhoneが大学のWi-Fiに接続できないということが判明してしまい、Websocketでするなら外部サーバが必要になるということがわかってしまいました。
さすがに...授業でそこまでするのは大掛かりすぎると考え、代替案を考えていました。

そこで、**FirebaseのFirestore**というサービスがあり、それが使えそうということがわかったのでそれを使おうということになりました。

ですが、初めて使うものだったのでいくつかつまづいてしまう箇所があり、やり方を含めメモしておこうと思います。

# 開発環境
PC: **Ubuntu 20.04**(サポート終わってるというね...)
iOS: **iPhone 15(iOS 18.5)**
gcc: **9.3.0**
Xcode: **16.4**

# 作ったもの
まずは、先にどんなものを開発しているかというのを紹介しておこうと思います。

![](https://storage.googleapis.com/zenn-user-upload/3c0a8a594a7d-20250712.gif)
![](https://storage.googleapis.com/zenn-user-upload/b100804ef425-20250712.gif)

こんな感じで、iOSのほうで玉を動かせばPC側で玉が発射されるというものです。

# Firestoreとは？
Firebaseのサービスの１つで、クラウド上のデータベースのことです。そして、この一番の特徴は、面倒くさいサーバの構築などをせずに**複数のアプリ内でデータをリアルタイムで共有することができる**ことです。

今回の開発では、この点をうまく使えると考えたためこれを採用することにしました。

# 準備
それでは、まずは開発するにあたって必要な準備をしていきます。

## Swift側の環境構築
めっちゃ簡単！
基本的にチュートリアルに従えば大丈夫！

[Firebase Console](https://console.firebase.google.com/)にアクセスして、「Firebaseプロジェクトを作成する」をクリックする。

![](https://storage.googleapis.com/zenn-user-upload/7accfffb53b7-20250712.png)

で、そのまま画面に書いている通りに進めていきプロジェクトの作成を完了する。

次にプラットフォームを選択しなければいけない。
今回はiOS用なので、**「iOS」** を選択する。

![](https://storage.googleapis.com/zenn-user-upload/ee2a3fb68a82-20250712.png)

次に色々入力する画面が出てくる。

![](https://storage.googleapis.com/zenn-user-upload/ec9d1d3f9293-20250712.png)

まずは、**「Apple バンドルID」** を入力する。
これはXcodeの.xcodeprojファイルのTargetを選択して、そこの「General」に書いてある。

![](https://storage.googleapis.com/zenn-user-upload/36f28474ed70-20250712.png)

それをそのまま打ち込む。

次の画面では、設定ファイルをDLする。
        **「GoogleService-Info.plistのダウンロード」** というボタンがあるのでそれをクリックしてファイルをダウンロードする。

これを、プロジェクトの直下に置く(つまり.swiftがあるところ)。

![](https://storage.googleapis.com/zenn-user-upload/7362abe2ca35-20250712.png)

次にFirebaseのFirestoreのパッケージを入れる。
プロジェクトナビゲーター上で右クリックをして、下の画像のように **「Add Package Dependencies」** をクリックする。
![](https://storage.googleapis.com/zenn-user-upload/2f96ffc742f7-20250713.png)

出てきたウィンドウで`https://github.com/firebase/firebase-ios-sdk`で検索する。
そして、「Add Package」をクリックする。
次に出てきたウィンドウでパッケージの中のどのライブラリをインストールするか選べるので、その画面で **「Firebase Firestore」** の「Add to Target」で自分のプロジェクトのターゲットを選択する。

これで基本的な慣行構築は完了しました。
あとは、このFirestoreを使うために初期化用のコードをコピーして実装しておきましょう。

```swift
import SwiftUI
import FirebaseCore


class AppDelegate: NSObject, UIApplicationDelegate {
  func application(_ application: UIApplication,
                   didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
    FirebaseApp.configure()

    return true
  }
}

@main
struct YourApp: App {
  // register app delegate for Firebase setup
  @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate


  var body: some Scene {
    WindowGroup {
      NavigationView {
        ContentView()
      }
    }
  }
}
```
これを実装します。

## C++側の環境構築
**Firestore C++ SDK**というものがあり、それをダウンロードしてくる。
[Firebase for C++](https://firebase.google.com/docs/cpp/setup?hl=ja&_gl=1*13srvj7*_up*MQ..*_ga*MzYzNzgwNTY2LjE3NTIzNjU5MDY.*_ga_CW55HF8NVT*czE3NTIzNjU5MDUkbzEkZzAkdDE3NTIzNjU5MDUkajYwJGwwJGgw&platform=ios)でダウンロードしてくる。
圧縮ファイルなので、それを解凍して、プロジェクトのどこか適当な場所に置いておく。

そして、`CMakeLists.txt`を書きましょう。
```
# Firebase SDKで必要なライブラリを検索
find_package(PkgConfig REQUIRED)
pkg_check_modules(LIBSECRET REQUIRED libsecret-1)
pkg_check_modules(GLIB REQUIRED glib-2.0)

# ------------------------------------------------------------------
# Firebase C++ SDKの手動設定
# ------------------------------------------------------------------
set(FIREBASE_CPP_SDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/[firebase-cpp-sdkのパス]/firebase_cpp_sdk")

message(STATUS "Firebase C++ SDK Path: ${FIREBASE_CPP_SDK_DIR}")

# Firebase SDKが存在するかチェック
if(EXISTS "${FIREBASE_CPP_SDK_DIR}/include" AND EXISTS "${FIREBASE_CPP_SDK_DIR}/libs")
    set(FIREBASE_AVAILABLE TRUE)
    message(STATUS "Firebase C++ SDK found")
    
    # Firebaseのヘッダーファイルが含まれるディレクトリを設定
    set(FIREBASE_INCLUDE_DIR "${FIREBASE_CPP_SDK_DIR}/include")
    
    # Firebaseのライブラリファイルが含まれるディレクトリを設定（Linux/x86_64向け）
    set(FIREBASE_LIB_DIR "${FIREBASE_CPP_SDK_DIR}/libs/linux/x86_64/cxx11")
    
    # 使用するFirebase静的ライブラリのパスを個別に指定
    set(FIREBASE_APP_LIBRARY "${FIREBASE_LIB_DIR}/libfirebase_app.a")
    set(FIREBASE_AUTH_LIBRARY "${FIREBASE_LIB_DIR}/libfirebase_auth.a")
    set(FIREBASE_FIRESTORE_LIBRARY "${FIREBASE_LIB_DIR}/libfirebase_firestore.a")
    
else()
    set(FIREBASE_AVAILABLE FALSE)
    message(WARNING "Firebase C++ SDK not found at ${FIREBASE_CPP_SDK_DIR}")
endif()

# ------------------------------------------------------------------
# メイン実行ファイルへのFirebaseライブラリのリンク
# ------------------------------------------------------------------
if(TARGET ${PROJECT_NAME})
    # Firebase C++ SDKをリンク（利用可能な場合）
    if(FIREBASE_AVAILABLE)
        # ヘッダーのパスを追加
        target_include_directories(${PROJECT_NAME} PRIVATE ${FIREBASE_INCLUDE_DIR})
        
        # ライブラリをリンク
        target_link_libraries(${PROJECT_NAME} PRIVATE
            ${FIREBASE_FIRESTORE_LIBRARY}
            ${FIREBASE_AUTH_LIBRARY}
            ${FIREBASE_APP_LIBRARY}
            Threads::Threads          # pthread
            dl                        # dlopen/dlsym関数用
            ssl crypto                # OpenSSL libraries
            ${LIBSECRET_LIBRARIES}    # libsecret
            ${GLIB_LIBRARIES}         # glib
        )
        
        # 依存ライブラリのインクルードパスとコンパイラフラグも追加
        target_include_directories(${PROJECT_NAME} PRIVATE ${LIBSECRET_INCLUDE_DIRS} ${GLIB_INCLUDE_DIRS})
        target_compile_options(${PROJECT_NAME} PRIVATE ${LIBSECRET_CFLAGS_OTHER} ${GLIB_CFLAGS_OTHER})
    endif()
endif()

# ------------------------------------------------------------------
# プロジェクト内ライブラリへのFirebaseライブラリのリンク (もしあれば)
# ------------------------------------------------------------------
if(TARGET ${PROJECT_NAME}_lib)
    # Firebase C++ SDKをライブラリにもリンク（利用可能な場合）
    if(FIREBASE_AVAILABLE)
        # ヘッダーのパスをPUBLICで追加し、このライブラリを使う側にも伝播させる
        target_include_directories(${PROJECT_NAME}_lib PUBLIC ${FIREBASE_INCLUDE_DIR})
        
        # ライブラリをリンク
        target_link_libraries(${PROJECT_NAME}_lib PRIVATE
            ${FIREBASE_FIRESTORE_LIBRARY}
            ${FIREBASE_AUTH_LIBRARY}
            ${FIREBASE_APP_LIBRARY}
            Threads::Threads
            dl
            ssl crypto
            ${LIBSECRET_LIBRARIES}
            ${GLIB_LIBRARIES}
        )
        
        # 依存ライブラリのインクルードパスとコンパイラフラグも追加
        target_include_directories(${PROJECT_NAME}_lib PUBLIC ${LIBSECRET_INCLUDE_DIRS} ${GLIB_INCLUDE_DIRS})
        target_compile_options(${PROJECT_NAME}_lib PRIVATE ${LIBSECRET_CFLAGS_OTHER} ${GLIB_CFLAGS_OTHER})
    endif()
endif()
```
基本的な環境構築はこれで終了です。

## Firestoreの設定
次に、Firestoreの設定をしておきましょう。

以下の画像のように「構築」 -> 「Firestore Database」をクリックしてください。
![](https://storage.googleapis.com/zenn-user-upload/8079f1785faf-20250713.png)

そして「データベースの作成」をクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/e8caa1fb89fb-20250713.png)

「次へ」をクリック

![](https://storage.googleapis.com/zenn-user-upload/f3d546ff4e2b-20250713.png)

「作成」をクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/9b6860abc8b7-20250713.png)

そして、次に認証のルールを少し書き換えます。私がしていたとき、なぜか認証がうまくできず、データを追加することができませんでした。本来はこれはしてはいけないのですが、しぶしぶ設定しておきましょう。

**⚠️ 注意: 本番環境では絶対に使用しないでください**

Firestoreの画面の「ルール」をクリックしてください。

![](https://storage.googleapis.com/zenn-user-upload/10a0e3ca1eb7-20250713.png)

そして、出てきた画面で
```
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if false;
    }
  }
}
```
このように記述されていると思うんですが、

```diff
rules_version = '2';

service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
-      allow read, write: if false;
+      allow read, write: if true;
    }
  }
}
```

に書き換えておいてください。
どうやったらこれを変えずにできるのか誰か教えてほしいです...

# 実装
それでは、次に実装していきましょう。
ここで注意なのですが、ここで紹介するソースコードは全て**私が今回の開発で使ったFirestoreの機能だけ**です。
ですので、私のプログラム用に実装したコードをそのまま流用していますのでそれはご了承ください。

それではやっていきましょう！

## Swift側の実装
このプログラムでは、Swift側はデータを投げるだけなのでめっちゃ簡単です！

### TODO
- [ ] データのエンティティの構造体をつくる
- [ ] Firestoreの初期化をする
- [ ] データを送信する。

### データのエンティティの構造体をつくる
送信するデータの構造体をつくります。
この構造体は、`Codable`と`Identifiable`に準拠しておけば大丈夫です。

私のコードは
```swift
struct BallEntity: Codable, Identifiable {
    @DocumentID var id: String?
    var velocity_x: Float
    var velocity_y: Float
    @ServerTimestamp var created_at: Timestamp?
    var expires_at: Timestamp?
}
```
こんな感じです。
メンバ変数の名前はそのままDBのキーの名前になります。
`Timestamp`はFirestoreで使える時間を扱う型です。
このデータ(ドキュメント)をつくった時間(`created_at`)と、データの期限(`expires_at`)を格納しています。また、`@ServerTimestamp`をつけておくとサーバに書き込んだときの時刻が自動で入るようになっています。

ちなみに、このTimestampはSwiftの`Date`型から型変換することができるので
```swift
ballEntity.expires_at = Timestamp(date: Date().addingTimeInterval(5))
```
こうすると現在の時間から**5秒後**の時間を格納することができます。

今回はサーバに書き込むことしか紹介していませんが、もちろん取得することもできます。

###  Firestoreの初期化をする
```swift
let firestore = Firestore.firestore()
```
これで大丈夫です。

###  データを送信する。
```swift
do {
    try firestore.collection("Ball").addDocument(from: ballEntity)
    print("DBへの追加が成功しました。")
} catch {
    print("DBへの追加が失敗しました。エラー: \(error.localizedDescription)")
}
```
こんな感じで大丈夫です。
これで、`Ball`という名前のコレクションにデータを入れることができます。

これでiOS側の実装は完了です。

## C++側の実装
こちら側では結構面倒くさいです。
C++ではSwiftのようにJSONに対応していないので、そこらへんのデコードやエンコードの処理を自分で記述する必要があります。
そして、私が今回書いたソースコードも完璧ではない可能性が高いです。

一応、方針としては、こちら側はデータを取得するので、何かデータが送信されたらデータを取得して、それを扱いやすいように構造体にデコードするというものです。

### TODO
- [ ] Firebaseのアプリの設定
- [ ] FirebaseとFirestoreの初期化
- [ ] データを取得するためのクエリーの作成
- [ ] クエリーでデータを取得する
- [ ] データを構造体にデコードする

### Firebaseのアプリの設定
C++からは先ほどiOS用に作成したFirebase上のアプリに接続する必要があります。
ですので、アクセスするために以下のデータを設定しなければいけないです。

- APIキー
- アプリID
- プロジェクトID

下の画像のように、それぞれ

- `API_KEY`
- `GOOGLE_APP_ID`
- `PROJECT_ID`
を見れば書いてありますのでそれを参考にしてください。

![](https://storage.googleapis.com/zenn-user-upload/6b474d0b6336-20250713.png)

それでは、これらの情報を元に実装していきましょう。
```cpp
firebase::AppOptions firebaseConfigs;
firebaseConfigs.set_api_key("APIキー");
firebaseConfigs.set_app_id("アプリID");
firebaseConfigs.set_project_id("プロジェクトID")
```
ちなみに、これらの情報はあまり外に出してはいけない情報ですので、環境変数にしておいてそれを読み込むとかしておいたほうがいいと思います(iOSの方も)。

###  FirebaseとFirestoreの初期化
```cpp
// Firestore 初期化
firebase::App* app = firebase::App::Create(firebaseConfigs);
 if (!app) {
    std::cerr << "Firebaseアプリの初期化が失敗しました。" << std::endl;
    return false;
}

// Firestore 初期化
auto* firestore = firebase::firestore::Firestore::GetInstance(app);
if (!firestore) {
    std::cerr << "Firestoreのインスタンスの生成に失敗しました。" << std::endl;
    delete app;
    return -1;
}
```
こんな感じで大丈夫です。
Firebaseの方は先ほどの情報を使って初期化し、それに成功すればFirestoreを初期化します。

###  データを取得するためのクエリーの作成
```cpp
 firebase::firestore::Query query = firestore->Collection("Ball")
        .Where(firebase::firestore::Filter::GreaterThan("expires_at", firebase::firestore::FieldValue::Timestamp(firebase::Timestamp::Now())));
```
こんな感じです。ちょっと複雑なので、１つ１つ分解していきましょう。

```cpp
 firebase::firestore::Query query = firestore->Collection("Ball")
```
ここでクエリーを作成しています。`Ball`という名前のコレクションのデータを取得してくるという意味になります。

```cpp
.Where(firebase::firestore::Filter::GreaterThan("expires_at", firebase::firestore::FieldValue::Timestamp(firebase::Timestamp::Now())));
```
これで、`expired_at`のデータが現在の時刻(`firebase::Timestamp::Now()`) よりも大きいデータを取得するという意味になります。つまりフィルタですね。
ここでは書き方などあまり深い話はしないでおきますが、結構いろいろな設定ができるみたいなので、もし興味があったら調べてみてください。

###  クエリーでデータを取得する
DBにデータが追加されたら呼び出されるラムダ関数を定義しておきます。

```cpp
listenerRegistration = query.AddSnapshotListener(
    [](const firebase::firestore::QuerySnapshot& snapshot, firebase::firestore::Error error, const std::string& message)
    {
        if (error == firebase::firestore::kErrorOk) {
            // 全データのデコード処理
        } else {
            std::cerr << "[Firestore] 監視エラー: " << message << std::endl;
        }
    }
);
```
これで大丈夫です。
DBにデータが追加されたら、そのデータを取得して、もしなにか問題があればエラーを出すというものです。
取得したデータは、`snapshot`に入っていますので、次の処理ではその変数を使っていきます。

### データを構造体にデコードする
次にデータを構造体にデコードしていきましょう。

まずはデコードするための構造体を定義しておきます。
```cpp
struct BallEntity
{
    std::string id;
    firebase::Timestamp createdAt;
    SDL_Point velocity;
    firebase::Timestamp expiresAt;
};
```
こんな感じでいいですかね。Swiftの方とあまり変わらないですね。

次にこの構造体にデコードする関数を定義します。
```cpp
std::vector<BallEntity> decodeDocs(std::vector<firebase::firestore::DocumentSnapshot> docs)
{
    std::vector<BallEntity> ballEntities;
    for (const auto& doc : docs)
    {
        BallEntity ballEntity;
        
        ballEntity.id = doc.id();
        for (const auto& kv : doc.GetData())
        {
            if (kv.first == "created_at")
            {
                ballEntity.createdAt = kv.second.timestamp_value();
            }
            else if (kv.first == "velocity_x")
            {
                ballEntity.velocity.x = kv.second.double_value();
            }
            else if (kv.first == "velocity_y")
            {
                ballEntity.velocity.y = kv.second.double_value();
            }
            else if (kv.first == "expires_at")
            {
                ballEntity.expiresAt = kv.second.timestamp_value();
            }
            else
            { 
                std::cout << "不正なkey: " << kv.first << std::endl;
                Error::showError("デコードに失敗しました。");
            }
        }
        ballEntities.push_back(ballEntity);
    }

    return ballEntities;
}
```
データは、`docs`という引数に格納しておき、これは`vector`なので１つずつデータを読み込んでいます。そして、それぞれのデータは`pair`なので`first`と`second`を使ってデータを取得します。firstにはキーが入っており、secondにはデータが入っています。
また、`docs.id()`でIDが取得できます。

このコードでは、キーからなんのデータか判断して、対応するメンバー変数に入れています。また、その際データの型はジェネラルなので毎回データを変換しなければいけないです。

そして、この関数を先ほどのコードに入れておきましょう。

```cpp
listenerRegistration = query.AddSnapshotListener(
    [](const firebase::firestore::QuerySnapshot& snapshot, firebase::firestore::Error error, const std::string& message)
    {
        if (error == firebase::firestore::kErrorOk) {
            // 全データのデコード処理
            ballEntities = decodeDocs(snapshot.documents());
        } else {
            std::cerr << "[Firestore] 監視エラー: " << message << std::endl;
        }
    }
);
```

ドキュメントは
```cpp
snapshot.documents()
```
で取得できますので、これで大丈夫ですね。

# まとめ
これで、ひとまずはFirestoreにデータを追加してそれをリアルタイムで受け取るという処理を実装できたかと思います。

結構思いつきで始めたことだったんですが、思ったよりも紆余曲折あってちょっと大変でした。
特に、C++側の作業が大変でした...
そもそもこれはモバイル開発向けのサービスなので、PC側で扱うときの資料などあまりなく、どうやったらいいのか試行錯誤することになりました。
また、環境構築も結構面倒くさかったです(もうやりたくない...)。
ですが、まあおもしろいものができたのでやってよかったなと思います！

あと、あくまでもこれは授業の課題なのですが、こんな大掛かりなものはやめたほうがいいんだなと思いました！！

それでは、ここまで読んでくださってありがとうございました！