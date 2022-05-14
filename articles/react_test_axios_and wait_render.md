---
title: "【React】axiosをmockしてstateの変更とrenderを待ってテストを実行するsample" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "Enzyme", "jest", "React Testing Library"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

# 今回使ってるもののバージョン
バージョンを適当に入れていくとうまく動きません。執筆時点ではEnzyme AdapterがReact16までの対応でReact16でReact Testing Libraryを使うには12.1.4以下にする必要がありました。

```
$ npm list --depth=0"
app@0.1.0 /usr/src/app/app
+-- @testing-library/jest-dom@5.16.4
+-- @testing-library/react@12.1.4
+-- @testing-library/user-event@13.5.0
+-- axios@0.27.2
+-- enzyme-adapter-react-16@1.15.6
+-- enzyme@3.11.0
+-- react-dom@16.14.0
+-- react-scripts@5.0.1
+-- react@16.14.0
`-- web-vitals@2.1.4
```

# テスト対象のサンプルコード

ページを開くかreloadボタンを押すと`https://dog.ceo/api/breeds/image/random`から犬画像のURLを取得し、画面に表示するページになります。
このページの`<img>`をテストするためには、外部APIを呼び出しているaxiosのmockを作り、ページを開いてからstateが変更され再renderを待ってから`<img>`をテストする必要があります。


```
import React, { useEffect, useState } from 'react';
import axios from "axios";

function RandomDog() {
  const [isLoading, setIsLoading] = useState(true);
  const [dogImage, setDogImage] = useState('');

  const getDog = async () => {
    setIsLoading(true);
    await axios.get('https://dog.ceo/api/breeds/image/random')
      .then((res) => {
        setDogImage(res.data.message);
        setIsLoading(false);
      })
      .catch(() =>{
        setIsLoading(false);
      })
  }

  useEffect(() => {
    getDog()
  }, [])

  return (
    <>
      {isLoading ? (
        <div>loading</div>
      ) : (
        <div>
          <button onClick={getDog}>  reload </button> <br />
          <img src={dogImage}  alt='dog'/>
        </div>
      )}
    </>
  );
}

export default RandomDog;
```
