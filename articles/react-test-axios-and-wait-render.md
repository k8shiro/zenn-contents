---
title: "【React】axiosをmockしてuseEffectのstateの変更とrenderを待ってテストを実行するsample" # 記事のタイトル
emoji: "🐻" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["React", "Enzyme", "jest", "React Testing Library"] # トピックス（タグ）["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---

Reactで単体テストを書く時のサンプルです。外部APIの呼び出しをmockする必要がありテスト中にページが再renderされるコードに対応しています。

# 今回使ってるもののバージョン
バージョンを適当に入れていくとうまく動きません。執筆時点ではEnzyme AdapterがReact16までの対応でReact16でReact Testing Libraryを使うには12.1.4以下にする必要がありました。  
Enzymeが不要な場合はあまり気にしなくても大丈夫かもしれません。

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
  };

  useEffect(() => {
    getDog()
  }, []);

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

# 1. EnzymeでmountしてReact Testing LibraryのwaitForで再描画待ちするサンプル
Enzymeが使えたほうが便利なので基本的にはこの方法が安定かと思います。  
axiosのmockはjestのspyOnを使います。Enzymeだけだと再描画待ちをしてくれないのでReact Testing LibraryのwaitForを使います。  
waitForの中で`wrapper.update();`していることに注意してください。



```
test('test RandomDog with mount and waitFor', async () => {
  jest.spyOn(axios, 'get').mockResolvedValue({"data": {"message":"https://example.com/test_image.jpg", "status":"success"}});

  let wrapper;
  wrapper = mount(<RandomDog />);

  await waitFor(() => {
    wrapper.update();
    expect(wrapper.find('img').props().src).toBe('https://example.com/test_image.jpg');
  });
});
```

# 2. React Testing LibraryのrenderとwaitForで再描画待ちするサンプル
Enzymeを使わないのでバージョンの依存関係が楽になります。  
waitForで待つのは同じです。

```
test('test RandomDog with render and waitFor ', async () => {
  jest.spyOn(axios, 'get').mockResolvedValue({"data": {"message":"https://example.com/test_image.jpg", "status":"success"}});
  render(<RandomDog />);

  await waitFor(() => {
    const img = screen.getByRole('img');
    expect(img.getAttribute('src')).toBe('https://example.com/test_image.jpg');
  });
});
```

# 3. setTimeoutで再描画待ちするサンプル

無理やり動かしているので非推奨です。再描画をsetTimeoutを使ったsleepで待つようにしてます。actの中に入れないと警告が出るので以下のようなコードになってます。

```
test('test RandomDog with mount and act and setTimeout', async () => {
  jest.spyOn(axios, 'get').mockResolvedValue({"data": {"message":"https://example.com/test_image.jpg", "status":"success"}});
  let wrapper;
  await act(async () => {
    wrapper = mount(<RandomDog />);
    await new Promise((r) => setTimeout(r, 2000));
  });

  wrapper.update()
  expect(wrapper.find('img').props().src).toBe('https://example.com/test_image.jpg');
});
```
